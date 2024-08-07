# LEMP STACK PROJECT


A “LEMP” stack is a set of open-source software that is typically installed together to enable a server to host dynamic websites and web apps written in PHP. LAMP is an acronym that represents the Linux operating system with the Nginx web server. The site data is stored in a MySQL database, and dynamic content is processed by PHP.

## How to setup a LEMP stack on an Ubuntu 22.04 server 

I created a project called LEMP Stack Project to display a virtual host that served content stored in a MySQL database and processed by PHP

### Prerequisites: Ubuntu 22.04 server with a non-root sudo-enabled user account

### Step 1: Installing Nginx and Updated the Firewall
- First updated package manager cache - `sudo apt update`
- Then, installed Nginx - `sudo apt install nginx`
- Adjusted firewall settings to allow HTTP traffic using Ubuntu’s default firewall configuration tool called Uncomplicated Firewall (UFW) by listing currently available UFW application profiles and allowing HTTP traffic for Nginx on preferred option which was on only port 80  - `sudo ufw allow app list` and `sudo ufw allow “Nginx HTTP”`
- Enabled the ufw firewall - `sudo ufw enabled`
- Verified the change - `sudo ufw status`
- Finally checked my server’s public IP address in a web browser to spot check - `http://l0.0.2.15`
- My server’s IP address was gotten from - `hostname -i`

![Installing Nginx](images/nginx-vagrant_getting_started_default_1699627975212_71996%20[Running]%20-%20Oracle%20VM%20VirtualBox%2011_25_2023%201_29_14%20PM.png)

### Step 2: Installing MySQL
- Installed the MySQL database system to store and manage data for my site - `sudo apt install mysql-server`
- When the installation finished, I ran a security script that came pre-installed to remove some insecure default settings and lock down access to my database system and this would attempt to set a password for the installation’s root MySQL account (even though it wouldn’t require me to provide a password to connect as the root user having defined one when running the secure script because the default authentication method for the admin MySQL user is unix-socket instead of password as it allows only system users with sudo privileges to log in as the MySQL user; so setting a password for the root MySQL account is a safeguard in case the default authentication method is changed from unix_socket to password). In the secure script I accepted the rest of the questions with a Yes such as: to remove some anonymous users and the test database, disable remote root logins and so on – `sudo mysql_secure_installation`
- Exited the console (with exit command), and tested my login access to the database with the root user now authenticated – `sudo mysql`
- Exited the console again after I logged in successfully - `exit`

![Installing MySQL](images/mysql-vagrant_getting_started_default_1699627975212_71996%20[Running]%20-%20Oracle%20VM%20VirtualBox%2011_24_2023%2012_07_06%20PM.png)

### Step 3: Installing PHP
- PHP will process code to display dynamic content to the final user. Since Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server allowing for a better overall performance in most PHP-based websites (unlike Apache server that embeds the PHP interpreter in each request), it required additional configuration. I installed `php-fpm` (PHP fastCGI process manager) to tell Nginx to pass PHP requests to this software for processing. Also installed `php-mysql` module to allow php communicate with MySql-based databases. Core PHP packages would automatically be installed as dependencies – `sudo apt install php7.4-fpm php-mysql`
  
