# complete the following task
1. slurmdbd configuration
1. configless setup : client should sync the file automatically from the master
1. High availablity
1. reservation 

## solution 1 :
### Step 1: install database on master and start the services
```bash
# installing mariadb
yum install mariadb-server mariadb-devel -y

# start the services in mariadb
systemctl enable mariadb;
systemctl start mariadb;
systemctl status mariadb;
```
### Step 2: create mysql user and give privileges
```bash
# to create user
mysql
    # in mariadb 
    GRANT ALL ON slurm_acct_db.* TO 'slurm'@'localhost' IDENTIFIED BY '1234' with grant option;
    SHOW VARIABLES LIKE 'have_innodb';
    FLUSH PRIVILEGES;
    CREATE DATABASE slurm_acct_db;
    quit;

# to verify the database permission for slurm user 
mysql -p -u slurm
    # try to login with 1234 password

# create a new file at location /etc/my.cnf.d/innodb.cnf
[mysqld]
innodb_buffer_pool_size=1024M
innodb_log_file_size=64M
innodb_lock_wait_timeout=900

# restart the mariadb service 
systemctl stop mariadb
mv /var/lib/mysql/ib_logfile? /tmp/
systemctl start mariadb

# to check current setting in MySQL:
mysql -p -u slurm
    # type the following command
    SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

# create slurmdbd.conf file
cat << EOF >> /etc/slurm/slurmdbd.conf
DbdAddr=localhost
DbdHost=localhost
DbdPort=6819
StoragePass=1234
StorageLoc=slurm_acct_db
StorageType=accounting_storage/mysql
EOF

# change permissions of the slurmdbd
chown slurm: /etc/slurm/slurmdbd.conf
chmod 600 /etc/slurm/slurmdbd.conf
touch /var/log/slurmdbd.log
chown slurm: /var/log/slurmdbd.log

# to check logs of slurmdbd.conf
slurmdbd -D -vvv

```
### Step 3: restart the slurmctld to see the effects
```bash
# start he slurmdbd service
systemctl enable slurmdbd 
systemctl start slurmdbd 
systemctl status slurmdbd
```


## solution 2 : 
### Step 1: 
```bash
# in order to configure slurm configless we need to change a property in slurm.conf file which is 
echo "SlurmctldParameters=enable_configless" >> /etc/slurm/slurm.conf
systemctl restart slurmctld;
systemctl status slurmctld;
```
### Step 2: 
```bash
# make entry in resolv.conf 
echo "_slurmctld._tcp 3600 IN SRV 0 0 6817 slurm-master" >> /etc/slurm/resolve.conf
```
### Step 3: configure slurmd configure on client
```bash
# start slurmd with --conf-server parameter on client 
slurmd --conf-server slurm-master:6817

# restart the services
systemctl restart slurmd;
systemctl status slurmd;

```
## solution 3 :

## solution 4 :


