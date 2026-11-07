# Headless Debian Web Server on Orange Pi 3 LTS

A headless Linux web server made with **Orange Pi 3 LTS** single-board computer.

## Project Overview & Specifications

* **Hardware**: Orange Pi 3 LTS (Allwinner H6 Quad-Core ARM SoC, 2GB LPDDR3 RAM, 8GB eMMC Flash)
* **Power Supply**: 5V / 3A USB-C Power Adapter
* **Cooling**: Passive Aluminum & Copper Heatsink Set
* **Operating System**: Debian 12 (Bookworm) / Armbian Minimal CLI
* **Web Stack**: Nginx Web Server
* **DNS & Security**: DuckDNS (Dynamic DNS), Let's Encrypt Certbot (HTTPS / SSL)
* **Local Networking**: `dnsmasq` for static routing and local DHCP leases

## Step 1: Operating System Installation and eMMC Migration

### 1.1 Flash the Initial Image
1. Download the **Debian Minimal CLI** image for Orange Pi 3 LTS.
2. Flash the `.img` file onto a MicroSD card using [BalenaEtcher](https://etcher.balena.io/).
3. Insert the MicroSD card into the Orange Pi 3 LTS, connect the primary Gigabit Ethernet port to your router, and plug in the 5V/3A USB-C power supply.

### 1.2 Initial Headless SSH Access
Find the device's IP address using your router's admin panel or a network scanner (`nmap` / *Fing* app), then connect via SSH (windows):

# ssh root@<ORANGE_PI_IP>

(Default password is typically 1234)
1.3 Migrate OS to Onboard eMMC Flash
To drastically improve read/write speeds and system durability compared to SD cards, transfer the system directly to the 8GB integrated eMMC memory:
sudo nand-sata-installer

 * Select Boot from eMMC - system on eMMC.
 * Follow the on-screen prompts and wait for the transfer to complete.
 * Shut down the system (sudo poweroff), remove the MicroSD card, and power back on. The server will now boot blazingly fast directly from the eMMC!
# Step 2: Network Configuration & Local DHCP (dnsmasq)
This setup configures a secondary network interface (e.g., USB-to-Ethernet adapter or second port eth1) with a static IP and a local DHCP server to directly connect a client laptop.
2.1 Set Up a Static IP Interface
Edit the network interfaces configuration file:
sudo nano /etc/network/interfaces

Append the static IP configuration for the secondary interface (eth1):
auto eth1
iface eth1 inet static
    address 192.168.50.1
    netmask 255.255.255.0

Apply changes:
sudo ifdown eth1 && sudo ifup eth1

2.2 Install and Configure dnsmasq
Install dnsmasq to act as the local DHCP server for connected client devices:
sudo apt update && sudo apt install -y dnsmasq

Edit /etc/dnsmasq.conf:
sudo nano /etc/dnsmasq.conf

Add the following configuration to the bottom of the file:
interface=eth1
dhcp-range=192.168.50.10,192.168.50.50,255.255.255.0,12h
dhcp-option=option:router,192.168.50.1
dhcp-option=option:dns-server,1.1.1.1,8.8.8.8

Restart and enable the service:
sudo systemctl restart dnsmasq
sudo systemctl enable dnsmasq

# Step 3: Web Server, Dynamic DNS & SSL Certificate
3.1 Install Nginx & Deploy Landing Page
Install the Nginx web server:
sudo apt install -y nginx

Create a custom index.html landing page at /var/www/html/index.html:
sudo nano /var/www/html/index.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Debian Web Server</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #121212; color: #ffffff; text-align: center; padding-top: 100px; }
        h1 { color: #d70a53; }
        .card { background: #1e1e1e; padding: 20px; display: inline-block; border-radius: 8px; box-shadow: 0 4px 10px rgba(0,0,0,0.5); }
    </style>
</head>
<body>
    <div class="card">
        <h1>Server Online </h1>
        <p>Hosted on an <strong>Orange Pi 3 LTS</strong> running <strong>Debian Linux</strong> (eMMC Storage).</p>
    </div>
</body>
</html>

3.2 Automate Dynamic DNS (DuckDNS)
Create an account on DuckDNS.org and register a domain (e.g., myproject.duckdns.org).
Create the automatic updater script on the server:
sudo nano /usr/local/bin/duckdns.sh

Paste the following script (replace YOUR_DOMAIN and YOUR_TOKEN):
#!/bin/bash
echo url="[https://www.duckdns.org/update?domains=YOUR_DOMAIN&token=YOUR_TOKEN&ip=](https://www.duckdns.org/update?domains=YOUR_DOMAIN&token=YOUR_TOKEN&ip=)" | curl -k -s -K -

Make it executable:
sudo chmod +x /usr/local/bin/duckdns.sh

Schedule it to run every 5 minutes via cron:
sudo crontab -e

Add this line at the bottom:
*/5 * * * * /usr/local/bin/duckdns.sh >/dev/null 2>&1

3.3 Enable HTTPS with Let's Encrypt (Certbot)
Install Certbot and the Nginx plugin:
sudo apt install -y certbot python3-certbot-nginx

Obtain and automatically configure the SSL/TLS certificate:
sudo certbot --nginx -d YOUR_DOMAIN.duckdns.org

Certbot will verify your domain, apply the SSL certificates, and automatically configure Nginx to force secure HTTPS traffic!
  Performance & Hardware Notes
 * Power Consumption: ~2.5W - 4W under standard headless operation.
 * Thermal Management: Passively cooled with copper/aluminum heatsinks; temperatures stay within optimal ranges (~40°C - 50°C) during standard server operations.
 * Storage Performance: The onboard 8GB eMMC offers superior random read/write speeds (4K I/O) over standard Class 10 SD cards, preventing filesystem corruption during continuous long-term uptime.

# 3d printed case for the server

I already have made a 3d printer case for it, you can download it from my repo, or make a new one for yourself.


