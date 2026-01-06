## 📌 3-Tier Architecture on AWS with Load Balancer & Auto Scaling

This project demonstrates the design and implementation of a **highly available, scalable, and secure 3-tier web architecture on AWS** using core AWS services. The architecture follows industry best practices by separating the application into **Web, Application, and Database tiers**, each deployed in isolated subnets within a custom VPC.

### 🔧 Architecture Overview

* **Web Tier**
  Deployed on EC2 instances running **Nginx**, placed behind an **internet-facing Application Load Balancer** to handle incoming client traffic.

* **Application Tier**
  Runs **PHP-based application logic** on EC2 instances, fronted by an **internal Application Load Balancer** to ensure secure internal communication.

* **Database Tier**
  Uses **Amazon RDS (MySQL)** deployed in private subnets, accessible only from the application tier for enhanced security.
  Nice, this is the **missing polish** every good repo needs.
Below is a **copy-paste–ready “Architecture Diagram (Text)” section** you can drop straight into your **README.md**. It’s clean, visual (even without an image), and interview-friendly.

---

## 🏗️ Architecture Diagram (Text Representation)


                         ┌───────────────────────────┐
                         │          Clients          │
                         │     (Browser / Users)     │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                  ┌──────────────────────────────────────┐
                  │  Internet Facing Application Load    │
                  │           Balancer (ALB)             │
                  └─────────────┬─────────────┬──────────┘
                                │             │
                                ▼             ▼
                ┌────────────────────┐   ┌────────────────────┐
                │   Web EC2 (AZ-A)   │   │   Web EC2 (AZ-B)   │
                │   Nginx Web Tier   │   │   Nginx Web Tier   │
                └──────────┬─────────┘   └──────────┬─────────┘
                           │                        │
                           └─────────────┬──────────┘
                                         ▼
                 ┌──────────────────────────────────────┐
                 │     Internal Application Load        │
                 │           Balancer (ALB)             │
                 └─────────────┬─────────────┬──────────┘
                               │             │
                               ▼             ▼
              ┌────────────────────┐   ┌────────────────────┐
              │   App EC2 (AZ-A)   │   │   App EC2 (AZ-B)   │
              │   PHP App Tier     │   │   PHP App Tier     │
              └──────────┬─────────┘   └──────────┬─────────┘
                         │                        │
                         └─────────────┬──────────┘
                                       ▼
                 ┌──────────────────────────────────────┐
                 │        Amazon RDS (MySQL)            │
                 │        Database Tier (Private)       │
                 │      Multi-AZ / Subnet Group         │
                 └──────────────────────────────────────┘


---

## 🔐 Network & Security Layout


VPC (10.0.0.0/16)
│
├── Public Subnets (AZ-A, AZ-B)
│   └── Internet-facing ALB
│
├── Web Tier Subnets (AZ-A, AZ-B)
│   └── Web EC2 Instances (Auto Scaling Group)
│
├── App Tier Subnets (AZ-A, AZ-B)
│   └── App EC2 Instances (Auto Scaling Group)
│
└── DB Subnets (AZ-A, AZ-B)
    └── Amazon RDS MySQL (Private, No Public Access)


---

## 🔄 Traffic Flow Explanation

1. Users access the application via a browser.
2. Requests hit the **internet-facing Application Load Balancer**.
3. Traffic is distributed to **Web Tier EC2 instances (Nginx)**.
4. Web tier forwards requests to the **internal Application Load Balancer**.
5. Internal ALB routes traffic to **Application Tier EC2 instances (PHP)**.
6. Application tier communicates securely with **Amazon RDS MySQL**.
7. Response flows back through the same path to the user.

---

## 📌 Key Architecture Principles

* **High Availability:** Multi-AZ deployment across all tiers
* **Scalability:** Independent Auto Scaling Groups for Web & App tiers
* **Security:** Database tier isolated in private subnets
* **Separation of Concerns:** Web, App, and DB tiers independently managed


### ⚙️ Key AWS Services Used

* Amazon VPC (Custom networking with public & private subnets)
* EC2 (Web & App servers)
* Application Load Balancer (Public & Internal)
* Auto Scaling Groups
* Amazon RDS (MySQL)
* Security Groups & Route Tables
* Amazon Machine Images (AMI)

### 🚀 Features

* High availability across multiple Availability Zones
* Auto scaling for Web and Application tiers
* Secure network isolation using private subnets
* Layered security with tier-specific security groups
* Fault tolerance with health checks and load balancing

### 🎯 Purpose

This project is built to:

* Understand real-world AWS architecture patterns
* Practice deploying scalable and secure cloud infrastructure
* Serve as a reference for interviews and production-ready designs

