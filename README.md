# postgresql
upgrade 
PostgreSQL database Upgrade from 9.6 to 12.2

PostgreSQL databases can be migrated without doing database dump-and-restore. Here are the steps to migrate from Postgres 9.6 to 12.2 in Ubuntu 18
Assuming 9.6 is already installed on server

Upgrade Pre-checks ( good practice);
1.	Before upgrade take full backup of database from pg_dump
2.	Check active sessions
3.	Check jobs scheduled on particular time
4.	Note down 9.6 directories for future use if required
Start the upgrade process:
Step 1:  
Let’s first check the list of installed postgres related packages, service status and the running cluster details.
Packages list: $ dpkg -l | grep postgres
 
Service:
$ systemctl list-dependencies postgresql
 
Cluster:
$ pg_lsclusters
 

Step 2: Installation of Postgresql 12
* note that apt repo is already added during installation of the existing version
 
$ apt-get update
$ apt-get install postgresql-12
Postgres 12 cluster will start automatically after the installation.
Verify status:
After the installation , again run 
dpkg -l | grep postgres
 

Run pg_clusters again, 
 
Now you will see both 9.6 and 12 is online.
Step 3: Upgrading Postgresql 9.6 to 12:
To perform the upgrade we will use pg_upgrade which comes along with postgres package. Format will be as below

<target_version_pg_upgrade> \
    -b <source_version_binary_dir> \
    -B <target_version_binary_dir> \
    -d <source_version_config_dir> \
    -D <source_version_config_dir>
Binary & config directory paths required to fulfill the above requirements
To get the above requirement run the below command 
systemctl status postgresql@9.6-main
  
systemctl status postgresql@12-main
 

After getting informations, stop both the clusters on the server i.e 9.6 & 12
sudo pg_ctlcluster 9.6 main stop
sudo pg_ctlcluster 12 main stop

Now Run Upgrade

$ su – postgres
postgres@istance:~$ /usr/lib/postgresql/12/bin/pg_upgrade \
    -b /usr/lib/postgresql/9.6/bin \
    -B /usr/lib/postgresql/12/bin \
    -d /etc/postgresql/9.6/main \
    -D /etc/postgresql/12/main \
    --verbose

Data migration will take some time depends on database size.
Once migration gets complete, do the following 

Shutting down the old cluster, making the new cluster as the default
Switch ports
9.6 is currently configured to run on port 5432 and 12 in 5433. This need to be switched:
$ vi /etc/postgresql/9.6/main/postgresql.conf
port = 5432 # <-- change to 5433
$ vim /etc/postgresql/12/main/postgresql.conf
port = 5433 # <-- change to 5432
Verify changes:
$ grep -H '^port' /etc/postgresql/*/main/postgresql.conf
/etc/postgresql/12/main/postgresql.conf:port = 5432
/etc/postgresql/9.6/main/postgresql.conf:port = 5433
Start the new cluster
At this point both clusters are not running. We can start version 12 alone.
sudo pg_ctlcluster 12 main start 
OR
systemctl start postgresql@12-main

Run pg_clusters again to check the status
$ pg_lsclusters
 

The migration has completed, verify whether psql connects to the version 12 cluster by default:
$ su - postgres
postgres@istance:~$ psql
psql (12.2 (Ubuntu 12.2-2.pgdg18.04+1))
Type "help" for help.

postgres=#

NOTE: Do not remove old cluster immediately after the upgrade, first check if applications are getting connected and working as expected then after 2-3days remove the old cluster.

