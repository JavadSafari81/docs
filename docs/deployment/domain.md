# Domain



To configure your domain for hosting, follow these steps:

## Purchase a Domain

Buy a domain from a domain registration service like [Nic.ir](https://www.nic.ir).

#### Configure DNS Using a DNS Provider

You can use DNS services like [ArvanCloud](https://www.arvancloud.ir) or [Cloudflare](https://www.cloudflare.com) to manage your DNS records.

#### Steps to Configure DNS via ArvanCloud:

1. **Sign up and log in** to your [ArvanCloud account](https://www.arvancloud.ir).
2. Go to the **CDN tab**.
3. Enter your **domain name** in the appropriate section.
4. **Create an A record** pointing to your web server’s IP address.
```
A record | @ | your-server-ip
```
## Obtain SSL 
#### 1. Install snapd
```
sudo apt update
sudo apt install snapd
```
#### 2. Install Certbot
```
sudo snap install --classic certbot
```
#### 3. Prepare the Certbot command
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
#### 4. Run Certbot
```
sudo certbot --nginx -d practicedevops.ir -d www.practicedevops.ir

```
