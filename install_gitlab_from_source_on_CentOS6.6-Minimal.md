# GitLab Install Guide

```
Distribution      : CentOS6.5/6 Minimal
GitLab version    : 7.13.2
Web Server        : Nginx 1.9.3
Database          : MariaDB 10.1.5
Contributors      : @lvfeng1130
Additional Notes  : In order to install gitlab under a circumstance that Linux server has no internet access, All reiled code or pacakges are downloaded via chrome, A great thanks to the community for providing such an amazing product, also a lot thanks to these who helped on the guides.
```

## Overview

The installation manual base itself on [gitlab-recipes Install Guild](https://github.com/gitlabhq/gitlab-recipes/tree/master/install/centos),
It attempts to install Gitlab from ground up with http access in chrome and only yum local repository.
Some of the install procedures are from [official install guide](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/installation.md)

## Important Notes

The following steps have been known to work and should be followed from up to bottom.
If you deviate from this guide, do it with caution and make sure you don't violate
any assumptions GitLab makes about its environment. I personally tried this on REHL6.3 but got stuck at the installation of gem charlock_holmes.
(You may follow the manual here [gitlab-recipes Install Guild](https://github.com/gitlabhq/gitlab-recipes/tree/master/install/centos), it claim to be tested on RHEL6.3)

## Add disk space
### Extend the size of lvm partition /opt
1. Create the physical volume
```
pvcreate /dev/sdb
```
2. Add pv /dev/sdc into  the volume group 'systemvg' which includes partition /opt
```
vgextend systemvg /dev/sdb
```
3. extend logical volume systemvg-optlv
```
lvextend -L 100G /dev/mapper/systemvg-optlv
```
4. resize to file system (to see in df -h)
```
resize2fs /dev/mapper/systemvg-optlv
```

## Install the operating system (CentOS6.6 Minimal)
>I start with a completely clean CentOS6.6 "minimal" installation which can be accomplished by downloading from different mirrors.
e.g.

- [Aliyun](http://mirrors.aliyun.com/)
- [NetEase](http://mirrors.163.com/)
- [Sohu](http://mirrors.sohu.com/)

## Install backbone software and dependencies

###  Development tools
```
yum groupinstall "Development tools"
```

### Requied headers files for git
```
yum -y install gcc gcc-c++ zlib-devel perl-ExtUtils-MakeMaker autoconf kernel-devel tcl tcl-devel
```

### Complile git-2.4.5.tar.xz from source
```
tar xvf git-2.4.5.tar.xz
cd git-2.4.5
make configure
./configure --prefix=/usr
make -j 8 all   # fast as shit..
sudo make install
```

### Compile nodejs for gitlab environment which will be used later
```shell
tar xvf node-v0.12.7.tar.gz
cd  node-v0.12.7
./configure
make -j 4
sudo make install
```


#### Install these modules for python
```shell

# install basic headers
yum install sqlite-devel openssl-devel readline-devel gdbm-devel bzip2-devel ncurses-devel

# install extra headers in case we will meet the warning when compile Python, It can be ignored though
 yum install 1:tk-8.5.7-5.el6.x86_64 1:tix-8.4.3-5.el6.x86_64 tkinter-2.6.6-64.el6.x86_64 xorg-x11-proto-devel-7.7-9.el6.noarch freetype-devel-2.3.11-15.el6_6.1.x86_64 fontconfig-devel-2.8.0-5.el6.x86_64 libXau-devel-1.0.6-4.el6.x86_64 libxcb-devel-1.9.1-3.el6.x86_64 libX11-devel-1.6.0-6.el6.x86_64 libXrender-devel-0.9.8-2.1.el6.x86_64 libXft-devel-2.3.1-2.el6.x86_64 1:tk-devel-8.5.7-5.el6.x86_64 1:tix-devel-8.4.3-5.el6.x86_64 bzip2-devel-1.0.5-7.el6_0.x86_64
```

### Python related
> It's recommaned to update python from 2.6 to 2.7 for some custom settings

```
# compile from source
./configure
make
sudo make install

# modify /usr/bin/yum
sed -i -e '1 s/python/python2.6/g' /usr/bin/yum
```


### Compile cmake to install mariadb later
```shell
cd cmake-3.2.3
./bootstrap && make -j 4
sudo make install
```


*It's recommaned that we'are gonna need EPEL and PUIAS Computational repositories to properly install environment for GitLab  
so I put their online repositorie http addresses here, go manually fetch any packages if there is requirement.  :pray: mind fuck!*

* Aliyun: http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/
* EPEL: http://dl.fedoraproject.org/pub/epel/6/x86_64/
* PUIAS: http://springdale.math.ias.edu/data/puias/computational/6.6/x86_64/

> Dependencies which are necessary for GitLab, you may not be able to install all of them  from your local repository.

```shell
  yum -y install  readline readline-devel ncurses-devel gdbm-devel glibc-devel tcl-devel openssl-devel curl-devel expat-devel db4-devel byacc sqlite-devel gcc-c++ libyaml libyaml-devel libffi libffi-devel libxml2 libxml2-devel libxslt libxslt-devel libicu libicu-devel system-config-firewall-tui python-devel redis sudo wget crontabs logwatch logrotate perl-Time-HiRes git vim-enhanced
```

> Exclude these which have existed in system by default, get a list of which we're missing.

```shell
[root@lifestealer ~]# for i in `echo "vim-enhanced readline readline-devel ncurses-devel gdbm-devel glibc-devel tcl-devel openssl-devel curl-devel expat-devel db4-devel byacc sqlite-devel gcc-c++ libyaml libyaml-devel libffi libffi-devel libxml2 libxml2-devel libxslt libxslt-devel libicu libicu-devel system-config-firewall-tui python-devel redis sudo wget crontabs logwatch logrotate perl-Time-HiRes git"` ;do  rpm -q $i  1>/dev/null || echo $i  ;done |xargs
```

> Some of above packages can obviously be installed from local repository (The DVD Distro, learn how to yum install from iso  CentOS-6.6-x86_64-bin-DVD1.iso)
e.g.

```shell
yum install openssl-devel readline-devel tcl-devel curl-devel expat-devel sqlite-devel libyaml libyaml-devel python-devel logwatch perl-Time-HiRes newt-python
# not necessary need system-config-firewall-tui
```
> while others may need to download from officical mirrors, we will put redis aside cause we're going to compile it seperately.

```shell
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libffi-3.0.5-3.2.el6.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libffi-devel-3.0.5-3.2.el6.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxml2-2.7.6-14.el6_5.2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxml2-devel-2.7.6-14.el6_5.2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxslt-1.1.26-2.el6_3.1.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxslt-devel-1.1.26-2.el6_3.1.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libicu-4.2.1-9.1.el6_2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libicu-devel-4.2.1-9.1.el6_2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/system-config-firewall-tui-1.2.27-7.1.el6.noarch.rpm
```

### Compile redis
```
tar xvf redis-3.0.3.tar.gz
cd redis-3.0.3
make MALLOC=jemalloc
make test
```


### Configure sendmail
yum install sendmail-cf



### Compile and Install Mariad
#### Install libevent and lzo either from ISO image or official repo
>  they're both builtin if you didn't choose a minimal installation.

```
yum install libevent-devel libevent lzo lzop
```
#### Download lz4 rpm pacakges from EPEL repo
```
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/lz4-static-r131-1.el6.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/lz4-r131-1.el6.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/lz4-devel-r131-1.el6.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/msgpack-0.5.7-5.el6.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/msgpack-devel-0.5.7-5.el6.x86_64.rpm
wget ftp://bo.mirror.garr.it/pub/1/slc/slc65/x86_64/Packages/cracklib-devel-2.8.16-4.el6.x86_64.rpm
http://ftp.rediris.es/volumes/sites/linuxsoft.cern.ch/slc/slc62/x86_64/Packages/cracklib-devel-2.8.16-4.el6.x86_64.rpm
wget ftp://bo.mirror.garr.it/pub/1/slc/slc65/x86_64/Packages/libevent-devel-1.4.13-4.el6.x86_64.rpm
wget ftp://bo.mirror.garr.it/pub/1/centos/6.6/os/x86_64/Packages/libevent-headers-1.4.13-4.el6.noarch.rpm
wget http://bo.mirror.garr.it/pub/1/centos/6.6/os/x86_64/Packages/libevent-doc-1.4.13-4.el6.noarch.rpm

sudo rpm -ivh lz4-r131-1.el6.x86_64.rpm
sudo rpm -ivh lz4-static-r131-1.el6.x86_64.rpm
sudo rpm -ivh lz4-devel-r131-1.el6.x86_64.rpm
sudo rpm -ivh msgpack-0.5.7-5.el6.x86_64.rpm
sudo rpm -ivh msgpack-devel-0.5.7-5.el6.x86_64.rpm
sudo rpm -ivh cracklib-devel-2.8.16-4.el6.x86_64.rpm
sudo rpm -ivh libevent-headers-1.4.13-4.el6.noarch.rpm libevent-devel-1.4.13-4.el6.x86_64.rpm libevent-doc-1.4.13-4.el6.noarch.rpm
sudo rpm -ivh lzo-minilzo-2.03-3.1.el6.x86_64.rpm lzo-devel-2.03-3.1.el6.x86_64.rpm
```


#### Install package kytea and libzmq from source
```
wget http://www.phontron.com/kytea/download/kytea-0.4.7.tar.gz
wget http://download.zeromq.org/zeromq-4.1.2.tar.gz
```

#### Complile kytea
```shell
tar -xzf kytea-0.4.7.tar.gz
cd kytea-0.4.7
./configure
make -j 4
make check
make install
```

#### Compile zeromq (download libsodium for dependencies)
```
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/libsodium-0.4.5-3.el6.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/libsodium-devel-0.4.5-3.el6.x86_64.rpm
```

```
tar xvf zeromq-4.1.2.tar.gz
cd zeromq-4.1.2
./configure && make -j8 && sudo make install
```

#### compile openpam.h
```
./configure --with-pam-unix --with-su
make
sudo make install
```

#### Enable tcmalloc for mariadb
```
##
#wget http://down1.chinaunix.net/distfiles/libunwind-1.0.tar.gz
wget http://cznic.dl.sourceforge.net/project/openpam/openpam/Ourouparia/openpam-20140912.tar.gz
wget http://nchc.dl.sourceforge.net/project/cracklib/cracklib/2.9.2/cracklib-2.9.2.tar.gz
wget https://codeload.github.com/msgpack/msgpack-c/zip/master
wget http://snowball.tartarus.org/dist/libstemmer_c.tgz
```

#### Installation of CrackLib
```
tar xvf cracklib-2.9.2.tar.gz
cd cracklib-2.9.2
sed -i '/skipping/d' util/packer.c &&

./configure --prefix=/usr    \
            --disable-static \
            --with-default-dict=/lib/cracklib/pw_dict &&
make
make install                      &&
mv -v /usr/lib/libcrack.so.* /lib &&
ln -sfv ../../lib/$(readlink /usr/lib/libcrack.so) /usr/lib/libcrack.so
```

#### Install msgpack-c
```
unzip msgpack-c-master.zip
cd msgpack-c-master
./bootstrap
./configure
make
sudo make install
```

#### Install openpam
```shell
tar xvf openpam-20140912.tar.gz
cd openpam-20140912
./configure --with-pam-unix --with-su
make -j8
sudo make install
```

>I was planning to install the lastest version but, but failed to compile
>well, gperftools-2.0.tar.gz was pickedup to work with libunwind-0.99

```
git clone https://github.com/mobing/gperftools.git
# gperftools need  Autoconf version 2.68 or higher is required
rpm -e --nodeps autoconf-2.63
#install autoconf
tar xvf autoconf-2.69.tar.xz
cd  autoconf-2.69
./configure
make
make install
```

> there is an error when make install libunwind because of the compability of autoconf tools, run autoreconf -i -f

#### Install libunwind
```
tar xvf libunwind-0.99.tar.gz
cd libunwind-0.99
CFLAGS=-fPIC ./configure
make CFLAGS=-fPIC
sudo make CFLAGS=-fPIC install
```

#### Complie gperftools
```
tar xvf gperftools-2.0.tar.gz
cd gperftools-2.0
./configure
make -j 2
sudo make install
```

#### load gperftool
```TODO
# run with root
cat >> /etc/ld.so.conf.d/maradb-10.1.5.conf<<EOF
/usr/local/lib/
EOF

# this is necessary when compile mariadb
ldconfig
```


#### Install libstemmer-c
```
unzip libstemmer_c-master.zip
cd libstemmer_c-master
```

#### Install boost support for mariadb-10.1
```
yum -y install boost boost-devel bison libtool
```

#### Create Soft link for libkytea.* and msgpack
```
sudo ln -s /usr/local/lib/libkytea.* /usr/lib64
sudo ln -s /usr/local/lib/libmsgpack* /usr/lib64
```

#### Install MariaDB
```shell
# Define the mysql source tarball name here
export MYSQL_TAR_GZ=""

groupadd mysql
useradd -g mysql mysql

tar xvf ${MYSQL_TAR_GZ}
cd ${MYSQL_TAR_GZ%.tar*gz}

MYSQL_SOCK_LOCATION="/var/run/mysql/run/mysql.socket"
MYSQL_INSTALL_DIR="/opt/mysql"
MYSQLDATADIR="/data/mysql"

# these options are based on personal preferences
### removed this options in Red Hat Enterprise Linux Server
# -DCMAKE_EXE_LINKER_FLAGS='-ltcmalloc' \
# release 6.3 (Santiago) because of the unconqurable compiler error
cmake . \
-DCMAKE_INSTALL_PREFIX=${MYSQL_INSTALL_DIR} \
-DMYSQL_DATADIR=${MYSQLDATADIR} \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DCMAKE_EXE_LINKER_FLAGS='-ltcmalloc' \
-DWITH_SSL=system \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DMYSQL_TCP_PORT=3306 \
-DMYSQL_UNIX_ADDR=${MYSQL_SOCK_LOCATION} \
-DWITH_READLINE=ON \
-DMYSQL_USER=mysql \
-DWITH_EXTRA_CHARSETS=all \
-DWITH_PARTITION_STORAGE_ENGINE=ON \
-DWITH_SPHINX_STORAGE_ENGINE=ON \
-DWITH_BLACKHOLE_STORAGE_ENGINE=ON

make -j4
sudo make install

# create mysql data directory
if [ ! -d ${MYSQLDATADIR} ];then
    sudo  mkdir -p ${MYSQLDATADIR}
fi

# assgin the correct permission
sudo chmod +w ${MYSQL_INSTALL_DIR}
sudo chown -R mysql:mysql ${MYSQLDATADIR}
sudo chmod go-rwx ${MYSQLDATADIR} -R
# enable socket creation permission for user mysql
sudo chown mysql:mysql /var/run/mysql/ -R

# prepare the bootstrap config file
sudo cp support-files/my-large.cnf /etc/my.cnf
sudo cp support-files/mysql.server /etc/init.d/mysqld

# modify initd script to make it start properly
sudo sed -i "s#^basedir=.*#&${MYSQL_INSTALL_DIR}#" /etc/init.d/mysqld
sudo sed -i "s#^datadir=.*#&${MYSQLDATADIR}#" /etc/init.d/mysqld

sudo chmod 755 /etc/init.d/mysqld

# add auto start
sudo chkconfig --add mysqld
sudo chkconfig --level 345  mysqld on

# link the library and headers to system path
sudo ln -s ${MYSQL_INSTALL_DIR}/lib/ /usr/lib/mysql
sudo ln -s ${MYSQL_INSTALL_DIR}/include/mysql/ /usr/include/mysql

# link the exec file
sudo ln -s ${MYSQL_INSTALL_DIR}/bin/mysql /usr/bin/mysql

# add the ld config path, run with root
MYSQL_INSTALL_DIR="/opt/mysql"
cat >> /etc/ld.so.conf.d/maradb-10.1.8.conf<<EOF
${MYSQL_INSTALL_DIR}/lib/mysql
${MYSQL_INSTALL_DIR}/lib
EOF

# make mysql lib affect
sudo ldconfig

# Install builtin tables
MYSQL_INSTALL_DIR="/opt/mysql"
MYSQLDATADIR="/data/mysql"
${MYSQL_INSTALL_DIR}/scripts/mysql_install_db --basedir=${MYSQL_INSTALL_DIR} \
--datadir=${MYSQLDATADIR} \
--user=mysql

# restart the services
service mysqld start

# change default access rules
${MYSQL_INSTALL_DIR}/bin/mysql_secure_installation

# this is an approach for the rails mysql driver which search mysql headers on
# several specific places, here we pick up on /usr/local/mysql, it's not necessary
# if you install mariadb under /usr /usr/local /opt/local /var/, etc.
sudo ln -s /opt/mysql/ /usr/local/mysql
```


#### Create instance for gitlab
```MySQL
# Login to MySQL
mysql -u root -p
# Type the database root password
# Create a user for GitLab. (change supersecret to a real password)
CREATE USER 'gitlab'@'localhost' IDENTIFIED BY 'supersecret';

# Create the GitLab production database
CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

# Grant the GitLab user necessary permissopns on the table.
GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';

#update
flush privileges;

# Quit the database session
\q
```

#### Install redis
```shell
[root@itsasitapp203 ~]# groupadd redis
[root@itsasitapp203 ~]# useradd -m -g redis redis
[root@itsasitapp203 ~]# groupadd git
[root@itsasitapp203 ~]# useradd -m -g git git
# rails will aceess redis as user git
[root@itsasitapp203 ~]# usermod -Ggit,redis -a git
[root@itsasitapp203 ~]# groupadd mysql
[root@itsasitapp203 ~]# useradd -m -g mysql mysql
# rails will aceess mysql as user git
[root@itsasitapp203 ~]# usermod -Gmysql,git git
[root@itsasitapp203 ~]# groupadd nginx
[root@itsasitapp203 ~]# useradd -s /sbin/nologin -g nginx nginx


# Create the directory which contains the socket, run with root
mkdir /var/run/redis
chown redis:redis /var/run/redis
chmod 755 /var/run/redis


# compile redis with jemalloc
tar xvf redis-3.0.3.tar.gz
cd redis-3.0.3
make MALLOC=jemalloc
make test

# well, make sure your redis works fine
# Here are the output of test
# Execution time of different units:
  1 seconds - unit/printver
  0 seconds - unit/quit
  1 seconds - unit/scan
  1 seconds - unit/auth
  2 seconds - unit/multi
  3 seconds - unit/protocol
  11 seconds - unit/expire
  14 seconds - unit/type/hash
  15 seconds - unit/other
  15 seconds - unit/type/list-2
  16 seconds - unit/type/list
  2 seconds - integration/rdb
  2 seconds - integration/convert-zipmap-hash-on-load
  1 seconds - integration/logging
  0 seconds - unit/pubsub
  1 seconds - unit/slowlog
  4 seconds - integration/aof
  0 seconds - unit/introspection
  2 seconds - unit/limits
  6 seconds - unit/scripting
  24 seconds - unit/dump
  24 seconds - unit/type/set
  29 seconds - unit/type/zset
  11 seconds - unit/bitops
  31 seconds - integration/replication-2
  16 seconds - unit/maxmemory
  33 seconds - unit/sort
  37 seconds - unit/basic
  37 seconds - unit/aofrw
  38 seconds - integration/replication
  18 seconds - unit/memefficiency
  41 seconds - unit/type/list-3
  21 seconds - unit/hyperloglog
  49 seconds - integration/replication-3
  48 seconds - integration/replication-4
  42 seconds - integration/replication-psync
  35 seconds - unit/obuf-limits

\o/ All tests passed without errors!

# create redis home direcotry
sudo mkdir /opt/redis
sudo mv  redis-3.0.3/ /opt/redis/redis-3.0.3/

sudo chown redis:redis /opt/redis -R

# ass wipe some simple configurations
sudo mkdir /etc/redis
egrep -Ev "^$|^#" /opt/redis/redis-3.0.3/redis.conf > /etc/redis/redis.conf

# change daemonize to yes
sed -i 's/^daemonize .*/daemonize yes/g' /etc/redis/redis.conf

# Disable Redis listening on TCP by setting 'port' to 0
sed -i 's/^port .*/port 0/' /etc/redis/redis.conf

# Enable Redis socket for default Debian / Ubuntu path
echo 'unixsocket /var/run/redis/redis.sock'  |tee -a /etc/redis/redis.conf

# change pid path to the same directory as socket, why am I doing such a silly thing? :(
sed -i 's#^pidfile .*#pidfile /var/run/redis/redis.pid#' /etc/redis/redis.conf

# Grant permission to the socket to all members of the redis group, here
#be aware that the gitlab must be part of the group later
echo 'unixsocketperm 770'|tee -a /etc/redis/redis.conf
```

#### Here are some optional tips which will make your redis more friendly
```shell
su - redis

# some tips make the environent looks more friendly
# put below contents insdie .bash_profile

# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific aliases and functions

REDIS_HOME="/opt/redis/redis-3.0.3"
REDIS_CLI="${REDIS_HOME}/src/redis-cli"
REDISSOCKET="/var/run/redis/redis.sock"

# a quick approach alias for redis-cli
alias redis="${REDIS_CLI} -s ${REDISSOCKET}"

cd /opt/redis
echo "Current Direcotry: $(pwd)"

PS1="[\u@\h \W]\$ "
export PS1

```

#### yet another /etc/init.d/redis
> rewrite it since the offical generate script turn out to be a piece of shit

```
# put script redis.sh under /etc/init.d so it will bootstrap with init

#!/bin/sh
#set -xv
# written by Feng LYU
# Email: aragon1130@gmail.com

# Source function library.
. /etc/rc.d/init.d/functions

REDIS_HOME="/opt/redis/redis-3.0.3"
REDIS_SERVER="${REDIS_HOME}/src/redis-server"
REDIS_CLI="${REDIS_HOME}/src/redis-cli"
CONF="/etc/redis/redis.conf"
REDISSOCKET="/var/run/redis/redis.sock"
REDIS_USER="redis"

SUDO_REDIS="sudo su - ${REDIS_USER} -c"

# fix me if you have a more pretty way

#PID=$(/usr/bin/pgrep -u redis redis-server 2>/dev/null)
PID=$(/usr/bin/pgrep -u ${REDIS_USER} redis-server)

case "$1" in
    start)
        if [ "x" != "x${PID}" ]
        then
            action "Redis process is already running" /bin/false
        else
            ${SUDO_REDIS} "${REDIS_SERVER} ${CONF}"

            if test $(/usr/bin/pgrep -u ${REDIS_USER} redis-server) ;then
                action "Redis process started" /bin/true
            fi
        fi
        ;;
    stop)
        if [ "x" = "x${PID}" ]
        then
            action "Redis process is not running" /bin/false
        else
            ${SUDO_REDIS} "${REDIS_CLI} -s ${REDISSOCKET} shutdown" &
            while [ -x /proc/${PID} ]
            do
                echo "[$(date +%H:%M:%S)] Waiting for Redis to shutdown ..."
                sleep 1
            done
            action "Redis stopped" /bin/true
        fi
        ;;
    status)
        if [ "x" = "x${PID}" ]
        then
            action 'Redis process is not running' /bin/false
        else
            action "Redis is running ${PID}" /bin/true
        fi
        ;;
    info)
        if [ "x" = "x${PID}" ]
        then
            action "Redis process is not running" /bin/false
        else
            ${SUDO_REDIS} "${REDIS_CLI} -s ${REDISSOCKET} info"
        fi
        ;;
    restart)
        $0 stop
        $0 start
        ;;
    *)
        echo "Please use start, stop, restart or status as first argument"
        ;;
esac

# assign execute permission
chmod u+x /etc/init.d/redis
#start redis
service redis start
# connect to redis
[root@itsasitapp201 init.d]# su - redis
Current Direcotry: /opt/redis
[redis@itsasitapp201 redis]$ redis
redis /var/run/redis/redis.sock> info
# Server
redis_version:3.0.3
redis_git_sha1:00000000
```

#### Create System user git
groupadd git
useadd -m -g git git

#### Config Sendmail
```
#Forwarding all emails
# although I don't know what below thing is doing.
# Now we want all logging of the system to be forwarded to a central email address:
su -
echo adminlogs@example.com > /root/.forward
chown root /root/.forward
chmod 600 /root/.forward
restorecon /root/.forward

echo adminlogs@example.com > /home/git/.forward
chown git /home/git/.forward
chmod 600 /home/git/.forward
restorecon /home/git/.forward
```

#### Instill ruby
```
# Based on the offcial installation that ruby 2.1.6 is the supported version
tar xvf ruby-2.1.6.tar.gz
cd ruby-2.1.6
./configure && make -j4 && sudo make install
```

#### Configure GitLab, Gitlab-shell, etc
```
This is the directory structure you will end up with following the instructions in the Installation Guide.
 |-- home
 |   |-- git
 |       |-- .ssh
 |       |-- gitlab
 |       |-- gitlab-satellites
 |       |-- gitlab-shell
 |       |-- repositories

/home/git/.ssh - contains openssh settings. Specifically the authorized_keys file managed by gitlab-shell.
/home/git/gitlab - GitLab core software.
/home/git/gitlab-satellites - checked out repositories for merge requests and file editing from web UI. This can be treated as a temporary files directory.
/home/git/gitlab-shell - Core add-on component of GitLab. Maintains SSH cloning and other functionality.
/home/git/repositories - bare repositories for all projects organized by namespace. This is where the git repositories which are pushed/pulled are maintained for all projects. This area is critical data for projects. Keep a backup
Note: the default locations for gitlab-satellites and repositories can be configured in config/gitlab.yml of GitLab and config.yml of gitlab-shell.
```

#### Configuration details
```
# Gitlab-shell
ln -s gitlabhq-7-13-stable gitlab
ln -s gitlab-shell-master gitlab-shell

cd gitlab-shell
cp -v config.yml.example config.yml

# gitlab_url, check the introduciton for advise
sed -i 's#^gitlab_url: .*#gitlab_url: "http://localhost:8080/"#g' config.yml

# redis path (orignal: /usr/bin/redis-cli)
sed -i 's#bin: .*#bin: /opt/redis/redis-3.0.3/src/redis-cli#g' config.yml

# repos_path
sed -i 's#^repos_path: .*#repos_path: "/data/git/repositories"#g' config.yml

# log file path
mkdir /opt/git/gitlab-shell/log/
chown git:git /opt/git/gitlab-shell/log/ -R
chmod -R u+rwX,go-w /opt/git/gitlab-shell/log/

sed -i 's@^# \(log_file: \)"/home/git/gitlab-shell/gitlab-shell.log"@\1"/opt/git/gitlab-shell/log/gitlab-shell.log"@g' config.yml

# create with root
mkdir /data/git
chown git:git /data/git/ -R
chmod g-x,o-x /data/git/ -R

# Do setup
./bin/install

# exit
cd ..

# GitLab
cd gitlab

cp -v config/gitlab.yml.example config/gitlab.yml

sudo chown -R git log/
sudo chown -R git tmp/
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/
sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/
sudo chmod -R u+rwX  public/uploads

# the sed will speak for itself as regard of settings
sed -i \
-e 's#\(.*host:\) localhost#\1 gitlab.example.com#g' \
-e 's#\(.*port:\) 80#\1 443#g' \
-e 's#\(.*https:\) false#\1 true#g' \
-e 's#\(.*email_from:\) example@example.com#\1 lvfeng@example.com#g' \
-e 's#\(.*repos_path:\) /home/git/repositories/#\1 /data/git/repositories/#g'
-e 's#/home/git#/opt/git#g' \
config/gitlab.yml

# set smtp email service,
cp -v config/initializers/smtp_settings.rb.sample config/initializers/smtp_settings.rb
sed -i \
-e 's#\(address:\) "email.server.com"#\1 "mail.example.com"#g' \
-e 's#\(port:\) 456#\1 25#g' \
-e 's#\(user_name:\) "smtp"#\1 "username"#g' \
-e 's#\(domain:\) "gitlab.company.com"#\1 "example.com"#g' \
-e 's#\(password:\) "123456"#\1 "secret"#g' \
config/initializers/smtp_settings.rb


mkdir /opt/git/gitlab-satellites
sudo chmod u+rwx,g=rx,o-rwx /opt/git/gitlab-satellites


# Enable cluster mode if you expect to have a high load instance
# Ex. change amount of workers to 3 for 2GB RAM server
# Set the number of workers to at least the number of cores
cp -v config/unicorn.rb.example config/unicorn.rb

# configure unicorn.rb
sed -i \
-e "s#\(worker_processes\) 3#\1 $(nproc)#g" \
-e 's#\(working_directory\) "/home/git/gitlab"#\1 "/opt/git/gitlab"#g' \
-e 's#\(listen\) "/home/git/gitlab/tmp/sockets/gitlab.socket"#\1 "/opt/git/gitlab/tmp/sockets/gitlab.socket"#g' \
-e 's#\(pid\) "/home/git/gitlab/tmp/pids/unicorn.pid"#\1 "/opt/git/gitlab/tmp/pids/unicorn.pid"#g' \
-e 's#\(stderr_path\) "/home/git/gitlab/log/unicorn.stderr.log"#\1 "/opt/git/gitlab/log/unicorn.stderr.log"#g' \
-e 's#\(stdout_path\) "/home/git/gitlab/log/unicorn.stdout.log"#\1 "/opt/git/gitlab/log/unicorn.stdout.log"#g' \
config/unicorn.rb

# Copy the example Rack attack config
cp -v config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb

# Configure Git global settings for git user, used when editing via web editor
git config --global core.autocrlf input

# Configure Redis connection settings
cp -v config/resque.yml.example config/resque.yml

# Configure GitLab DB Settings
# MySQL only:
cp -v config/database.yml.mysql config/database.yml
sed -i \
-e 's#\(.*pool:\) 10#\1 20#' \
-e 's#\(.*username:\) git#\1 gitlab#' \
-e 's#\(.*password:\) "secure password"#\1 "secret"#' \
-e 's@\(.*\)# \(host:\) localhost@\1\2 localhost@' \
-e 's@\(.*\)# \(socket:\) /tmp/mysql.sock@\1\2 /var/run/mysql/run/mysql.socket@' \
config/database.yml
```

#### Gitlab ruby on Rails
> here we start to face the ruby on rails dependency issue which worries me a lot.
> In order to make life easier, I made my own wheel.

```
# Install the Bundler Gem and all the related rails gems
 gem install bundler --no-ri --no-rdoc

# add one entry in your /etc/hosts file, it's intended to map the ip with a constant hostname
# the ip address can be replaced with either an aceessable gem repository url or a self-constructed ip address
192.168.1.11 gems

# declare your own repo address for gem
[root@gitlab ~]# gem sources -a http://gems:9292/
http://gems:9292/ added to sources
 [root@gitlab ~]# gem sources -r https://rubygems.org/
 https://rubygems.org/ removed from sources
 [root@gitlab ~]# gem sources -l
 *** CURRENT SOURCES ***

http://gems:9292/

 [root@gitlab ~]#  gem install bundler --no-ri --no-rdoc
 Fetching: bundler-1.10.6.gem (100%)
 Successfully installed bundler-1.10.6
 1 gem installed
 [root@gitlab ~]#
```

# config bundler source mirror if it's the first time you initialize your bundler
```
bundle config mirror.https://rubygems.org https://ruby.taobao.org
```

#### Install Gems
Note: As of bundler 1.5.2, you can invoke bundle install -jN (where N the number of your processor cores) and enjoy the parallel gems installation with measurable difference in completion time (~60% faster). Check the number of your cores with nproc. For more information check this post. First make sure you have bundler >= 1.5.2 (run bundle -v) as it addresses some issues that were fixed in 1.5.2.
```
# For PostgreSQL (note, the option says "without ... mysql")
bundle install --deployment --without development test mysql aws kerberos

# Or if you use MySQL (note, the option says "without ... postgres")
bundle install --deployment --without development test postgres aws kerberos
```
Note: If you want to use Kerberos for user authentication, then omit kerberos in the --without option above.


#### Initialize Database and Activate Advanced Features
```
bundle exec rake gitlab:setup RAILS_ENV=production
# Type 'yes' to create the database tables.
# When done you see 'Administrator account created:'
```
Note: You can set the Administrator/root password by supplying it in environmental variable GITLAB_ROOT_PASSWORD as seen below. If you don't set the password (and it is set to the default one) please wait with exposing GitLab to the public internet until the installation is done and you've logged into the server the first time. During the first login you'll be forced to change the default password.
```
bundle exec rake gitlab:setup RAILS_ENV=production GITLAB_ROOT_PASSWORD=yourpassword
```

# default passowrd for root access
```
login.........root
password......5iveL!fe
```

#### Install schedules
```
# Setup schedules
sudo -u gitlab_ci -H bundle exec whenever -w RAILS_ENV=production
```

#### Install Init Script
Download the init script (will be /etc/init.d/gitlab):
```
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
```
If you installed GitLab in another directory or as a user other than the default you should change these settings in /etc/default/gitlab. Do not edit /etc/init.d/gitlab as it will be changed on upgrade.
Make GitLab start on boot:
```
chkconfig gitlab on
```

#### Setup Logrotate
```
sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
```

#### Check Application Status
Check if GitLab and its environment are configured correctly:
```
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
```

#### Compile Assets
```
sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
```

#### Start Your GitLab Instance
```
sudo service gitlab start
# or
sudo /etc/init.d/gitlab restart
```

### Nginx
```
tar xvf nginx-1.9.3.tar.gz
cd nginx-1.9.3
./configure --prefix=/opt/nginx --with-http_ssl_module --with-ipv6 --user=nobody --group=nobody --with-threads --with-stream --with-stream_ssl_module --with-http_gzip_static_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_gunzip_module --with-http_degradation_module
## --with-debug
make -j 4
sudo make install
```

#### Site Configuration
Copy the example site config:
```
sudo cp lib/support/nginx/gitlab /opt/nginx/conf/gitlab.conf
```


### Advanced Setup Tips
#### Using HTTPS
To use GitLab with HTTPS:
1. In gitlab.yml:
  1. Set the port option in section 1 to 443.
  2. Set the https option in section 1 to true.
2. In the config.yml of gitlab-shell:
  1. Set gitlab_url option to the HTTPS endpoint of GitLab (e.g. https://git.example.com).
  2. Set the certificates using either the ca_file or ca_path option.
3. Use the gitlab-ssl Nginx example config instead of the gitlab config.
  1. Update YOUR_SERVER_FQDN.
  2. Update ssl_certificate and ssl_certificate_key.
  3. Review the configuration file and consider applying other security and performance enhancing features.

Using a self-signed certificate is discouraged but if you must use it follow the normal directions then:
1. Generate a self-signed SSL certificate:
```
mkdir -p /etc/nginx/ssl/
cd /etc/nginx/ssl/
sudo openssl req -newkey rsa:2048 -x509 -nodes -days 3560 -out gitlab.crt -keyout gitlab.key
sudo chmod o-r gitlab.key
```
2. In the config.yml of gitlab-shell set self_signed_cert to true.


#### Tips and Helps
```
# compile charlock_holmes
sudo rpm -ivh http://www.muug.mb.ca/mirror/centos/6.6/os/x86_64/Packages/libicu-4.2.1-9.1.el6_2.x86_64.rpm
sudo rpm -ivh http://www.muug.mb.ca/mirror/centos/6.6/os/x86_64/Packages/libicu-devel-4.2.1-9.1.el6_2.x86_64.rpm
yum -y install patch

[root@gitlab charlock_holmes-0.6.9.4]# gem install charlock_holmes -v "0.6.9.4"
Building native extensions.  This could take a while...
Successfully installed charlock_holmes-0.6.9.4
Parsing documentation for charlock_holmes-0.6.9.4
Installing ri documentation for charlock_holmes-0.6.9.4
Done installing documentation for charlock_holmes after 0 seconds
1 gem installe
```


#### Others
```
# Relative url support
    # Uncomment and customize the last line to run in a non-root path
    # WARNING: We recommend creating a FQDN to host GitLab in a root path instead of this.
    # Note that following settings need to be changed for this to work.
    # 1) In your application.rb file: config.relative_url_root = "/gitlab"
    # 2) In your gitlab.yml file: relative_url_root: /gitlab
    # 3) In your unicorn.rb: ENV['RAILS_RELATIVE_URL_ROOT'] = "/gitlab"
    # 4) In ../gitlab-shell/config.yml: gitlab_url: "http://127.0.0.1/gitlab"
    # 5) In lib/support/nginx/gitlab : do not use asset gzipping, remove block starting with "location ~ ^/(assets)/"
    #
    # To update the path, run: sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
    #
    # config.relative_url_root = "/gitlab"
```
