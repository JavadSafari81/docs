#Web Server


#### Installing Nginx
```
sudo apt update
sudo apt install nginx
```
#### Adjusting the Firewall
Before testing Nginx, the firewall software needs to be adjusted to allow access to the service. Nginx registers itself as a service with ufw upon installation, making it straightforward to allow Nginx access.

List the application configurations that ufw knows how to work with by typing:
```
sudo ufw app list
```
You should get a listing of the application profiles:

```
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
It is recommended that you enable the most restrictive profile that will still allow the traffic you’ve configured. Right now, we will only need to allow traffic on port 80.

You can enable this by typing: 
```
sudo ufw allow 'Nginx HTTP'
```
You can verify the change by typing:
```
sudo ufw status
```
#### Checking your Web Server
```
systemctl status nginx
``` 
You must see this:
```
Active: active (running) since Fri 2020-04-20 16:08:19 UTC;
```
You can now see your nginx default page in:
```
http://your_server_ip
http://188.121.111.85
```
