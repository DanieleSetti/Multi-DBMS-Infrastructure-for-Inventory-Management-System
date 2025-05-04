# 🧰 Multi-DBMS Infrastructure Lab

This repository simulates the backend infrastructure of a fictional **Enterprise Inventory & Asset Management System**. The system is designed to manage a mid-sized enterprise’s hardware and software assets, tracking their lifecycle, inventory logs, vendor relations, and audit history.

The project demonstrates the integration and management of **PostgreSQL**, **MySQL**, and **MongoDB**, along with automation, security, backup strategies, and disaster recovery solutions. It aims to simulate the technical challenges and solutions typically faced by a **Database Administrator (DBA)** in a multi-database, enterprise-level environment.

🎯 **Goal**: To showcase comprehensive **system administration** and **DBA** skills through a fully automated and secure infrastructure, complete with backup/recovery, performance tuning, and disaster recovery.

---

## 📝 Table of Contents

1. [Overview](#overview)
2. [Technologies Used](#technologies-used)
3. [Setup Instructions](#setup-instructions)
   - [Prerequisites](#prerequisites)
   - [Installation Steps](#installation-steps)
4. [Architecture](#architecture)
5. [Features & Functionality](#features-functionality)
6. [Backup & Recovery](#backup-recovery)
7. [Security & Access Control](#security-access-control)
8. [Performance Tuning](#performance-tuning)
9. [Disaster Recovery](#disaster-recovery)
10. [Future Improvements](#future-improvements)
11. [Contributing](#contributing)
12. [License](#license)

---

### Technologies Used

```markdown
## 🛠️ Technologies Used

| Technology     | Enterprise Justification                                           |
| -------------- | ------------------------------------------------------------------ |
| **Linux (Ubuntu 22.04)** | Multi-platform environments are standard in large orgs |
| **Ansible**    | For provisioning, updates, and consistent DBMS deployment          |
| **PostgreSQL** | Open-source relational DB used for core business data             |
| **MySQL**      | Open-source relational DB for handling additional enterprise data |
| **MongoDB**    | NoSQL DB used for flexible or evolving schemas                     |
| **Database Tuning** | Simulated slow queries, indexing, execution plans            |
| **Backup & Recovery** | Enterprise requirement — planned failure and restore tests  |
| **Security & Access Control** | Enforced user roles and permissions             |
| **Automation** | Ansible for deployment automation                                  |
| **Disaster Recovery** | Basic outline for DR process                            |
```

---

### Architecture

In this section, you'll briefly describe the architecture and how the components work together. I suggest adding a diagram here, but let's first focus on the content.

## 🏗️ Architecture

The infrastructure is built using **Ubuntu 22.04** VMs, managed by **Ansible**. The backend consists of multiple DBMS:
- **PostgreSQL**: For managing structured business data.
- **MySQL**: For handling additional relational data.
- **MongoDB**: For NoSQL data storage with flexible schema designs.

The setup includes:
- **SSH Access**: Secure access to all VMs using SSH keys.
- **Ansible Automation**: For provisioning, updates, and configuration of DBMS.
- **Firewall**: Enforced via UFW to restrict access to necessary ports.
- **Backup & Recovery**: Scheduled with cron jobs and stored in secure locations.
- **Performance Tuning**: Indexing and query optimization in PostgreSQL and MySQL.
- **Security**: RBAC applied to each DBMS for strict user management.

---

### Features & Functionality

This section will describe the main features and what’s implemented, followed by the `Backup & Recovery` and `Security & Access Control` sections.

## 🎯 Features & Functionality

- **DBMS Setup**: Automated deployment of PostgreSQL, MySQL, and MongoDB.
- **Schema Creation**: Enterprise-level schemas for Inventory and Asset Management.
- **User Management**: Role-based access control (RBAC) with secure user setups.
- **Backup & Recovery**: Automated backup scripts using cron and secure storage.
- **Performance Tuning**: Indexing, slow query log analysis, and query optimization.
- **Security Hardening**: Disable root, enforce SSH key-based authentication, and firewall rules.

### 🔒 Security & Access Control

- Role-based access (admin, read-only, app users)
- Secure DB connections with SSL/TLS
- IP whitelisting and firewall rules via UFW
- Password policies and account lockout mechanisms

---

**Next Steps:**
- **Backup & Recovery**: Expand on how backups are handled and documented.
- **Disaster Recovery**: Document your basic DR plan.
- **Polishing**: Add any final thoughts and contributions.

Let me know if you'd like to adjust any sections, or if we should proceed with adding the next parts!
