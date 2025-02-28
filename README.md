# ERPNext Installation Guide for Ubuntu 22.04

This guide provides step-by-step instructions for installing ERPNext on an Ubuntu 22.04 server.

## Prerequisites

- **Ubuntu 22.04 Server**: A clean Ubuntu 22.04 server instance.
- **Root or sudo access**: You'll need root or sudo privileges to install software.
- **Internet connection**: For downloading packages.

## Required Software

- Python 3.10+
- Node.js 18+ (LTS)
- Redis 6+
- MariaDB 10.6+
- yarn 1.22+
- pip 23+
- wkhtmltopdf (patched qt)
- cron
- NGINX

---

## Step 1: System Updates and Basic Tools

First, update your system and install essential tools.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl wget software-properties-common -y
```

### Install and Configure Firewall (ufw)

Configuring the firewall before starting the bench is highly recommended.

```bash
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw allow 8000
sudo ufw enable
sudo ufw status
```

---

## Step 2: Install MariaDB

### Add MariaDB repository:

```bash
sudo apt-get install dirmngr
sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirrors.xtom.com/mariadb/repo/10.11/ubuntu jammy main'
sudo apt update
```

### Install MariaDB server:

```bash
sudo apt install mariadb-server -y
```

### Secure MariaDB installation:

```bash
sudo mysql_secure_installation
```

Follow the prompts to set a root password and secure your MariaDB installation.

### Install MariaDB development files:

```bash
sudo apt install libmariadb-dev -y
```

### Configure MariaDB for UTF-8 encoding:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add or modify the following lines under `[mysqld]`:

```ini
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

Add or modify the following lines under `[client-mariadb]`:

```ini
default-character-set = utf8mb4
```

### Restart MariaDB:

```bash
sudo systemctl restart mariadb
```

---

## Step 3: Install Redis and Configure It

### Install Redis
```bash
sudo apt install redis-server -y
```

### Enable Redis to Start on Boot:
```bash
sudo systemctl enable redis
sudo systemctl start redis
```
Verify Redis is running:
```bash
systemctl status redis
```

### Configure Redis for ERPNext:
Open the Redis configuration file:
```bash
sudo vim /etc/redis/redis.conf
```
Modify these settings:
```bash
supervised systemd
maxmemory 256mb
maxmemory-policy allkeys-lru
```
Save and exit Vim (ESC, :wq, then Enter).

Restart Redis to apply changes:
```bash
sudo systemctl restart redis
```

### Check ERPNext Redis Connection:
```bash
bench doctor
```
If Redis is misconfigured, errors related to queueing or caching may appear.

### Enable Redis Queue in ERPNext:
```bash
bench setup redis
bench restart
```

---

## Step 4: Install Node.js and Yarn

### Add Node.js repository:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y
```

### Install Yarn:

```bash
sudo npm install -g yarn
```

---

## Step 5: Install Python and pip

Ubuntu 22.04 comes with Python 3.10 pre-installed. Ensure pip is updated.

```bash
sudo apt install python3-dev python3-venv -y
python3 -m pip install --upgrade pip
```

---

## Step 6: Install wkhtmltopdf

```bash
sudo apt install xvfb libfontconfig wkhtmltopdf -y
```

---

## Step 7: Install frappe-bench

```bash
sudo python3 -m pip install frappe-bench
```

---

## Step 8: Initialize frappe-bench

```bash
bench init frappe-bench --frappe-branch version-14
cd frappe-bench
```

---

## Step 9: Create a New Site

```bash
bench new-site yoursite.com
```

Replace `yoursite.com` with your desired site name.

---

## Step 10: Install ERPNext

```bash
bench get-app erpnext --branch version-14
bench --site yoursite.com install-app erpnext
```

---

## Step 11: Set Admin Password and Migrate

```bash
bench --site yoursite.com set-admin-password yourpassword
bench --site yoursite.com migrate
```

Replace `yourpassword` with a strong password.

---

## Step 12: Start the Bench

```bash
bench start
```

You can now access ERPNext in your browser at `http://your_server_ip:8000`.

---

## Step 13: Production Setup (Optional)

### Create a new user:

```bash
sudo adduser erpnext-user
sudo usermod -aG sudo erpnext-user
su - erpnext-user
```

Repeat steps 8-11 inside the new user's home directory.

### Setup production:

```bash
sudo bench setup production erpnext-user
```

### Restart bench:

```bash
sudo bench restart
```

### Configure NGINX:

```bash
sudo bench setup nginx
sudo systemctl reload nginx
```

### Configure Supervisor:

```bash
sudo systemctl restart supervisor
```

---

## Step 14: Configure DNS and HTTPS (Optional)

### Configure DNS:

Point your domain name to your server's IP address.

### Install Certbot for HTTPS:

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yoursite.com
```

Follow the prompts to configure HTTPS.

---

## Step 15: Multiple Site Configuration

### Disable DNS multitenancy:

```bash
bench config dns_multitenant off
```

### Create a new site:

```bash
bench new-site site2.com
```

### Set NGINX port:

```bash
bench set-nginx-port site2.com 82
```

### Regenerate NGINX configuration:

```bash
bench setup nginx
sudo systemctl reload nginx
```

### Restart Supervisor:

```bash
sudo systemctl restart supervisor
```

---

## Important Notes

- Replace placeholders like `yoursite.com`, `yourpassword`, and `your_server_ip` with your actual values.
- Always use strong passwords.
- Keep your system and ERPNext installation updated.
- Always backup your data before making changes.
- When troubleshooting, use commands like `bench --site yoursite.com doctor`, and check the NGINX and Supervisor logs.
- To update the database of the browsers list, use:
  ```bash
  npx browserslist@latest --update-db
  yarn upgrade caniuse-lite browserslist
  ```
  inside the `frappe-bench` folder.

This comprehensive guide should help you successfully install and configure ERPNext on your Ubuntu 22.04 server. Remember to adapt the steps to your specific environment.
