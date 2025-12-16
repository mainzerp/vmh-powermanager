# VM Host Power Manager - Frontend User Guide

**Version**: 1.1.0  
**Last Updated**: December 1, 2025

This guide explains how to use the VM Host Power Manager web interface to monitor UPS devices, manage virtual machine hosts, and configure automated shutdown procedures.

---

## 📚 Table of Contents

1. [Getting Started](#getting-started)
2. [Dashboard](#dashboard)
3. [UPS Devices](#ups-devices)
4. [Managed Systems](#managed-systems)
5. [Orchestration Plans](#orchestration-plans)
6. [Notifications](#notifications)
7. [Events & Logs](#events--logs)
8. [Settings](#settings)
9. [Backup & Restore](#backup--restore)

---

## Getting Started

### First Login

1. Navigate to `https://your-server:port`
2. Log in with your admin credentials
3. On first login, you'll be prompted to:
   - Change your password
   - Set up 2FA (recommended)
   - Configure your first UPS device

### Navigation

The main navigation is on the left sidebar:

| Icon | Page | Description |
|------|------|-------------|
| 📊 | Dashboard | Overview of all systems and UPS status |
| 🔋 | UPS Devices | Manage and monitor UPS units |
| 🖥️ | Systems | Manage VMware/Proxmox hosts |
| 📋 | Plans | Orchestration shutdown plans |
| 🔔 | Notifications | Alert settings (Email, Webhook) |
| 📜 | Events | System event log |
| ⚙️ | Settings | Platform configuration |
| 💾 | Backup | Backup and restore |

---

## Dashboard

The Dashboard provides a real-time overview of your infrastructure.

### Status Cards

| Card | Shows |
|------|-------|
| **UPS Status** | Battery %, load, runtime remaining |
| **Systems** | Online/offline hosts, VM count |
| **Active Plans** | Running orchestration plans |
| **Recent Events** | Last 10 system events |

### Health Indicators

- 🟢 **Green**: All systems healthy
- 🟡 **Yellow**: Warning (e.g., battery < 50%)
- 🔴 **Red**: Critical (e.g., on battery, system offline)

### Quick Actions

- **Refresh All**: Update all status data

---

## UPS Devices

### Adding a UPS Device

1. Click **+ Add UPS**
2. Enter the following information:

| Field | Description | Example |
|-------|-------------|---------|
| Name | Friendly name | `Server Room UPS 1` |
| IP Address | SNMP IP address | `192.168.1.100` |
| SNMP Version | v1, v2c, or v3 | `v2c` |
| Community | SNMP community string | `public` |
| Poll Interval | Seconds between polls | `30` |

3. Click **Test Connection** to verify SNMP access
4. Click **Save**

### UPS Status View

Each UPS shows:

| Metric | Description |
|--------|-------------|
| **Battery %** | Current charge level |
| **Runtime** | Estimated minutes remaining |
| **Input Voltage** | Mains voltage |
| **Status** | Online, On Battery, Low Battery, etc. |

### UPS Actions

- **📊 View Details**: Full status and history
- **⚙️ Edit**: Modify settings
- **🔄 Refresh**: Force status update
- **🗑️ Delete**: Remove UPS device

### SNMP v3 Configuration

For SNMPv3, additional fields are required:

| Field | Description |
|-------|-------------|
| Username | SNMPv3 username |
| Auth Protocol | MD5 or SHA |
| Auth Password | Authentication password |
| Priv Protocol | DES or AES |
| Priv Password | Privacy password |

---

## Managed Systems

### Supported Platforms

- **VMware vSphere/ESXi** (6.5+)
- **Proxmox VE** (7.0+)

### Adding a System

1. Click **+ Add System**
2. Select platform type (VMware or Proxmox)
3. Enter connection details:

**VMware:**
| Field | Description |
|-------|-------------|
| Name | Display name |
| vCenter/ESXi Host | IP or hostname |
| Port | Default: 443 |
| Username | vSphere admin user |
| Password | vSphere password |
| Verify SSL | Enable for production |

**Proxmox:**
| Field | Description |
|-------|-------------|
| Name | Display name |
| Host | Proxmox IP/hostname |
| Port | Default: 8006 |
| Username | root@pam or API user |
| Token ID | API token name |
| Token Secret | API token secret |

4. Click **Test Connection**
5. Click **Save**

### System Status

| Status | Description |
|--------|-------------|
| 🟢 Online | Connected and responsive |
| 🟡 Degraded | Partial connectivity |
| 🔴 Offline | Cannot connect |
| ⏸️ Maintenance | Manually paused |

### VM List

Click on a system to see its VMs:

| Column | Description |
|--------|-------------|
| Name | VM name |
| Status | Running, Stopped, Suspended |

---

## Orchestration Plans

Plans define automated shutdown and startup sequences triggered by UPS runtime thresholds. The **Flow Editor** provides a visual interface to create and manage these plans.

### The Flow Editor

The Flow Editor displays your plan as a visual flow diagram with three main sections:

1. **UPS Mappings** (left) - Which UPS devices trigger this plan
2. **Shutdown Phases** (center) - Phases executed during power failure
3. **Startup Phases** (right) - Phases executed during power recovery

### Creating a Plan

1. Navigate to **Plans** and click **+ Create Plan**
2. Enter plan name and description
3. Use the Flow Editor to build your plan

### Plan Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Name** | Plan identifier | - |
| **Description** | Optional notes | - |
| **Active** | Enable/disable plan | Yes |
| **Auto-Mirror Startup** | Automatically reverse shutdown order for startup | Yes |

### UPS Mappings

Map which UPS devices trigger this plan:

1. Click **+ Add UPS Mapping** in the Flow Editor
2. Select a UPS device
3. Configure settings:

| Setting | Description | Default |
|---------|-------------|---------|
| **Priority** | Execution order (lower = first) | 1 |
| **Watch only for shutdown** | Only monitor for shutdown (not startup) | No |

### Creating Phases

Phases group devices that should shut down together. Click **+ Add Phase** to create:

| Field | Description | Default |
|-------|-------------|---------|
| **Phase Name** | Descriptive name (e.g., "Non-Critical VMs") | - |
| **Description** | Optional details | - |
| **Parallel Execution** | Run all devices simultaneously | No |
| **Wait for Completion** | Wait before next phase starts | Yes |
| **Step Delay (s)** | Wait time after starting each device | 0 |
| **Shutdown Threshold (min)** | Execute when runtime drops to this value | 30 |
| **Startup Threshold (min)** | Execute when runtime rises above this value | 60 |

> **Note:** Phase Timeout is auto-calculated from device shutdown timeouts and cannot be edited manually.

### Runtime Thresholds Explained

Phases execute based on UPS runtime thresholds:

`
UPS Runtime    Shutdown Example (descending)
--------------------------------------------
100 min        
                 
 60 min        <- Phase 1 triggers (non-critical)
                 
 30 min        <- Phase 2 triggers (standard)
                 
 15 min        <- Phase 3 triggers (critical)
                 
  5 min        <- Phase 4 triggers (hosts)
                 
  0 min        
--------------------------------------------
`

During startup (power recovery), phases execute in **reverse order** based on startup thresholds.

### Adding Devices to Phases

1. Click on a phase node in the Flow Editor
2. Click **+ Add Device**
3. Select from available devices:
   - **Systems** (VMware/Proxmox hosts)
   - **VMs** (Virtual machines)
4. Configure device settings:

| Setting | Description | Default |
|---------|-------------|---------|
| **Execution Order** | Order within phase | Auto |
| **Shutdown Timeout (s)** | Max wait for graceful shutdown | 300 |
| **Custom Command** | Optional shutdown command override | - |

### Plan Execution Flow

`
Power Failure Detected
        |
        v
Check UPS Runtime
        |
        v
+----------------------------------+
|  Runtime <= Phase 1 Threshold?   |
|  -> Execute Phase 1              |
|     -> Shutdown non-critical VMs |
+----------------------------------+
        |
        v (wait for completion)
+----------------------------------+
|  Runtime <= Phase 2 Threshold?   |
|  -> Execute Phase 2              |
|     -> Shutdown standard VMs     |
+----------------------------------+
        |
        v
    ... more phases ...
        |
        v
    All systems safe
`

### Plan Actions

- **Save**: Save current plan state
- **Execute Shutdown**: Manually trigger shutdown sequence
- **Execute Startup**: Manually trigger startup sequence
- **Clone**: Duplicate entire plan
- **Delete**: Remove plan

### Manual Execution

1. Open plan in Flow Editor
2. Click **Execute Shutdown** or **Execute Startup**
3. Confirm the action
4. Monitor progress in real-time via the Flow Editor

The Flow Editor shows live status during execution:
- **Pending** - Waiting to execute
- **Running** - Currently executing
- **Completed** - Successfully finished
- **Failed** - Error occurred

---

## Notifications

Configure alerts for UPS events and system status changes.

### Email Notifications

1. Go to **Notifications** → **Email**
2. Configure SMTP settings:

| Field | Description |
|-------|-------------|
| SMTP Server | Mail server hostname |
| Port | 25, 465 (SSL), or 587 (TLS) |
| Username | SMTP username |
| Password | SMTP password |
| From Address | Sender email |
| To Addresses | Recipients (comma-separated) |

3. Click **Test Email** to verify
4. Click **Save**

### Webhook Notifications

1. Go to **Notifications** → **Webhooks**
2. Add a webhook:

| Field | Description |
|-------|-------------|
| Name | Webhook identifier |
| URL | Endpoint URL |
| Method | POST (recommended) |
| Headers | Custom headers (JSON) |

3. Select events to trigger webhook
4. Click **Save**

### Notification Events

| Event | Description |
|-------|-------------|
| UPS On Battery | Power failure detected |
| UPS Low Battery | Battery critically low |
| UPS Back Online | Power restored |
| Plan Started | Shutdown sequence began |
| Plan Completed | Shutdown sequence finished |
| System Offline | Host unreachable |
| System Online | Host reconnected |

---

## Events & Logs

### Event Log

The Events page shows all system activity:

| Column | Description |
|--------|-------------|
| Timestamp | When event occurred |
| Severity | Info, Warning, Error, Critical |
| Source | Component (UPS, System, Plan) |
| Message | Event description |

### Filtering Events

- **Date Range**: Select start/end dates
- **Severity**: Filter by level
- **Source**: Filter by component
- **Search**: Free-text search

### Exporting Events

1. Apply desired filters
2. Click **Export CSV**
3. Download event data

---

## Settings

### User Settings

| Setting | Description |
|---------|-------------|
| **Profile** | Change name, email |
| **Password** | Update password |
| **2FA** | Enable/disable TOTP |
| **Theme** | Light/Dark mode |
| **Language** | Interface language |

---

## Backup & Restore

### Creating a Backup

1. Go to **Backup & Restore**
2. Click **Create Backup**
3. Download the `.json` or `.zip` file

### Restoring from Backup

1. Go to **Backup & Restore**
2. Click **Restore**
3. Upload backup file
4. Select what to restore
5. Click **Restore**

---

## Troubleshooting

### UPS Not Responding

1. Verify IP address is correct
2. Check SNMP is enabled on UPS
3. Verify community string/credentials
4. Check firewall allows UDP 161
5. Try `snmpwalk` from server command line

### System Connection Failed

1. Verify credentials are correct
2. Check network connectivity
3. For VMware: Verify vCenter/ESXi is accessible on port 443
4. For Proxmox: Verify API is enabled on port 8006
5. Check SSL certificate settings

### Plan Not Executing

1. Verify plan is enabled (not paused)
2. Check trigger conditions match current UPS state
3. Review Events log for errors
4. Verify system credentials are still valid
5. Check Celery worker is running (Docker logs)

### Notifications Not Sending

1. Test notification from Settings
2. Check SMTP/Webhook credentials
3. Review Events log for delivery errors
4. Verify firewall allows outbound connections
5. Check spam folder for emails
