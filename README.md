# How to build OpenCog with all dependencies #

Building OpenCog is a moving target. As of 27th of April 2018, these instructions work on Ubuntu 16.04 LTS.

## Setup Vagrant with Ubuntu 16.04 LTS

1. Install VirtualBox and Vagrant(the method depends on your OS; In Mac OS High Sierra I use Homebrew by calling brew install vagrant)
2. Init with Ubuntu 16.04 LTS: vagrant init ubuntu/trusty64
3. vagrant up
4. vagrant ssh

## Add swap to Vagrant (3GB in this example; adjust to your liking)

1. cd /
2. sudo dd if=/dev/zero of=swapfile bs=1M count=3000
3. sudo mkswap swapfile
4. sudo swapon swapfile
5. sudo vim etc/fstab

### Add the following line to fstab:
/swapfile none swap sw 0 0

6. check that the swapfile was actually added: cat /proc/meminfo

### Install Git

1. sudo apt-get install git

## Get Octool and install all dependencies (https://github.com/opencog/ocpkg)

1. wget http://raw.github.com/opencog/ocpkg/master/ocpkg -O octool && chmod +rx octool && ./octool -h
2. ./octool -rdopsicamgbet -l default

## Update GCC and G++

The default GCC 4.8 is missing stdatomic.h and fails at multithreading tests when building Link-Grammar. Therefore you should update to GCC 4.9

1. sudo apt-repository ppa: ubuntu-toolchain-r/test
2. sudo apt-get update
3. sudo apt-get install gcc-4.9 g++-4.9
4. sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.9

5. Check that you indeed have a proper version: gcc -v


## Install some missing dependencies

1. sudo apt-get install valgrind default-jdk ant sqlite libgtk-3-dev

## Install Editline

1. Go to http://thrysoee.dk/editline/
2. Get the link to the latest version and download it: wget <link>
3. tar -xvf <tarball>
4. sudo apt-get install libncurses5-dev
5. cd libeditline
6. ./configure
7. make all
8. sudo make install
  
## Generate all locales (for Link-Grammar)

1. sudo ln -s /usr/share/i18n/SUPPORTED /var/lib/locales/supported.d/all
2. sudo locale-gen

## Get all repositories
1. git clone https://github.com/opencog/cogutil.git
2. git clone https://github.com/opencog/moses.git
3. git clone https://github.com/opencog/atomspace.git
4. git clone https://github.com/opencog/link-grammar.git

## Build and install Cogutil

1. cd cogutil
2. mkdir build
3. cd build
4. cmake ..
5. make
6. make test

### if all tests pass
7. sudo make install

## Initialize PosgreSQL database

1. sudo -u posrgres psql template1
2. ALTER USER posrgres with encrypted password 'password';

### Edit confs
1. sudo vim /etc/postgresql/9.3/main/postgresql.conf

#### Enable listening to all addresses
listen_address = '*'

### User permissions
1. sudo vim /etc/postgresql/9.3/main/pg_hba.conf

#### Main user
replace peer with md5:
local      all     postgres     md5

#### Other users
host all all 0.0.0.0/0 md5

### Configure ODBC drivers:

1. see: https://github.com/opencog/atomspace/blob/master/opencog/persist/sql/README.md

### Continue initialization

1. sudo /etc/init.d/postgresql restart
2. createuser -U postgres -d -e -E -l -P -r -s opencog_tester         (set password cheese)
3. createuser -U postgres -d -e -E -l -P -r -s opencog_user           (set password cheese)

### Create databases

1. sudo -u postgres createdb mycogdata
2. sudo -u postgres createdb opencog_test

### Login to PostgreSQL (psql -U postgres) and give permissions

1. GRANT ALL privileges on database mycogdata to opencog_user;
2. GRANT ALL privileges on database opencog_test to opencog_tester;

### Add tables to databases
1. cd atomspace
2. sudo cat opencog/persist/sql/multi-driver/atom.sql | psql mycogdata -U opencog_user -W -h localhost
3. sudo cat opencog/persist/sql/multi-driver/atom.sql | psql opencog_test -U opencog_tester -W -h localhost

## Build and install Atomspace

1. cd atomspace
2. mkdir build
3. cd build
4. cmake ..
5. make
6. make test

### NOTE: the following tests will fail: PutLinkUTest, AttentionUTest

7. sudo make install

## Build and install MOSES

1. cd moses
2. mkdir build
3. cd build
4. cmake -DCMAKE_BUILD_TYPE=Release ..
5. make
6. make test

### if all tests pass:
7. sudo make install

## Get, build and install Link-Grammar

1. See https://github.com/opencog/link-grammar and get the tarball or git clone

2. ./configure --enable-widec
3. make
4. make check

### if all tests pass:
5. sudo su
6. make install
7. ldconfig
8. exit
9. make installcheck

## Build and install OpenCog

1. cd opencog
2. mkdir build
3. cd build
4. cmake ..
5. make
6. make test

### if all tests pass:
7. sudo make install
