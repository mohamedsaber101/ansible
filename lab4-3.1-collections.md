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
│   ├── redhat/insights/
│   └── redhat/rhel_system_roles/
```

---

## 🛠️ **1. Install Ansible Collections**

### **Install gls.utils Collection Directly**

Use the `ansible-galaxy` command to install the `gls.utils` collection directly from a tarball.

```bash
ansible-galaxy collection install gls-utils-0.0.1.tar.gz -p collections
```

### **Install Additional Collections (Optional)**

If other collections are required, use the same command format:

```bash
ansible-galaxy collection install redhat-insights-1.0.7.tar.gz -p collections
ansible-galaxy collection install redhat-rhel_system_roles-1.19.3.tar.gz -p collections
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
redhat.insights          1.0.7
redhat.rhel_system_roles 1.19.3
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

## 🧪 **3. System Configuration Using Red Hat Collections**

**Playbook:** `new_system.yml`

```yaml
---
- name: Configure the system
  hosts: servera.lab.example.com
  become: true
  gather_facts: true

  tasks:
    # Register the system with Red Hat Insights
    - name: Ensure the system is registered with Insights
      ansible.builtin.include_role:
        name: redhat.insights.insights_client
      vars:
        auto_config: false
        insights_proxy: http://proxy.example.com:8080

    # Ensure SELinux is set to Enforcing
    - name: Ensure SELinux mode is Enforcing
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.selinux
      vars:
        selinux_state: enforcing
```

### **Run the Playbook in Check Mode**
```bash
ansible-navigator run -m stdout new_system.yml --check
```

**Expected Output:**
```
TASK [redhat.insights.insights_client : Set Insights Configuration Values] ***
changed: [servera.lab.example.com]

TASK [redhat.rhel_system_roles.selinux : Install SELinux python3 tools] ****
ok: [servera.lab.example.com]
```

---

## ✅ **4. Verify Configurations**

### **Check Apache Configuration**

```bash
ansible-navigator run -m stdout verify-config.yml
```

**Expected Output:**
```
<VirtualHost *:80>
    ServerAdmin webmaster@servera.lab.example.com
    ServerName servera.lab.example.com
    ErrorLog logs/servera-error.log
    DocumentRoot /var/www/vhosts/servera/
</VirtualHost>
```

### **Verify Web Content**
```bash
curl http://servera.lab.example.com
```

**Expected Output:**
```
simple index
```

---

## 📖 **Key Concepts Used**

- **Ansible Galaxy:** Install and manage collections with `ansible-galaxy`.
- **Modules:** Use `gls.utils.newping` for connectivity tests.
- **Roles:** Utilize `gls.utils.backup`, `redhat.insights.insights_client`, and `redhat.rhel_system_roles.selinux`.
- **Check Mode:** Test playbook changes without applying them.
- **Syntax Validation:** Ensure playbooks are error-free with `--syntax-check`.

---

## 🚦 **5. Clean Up the Environment**

Finish the lab session:
```bash
lab finish role-collections
```

---

Happy Automating! 🚀✨

