---
categories:
- DevOps
- Networking
- Linux
- Tutorial
- Web Development
comments: true
cover:
  image: https://images.pexels.com/photos/1054397/pexels-photo-1054397.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Learn how to host a static website locally using a Raspberry Pi or, in
  limited cases, your home router. This guide covers setup, web server configuration
  (Nginx, Apache), network basics, and making your site accessible, all with practical
  command-line examples for developers.
tags:
- Linux
- Networking
- Web Hosting
- Raspberry Pi
- Nginx
- Apache
- Router
- Self-Hosting
- CLI
- DevOps
title: How to Host a Static Site from Your Router or Raspberry Pi
---

![Close-up of network cables and ports in a server rack, showcasing connectivity.](https://images.pexels.com/photos/1054397/pexels-photo-1054397.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Close-up of network cables and ports in a server rack, showcasing connectivity.")

## How to Host a Static Site from Your Router or Raspberry Pi

Hosting your own static website from home might sound like something from the early internet, but for personal projects, internal tools, or just learning, it's incredibly powerful. You gain complete control, learn invaluable networking and server administration skills, and avoid recurring cloud hosting fees for small projects.

This post will guide you through setting up a static site server using a Raspberry Pi (the recommended and most flexible approach) and briefly touch on using a router's built-in capabilities (which are often very limited). We'll focus on minimal, command-line-driven setups for developers who prefer getting their hands dirty.

## Why Host Locally?

*   **Control**: You own the hardware and the software stack.
*   **Cost-Effective**: After the initial hardware purchase (e.g., a Raspberry Pi), there are no monthly hosting fees.
*   **Privacy/Security**: For internal tools or private data, keeping it on your local network can be more secure than public cloud providers.
*   **Learning**: It's a fantastic way to understand web servers, networking, and Linux administration.
*   **Specific Use Cases**: Perfect for smart home dashboards, local documentation, small family photo galleries, or testing web projects before deployment.

**Note**: This setup is generally not suitable for high-traffic public websites. Home internet connections often have limited upstream bandwidth, and residential ISPs might block common ports like 80 (HTTP) and 443 (HTTPS) or change your public IP address frequently.

## The Basics of Local Web Serving

A static site consists of files like HTML, CSS, JavaScript, and images. It doesn't require server-side processing (like PHP, Python, or Node.js to generate pages). A web server's job is simply to listen for requests, locate the requested file, and send it back to the client.

Core components you'll interact with:

*   **Web Server Software**: Nginx or Apache are the industry standards.
*   **Network Configuration**: Ensuring your server has a stable IP address and can be reached.
*   **Static Site Files**: Your actual `index.html`, `style.css`, etc.

## Hosting on a Raspberry Pi (Recommended Approach)

The Raspberry Pi is an excellent choice for a low-power, dedicated home server. It runs a full Linux distribution (Raspberry Pi OS), giving you complete control.

### 1. Raspberry Pi Setup & Network Configuration

First, ensure your Raspberry Pi OS is up-to-date and has a stable network configuration.

#### Update Your Pi

```bash
sudo apt update && sudo apt upgrade -y
```

```output
Get:1 http://raspbian.raspberrypi.org/raspbian bullseye/main armhf Packages [13.0 MB]
Get:2 http://raspbian.raspberrypi.org/raspbian bullseye/main armhf Release.gpg [2,504 B]
...
Setting up some-package (1.2.3-4) ...
Processing triggers for man-db (2.9.4-2) ...
```

#### Set a Static IP Address

For a server, a static IP is crucial so its address doesn't change, breaking your server configuration or port forwarding rules. We'll configure this by editing `/etc/dhcpcd.conf`.

**Find your current IP, Gateway, and DNS:**

```bash
ip addr show eth0 # Or wlan0 if using Wi-Fi
ip route show default
cat /etc/resolv.conf | grep nameserver
```

```output
# Example `ip addr show eth0` output
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86241sec preferred_lft 75441sec

# Example `ip route show default` output
default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.100 metric 202

# Example `cat /etc/resolv.conf | grep nameserver` output
nameserver 192.168.1.1
```

From the output, you can typically identify:
*   Your Pi's current IP: `192.168.1.100` (this will become your static IP)
*   Your router/gateway IP: `192.168.1.1`
*   Your DNS server IP: `192.168.1.1` (often the same as your gateway)

**Edit `/etc/dhcpcd.conf`**:

```bash
sudo nano /etc/dhcpcd.conf
```

Scroll to the end of the file and add (or uncomment and modify) a section like this, replacing the example IPs with your network's details:

```conf
# Example static IP configuration for eth0 (Ethernet)
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
```

*   `static ip_address`: Your chosen static IP for the Pi, followed by the CIDR subnet mask (e.g., `/24` for most home networks).
*   `static routers`: Your router's IP address.
*   `static domain_name_servers`: Your DNS server(s). You can use your router's IP and/or public DNS like Google (8.8.8.8) or Cloudflare (1.1.1.1).

Save and exit (`Ctrl+X`, then `Y`, then `Enter`).

**Apply the changes:**

```bash
sudo systemctl reboot
```

After rebooting, verify your IP address:

```bash
ip addr show eth0 | grep "inet "
```

```output
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
```

Great! Your Pi now has a consistent IP address on your local network.

### 2. Choosing and Configuring a Web Server

You have two primary choices: Nginx or Apache. For serving static files, Nginx is often preferred due to its lightweight nature and high performance. Apache is more feature-rich but can be heavier. We'll cover Nginx first.

#### Option A: Nginx (Recommended for Static Sites)

##### Install Nginx

```bash
sudo apt install nginx -y
```

```output
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  nginx-common nginx-core
Suggested packages:
  fcgiwrap nginx-doc
The following NEW packages will be installed:
  nginx nginx-common nginx-core
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 852 kB of archives.
After this operation, 3,116 kB of additional disk space will be used.
...
Setting up nginx-common (1.18.0-6.1) ...
Setting up nginx-core (1.18.0-6.1) ...
Setting up nginx (1.18.0-6.1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
```

Nginx should start automatically. You can check its status:

```bash
sudo systemctl status nginx
```

```output
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-10-27 10:30:00 UTC; 5s ago
       Docs: man:nginx(8)
    Process: 1234 ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;' (code=exited, status=0/SUCCESS)
    Process: 1235 ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;' (code=exited, status=0/SUCCESS)
   Main PID: 1236 (nginx)
      Tasks: 2 (limit: 4915)
        CPU: 12ms
     CGroup: /system.slice/nginx.service
             ├─1236 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─1237 "nginx: worker process"
```

##### Nginx Configuration

Nginx serves files from `/var/www/html` by default. Let's create a dedicated directory for our site and a new Nginx configuration.

1.  **Create your site directory**:

    ```bash
    sudo mkdir -p /var/www/my-static-site
    sudo chown -R pi:pi /var/www/my-static-site # Give your user ownership
    ```

2.  **Create a simple `index.html`**:

    ```bash
    nano /var/www/my-static-site/index.html
    ```

    Paste the following content:

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>My Awesome Static Site</title>
        <style>
            body { font-family: sans-serif; text-align: center; margin-top: 50px; background-color: #f0f0f0; }
            h1 { color: #333; }
            p { color: #666; }
            .container { background-color: white; padding: 30px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); display: inline-block; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>Hello from My Raspberry Pi!</h1>
            <p>This is a static site proudly served by Nginx.</p>
            <p>Current server time: <span id="time"></span></p>
        </div>
        <script>
            function updateTime() {
                const now = new Date();
                document.getElementById('time').textContent = now.toLocaleString();
            }
            setInterval(updateTime, 1000);
            updateTime(); // Initial call
        </script>
    </body>
    </html>
    ```

    Save and exit.

3.  **Create a new Nginx server block configuration**:

    ```bash
    sudo nano /etc/nginx/sites-available/my-static-site.conf
    ```

    Add the following:

    ```nginx
    server {
        listen 80;
        listen [::]:80;

        root /var/www/my-static-site;
        index index.html index.htm;

        server_name your_pi_ip_address_or_domain; # e.g., 192.168.1.100 or mypisite.com

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

    **Note**: Replace `your_pi_ip_address_or_domain` with your Pi's static IP (e.g., `192.168.1.100`). If you later set up a domain name, you'd put that here.

    Save and exit.

4.  **Enable the new site and disable the default Nginx page**:

    ```bash
    sudo ln -s /etc/nginx/sites-available/my-static-site.conf /etc/nginx/sites-enabled/
    sudo rm /etc/nginx/sites-enabled/default
    ```

5.  **Test Nginx configuration for syntax errors**:

    ```bash
    sudo nginx -t
    ```

    ```output
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

    If you see "syntax is ok" and "test is successful", you're good to go.

6.  **Reload Nginx to apply changes**:

    ```bash
    sudo systemctl reload nginx
    ```

#### Option B: Apache2 (Alternative)

If you prefer Apache or have existing Apache configurations, here's how to set it up.

##### Install Apache2

```bash
sudo apt install apache2 -y
```

```output
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap
Suggested packages:
  apache2-doc apache2-suexec-custom | apache2-suexec-pristine www-browser
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap
0 upgraded, 8 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,514 kB of archives.
After this operation, 5,528 kB of additional disk space will be used.
...
Setting up apache2-bin (2.4.54-1~deb11u1) ...
Setting up apache2 (2.4.54-1~deb11u1) ...
Enabling module mpm_event.
Enabling module authz_core.
...
Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /lib/systemd/system/apache2.service.
```

Apache should start automatically. Check its status:

```bash
sudo systemctl status apache2
```

```output
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-10-27 10:45:00 UTC; 5s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 5678 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 5679 (apache2)
      Tasks: 5 (limit: 4915)
        CPU: 18ms
     CGroup: /system.slice/apache2.service
             ├─5679 /usr/sbin/apache2 -k start
             ├─5680 /usr/sbin/apache2 -k start
             ├─5681 /usr/sbin/apache2 -k start
             ├─5682 /usr/sbin/apache2 -k start
             └─5683 /usr/sbin/apache2 -k start
```

##### Apache Configuration

Apache serves files from `/var/www/html` by default. Let's reuse the `/var/www/my-static-site` directory we created earlier.

1.  **Create a new Apache Virtual Host configuration**:

    ```bash
    sudo nano /etc/apache2/sites-available/my-static-site.conf
    ```

    Add the following:

    ```apache
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/my-static-site
        ServerName your_pi_ip_address_or_domain # e.g., 192.168.1.100 or mypisite.com

        <Directory /var/www/my-static-site>
            Options Indexes FollowSymLinks
            AllowOverride None
            Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```

    **Note**: Replace `your_pi_ip_address_or_domain` with your Pi's static IP (e.g., `192.168.1.100`).

    Save and exit.

2.  **Enable the new site and disable the default Apache page**:

    ```bash
    sudo a2ensite my-static-site.conf
    sudo a2dissite 000-default.conf
    ```

    ```output
    Enabling site my-static-site.
    To activate new configuration, you need to run:
      systemctl reload apache2
    Disabling site 000-default.
    To activate new configuration, you need to run:
      systemctl reload apache2
    ```

3.  **Test Apache configuration**:

    ```bash
    sudo apache2ctl configtest
    ```

    ```output
    Syntax OK
    ```

4.  **Reload Apache to apply changes**:

    ```bash
    sudo systemctl reload apache2
    ```

### 3. Configure the Firewall (UFW)

It's crucial to enable a firewall to protect your Pi. UFW (Uncomplicated Firewall) is easy to use.

```bash
sudo apt install ufw -y
```

```output
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  ufw
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 158 kB of archives.
After this operation, 1,024 kB of additional disk space will be used.
...
Setting up ufw (0.36.1-4) ...
```

**Allow SSH (so you don't lock yourself out!):**

```bash
sudo ufw allow ssh
```

```output
Rules updated
Rules updated (v6)
```

**Allow HTTP (port 80) and HTTPS (port 443):**

If you used Nginx:

```bash
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS' # For later when you set up SSL
```

If you used Apache:

```bash
sudo ufw allow 'Apache'
sudo ufw allow 'Apache Full' # For later when you set up SSL
```

**Enable UFW:**

```bash
sudo ufw enable
```

```output
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

**Check UFW status:**

```bash
sudo ufw status verbose
```

```output
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
443/tcp (v6)               ALLOW IN    Anywhere (v6)
```

Your Pi is now serving your static site!

### 4. Test Your Local Static Site

From any device on the *same local network* (e.g., your laptop, phone), open a web browser and navigate to your Pi's static IP address:

`http://192.168.1.100` (replace with your actual Pi IP)

You should see your "Hello from My Raspberry Pi!" page.

## Hosting on Your Router (Limited Capabilities)

Some consumer routers come with basic web server functionality, typically tied to a USB port where you can plug in a flash drive. This is **severely limited** compared to a Raspberry Pi and is generally **not recommended** for anything beyond internal file sharing or an *extremely* basic personal page.

**Limitations you'll face**:

*   **No SSL/HTTPS**: Almost certainly. Your site will be insecure (HTTP only).
*   **No custom configuration**: No virtual hosts, no redirects, no advanced features.
*   **Performance**: Very slow, especially with multiple simultaneous connections.
*   **Reliability**: Router firmware can be buggy, and features might be unstable.
*   **Security**: Router firmware often has known vulnerabilities that aren't patched quickly.
*   **File System**: You'll usually be limited to placing files on a USB drive in a specific folder (e.g., `/www` or `/web`).

### How it Generally Works (Conceptual Steps):

1.  **Plug in a USB drive**: Format it correctly (FAT32 or NTFS are common).
2.  **Place your static files**: Copy your `index.html`, `css/`, `js/` into a designated folder on the USB drive (e.g., a folder named `web` or `www`).
3.  **Access Router Settings**: Log in to your router's web interface (usually `http://192.168.1.1` or `http://192.168.0.1`).
4.  **Find "Web Server" or "Media Server" Feature**: This is often found under "USB Applications," "Storage," or "Advanced Settings."
5.  **Enable the Feature**: There might be a toggle or checkbox.
6.  **Note the URL/IP**: The router will tell you the local IP address and port (often port 80 or 8080) where the web server is accessible.

**Example (Conceptual Router UI)**:

![Conceptual Router Web Server UI](https://via.placeholder.com/600x300?text=Conceptual+Router+Web+Server+Settings)
*(Image is a placeholder as actual router interfaces vary wildly)*

**To test**: Open a browser on a device on your local network and navigate to the IP address and port your router specifies (e.g., `http://192.168.1.1:8080`).

**Verdict on Router Hosting**: Use a Raspberry Pi. Seriously.

## Making Your Site Accessible Beyond Your Local Network

If you want your static site to be reachable from the internet, you'll need to configure your router and possibly a Dynamic DNS service.

**Note**: Before exposing your server to the internet, ensure your Raspberry Pi's OS and web server are fully updated and your firewall (UFW) is correctly configured.

### 1. Port Forwarding

Your router acts as a gatekeeper. External requests hit your router's public IP address. Port forwarding tells the router to send requests on a specific port (e.g., 80 for HTTP, 443 for HTTPS) to a specific internal IP address (your Pi's static IP) and port.

**Steps (General)**:

1.  **Log in to your router's web interface.**
2.  **Find "Port Forwarding," "NAT," "Virtual Servers," or similar.** The exact name varies by router manufacturer.
3.  **Create a new rule for HTTP (Port 80):**
    *   **Service Port / External Port**: `80`
    *   **Internal Port**: `80`
    *   **Internal IP Address / Server IP**: Your Pi's static IP (e.g., `192.168.1.100`)
    *   **Protocol**: `TCP` (or `Both/TCP/UDP`)
    *   **Description**: `HTTP Server`
4.  **Create a new rule for HTTPS (Port 443):** (Highly recommended after setting up SSL)
    *   **Service Port / External Port**: `443`
    *   **Internal Port**: `443`
    *   **Internal IP Address / Server IP**: Your Pi's static IP
    *   **Protocol**: `TCP`
    *   **Description**: `HTTPS Server`
5.  **Save/Apply the changes.**

**Note**: Some ISPs block common ports like 80 and 443 for residential customers. If your site isn't accessible after port forwarding, check with your ISP or try forwarding to a non-standard port (e.g., `8080` external to `80` internal), though this requires users to specify the port in the URL (`http://your-domain.com:8080`).

### 2. Dynamic DNS (DDNS)

Most home internet connections have dynamic public IP addresses, meaning your ISP can change your IP address at any time. This is a problem if you want to access your site via a domain name (e.g., `my-pi-site.com`). Dynamic DNS services solve this by providing a hostname that automatically updates to point to your current public IP address.

**Popular DDNS Providers**:

*   **No-IP**
*   **DuckDNS**
*   **Dynu**
*   **Cloudflare DNS** (if you use their DNS for your domain, you can use their API to update records)

**Setting up DDNS**:

1.  **Sign up with a DDNS provider** and create a hostname (e.g., `myraspi.ddns.net`).
2.  **Configure the DDNS client**:
    *   **On your router**: Many routers have built-in DDNS clients. This is the easiest method. Find the DDNS section in your router's settings and enter your provider's credentials.
    *   **On your Raspberry Pi**: If your router doesn't support your chosen provider, you can install a client on your Pi. For example, for DuckDNS:

        ```bash
        mkdir -p /home/pi/duckdns
        cd /home/pi/duckdns
        nano duck.sh
        ```

        ```bash
        echo url="https://www.duckdns.org/update?domains=YOUR_SUBDOMAIN&token=YOUR_TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
        ```

        Replace `YOUR_SUBDOMAIN` and `YOUR_TOKEN` with your DuckDNS details.
        Make it executable: `chmod 700 duck.sh`
        Schedule it with cron to run every 5 minutes: `crontab -e` and add:

        ```cron
        */5 * * * * /home/pi/duckdns/duck.sh >/dev/null 2>&1
        ```

Now, your chosen DDNS hostname (`myraspi.ddns.net`) should always point to your home's public IP address.

### 3. HTTPS with Let's Encrypt

For any site accessible over the internet, **HTTPS is mandatory**. It encrypts communication, protecting your visitors' privacy and trust. Let's Encrypt provides free SSL/TLS certificates, and `certbot` automates the process.

**Prerequisites**:

*   A domain name (or a DDNS hostname) pointing to your public IP.
*   Port 80 and 443 forwarded to your Pi.

**Install Certbot**:

```bash
sudo apt install certbot python3-certbot-nginx -y # For Nginx
# OR
sudo apt install certbot python3-certbot-apache -y # For Apache
```

```output
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
...
Setting up python3-certbot-nginx (1.12.0-2) ...
Setting up certbot (1.12.0-2) ...
```

**Obtain and install SSL certificate**:

```bash
sudo certbot --nginx # For Nginx
# OR
sudo certbot --apache # For Apache
```

```output
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): your_email@example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-April-3-2023.pdf. You must agree in
order to register with the ACME server.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you emails about our work
and opportunities to support Internet freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: myraspi.ddns.net
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number[s] from the list above (comma-separated, empty
for all): 1
Requesting a certificate for myraspi.ddns.net

Successfully received certificate.
Successfully deployed certificate for myraspi.ddns.net to /etc/nginx/sites-enabled/my-static-site.conf
Congratulations! You have successfully enabled HTTPS on https://myraspi.ddns.net
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
...
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Your certificate will expire on 2024-01-25. To obtain a new or tweaked
version of this certificate in the future, simply run certbot again. To
unattendedly renew *all* of your certificates, run "certbot renew"
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Follow the prompts. Certbot will automatically modify your Nginx/Apache configuration, enable HTTPS, and set up automatic renewals. Now, your site should be accessible via `https://myraspi.ddns.net` (using your DDNS hostname).

## Troubleshooting & Common Issues

*   **"Site not reachable" on local network**:
    *   **Check Pi's IP**: `ip addr show eth0`. Is it the static IP you configured?
    *   **Web server status**: `sudo systemctl status nginx` or `sudo systemctl status apache2`. Is it active (running)?
    *   **Firewall**: `sudo ufw status verbose`. Are ports 80 (and 443) allowed? Temporarily disable (`sudo ufw disable`) to test if it's the culprit, then re-enable.
    *   **Wrong DocumentRoot/web directory**: Ensure your `index.html` is in the path specified in your Nginx/Apache config.
*   **"Site not reachable" from internet**:
    *   **Port forwarding**: Double-check your router's port forwarding rules. Is the external port correctly mapped to your Pi's *internal* IP and port?
    *   **Public IP**: What's your current public IP? Search "what is my ip" on Google. Does your DDNS hostname point to this IP?
    *   **ISP blocking ports**: Call your ISP or check their terms of service. Some residential ISPs block inbound connections on ports 80 and 443.
    *   **Router Firewall**: Some routers have their own firewall features separate from port forwarding that might block traffic.
    *   **DDNS client**: Is your DDNS client running and successfully updating? Check its logs.
*   **`certbot` fails**:
    *   Ensure your domain/DDNS is pointing to your public IP *before* running `certbot`. Let's Encrypt needs to verify domain ownership.
    *   Make sure port 80 is forwarded and accessible from the internet, as `certbot` often uses HTTP-01 challenge.

## Security Considerations

When exposing your Raspberry Pi to the internet:

*   **Keep your Pi updated**: Regularly run `sudo apt update && sudo apt upgrade -y`.
*   **Strong Passwords**: For your Pi's `pi` user (or any other user), and for your router's admin interface. Change default passwords immediately.
*   **Limit Exposed Services**: Only port forward necessary ports (80, 443, and 22 for SSH if you need remote access). Disable SSH password authentication and use SSH keys for better security.
*   **Use HTTPS**: Always.
*   **Firewall (UFW)**: Keep it enabled and configured to only allow necessary ports.
*   **No Sensitive Data**: Avoid hosting highly sensitive personal or business data on a home server unless you fully understand and implement advanced security measures.

## Conclusion

Hosting a static site from your Raspberry Pi is an excellent project for developers. It's a pragmatic way to host personal projects, learn server administration, and gain a deeper understanding of web infrastructure. While a router's built-in web server might seem tempting, the flexibility, power, and security of a Raspberry Pi make it the undisputed champion for this task.

Embrace the DIY spirit, and happy self-hosting!
