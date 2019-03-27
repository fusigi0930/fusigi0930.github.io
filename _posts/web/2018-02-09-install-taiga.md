---
layout: post
title: Install Taiga (a kanban tracking system)
description: how to install taiga
tags: tracking web
---

A web based tracking system and support Scrum (a agile software development) management tool.
the official web site is [https://taiga.io/](https://taiga.io/)

# Install on ubuntu 16.04
I did refer the [document](https://taigaio.github.io/taiga-doc/dist/setup-production.html) from taigaio in github,
and just adjust some step about it.

## Init your ubuntu
you can use the original ubuntu 16.04 or use lxc container, in my case, I use lxc container to setup my ubuntu 16.04
by using this command:
```shell
lxc-create -n taiga -t ubuntu -- -r xenial
```

start the lxc container by this command:
```shell
lxc-start -n taiga
```

we assume the environment ip address is 10.0.3.220, let starting to setup the ubuntu environment.

### initial os environment
* in taiga container, we do these commands to install the environment:
```shell
sudo apt-get update
sudo apt-get install -y build-essential binutils-doc autoconf flex bison libjpeg-dev \
libfreetype6-dev zlib1g-dev libzmq3-dev libgdbm-dev libncurses5-dev \
automake libtool libffi-dev curl git tmux gettext \
nginx \
rabbitmq-server redis-server \
circus \
postgresql-9.5 postgresql-contrib-9.5 \
postgresql-doc-9.5 postgresql-server-dev-9.5 \
python3 python3-pip python-dev python3-dev python-pip \
virtualenvwrapper \
libxml2-dev libxslt-dev \
libssl-dev libffi-dev \
nodejs
```

* create a new user "taiga"
```shell
sudo adduser taiga
sudo adduser taiga sudo
sudo passwd taiga
```
then re-login in user "taiga", and create a log folder
```shell
mkdir -p ~/logs
```

### initial Database
* create database and user
```shell
sudo -u postgres createuser taiga
sudo -u postgres createdb taiga -O taiga --encoding='utf-8' --locale=en_US.utf8 --template=template0
```

* set the user password for database
in the real case, the taiga works after I set database configuration, it needs user and password.
and it will be more convenint to manage the database.
```shell
sudo -u taiga psql taiga
```
run the sql statement in psql:
```sql
ALTER USER taiga WITH PASSWORD '<password>';
```
in my case, the password I set it to "taiga_home".

* create a user for rabitmq (message queue software)
```shell
sudo rabbitmqctl add_user taiga taiga_home
sudo rabbitmqctl add_vhost taiga
sudo rabbitmqctl set_permissions -p taiga taiga ".*" ".*" ".*"
```

## Install taiga
taiga is a opensource project management tool, the source code is store in github.

### download the source code
* backend
```shell
cd ~
git clone https://github.com/taigaio/taiga-back.git taiga-back
cd taiga-back
git checkout -b stable remotes/origin/stable
```

* frontend
```shell
cd ~
git clone https://github.com/taigaio/taiga-front-dist.git taiga-front-dist
cd taiga-front-dist
git checkout -b stable remotes/origin/stable
```

* events
```shell
git clone https://github.com/taigaio/taiga-events.git taiga-events
```

### setup taiga-backend
* create a new virtual env
```shell
cd ~/taiga-back
mkvirtualenv -p /usr/bin/python3.5 taiga
```

* install dependencies and populate database
```shell
pip install -r requirements.txt
python manage.py migrate --noinput
python manage.py loaddata initial_user
python manage.py loaddata initial_project_templates
python manage.py compilemessages
python manage.py collectstatic --noinput
```

after the commands, the taiga has a default user "admin" with password "123123".

* setting backend
create the python file in ~/taiga-back/settings/local.py with content:

```python
from .common import *

DATABASES = {
        'default': {
                'ENGINE': 'django.db.backends.postgresql',
                'NAME': 'taiga',
                'USER': 'taiga',
                'PASSWORD': 'taiga_home',
                'HOST': 'localhost',
                'PORT': '5432',
        }
}

MEDIA_URL = "http://10.0.3.220/media/"
STATIC_URL = "http://10.0.3.220/static/"
SITES["front"]["scheme"] = "http"
SITES["front"]["domain"] = "10.0.3.220"
SITES["api"]["scheme"] = "http"
SITES["api"]["domain"] = "localhost:8000"
SITES["api"]["name"] = "api"

SECRET_KEY = "taigakey"

DEBUG = False
PUBLIC_REGISTER_ENABLED = False

#DEFAULT_FROM_EMAIL = "no-reply@example.com"
#SERVER_EMAIL = DEFAULT_FROM_EMAIL

#CELERY_ENABLED = True

EVENTS_PUSH_BACKEND = "taiga.events.backends.rabbitmq.EventsPushBackend"
EVENTS_PUSH_BACKEND_OPTIONS = {"url": "amqp://taiga:taiga_home@localhost:5672/taiga"}

# Uncomment and populate with proper connection parameters
# for enable email sending. EMAIL_HOST_USER should end by @domain.tld
#EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
#EMAIL_USE_TLS = False
#EMAIL_HOST = "localhost"
#EMAIL_HOST_USER = ""
#EMAIL_HOST_PASSWORD = ""
#EMAIL_PORT = 25

# Uncomment and populate with proper connection parameters
# for enable github login/singin.
#GITHUB_API_CLIENT_ID = "yourgithubclientid"
#GITHUB_API_CLIENT_SECRET = "yourgithubclientsecret"
```

### setup taiga-frontend
* config frontend
copy the config file from example file
```shell
cd ~/taiga-front-dist
cp dist/conf.example.json dist/conf.json
```
then modify the content to:
```json
{
        "api": "http://10.0.3.220/api/v1/",
        "eventsUrl": "ws://10.0.3.220/events",
        "debug": "true",
        "publicRegisterEnabled": true,
        "feedbackEnabled": true,
        "privacyPolicyUrl": null,
        "termsOfServiceUrl": null,
        "maxUploadFileSize": null,
        "contribPlugins": []
}
```

### setup taiga-events
* install nodejs 6.x version
```shell
cd ~/taiga-events
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

* install javascript dependencies
```shell
npm install
sudo npm install -g coffee-script
```

* config events
create the config file from example file
```shell
cp config.example.json config.json
```
then adjust the content as:
```json
{
    "url": "amqp://taiga:taiga_home@localhost:5672",
    "secret": "taigakey",
    "webSocketServer": {
        "port": 8888
    }
}
```
notice, the secret must be the same with backend settings in local.py

### setting taiga relate in circus 
* set taiga events 
create the file /etc/circus/conf.d/taiga-events.ini with content:
```ini
[watcher:taiga-events]
working_dir = /home/taiga/taiga-events
cmd = /usr/bin/coffee
args = index.coffee
uid = taiga
numprocesses = 1
autostart = true
send_hup = true
stdout_stream.class = FileStream
stdout_stream.filename = /home/taiga/logs/taigaevents.stdout.log
stdout_stream.max_bytes = 10485760
stdout_stream.backup_count = 12
stderr_stream.class = FileStream
stderr_stream.filename = /home/taiga/logs/taigaevents.stderr.log
stderr_stream.max_bytes = 10485760
stderr_stream.backup_count = 12
```

* set taiga
create the file /etc/circus/conf.d/taiga.ini with content:
```ini
[watcher:taiga]
working_dir = /home/taiga/taiga-back
cmd = gunicorn
args = -w 3 -t 60 --pythonpath=. -b 127.0.0.1:8001 taiga.wsgi
uid = taiga
numprocesses = 1
autostart = true
send_hup = true
stdout_stream.class = FileStream
stdout_stream.filename = /home/taiga/logs/gunicorn.stdout.log
stdout_stream.max_bytes = 10485760
stdout_stream.backup_count = 4
stderr_stream.class = FileStream
stderr_stream.filename = /home/taiga/logs/gunicorn.stderr.log
stderr_stream.max_bytes = 10485760
stderr_stream.backup_count = 4

[env:taiga]
PATH = /home/taiga/.virtualenvs/taiga/bin:$PATH
TERM=rxvt-256color
SHELL=/bin/bash
USER=taiga
LANG=en_US.UTF-8
HOME=/home/taiga
PYTHONPATH=/home/taiga/.virtualenvs/taiga/lib/python3.5/site-packages
```

* restart the circus service
```shell
sudo service circusd restart
```
then we can check the taiga/taiga-events status by command:
```shell
circusctl status
```

### settgins for web server 
taiga use nginx web server, we need set the nginx as following...

* disable the default web site
```shell
sudo rm /etc/nginx/sites-enabled/default
```

* set the taiga web configuration
store the content in the file /etc/nginx/conf.d/taiga.conf
```apacheconf
server {
    listen 80 default_server;
    server_name _;

    large_client_header_buffers 4 32k;
    client_max_body_size 50M;
    charset utf-8;

    access_log /home/taiga/logs/nginx.access.log;
    error_log /home/taiga/logs/nginx.error.log;

    # Frontend
    location / {
        root /home/taiga/taiga-front-dist/dist/;
        try_files $uri $uri/ /index.html;
    }

    # Backend
    location /api {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:8001/api;
        proxy_redirect off;
    }

    # Django admin access (/admin/)
    location /admin {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:8001$request_uri;
        proxy_redirect off;
    }

    # Static files
    location /static {
        alias /home/taiga/taiga-back/static;
    }

    # Media files
    location /media {
        alias /home/taiga/taiga-back/media;
    }

        # Taiga-events
        location /events {
        proxy_pass http://localhost:8888/events;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
        }
}
```

* re configure nginx
```shell
sudo nginx -t
sudo service nginx restart
```

now, we should use taiga in http://10.0.3.220
