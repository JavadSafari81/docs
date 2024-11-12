#Django & PostgreSQL
#### Update system
```
sudo apt update
sudo apt upgrade
```
#### Install required packages
???+ Note
        Use virtual Environment, it helps to manage dependencies and avoid conflicts with system-level Python installations.
```
sudo apt install python3-pip python3-dev python3-venv nginx
pip install django Gunicorn
```
#### Create a Django Project
```
django-admin startproject dproject .
```
#### Configure Gunicorn
To serve Django as a service, you must configure Gunicorn in the [Service Configuration](service-conf) section.

### Establish Django PostgreSQL Connection
#### Install PostgreSQL
```
sudo apt update
sudo apt install postgresql postgresql-contrib
```
#### Create a PostgreSQL Database and User
```
sudo -i -u postgres
```
Create database with your database name:
```
CREATE DATABASE dproject;
```
Create a new PostgreSQL user:
```
CREATE USER user1 WITH PASSWORD 'mjrude1212';
```
Grant all privileges on the newly created database to the new user
```
GRANT ALL PRIVILEGES ON DATABASE dproject TO user1;
```
Exit the PostgreSQL prompt:
```
\q

```
Exit the postgres user shell:
```
exit
```
#### Configure Django to Use PostgreSQL
Install psycopg2:
```
pip install psycopg2
```
Open django project settings file:
```
vim ~/dproject/project/settings.py
```
Change Database config:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'dproject',
        'USER': 'user1',
        'PASSWORD': 'mjrude1212',
        'HOST': 'localhost',  # or ''
        'PORT': '5432',
    }
}
```
#### Migrate Your Database
```
python manage.py migrate
```
