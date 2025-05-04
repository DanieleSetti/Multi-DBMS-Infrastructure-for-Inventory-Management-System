# 🗄️ Multi-DBMS Infrastructure Lab

## 📘 Project Overview

This project simulates the infrastructure supporting a fictional Enterprise Inventory & Asset Management System. The goal is to reflect realistic challenges and solutions a DBA faces when managing diverse data workloads across multiple platforms (PostgreSQL, MySQL, SQL Server, MongoDB) with attention to automation, security, and operational resilience.

---

## 🔧 Objective

To showcase hands-on competence in:
- Installing and configuring relational and non-relational DBMS
- Automating deployments with Ansible
- Managing users and enforcing role-based security policies
- Designing backup and restore strategies
- Simulating indexing and query tuning
- Planning for disaster recovery and scalability

---

## 🎯 Goals

* Create a realistic, testable multi-DBMS lab environment using PostgreSQL, MariaDB, and MongoDB
* Use Linux-native tools and practices for automation, scripting, and access control
* Focus on **practical sysadmin tasks**: installation, secure configuration, logging, user management
* Simulate batch inserts and performance bottlenecks to apply indexing and slow query logging
* Enable daily backups and prepare for restore scenarios
* Document operational steps clearly enough for reuse, audit, or handover

---

## ⚙️ Scripts

All automation and manual utilities are placed under the `/scripts` folder:

* `pg_backup.sh` – Backs up PostgreSQL database with timestamped dumps
* `mariadb_backup.sh` – Dumps MariaDB using `mysqldump` and rotates old backups
* `mongodump.sh` – Backs up MongoDB collections
* `batch_insert.sh` – Simulates large-scale data insertion into PostgreSQL
* `mariadb_batch_insert.sh` – Equivalent for MariaDB
* `check_services.sh` – Verifies that all database services are active and reachable

Each script is written for **manual execution or scheduling via `cron`**, and includes logging or error output when relevant.

---

## ⚙️ Initial Infrastructure Setup & Ansible Configuration

This section documents the initial setup of the virtual lab environment and the baseline configuration for Ansible-based automation.

### 🖥️ VM Configuration

Two Ubuntu-based virtual machines were created using VirtualBox:

* **VM1** (Controller): `192.168.56.10`
* **VM2** (Database Node): `192.168.56.11`
* **User**: `admin`
* **Password**: `test123`

Each VM was configured with **two network adapters**:

* `enp0s3` (NAT): for internet access
* `enp0s8` (Host-Only): for internal communication

Netplan was configured manually on both VMs to assign static IPs to `enp0s8`:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.10/24  # For VM1
        - 192.168.56.11/24  # For VM2
```

---

### 🔐 SSH Key Authentication

* Installed `openssh-server` to enable SSH access.
* Used `ssh-copy-id` from Git Bash (host is Windows) to enable passwordless SSH from VM1 to VM2.
* Configured mutual SSH access between VMs to prepare for distributed tasks (e.g., backups, replication).

---

### 🧰 Ansible Setup

On **VM1 (Controller)**:

* Installed Ansible via APT.
* Created working directory: `~/db-lab-ansible/`
* Created an inventory file:

```ini
# ~/db-lab-ansible/inventory.ini
[db_servers]
192.168.56.11 ansible_user=admin ansible_ssh_private_key_file=~/.ssh/id_rsa
```

* Created an Ansible config file:

```ini
# ~/db-lab-ansible/ansible.cfg
[defaults]
inventory = ./inventory.ini
```

* Exported config path globally:

```bash
echo 'export ANSIBLE_CONFIG=~/db-lab-ansible/ansible.cfg' >> ~/.bashrc
source ~/.bashrc
```

---

### 📝 Sample Playbook

Tested Ansible connectivity and sudo privileges using a simple playbook:

```yaml
# ~/db-lab-ansible/playbooks/common.yml
- name: Basic package install
  hosts: all
  become: yes
  tasks:
    - name: Install htop
      apt:
        name: htop
        state: present
