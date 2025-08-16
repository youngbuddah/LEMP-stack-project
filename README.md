Perfect 🚀 — let’s build you a ready-to-run LEMP project on Ubuntu 24.04, where:

Server → Ubuntu 24.04 at 13.234.29.239

Web server → Nginx + PHP-FPM

Database → AWS RDS (MariaDB/MySQL)

App → Simple PHP project that inserts and displays data from your RDS database

📦 Project Setup (Step by Step)
1. Install Dependencies on Ubuntu

SSH into your server:

ssh ubuntu@13.234.29.239

Then install Nginx, PHP-FPM, and MySQL extension:

sudo apt update && sudo apt upgrade -y
sudo apt install nginx php-fpm php-mysql -y

2. Project Directory

Create your app folder:

sudo mkdir -p /var/www/lemp-project
sudo chown -R $USER:$USER /var/www/lemp-project
sudo cd  /var/www/lemp-project

3. Project Files
config.php

Edit with your AWS RDS endpoint, user, password, and DB:

<?php
$host = "<aws-rds-server>"; // your RDS endpoint
$port = "3306";                                     // default RDS port
$db   = "myapp";                                    // database name
$user = "admin";                                    // RDS username
$pass = "<mysql-password>";                               // RDS password

$dsn = "mysql:host=$host;port=$port;dbname=$db;charset=utf8mb4";

index.php

<?php
require 'config.php';

try {
    $pdo = new PDO($dsn, $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    echo "<h1>✅ Connected to AWS RDS from Ubuntu 24.04 LEMP</h1>";

    // Insert sample data
    if (isset($_POST['name']) && !empty($_POST['name'])) {
        $stmt = $pdo->prepare("INSERT INTO users (name) VALUES (?)");
        $stmt->execute([$_POST['name']]);
        echo "<p>Inserted user: " . htmlspecialchars($_POST['name']) . "</p>";
    }

    // Show users
    $stmt = $pdo->query("SELECT * FROM users");
    echo "<h2>Users in Database:</h2><ul>";
    while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
        echo "<li>" . htmlspecialchars($row['name']) . "</li>";
    }
    echo "</ul>";

} catch (PDOException $e) {
    echo "<h1>❌ DB Connection Failed</h1>";
    echo $e->getMessage();
}
?>

<form method="POST">
    <input type="text" name="name" placeholder="Enter name" required>
    <button type="submit">Add User</button>
</form>

4. AWS RDS Setup

Log in to your RDS (via MySQL client or AWS console query editor) and run:

CREATE DATABASE myapp;
USE myapp;

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

INSERT INTO users (name) VALUES ('Rock'), ('LEMP Test User');

5. Nginx Configuration

Create new config:

sudo nano /etc/nginx/sites-available/lemp-project

Paste:

server {
    listen 80;
    server_name <public-ip-address>; # enter your public address

    root /var/www/lemp-project;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;   # check version with: ls /var/run/php/
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}

Enable the site:

sudo ln -s /etc/nginx/sites-available/lemp-project /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx

6. Test

Go to your browser:
http:<public-ip-address>

You’ll see connection success
Users list will load from RDS
You can insert new users with the form

✅ This gives you a fully working LEMP project with AWS RDS:

index.php → Insert & display data
config.php → Stores DB connection details
Nginx serves the app on your server IP
