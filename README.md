# Django app deployement on Linux server

# Requirements

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
apt install python3-venv postgresql nginx
```

## Setup the Postgresql server 

> Test the Postgresql install
```bash
systemctl status postgresql -> return active
netstat -ntlp -> postgres is listening on 5432
```

> Postgresql cheat sheet
```bash
Connexion : sudo -u postgres psql
List database : \l
List user : \du
Exit : \q
```

## Configure the datatase
```bash
sudo -u postgres psql
CREATE DATABASE mydb;
CREATE USER myuser WITH ENCRYPTED PASSWORD 'mypass';
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
```

## Setup the django project
### Install the virtual env
```bash
python3 -m venv /opt/venv/
sudo /opt/venv/bin/python -m pip install django
sudo /opt/venv/bin/django-admin startprojet myproject
sudo /opt/venv/bin/python -m pip install psycopg2-binary
sudo /opt/venv/bin/python -m pip install gunicorn
```
### Django settings

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

### Migration and static file

```
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