```

Ran using:

```bash
ansible-playbook -i ~/db-lab-ansible/inventory.ini ~/db-lab-ansible/playbooks/common.yml --ask-become-pass
```

The successful execution confirms that:

* Inventory and config files are functional.
* SSH keys and privilege escalation are working.
* Folder structure and playbook invocation are correct.


## 📅 PostgreSQL Installation & Remote Access

### ⚙️ Ansible Playbook: `playbooks/postgresql.yml`

This playbook automates the full installation and configuration of PostgreSQL on the target database server (VM2).

```yaml
# playbooks/postgresql.yml
- name: Install and configure PostgreSQL
  hosts: db_servers
  become: yes
  tasks:
    - name: Ensure PostgreSQL is installed
      apt:
        name: postgresql
        state: present
        update_cache: yes

    - name: Allow remote connections in postgresql.conf
      lineinfile:
        path: /etc/postgresql/16/main/postgresql.conf
        regexp: '^#?listen_addresses\s*='
        line: "listen_addresses = '*'"
      notify: Restart PostgreSQL

    - name: Allow connections from LAN in pg_hba.conf
      lineinfile:
        path: /etc/postgresql/16/main/pg_hba.conf
        insertafter: '^# TYPE'
        line: "host all all 192.168.56.0/24 md5"
      notify: Restart PostgreSQL

    - name: Set a password for the 'postgres' user
      shell: sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'postgres';"
      args:
        executable: /bin/bash
      become: yes

  handlers:
    - name: Restart PostgreSQL
      service:
        name: postgresql
        state: restarted
```

---

### 🧠 What This Playbook Does

1. **Installs PostgreSQL** if not already present.
2. **Enables remote access** by modifying `postgresql.conf` (`listen_addresses = '*'`).
3. **Authorizes the internal LAN (192.168.56.0/24)** in `pg_hba.conf` using MD5 authentication.
4. **Sets a password for the `postgres` user**, which is required for password-based remote access.
5. **Restarts PostgreSQL** automatically after configuration changes using the handler mechanism.

---

### 🔍 Technical Validation

| Component               | Status       |
| ----------------------- | ------------ |
| PostgreSQL Installation | ✅ Completed  |
| Remote Access           | ✅ Verified   |
| Configuration Changes   | ✅ Applied    |
| Ansible Automation      | ✅ Functional |
| Internal Network Access | ✅ Authorized |

---

## Why Use MariaDB in This Lab?

We chose **MariaDB** over Oracle MySQL because:

* It is **the default MySQL-compatible server** in most modern Linux distributions.
* It offers **no meaningful functional difference for 95% of use cases**, especially in training and lab environments.
* It avoids **Oracle’s restrictive licensing** and ecosystem lock-in.

---

## Why Not Just Use PostgreSQL for Everything?

Technically, you could — and in some modern systems, you *should*. But real-world sysadmins must face historical, compatibility, and ecosystem constraints.

### ✅ PostgreSQL — Where It Shines

* Fully **standards-compliant**, robust, and feature-rich.
* Ideal for **complex queries**, **JSON processing**, **custom types**, **full-text search**, and **data integrity**.
* Strong choice for **analytics, large-scale systems**, and **compliance-heavy** environments.

### ❌ Why Not Always PostgreSQL?

* **Legacy and enterprise systems** are still deeply integrated with MySQL/MariaDB.
* **Web hosting environments** (e.g., WordPress, Joomla, cPanel) continue to rely on MySQL.
* **Lightweight and fast** for read-heavy, simple applications.

---

## 💡 Why Use Both in This Lab?

### 🔍 1. Comparative Learning

You will gain:

* Practical experience with **different syntaxes**, **configuration patterns**, and **performance behaviors**.
* Recruiters value **versatility** — being able to claim hands-on exposure to both PostgreSQL and MySQL/MariaDB increases your market value.

### 🛠 2. Real-World Scenarios

It is common to see **both DBMS in the same infrastructure**:

* ERP or analytics system → PostgreSQL.
* CMS, user portals, SaaS integrations → MySQL/MariaDB.

A competent sysadmin must deploy, harden, and maintain both.

### 🧠 3. Architectural Maturity

| Feature           | PostgreSQL                       | MySQL/MariaDB                         |
| ----------------- | -------------------------------- | ------------------------------------- |
| Power             | Advanced, standards-compliant    | Lightweight, simpler                  |
| Use Case          | Complex systems, analytics       | Web apps, legacy apps, CMS            |
| Default in Ubuntu | `postgresql`                     | `mariadb-server` (not Oracle MySQL)   |
| Why in Lab?       | Simulate enterprise hybrid stack | Mirror real-world sysadmin challenges |

---

## 📜 Ansible Logic: Ordering and Readability

* **Ansible executes tasks sequentially**, so the order of tasks matters.
* But inside a module (e.g. `mysql_user`), **parameter order is irrelevant**.

**Why put `login_user` and `login_password` at the end?**

1. **Readability**: Primary task actions (name, state, host) are prioritized; auth details come last.
2. **Maintainability**: When using roles or variable overrides (e.g., via Ansible Vault), it’s easier to override `login_user` and `login_password` when they are grouped consistently.

---

## ✅ Final MariaDB Playbook (`mysql.yml`)

```yaml
# ~/db-lab-ansible/playbooks/mysql.yml
- name: Install and configure MySQL (MariaDB)
  hosts: db_servers
  become: yes
