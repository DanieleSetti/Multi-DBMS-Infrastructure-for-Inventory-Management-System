# 🗄️ Multi-DBMS Infrastructure Lab

## 📘 Project Overview

This project simulates the infrastructure supporting a fictional Enterprise Inventory & Asset Management System. The goal is to reflect realistic challenges and solutions a DBA faces when managing diverse data workloads across multiple platforms (PostgreSQL, MySQL, Oracle, SQL Server, MongoDB), with attention to automation, security, and operational resilience.

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

## 📚 Table of Contents

* [Project Overview](#project-overview)
* [Objective](#objective)
* [Motivation](#motivation)
* [Goals](#goals)
* [Scripts](#scripts)
* [Database Setup](#database-setup)
* [Backup & Restore](#backup--restore)
* [Performance Testing](#performance-testing)
* [Firewall Configuration](#firewall-configuration)
* [Disaster Recovery](#disaster-recovery)
* [Architecture](#architecture)
* [Usage](#usage)
* [Future Improvements](#future-improvements)

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
