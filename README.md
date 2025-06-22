
# 🏗️ Advanced 3-Tier Architecture on AWS with Auto Scaling and Load Balancers

This setup includes:
- Multiple subnets across AZs (High Availability)
- Internal & Internet-facing Load Balancers
- Auto Scaling for Web and App tiers
- RDS in private subnets
- Nginx, PHP, MariaDB stack

---

## 🔹 Step 1: Login to AWS

🔐 Go to [AWS Console](https://aws.amazon.com/console/)  
🔑 Login using your credentials

---

## 🔹 Step 2: VPC, Subnets, Internet Gateway, Route Tables

1️⃣ **Create VPC**  
   - CIDR: `10.0.0.0/16`

2️⃣ **Create Subnets**  
   - Public Subnet_A: `10.0.0.0/20`  
   - Public Subnet_B: `10.0.16.0/20`  
   - Web Subnet_A: `10.0.32.0/20`  
   - Web Subnet_B: `10.0.48.0/20`  
   - App Subnet_A: `10.0.64.0/20`  
   - App Subnet_B: `10.0.80.0/20`  
   - DB Subnet_A: `10.0.96.0/20`  
   - DB Subnet_B: `10.0.112.0/20`

3️⃣ **Create Internet Gateway**  
   - Attach to VPC

4️⃣ **Create Route Tables**  
   - Public RT: `0.0.0.0/0 → IGW`  
   - Private RT: No IGW  
   - Associate Public Subnets with Public RT  
   - Associate Private Subnets with Private RT

---

## 🔹 Step 3: RDS Database Setup

🗂️ **Create DB Subnet Group**

🛢️ **Launch RDS MySQL Instance**
- Use DB Subnet_A and DB Subnet_B
- Private Subnet only

🔌 **Connect from App-Tier**  
```bash
sudo mysql -h <rds-endpoint> -u username -p
```

🗄️ **Create Database & Table**
```sql
CREATE DATABASE mydb;
USE mydb;
CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255));
```

---

## 🔹 Step 4: App-Tier Dev Instance (in Public Subnet)

🚀 **Launch EC2 Instance**
- In Public Subnet
- Allow `SSH (22)` & `HTTP (80)`

🔧 **Install Packages**
```bash
sudo yum install php mariadb105-server php8.3-mysqlnd.x86_64 -y
sudo service mariadb start
sudo service php-fpm start
sudo systemctl enable mariadb.service
sudo systemctl enable php-fpm.service
```

📝 **Create `submit.php`**
```php
<?php
$conn = new mysqli("rds-endpoint", "username", "password", "mydb");
$name = $_POST['username'];
$conn->query("INSERT INTO users (name) VALUES ('$name')");
echo "Data Submitted!";
?>
```

📦 **Save to** `/usr/share/nginx/html/submit.php`

---

## 🔹 Step 5: Create AMI of App-Tier

📸 Create image of App-Tier for use in Auto Scaling Group

---

## 🔹 Step 6: Create Auto Scaling Group for App-Tier

🔁 Launch config/template from App-Tier AMI  
🎯 Attach to **Internal Load Balancer**

---

## 🔹 Step 7: Web-Tier Dev Instance (in Public Subnet)

🚀 **Launch EC2 Instance**
- In Public Subnet
- Allow `SSH (22)` & `HTTP (80)`

🔧 **Install Packages**
```bash
sudo yum install nginx php mariadb105-server -y
sudo service nginx start
sudo service php-fpm start
sudo systemctl enable nginx
sudo systemctl enable php-fpm
```

📝 **Create `form.html`**
```html
<form action="submit.php" method="post">
    <input type="text" name="username" placeholder="Enter your name" required>
    <input type="submit" value="Submit">
</form>
```

📁 Save to: `/usr/share/nginx/html/form.html`

🛠️ **Update NGINX Config**
```nginx
location ~ \.php$ {
    proxy_pass http://<App-tier-IP or LB-DNS>;
}
```
🔁 Reload NGINX:
```bash
sudo service nginx reload
```

---

## 🔹 Step 8: Create AMI of Web-Tier

📸 Create image of Web-Tier for Auto Scaling use

---

## 🔹 Step 9: Auto Scaling Group for Web-Tier

🔁 Launch config/template from Web-Tier AMI  
🌐 Attach to **Internet-facing Load Balancer**

---

## 🔒 Security Group Rules

✅ Allow:
- Internet LB SG → Web-Tier SG (HTTP 80)  
- Web-Tier SG → Internal LB SG (HTTP 80)  
- Internal LB SG → App-Tier SG (HTTP 80)  
- App-Tier SG → RDS SG (MySQL 3306)

---

## ✅ Step 10: Final Verification

🌐 Access:  
```
http://<web-tier-public-ip>/form.html
```

📤 Submit a name

📊 Query in RDS:
```sql
SELECT * FROM users;
```
---

## 💬 Author

Created by **Raj Chauhan**  
Internship Project: **AWS 3-Tier with Auto Scaling & Load Balancing**
