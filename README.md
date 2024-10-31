Voici un exemple de `README.md` avec toutes les instructions pour déployer Strapi sur une instance AWS EC2 avec SSL gratuit et Nginx en tant que proxy inverse.

---

# Strapi Deployment on AWS EC2 with Free SSL and Nginx Reverse Proxy

This guide provides step-by-step instructions to deploy a Strapi application on an AWS EC2 Ubuntu 22.04 server with free SSL using Certbot and a reverse proxy setup with Nginx.

## Requirements

- **AWS Account**
- **Registered Domain** (Optional but recommended)
- **Elastic IP** (Optional but recommended)

## Steps

### 1. Launch an Amazon EC2 Ubuntu Server

1. Go to AWS EC2 dashboard and launch a new instance with **Ubuntu 22.04**.
2. Attach an **Elastic IP** to your instance (optional but makes IP management easier).

### 2. Connect to Your Instance

SSH into your server:
```sh
ssh -i <key.pem> ubuntu@<your-ip-address>
```

### 3. Update the Server and Install Required Packages

Update and upgrade the instance, then install Git, htop, and wget:
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install -y git htop wget
```

### 4. Install Node Version Manager (nvm) and Node.js

1. Install nvm:
   ```sh
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
   ```

2. Add nvm to your shell profile:
   ```sh
   export NVM_DIR="$HOME/.nvm"
   [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
   [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
   ```

3. Verify the nvm installation:
   ```sh
   nvm --version
   ```

4. Install the latest stable version of Node.js:
   ```sh
   nvm install --lts
   node --version
   npm -v
   ```

### 5. Install and Configure Strapi

1. Install Strapi globally:
   ```sh
   npm install -g create-strapi-app
   ```

2. Create a new Strapi project (e.g., `my-strapi-app`):
   ```sh
   npx create-strapi-app my-strapi-app --quickstart
   ```

3. Go to your project directory and run Strapi:
   ```sh
   cd my-strapi-app
   npm run develop
   ```

4. You should be able to access Strapi at `http://localhost:1337`.

### 6. Install pm2 to Run Strapi as a Background Service

1. Install pm2 globally:
   ```sh
   npm install -g pm2
   ```

2. Start the Strapi app with pm2:
   ```sh
   pm2 start npm --name=my-strapi-app -- run develop
   pm2 save
   ```

3. Enable pm2 to start on system boot:
   ```sh
   pm2 startup
   ```

### 7. Install and Configure Nginx as a Reverse Proxy

1. Install Nginx:
   ```sh
   sudo apt install nginx -y
   ```

2. Configure Nginx for Strapi:
   ```sh
   sudo nano /etc/nginx/sites-available/default
   ```

   Add the following configuration in the `server` block, replacing `yourdomain.com` with your actual domain:

   ```nginx
   server {
       listen 80;
       server_name yourdomain.com www.yourdomain.com;

       location / {
           proxy_pass http://localhost:1337;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. Test the Nginx configuration:
   ```sh
   sudo nginx -t
   ```

4. Restart Nginx:
   ```sh
   sudo service nginx restart
   ```

   Now, you should be able to access Strapi via your server's public IP.

### 8. (Optional) Configure Domain in GoDaddy

If you have a domain, add an **A Record** pointing to your EC2 instance's IP in your DNS settings on GoDaddy.

### 9. Set Up Free SSL Certificate with Certbot

1. Install Certbot:
   ```sh
   sudo snap install core; sudo snap refresh core
   sudo apt remove certbot
   sudo snap install --classic certbot
   sudo ln -s /snap/bin/certbot /usr/bin/certbot
   ```

2. Update the Nginx configuration for your domain:
   ```sh
   sudo nano /etc/nginx/sites-available/default
   ```

   Make sure the `server_name` is correctly set:

   ```nginx
   server_name yourdomain.com www.yourdomain.com;
   ```

3. Test and reload Nginx:
   ```sh
   sudo nginx -t
   sudo systemctl reload nginx
   ```

4. Obtain SSL certificate with Certbot:
   ```sh
   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

5. Confirm the certificate installation. You should see output like:
   ```
   Certificate is saved at: /etc/letsencrypt/live/yourdomain.com/fullchain.pem
   ```

6. Test the auto-renewal of your SSL certificate:
   ```sh
   sudo certbot renew --dry-run
   ```

### 10. Access Strapi Over HTTPS

Now, you can access your Strapi instance securely over HTTPS at `https://yourdomain.com`.

---

## Troubleshooting

- **Nginx Issues**: Check the Nginx configuration with `sudo nginx -t`.
- **Certbot SSL Renewal**: Check the status of the Certbot service using:
  ```sh
  sudo systemctl status snap.certbot.renew.service
  ```

## Enjoy Your Strapi Deployment 🎉

Your Strapi server is now live with SSL and can be accessed over a secure HTTPS connection!

---

## Support 🙏

For further help, visit the [Strapi documentation](https://strapi.io/documentation/) or the [Nginx documentation](https://nginx.org/en/docs/).
