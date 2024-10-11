# EC2 Node.js App Deployment with Nginx and SSL

This guide will walk you through deploying a Node.js application on an AWS EC2 instance running Ubuntu, setting up Nginx as a reverse proxy, and securing your application with an SSL certificate using Certbot.

## Prerequisites

- AWS Account
- Domain name (managed via DNS)
- Basic knowledge of Linux commands and SSH

---

## 1. Launch EC2 Instance

1. Log in to your AWS account and select your preferred region.
2. Go to **EC2 Dashboard** and click **Launch Instance**.
3. Choose **Ubuntu** as the operating system.
4. Select the appropriate instance size (e.g., `t2.micro` for free tier).
5. Create and download the SSH key pair (`.pem` file).
6. Ensure the security group allows inbound access for HTTP, HTTPS, and SSH.
7. Launch the instance.

---

## 2. Connect to EC2 via SSH

1. Allocate an Elastic IP in the EC2 console and associate it with your instance.
2. Run the following command to SSH into the machine:

    ```bash
    ssh -i "your-key.pem" ubuntu@your-ec2-ip
    ```

3. Update your packages:

    ```bash
    sudo apt-get update
    ```

4. Install Node.js and npm:

    ```bash
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install -y nodejs
    node --version
    npm --version
    ```

---

## 3. Clone and Set Up Your Node.js Application

1. Clone your repository:

    ```bash
    git clone https://github.com/your-username/your-repo.git
    ```

2. Navigate to the project directory and install dependencies:

    ```bash
    cd your-repo
    npm install
    ```

3. Install **PM2** to keep the app running in the background:

    ```bash
    sudo npm install -g pm2
    pm2 start app.js
    pm2 logs
    ```

4. Add your environment variables:

    ```bash
    export MONGO_URL="mongodb+srv://username:password@cluster0.mongodb.net/dbname"
    ```

---

## 4. Set Up Nginx as a Reverse Proxy

1. Install Nginx:

    ```bash
    sudo apt install nginx
    ```

2. Go to the root directory:

    ```bash
    cd /
    ```

3. Reload Nginx:

    ```bash
    sudo nginx -s reload
    ```

4. Open the Nginx default config file:

    ```bash
    sudo vim /etc/nginx/sites-available/default
    ```

5. Update the configuration to act as a reverse proxy:

    ```nginx
    server {
        server_name yourdomain.com www.yourdomain.com;

        location / {
            proxy_pass http://localhost:8000;  # Replace 8000 with your app’s port
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
    ```

6. Reload Nginx:

    ```bash
    sudo nginx -s reload
    ```

---

## 5. Obtain an SSL Certificate with Certbot

1. Add the Certbot repository and install Certbot for Nginx:

    ```bash
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install python3-certbot-nginx
    ```

2. Obtain and configure your SSL certificate:

    ```bash
    sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
    ```

3. Test automatic renewal:

    ```bash
    certbot renew --dry-run
    ```

---

## 6. DNS Configuration

1. Add an **A Record** in your DNS management console to point to your EC2’s Elastic IP.
2. Verify the DNS is set up correctly:

    ```bash
    nslookup yourdomain.com
    ```

    You should see a response like this:

    ```bash
    Non-authoritative answer:
    Name:    yourdomain.com
    Address: your-elastic-ip
    ```

---

## 7. Access Your Application

You can now access your Node.js application on `http://yourdomain.com`.

---

## Troubleshooting

- **Nginx errors**: Check Nginx logs at `/var/log/nginx/error.log`.
- **PM2 issues**: View PM2 logs using `pm2 logs`.
