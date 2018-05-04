# How to build OpenCog with all dependencies #

Building OpenCog is a moving target. As of 4th of May 2018, these instructions work on Ubuntu 18.04 LTS.

NOTE: I encountered an error with Link-Grammar when building OpenCog. Apparently version 5.5 will fix this issue. So this guide does not work in its current state.

NOTE2: Also MOSES CythonTest fails.

## Setup Vagrant (VirtualBox) with Ubuntu 18.04 LTS

1. Install VirtualBox and Vagrant(the method depends on your OS; In Mac OS High Sierra I use Homebrew by calling brew install vagrant)
2. Init with Ubuntu 18.04 LTS: vagrant init bento/ubuntu-18.04
3. vagrant up
4. vagrant ssh

## Add swap to Vagrant (8GB in this example; adjust to your liking)

1. cd /
2. sudo dd if=/dev/zero of=swapfile bs=1M count=8000
3. sudo chmod 600 swapfile
4. sudo mkswap swapfile
5. sudo swapon swapfile
6. sudo vim etc/fstab

### Add the following line to fstab:
/swapfile none swap sw 0 0

7. check that the swapfile was actually added: swapon --show

## You may also want to add more storage to your Virtual Machine (depends on your VirtualBox environment):

1. vagrant halt
2. Go to the directory where you have your Virtual Box VMs (e.g. in Mac cd ~/VirtualBox VMs/some_id)
3. VBoxManage clonehd ubuntu_something.vmdk ubuntu_something.vdi --format VDI
4. VBoxManage ubuntu_something.vdi --resize 51200 (50 GB)
5. Go to VirtualBox, unmount and delete the old .vmdk and mount the newly created .vdi.

### Install dependencies

1. bash install-debian-dependencies.sh
2. sudo apt-get install gcc g++ make cmake libboost-all-dev cxxtest libiberty-dev doxygen valgrind default-jdk ant sqlite libopenmpi-dev postgresql postgresql-contrib libpq-dev pgadmin3 postgresql-client libgtk-3-dev swig m4 autoconf autoconf-archive flex graphviz hunspell sqlite3 aspell clang cython python-pip python3-pip guile-2.0-dev libzmq3-dev libprotobuf-dev libgearman-dev liboctomap-dev libncurses5-dev
3. sudo pip install nose pytest Cython
4. pip install nose pytest Cython
5. sudo pip3 install nose pytest Cython
6. pip3 install nose pytest Cython
7. sudo easy_install cython nose

## Install Oracle Java 8 and set it as default
1. sudo add-apt-repository ppa:webupd8team/java
2. sudo apt-get update
3. sudo apt-get install oracle-java8-installer
4. sudo apt install oracle-java8-set-default

## Install GHC (Haskell)
1. sudo add-apt-repository ppa:hvr/ghc
2. sudo apt-get update
3. sudo apt-get install ghc

## Generate all locales (for Link-Grammar)

1. sudo ln -s /usr/share/i18n/SUPPORTED /var/lib/locales/supported.d/all
2. sudo locale-gen

## Be sure not to have any ODBC drivers installed.

1. dpkg --get-selections | grep odbc
2. sudo apt-get purge [package]

## Install Editline

1. Go to http://thrysoee.dk/editline/
2. Get the link to the latest version and download it: wget <link>
3. tar -xvf <tarball>
4. sudo apt-get install libncurses5-dev
5. cd libeditline
6. ./configure
7. make all
8. sudo make install

## Get all repositories
1. git clone https://github.com/opencog/cogutil.git
2. git clone https://github.com/opencog/moses.git
3. git clone https://github.com/opencog/atomspace.git
4. git clone https://github.com/opencog/opencog.git

## Build and install Cogutil

1. cd cogutil
2. mkdir build
3. cd build
4. cmake ..
5. make
6. make test

### if all tests pass
7. sudo make install

## Initialize PostgreSQL database

### Add DB tweaks

I use the following config at the end of /etc/postgresql/10/main/postgresql.conf:

```
shared_buffers = 256MB
work_mem = 32MB
effective_cache_size = 512MB
synchronous_commit = off

checkpoint_timeout = 1h
max_wal_size = 4GB
checkpoint_completion_target = 0.9

# If you have an SSD drive add the following:
seq_page_cost = 0.1
random_page_cost = 0.15
effective_io_concurrency = 5
```

## Initialize the first users (note: replace vagrant with your username if you are not using Vagrant).

1. sudo -u postgres psql template1
2. ALTER USER postgres with encrypted password 'password';
3. CREATE ROLE vagrant WITH SUPERUSER;
4. ALTER ROLE vagrant WITH LOGIN;

### Edit confs
1. sudo vim /etc/postgresql/10/main/postgresql.conf

#### Enable listening to all addresses
listen_address = '*'

### User permissions
1. sudo vim /etc/postgresql/10/main/pg_hba.conf

#### Main user
replace peer with md5:
local      all     postgres     md5

#### Other users
local all all   md5
host all all 0.0.0.0/0 md5

### Continue initialization

1. sudo /etc/init.d/postgresql restart
2. createuser -U postgres -d -e -E -l -P -r -s opencog_tester         (set password cheese)
3. createuser -U postgres -d -e -E -l -P -r -s opencog_user           (set password cheese)

### Create databases

1. sudo -u postgres createdb mycogdata
2. sudo -u postgres createdb opencog_test

### Login to PostgreSQL (psql -U postgres) and give permissions

1. GRANT ALL privileges ON DATABASE mycogdata to opencog_user;
2. GRANT ALL privileges ON DATABASE opencog_test to opencog_tester;

### Add tables to databases
1. cd atomspace
2. cat opencog/persist/sql/multi-driver/atom.sql | psql mycogdata -U opencog_user -W -h localhost
3. cat opencog/persist/sql/multi-driver/atom.sql | psql opencog_test -U opencog_tester -W -h localhost

### Test that DB works
1. psql mycogdata -U opencog_user
2. INSERT INTO TypeCodes (type, typename) VALUES (97, 'SemanticRelationNode');
3. \q
4. psql opencog_test -U opencog_tester
5. INSERT INTO TypeCodes (type, typename) VALUES (97, 'SemanticRelationNode');

Both should display:
```
INSERT 0 1
```
NOTE: Also check that all tables in both databases are owned by their intended users by using \l


## Build and install Atomspace

1. cd atomspace
2. mkdir build
3. cd build
4. cmake ..
5. make
6. make test

### NOTE: the following tests will fail: PutLinkUTest

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
2. ./configure
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
