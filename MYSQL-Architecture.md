## SERVER AND CLIENT MYSQL DATABASE ARCHITECTUTRE IMPLEMENTATION
In this exercise, I will working you through how to set up MYSQL database architecture. In this practical work, we will use use two Amazon Elastic Cloud Compute (EC2). One will servers as Our database server an the other as the client. Before getting started, lets discuss a little about database mangement system.

![MYSQL DATABASE ARCHITECTUTRE](images/mysql-client-server.png)

## MYSQL SERVER
MySQL server is the central software program that manages the database objects, data, etc. The MySQL Server(mysqld) listens on a specific network port for incoming connection requests from the clients. The clients can connect to the server to access or modify data in the database. The default port of the MySQL Server is 3306.

<h4>MySQL Server Architecture</h3>

Connection Handling: The server manages incoming client connections, authenticating users, and establishing communication channels.

SQL Interface and Parser: When a client sends an SQL query, the server's SQL interface and parser process the query, breaking it down into a structured format for further processing.

Optimizer: The query optimizer evaluates the query and generates an optimal execution plan, which outlines how to retrieve or modify the data efficiently.

Execution Engine: The execution engine carries out the execution plan. It interacts with the storage engine to retrieve or modify data based on the query.

Storage Engine: MySQL supports different storage engines (e.g., InnoDB, MyISAM). The storage engine is responsible for managing the physical storage of data, including reading/writing to disk and maintaining indexes.

Buffer Pool: Frequently accessed data is cached in the buffer pool, which is a portion of memory. This reduces the need to access data from disk, improving query performance.

Transaction Management: The server ensures transactions follow the ACID properties (Atomicity, Consistency, Isolation, Durability) to maintain data integrity. It manages locks and handles concurrent access.

Logs: MySQL maintains various logs, including binary logs (for replication), error logs (for debugging), and transaction logs (for recovery).

## MySQL Client
MySQL clients are applications that sends requests to the database server. MySQL clients are software applications that connect to the server to access and manipulate the data in the database. In order for the clients to access data, the MySQL server must be running.

The client connects to the server, sends requests to the server. The server handles the requests and processes the requests. The client receives the response and disconnects from the server. MySQL Server can handle multiple database client connections at once.

MySQL clients can be

Local – local clients run on the same machine where the MySQL Server runs.
Remote – remote clients run on a different remote machine.

<h4>MySQL Server Architecture</h3>

User Interaction: The client application allows users to interact with the MySQL server. This can be through a command-line interface, a graphical tool, or a programmatic library.

Query Generation: Users or applications generate SQL queries using the client. These queries are structured commands that define actions like data retrieval, insertion, updates, or deletions.

Connection Handling: The client establishes a connection to the MySQL server using the provided hostname, port, username, and password. The connection is a network link that facilitates communication.

Sending Queries: The client sends SQL queries to the server over the established connection. These queries are processed by the server's SQL interface and other components.

Receiving Results: The server processes the query and sends back results to the client. The client then presents these results to the user or application.

Error Handling: If there are any issues with the query or the server's response, the client handles error messages and notifications, providing feedback to the user.

## IMPLEMENTATION STAGE
<h2>SERVER SET UP</h2>
In this step, we will lanuch two EC2 intances, Database_Server and Database_client with SSH access on port 22.

![MYSQL DATABASE ARCHITECTUTRE](images/instances.jpg)

Then, we sign into the database_server through SSH
![MYSQL DATABASE ARCHITECTUTRE](images/ssh.jpg)

Follow by updating and upgrading the machine

    sudo apt update && sudo apt upgrade -y

![MYSQL DATABASE ARCHITECTUTRE](images/update.jpg)

Next, we install mysql-server

    Sudo apt install mysql-server -y
    
![MYSQL DATABASE ARCHITECTUTRE](images/mysql-server.jpg)

Now, we configure the mysql server application with the line below

    sudo mysql_secure_installation

For the purpose of this exercise, we not use a secure security password, we'll allow the default setting by inputting any other letter aside 'y'
![MYSQL DATABASE ARCHITECTUTRE](images/no.jpg)

Afterward, we will select yes 'y' for every other configuration prompt.

![MYSQL DATABASE ARCHITECTUTRE](images/yes.jpg)

After the installation and configuration settings of MYSQL-serevr, we then navigate into the mysql console by typing the command below

    Sudo mysql

