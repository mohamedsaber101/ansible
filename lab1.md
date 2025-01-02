# 🚀 **Ansible Playbook: Web Server Deployment and Testing**

This project contains three Ansible playbooks to automate the deployment, configuration, and validation of web servers.

## 📂 **Project Structure**

```
/review-cr2
├── dev_deploy.yml      # Playbook for deploying and configuring web servers
├── get_web_content.yml # Playbook for testing web server content
├── site.yml            # Master playbook to run both playbooks
├── templates/
│   └── vhost.conf.j2   # Jinja2 template for virtual host configuration
└── files/
    └── index.html      # Sample web content file
```

---

## 🛠️ **1. dev_deploy.yml**

### **Description:**  
This playbook installs and configures Apache HTTP servers on managed hosts.

```yaml
---
- name: Install and configure web servers
  hosts: webservers
  become: true

  vars:
    web_packages:
      - httpd
      - firewalld
    firewall_services:
      - http

  tasks:
    # Display system facts such as OS, hostname, memory, and CPU details
    - name: Display system facts
      ansible.builtin.debug:
        msg:
          - "Operating System: {{ ansible_facts['os_family'] }}"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Memory Total: {{ ansible_facts['memtotal_mb'] }} MB"
          - "CPU Cores: {{ ansible_facts['processor_cores'] }}"

    # Install required packages
    - name: Install required packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ web_packages }}"

    # Start required services
    - name: Start services
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop: "{{ web_packages }}"

    # Deploy the virtual host configuration template
    - name: Deploy configuration template
      ansible.builtin.template:
        src: templates/vhost.conf.j2
        dest: /etc/httpd/conf.d/vhost.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart httpd

    # Copy the index.html file to the virtual host directory
    - name: Copy index.html
      ansible.builtin.copy:
        src: files/
        dest: "/var/www/vhosts/{{ ansible_facts['hostname'] }}/"
        owner: root
        group: root
        mode: '0644'

    # Ensure the firewall allows necessary services
    - name: Ensure web server ports are open
      ansible.posix.firewalld:
        state: enabled
        permanent: true
        immediate: true
        service: "{{ item }}"
      loop: "{{ firewall_services }}"
      when: ansible_facts['os_family'] == 'RedHat'

  handlers:
    # Restart the HTTPD service when notified
    - name: Restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted
```

---
## 📝 **2. templates/vhost.conf.j2**

```jinja
<VirtualHost *:80>
    ServerName {{ ansible_facts['hostname'] }}
    DocumentRoot "/var/www/vhosts/{{ ansible_facts['hostname'] }}"

    <Directory "/var/www/vhosts/{{ ansible_facts['hostname'] }}">
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog "/var/log/httpd/{{ ansible_facts['hostname'] }}_error.log"
    CustomLog "/var/log/httpd/{{ ansible_facts['hostname'] }}_access.log" combined
</VirtualHost>
```

---

## 📄 **3. files/index.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Welcome to {{ ansible_facts['hostname'] }}</title>
</head>
<body>
    <h1>Welcome to {{ ansible_facts['hostname'] }} Web Server!</h1>
    <p>This page is deployed and managed using Ansible.</p>
    <p>Happy Automation 🚀✨</p>
</body>
</html>
```
---
## 🧪 **2. get_web_content.yml**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

```yaml
---
- name: Test web content
  hosts: workstation
  become: true

  vars:
    target_servers:
      - servera.lab.example.com
      - serverb.lab.example.com

  tasks:
    # Display system facts such as OS, hostname, memory, and CPU details
    - name: Display system facts
      ansible.builtin.debug:
        msg:
          - "Operating System: {{ ansible_facts['os_family'] }}"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Memory Total: {{ ansible_facts['memtotal_mb'] }} MB"
          - "CPU Cores: {{ ansible_facts['processor_cores'] }}"

    # Attempt to retrieve web content from target servers
    - name: Retrieve web content and write to error log on failure
      block:
        - name: Retrieve web content
          ansible.builtin.uri:
            url: http://{{ item }}
            return_content: true
          register: content
      rescue:
        # Log error to a file if the retrieval fails
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/review-cr2/error.log
            line: "Error retrieving content from {{ item }}: {{ content }}"
            create: true
      loop: "{{ target_servers }}"
