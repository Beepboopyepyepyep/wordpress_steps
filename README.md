# wordpress_steps
Steps taken to set up a self-hosted wordpress on GCP VM with firewall and server hardening for security.


# Deploying WordPress on GCP

## 1. Overview

Documenting the steps for deploying a WordPress site on Google Cloud Platform (GCP) VM.  
It includes steps to set up the server, install WordPress, secure it with HTTPS, set up a free domain via No-IP, and harden the system, protect WordPress Site with GCP Firewall.

## 2. GCP Free Tier Setup

- Create a GCP account: https://console.cloud.google.com 
- Spin up non-preemptible e2-micro VM instance with OS: **Ubuntu 22.04 LTS** in Iowa: us-central1
- Generate SSH key and add it to the GCP VM

## 3. SSH Access and Firewall Configuration

- SSH via terminal
- Create a new sudo user:
  ```bash
  sudo adduser wordpress
  sudo usermod -aG sudo wordpress
  ```
- Enable and configure UFW firewall:
  ```bash
  sudo ufw allow OpenSSH
  sudo ufw allow 'Apache Full'
  sudo ufw enable
  ```

## 4. Installing LAMP Stack and WordPress

- Update the server:
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

- Install Apache, MySQL, PHP and other tools:
  ```bash
  sudo apt install apache2 mysql-server php php-mysql libapache2-mod-php unzip curl -y
  ```

- Download and set up WordPress:
  ```bash
  cd /var/www/html
  sudo rm index.html
  sudo curl -O https://wordpress.org/latest.tar.gz
  sudo tar -xvzf latest.tar.gz
  sudo mv wordpress/* .
  sudo chown -R www-data:www-data /var/www/html
  sudo chmod -R 755 /var/www/html
  ```

## 5. MySQL Database Setup

- Secure MySQL:
  ```bash
  sudo mysql_secure_installation
  ```

- Create database and user:
  ```sql
  sudo mysql -u root -p

  CREATE DATABASE wp_db;
  CREATE USER 'wp_user'@'localhost' IDENTIFIED BY '<PASSWORD>';
  GRANT ALL PRIVILEGES ON wp_db.* TO 'wp_user'@'localhost';
  FLUSH PRIVILEGES;
  EXIT;
  ```

## 6. Free Domain with No-IP

- Sign up: https://www.noip.com and create a free subdomain.
- Visit the wordpress No-IP subdomain to complete WordPress installation.

## 7. SSL Certificate with Let’s Encrypt

- Install Certbot:
  ```bash
  sudo apt install certbot python3-certbot-apache -y
  ```
- Get and install the SSL certificate:
  ```bash
  sudo certbot --apache
  ```

- Test renewal:
  ```bash
  sudo certbot renew --dry-run
  ```
 
- Verify autorenewal:
  ```bash
  systemctl list-timers | grep certbot
  ```

## 8. GCP Firewall Rules 

- Go to **VPC Network > Firewall Rules** in GCP Console, enable firewall with the following rules:
    - Disallow port **80**.
    - Allow port **443**.
    - Only allow access from my public IP.

- Attach the rules to VM using **tags**.

- Save and Test
    - Visit the site to confirm port **80** is disabled.
    - Verify HTTPS is working.

## 9. Server Hardening

- **Disable root SSH login**:
  Edit `/etc/ssh/sshd_config` and set:
  ```
  PermitRootLogin no
  ```

- **Install Fail2Ban**:
  ```bash
  sudo apt install fail2ban
  ```

- **Disable http port on ufw**:
  ```bash
  sudo ufw deny 80
  sudo ufw status numbered
  ```

## 10. Final Steps and Testing

- Document the work in a **GitHub README** with:
  - All commands
  - Screenshots
  - Description of each step