# variables are hardcoded, not acceptable in production, Should Use Ansible Vault
  vars:
    mysql_root_password: "rootpass"
    db_test_user: "testuser"
    db_test_pass: "testpass"
    db_test_name: "testdb"

  tasks:
    - name: Install MariaDB server
      apt:
        name: mariadb-server
        state: present
        update_cache: yes

    - name: Ensure MariaDB is running
      service:
        name: mariadb
        state: started
        enabled: yes

# Initially root dont have password, can be set only localy using unix socket
    - name: Set root password
      mysql_user:
        name: root
        host: localhost
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock

# anonymous user is user without name, historical artifact, huge security risk
    - name: Remove anonymous users
      mysql_user:
        name: ''
        host_all: yes
        state: absent
        login_user: root
        login_password: "{{ mysql_root_password }}"

# root@% Shouldn't be created, but for security I remove it
    - name: Disallow remote root login
      mysql_user:
        name: root
        host: '%'
        state: absent
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Create test database
      mysql_db:
        name: "{{ db_test_name }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Create test user with full access to test database
      mysql_user:
        name: "{{ db_test_user }}"
        password: "{{ db_test_pass }}"
        priv: "{{ db_test_name }}.*:ALL"
        host: '%'
# need to Restricting IPs at the firewall
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Allow remote connections in MariaDB config
      lineinfile:
        path: /etc/mysql/mariadb.conf.d/50-server.cnf
        regexp: '^bind-address'
        line: 'bind-address = 0.0.0.0'
#Listen on all available network interfaces, configure firewall, or give single IP access and connect locally using socket
      notify: Restart MySQL

  handlers:
    - name: Restart MySQL
      service:
        name: mariadb
        state: restarted
```

> ⚠️ In production, hardcoded credentials should be moved to **Ansible Vault** or secure variables.

---

## 🛠 PyMySQL Module Required

You must install the `python3-pymysql` package on the database VM (VM2) to allow Ansible’s MySQL modules to function correctly. Otherwise, you will encounter:

```
fatal: [192.168.56.11]: FAILED! => "A MySQL module is required..."
```

Run:

```bash
sudo apt install python3-pymysql
```

---

## ✅ Database Connectivity Test

From VM1 (Ansible host):

```bash
mysql -h 192.168.56.11 -u testuser -p
```

Successful connection confirms that:

* MariaDB is configured to accept remote clients
* The firewall is properly set
* User privileges are valid

---

## 🔥 UFW Configuration for Database Access

```yaml
- name: Configure UFW on database servers
  hosts: db_servers
  become: yes

  tasks:
    - name: Ensure UFW is installed
      apt:
        name: ufw
        state: present
        update_cache: yes

    - name: Allow SSH from Ansible control host
      ufw:
        rule: allow
        from_ip: 192.168.56.10
        port: 22
        proto: tcp

    - name: Allow PostgreSQL port (5432)
      ufw:
        rule: allow
        from_ip: 192.168.56.10
        port: 5432
        proto: tcp

    - name: Allow MySQL/MariaDB port (3306)
      ufw:
        rule: allow
        from_ip: 192.168.56.10
        port: 3306
        proto: tcp

    - name: Deny all other incoming connections
      ufw:
        direction: incoming
        policy: deny #by default will deny all, only exeptions (rule: allow) will work

    - name: Enable UFW with logging
      ufw:
        state: enabled
        logging: 'on'