![MYSQL DATABASE ARCHITECTUTRE](images/sudo-mysql.jpg)

Since we are in the mysql console, we will create a database 'employees_data'

     mysql> CREATE DATABASE employees_data;

![MYSQL DATABASE ARCHITECTUTRE](images/create.jpg)

To display available databases:

    mysql> SHOW DATABASES;

![MYSQL DATABASE ARCHITECTUTRE](images/showdb.jpg)

Create 

Follow by creating a table 'employees_list' in the employees_data.

    mysql> CREATE TABLE employees_data.employees_list (item_id INT AUTO_INCREMENT, Name VARCHAR(255), Email varchar(100) NOT NULL, Phone varchar(11) NOT NULL, PRIMARY KEY(item_id) );

![MYSQL DATABASE ARCHITECTUTRE](images/list.jpg)

After adding the table the database, we will some data into the table

    mysql> INSERT INTO employees_data.employees_list VALUES (1, 'David . A', 'david@gmail.com', '08146548610');

![MYSQL DATABASE ARCHITECTUTRE](images/david.jpg)

Second data:

    mysql> INSERT INTO employees_data.employees_list VALUES (2,'Isael . A', 'israel@gmail.com', '07068974570');

![MYSQL DATABASE ARCHITECTUTRE](images/israel.jpg)

To view all data in the 'employees_list' table;

    mysql> SELECT * FROM employees_data.employees_list;

![MYSQL DATABASE ARCHITECTUTRE](images/listtb.jpg)

After completing all of these steps, we now need to create a remote user who can sign in from the client machine. To do this, copy and paste the command below:

    mysql> CREATE USER 'israel'@'%' IDENTIFIED BY 'password';

The '%' symbol in the CREATE USER command indicates from any IP address. Additionally, a specific IP address or CIDR can be assigned.

![MYSQL DATABASE ARCHITECTUTRE](images/user.jpg)

After the user account has been created, we have to grant neccessary privilleges to the user account.

    mysql> GRANT ALL PRIVILEGES ON employees_data.* TO 'israel'@'%' WITH GRANT OPTION;

After granting privileges, you need to apply the changes.

    mysql> FLUSH PRIVILEGES;

You can exit the MySQL console by typing:

    mysql> exit;

![MYSQL DATABASE ARCHITECTUTRE](images/exit.jpg)

The last two actions to make database accessible is the edit mysql file to allow traffic from outside to localhost. to do this,

    sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

In this file, you local the <b>Bind Address  = 127.0.0.1</b> and change it to <b>Bind Address =  0.0.0.0</b> <br>Save and close

![MYSQL DATABASE ARCHITECTUTRE](images/cnf.jpg)

The second last action is to allow incoming connections on the MySQL port 3306 to allow remote access in the database_server EC2, via the security inbound rules.

![MYSQL DATABASE ARCHITECTUTRE](images/sg.jpg)

Restart the MySQL service to apply the changes you made to the configuration file.

    sudo systemctl restart mysql

<br>

<h2>CLIENT MACHINE SET UP</h2>

From another terminal, sign into the database_client machine through SSH

![MYSQL DATABASE ARCHITECTUTRE](images/cssh.jpg)

Update and upgrade the machine at once;

    sudo apt update && sudo apt upgrade -y

![MYSQL DATABASE ARCHITECTUTRE](images/cup.jpg)

Then, we install mysql-client

    sudo apt install mysql-client

![MYSQL DATABASE ARCHITECTUTRE](images/mysql-client.jpg)


Now, it is time to sign into the user account we created from the database_server machine:

     mysql -u israel -h 172.31.45.182 -p;

IP address 172.31.45.182 is over database_server ip address

![MYSQL DATABASE ARCHITECTUTRE](images/remote.jpg)

From the remote user, display available databases with the command below:

     mysql> SHOW DATABAES;

![MYSQL DATABASE ARCHITECTUTRE](images/showdbc.jpg)

Show data in the employees_list table with this:

     mysql> SELECT * FROM employees_data.employees_list;

![MYSQL DATABASE ARCHITECTUTRE](images/ctable.jpg)

## CONCLUSION
If you are able to follow this steps and you are able to display the available databases and employees_list table content, I am happy to tell you that you have successfully set a mysql database architecture. <br>
Do not hesitate to play around other mysql functions like Inset, retrieve, Update, and Delete data.
