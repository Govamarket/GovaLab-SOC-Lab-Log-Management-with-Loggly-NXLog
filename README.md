# GovaLab-SOC-Lab-Log-Management-with-Loggly-NXLog


![SOC Lab](https://img.shields.io/badge/SOC-Lab-blue) ![Loggly](https://img.shields.io/badge/Loggly-Log%20Management-orange) ![NXLog](https://img.shields.io/badge/NXLog-CE-green) ![Windows](https://img.shields.io/badge/OS-Windows-blue)

> Part of my ongoing journey progressively building a Security Operations Center (SOC) lab at **GovaLab**.

---

## 📌 Overview

This guide walks through setting up **Loggly** as a log management solution to collect, monitor, and analyze **Windows Event Logs** in real time using **NXLog-CE** as the log forwarder.

---

##  What is Loggly?

Loggly is a cloud-based **log management and analysis** platform. It is considered a **semi-SIEM tool** — it handles:

| Feature | Loggly |
|--------|--------|
| Log Collection | ✅ |
| Log Search & Analysis | ✅ |
| Basic Alerting | ✅ |
| Threat Detection / Correlation | ❌ |
| Incident Response | ❌ |

For a full SIEM experience, pair Loggly with tools like **Wazuh** or **Elastic SIEM**.

---

##  Prerequisites

- Windows machine (Vista/2008 or later)
- Loggly account — [Sign up here](https://www.loggly.com)
- NXLog Community Edition — [Download here](https://nxlog.co/downloads/nxlog-ce)

---

##  Setup Guide

### Step 1: Create a Loggly Account
1. Go to [loggly.com](https://www.loggly.com) and create a free account
2. Navigate to **Source Setup → Add Log**
3. Select **Windows** as your platform

### Step 2: Download & Install NXLog-CE
1. Download `nxlog-ce-3.x.xxxx.msi` from [nxlog.co](https://nxlog.co/downloads/nxlog-ce)
2. **Right-click the installer → Run as Administrator**
3. Complete the installation wizard

> ⚠️ **Important:** Always run the installer as Administrator to avoid Error 1920 (service failed to start)

### Step 3: Configure NXLog

Navigate to:
```
C:\Program Files\nxlog\conf\
```

Open `nxlog.conf` with **Notepad (as Administrator)** and replace the contents with:

```conf
define ROOT C:\\Program Files\\nxlog
define ROOT_STRING C:\\Program Files\\nxlog
define CERTDIR %ROOT%\\cert

Moduledir %ROOT%\\modules
CacheDir %ROOT%\\data
Pidfile %ROOT%\\data\\nxlog.pid
SpoolDir %ROOT%\\data
LogFile %ROOT%\\data\\nxlog.log

<Extension json>
    Module      xm_json
</Extension>

<Extension syslog>
    Module xm_syslog
</Extension>

<Input internal>
    Module im_internal
    Exec  $Message = to_json();
</Input>

<Input eventlog>
    Module im_msvistalog
    Exec  $Message = to_json();
</Input>

<Processor buffer>
    Module pm_buffer
    MaxSize 102400
    Type disk
</Processor>

<Output out>
    Module om_tcp
    Host logs-01.loggly.com
    Port 514
    Exec to_syslog_ietf();
    Exec $raw_event =~ s/(\[.*])//g; $raw_event = replace($raw_event, '{', '[YOUR-LOGGLY-TOKEN@41058 tag="windows"] {', 1);
</Output>

<Route 1>
    Path internal, eventlog => buffer => out
</Route>
```

> ⚠️ **Note:** Replace `YOUR-LOGGLY-TOKEN` with your actual Loggly customer token.
> 
> ⚠️ **32-bit OS users:** Change `Program Files` to `Program Files (x86)` in the ROOT definition.

### Step 4: Start the NXLog Service

Open **Command Prompt as Administrator** and run:
```cmd
net start nxlog
```

You should see:
```
The nxlog service was started successfully.
```

Alternatively, open `services.msc`, find **nxlog**, and click **Start/Restart**.

### Step 5: Verify Logs in Loggly

1. Go to your Loggly dashboard
2. Click **Search**
3. Use this query to find Windows Logon Events:
```
json.eventid:4624
```

---

##  Key Event IDs to Monitor

| Event ID | Description |
|----------|-------------|
| **4624** | Successful Logon |
| **4625** | Failed Logon |
| **4648** | Logon with Explicit Credentials |
| **4720** | User Account Created |
| **4726** | User Account Deleted |
| **4732** | User Added to Admin Group |

---

##  Troubleshooting

### Error 1920 — Service failed to start
- Run installer as Administrator
- Delete old service: `sc delete nxlog`
- Delete old folder: `rmdir /s /q "C:\Program Files\nxlog"`
- Reboot and reinstall

### Error 1053 — Service did not respond in time
- Check your `nxlog.conf` for errors
- Verify the ROOT path matches your installation directory (32-bit vs 64-bit)

### No logs appearing in Loggly
- Check NXLog logs: `type "C:\Program Files\nxlog\data\nxlog.log"`
- Verify your Loggly token is correct in the config
- Ensure port 514 (TCP) is not blocked by your firewall

---




##  Roadmap

- [x] Log Management with Loggly + NXLog
- [ ] Full SIEM setup with Wazuh
- [ ] Threat Detection Rules
- [ ] Incident Response Playbooks
- [ ] Network Monitoring with Zeek/Suricata

---

##  License

MIT License — feel free to use and adapt for your own SOC lab!