```

---

## 🛡️ Host Machine Cannot Connect (By Design)

Your host PC cannot access the database VM directly. This is expected — the firewall only allows access from **VM1 (192.168.56.10)**, the Ansible control node.

✔️ This enforces a **bastion model**: external traffic must first connect to the Ansible host, then from there to internal VMs (e.g. via SSH or DB client).

---

## 🧪 Load Simulation & Query Performance Testing

### Load Simulation with Batch Inserts

To simulate real-world data load and test query behavior under realistic conditions, we inserted 10,000 entries into both PostgreSQL and MariaDB using Bash scripts.

Each entry is uniquely identified (`Product_1`, `Product_2`, ..., `Product_10000`) and assigned a pseudo-random quantity.

#### PostgreSQL: `batch_insert.sh`

```bash
#!/bin/bash
for i in {1..1000}; do
  values=""
  for j in {1..10}; do
    idx=$(( (i - 1) * 10 + j ))
    quantity=$((RANDOM % 100))
    values+="('Product_$idx', $quantity),"
  done
  values=${values::-1}
  sudo -u postgres psql -d postgres -c "INSERT INTO inventory (product_name, quantity) VALUES $values;"
done
```

**Verification:**

```sql
SELECT COUNT(*) FROM inventory;  -- Returns 10,000
SELECT * FROM inventory LIMIT 100;
```

#### MariaDB: `mariadb_batch_insert.sh`

```bash
#!/bin/bash
for i in {1..1000}; do
  values=""
  for j in {1..10}; do
    idx=$(( (i - 1) * 10 + j ))
    quantity=$((RANDOM % 100))
    values+="('Product_$idx', $quantity),"
  done
  values=${values::-1}
  mysql -u testuser -p'testpass' -D testdb -e "INSERT INTO products (name, quantity) VALUES $values;"
done
```

**Verification:**

```sql
SELECT COUNT(*) FROM products;   -- Returns 10,000
SELECT * FROM products LIMIT 50;
```

---

### 2.1. Indexing, EXPLAIN & Performance Logging

Indexes were created to evaluate impact on query performance.

**PostgreSQL:**

```sql
CREATE INDEX idx_inventory_name ON inventory (product_name);
```

**MariaDB:**

```sql
CREATE INDEX idx_products_name ON products (name);
```

#### EXPLAIN ANALYZE Results

* **PostgreSQL Before Index**: Execution Time ≈ 2.583 ms
* **PostgreSQL After Index**: Execution Time ≈ 0.104 ms
* **MariaDB Before/After**: \~0.001 sec (minimal improvement due to smaller dataset and internal engine optimizations)

**Queries:**

```sql
-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM inventory WHERE product_name = 'Product_9999';

-- MariaDB
EXPLAIN SELECT * FROM products WHERE name = 'Product_9999';
```

---

### 2.2. Enabling Slow Query Logging

#### PostgreSQL Configuration

```sql
ALTER SYSTEM SET logging_collector = 'on';
ALTER SYSTEM SET log_directory = 'log';
ALTER SYSTEM SET log_filename = 'pg.log';
ALTER SYSTEM SET log_min_duration_statement = 500;  -- in ms
SELECT pg_reload_conf();
```

**Service Restart:**

```bash
sudo systemctl restart postgresql
```

#### MariaDB Configuration (`/etc/mysql/mariadb.conf.d/50-server.cnf`)

```ini
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 0.5
log_queries_not_using_indexes = 1
```

**Service Restart:**

```bash
sudo systemctl restart mariadb
```

---

## 🧩 MongoDB Installation, Configuration & Usage

### 📦 1. Installation on Ubuntu (MongoDB 8.0)

Following official guide: [MongoDB Manual](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)

```bash
sudo apt-get install gnupg curl

curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" \
| sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

sudo apt-get update
sudo apt-get install -y mongodb-org
```

### 🛠 2. Post-Installation Fix

Error:
`"Failed to unlink socket file"` → `/tmp/mongodb-27017.sock`

Cause: Socket was created by wrong user (`admin` instead of `mongodb`).
Fix:

```bash
sudo rm /tmp/mongodb-27017.sock
sudo systemctl restart mongod
```

Confirmed connection via:

```bash
mongosh
```

---

### 🧾 3. Initial Setup & Test Inserts

#### Create Collections

```javascript
use testdb;

