# 🌐 AWS Elastic IP — Static Public IP for EC2 Instances

> **A hands-on AWS lab demonstrating the problem of dynamic public IPs on EC2 instances and how Elastic IP solves it.**

---

## 📋 Table of Contents

- [Overview](#-overview)
- [The Problem — Dynamic Public IPs](#-the-problem--dynamic-public-ips)
- [Why This Matters in Real Projects](#-why-this-matters-in-real-projects)
- [Lab Workflow](#-lab-workflow)
  - [Step 1 — Create EC2 Instance](#step-1--create-ec2-instance)
  - [Step 2 — Observe Dynamic IP Behavior](#step-2--observe-dynamic-ip-behavior)
  - [Step 3 — Allocate an Elastic IP](#step-3--allocate-an-elastic-ip)
  - [Step 4 — Associate Elastic IP with EC2](#step-4--associate-elastic-ip-with-ec2)
  - [Step 5 — Verify Static IP Behavior](#step-5--verify-static-ip-behavior)
- [Results & Comparison](#-results--comparison)
- [Architecture Diagram](#-architecture-diagram)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Conclusion](#-conclusion)
- [Technologies Used](#-technologies-used)

---

## 🔍 Overview

When you launch an EC2 instance on AWS with the default settings, it gets assigned a **public IP address automatically**. This sounds convenient — but there's a catch: **every time you stop and restart the instance, it gets a completely different public IP address.**

This lab walks through:
1. The default (problematic) behavior of EC2 public IPs
2. How **AWS Elastic IP** solves this by providing a **permanent, static public IP**

---

## ⚠️ The Problem — Dynamic Public IPs

By default, EC2 instances are launched with **"Auto-assign Public IP"** enabled. This means:

| Action | Public IP | Private IP |
|---|---|---|
| Instance **Running** | ✅ Assigned (e.g., `3.239.182.158`) | ✅ Stays the same |
| Instance **Stopped** | ❌ Released — IP is gone | ✅ Stays the same |
| Instance **Started again** | ✅ New IP assigned (e.g., `44.200.125.224`) | ✅ Stays the same |

> 🔑 **Key insight:** The **Private IP never changes**. Only the **Public IP** is dynamic and gets reassigned on each start/stop cycle.

---

## 🏢 Why This Matters in Real Projects

Imagine you're running a web server or an API backend on EC2. Here's what happens without a static IP:

- 🔗 **DNS records break** — If you pointed a domain name to your server's IP, it stops working after a restart.
- 🔒 **Firewall rules break** — If clients or partners whitelisted your server's IP, they'll be blocked after it changes.
- 🔧 **Configuration management fails** — Any system that hardcodes the IP (load balancers, monitoring tools, etc.) will lose connection.
- 📞 **Clients can't reach you** — Applications expecting a consistent endpoint will fail.

**Bottom line:** Dynamic IPs are fine for testing, but completely impractical for production or team environments.

---

## 🔬 Lab Workflow

### Step 1 — Create EC2 Instance

A new EC2 instance named `lab-elastic` was launched with the following configuration:

- **Instance type:** `t3.micro`
- **Region / AZ:** `us-east-1a`
- **Auto-assign Public IP:** Enabled ✅
- **Instance ID:** `i-0014fc03d69e032a7`
- **Private IP:** `172.31.15.62` *(this never changes)*

The instance started with an initial public IP of `3.239.182.158`.

📸 **Screenshot — EC2 Instance Running (Initial State):**
![](./assets/Screenshot%202026-05-23%20214510.png)


---

### Step 2 — Observe Dynamic IP Behavior

To demonstrate the problem, the instance was stopped and restarted several times:

**🔴 After stopping the instance:**
- Instance state: `Stopped`
- Public IP: **Gone / Empty** (`—`)
- Private IP: `172.31.15.62` ✅ (unchanged)

📸 **Screenshot — Instance Stopped (Public IP Disappeared):**

![](./assets/Screenshot%202026-05-23%20214607.png)


---

**🟢 After starting the instance again:**
- Instance state: `Running`
- Public IP: **New IP assigned** — `44.200.125.224` *(different from the original!)*
- Private IP: `172.31.15.62` ✅ (still unchanged)

📸 **Screenshot — Instance Restarted (New Public IP Assigned):**

![](./assets/Screenshot%202026-05-23%20214638.png)


> ⚠️ This confirms the problem: the public IP changed from `3.239.182.158` → gone → `44.200.125.224`

---

### Step 3 — Allocate an Elastic IP

An **Elastic IP address** was allocated from AWS's pool of public IPv4 addresses.

- Navigate to: **EC2 Console → Network & Security → Elastic IPs**
- Click **"Allocate Elastic IP address"**
- The Elastic IP `34.232.221.67` was allocated with Allocation ID: `eipalloc-0385ef0c84e8af28f`

📸 **Screenshot — Elastic IP Allocated:**

![](./assets/Screenshot%202026-05-23%20214748.png)


---

### Step 4 — Associate Elastic IP with EC2

The allocated Elastic IP was then **associated** with the `lab-elastic` EC2 instance:

- Select the Elastic IP → **Actions → Associate Elastic IP address**
- Choose the target instance: `i-0014fc03d69e032a7` (`lab-elastic`)
- Confirm association

✅ **Success message:** *"Elastic IP address 34.232.221.67 has been associated with instance i-0014fc03d69e032a7"*

![](./assets/Screenshot%202026-05-23%20214748.png)

After association:
- **Public IP on instance:** `34.232.221.67` (the Elastic IP)
- **Private IP:** `172.31.15.62` (unchanged)

📸 **Screenshot — Elastic IP Successfully Associated:**

![](./assets/Screenshot%202026-05-23%20214815.png)

---

### Step 5 — Verify Static IP Behavior

Now with the Elastic IP attached, the stop/start cycle was tested again:

**🔴 After stopping the instance:**
- Instance state: `Stopped`
- Public IP: **`34.232.221.67`** ✅ *(still there!)*
- Private IP: `172.31.15.62` ✅

📸 **Screenshot — Instance Stopped (Elastic IP Persists):**

![](./assets/Screenshot%202026-05-23%20214907.png)

---

**🟢 After starting the instance again:**
- Instance state: `Running`
- Public IP: **`34.232.221.67`** ✅ *(same IP — no change!)*
- Private IP: `172.31.15.62` ✅

📸 **Screenshot — Instance Restarted (Elastic IP Unchanged):**

![](./assets/Screenshot%202026-05-23%20214923.png)
> ✅ **Problem solved!** The Elastic IP stays the same across stop/start cycles.

---

## 📊 Results & Comparison

| Scenario | Public IP | Private IP | Consistent? |
|---|---|---|---|
| EC2 started (no Elastic IP) | `3.239.182.158` | `172.31.15.62` | — |
| EC2 stopped (no Elastic IP) | ❌ Gone | `172.31.15.62` | ❌ No |
| EC2 restarted (no Elastic IP) | `44.200.125.224` | `172.31.15.62` | ❌ No |
| EC2 running **with Elastic IP** | `34.232.221.67` | `172.31.15.62` | ✅ Yes |
| EC2 stopped **with Elastic IP** | `34.232.221.67` | `172.31.15.62` | ✅ Yes |
| EC2 restarted **with Elastic IP** | `34.232.221.67` | `172.31.15.62` | ✅ Yes |

---

## 🏗️ Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Cloud                            │
│                                                             │
│   ┌──────────────────────────────────────────────────────┐  │
│   │                   VPC (us-east-1)                    │  │
│   │                                                      │  │
│   │   ┌──────────────────────────────────────────────┐   │  │
│   │   │          Availability Zone: us-east-1a       │   │  │
│   │   │                                              │   │  │
│   │   │    ┌─────────────────────────────────────┐   │   │  │
│   │   │    │       EC2 Instance (lab-elastic)     │   │   │  │
│   │   │    │       Type: t3.micro                 │   │   │  │
│   │   │    │                                      │   │   │  │
│   │   │    │  Private IP:  172.31.15.62 (fixed)   │   │   │  │
│   │   │    │  Public IP:   34.232.221.67 ◄────────┼───┼───┼── Elastic IP
│   │   │    └─────────────────────────────────────┘   │   │  │
│   │   └──────────────────────────────────────────────┘   │  │
│   └──────────────────────────────────────────────────────┘  │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │   Elastic IP Pool                                   │   │
│   │   📌 34.232.221.67  →  Allocated & Associated       │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                     🌐 Internet
               (Users/clients connect via
                static IP: 34.232.221.67)
```

**Workflow Summary:**

```
Launch EC2        Stop Instance      Start Again
    │                  │                 │
    ▼                  ▼                 ▼
Public IP:          Public IP:       New Public IP:
3.239.182.158   ❌  (gone)      ❌  44.200.125.224
                                    (different!)

         ── Attach Elastic IP ──▶

Launch EC2        Stop Instance      Start Again
    │                  │                 │
    ▼                  ▼                 ▼
Elastic IP:         Elastic IP:      Elastic IP:
34.232.221.67   ✅  34.232.221.67 ✅ 34.232.221.67
                     (still there!)   (unchanged!)
```

---

## 💡 Key Concepts

| Term | Definition |
|---|---|
| **Public IP** | Temporary IP assigned automatically to an EC2 instance. Released when the instance stops. |
| **Private IP** | Internal IP within the VPC. Permanent and never changes for the lifetime of the instance. |
| **Elastic IP (EIP)** | A static, persistent public IPv4 address that you allocate from AWS and associate with an instance. |
| **Allocation** | Reserving an Elastic IP address from AWS's pool for your account. |
| **Association** | Linking an allocated Elastic IP to a specific EC2 instance or network interface. |

> ⚠️ **Important Billing Note:** AWS charges for Elastic IP addresses that are **allocated but NOT associated** with a running instance. Always release Elastic IPs you're not using to avoid unexpected charges.

---

## 📸 Screenshots

All screenshots are located in the [`screenshots/`](./screenshots/) folder.

| # | Description | File |
|---|---|---|
| 1 | EC2 instance running with initial auto-assigned public IP | `01_ec2_running_initial.png` |
| 2 | Instance stopped — public IP disappeared | `02_ec2_stopped_no_ip.png` |
| 3 | Instance restarted — new public IP assigned | `03_ec2_running_new_ip.png` |
| 4 | Elastic IP allocated from AWS pool | `04_elastic_ip_allocated.png` |
| 5 | Elastic IP successfully associated with EC2 instance | `05_elastic_ip_associated.png` |
| 6 | Instance stopped — Elastic IP remains | `06_ec2_stopped_elastic_ip_persists.png` |
| 7 | Instance restarted — Elastic IP still unchanged | `07_ec2_running_elastic_ip_unchanged.png` |

---

## ✅ Conclusion

This lab clearly demonstrates the difference between a **dynamic public IP** and a **static Elastic IP** on AWS EC2:

- ❌ **Without Elastic IP:** Every stop/start cycle assigns a new, unpredictable public IP — unsuitable for real-world applications.
- ✅ **With Elastic IP:** The public IP remains constant across reboots, stops, and starts — making your server reliably reachable.

**Use Elastic IPs whenever you need:**
- A stable endpoint for your web server or API
- A consistent IP for DNS records or domain mapping
- A whitelisted IP for firewall rules or third-party integrations
- A reliable address for SSH access or remote management

---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| ![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazon-aws) | Virtual server (compute) |
| ![AWS](https://img.shields.io/badge/AWS-Elastic_IP-blue?logo=amazon-aws) | Static public IP management |
| ![AWS](https://img.shields.io/badge/AWS-VPC-purple?logo=amazon-aws) | Virtual network for EC2 |

- **AWS EC2** — Amazon Elastic Compute Cloud (virtual servers)
- **AWS Elastic IP** — Static public IPv4 address service
- **AWS VPC** — Virtual Private Cloud (networking layer)
- **AWS Management Console** — Web-based GUI used throughout this lab

---

> 📝 **Author Note:** This project was completed as a hands-on AWS networking lab to understand public IP behavior and the practical use of Elastic IP addresses in cloud infrastructure.
#   e l a s t i c - i p - l a b  
 