# GitLab Install Guide
```
Distribution      : CentOS6.5/6 Minimal
GitLab version    : 7.13.2
Web Server        : Nginx
Init system       : sysvinit
Database          : MySQL
Contributors      : @lvfeng1130
Additional Notes  : In order to install gitlab under a circumstance that Linux server has no internet access, All reiled code or pacakges are downloaded
via chrome
```

## Overview
The installation manual base itself on [gitlab-recipes Install Guild](https://github.com/gitlabhq/gitlab-recipes/tree/master/install/centos),
It attempts to install Gitlab from ground up with http access in chrome and only yum local repository.

### Important Notes

## Extend the size of partition /opt

df -h
fdisk /dev/sdc
pvs
pvcreate /dev/sdc
pvs
vgextend systemvg /dev/sdc
lvextend -L 100G /dev/mapper/systemvg-optlv
resize2fs /dev/mapper/systemvg-optlv


1. install Development tools
yum groupinstall "Development tools"

2. install requied headers files for git
yum -y install gcc gcc-c++ zlib-devel perl-ExtUtils-MakeMaker autoconf kernel-devel tcl tcl-devel

3. complile git-2.4.5.tar.xz from source
tar xvf git-2.4.5.tar.xz
cd git-2.4.5
make configure
./configure --prefix=/usr
make -j 8 all   # fast as shit..
sudo make install

#. compile nodejs for gitlab environment later
tar xvf node-v0.12.7.tar.gz
cd  node-v0.12.7
./configure
make -j 4
sudo make install

#. update python from 2.6 to 2.7 for some custom settings
# I didn't install python for the sake of being an idiot in python :(
# Install these modules for python
yum install sqlite-devel openssl-devel readline-devel gdbm-devel bzip2-devel ncurses-devel

./configure --prefix=/usr/local
make -j8
sudo make install

#. compile cmake to install mariadb later
cd cmake-3.2.3
./bootstrap && make -j 4
sudo make install

4.  it's recommaned that we'are gonna need EPEL and PUIAS Computational repositories to properly install environment for GitLab  
so I put their online repo https here, to manually fetch any packages if required.  :( mind fuck!

Aliyun: http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/
EPEL: http://dl.fedoraproject.org/pub/epel/6/x86_64/
PUIAS: http://springdale.math.ias.edu/data/puias/computational/6.6/x86_64/

5. the dependencies tools requied for GitLab, you may not be able to install them all from your local repository.
  yum -y install  readline readline-devel ncurses-devel gdbm-devel glibc-devel tcl-devel openssl-devel curl-devel expat-devel db4-devel byacc sqlite-devel gcc-c++ libyaml libyaml-devel libffi libffi-devel libxml2 libxml2-devel libxslt libxslt-devel libicu libicu-devel system-config-firewall-tui python-devel redis sudo wget crontabs logwatch logrotate perl-Time-HiRes git vim-enhanced

6. what we truely need is, be aware that we already compile git from source, so exclude it from the list.
[root@lifestealer ~]# for i in `echo "vim-enhanced readline readline-devel ncurses-devel gdbm-devel glibc-devel tcl-devel openssl-devel curl-devel expat-devel db4-devel byacc sqlite-devel gcc-c++ libyaml libyaml-devel libffi libffi-devel libxml2 libxml2-devel libxslt libxslt-devel libicu libicu-devel system-config-firewall-tui python-devel redis sudo wget crontabs logwatch logrotate perl-Time-HiRes git"` ;do  rpm -q $i  1>/dev/null || echo $i  ;done |xargs

vim-enhanced readline-devel ncurses-devel tcl-devel openssl-devel curl-devel expat-devel sqlite-devel libyaml libyaml-devel libffi-devel libxml2-devel libxslt libxslt-devel libicu libicu-devel system-config-firewall-tui python-devel redis wget logwatch perl-Time-HiRes git

7. some of above packages can obviously be installed from local repository (The DVD Distro, learn how to yum install from CentOS-6.6-x86_64-bin-DVD1.iso)
e.g.
yum install openssl-devel readline-devel tcl-devel curl-devel expat-devel sqlite-devel libyaml libyaml-devel python-devel logwatch perl-Time-HiRes newt-python
# not necessary need system-config-firewall-tui

8. while others may need to download online, we will put redis aside cause we're going to compile it seperately
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libffi-3.0.5-3.2.el6.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libffi-devel-3.0.5-3.2.el6.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxml2-2.7.6-14.el6_5.2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxml2-devel-2.7.6-14.el6_5.2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxslt-1.1.26-2.el6_3.1.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libxslt-devel-1.1.26-2.el6_3.1.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libicu-4.2.1-9.1.el6_2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/libicu-devel-4.2.1-9.1.el6_2.x86_64.rpm
wget -nc -c --progress http://mirrors.aliyun.com/centos/6.6/os/x86_64/Packages/system-config-firewall-tui-1.2.27-7.1.el6.noarch.rpm

9. install redis from source
tar xvf redis-3.0.3.tar.gz
cd redis-3.0.3
vi README
make MALLOC=jemalloc
make test

Manually define
cat>>/opt/redis/redis.conf<<EOF
daemonize yes
appendonly yes
#disable dump
#db
EOF

10. start redis with src/redis-server /opt/redis/redis.conf

11. Configure sendmail
yum install sendmail-cf



# install Mariadb

yum install libtevent-devel libevent lzo lzop

Download lz4 rpm pacakges from EPEL repo
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


# install package kytea and libzmq from source
wget http://www.phontron.com/kytea/download/kytea-0.4.7.tar.gz
wget http://download.zeromq.org/zeromq-4.1.2.tar.gz

#install
sudo rpm -ivh lz4-r131-1.el6.x86_64.rpm
sudo rpm -ivh lz4-static-r131-1.el6.x86_64.rpm
sudo rpm -ivh lz4-devel-r131-1.el6.x86_64.rpm
sudo rpm -ivh msgpack-0.5.7-5.el6.x86_64.rpm
sudo rpm -ivh msgpack-devel-0.5.7-5.el6.x86_64.rpm
sudo rpm -ivh cracklib-devel-2.8.16-4.el6.x86_64.rpm
cracklib-devel-2.8.16-4.el6.x86_64.rpm
sudo rpm -ivh libevent-headers-1.4.13-4.el6.noarch.rpm libevent-devel-1.4.13-4.el6.x86_64.rpm libevent-doc-1.4.13-4.el6.noarch.rpm
sudo rpm -ivh lzo-minilzo-2.03-3.1.el6.x86_64.rpm lzo-devel-2.03-3.1.el6.x86_64.rpm


# kytea
tar -xzf kytea-0.4.7.tar.gz
cd kytea-0.4.7
./configure
make -j 4
make check
make install
kytea --help

# install zeromq
# download libsodium for dependencies
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/libsodium-0.4.5-3.el6.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/libsodium-devel-0.4.5-3.el6.x86_64.rpm



tar xvf zeromq-4.1.2.tar.gz
cd zeromq-4.1.2
./configure && make -j8 && sudo make install

# compile openpam.h
./configure --with-pam-unix --with-su
make
sudo make install

# enable tcmalloc for mariadb
wget http://download.savannah.gnu.org/releases/libunwind/libunwind-1.1.tar.gz #not usable
wget http://down1.chinaunix.net/distfiles/libunwind-1.0.tar.gz
wget http://cznic.dl.sourceforge.net/project/openpam/openpam/Ourouparia/openpam-20140912.tar.gz
wget http://nchc.dl.sourceforge.net/project/cracklib/cracklib/2.9.2/cracklib-2.9.2.tar.gz
wget https://codeload.github.com/msgpack/msgpack-c/zip/master
wget http://snowball.tartarus.org/dist/libstemmer_c.tgz

#Installation of CrackLib

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

# Install msgpack-c
unzip msgpack-c-master.zip
cd msgpack-c-master
./bootstrap
./configure
make
sudo make install

#install openpam
tar xvf openpam-20140912.tar.gz
cd openpam-20140912
./configure --with-pam-unix --with-su
make -j8
sudo make install


git clone https://github.com/mobing/gperftools.git
cd gperftools

# gperftools need  Autoconf version 2.68 or higher is required
rpm -e --nodeps autoconf-2.63
#install autoconf
tar xvf autoconf-2.69.tar.xz
cd  autoconf-2.69
./configure
make
make install


# there is an error when make install libunwind because of the compability of autoconf tools, run autoreconf -i -f
tar xvf libunwind-0.99.tar.gz
cd libunwind-0.99
CFLAGS=-fPIC ./configure
make CFLAGS=-fPIC
sudo make CFLAGS=-fPIC install

# complie gperftools
tar xvf gperftools-2.0.tar.gz
cd gperftools-2.0
./configure
make -j 2
sudo make install

cat >> /etc/ld.so.conf.d/maradb-10.1.5.conf<<EOF
/usr/local/lib/
EOF

#install libstemmer-c
#tar xvf libstemmer_c.tgz #
unzip libstemmer_c-master.zip
cd libstemmer_c-master


# install boost support for mariadb-10.1
  yum -y install boost boost-devel bison libtool


sudo ln -s /usr/local/lib/libkytea.* /usr/lib64
sudo ln -s /usr/local/lib/libmsgpack* /usr/lib64


MYSQL_TAR_GZ=""


groupadd mysql
useradd -g mysql mysql

tar xvf ${MYSQL_TAR_GZ}
cd ${MYSQL_TAR_GZ%.tar*gz}

MYSQL_SOCK_LOCATION="/var/run/mysql/run/mysql.socket"
MYSQL_INSTALL_DIR="/opt/mysql"
MYSQLDATADIR="/data/mysql"

cmake . \
-DCMAKE_INSTALL_PREFIX=${MYSQL_INSTALL_DIR} \
-DMYSQL_DATADIR=${MYSQLDATADIR} \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
### removed this options in Red Hat Enterprise Linux Server
# release 6.3 (Santiago) because of the unconqured compile error
#-DCMAKE_EXE_LINKER_FLAGS='-ltcmalloc' \
###
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

make -j8
sudo make install



if [ ! -d ${MYSQLDATADIR} ];then
    sudo  mkdir -p ${MYSQLDATADIR}
fi

sudo chmod +w ${MYSQL_INSTALL_DIR}
sudo chown -R mysql:mysql ${MYSQLDATADIR}
sudo chmod go-rwx ${MYSQLDATADIR} -R


sudo cp support-files/my-large.cnf /etc/my.cnf
sudo cp support-files/mysql.server /etc/init.d/mysqld

sudo sed -i "s#^basedir=.*#&${MYSQL_INSTALL_DIR}#" /etc/init.d/mysqld
sudo sed -i "s#^datadir=.*#&${MYSQLDATADIR}#" /etc/init.d/mysqld

sudo chmod 755 /etc/init.d/mysqld

sudo chkconfig --add mysqld
sudo chkconfig --level 345  mysqld on


sudo ln -s ${MYSQL_INSTALL_DIR}/lib/ /usr/lib/mysql
sudo ln -s ${MYSQL_INSTALL_DIR}/include/mysql/ /usr/include/mysql

sudo ln -s ${MYSQL_INSTALL_DIR}/bin/mysql /usr/bin/mysql
sudo ln -s ${MYSQL_INSTALL_DIR}

sudo cat > /etc/ld.so.conf.d/maradb-10.1.5.conf<<EOF
${MYSQL_INSTALL_DIR}/lib/mysql
${MYSQL_INSTALL_DIR}/lib
EOF

sudo ldconfig
${MYSQL_INSTALL_DIR}/scripts/mysql_install_db --basedir=${MYSQL_INSTALL_DIR} \
--datadir=${MYSQLDATADIR} \
--user=mysql

service mysqld start

${MYSQL_INSTALL_DIR}/bin/mysql_secure_installation

sudo ln -s /opt/mysql/ /usr/local/mysql

UserName: root
Password: L!f3@SuN!ing^^

# Create instance for gitlab
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

UserName: gitlab
Password: China4theWin@TI5

[root@itsasitapp203 ~]# groupadd redis
[root@itsasitapp203 ~]# useradd -m -g redis redis
[root@itsasitapp203 ~]# groupadd git
[root@itsasitapp203 ~]# useradd -m -g git git
[root@itsasitapp203 ~]# usermod -Ggit,redis -a git
[root@itsasitapp203 ~]# groupadd mysql
[root@itsasitapp203 ~]# useradd -m -g mysql mysql
[root@itsasitapp203 ~]# usermod -Gmysql,git git
[root@itsasitapp203 ~]# groupadd nginx
[root@itsasitapp203 ~]# useradd -s /sbin/nologin -g nginx nginx


# Install redis
# Create the directory which contains the socket
mkdir /var/run/redis
chown redis:redis /var/run/redis
chmod 755 /var/run/redis


tar xvf redis-3.0.3.tar.gz
cd redis-3.0.3
make MALLOC=jemalloc
make test

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

Cleanup: may take some time... OK
make[1]: Leaving directory `/opt/redis/redis-3.0.3/src'

sudo mkdir /opt/redis
sudo mv  redis-3.0.3/ /opt/redis/redis-3.0.3/

sudo chown redis:redis /opt/redis -R

# ass wipe some simple configurations
sudo mkdir /etc/redis
sudo egrep -Ev "^$|^#" /opt/redis/redis-3.0.3/redis.conf > /etc/redis/redis.conf

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

su - redis

# some tips make the environent looks more friendly
cat >> .bash_profile<<EOF
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
EOF

# put script redis.sh under /etc/init.d so it will bootstrap with init
cat > /etc/init.d/redis<<EOF
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
EOF

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


# Create System user git
groupadd git
useadd -m -g git git

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


# Instill ruby, Based on the offcial installation that ruby 2.1.6 is the supported version

tar xvf ruby-2.1.6.tar.gz
cd ruby-2.1.6
make
make install

# here we start to face the ruby on rails dependency issue which worries me the most.
# To Install the Bundler Gem and all the related rails gems
 gem install bundler --no-ri --no-rdoc

# map the ip with another url
10.19.37.141 gems

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

 # Configure GitLab, Gitlab-shell, etc

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

# Gitlab-shell
ln -s gitlabhq-7-13-stable gitlab
ln -s gitlab-shell-master gitlab-shell

cd gitlab-shell
cp config.yml.example config.yml

# gitlab_url, check the introduciton for advise
sed -i 's#^gitlab_url: .*#gitlab_url: "http://localhost:8080/"#g' config.yml

# repos_path
sed -i 's#^repos_path: .*#repos_path: "/data/git/repositories"#g' config.yml

# log file path
mkdir /opt/git/gitlab-shell/log/
chown git:git /opt/git/gitlab-shell/log/ -R
chmod -R u+rwX,go-w /opt/git/gitlab-shell/log/

sed -i 's@^# \(log_file: \)"/home/git/gitlab-shell/gitlab-shell.log"@\1"/opt/git/gitlab-shell/log/gitlab-shell.log"@g' config.yml

# Do setup
./bin/install

# GitLab
cd gitlab

cp config/gitlab.yml.example config/gitlab.yml

sudo chown -R git log/
sudo chown -R git tmp/
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/
sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/
sudo chmod -R u+rwX  public/uploads

# the sed will speak for itself as regard of settings
sed -i \
-e 's#\(.*host:\) localhost#\1 gitlab.cnsuning.com#g' \
-e 's#\(.*port:\) 80#\1 443#g' \
-e 's#\(.*https:\) false#\1 true#g' \
-e 's#\(.*email_from:\) example@example.com#\1 lvfeng@cnsuning.com#g' \
config/gitlab.yml

# set smtp email service,
cp config/initializers/smtp_settings.rb.sample config/initializers/smtp_settings.rb
sed -i \
-e 's#\(address:\) "email.server.com"#\1 "mail.cnsuning.com"#g' \
-e 's#\(port:\) 456#\1 25#g' \
-e 's#\(user_name:\) "smtp"#\1 "lvfeng"#g' \
-e 's#\(domain:\) "gitlab.company.com"#\1 "cnsuning.com"#g' \
-e 's#\(password:\) "123456"#\1 "@Cqmyg1130"#g' \
config/initializers/smtp_settings.rb


mkdir /opt/git/gitlab-satellites
sudo chmod u+rwx,g=rx,o-rwx /opt/git/gitlab-satellites


# Enable cluster mode if you expect to have a high load instance
# Ex. change amount of workers to 3 for 2GB RAM server
# Set the number of workers to at least the number of cores
cp config/unicorn.rb.example config/unicorn.rb

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
cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb

# Configure Git global settings for git user, used when editing via web editor
git config --global core.autocrlf input

# Configure Redis connection settings
cp config/resque.yml.example config/resque.yml

# Configure GitLab DB Settings
# MySQL only:
cp config/database.yml.mysql config/database.yml
sed -i \
-e 's#\(.*pool:\) 10#\1 20#' \
-e 's#\(.*username:\) git#\1 gitlab#' \
-e 's#\(.*password:\) "secure password"#\1 "China4theWin@TI5"#' \
-e 's@\(.*\)# \(host:\) localhost@\1\2 localhost@' \
-e 's@\(.*\)# \(socket:\) /tmp/mysql.sock@\1\2 /var/run/mysql/run/mysql.socket@' \
config/database.yml

#


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