db.createCollection("logs");
db.logs.insert({
  "timestamp": new Date(),
  "level": "INFO",
  "message": "Service started successfully",
  "user_id": 1
});

db.createCollection("telemetry");
db.telemetry.insert({
  "timestamp": new Date(),
  "cpu_usage": 45,
  "memory_usage": 72,
  "disk_io": 35
});
```

---

### 🔐 4. Security Configuration

#### Binding IP (already localhost by default):

```yaml
# /etc/mongod.conf
net:
  port: 27017
  bindIp: 127.0.0.1
```

#### User Creation and Authentication Setup

```javascript
use admin;
db.createUser({
  user: "admin",
  pwd: "adminpassword",
  roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
});
```

Enable auth:

```yaml
# /etc/mongod.conf
security:
  authorization: "enabled"
```

```bash
sudo systemctl restart mongod
```

Login and confirm:

```javascript
use admin;
db.auth("admin", "adminpassword");
db.runCommand({ usersInfo: 1 });
```

#### App-Level User

```javascript
use testdb;
db.createUser({
  user: "appuser",
  pwd: "apppassword",
  roles: [{ role: "readWrite", db: "testdb" }]
});
```

---

### 📚 5. Theoretical Schema Design for MongoDB

#### Collection: `users`

```json
{
  "_id": ObjectId("abc123"),
  "username": "daniele",
  "email": "daniele@example.com",
  "bio": "Sysadmin student",
  "followers": ["user123", "user456"],
  "created_at": ISODate("2025-05-03T20:00:00Z")
}
```

#### Collection: `posts`

```json
{
  "_id": ObjectId("post123"),
  "user_id": ObjectId("abc123"),
  "content": "Learning MongoDB!",
  "likes": 42,
  "comments": [
    {
      "user_id": "user456",
      "text": "Good job!",
      "timestamp": ISODate("2025-05-03T20:15:00Z")
    }
  ],
  "timestamp": ISODate("2025-05-03T20:10:00Z")
}
```

---

## 📦 Backup & Restore Scripts (Day 5)

> All backups are designed to be integrated into future **Ansible Playbooks** for automation.

---

### 1.1 PostgreSQL Backup Script (`pg_backup.sh`)

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/postgres"
mkdir -p "$BACKUP_DIR"
FILENAME="pg_backup_$(date +%F_%H-%M).sql"
sudo -u postgres pg_dumpall > "$BACKUP_DIR/$FILENAME"
```

---

### 1.3 MongoDB Backup Script (`mongo_backup.sh`)

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/mongodb"
mkdir -p "$BACKUP_DIR"
TIMESTAMP=$(date +%F_%H-%M)

MONGO_USER="admin"
MONGO_PASS="adminpassword"
MONGO_AUTH_DB="admin"

mongodump --out "$BACKUP_DIR/mongodump_$TIMESTAMP" \
          --username "$MONGO_USER" \
          --password "$MONGO_PASS" \
          --authenticationDatabase "$MONGO_AUTH_DB"
```

#### Restore Example

```bash
mongorestore "$BACKUP_DIR/mongodump_2025-05-03_10-45"
```

#### Grant backup role if needed

```javascript
use admin;
db.grantRolesToUser("admin", [{ role: "backup", db: "admin" }]);
```

---

### ✅ Status Summary

| DBMS       | Insert Simulation | Index Tested   | Logging Config | Backup Script | Authentication |
| ---------- | ----------------- | -------------- | -------------- | ------------- | -------------- |
| PostgreSQL | ✅ 10K rows        | ✅ (EXPLAIN)    | ✅ `>500ms`     | ✅             | N/A            |
| MariaDB    | ✅ 10K rows        | ✅              | ✅ `>0.5s`      | To do (1.2)   | N/A            |
| MongoDB    | ✅ sample docs     | N/A (No JOINs) | N/A            | ✅             | ✅              |

---

## 🔧 TASK 2 – Service Monitoring (Manual)

To monitor the status of your services and ensure they are running and set to start on boot, use the following script `check_services.sh`:

```bash
for service in postgresql mysql mongod; do
  if systemctl is-active --quiet $service; then
    echo "$service is running"
  else
    echo "$service is DOWN"
  fi
