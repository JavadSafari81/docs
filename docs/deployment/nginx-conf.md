# Nginx Configuration
## Create a config file with domain name
```
touch /etc/nginx/sites-available/practicedevops.ir
```
### Configure Nginx for practicedevops.ir domain
Configure the domain to listen on both ports 80 and 443. All HTTP requests on port 80 are redirected to HTTPS on port 443 for secure communication.
```
server {
    listen 80;
    server_name practicedevops.ir www.practicedevops.ir;

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl;
    server_name practicedevops.ir www.practicedevops.ir;
    ssl_certificate /etc/letsencrypt/live/practicedevops.ir/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/practicedevops.ir/privkey.pem;
}
```
#### Create a Symbolic Link
```
sudo ln -s /etc/nginx/sites-available/practicedevops.ir /etc/nginx/sites-enabled/
```
### Configure Djnago
```
        location /django/ {
        proxy_pass http://unix:/run/gunicorn.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        }
```
### Configure Laravel
```
        location /laravel/ {
        proxy_pass http://127.0.0.1:8001; # Laravel application
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
```
#### Test nginx:
```
sudo nginx -t
```
#### Restart nginx:
```
sudo systemctl restart nginx
```

