# WordPress-Deployment-Assignment 
   
[https://wpwebsiteproject.ddns.net/](https://wpwebsiteproject.ddns.net/)

# 🚀 Automated WordPress Deployment using LEMP Stack and GitHub Actions 🚀

Welcome to a step-by-step guide on setting up an automated deployment pipeline for your WordPress website using the powerful LEMP stack (Linux, Nginx, MySQL, PHP) on an AWS EC2 instance. And that's not all – we're integrating the magic of GitHub Actions to make this process as smooth as possible.

## 📋 Prerequisites

To embark on this exciting journey, you'll need a few things in place:

- **AWS Account and EC2 Instance:** Make sure you've got an AWS account set up and an EC2 instance ready to roll.
- **Domain or Temporary Hostname:** Secure a domain name or have a temporary hostname at the ready.
- **GitHub Account:** You'll need a GitHub account for version control and collaborative magic.

## 🚀 Server Provisioning on AWS

```shell
# Launch an EC2 Instance:
# Choose an Ubuntu 22.04 AMI to start.
# Configure security groups for HTTP (port 80) and HTTPS (port 443) traffic.
# Create or select an SSH key pair for safe access.

# Allocate an Elastic IP (Optional but Recommended):
# Navigate to "Elastic IPs" in your AWS Console.
# Allocate a fresh Elastic IP and associate it with your EC2 instance.

# SSH into Your Instance:
ssh -i wp_server.pem ubuntu@3.230.213.39

# Install the LEMP Stack:
# Elevate your server's capabilities:
sudo apt update && sudo apt upgrade -y
sudo apt install nginx mysql-server php-fpm php-mysql -y

**🎉 WordPress Installation and Configuration**
 ```shell
# Securely Configure MySQL:
# Tighten up MySQL and set up your database:
sudo mysql_secure_installation
# Inside MySQL
CREATE DATABASE wordpressdb;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpressdb.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

# Download and Configure WordPress:
cd tmp
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
sudo cp -a /tmp/wordpress/.  /var/www/html
sudo chown -R www-data:www-data /var/www/html/

**Set up your wp-config.php with the database details.**

**⚙️ Nginx Configuration**

# Configure Nginx Server Block:
sudo vi /etc/nginx/sites-available/wordpress

# Craft your Nginx server block configuration and enable it:
server {
   listen 80;
   server_name wpwebsiteproject.ddns.net www.wpwebsiteproject.ddns.net;   
   root /var/www/html/;   
   index index.php;

   location / {
       try_files $uri $uri/ /index.php?$args;
   }

   location ~ \.php$ {
       include snippets/fastcgi-php.conf;
       fastcgi_pass unix:/run/php/php8.1-fpm.sock;   
   }
}

# Update wp-config.php with Database Details
cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo vi /var/www/html/wp-config.php

# Update the following lines in wp-config.php
define('DB_NAME', 'wordpressdb');
define('DB_USER', 'wordpressuser');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'localhost');

# Activate your configuration and restart Nginx:
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Enable HTTPS with Let's Encrypt:
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx wpwebsiteproject.ddns.net 

# Optimize Nginx for Performance:
# Edit /etc/nginx/nginx.conf and enable Gzip compression:
gzip on;
gzip_types ...;

# Supercharge your site with browser caching:
location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
    expires max;
    log_not_found off;
}

**🚚 GitHub Actions Workflow Setup**


GitHub Actions Workflow Setup

To automate the deployment process, we will be using GitHub Actions, which allows you to define workflows in code.
Follow these steps to set up the workflow for your WordPress website:

1. **Create a New GitHub Repository:**

   Start by creating a new repository on GitHub to host your WordPress project.

2. **Configure GitHub Actions Workflow:**

   Inside your repository, create a directory named `.github/workflows`. In this directory, create a file named `deploy.yml`.
   # GitHub Actions Workflow Setup

Automate your WordPress deployment using GitHub Actions with this step-by-step guide. Below are the contents of your `deploy.yml` and `deploy_script.sh` files.

## 1. `deploy.yml` - GitHub Actions Workflow Configuration

```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up SSH key
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -t rsa 3.230.213.39 >> ~/.ssh/known_hosts
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Install dependencies
        run: |
          sudo apt update && sudo apt install -y \
            nginx mysql-client
          # Add your specific dependency installation and build commands here

      - name: Transfer files to server
        run: |
          scp -r * ubuntu@3.230.213.39:/var/www/html/

      - name: Run deployment script on server
        run: |
          ssh ubuntu@3.230.213.39 '/var/www/html/deploy_script.sh'

      - name: Restart services on server
        run: |
          ssh ubuntu@3.230.213.39 "sudo systemctl restart nginx"

2. deploy_script.sh - Deployment Script on Server

  #!/bin/bash

# Set up variables
SERVER_IP="3.230.213.39"
REMOTE_DIR="/var/www/html/"

# Connect to the server and perform deployment tasks
ssh ubuntu@$SERVER_IP "cd $REMOTE_DIR && \
                      sudo sed -i 's/database_name_here/wordpressdb/' wp-config.php && \
                      sudo sed -i 's/username_here/wordpressuser/' wp-config.php && \
                      sudo sed -i 's/password_here/$DB_PASSWORD/' wp-config.php && \
                      sudo chown -R www-data:www-data $REMOTE_DIR && \
                      sudo systemctl restart nginx"


Safeguard Secrets using GitHub Repository Secrets:

Go to "Settings" > "Secrets" > "New repository secret".
Create secrets like SSH_PRIVATE_KEY and DB_PASSWORD.
Commit and Push Your Workflow:
Commit and push deploy.yml to your repository. GitHub Actions will trigger on pushes to the main branch.

**Congratulations!  successfully set up an automated deployment pipeline for your WordPress website using the LEMP stack, Nginx, and GitHub Actions. Your website is now ready to shine on the web.**

https://wpwebsiteproject.ddns.net/
  