```

---

## 📋 **3. site.yml**

### **Description:**  
This master playbook imports and executes `dev_deploy.yml` and `get_web_content.yml` sequentially.

```yaml
---
# Deploy and configure web servers
- name: Deploy web servers
  ansible.builtin.import_playbook: dev_deploy.yml

# Validate the web servers by retrieving web content
- name: Retrieve web content
  ansible.builtin.import_playbook: get_web_content.yml
```

---

## 🚦 **How to Run the Playbooks**

1. **Navigate to the project directory:**
   ```bash
   cd /home/student/review-cr2
   ```

2. **Run the playbooks using `ansible-navigator`:**
   ```bash
   ansible-navigator run -m stdout site.yml
   ```

3. **Expected Output:**
   - All tasks should complete successfully.
   - No errors should appear in the output.

---

## ✅ **Expected Results**

| Host                 | OK | Changed | Unreachable | Failed | Skipped | Rescued | Ignored |
|-----------------------|----|---------|------------|--------|---------|---------|---------|
| servera.lab.example.com | 7  | 6       | 0          | 0      | 0       | 0       | 0       |
| serverb.lab.example.com | 7  | 6       | 0          | 0      | 0       | 0       | 0       |
| workstation           | 2  | 0       | 0          | 0      | 0       | 0       | 0       |

---

Happy Automating! 🚀✨
## 📝 **2. templates/vhost.conf.j2**

```jinja
<VirtualHost *:80>
    ServerName {{ ansible_facts['hostname'] }}
    DocumentRoot "/var/www/vhosts/{{ ansible_facts['hostname'] }}"

    <Directory "/var/www/vhosts/{{ ansible_facts['hostname'] }}">
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog "/var/log/httpd/{{ ansible_facts['hostname'] }}_error.log"
    CustomLog "/var/log/httpd/{{ ansible_facts['hostname'] }}_access.log" combined
</VirtualHost>
```

---

## 📄 **3. files/index.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Welcome to {{ ansible_facts['hostname'] }}</title>
</head>
<body>
    <h1>Welcome to {{ ansible_facts['hostname'] }} Web Server!</h1>
    <p>This page is deployed and managed using Ansible.</p>
    <p>Happy Automation 🚀✨</p>
</body>
</html>
```
## 🧪 **4. get_web_content.yml**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

yaml
---
- name: Test web content
  hosts: workstation
  become: true

  vars:
    target_servers:
      - servera.lab.example.com
      - serverb.lab.example.com

  tasks:
    # Display system facts such as OS, hostname, memory, and CPU details
    - name: Display system facts
      ansible.builtin.debug:
        msg:
          - "Operating System: {{ ansible_facts['os_family'] }}"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Memory Total: {{ ansible_facts['memtotal_mb'] }} MB"
          - "CPU Cores: {{ ansible_facts['processor_cores'] }}"

    # Attempt to retrieve web content from target servers
    - name: Retrieve web content and write to error log on failure
      block:
        - name: Retrieve web content
          ansible.builtin.uri:
            url: http://{{ item }}
            return_content: true
          register: content
      rescue:
        # Log error to a file if the retrieval fails
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/review-cr2/error.log
            line: "Error retrieving content from {{ item }}: {{ content }}"
            create: true
      loop: "{{ target_servers }}"


---

## 📋 **5. site.yml**

### **Description:**  
This master playbook imports and executes dev_deploy.yml and get_web_content.yml sequentially.

yaml
---
# Deploy and configure web servers
- name: Deploy web servers
  ansible.builtin.import_playbook: dev_deploy.yml

# Validate the web servers by retrieving web content
- name: Retrieve web content
  ansible.builtin.import_playbook: get_web_content.yml


---

## 🚦 **How to Run the Playbooks**

1. **Navigate to the project directory:**
   
bash
   cd /home/student/review-cr2


2. **Run the playbooks using ansible-navigator:**
   
bash
   ansible-navigator run -m stdout site.yml


3. **Expected Output:**
   - All tasks should complete successfully.
   - No errors should appear in the output.

---

## ✅ **Expected Results**

| Host                 | OK | Changed | Unreachable | Failed | Skipped | Rescued | Ignored |
|-----------------------|----|---------|------------|--------|---------|---------|---------|
| servera.lab.example.com | 7  | 6       | 0          | 0      | 0       | 0       | 0       |
| serverb.lab.example.com | 7  | 6       | 0          | 0      | 0       | 0       | 0       |
| workstation           | 2  | 0       | 0          | 0      | 0       | 0       | 0       |

---