done
````

**Example Output:**

```bash
postgresql is running
mysql is running
mongod is running
```

---

## 🔧 Basic Health Checks

### Example Health Check Script

```bash
#!/bin/bash

LOG_FILE="/var/log/db_health.log"

# PostgreSQL credentials
PGUSER="postgres"
PGPASSWORD="your_postgres_password"
export PGUSER PGPASSWORD

# MariaDB/MySQL credentials
MYSQL_USER="root"
MYSQL_PASSWORD="your_mysql_password"

# MongoDB credentials
MONGO_USER="admin"
MONGO_PASSWORD="your_mongo_password"
MONGO_AUTH_DB="admin"

# Check PostgreSQL
psql -h localhost -c '\l' &> /dev/null
if [ $? -eq 0 ]; then
  echo "$(date) - PostgreSQL is healthy" >> "$LOG_FILE"
else
  echo "$(date) - PostgreSQL FAILED" >> "$LOG_FILE"
fi
unset PGUSER PGPASSWORD

# Check MariaDB / MySQL
mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" -e "SHOW DATABASES;" &> /dev/null
if [ $? -eq 0 ]; then
  echo "$(date) - MariaDB is healthy" >> "$LOG_FILE"
else
  echo "$(date) - MariaDB FAILED" >> "$LOG_FILE"
fi

# Check MongoDB
mongo --username "$MONGO_USER" --password "$MONGO_PASSWORD" --authenticationDatabase "$MONGO_AUTH_DB" --eval "db.adminCommand('ping')" &> /dev/null
if [ $? -eq 0 ]; then
  echo "$(date) - MongoDB is healthy" >> "$LOG_FILE"
else
  echo "$(date) - MongoDB FAILED" >> "$LOG_FILE"
fi
```

### Explanation:

* **PostgreSQL:** Pings PostgreSQL using the `psql` command.
* **MariaDB/MySQL:** Uses the `mysql` command to check if it can show databases.
* **MongoDB:** Uses the `mongo` shell to execute a ping command and verify the service's health.

The results are logged to `/var/log/db_health.log`, where each entry includes the timestamp and the health status of each DBMS.

**Sample log output:**

```bash
2025-05-03 20:00:00 - PostgreSQL is healthy
2025-05-03 20:00:05 - MariaDB is healthy
2025-05-03 20:00:10 - MongoDB is healthy
```
### Service Monitoring (Manual)

#### Checking Services Status

To ensure that your database services are running and set to start on boot, we can create a script `check_services.sh` to monitor their statuses.

**Script: `check_services.sh`**

```bash
#!/bin/bash

for service in postgresql mysql mongod; do
  if systemctl is-active --quiet $service; then
    echo "$service is running"
  else
    echo "$service is DOWN"
  fi
done
```

**Output Example:**

```
postgresql is running
mysql is running
mongod is running
```

This script checks the status of `postgresql`, `mysql`, and `mongod` using `systemctl is-active`. If the service is running, it outputs "running"; otherwise, it reports that the service is down.

---

### Basic Health Checks for Databases

To perform additional health checks on each of the DBMS (PostgreSQL, MySQL/MariaDB, MongoDB), we can use a script to ping each database via their respective command-line tools (`psql`, `mysql`, `mongo`). The health check script will log the results to a file located at `/var/log/db_health.log`.

#### Example Script: `db_health_check.sh`

```bash
#!/bin/bash

LOG_FILE="/var/log/db_health.log"

# PostgreSQL credentials
PGUSER="postgres"
PGPASSWORD="your_postgres_password"
export PGUSER PGPASSWORD

# MariaDB/MySQL credentials
MYSQL_USER="root"
MYSQL_PASSWORD="your_mysql_password"

# MongoDB credentials
MONGO_USER="admin"
MONGO_PASSWORD="your_mongo_password"
MONGO_AUTH_DB="admin"

# Check PostgreSQL
psql -h localhost -c '\l' &> /dev/null
if [ $? -eq 0 ]; then
  echo "$(date) - PostgreSQL is healthy" >> "$LOG_FILE"
else
  echo "$(date) - PostgreSQL FAILED" >> "$LOG_FILE"
fi
unset PGUSER PGPASSWORD

# Check MariaDB / MySQL
mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" -e "SHOW DATABASES;" &> /dev/null
if [ $? -eq 0 ]; then
  echo "$(date) - MariaDB is healthy" >> "$LOG_FILE"
else
  echo "$(date) - MariaDB FAILED" >> "$LOG_FILE"
fi

# Check MongoDB
mongo --username "$MONGO_USER" --password "$MONGO_PASSWORD" --authenticationDatabase "$MONGO_AUTH_DB" --eval "db.adminCommand('ping')" &> /dev/null
if [ $? -eq 0 ]; then
  echo "$(date) - MongoDB is healthy" >> "$LOG_FILE"
else
  echo "$(date) - MongoDB FAILED" >> "$LOG_FILE"
fi
```

