# Service Configuration


## Configure Gunicorn
Create  service and socket to serve django project
```
sudo touch /etc/systemd/system/gunicorn.service
sudo touch /etc/systemd/system/gunicorn.socket
```
gunicorn.service:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/dproject
ExecStart=/home/ubuntu/dproject/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          project.wsgi:application

[Install]
WantedBy=multi-user.target
```
gunicorn.socket:
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```
Start and Enable Gunicorn:
```
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

## Create a systemd Service for Laravel
#### Create a new service file
```
sudo vim /etc/systemd/system/lproject.service
```
Add the following content to the service file
```
[Unit]
Description=Laravel Application
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/Lproject
ExecStart=/usr/bin/php /home/ubuntu/Lproject/artisan serve --host=0.0.0.0 --port=8001
Restart=always

[Install]
WantedBy=multi-user.target
```
#### Reload systemd to apply the new service
```
sudo systemctl daemon-reload
```
#### Start and enable the Laravel service
```
sudo systemctl start laravel
sudo systemctl enable laravel
```
