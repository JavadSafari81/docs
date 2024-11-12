# Laravel & MySQL
#### Update system
```
sudo apt update
```
#### Install required packages
???+ Note
	Use virtual Environment, it helps to manage dependencies and avoid conflicts with system-level Python installations.
```
sudo apt install php php-cli php-mbstring php-xml php-bcmath php-json php-zip php-curl php-mysql unzip
```
#### Install Composer
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
```
#### Create a Laravel project using composer
```
composer create-project --prefer-dist laravel/laravel Lproject
```
Set Set Permissions:
```
sudo chmod -R 775 ~/Lproject/storage
sudo chmod -R 775 ~/Lproject/bootstrap/cache
```
#### Configure service
To serve Laravel as a systemctl service, you must configure a service in the [Service Configuration](service-conf) section.
## Establish Django PostgreSQL Connection
#### Install MySQL
```
sudo apt update
sudo apt install mysql-server
```
#### Secure MySQL Installation
```
sudo mysql_secure_installation
```
#### Log In to MySQL
```
sudo mysql -u root -p
```
#### Create a MySQL Database
```
CREATE DATABASE travellist;
```
#### Create a new MySQL user
```
CREATE USER 'travellist_user'@'localhost' IDENTIFIED BY 'MjRude_1212';
```
#### Grant privileges to the user:
```
GRANT ALL PRIVILEGES ON my_database.* TO 'travellist_user'@'localhost';
```
#### Flush the privileges to apply changes and Exit
```
FLUSH PRIVILEGES;
Exit
```
### Configure Laravel to Use MySQL
#### Open your Laravel project’s .env file
```
vim ~/Lproject/.env
```
#### Update the database configuration
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=travellist
DB_USERNAME=travellist_user
DB_PASSWORD=MjRude_1212
```
### Run Laravel Migrations
#### Navigate to your Laravel project directory
```
cd ~/Lproject
```
#### Run the migration command
```
php artisan migrate
```
