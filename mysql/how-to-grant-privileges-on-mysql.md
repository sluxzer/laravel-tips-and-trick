# How to Grant All Privileges on a Database in MySQL

To begin editing privileges in MySQL, you must first login to your server and then connect to the mysql client. Typically you’ll want to connect with root or whichever account is your primary, initial ‘super user’ account that has full access throughout the entire MySQL installation.

Typically the root user will have been assigned an authentication password when MySQL was installed, but if that is not the case, you should take steps to up your security by adding root passwords as illustrated in the official documentation.

## Connecting to the MySQL Command-Line Tool
For this example, we’ll assume root is the primary MySQL account. To begin using the MySQL Command-Line Tool (mysqlcli), connect to your server as the root user, then issue the mysql command:

```
$ mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 112813
Server version: 5.5.43-0ubuntu0.14.04.1 (Ubuntu)
[...]
mysql>
If successful, you’ll see some output about your MySQL connection and be facing down the mysql prompt.
```

Note: In the event that you’re unable to connect directly to the server as the root user before connecting to mysql, you can specify the user you wish to connect as by adding the --user= flag:

```	
$ mysql --user=username
```

## Granting Privileges
Now that you are at the mysqlcli prompt, you need only issue the GRANT command with the necessary options to apply the appropriate permissions.

### Privilege Types
The GRANT command is capable of applying a wide variety of privileges, everything from the ability to CREATE tables and databases, read or write FILES, and even SHUTDOWN the server. There are a wide range of flags and options available to the command, so you may wish to familiarize yourself with what GRANT can actually do by browsing through the official documentation.

### Database-Specific Privileges
In most cases, you’ll be granting privileges to MySQL users based on the particular database that account should have access to. It is common practice, for example, for each unique MySQL database on a server to have its own unique user associated with it, such that only one single user has authentication access to one single database and vice-versa.

To GRANT ALL privileges to a user, allowing that user full control over a specific database, use the following syntax:

```
mysql> GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';
```

With that command, we’ve told MySQL to:

GRANT the PRIVILEGES of type ALL (thus everything of course). Note: Most modern MySQL installations do not require the optional PRIVILEGES keyword.
These privileges are for database_name and it applies to all tables of that database, which is indicated by the .* that follows.
These privileges are assigned to username when that username is connected through locally, as specified by @'localhost'. To specify any valid host, replace 'localhost' with '%'.
Rather than providing all privileges to the entire database, perhaps you want to give the tolkien user only the ability to read data (SELECT) from the authors table of the books database. That would be easily accomplished like so:

```
mysql> GRANT ALL PRIVILEGES ON books.authors  TO 'tolkien'@'localhost';
```

### Creating Another Super User
While not particularly secure, in some cases you may wish to create another ‘super user’, that has ALL privileges across ALL databases on the server. That can be performed similar to above, but by replacing the database_name with the wildcard asterisk:

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'tolkien'@'%';
```

Now tolkien has the same privileges as the default root account, beware!

### Saving Your Changes
As a final step following any updates to the user privileges, be sure to save the changes by issuing the FLUSH PRIVILEGES command from the mysql prompt:

```
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
```
`
### Update User Password
```
mysql> ALTER USER 'local_user'@'localhost' IDENTIFIED BY 'new_password';
```

### Create new Database
```
CREATE DATABASE `dpmptsp_tangsel_staging`
```

### Add Privileges to Spesific Database and Spesific User
```
GRANT ALL PRIVILEGES ON dpmptsp_tangsel_staging.* TO 'user_dpmptsp'@'localhost';
```

Ref:
https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql
