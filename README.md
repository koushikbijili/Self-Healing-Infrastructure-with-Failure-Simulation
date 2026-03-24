# 🚀 AWS Self-Healing Infrastructure with Failure Simulation

> Designing infrastructure that assumes failure — and recovers automatically.

---

## 📌 Overview

This project demonstrates a **self-healing web architecture on AWS** using:

* Auto Scaling Group (ASG)
* Application Load Balancer (ALB)
* Multi-AZ deployment
* CloudWatch monitoring
* SNS alerting

Failures were intentionally introduced to validate automated recovery behavior under real conditions.

---

## 🏗 Architecture

```
[User]
   ↓
[Route 53 (DNS resolves to ALB)]
   ↓
[Application Load Balancer]
   ↓
[Forward to Healthy EC2 (Target Group)]
   ↓
[Response to User]

```

![project3](https://github.com/user-attachments/assets/ae6c9ebd-808a-417d-889d-2010abd57fc2)

**(with failure handling — more realistic)**

```
[User]
   ↓
[Route 53 (DNS → ALB)]
   ↓
[Application Load Balancer]
   ↓
[Healthy Targets Available?]

   ├── No → [ASG terminates unhealthy EC2 and launches new ones]
   │
   └── Yes
         ↓
   [Forward request to EC2 (Nginx)]
         ↓
   [Response to User]
```

# 🛠 Implementation

---

## 1️⃣ Base EC2 Setup

* Ubuntu 22.04
* t2.micro
* Ports opened: 22 (SSH), 80 (HTTP)

Installed Nginx:

```bash
sudo apt update
sudo apt install nginx -y
```

<img width="1704" height="443" alt="Screenshot 2026-02-27 095109" src="https://github.com/user-attachments/assets/e5758456-7e38-4afc-aeb0-7f48dc792fc6" />


---

## 2️⃣ Create Golden AMI

Created a custom AMI from the configured EC2 instance:

**Name:** `self-healing-nginx-ami`

Purpose:

* Enables identical instance recreation
* Supports immutable infrastructure model

<img width="1687" height="985" alt="Screenshot 2026-02-27 095042" src="https://github.com/user-attachments/assets/530558a0-0c95-47cc-95fb-4a57db4269b5" />

---

## 3️⃣ Launch Template & Auto Scaling Group

ASG Configuration:

* Minimum capacity: 1
* Desired capacity: 2
* Maximum capacity: 3
* Multi-AZ enabled

Ensures automatic replacement of failed instances.

<img width="1882" height="846" alt="Screenshot 2026-02-27 121739" src="https://github.com/user-attachments/assets/118bdb61-7c5a-429c-804a-e6f53ee85d7e" />

<img width="1858" height="853" alt="image" src="https://github.com/user-attachments/assets/bb866121-a71d-472a-8d3a-a8bb8dd9939d" />

---

## 4️⃣ Application Load Balancer

* HTTP Listener (Port 80)
* Health Check Path: `/`
* Routes traffic only to healthy targets
<img width="1495" height="676" alt="image" src="https://github.com/user-attachments/assets/89bfdce6-db4d-4fc8-8d5a-be6e06eb18a8" />

---

# 🔥 Failure Simulation

---

## 🧨 Scenario 1 — Instance Termination

**Action:** Manually terminated EC2 instance

**Result:**
Auto Scaling detected capacity loss and launched a replacement automatically (~90 seconds).

<img width="1685" height="424" alt="Screenshot 2026-02-27 121458" src="https://github.com/user-attachments/assets/54f0387c-363b-4f19-8229-842008c81935" />
<img width="1706" height="439" alt="Screenshot 2026-02-27 121613" src="https://github.com/user-attachments/assets/167e7b8f-513e-47b9-8e17-e02fb7d17150" />

---

## 🧨 Scenario 2 — Nginx Crash

```bash
sudo systemctl stop nginx
```
<img width="1457" height="260" alt="Screenshot 2026-02-27 115424" src="https://github.com/user-attachments/assets/afbc90c3-b719-49f5-b36b-85eee3fae246" />
<img width="1749" height="810" alt="Screenshot 2026-02-27 122316" src="https://github.com/user-attachments/assets/5662adf9-33ab-4a91-8f17-16e034795ce8" />


**Result:**

* Target marked unhealthy
* Traffic automatically routed to healthy instance (~30 seconds)

---

## 🧨 Scenario 3 — CPU Stress Test

```bash
sudo apt install stress -y
stress --cpu 2 --timeout 300
```
<img width="761" height="66" alt="Screenshot 2026-02-27 122603" src="https://github.com/user-attachments/assets/4bbf563b-6015-4de0-b49f-124e2a2b54c6" />

<img width="1896" height="554" alt="Screenshot 2026-02-27 141532" src="https://github.com/user-attachments/assets/009054af-e466-44c8-92ef-6dccdb4afd70" />


**Result:**

* CPU exceeded 70% threshold
* CloudWatch alarm triggered
* ASG scaled out automatically (~2 minutes)
  
<img width="761" height="66" alt="Screenshot 2026-02-27 122603" src="https://github.com/user-attachments/assets/4436d159-c68b-44f8-be46-58452ca819b5" />

<img width="1918" height="874" alt="Screenshot 2026-02-27 142154" src="https://github.com/user-attachments/assets/0a5a2d97-1e29-4e9d-bb29-95b32b533c60" />

---

# 📊 Monitoring & Alerts

* CloudWatch CPU metrics enabled
* Alarm threshold set at 70%
* SNS email notifications configured
* Auto Scaling policy attached to alarm

<img width="1898" height="810" alt="Screenshot 2026-02-27 155139" src="https://github.com/user-attachments/assets/b2ecc6ba-901d-4b38-b481-c8cb0f9c9559" />
<img width="1335" height="535" alt="Screenshot 2026-02-27 155421" src="https://github.com/user-attachments/assets/9dd68068-82cb-4aad-846d-877d5e932e79" />
<img width="1735" height="862" alt="Screenshot 2026-02-27 155734" src="https://github.com/user-attachments/assets/ee184e5a-0dde-4f8d-82bf-e1e3c2cbd1a4" />




---

# ⏱ Recovery Summary

| Failure Type        | Detection Mechanism     | Automated Action     | Recovery Time |
| ------------------- | ----------------------- | -------------------- | ------------- |
| Instance Terminated | ASG capacity monitoring | New EC2 launched     | ~90 sec       |
| Web Server Crash    | ALB health check        | Traffic rerouted     | ~30 sec       |
| CPU Spike           | CloudWatch alarm        | Horizontal scale-out | ~2 min        |

---
 👨‍💻 Author

Koushik Bijili
