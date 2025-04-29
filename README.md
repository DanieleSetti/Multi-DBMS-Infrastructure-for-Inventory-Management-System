# 🗄️ Multi-DBMS Infrastructure Lab – Prepared for Kyndryl

## 📘 Project Overview

This lab simulates the deployment and administration of a multi-database infrastructure for a fictional mid-size **Enterprise Inventory Management System**, with a focus on real-world DBA responsibilities.  
It is built to reflect operational environment: multiple DBMS platforms, automation, secure access, disaster recovery, and performance considerations — all designed to mirror enterprise-scale challenges.

---

## 🔧 Objective

To showcase hands-on competence in:
- Installing and configuring relational and non-relational DBMS
- Automating deployments with Ansible
- Managing users and enforcing role-based security policies
- Designing backup and restore strategies
- Simulating indexing and query tuning
- Planning for disaster recovery and scalability

> This lab complements my previous **Secure LAN Infrastructure for R&D** project by transitioning from system-level administration to **dedicated database infrastructure engineering**.

---

## 🏗️ Phase 1 – Core Setup (✔️ Completed)

- PostgreSQL and MySQL installed on Linux VMs via Ansible
- Secure user management (admin and application roles)
- Minimal hardening (bind to localhost, remove remote root)
- Backups simulated using `pg_dump` and `mysqldump`
- Sample data for indexing and query optimization scenarios

---

## 🧩 Phase 2 – Planned Features (WIP)

- [ ] **MongoDB Deployment** – flexible NoSQL layer for product metadata
- [ ] **Oracle XE Setup** – simulate legacy enterprise dependency
- [ ] **Windows Server Deployment** – alternative platform simulation
- [ ] **Role-Based Access Control** – least-privilege and read-only roles
- [ ] **Monitoring Stack** – Prometheus + Grafana integration (Linux only)
- [ ] **Security Hardening** – audit logging, encryption at rest (where applicable)
- [ ] **Disaster Recovery Drill** – simulate partial and full failure scenarios
- [ ] **Capacity Planning** – mock reports and sizing documentation
- [ ] **Automated Provisioning** – refactor all DBMS setups into reusable Ansible roles

---

## 🗂️ Project Structure

```bash
/
├── ansible/
│   ├── inventory
│   ├── postgresql.yml
│   └── mysql.yml
├── backups/
│   ├── postgres/
│   └── mysql/
├── sql/
│   ├── sample_data.sql
│   ├── indexing_example.sql
├── docs/
│   ├── security.md
│   └── capacity_plan.md
└── README.md
