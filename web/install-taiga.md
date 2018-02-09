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