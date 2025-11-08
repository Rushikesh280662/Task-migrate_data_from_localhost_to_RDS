# 🚀 Database Migration from Localhost (EC2) to Amazon RDS (MySQL)

This guide provides a **complete step-by-step process** to migrate a database from a local EC2 instance (using MariaDB/MySQL) to an Amazon RDS MySQL database.

---

## 🧰 Prerequisites
- AWS account with EC2 & RDS permissions  
- SSH key pair (.pem file)  
- RDS endpoint, username, and password  
- EC2 instance access (Linux)  
- Basic shell & MySQL knowledge
  
---

## 🧭 **Overview**
We’ll complete this task in three major parts:
1. **Set up the EC2 instance**
2. **Create the database and generate a backup**
3. **Set up RDS and restore the data**

---

## ⚙️ **Part 1: Set up the EC2 Instance**

### 🪟 Step 1 — Create the Instance
1. Log in to your AWS Management Console.  
2. Go to **EC2 → Instances → Launch instance.**
3. **Name the instance** (e.g., `migration-ec2`).  
4. **Choose OS:** Select **Amazon Linux**.  
5. **Key pair:** Choose or create a new key pair, then download the `.pem` file.
6. **Network settings:** Click **Edit** and enable **Auto-assign Public IP**.
7. Click **Launch instance**.

---

### 🔑 Step 2 — Connect to the EC2 Instance (SSH)
1. Open your terminal or PowerShell.  
2. Run the SSH command (replace with your file path and instance public IP):
   ```bash
   ssh -i .\path	o\your-key.pem ec2-user@<EC2_PUBLIC_IP>
   ```

---

### 🧩 Step 3 — Install and Configure MariaDB
Once inside your EC2 instance:

1. **Update your packages:**
   ```bash
   sudo yum update -y
   ```
2. **Install MariaDB server:**
   ```bash
   sudo yum install mariadb105-server -y
   ```
3. **Start MariaDB service:**
   ```bash
   sudo service mariadb start
   ```
4. **Enable MariaDB to start automatically on boot:**
   ```bash
   sudo systemctl enable mariadb.service
   ```
5. **Check the service status:**
   ```bash
   sudo service mariadb status
   ```

✅ You should see that MariaDB is active and running.

---

## 🗃️ **Part 2: Create Database and Backup**

### 💻 Step 1 — Access MySQL
```bash
sudo mysql
```

### 🔐 Step 2 — Set Root User Password
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'pass!123';
```
Then type:
```sql
EXIT;
```

Now log in again with the password:
```bash
sudo mysql -u root -p
```

---

### 🏗️ Step 3 — Create a Database and Table
Once inside MySQL:

```sql
CREATE DATABASE studentdb;
USE studentdb;

CREATE TABLE students (
  roll_no INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  contact VARCHAR(20),
  address VARCHAR(255)
);
```

---

### 🧾 Step 4 — Insert Dummy Data
```sql
INSERT INTO students (name, contact, address)
VALUES
('Amit Kumar', '9876543210', 'Delhi'),
('Riya Sharma', '9123456780', 'Mumbai'),
('Sanjay Verma', '9988776655', 'Kolkata'),
('Neha Patel', '9090909090', 'Ahmedabad'),
('Vikas Singh', '9812345678', 'Bangalore'),
('Pooja Mehta', '9001234567', 'Chennai'),
('Rahul Das', '9822334455', 'Pune'),
('Simran Kaur', '9911223344', 'Chandigarh');
```

Now verify the entries:
```sql
SELECT * FROM students;
```

Once confirmed, exit:
```sql
EXIT;
```

---

### 💾 Step 5 — Create a Database Backup
Run the following command in your EC2 terminal:
```bash
mysqldump -u root -p studentdb > Mybackup.sql
```

- `studentdb` → Your database name  
- `Mybackup.sql` → Backup file name  

After typing your password, check if the file exists:
```bash
ls
```

Confirm it contains your table structure:
```bash
grep -i "create table" Mybackup.sql
```

✅ You should see “CREATE TABLE `students`” in the output.

---

## ☁️ **Part 3: Set up Amazon RDS and Restore Backup**

### 🧱 Step 1 — Create RDS Database
1. Go to **AWS Console → RDS → Create database.**
2. Choose **Easy create.**
3. **Engine type:** Select **MySQL**.
4. **DB instance size:** Choose **Free tier**.
5. **DB instance identifier:** Leave default or rename.
6. **Credentials:** Choose **Self managed**, then provide and confirm a password.
7. Under **EC2 compute resource**, choose your **EC2 instance**.
8. Click **Create database**.

If any pop-up appears, close it.

---

### 🔗 Step 2 — Connect EC2 to RDS
In your EC2 terminal, connect using this format:
```bash
mysql -h <RDS_ENDPOINT> -u admin -p
```
Example:
```bash
mysql -h database-1.cbu8u8gecfhz.us-west-2.rds.amazonaws.com -u admin -p
```

Enter your password when prompted.

---

### 🗄️ Step 3 — Create Database on RDS
Inside the MySQL shell on RDS:
```sql
CREATE DATABASE studentdb;
EXIT;
```

---

### 📤 Step 4 — Restore the Data from Backup
Run the command below in EC2 terminal:
```bash
mysql -h <RDS_ENDPOINT> -u admin -p studentdb < Mybackup.sql
```
Example:
```bash
mysql -h database-1.cbu8u8gecfhz.us-west-2.rds.amazonaws.com -u admin -p studentdb < studentdbbackup.sql
```

Enter your password.

---

### 🧮 Step 5 — Verify the Restored Data
Check if the data is successfully migrated to RDS:
```bash
mysql -h <RDS_ENDPOINT> -u admin -p
USE studentdb;
SHOW TABLES;
SELECT * FROM students;
```

✅ You should see all your inserted rows (e.g., Amit Kumar, Riya Sharma, etc.).

---

## 🎯 **Final Verification**
1. Compare the number of rows in EC2 and RDS:
   ```sql
   SELECT COUNT(*) FROM students;
   ```
2. Check if table structure matches:
   ```sql
   DESCRIBE students;
   ```

---

## 🔐 **Cleanup and Best Practices**
- Remove `.pem` key and backup files from EC2 once migration is complete.  
- Disable **public access** to RDS and EC2.  
- Take an **RDS snapshot** after successful migration.  
- Always use **strong passwords** and rotate credentials regularly.  

---

## ✅ **Summary of Key Commands**

| Task | Command |
|------|----------|
| SSH to EC2 | `ssh -i .\path	o\key.pem ec2-user@<EC2_PUBLIC_IP>` |
| Install MariaDB | `sudo yum install mariadb105-server -y` |
| Start service | `sudo service mariadb start` |
| Backup database | `mysqldump -u root -p studentdb > Mybackup.sql` |
| Restore to RDS | `mysql -h <endpoint> -u admin -p studentdb < Mybackup.sql` |
| Verify data | `SELECT * FROM students;` |

---

**🎉 Migration Complete!**  
Your database has been successfully transferred from EC2 (localhost) to AWS RDS MySQL.

## Author
**Rushikesh Dase**  
Cloud Computing Enthusiast | AWS Learner  
📧 daserushikesh@gmail.com  
🔗 https://www.linkedin.com/in/rushi-dase  
**Category:** | AWS | Cloud Computing | EC2 | VPC | Load Balancing | RDS |
