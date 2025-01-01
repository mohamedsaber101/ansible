# 🚀 **Ansible Playbook: Managing Users, Groups, and SSH Configuration**

This project demonstrates how to manage users, groups, sudo permissions, and SSH configurations using Ansible.

## 📂 **Project Structure**

```
/system-users
├── users.yml          # Playbook for managing users and authentication
├── inventory
├── vars/
│   └── users_vars.yml # Variables for users
├── files/
│   ├── user1.key.pub  # SSH public key for user1
│   ├── user2.key.pub  # SSH public key for user2
│   ├── user3.key.pub  # SSH public key for user3
│   ├── user4.key.pub  # SSH public key for user4
│   ├── user5.key.pub  # SSH public key for user5
```

---

## 🛠️ **1. Variables for Users and Groups**

Define users and their groups in a variable file.

**File:** `vars/users_vars.yml`

```yaml
---
users:
  - username: user1
    groups: webadmin
  - username: user2
    groups: webadmin
  - username: user3
    groups: webadmin
  - username: user4
    groups: webadmin
  - username: user5
    groups: webadmin
```

---

## 📋 **2. Playbook: `users.yml`**

This playbook creates users, manages their SSH keys, configures sudo permissions, and ensures SSH root login is disabled.

**File:** `users.yml`

```yaml
---
- name: Create multiple local users
  hosts: webservers
  become: true
  vars_files:
    - vars/users_vars.yml

  handlers:
    - name: Restart sshd
      ansible.builtin.service:
        name: sshd
        state: restarted

  tasks:
    # Ensure the webadmin group exists
    - name: Add webadmin group
      ansible.builtin.group:
        name: webadmin
        state: present

    # Create user accounts based on variables
    - name: Create user accounts
      ansible.builtin.user:
        name: "{{ item['username'] }}"
        groups: "{{ item['groups'] }}"
      loop: "{{ users }}"

    # Add SSH authorized keys for each user
    - name: Add authorized keys
      ansible.posix.authorized_key:
        user: "{{ item['username'] }}"
        key: "{{ lookup('file', 'files/' + item['username'] + '.key.pub') }}"
      loop: "{{ users }}"

    # Configure sudo to allow passwordless access for webadmin group
    - name: Modify sudo config to allow webadmin users sudo without a password
      ansible.builtin.lineinfile:
        path: /etc/sudoers.d/webadmin
        state: present
        create: true
        mode: 0440
        line: "%webadmin ALL=(ALL) NOPASSWD: ALL"
        validate: /usr/sbin/visudo -cf %s

    # Disable root login via SSH
    - name: Disable root login via SSH
      ansible.builtin.lineinfile:
        dest: /etc/ssh/sshd_config
        regexp: "^PermitRootLogin"
        line: "PermitRootLogin no"
      notify: Restart sshd
```

### **Task Explanation:**
1. **Add webadmin group:** Ensures the `webadmin` group exists.
2. **Create user accounts:** Creates users based on variables.
3. **Add SSH authorized keys:** Adds SSH keys from the `files/` directory.
4. **Modify sudo configuration:** Allows `webadmin` group passwordless sudo access.
5. **Disable root login via SSH:** Prevents direct root login.

---

## ✅ **3. Verify Syntax and Run the Playbook**

### **Check Syntax**
```bash
ansible-navigator run -m stdout users.yml --syntax-check
```

### **Run the Playbook**
```bash
ansible-navigator run -m stdout users.yml
```

**Expected Output:**
```
TASK [Add webadmin group] ****************************************************
changed: [servera.lab.example.com]

TASK [Create user accounts] **************************************************
changed: [servera.lab.example.com] => (item={'username': 'user1', 'groups': 'webadmin'})
...

TASK [Add authorized keys] ***************************************************
changed: [servera.lab.example.com] => (item={'username': 'user1', 'groups': 'webadmin'})
...

TASK [Modify sudo config to allow webadmin users sudo without a password] ***
changed: [servera.lab.example.com]

TASK [Disable root login via SSH] *******************************************
changed: [servera.lab.example.com]

RUNNING HANDLER [Restart sshd] **********************************************
changed: [servera.lab.example.com]
```

---

## 🧪 **4. Validate the Configuration**

### **SSH Login as a User**
```bash
ssh user1@servera
sudo -i
```

**Expected Output:**
```
[root@servera ~]#
```

### **SSH Login as Root (Should Fail)**
```bash
ssh root@servera
```

**Expected Output:**
```
Permission denied, please try again.
```

---

## 📖 **Key Concepts Used**

- **User Management:** Create and manage user accounts and groups.
- **SSH Keys:** Secure SSH authentication using `ansible.posix.authorized_key`.
- **Sudo Configuration:** Enable passwordless sudo for specific groups.
- **SSH Daemon Configuration:** Secure SSH access and prevent root login.
- **Handlers:** Restart SSH daemon after configuration changes.

---

## 🚦 **5. Clean Up the Environment**

Finish the lab session:
```bash
lab finish system-users
```

---

Happy Automating! 🚀✨
