# 🚀 **Ansible Role: myvhost - Apache Virtual Host Deployment**

This project contains an Ansible role named `myvhost` to automate the deployment, configuration, and validation of an Apache web server with a virtual host.

## 📂 **Project Structure**

```
/role-create
├── use-vhost-role.yml   # Playbook using the myvhost role
├── inventory
├── roles/
│   ├── myvhost/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── vhost.conf.j2
│   │   ├── files/
│   │   │   ├── html/
│   │   │   │   └── index.html
```

---

## 🛠️ **1. roles/myvhost**

### **Description:**  
This role installs and configures the Apache HTTP server with a virtual host.

**Tasks File:** `roles/myvhost/tasks/main.yml`

```yaml
---
# Ensure the Apache package is installed
- name: Ensure httpd is installed
  ansible.builtin.dnf:
    name: httpd
    state: latest

# Ensure the Apache service is enabled and started
- name: Ensure httpd is started and enabled
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

# Deploy the virtual host configuration
- name: vhost file is installed
  ansible.builtin.template:
    src: vhost.conf.j2
    dest: /etc/httpd/conf.d/vhost.conf
    owner: root
    group: root
    mode: '0644'
  notify:
    - restart httpd
```

**Handlers File:** `roles/myvhost/handlers/main.yml`

```yaml
---
# Restart Apache service when configuration changes
- name: restart httpd
  ansible.builtin.service:
    name: httpd
    state: restarted
```

**Template File:** `roles/myvhost/templates/vhost.conf.j2`

```apache
# {{ ansible_managed }}

<VirtualHost *:80>
    ServerAdmin webmaster@{{ ansible_fqdn }}
    ServerName {{ ansible_fqdn }}
    ErrorLog logs/{{ ansible_hostname }}-error.log
    CustomLog logs/{{ ansible_hostname }}-common.log common
    DocumentRoot /var/www/vhosts/{{ ansible_hostname }}/

    <Directory /var/www/vhosts/{{ ansible_hostname }}/>
        Options +Indexes +FollowSymlinks +Includes
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

**Files:**
- `files/html/index.html`: Contains the web server content (`simple index`).

---

## 📋 **2. use-vhost-role.yml**

### **Description:**  
This playbook uses the `myvhost` role to configure and deploy the Apache virtual host and ensures web content is copied.

```yaml
---
- name: Use myvhost role playbook
  hosts: webservers
  become: true

  pre_tasks:
    - name: pre_tasks message
      ansible.builtin.debug:
        msg: 'Ensure web server configuration.'

  roles:
    - myvhost

  post_tasks:
    - name: HTML content is installed
      ansible.builtin.copy:
        src: files/html/
        dest: "/var/www/vhosts/{{ ansible_hostname }}"

    - name: post_tasks message
      ansible.builtin.debug:
        msg: 'Web server is configured.'
```

---

## 🚦 **How to Run the Playbooks**

1. **Navigate to the project directory:**
   ```bash
   cd /home/student/role-create
   ```

2. **Verify Playbook Syntax:**
   ```bash
   ansible-navigator run -m stdout use-vhost-role.yml --syntax-check
   ```

3. **Run the Playbook:**
   ```bash
   ansible-navigator run -m stdout use-vhost-role.yml
   ```

4. **Verify Apache Configuration:**
   ```bash
   ansible-navigator run -m stdout verify-config.yml
   ```

5. **Verify Content Availability:**
   ```bash
   curl http://servera.lab.example.com
   ```

---

## ✅ **Expected Results**

| Host                 | OK | Changed | Unreachable | Failed | Skipped | Rescued | Ignored |
|-----------------------|----|---------|------------|--------|---------|---------|---------|
| servera.lab.example.com | 8  | 5       | 0          | 0      | 0       | 0       | 0       |

**Content Verification:**
```
simple index
```

---

## 📖 **Key Concepts Used**

- **Roles:** Structure tasks, handlers, files, and templates.
- **Variables:** Dynamic configurations in templates.
- **Handlers:** Restart services on configuration changes.
- **Templates:** Jinja2 for virtual host configurations.
- **File Management:** Copy static content.

---

Happy Automating! 🚀✨
