# Node.js NGINX Deployment 

> Steps to deploy a Node.js app to [Database Mart](https://www.databasemart.com/) using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Sign up for VPS or Dedicated Server with Linux Ubuntu OS
In this tutorial I prefer to buy VPS from [Database Mart VPS](https://www.databasemart.com/vps-hosting).

## 2. Connect to the ordered VPS server using ssh remote
Open cmd or terminal in Windows. but I prefer to use terminal in this case and it recommended.
```bash
# I am using administrator as username for my vps. but in some vps provider using root as username.
ssh administrator@vps_ip_address
```
After the ssh connected sucessfully, now we need to run two commands below. I recommended to run these command before do anything when you buy a new VPS or Dedicated Linux Server.
```bash
# Check for the dependencies update
sudo apt update

# Upgrade linux dependencies to the latest version
sudo apt upgrade 
```

## 3. Install Node/NPM into the VPS Unbuntu server
```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.2/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 20

# Verify the Node.js version:
node -v # Should print "v20.19.0".
nvm current # Should print "v20.19.0".

# Verify npm version:
npm -v # Should print "10.8.2".
```

## 4. Clone your project from Github
There are a few ways to get your files on to the server, I would suggest using Git
```bash
# Clone your nodejs project from github repository.
git clone yourproject.git
```

### 5. Install dependencies and test app
```bash
# Navigate to the nodejs project directory.
cd yourproject

# Now install all the necessary dependencies in the package.json
npm install

# Run the app with the start command.
npm start

# Stop app
Ctrl + C
```
## 6. Setup PM2 process manager to keep your app running
```bash
# Install pm2 to manage and monitor the nodejs app.
sudo npm install -g pm2

# Navigate to the current app project directory.
cd yourproject

# Now we can start the nodejs app in the current project directory by run the command below.
pm2 start "npm start"

# Useful commands for pm2 👇

# List all processing instance
pm2 list

# Check processing instance status
pm2 status

# Stop the current processing instance with id 0
pm2 stop 0

# Delete the current processing instance with id 0
pm2 delete 0

# To make sure app starts when reboot
pm2 startup ubuntu
```
### You should now be able to access your app using your IP and port. Now we want to setup a firewall blocking that port and setup NGINX as a reverse proxy so we can access it directly using port 80 (http)

## 7. Setup ufw firewall
```bash
# Enable firewall network
sudo ufw enable

# Check for the firewall network status
sudo ufw status

# Allow the ssh port for firewall network to be accessable from remote ssh.
sudo ufw allow ssh (Port 22)

# Allow http request to be accessable from everywhere.
sudo ufw allow http (Port 80)

# Allow https request to be accessable from everywhere.
sudo ufw allow https (Port 443)
```

## 8. Install NGINX and configure
```bash
# Install nginx into the Ubuntu server.
sudo apt install nginx

# Edit the default nginx config file with the following configuration below.
sudo nano /etc/nginx/sites-available/default
```
Add the following configuration to the location part of the server block
```nginxconf
server_name domain.com www.domain.com;

location / {
    proxy_pass http://localhost:8080; #whatever port your app runs on
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
}
```
The config file should be look like this
```nginxconf
# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;
        
	index index.html index.htm index.nginx-debian.html;

        # domain.com and www.domain.com is the domain that we will buy in the step 9 below.
	server_name domain.com www.domain.com;
        
        location / {
                proxy_pass http://localhost:8080; #whatever port your app runs on
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }
}
```

```bash
# Check the NGINX config file format is correct or not
sudo nginx -t

# Restart NGINX server
sudo service nginx restart
```

### You should now be able to visit your IP with no port (port 80) and see your app.

## 9. Add custom domain name
Now goto buy a domain name from any domain provider. but in this case I prefer to use [Namecheap](https://namecheap.com) as domain provider.

Now add some dns record to the domain:
- Add A record for @ and value is the vps ip address.
- Add A record for www and value is the vps ip address same as A record above.

It may take a bit to propogate. after that you should be able to access your website through the domain name as well.

## 10. Add SSL with LetsEncrypt
```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Run Certbot to obtain and install the SSL certificate for your domain.
# Replace `domain.com` and `www.domain.com` with your actual domain name that you buy from step 9.
sudo certbot --nginx -d domain.com -d www.domain.com
```
- Certbot will interactively ask you for information and configure NGINX to use the obtained certificate.
- Certbot will ask if you want to redirect HTTP traffic to HTTPS. Choose the appropriate option based on your preference. Redirecting is recommended for better security.

You can verify that the automatic renewal process works by running the following command (Optional):
```bash
# Only valid for 90 days, Renewal the ssl with the command below.
sudo certbot renew --dry-run
```

Now visit https://domain.com and you should see your Node app.
