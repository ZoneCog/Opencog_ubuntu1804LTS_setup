# How to build OpenCog with all dependencies #

Building OpenCog is a moving target. As of 27th of April 2018, these instructions work on Ubuntu 16.04 LTS.

## Get Octool and install all dependencies

1. git clone https://github.com/opencog/ocpkg.git
2. ./octool -rdopsicamgbet -l default

## Update GCC and G++

The default GCC 4.8 is missing stdatomic.h and fails at multithreading tests when building Link-Grammar. Therefore you should update to GCC 4.9

sudo apt-repository ppa: ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install gcc-4.9 g++-4.9
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.9

Check that you indeed have a proper version: gcc -v


## Install some missing dependencies

sudo apt-get install valgrind default-jdk ant sqlite libgtk-3-dev

## Install Editline

Go to http://thrysoee.dk/editline/
Get the link to the latest version and download it: wget <link>
tar -xvf <tarball>
sudo apt-get install libncurses5-dev
cd libeditline
./configure
make all
sudo make install
  
## Generate all locales (for Link-Grammar)

sudo ln -s /usr/share/i18n/SUPPORTED /var/lib/locales/supported.d/all
sudo locale-gen

## Get all repositories
git clone https://github.com/opencog/cogutil.git
git clone https://github.com/opencog/moses.git
git clone https://github.com/opencog/atomspace.git
git clone https://github.com/opencog/link-grammar.git

## Build and install Cogutil

cd cogutil
mkdir build
cd build
cmake ..
make
make test

### if all tests pass
sudo make install

## Initialize PosgreSQL database

sudo -u posrgres psql template1
ALTER USER posrgres with encrypted password 'password';

### Edit confs
sudo vim /etc/postgresql/9.3/main/postgresql.conf

### Enable listening to all addresses
listen_address = '*'

### User permissions
sudo vim /etc/postgresql/9.3/main/pg_hba.conf

### Main user
replace peer with md5:
local      all     postgres     md5

### Other users
host all all 0.0.0.0/0 md5

### Configure ODBC drivers:

https://github.com/opencog/atomspace/blob/master/opencog/persist/sql/README.md

### Continue initialization

sudo /etc/init.d/postgresql restart
createuser -U postgres -d -e -E -l -P -r -s opencog_tester         (set password cheese)
createuser -U postgres -d -e -E -l -P -r -s opencog_user           (set password cheese)

### Create databases

sudo -u postgres createdb mycogdata
sudo -u postgres createdb opencog_test

### Login to PostgreSQL (psql -U postgres) and give permissions

GRANT ALL privileges on database mycogdata to opencog_user;
GRANT ALL privileges on database opencog_test to opencog_tester;

### Add tables to databases
cd atomspace
sudo cat opencog/persist/sql/multi-driver/atom.sql | psql mycogdata -U opencog_user -W -h localhost
sudo cat opencog/persist/sql/multi-driver/atom.sql | psql opencog_test -U opencog_tester -W -h localhost

## Install Atomspace

cd atomspace
mkdir build
cd build
cmake ..
make
make test

### NOTE: the following tests will fail: PutLinkUTest, AttentionUTest

sudo make install

## Install MOSES

cd moses
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
make test

### if all tests pass:
sudo make install

## Get and install Link-Grammar

See https://github.com/opencog/link-grammar and get the tarball or git clone

./configure --enable-widec
make
make check

### if all tests pass:
sudo su
make install
ldconfig
exit
make installcheck

## Install OpenCog

cd opencog
mkdir build
cd build
cmake ..
make
make test

### if all tests pass:
sudo make install
