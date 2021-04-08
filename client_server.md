## Client-Server Architecture with MySQL

The following demonstrates a quick example of a basic client-Server communication, this time in Mysql.

1. Create and configure two Linux-based Virtual Machine (Here I'm using Oracle VirtualBox to spin them up. I'd be name my mysql-server as *Shelly* and the mysql-client *Remoteful*).
   
2. On Shelly install MySQL Server software, using:

`sudo apt install mysql-server`

After the installation, you can check if to see if the mysqld (mysql daemon) is running. 

`sudo systemctl status mysql`

```
Output:

● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: en>
     Active: active (running) since Thu 2021-04-08 18:32:47 WAT; 51s ago
   Main PID: 5430 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 3355)
     Memory: 331.6M
     CGroup: /system.slice/mysql.service
             └─5430 /usr/sbin/mysqld

Apr 08 18:32:46 molokofi systemd[1]: Starting MySQL Community Server...
Apr 08 18:32:47 molokofi systemd[1]: Started MySQL Community Server.

```

3. Likewise on the mysql-client i.e Remoteful, install the Mysql client package

`sudo apt install mysql-client`

After the installation process you can check the version and also see if the mysql is running as a process.

`$ mysql --version`

```
Output:

mysql  Ver 8.0.23-0ubuntu0.20.04.1 for Linux on x86_64 ((Ubuntu))
```

`$ ps aux | grep mysql`

```
Output:

  4378  0.0  0.0   8908   732 pts/0    S+   20:02   0:00 grep --color=auto mysql

```

The Ubuntu mysql-client package includes following command line tools (and more):

- mysql - the mysql command-line client to run SQL statements.
- mysqladmin - client for administering a MySQL server.
- mysqldump - a database backup program. The mysqldump command writes the contents of database tables into text files which you can use to restore databases.
- mysqlreport - Makes a friendly report of important MySQL status values.
- mysqlcheck - a command line client to check, repair, and optimize tables.


4. By default MySQL server uses TCP port 3306. Now, we'd use mysql server's local IP address to connect from mysql client. Which means so to creat a new  ‘Inbound rules’ in ‘mysql server (Shelly)’. For extra security, we'd only allow access to the specific local IP address of the ‘mysql client (Remoteful)’.

`$ sudo ufw allow from 192.168.x.x/24 to any port 3306`

```
Output:

WARN: Rule changed after normalization
Rule added
```

You can verify that this new rule has been created by running:

`$ sudo ufw status`

```
Output:

Status: active
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
3306                       ALLOW       192.168.x.x/24            
OpenSSH (v6)               ALLOW       Anywhere (v6) 

```

5. Now, you can further secure the mysql-server by running:

`sudo mysql_secure_installation`

You can mostly answer "y" for the subsequent prompts. Afterwards, login to your mysql server with:

`sudo mysql`

6. Now create a user and a db, and grant the new user every privilege to the newly created db, with following commands

`mysql> CREATE USER 'remoteful'@'%' IDENTIFIED BY 's***y_b88';`

`mysql> CREATE DATABASE shelly_db;`

`mysql> GRANT ALL PRIVILEGES ON shelly_db.* TO 'remoteful'@'%';`

7. You'd also need to configure MySQL server to allow connections from remote hosts.
   
`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf `

In the configuration file. replace ‘127.0.0.1’ to ‘0.0.0.0’ like this:

*bind image

Do well to restart the mysql server after this adjustment

`sudo systemctl restart mysql`

8. It's now time to connect mysql client  remotely to mysql server Database Engine, without using SSH. but with the mysql utility.

`$ sudo mysql -u remoteful -h 192.168.1.xx -p`


```
Output:

Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.23-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

9. Now you're finally connected to your server from your client. You can run some basic command and bask in your success!

`mysql> SHOW DATABASES;`

```
Output:

+--------------------+
| Database           |
+--------------------+
| information_schema |
| shelly_db          |
+--------------------+
2 rows in set (0.01 sec)
```

- You can Show current logged users. Which would list all users that are currently logged in the MySQL database server, by executing the following statement:

`mysql> SELECT 
    -> user, 
    -> host, 
    -> db, 
    -> command 
    -> FROM 
    -> information_schema.processlist;`

```
Output:

+-----------+---------------------+------+---------+
| user      | host                | db   | command |
+-----------+---------------------+------+---------+
| remoteful | 192.168.1.191:44632 | NULL | Query   |
+-----------+---------------------+------+---------+
1 row in set (0.00 sec)
```

- You can also check the size of our created database, by running:

`mysql> SELECT 
    ->     table_schema 'Database Name',
    ->     SUM(data_length + index_length) 'Size in Bytes',
    ->     ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) 'Size in MiB'
    -> FROM information_schema.tables 
    -> WHERE table_schema = 'shelly_db';`

```
Output:

+---------------+---------------+-------------+
| Database Name | Size in Bytes | Size in MiB |
+---------------+---------------+-------------+
| NULL          |          NULL |        NULL |
+---------------+---------------+-------------+
1 row in set (0.01 sec)
```

Congratulations! As you've just successfully deloyed a fully functional MySQL Client-Server setup.