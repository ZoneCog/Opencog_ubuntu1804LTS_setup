# How to build OpenCog with all dependencies #

Building OpenCog is a moving target. As of 8th of May 2018, these instructions work on Ubuntu 18.04 LTS.

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

## Install dependencies (Contains extras for making chat related stuff work)

1. sudo apt-get install gcc g++ make cmake libboost-all-dev cython python-nose python3-nose python-pytest python3-pytest postgresql postgresql-contrib postgresql-client guile-2.2-dev libtbb-dev libgearman-dev libpq-dev python-pip libblas-dev liblapack-dev uuid-dev doxygen libiberty-dev binutils-dev valgrind postgresql postgresql-client postgresql-contrib libpq-dev liblogging-stdlog0 liblogger-syslog-perl libzmq3-dev libprotobuf-dev libgtk-3-dev default-jdk python-cffi libffi-dev irssi irssi-scripts ca-certificates libcrypt-blowfish-perl libcrypt-dh-perl libcrypt-openssl-bignum-perl libmath-bigint-gmp-perl

2. Read what apt-get says, if it tells to update then run `sudo apt-get update` and re-run step 1.
3. sudo pip install -U setuptools
4. sudo easy_install cython nose
5. sudo ln -s /usr/share/i18n/SUPPORTED /var/lib/locales/supported.d/all
6. sudo locale-gen
7. Get Octool by following the guide here: https://github.com/opencog/ocpkg
8. ./octool -rdospicamgvbe -l default -l java

## Update PYTHONPATH

1. Check where your link-grammar exists (most likely in /usr/local/lib/python3.6/site-packages/link-grammar)
2. Add the following to your ~/.bash_profile

```
export PYTHONPATH=$PYTHONPATH:/usr/local/share/moses/python/:/usr//local/lib/python3.5/dist-packages/:/usr/local/lib/python3.5/site-packages/:/usr//local/lib/python3.6/dist-packages/:/usr/local/lib/python3.6/site-packages/link-grammar/
```
2. source ~/.bash_profile

NOTE: You need to check where atomspace actually is. You can do it by running `find /usr |grep opencog |grep python`

## Update Guile paths

1. Add the following to ~/.guile

```
(add-to-load-path "/usr/local/share/opencog/scm")
(add-to-load-path ".")
```

## Initialize PostgreSQL database

### Add DB tweaks

I use the following config at the end of /etc/postgresql/10/main/postgresql.conf:

```
shared_buffers = 256MB
work_mem = 32MB
effective_cache_size = 512MB
fsync = off
synchronous_commit = off
ssl = off
autovacuum = on
track_counts = on

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
5. \q

### Edit confs
1. sudo vim /etc/postgresql/10/main/postgresql.conf

#### Enable listening to all addresses
listen_address = '*'

#### sysctl.conf
1. sudo vim /etc/sysctl.conf
2. Add the following:

```
kernel.shmmax = 6440100100
```

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
1. git clone https://github.com/opencog/atomspace.git
2. cd atomspace
3. cat opencog/persist/sql/multi-driver/atom.sql | psql mycogdata -U opencog_user -W -h localhost
4. cat opencog/persist/sql/multi-driver/atom.sql | psql opencog_test -U opencog_tester -W -h localhost

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

## Build and install OpenCog

1. cd opencog
2. mkdir build
3. cd build
4. cmake ..
5. make
6. make test

### Some tests will fail:
7. sudo make install

## Install RelEx

1. git clone https://github.com/opencog/relex
2. cd relex
3. install-scripts/install-ubuntu-dependencies.sh

## How to run e.g. chatbot

1. Open another terminal window and log in: `vagrant ssh`
2. cd relex
3. bash opencog_server.sh
4. Go back to your original window and go to opencog build directory: `cd ~/opencog/build`
5. `guile -l ../opencog/nlp/chatbot/run-chatbot.scm`
6. You can post a question using guile, e.g.: `(process-query "luser" "Are you a bot?")`
6. For more examples see: https://github.com/opencog/opencog/tree/master/opencog/nlp/chatbot
