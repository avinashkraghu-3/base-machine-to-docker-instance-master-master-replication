1). Base machine steps 

Update repository → apt-get update

Install mysql in base machine -- > root@ip-10-0-0-231:~# apt-get install mysql-server

2)install docker engine in Ubuntu

#sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    Lsb-release

#Add Docker’s official GPG key:
root@ip-10-0-0-231:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
root@ip-10-0-0-231:~# 


root@ip-10-0-0-231:~# echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
###Install Docker-engine###

Update apt package index

#sudo apt-get update
#sudo apt-get install docker-ce docker-ce-cli containerd.io

###add user to docker group###
root@docker:~# adduser docker-user
Adding user `docker-user' ...
Adding new group `docker-user' (1001) ...
Adding new user `docker-user' (1001) with group `docker-user' ...
Creating home directory `/home/docker-user' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for docker-user
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] 
root@docker:~# 
root@docker:~# usermod -aG docker docker-user 
root@docker:~# 

root@docker:~# id docker-user 
uid=1001(docker-user) gid=1001(docker-user) groups=1001(docker-user),998(docker)
root@docker:~# 

###backup the mysqld.cnf file###
root@docker:/etc/mysql# ls
conf.d        debian.cnf  my.cnf.fallback  mysql.conf.d
debian-start  my.cnf      mysql.cnf
root@docker:/etc/mysql# cd mysql.conf.d/
root@docker:/etc/mysql/mysql.conf.d# ls
mysql.cnf  mysqld.cnf
root@docker:/etc/mysql/mysql.conf.d# cp -pv mysqld.cnf mysqld.cnf_backup
'mysqld.cnf' -> 'mysqld.cnf_backup'
root@docker:/etc/mysql/mysql.conf.d# 

###open mysqld.cnf
root@docker:/etc/mysql/mysql.conf.d# vim mysqld.cnf
Then change the bind address to host private ip address then restart the mysql

bind-address            = 10.0.0.231 → private ip address of host machine

# Server ID of the Master
skip_name_resolve=1
server-id = 1
replicate-same-server-id = 0
auto-increment-increment = 3
auto-increment-offset = 1
replicate-do-db = avi_db
binlog-do-db = avi_db
log_bin = /var/log/mysql/mysql-bin.log
### appending the following code to mysqld.cnf of Base machine
Then Restart the mysql service using following command.

root@docker:/etc/mysql/mysql.conf.d# systemctl restart mysql

###Then check the status of mysql###
root@docker:/etc/mysql/mysql.conf.d# systemctl status mysql
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset:>
     Active: active (running) since Tue 2021-07-20 05:01:02 UTC; 18s ago
    Process: 5313 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=e>
   Main PID: 5336 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 2368)
     Memory: 336.0M
     CGroup: /system.slice/mysql.service
             └─5336 /usr/sbin/mysqld

Jul 20 05:01:01 docker systemd[1]: mysql.service: Succeeded.
Jul 20 05:01:01 docker systemd[1]: Stopped MySQL Community Server.
Jul 20 05:01:01 docker systemd[1]: Starting MySQL Community Server...
Jul 20 05:01:02 docker systemd[1]: Started MySQL Community Server.
root@docker:/etc/mysql/mysql.conf.d# 

### Then switch to ubuntu
root@docker:/etc/mysql/mysql.conf.d# su - ubuntu 
ubuntu@docker:~$ 

###install mysql_secure_installation


root@docker:/etc/mysql/mysql.conf.d# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.25-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql>  FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql>  exit;
Bye
root@docker:/etc/mysql/mysql.conf.d#

###Restart mysql service
root@docker:/etc/mysql/mysql.conf.d# systemctl restart mysql

### set root password of mysql
ubuntu@docker:~$ mysql_secure_installation
Securing the MySQL server deployment.
Connecting to MySQL using a blank password.
ubuntu@docker:~$ mysql_secure_installation 

Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
Please set the password for root here.

New password: 

Re-enter new password: 

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : Y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
Success.

All done! 
ubuntu@docker:~$ 

###Create Replica user 

mysql> CREATE USER 'replica'@'%' identified by 'Qwerty@123';
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql>  show grants for 'replica'@'%';
+-------------------------------------------------+
| Grants for replica@%                            |
+-------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO `replica`@`%` |
+-------------------------------------------------+
1 row in set (0.00 sec)

mysql> 

###change the Plugin of replica user to native_password
mysql> ALTER USER 'replica'@'%' IDENTIFIED WITH mysql_native_password BY 'Qwerty@123';
Query OK, 0 rows affected (0.01 sec)

mysql> 

###create database
mysql> create database avi_db;
Query OK, 1 row affected (0.01 sec)

mysql> 



####configuration for docker instance 1####


###switch to Docker user
ubuntu@docker:~$ su - docker-user 
Password: 
docker-user@docker:~$ 

docker-user@docker:~$ ls
docke
docker-user@docker:~$

docker-user@docker:~/docke$ ls
conf  docker-compose  docker-compose.yml
docker-user@docker:~/docke$

docker-user@docker:~/docke$ ./docker-compose up -d
WARNING: Some services (mysqlmaster2) use the 'deploy' key, which will be ignored. Compose does not support 'deploy' configuration - use `docker stack deploy` to deploy to a swarm.
Creating network "docke_mysql-ms-network" with driver "bridge"
Creating network "docke_default" with the default driver
Pulling mysqlmaster1 (mysql:latest)...
latest: Pulling from library/mysql
b4d181a07f80: Pulling fs layer
b4d181a07f80: Pull complete
a462b60610f5: Pull complete
578fafb77ab8: Pull complete
524046006037: Pull complete
d0cbe54c8855: Pull complete
aa18e05cc46d: Pull complete
fd6f649b1d0a: Pull complete
2a97d48c2fdc: Pull complete
30f0c7db48fc: Pull complete
f5dda8df049e: Pull complete
671b83fd7448: Pull complete
5d9cc55fa997: Pull complete
Digest: sha256:18d8d109aa64673c78aebfb845b929cfdac97a553332f4310f4de8d67ceb03d2
Status: Downloaded newer image for mysql:latest
Creating mysqlmaster1 ... done
Creating mysqlmaster2 ... done
Creating docke_runner_1 ... done
docker-user@docker:~/docke$ 

### using docker ps command to show the docker running instances
docker-user@docker:~/docke$ docker ps 
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                                  NAMES
c09733b72232   mysql:latest   "docker-entrypoint.s…"   22 seconds ago   Up 20 seconds   3306/tcp, 33060/tcp                                    docke_runner_1
f634392f5a1f   mysql:latest   "docker-entrypoint.s…"   25 seconds ago   Up 21 seconds   33060/tcp, 0.0.0.0:3308->3306/tcp, :::3308->3306/tcp   mysqlmaster2
a14d23a41fb4   mysql:latest   "docker-entrypoint.s…"   25 seconds ago   Up 21 seconds   33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp   mysqlmaster1
docker-user@docker:~/docke$ 

###use this command to login to mysqlmaster1 server
docker-user@docker:~/docke$ docker exec -it mysqlmaster1 /bin/bash
root@a14d23a41fb4:/# 
root@a14d23a41fb4:/# mysql -u root -p
Password:

mysql> SELECT user,host FROM mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| replica          | %         |
| root             | %         |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
6 rows in set (0.00 sec)

mysql> 

mysql>  show grants for 'replica'@'%';
+------------------------------------------------------+
| Grants for replica@%                                 |
+------------------------------------------------------+
| GRANT USAGE ON *.* TO `replica`@`%`                  |
| GRANT ALL PRIVILEGES ON `avi\_db`.* TO `replica`@`%` |
+------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> 

###Set to GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%'; 

mysql> GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> 

mysql>  show grants for 'replica'@'%';
+------------------------------------------------------+
| Grants for replica@%                                 |
+------------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO `replica`@`%`      |
| GRANT ALL PRIVILEGES ON `avi\_db`.* TO `replica`@`%` |
+------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> 


mysql> ALTER USER 'replica'@'%' IDENTIFIED WITH mysql_native_password BY 'Qwerty@123';
Query OK, 0 rows affected (0.00 sec)

mysql> 

###The same procedure do another docker instance


####in base machine 
Execute following command 

mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |     1463 | avi_db       |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>

Then execute following command in docker instance 1

mysql> stop slave;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> CHANGE MASTER TO MASTER_HOST = '10.0.0.231', MASTER_USER = 'replica', MASTER_PASSWORD = 'Qwerty@123', MASTER_LOG_FILE = 'mysql-bin.000002', MASTER_LOG_POS = 1463;
Query OK, 0 rows affected, 8 warnings (0.04 sec)

mysql>  start slave;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 10.0.0.231
                  Master_User: replica
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 1463
               Relay_Log_File: a14d23a41fb4-relay-bin.000002
                Relay_Log_Pos: 324
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: avi_db
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1463
              Relay_Log_Space: 540
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 3ae58892-e913-11eb-82fb-06f7463a77c8
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set, 1 warning (0.01 sec)

mysql> 



###use this command to find out the instance ip address
docker-user@docker:~/docke$ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysqlmaster1
172.25.0.101
docker-user@docker:~/docke$


Docker instance (mysqlmaster1 to mysqlmaster2)
### execute this command on docker instance 1

mysql> SHOW MASTER STATUS;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000002 |      674 | avi_db       |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>

###then execute in mysqlmaster2
 
mysql> stop slave;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST = '172.25.0.101', MASTER_USER = 'replica', MASTER_PASSWORD = 'Qwerty@123', MASTER_LOG_FILE = 'mysql-bin.000002', MASTER_LOG_POS = 674;
Query OK, 0 rows affected, 8 warnings (0.03 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> 


mysql>  show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 172.25.0.101
                  Master_User: replica
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000002
          Read_Master_Log_Pos: 674
               Relay_Log_File: f634392f5a1f-relay-bin.000002
                Relay_Log_Pos: 321
        Relay_Master_Log_File: binlog.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: avi_db
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 674
              Relay_Log_Space: 537
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 2
                  Master_UUID: 75120249-e919-11eb-a9ad-0242ac190065
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set, 1 warning (0.00 sec)

mysql> 


Then server 3 to server 1 (base machine)

###in  Docker instance 3 (execute this command)

mysql> SHOW MASTER STATUS;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000003 |      156 | avi_db       |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> 


mysql> stop slave;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> CHANGE MASTER TO MASTER_HOST = '172.25.0.102', MASTER_USER = 'replica', MASTER_PASSWORD = 'Qwerty@123', MASTER_LOG_FILE = 'binlog.000003', MASTER_LOG_POS = 156;
Query OK, 0 rows affected, 8 warnings (0.08 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.01 sec)



mysql>  show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.25.0.102
                  Master_User: replica
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000003
          Read_Master_Log_Pos: 156
               Relay_Log_File: docker-relay-bin.000002
                Relay_Log_Pos: 321
        Relay_Master_Log_File: binlog.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: avi_db
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 156
              Relay_Log_Space: 531
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 3
                  Master_UUID: 751327de-e919-11eb-9dec-0242ac190066
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set, 1 warning (0.01 sec)

mysql> 