### Step 4: Configuring Nginx to Use the PHP Processor
- Created a server block (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain from a single server. Did this first to hold my website’s files and folders before testing my LAMP setup with a PHP script. First, created a directory for my virtual host (domain) in Nginx default enabled server block configured to serve documents from the `/var/ww/html` directory but leaving `/var/www/html` as the default directory to be served if a client request doesn’t match any other sites and just using `/var/www/` instead – `sudo mkdir /var/www/lempstackproject`
- Assigned ownership of the directory with the $USER environment variable which will reference your current system user – `sudo chown -R $USER:$USER /var/www/lempstackproject`
- Then, opened a new configuration file in Nginx’s sites-available directory using a VI command-line editor – `sudo vi /etc/nginx/sites-available/lempstackproject`
```markdown  
server {
    listen 80;
    server_name lempstackproject www.lempstackproject;
    root /var/www/lempstackproject;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
}

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
- After which I activated my configuration by linking to the config file from Nginx’s sites-enabled directory – `sudo ln -s /etc/nginx/sites-available/lempstackproject /etc/nginx/sites-enabled/`
- Then unlinked the default configuration file from the /sites-enabled/ directory – `sudo unlink /etc/nginx/sites-enabled/default`
- Tested my configuration file for syntax errors. If any errors reported, then I had to go back to review its content before continuing – `sudo nginx -t`
- Reloaded Nginx so these changes take effect and my new website becomes active – `sudo systemctl reload nginx`
- Since my web root directory -`/var/www/lempstackproject` had no file yet, the new website didn’t serve any content even though it’s active. This was solved by creating a file e.g index.html in that location to test if the new server block worked as expected. VI editor was used to edit the file to display a very basic content that greeted ‘Hello World’ – `sudo vi /var/www/lempstackproject.index.html`
After which I opened a web browser to access my server’s domain name and view the content of the file (index.html) – `http://lempstackproject`

![Configure PHP Processor](images/indexhtml-lemp-vagrant_getting_started_default_1699627975212_71996%20[Running]%20-%20Oracle%20VM%20VirtualBox%2011_25_2023%204_20_04%20PM.png)

### Step 5: Testing PHP Processing with Nginx
- Since the index.html file was tested successfully and my LEMP stack completely set up, I went ahead and created an info.php file in my document root to test if that nginx can correctly handle .php files off to my PHP processor – `sudo vi /var/www/lempstackproject/info.php`
```php
	<?php 

	phpinfo();
```
- Confirmed by visiting a web browser to access my server’s domain name or IP address followed by the name of the script – `http://lempstackproject/info.php`

![Testing PHP Processor](images/22.png)

- Then, removed the info.php script  after checking the relevant info about my PHP server through the page since it contains relevant info and also sensitive info about my PHP environment and Ubuntu server – `sudo rm /var/www/lempstackproject/info.php`
Note that this file can always be regenerated if needed later.

### Step 6: Testing Database Connection from PHP
- Then went on to test whether PHP can connect to MySQL and execute database queries by creating a test table with test data in MySQL database, and query for its content from a PHP script. First connected to the MySQL console – `sudo mysql -u root -p or sudo mysql`
- In the console, created a new database – `CREATE DATABASE lempstack_database;`
- Since some versions of PHP from PHP 7.4 below don’t work reliably with the default caching_sha2_password authentication method, I created a new user to authenticate with `mysql_native_password` as default authentication  in order to connect to the MySQL database from PHP – `CREATE USER ‘Todun’’@’%’ IDENTIFIED WITH mysql_native_password BY ‘password’;`
- Gave the user permission over the database - `GRANT ALL ON lempstack_database.* TO ‘Todun’@’%’;`
- Exited the console – `exit`
- Then tested if the new user created i.e. Todun had the proper permissions by logging in to the MySQL console again – `sudo mysql -u Todun – p`. The `-p` flag prompted me for the password used when the user was created.
- Once back, confirmed I had access to the lampstack_database by viewing the list of databases available – `SHOW DATABASES;`
- Next, created a test table named todo_list –
```mysql
 CREATE TABLE lempstack_database.todo_list (
Item_id INT AUTO_INCREMENT,
Content VARCHAR(255),
PRIMARY KEY(item_id)
);
```
- Then inserted a few rows of content in the test table – INSERT INTO lampstack_database.todo_list (content) VALUES (“Networking”);
Confirmed the data was successfully saved to my table – SELECT * FROM lempstack_database.todo_list;
Exited the MySQL console once the confirmation was successful – `exit`
- Next step was to create the PHP script that would connect to MySQL and query for my content. The PHP file was created in my custom web root directory using Vi editor – `sudo vi /var/www/lempstackproject/todo_list.php`
```php
<?php
$user = "Todun";
$password = "password";
$database = "lempstack_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>Today’s Schedule</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
- Finally, accessed the page in my web browser by visiting my domain name or public IP address configured for my website, followed by `/todo_list.php` which means my PHP environment is ready to connect and interact with my MySQL server – `http://lempstackproject/todo_list.php`

![Testing DB from PHP](images/lemp-db-vagrant_getting_started_default_1699627975212_71996%20[Running]%20-%20Oracle%20VM%20VirtualBox%2011_25_2023%209_45_07%20PM.png)