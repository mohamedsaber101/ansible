# 🚀 **Ansible Playbook: Configuring Time Synchronization with System Roles**

This project demonstrates how to use the `redhat.rhel_system_roles.timesync` role to configure time synchronization and time zones on managed hosts.

## 📂 **Project Structure**

```
/role-system
├── configure_time.yml   # Playbook for time synchronization
├── inventory
├── ansible.cfg          # Ansible configuration file
├── collections/
│   ├── redhat/rhel_system_roles/
├── group_vars/
│   ├── all/
│   │   └── timesync.yml
│   ├── na_datacenter/
│   │   └── timezone.yml
│   ├── europe_datacenter/
│   │   └── timezone.yml
```

---

## 🛠️ **1. Install Required Collection**

### **Install `redhat.rhel_system_roles` Collection**

Use the `ansible-galaxy` command to install the `redhat.rhel_system_roles` collection directly from a tarball.

```bash
ansible-galaxy collection install redhat-rhel_system_roles-1.19.3.tar.gz -p collections
```

### **Update Ansible Configuration File**

Edit the `ansible.cfg` file to include the `collections_paths` variable:

```ini
[defaults]
inventory=./inventory
remote_user=devops
collections_paths=./collections:~/.ansible/collections:/usr/share/ansible/collections
```

### **Verify Installed Collections**

```bash
ansible-galaxy collection list -p collections
```

**Expected Output:**
```
Collection               Version
------------------------ -------
redhat.rhel_system_roles 1.19.3
```

---

## 📋 **2. Configure Time Synchronization**

### **Role Variables Configuration**

Create the `group_vars/all/timesync.yml` file to define the NTP configuration.

**File:** `group_vars/all/timesync.yml`

```yaml
---
# redhat.rhel_system_roles.timesync variables for all hosts

timesync_ntp_provider: chrony

timesync_ntp_servers:
  - hostname: classroom.example.com
    iburst: yes
```
## 🌍 **3. Configure Time Zones for Data Centers**

### **Create Time Zone Variables**

Create separate time zone files for different data centers.

**File:** `group_vars/na_datacenter/timezone.yml`
```yaml
host_timezone: America/Chicago
```

**File:** `group_vars/europe_datacenter/timezone.yml`
```yaml
host_timezone: Europe/Helsinki
```

### **Verify Time Zone Strings**

Use the `timedatectl list-timezones` command to verify the valid time zone strings.
```bash
timedatectl list-timezones | grep Chicago
timedatectl list-timezones | grep Helsinki
```

---
### **Playbook:** `configure_time.yml`

```yaml
---
- name: Time Synchronization
  hosts: database_servers
  become: true

  roles:
    - redhat.rhel_system_roles.timesync

  post_tasks:
    # Get the current time zone
    - name: Get time zone
      ansible.builtin.command: timedatectl show
      register: current_timezone
      changed_when: false

    # Set time zone if it's incorrect
    - name: Set time zone
      ansible.builtin.command: "timedatectl set-timezone {{ host_timezone }}"
      when: host_timezone not in current_timezone.stdout
      notify: reboot host

  handlers:
    - name: reboot host
      ansible.builtin.reboot:
```

---



## ✅ **4. Validate Playbook Syntax and Run**

### **Check Syntax**
```bash
ansible-navigator run -m stdout configure_time.yml --syntax-check
```

### **Run the Playbook**
```bash
ansible-navigator run -m stdout configure_time.yml
```

**Expected Output:**
```
TASK [Gathering Facts] *******************************************************
ok: [servera.lab.example.com]
ok: [serverb.lab.example.com]

TASK [Get time zone] *********************************************************
ok: [servera.lab.example.com]
ok: [serverb.lab.example.com]

TASK [Set time zone] *********************************************************
changed: [servera.lab.example.com]
changed: [serverb.lab.example.com]

RUNNING HANDLER [reboot host] *************************************************
changed: [servera.lab.example.com]
changed: [serverb.lab.example.com]
```

---

## 🧪 **5. Verify Time Synchronization and Time Zone**

### **Check Time Zone on Managed Hosts**
```bash
ssh servera date
ssh serverb date
```

**Expected Output:**
```
Tue Aug 16 07:43:33 PM CDT 2022
Wed Aug 17 03:43:41 AM EEST 2022
```

---

## 📖 **Key Concepts Used**

- **System Roles:** Utilize `redhat.rhel_system_roles.timesync` for consistent NTP configuration.
- **Group Variables:** Centralize configuration with `group_vars`.
- **Handlers:** Automate actions such as rebooting hosts after time zone changes.
- **Time Zones:** Use `timedatectl` for managing system time zones.
- **Syntax Validation:** Validate playbooks using `--syntax-check`.

---


