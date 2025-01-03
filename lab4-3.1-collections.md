# 🚀 **Ansible Playbook: Using Roles and Modules from Content Collections**

This project demonstrates how to install Ansible Content Collections, use roles, and leverage modules from those collections.

## 📂 **Project Structure**

```
/role-collections
├── bck.yml             # Playbook using gls.utils collection
├── new_system.yml      # Playbook using redhat.insights and selinux roles
├── inventory
├── collections/
│   ├── gls/utils/
```

---

## 🛠️ **1. Install Ansible Collections**

### **Install gls.utils Collection Directly**

Use the `ansible-galaxy` command to install the `gls.utils` collection directly from a tarball.

```bash
ansible-galaxy collection install gls-utils-0.0.1.tar.gz -p collections
```

### **Verify Installed Collections**

```bash
ansible-galaxy collection list -p collections
```

**Expected Output:**
```
Collection               Version
------------------------ -------
gls.utils                0.0.1
```

---

## 📋 **2. Backup Configuration Using gls.utils Collection**

**Playbook:** `bck.yml`

```yaml
---
- name: Backup the system configuration
  hosts: servera.lab.example.com
  become: true
  gather_facts: false

  tasks:
    # Verify connectivity with gls.utils.newping module
    - name: Ensure the machine is up
      gls.utils.newping:
        data: pong

    # Backup configuration files with gls.utils.backup role
    - name: Ensure configuration files are saved
      ansible.builtin.include_role:
        name: gls.utils.backup
      vars:
        backup_id: backup_etc
        backup_files:
          - /etc/sysconfig
          - /etc/yum.repos.d
```

### **Verify Syntax**
```bash
ansible-navigator run -m stdout bck.yml --syntax-check
```

### **Run the Playbook**
```bash
ansible-navigator run -m stdout bck.yml
```

**Expected Output:**
```
TASK [Ensure the machine is up] ************************************************
ok: [servera.lab.example.com]

TASK [Ensure configuration files are saved] ************************************
changed: [servera.lab.example.com]
```

---

## 📖 **Key Concepts Used**

- **Ansible Galaxy:** Install and manage collections with `ansible-galaxy`.
- **Modules:** Use `gls.utils.newping` for connectivity tests.
- **Roles:** Utilize `gls.utils.backup` 

---




