# DIO-Project2
## DareDevOps Project2
This project involves the use another web server- Nginx on an EC2 instance in AWS CLoud.  First we start with the installation of Nginx by updating the server's package since this is the first time it is being installed.

```bash
   sudo apt update
   sudo apt install nginx
```
To verify that nginx was successfully installed and is running as a service in Ubuntu, run:
```bash
   sudo systemctl status nginx
```
The output of this command confirms the nginx is running: 
![nginx status](https://user-images.githubusercontent.com/61512079/172841408-33310a93-37f0-4a87-bf49-6c77a4d05d41.PNG> "nginx status")

The nginx webserver is opened for web access on the AWS EC2 instance and confirmed working as shown below:

[Nginx Web Access](https://user-images.githubusercontent.com/61512079/172843244-5f92f98c-7a85-4ea8-9b66-c69e327bf76b.PNG "Nginx-Web-Access")

 Next is the installation of Database Management System (DBMS) to be able to store and manage data for the site in a relational database.
 In this project, the DBMS we installed  is MySQL within PHP environment.
 The package was installed with the command below:
 ```bash
    $ sudo apt install mysql-server
 ```
 After installation,  log in as root to confirm you are to log on to the mysql.
 ```bash
    $ sudo mysql
 ```
 From the mysql prompt, the default security settings were removed and a separate non-root account was create for the DB access.
 ```mysql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
    exit
 ```
 ```bash
    $ sudo mysql_secure_installation
    $ sudo mysql -p
 ```

 [MySQL Security](https://user-images.githubusercontent.com/61512079/172844981-2f50723b-5b0b-471b-b915-11a5ba821ef9.PNG "MYSQL Security Settings Modification COnfirmed")
 
 [New User creation](https://user-images.githubusercontent.com/61512079/172845249-8b144fa7-6b28-46d6-9b3a-9c7278e69d49.PNG "Non-root User Created")
 
  
Next is the installation of PHP to process code and generate dynamic content for the web server.
To install PHP, you need to install two additional packages:
1.php-fpm, *which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing.*
1.php-mysql, *a PHP module that allows PHP to communicate with MySQL-based databases.*

Once the two pakcages mentioned are installed, Core PHP packages will automatically be installed as dependencies.
```bash
$sudo apt install php-fpm php-mysql
```
After the PHP is installed, we created another sub-directory inside the default web folder: var/www

```bash
   $sudo mkdir /var/www/projectLEMP
```
Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:

```bash
   $sudo chown -R $USER:$USER /var/www/projectLEMP
```

Then, a new configuration file was created in Nginx’s sites-available directory
```bash
   $sudo nano /etc/nginx/sites-available/projectLEMP
```
And the script below is pasted into the file:

```php
 #/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```



The configuration is activated by linking to the config file from Nginx’s sites-enabled directory:
```bash
   $sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```
This will tell Nginx to use the configuration next time it is reloaded. THe configuration was tested for syntax errors by typing:
```bash
   $sudo nginx -t
```

We also disabled the default Nginx host that is currently configured to listen on port 80 by running the command below:

```bash
   sudo unlink /etc/nginx/sites-enabled/default
```
After, reload Nginx to apply the changes:
```bash
sudo systemctl reload nginx
```
The new website is now active after the previous steps but the web root /var/www/projectLEMP is still empty. 
Create an index.html file in that location so that we can test that your new server block works as expected. Copy some text inside the file as shown in below command:
```bash
   sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s     http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
The LEMP stack is now fully configured and the page is test by opening the link from a web browser.

[Site test](https://user-images.githubusercontent.com/61512079/172852837-442f9fd8-2dcb-4bd5-a5f4-26d559e61aa4.PNG "Site Test")

Next the we create a PHP script to validate that Nginx can correctly hand .php files off to your PHP processor.
THis is done by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:
```bash
   $sudo nano /var/www/projectLEMP/info.php
```

Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:
```php
<?php
phpinfo();
```
The page is now accessed in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:

```http
http://`server_domain_or_IP`/info.php
```
Out of the page test is shown below:
[Page test](https://user-images.githubusercontent.com/61512079/172854130-8a8ff87e-6ce1-4088-a0a4-9d862d032120.PNG "PHP Page Test")

After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:
```bash
   $sudo rm /var/www/your_domain/info.php
```
Finally, a  we create a test to retrieve data from mysql database with PHP. To do this  We create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

We created a database named **example_database** and a user named **example_user** from root access to the SQL.
```bash
   $sudo mysql
```
```mysql
   CREATE DATABASE `example_database`;
   CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY '<yourdefinedpassword>';
   GRANT ALL ON example_database.* TO 'example_user'@'%';
   exit
```
Test the new user access to the created database by logging on to the DB as follow:
```bash
   $mysql -u example_user -p
```
After accessing the database, you can display it by running the command below:
```mysql
  SHOW DATABASES;
```
Creat a test table called **todo_list** inside the DB:
```mysql
   CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT, content VARCHAR(255), PRIMARY KEY(item_id));
   INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```
   
To confirm that the data was successfully saved to your table, run:
```mysql
   SELECT * FROM example_database.todo_list;
```
The output of the listing is shown below:
[DB List](https://user-images.githubusercontent.com/61512079/172857673-64271233-a4bb-4ec6-91e6-6d4028f0fbae.PNG "Output of Todo_List table")

A PHP script that will connect to MySQL and query for your content was created the  custom web root directory:
```bash
   nano /var/www/projectLEMP/todo_list.php
```
The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. 

The PHP script below was copied into the todo_list.php:

```php
   ?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
The page is now tested on the web browser by using the path below:

```http
http://<Public_domain_or_IP>/todo_list.php
```
Output of the test is shown below:

[PHP Test](https://user-images.githubusercontent.com/61512079/172859198-76b883d9-2749-44f6-ada0-e5dac09b636f.PNG "PHP Test")