**Explanation:**

* The script performs health checks on three different database systems:

  * **PostgreSQL**: Pings the PostgreSQL database using `psql` and checks if it can list databases with the `\l` command.
  * **MariaDB/MySQL**: Uses the `mysql` command to show databases, ensuring the database is responsive.
  * **MongoDB**: Pings the MongoDB server using the `mongo` shell, with authentication details provided.
* The results of these health checks are logged in the `/var/log/db_health.log` file.

**Sample Log Output (`/var/log/db_health.log`):**

```
2025-05-05 12:00:01 - PostgreSQL is healthy
2025-05-05 12:00:01 - MariaDB is healthy
2025-05-05 12:00:01 - MongoDB is healthy
```

In case any database fails the check, an entry with `FAILED` is logged, indicating the service that needs attention.

### Automating Health Checks

To automate the health checks and monitor the status of the databases, you can schedule this script to run periodically using `cron`. Here’s how to schedule it to run every 5 minutes:

1. Open the crontab editor by running:

   ```bash
   crontab -e
   ```
2. Add the following line to schedule the script:

   ```
   */5 * * * * /path/to/db_health_check.sh
   ```

   This will run the health check script every 5 minutes and append the results to the log file.

### Monitoring and Alerts

Once you have set up health checks, you can extend the system to monitor database statuses and even trigger alerts if a failure is detected. For example, you could use `mail` or another alerting system to notify the system administrator if any database is down.

### Disaster Recovery Plan (Partial Documentation)

This section outlines a Disaster Recovery (DR) plan for a junior sysadmin, focusing on basic recovery processes for handling service failures and minimizing downtime. It covers the scenarios of data loss, service failure, firewall misconfigurations, and system loss. While it doesn't address full High Availability (HA) or replication, it provides a structured approach for restoring services and recovering data.

---

### 📌 Disaster Recovery Scenarios Covered

* **Data loss / DB corruption**
* **Service crash or failure to start**
* **Firewall misconfiguration / accidental lockout**
* **Full system loss (worst case)**

---

### 📘 1. **PostgreSQL Disaster Recovery**

#### Scenario: **Data Corruption or Accidental Deletion**

* **Backups stored in:** `/var/backups/postgres/`
* **Command to restore (cluster-level):**

  ```bash
  sudo -u postgres psql -f /var/backups/postgres/pg_backup_<timestamp>.sql
  ```

#### Scenario: **PostgreSQL Service Won’t Start**

* **Check logs:**

  ```bash
  journalctl -u postgresql
  ```

  Or:

  ```bash
  tail -f /var/lib/postgresql/data/log/pg.log
  ```

* **To restart PostgreSQL:**

  ```bash
  sudo systemctl restart postgresql
  ```

* **If it fails to restart:**

  * Check the `postgresql.conf` file for configuration issues.
  * Check disk space.
  * Look for corrupted files in the `PGDATA` directory.

#### Scenario: **PGDATA Wiped or Host is Dead**

* **Recovery Steps:**

  1. Provision a new VM or container.
  2. Reinstall PostgreSQL.
  3. Restore from the latest backup file:

     ```bash
     sudo -u postgres psql -f /var/backups/postgres/pg_backup_<timestamp>.sql
     ```
  4. Verify the ownership of the data directory:

     ```bash
     chown -R postgres:postgres /var/lib/postgresql
     ```

---

### 📘 2. **MariaDB Disaster Recovery**

#### Scenario: **Data Corruption or Loss**

