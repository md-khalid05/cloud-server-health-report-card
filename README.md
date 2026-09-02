# ☁️ AWS Cloud Server Health Card

[![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/ec2/)
[![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-0078D4?style=for-the-badge&logo=windows&logoColor=white)](https://www.microsoft.com/windows-server)
[![IIS](https://img.shields.io/badge/IIS-Web%20Server-0078D4?style=for-the-badge&logo=microsoft)](https://www.iis.net/)
[![PowerShell](https://img.shields.io/badge/PowerShell-Automation-5391FE?style=for-the-badge&logo=powershell&logoColor=white)](https://learn.microsoft.com/powershell/)
[![Status](https://img.shields.io/badge/Deployment-9%2F9%20PASS-success?style=for-the-badge)](#-verification-results)

> **A small AWS infrastructure lab with a real server, a real web endpoint, automated health collection, and deliberate failure/recovery tests.**

---

## 👀 Recruiter Quick Scan

|                     |                                       |
| ------------------- | ------------------------------------- |
| ☁️ **Cloud**        | AWS EC2                               |
| 🖥️ **Server**       | Windows Server 2022 Datacenter        |
| 🌐 **Web Server**   | IIS                                   |
| ⚙️ **Automation**   | PowerShell + Windows Scheduled Task   |
| ❤️ **Monitoring**   | `status.json` + 1-minute heartbeat    |
| 🔥 **Networking**   | AWS Security Group + Windows Firewall |
| 🧪 **Validation**   | 9 automated checks                    |
| ♻️ **Resilience**   | Reboot + EC2 stop/start tested        |
| ✅ **Final result** | **9/9 PASS**                          |

### What I built

```text
                🌍 Laptop
                    │
              HTTP :80
                    │
                    ▼
          ☁️ AWS EC2 Instance
          ┌─────────────────────┐
          │ Windows Server 2022 │
          │                     │
          │  🔥 AWS SG          │
          │  🔥 Windows FW      │
          │          │          │
          │          ▼          │
          │      IIS :80        │
          │          │          │
          │          ▼          │
          │     HealthCard      │
          │          │          │
          │          ▼          │
          │   data/status.json  │
          └──────────▲──────────┘
                     │
              PowerShell
                     │
             Scheduled Task
               every 1 min
```

**In one sentence:** PowerShell collects the server's health → writes JSON → IIS serves it → a scheduled task keeps it fresh → the verifier checks the whole deployment.

---

# 🚀 Live Deployment Snapshot

These are the actual values from the completed lab:

| Property                | Value                          |
| ----------------------- | ------------------------------ |
| **Cloud**               | AWS                            |
| **Region**              | `eu-north-1`                   |
| **Availability Zone**   | `eu-north-1a`                  |
| **Instance**            | `t3.micro`                     |
| **OS**                  | Windows Server 2022 Datacenter |
| **Hostname**            | `EC2AMAZ-I9L8E4D`              |
| **Private IPv4**        | `172.31.37.195`                |
| **IIS Site**            | `HealthCard`                   |
| **Port**                | `80`                           |
| **Scheduled Task**      | `HealthCard-Collector`         |
| **Task Account**        | `SYSTEM`                       |
| **Collector Frequency** | Every 1 minute                 |

> **Public IP is intentionally not treated as permanent.** The lab observed `16.171.146.72` before an EC2 stop/start and `13.53.174.154` after it, demonstrating ephemeral public IPv4 behavior.

---

# 🧩 How It Works

### 1️⃣ Provision

An AWS EC2 Windows Server instance provides the compute environment.

### 2️⃣ Configure IIS

`1-Setup-IIS.ps1`:

- Installs IIS
- Enables HTTP port 80 in Windows Firewall
- Copies the website
- Creates the `HealthCard` IIS site
- Starts the site
- Verifies local HTTP access

### 3️⃣ Collect health data

`2-Collect-Status.ps1` gathers:

- Hostname
- OS
- Private IPv4
- Public IPv4
- CPU load
- Memory
- Disk usage
- IIS state
- Deployment metadata
- Heartbeat timestamps

and writes:

```text
C:\inetpub\HealthCard\data\status.json
```

### 4️⃣ Keep it alive

`3-Schedule-Collector.ps1` creates:

```text
HealthCard-Collector
```

It runs as **SYSTEM**, every minute, with a startup trigger.

### 5️⃣ Expose it

The laptop reaches:

```text
http://<public-ip>/
```

Traffic must pass:

```text
Laptop
  ↓
AWS Security Group :80
  ↓
Windows Firewall :80
  ↓
IIS :80
  ↓
HealthCard
```

### 6️⃣ Verify everything

`4-Verify.ps1` checks nine critical conditions before submission.

---

# 📁 Project Structure

```text
cloud-server-health-card/
│
├── 📄 README.md
├── 📄 deployment.json
│
├── 📂 scripts/
│   ├── 1-Setup-IIS.ps1
│   ├── 2-Collect-Status.ps1
│   ├── 3-Schedule-Collector.ps1
│   └── 4-Verify.ps1
│
└── 📂 site/
    ├── index.html
    ├── health.txt
    ├── web.config
    ├── 📂 css/
    │   └── styles.css
    └── 📂 data/
        └── status.json
```

---

# 🛠️ Run It Yourself

> Run PowerShell **as Administrator** on the Windows EC2 instance.

### Step 1 — Install IIS

```powershell
cd C:\lab\cloud-server-health-card-main\scripts
.\1-Setup-IIS.ps1
```

Expected:

```text
[ok] W3SVC is Running
[ok] Site started
[ok] HTTP 200 from http://localhost:80/
```

---

### Step 2 — Generate server status

```powershell
.\2-Collect-Status.ps1
```

Expected:

```text
Wrote C:\inetpub\HealthCard\data\status.json
```

Check it:

```powershell
Invoke-WebRequest http://localhost/data/status.json -UseBasicParsing
```

Expected:

```text
StatusCode : 200
```

---

### Step 3 — Schedule the heartbeat

```powershell
.\3-Schedule-Collector.ps1
```

Expected:

```text
[ok] Runs every 1 minute(s) as SYSTEM
```

Check the task:

```powershell
Get-ScheduledTask -TaskName 'HealthCard-Collector' |
    Select-Object TaskName, State
```

---

### Step 4 — Reach it from your laptop

Find the current EC2 public IPv4 in AWS and open:

```text
http://<public-ip>/
```

Test port 80 from macOS:

```bash
nc -vz <public-ip> 80
```

Expected:

```text
Connection to <public-ip> port 80 [tcp/http] succeeded!
```

---

### Step 5 — Run the final verifier

```powershell
.\4-Verify.ps1
```

Goal:

```text
9 / 9 PASS
```

---

# 🧪 Verification Results

The completed deployment produced:

```text
  Deployment check - EC2AMAZ-I9L8E4D

Result Check
------ -----
PASS   IIS role installed
PASS   W3SVC running
PASS   Site 'HealthCard' started
PASS   Site bound to port 80
PASS   deployment.json installed and edited
PASS   status.json exists
PASS   status.json fresher than 3 minutes
PASS   Site answers HTTP 200 on localhost
PASS   status.json is served over HTTP

All checks passed.
```

## ✅ 9/9 Checks Passed

| Check                    | Result |
| ------------------------ | :----: |
| IIS role installed       |   ✅   |
| W3SVC running            |   ✅   |
| HealthCard started       |   ✅   |
| Port 80 binding          |   ✅   |
| Deployment configuration |   ✅   |
| `status.json` exists     |   ✅   |
| Collector data fresh     |   ✅   |
| Local HTTP 200           |   ✅   |
| JSON served over HTTP    |   ✅   |

---

# ❤️ Heartbeat Proof

The collector successfully produced multiple `ok: true` pulses.

Example observed pulses:

```text
19:55:58Z  ✅
20:05:02Z  ✅
20:12:12Z  ✅
20:12:37Z  ✅
20:13:34Z  ✅
20:14:36Z  ✅
20:15:34Z  ✅
20:16:35Z  ✅
```

The Health Card visually displayed multiple heartbeat steps, confirming that the scheduled collector was actively updating the server state.

---

# 🔥 Failure Test

A good infrastructure system should not only work — it should fail predictably and recover.

The collector was deliberately disabled:

```powershell
Disable-ScheduledTask -TaskName 'HealthCard-Collector'
```

After the data became stale, the task was restored:

```powershell
Enable-ScheduledTask -TaskName 'HealthCard-Collector'
```

The timestamp started advancing again.

### Result

```text
Collector disabled
      ↓
Data becomes stale
      ↓
Collector re-enabled
      ↓
New status.json timestamp
      ↓
Heartbeat recovered ✅
```

---

# ♻️ Recovery Testing

## Windows Reboot

The VM was rebooted:

```powershell
Restart-Computer
```

After reconnecting:

```text
W3SVC                 Running  ✅
HealthCard-Collector  Ready    ✅
Collector timestamp   Fresh    ✅
```

Observed timestamp after recovery:

```text
2026-09-01T20:52:32Z
```

### What this proves

IIS and the collector are configured to recover automatically after a Windows reboot.

---

## AWS EC2 Stop → Start

The instance was stopped and started from the AWS console.

### Public IP observed

```text
Before: 16.171.146.72
After:  13.53.174.154
```

The instance itself remained the same:

```text
Hostname:    EC2AMAZ-I9L8E4D
Private IP:  172.31.37.195
```

After recovery:

```text
W3SVC       Running  ✅
status.json Fresh    ✅
```

Observed collector timestamp:

```text
2026-09-01T21:09:33Z
```

### Key lesson

A normal EC2 public IPv4 address is **ephemeral**. If an application needs a stable public address, use an **Elastic IP** or another stable endpoint.

---

# 🌐 Networking in 20 Seconds

There are two IP addresses because they serve different purposes:

```text
Private IP
172.31.37.195
      │
      └── Used inside the AWS VPC


Public IP
16.171.146.72
      │
      └── Used to reach the server from the Internet
```

`ipconfig` only sees the Windows network interface, so it reports the private address rather than the AWS-managed public IPv4.

The collector discovers the public address using:

```text
https://api.ipify.org
```

---

# 🔥 Two Firewalls

Inbound HTTP traffic crosses two security layers:

```text
Internet
   │
   ▼
AWS Security Group
   │
   ▼
Windows Defender Firewall
   │
   ▼
IIS :80
```

Both need to allow TCP port `80`.

### AWS

```text
Security Group
→ Inbound
→ TCP
→ Port 80
```

### Windows

```powershell
Get-NetFirewallRule -DisplayName 'Lab HTTP 80 In'
```

If port 80 is open in only one firewall, the other layer can still block the request.

---

# 👤 Why SYSTEM?

The scheduled task runs as `SYSTEM` because the collector is a **machine-level background process**.

This means it can:

- Start without an interactive login
- Continue after Administrator logs out
- Run after reboot
- Access system-level information
- Avoid storing an Administrator password

---

# 📸 Proof of Work

> **Recommended repository layout:** save the six lab screenshots under `docs/screenshots/`.

```text
docs/
└── screenshots/
    ├── checkpoint-1-iis.png
    ├── checkpoint-2-collector.png
    ├── checkpoint-3-scheduler.png
    ├── checkpoint-4-heartbeat.png
    ├── checkpoint-5-laptop-access.png
    └── checkpoint-6-verification.png
```

## Checkpoint 1 — IIS Setup

**Proof:** IIS installation, W3SVC status, HealthCard deployment and local HTTP response.

![Checkpoint 1 - IIS Setup](docs/screenshots/checkpoint-1-iis1.png)
![Checkpoint 1 - IIS Setup](docs/screenshots/checkpoint-1-iis2.png)
![Checkpoint 1 - IIS Setup](docs/screenshots/checkpoint-1-iis3.png)

---

## Checkpoint 2 — Server Health Collector

**Proof:** `status.json` generated with deployment and machine information.

![Checkpoint 2 - Collector](docs/screenshots/checkpoint-2-collector.png)

---

## Checkpoint 3 — Scheduled Collector

**Proof:** `HealthCard-Collector` registered as a SYSTEM scheduled task running every minute.

![Checkpoint 3 - Scheduler](docs/screenshots/checkpoint-3-scheduler.png)

---

## Checkpoint 4 — Heartbeat

**Proof:** Health Card displaying multiple successful collector pulses.

![Checkpoint 4 - Heartbeat](docs/screenshots/checkpoint-4-heartbeat.png)

---

## Checkpoint 5 — Laptop Access

**Proof:** Health Card loaded externally from the laptop with the browser URL visible.

![Checkpoint 5 - Laptop Access](docs/screenshots/checkpoint-5-laptop-access.png)

---

## Checkpoint 6 — Final Verification

**Proof:** all nine automated checks passing.

![Checkpoint 6 - 9/9 Verification](docs/screenshots/checkpoint-6-verification.png)

> **Tip:** Keep the screenshots tightly cropped around the useful evidence. A recruiter should be able to understand each checkpoint in a few seconds.

---

# 🧠 What This Project Demonstrates

### Cloud

- AWS EC2
- Availability Zones
- Public/private networking
- Security Groups
- Ephemeral public IPv4

### Windows / Infrastructure

- Windows Server 2022
- IIS
- Windows Defender Firewall
- Windows Services
- Scheduled Tasks
- SYSTEM execution

### Automation

- PowerShell scripting
- Automated health collection
- Scheduled execution
- JSON generation
- Automated verification

### Troubleshooting

The deployment was tested layer-by-layer:

```text
localhost
   ↓
private IP
   ↓
public port
   ↓
laptop browser
```

This makes failures easier to isolate instead of changing infrastructure randomly.

---

# 📌 Key Takeaways

> **Build → Monitor → Expose → Verify → Break → Recover**

This lab demonstrates a complete mini production-style workflow rather than simply hosting an HTML page on EC2.

The final system:

**runs a real Windows server → hosts a real IIS application → collects real machine metrics → automatically refreshes health data → exposes the service through AWS networking → validates itself → survives reboot → recovers after EC2 stop/start.**

---

## 🏁 Final Status

```text
IIS Deployment             ✅
Health Collector            ✅
Scheduled Heartbeat         ✅
External HTTP Access        ✅
AWS + Windows Firewalls     ✅
9/9 Automated Checks        ✅
Failure Test                ✅
Windows Reboot Recovery     ✅
EC2 Stop/Start Recovery     ✅
Public/Private IP Demo      ✅
```

**Status: COMPLETE ✅**

### 1. Your VM has a public IP address. Why does `ipconfig` not show it?

`ipconfig` only shows the IP addresses assigned to the network interfaces inside the Windows VM. The EC2 instance's public IPv4 address is provided and managed by **AWS at the virtualization/networking layer**, rather than being directly assigned to the Windows network adapter.

Therefore, `ipconfig` shows the private IP:

```text
172.31.37.195
```

while the public IP:

```text
16.171.146.72
```

is associated with the instance externally through AWS networking. The server discovered its public IP by calling `api.ipify.org`.

---

### 2. The outbound call to `api.ipify.org` needed no firewall change, but your laptop's inbound request needed two rules. Why is the default asymmetric?

Firewalls commonly use an **asymmetric default policy**:

- **Outbound traffic** is generally allowed by default so that servers can access updates, APIs, DNS, and other external services.
- **Inbound traffic** is generally restricted by default to prevent unauthorized access to the server.

Therefore, the server could make an outbound HTTPS request to:

```text
api.ipify.org
```

without adding a new firewall rule.

However, when my laptop tried to reach the server on TCP port 80, the connection had to pass through **both the AWS cloud firewall and Windows Firewall**, requiring inbound port 80 to be allowed.

---

### 3. Name the two firewalls you configured. What happens if you open port 80 in only one?

The two firewalls were:

1. **AWS Security Group** — the cloud-level firewall controlling traffic to the EC2 instance.
2. **Windows Defender Firewall** — the operating-system-level firewall on the Windows Server.

Both must allow inbound **TCP port 80**.

If port 80 is opened in only one firewall, the other firewall can still block the connection.

For example:

```text
Laptop
   ↓
AWS Security Group       ❌ blocks
   ↓
Windows Firewall
   ↓
IIS
```

or:

```text
Laptop
   ↓
AWS Security Group       ✅ allows
   ↓
Windows Firewall         ❌ blocks
   ↓
IIS
```

In either case, the browser cannot reach the Health Card.

---

### 4. Why does the scheduled task run as SYSTEM rather than as Administrator?

The collector runs as **SYSTEM** so it can operate automatically as a machine-level task without requiring an interactive Administrator login or storing an Administrator password.

This is useful because the task must:

- Run automatically at startup.
- Continue running when no user is logged in.
- Access system information such as CPU, memory, disk, services, and IIS.
- Run reliably as a background machine task.

Using SYSTEM also avoids tying the heartbeat mechanism to a particular user's interactive session.

---

### 5. After stop/start, what changed and what stayed the same — and why?

After stopping and starting the EC2 instance, the **public IPv4 address changed**.

Before:

```text
16.171.146.72
```

After:

```text
13.53.174.154
```

The **hostname and private IP remained associated with the instance**.

The reason is that an EC2 public IPv4 address is normally **ephemeral**. When an instance is stopped, AWS can release that public address and assign another one when the instance starts again.

The private address belongs to the instance's private network interface, so it remains associated with the instance.

The practical consequence is that applications requiring a stable public address should use an **Elastic IP** or another stable endpoint rather than relying on the default public IPv4 address.

---

### 6. Name one thing you had to do in your cloud's console that would have been different in the other two.

I had to configure an **AWS Security Group inbound rule** to allow:

```text
Protocol: TCP
Port: 80
```

For the same purpose:

- **AWS:** Security Group
- **Azure:** Network Security Group (NSG)
- **GCP:** VPC firewall rule

The networking concept is the same, but the cloud console and firewall mechanism are different.
