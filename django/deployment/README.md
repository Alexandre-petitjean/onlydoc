# Django app deployement on Linux server

## Requirements

* A linux server (Debian or Ubuntu).
* A sodoer user.

> For this exemple the project is named "myproject".
>
> The user for the service is named "mylinuxuser".
>
> The project will be located in /opt/.
>
> The SGBD used is Postgresql.

## Install dependencies

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y python3-venv postgresql nginx net-tools

# Install git if you want the cookiecutter setup
sudo apt install -y git
```

## Setup the Postgresql server

Test the Postgresql install

```bash
systemctl status postgresql
# return active
netstat -ntlp
# postgres is listening on 5432
```

Postgresql cheat sheet

```bash
# Connexion
sudo -u postgres psql
# List database
\l
# List user
\du
# Exit
\q
```

## Configure the datatase

```bash
sudo -u postgres psql
CREATE DATABASE mydb;
CREATE USER myuser WITH ENCRYPTED PASSWORD 'mypass';
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
```

## Setup the django project

Here you have two options, either the classic Django `startproject cli` or the `cookiecutter framework`. I personally recommend the cookiecutter framework for all the pre configured option and module. But if it's your first time with django, go for the `Django startproject command setup`.

### Cookiecutter framework setup (Recommended)

Check the last version of cookiecutter before pulling the module.

> <https://github.com/cookiecutter/cookiecutter-django>

```bash
sudo python3 -m venv /opt/venv/
sudo /opt/venv/bin/python -m pip install "cookiecutter>=1.7.0"
sudo /opt/venv/bin/python -m cookiecutter https://github.com/cookiecutter/cookiecutter-django
```

#### Django settings

> Update your database settings following the postgresql configuration

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
    'USER': 'myuser',
    'PASSWORD': 'mypass',
    'HOST': '127.0.0.1',
    'PORT': 5432,
    }
}
```

> Only modify the MEDIA_ROOT url

```python
MEDIA_ROOT = BASE_DIR / 'uploads'
```

### Django startproject command setup

```bash
sudo python3 -m venv /opt/venv/
sudo /opt/venv/bin/python -m pip install django
sudo /opt/venv/bin/django-admin startprojet myproject
sudo /opt/venv/bin/python -m pip install psycopg2-binary
sudo /opt/venv/bin/python -m pip install gunicorn
```

#### Django settings

> Update your database settings following the postgresql configuration

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
    'USER': 'myuser',
    'PASSWORD': 'mypass',
    'HOST': '127.0.0.1',
    'PORT': 5432,
    }
}
```

> Add the settings for static and media files

```python
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'static'
MEDIA_URL = 'media'
MEDIA_ROOT = BASE_DIR / 'uploads'
```

## (Both setup) Migration and static file

```bash
sudo /opt/venv/bin/python ./myproject/manage.py migrate
sudo /opt/venv/bin/python manage.py collectstatic
sudo mkdir /opt/myproject/uploads
sudo chown -R root:mylinuxuser uploads
sudo chmod -R 775 uploads
```

### First test of the django projet

```bash
# Test the gunicorn install
sudo /opt/venv/bin/gunicorn --version
# Test the gunicorn server
cd /opt/myproject/
sudo /opt/venv/bin/gunicorn -b 127.0.0.1:8000 myproject.wsgi
```

## Configure the gunicorn service

First create our user "mylinuxuser" with no right to login and with no home.

```bash
sudo useradd --system --no-create-home --shell=/sbin/nologin mylinuxuser
```
Then create the service, an exemple is in the repo "mygunicorn.service"

```bash
sudo nano /etc/systemd/system/mygunicorn.service
```

Don't forget to start and enable the service !

```bash
systemctl start mygunicorn.service
systemctl enable mygunicorn.service
```

After that you can test if the service is working correctly

```bash
systemctl status mygunicorn.service
# return active
netstat -ntlp
# gunicorn use the port 8000

# display the django home page
curl://127.0.0.1:8000

# for debug
journalctl -u mygunicorn.service
```

## Configure the nginx server

For the SSL certificate I only provide exemple with Self signed cert.

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/myproject-selfsigned.key -out /etc/ssl/certs/myproject-selfsigned.crt
```

Test the install

```bash
systemctl status nginx
netstat -ntlp
# port 80
```

Create the virtual host conf file, an exemple is in the project "myproject.conf"
```bash
sudo nano /etc/nginx/sites-available/myproject.conf
```

Create a symbolic link between sites-available and sites-enabled
```bash
sudo ln -s /etc/nginx/sites-{available,enabled}/myproject.conf
systemctl reload nginx
```

> Done !
>
> You can test in your web browser https://yourhostname.local 