* **Backups stored in:** `/var/backups/mysql/`
* **Command to restore:**

  ```bash
  mysql -u root -p < /var/backups/mysql/mysql_backup_<timestamp>.sql
  ```

#### Scenario: **MariaDB Service Won’t Start**

* **Check status:**

  ```bash
  systemctl status mysql
  ```

* **Check logs:**

  ```bash
  journalctl -u mysql
  ```

  Or:

  ```bash
  tail -f /var/log/mysql/error.log
  ```

* **Configuration issues:** Check the config files in `/etc/mysql/mariadb.conf.d/`.

* **Restart MariaDB:**

  ```bash
  sudo systemctl restart mysql
  ```

#### Scenario: **Total Loss Recovery**

* **Steps:**

  1. Reinstall MariaDB.
  2. Recreate the user `testuser` and reset the password.
  3. Restore the database:

     ```bash
     mysql -u root -p < /var/backups/mysql/mysql_backup_<timestamp>.sql
     ```
  4. Re-apply `GRANT` statements (ensure these are in the backup or reapply manually).

---

### 📘 3. **MongoDB Disaster Recovery**

#### Scenario: **Lost or Corrupted Database**

* **Backups stored in:** `/var/backups/mongodb/`
* **Command to restore:**

  ```bash
  mongorestore /var/backups/mongodb/mongodump_<timestamp>
  ```

#### Scenario: **MongoDB Service Fails to Start**

* **Check logs:**

  ```bash
  journalctl -u mongod
  ```

  Or:

  ```bash
  tail -f /var/log/mongodb/mongod.log
  ```

* **Recheck MongoDB configuration:** Check the configuration file `/etc/mongod.conf`.

* **Restart MongoDB:**

  ```bash
  sudo systemctl restart mongod
  ```

#### Scenario: **Total Failure (Full Recovery)**

* **Steps:**

  1. Reinstall MongoDB.
  2. Restore from the latest `mongodump`.
  3. Recreate users and permissions if stored outside of the dump scope.

---

### 🛡️ 4. **Firewall Lockout (UFW Misconfiguration)**

#### Problem:

Accidentally blocked SSH or remote DB access due to misconfigured firewall rules.

#### Solution:

* **From local console:**

  ```bash
  sudo ufw disable
  ```
* **From rescue mode (cloud VPS):**

  1. Mount the disk.
  2. Edit `/etc/ufw/ufw.conf` and set `ENABLED=no`.

#### Prevention:

* Add the following rule during setup:

  ```bash
  ufw allow OpenSSH
  ```

* **Automate with Ansible:**
  Save the firewall rule configurations in an Ansible playbook to reapply easily across systems.

---

### 💀 5. **Total System Loss / VM Deleted**

#### Recovery Plan:

1. Provision a new host (local VM or cloud).
2. Reinstall services using Ansible playbooks or installation scripts.
3. Restore backups from external drives or cloud storage (ensure these exist and are updated).
4. **Post-install configuration:**

   * Set firewall rules.
   * Recreate user accounts.
   * Apply the backup data.
   * Run health checks on all services.

---

### 🛠️ **Verification Tasks (Post-DR)**

* Verify database access and run queries to test data availability.
* Check if slow query logs are re-enabled.
* Confirm that cron jobs for backups are correctly configured.
* Review UFW rules to ensure no misconfigurations.
* Re-run the `check_services.sh` script to verify all services are running.

---

### ❗ **Disaster Recovery Gaps (To Be Improved)**

* **Offsite Backups:** No offsite backups yet—currently, all backups are local.
* **Backup Encryption:** Backups are not encrypted, posing a potential security risk.
* **WAL Archiving (PostgreSQL):** No Write-Ahead Logging (WAL) archiving or Point-in-Time Recovery (PITR) setup for PostgreSQL.
* **Automated Restore Testing:** There is no automated process to test restores; manual testing is required.
* **Replication / Clustering:** No replication or clustering setup for high availability yet.

---

### ✅ **Conclusion**

For a junior sysadmin or lab scenario, this DR plan is a good starting point. It outlines clear recovery paths, backup strategies, and steps to identify and fix root causes. Though there are areas for improvement (such as offsite backups and encryption), the current plan addresses the most critical failure scenarios and provides a structured approach to minimize downtime and data loss.

