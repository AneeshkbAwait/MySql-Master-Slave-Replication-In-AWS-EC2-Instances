# MySql-Master-Slave-Replication-In-AWS-EC2-Instances

 The master-slave replication process enables database administrators to replicate or copy data stored in more than one server simultaneously. This helps the database administrator to create a live backup of the database all the time.

#### Requirements:

I suggest you to use the privite IP for the server communication ( We are using AWS ec2 instance and the privite IP wll communicate here properly ). Also I suggest you to enable the port 3306 on your server security firewall.

Here, we are using 3 servers for configuring the Master-Slave-Slave. The servers in this example have the following IPs:

Master IP: 172.31.12.78 Slave-1 IP: 172.31.12.23 Slave–2 IP: 172.31.19.123

#### Configure the Master Server

Open the MySQL configuration file and add the following lines in the [mysqld] section:
```
vi /etc/my.cnf

bind-address           = 0.0.0.0
server-id              = 1
log_bin                = mysql-bin
```

Then restart the MySQL service for changes to take effect;
```
systemctl restart mysqld
```

The next step is to create a new replication user. Login to MYSQL server as root and create user. Please use the followng SQL queries that will create the user and grant the REPLICATION SLAVE privilege to the user:
```
mysql> CREATE USER 'relication_user'@'%' IDENTIFIED BY 'strong_password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'relication_user'@'%';
mysql> FLUSHPRIVILEGES;
```

Verify the status of the master;
```
mysql> SHOW MASTER STATUS\G
Output
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 679
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
Take note of file name, ‘mysql-bin.000001’ and Position ‘679’. You’ll need these values when configuring the slave server. 
```
Take note of file name, ‘mysql-bin.000001’ and Position ‘679’. You’ll need these values when configuring the slave server. These values will probably be different on your server.

#### Configure the Slave-1 and Slave-2 Server
Here we have 2 Slave servers and we want to connect 2 Slave servers to the Master. Open the slave-1 and slave-2 server MySQL configuration file and edit the following lines:

Slave-1;
```
vi  /etc/my.cnf

bind-address           = 0.0.0.0
server-id              = 2
log_bin                = mysql-bin
```

Slave-2;
```
vi  /etc/my.cnf

bind-address           = 0.0.0.0
server-id              = 3
log_bin                = mysql-bin
```
Then Restart MySQL service in the slave-1 and slave-2 ( both ) server:
```
systemctl restart mysqld
```
The next step is to configure the parameters that the slave server will use to connect to the master server. First Login to the 2 slave server MySQL shell and run the commands in the both server.
```
mysql -u root -p
```
First, stop the slave threads and run the following query that will set up the slave to replicate the master:
```
mysql> STOP SLAVE;
mysql> CHANGE MASTER TO MASTER_HOST='72.31.12.78', MASTER_USER='relication_user', MASTER_PASSWORD='strong_password', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=679;
```
Please confirm you are using the correct IP address, user name, and password. The log file name and position must be the same as the values you obtained from the master server.
Once done, start the slave threads.
```
mysql> START SLAVE;
mysql> SHOW SLAVE STATUS\G;
output
MariaDB [(none)]> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 72.31.12.78
                  Master_User: relication_user
                  Master_Port: 3306
                Connect_Retry: 60
```
#### Test the Configuration

To verify that everything works as expected, I have created a new database on the master server and checked both slave servers, and confirmed the database was automatically created in the Slave servers. You can follow the command to check the database is created or not in the slave server.
```
Mysql -u root -p 
mysql>  SHOW DATABASES;
```
You can see that database in the Slave-1 and Slave-2 server. Mysql Master Slave replication completed.

In this tutorial, I have shown you set up a Master-Slave-Slave Replication using AWS EC2 Instance. This is the easier way to configure the Master-Slave-Slave Replication
