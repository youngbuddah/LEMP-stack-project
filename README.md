# 🚀 LEMP Project with AWS RDS (Ubuntu 24.04 + Nginx + PHP-FPM + RDS MySQL/MariaDB)

This project demonstrates how to deploy a **LEMP stack** on Ubuntu 24.04 with:
- **Linux (Ubuntu 24.04)**
- **Nginx** (web server)
- **PHP-FPM** (PHP runtime)
- **AWS RDS (MariaDB/MySQL)** (database)

It provides a simple PHP app that:
- Connects to AWS RDS
- Inserts user data
- Displays users from the database

---

## 🛠️ Setup Instructions

### 1. Connect to Ubuntu Server
SSH into your EC2 instance:
```bash
ssh ubuntu@<your-public-ip>
```

Update the system:
```bash
sudo apt update && sudo apt upgrade -y
```

---

### 2. Install Dependencies
Install Nginx, PHP-FPM, and MySQL extension:
```bash
sudo apt install nginx php-fpm php-mysql -y
```

---

### 3. Project Directory
Create project folder:
```bash
sudo mkdir -p /var/www/lemp-project
sudo chown -R $USER:$USER /var/www/lemp-project
cd /var/www/lemp-project
```

---

### 4. Application Files

#### `config.php`
```php
<?php
$host = "<aws-rds-endpoint>"; // e.g. mydb.xxxxxx.ap-south-1.rds.amazonaws.com
$port = "3306";               // default RDS port
$db   = "myapp";              // database name
$user = "admin";              // RDS username
$pass = "<mysql-password>";   // RDS password

$dsn = "mysql:host=$host;port=$port;dbname=$db;charset=utf8mb4";
```

#### `index.php`
```php
<?php
require 'config.php';

try {
    $pdo = new PDO($dsn, $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    echo "<h1>✅ Connected to AWS RDS from Ubuntu 24.04 LEMP</h1>";

    // Insert new user
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
```

---

### 5. AWS RDS Setup
Login to your RDS DB and run:
```sql
CREATE DATABASE myapp;
USE myapp;

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

INSERT INTO users (name) VALUES ('Rock'), ('LEMP Test User');
```

---

### 6. Nginx Configuration
Create a new server block:
```bash
sudo nano /etc/nginx/sites-available/lemp-project
```

Paste:
```nginx
server {
    listen 80;
    server_name <your-public-ip>;

    root /var/www/lemp-project;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock; # adjust if version differs
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/lemp-project /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

---

### 7. Test in Browser
Visit:
```
http://<your-public-ip>
```

- You should see **connection success** 🎉  
- Existing users loaded from RDS  
- You can insert new users using the form  

---

## ✅ Features
- PHP-FPM handles PHP scripts
- Nginx serves the app
- AWS RDS as external DB
- Insert + display users dynamically


## 📸 Demo Screenshot 
![Demo Screenshot](https://github.com/youngbuddah/LEMP-stack-project/blob/main/Screenshot%20from%202025-08-16%2022-55-12.png)


---

## 👨‍💻 Author
**Abhay Bendekar**  
🔗 [LinkedIn Profile](https://www.linkedin.com/in/abhay-bendekar-75474b372/)
