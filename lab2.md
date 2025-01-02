# 🚀 **Ansible Playbook: Database Server Deployment and Testing**

This project contains three Ansible playbooks to automate the deployment, configuration, and validation of database servers.

## 📂 **Project Structure**

```
/review-cr2
├── db_deploy.yml       # Playbook for deploying and configuring database servers
├── test_db_content.yml # Playbook for testing database server content
├── site.yml            # Master playbook to run both playbooks
├── templates/
│   └── db_config.cnf.j2 # Jinja2 template for database configuration
└── files/
    └── init_db.sql     # SQL script for initializing database
```

---

## 🛠️ **1. db_deploy.yml**

### **Description:**  
This playbook installs and configures MySQL database servers on managed hosts.

```yaml
---
- name: Install and configure database servers
  hosts: dbservers
  become: true

  vars:
    db_packages:
      - mysql-server
      - mysql-client
    db_service: mysqld
    db_config_file: /etc/my.cnf
    db_port: 3306

  tasks:
    # Display system facts such as OS, hostname, memory, and CPU details
    - name: Display system facts
      ansible.builtin.debug:
        msg:
          - "Operating System: {{ ansible_facts['os_family'] }}"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Memory Total: {{ ansible_facts['memtotal_mb'] }} MB"
          - "CPU Cores: {{ ansible_facts['processor_cores'] }}"

    # Install required database packages
    - name: Install database packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ db_packages }}"

    # Start and enable database service
    - name: Start and enable database service
      ansible.builtin.service:
        name: "{{ db_service }}"
        state: started
        enabled: true

    # Deploy database configuration template
    - name: Deploy database configuration
      ansible.builtin.template:
        src: templates/db_config.cnf.j2
        dest: "{{ db_config_file }}"
        owner: root
        group: root
        mode: '0644'
      notify: Restart database service

    # Ensure firewall allows database port
    - name: Ensure database port is open
      ansible.posix.firewalld:
        state: enabled
        permanent: true
        immediate: true
        port: "{{ db_port }}/tcp"
      when: ansible_facts['os_family'] == 'RedHat'

    # Initialize database with SQL script
    - name: Initialize database schema
      ansible.builtin.shell:
        cmd: "mysql < /files/init_db.sql"
        creates: /var/lib/mysql/initialized

  handlers:
    # Restart the MySQL service when notified
    - name: Restart database service
      ansible.builtin.service:
        name: "{{ db_service }}"
        state: restarted
```

---

---
## 📄 **templates/db_config.cnf.j2**

```jinja2
[mysqld]
user=mysql
bind-address=0.0.0.0
port={{ db_port }}
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Performance and tuning
max_connections=200
innodb_buffer_pool_size=1G
innodb_log_file_size=256M

[client]
port={{ db_port }}
socket=/var/lib/mysql/mysql.sock
```

**Explanation:**
- `bind-address`: Allows remote connections (`0.0.0.0`).
- `port`: Dynamic port assignment using `{{ db_port }}`.
- `max_connections`: Limits the number of concurrent connections.
- `innodb_*`: Configures InnoDB engine parameters.
- `client`: Ensures client configurations align with server settings.

---

## 📄 **files/init_db.sql**

```sql
-- Create a sample database
CREATE DATABASE IF NOT EXISTS sample_db;

-- Switch to the sample database
USE sample_db;

-- Create a sample table
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO users (username, email) VALUES ('admin', 'admin@example.com');
INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');

-- Create a dedicated database user
CREATE USER IF NOT EXISTS 'app_user'@'%' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON sample_db.* TO 'app_user'@'%';
FLUSH PRIVILEGES;
```

**Explanation:**
- `CREATE DATABASE`: Ensures a database named `sample_db` exists.
- `CREATE TABLE`: Defines a `users` table with sample fields.
- `INSERT INTO`: Adds sample records for testing.
- `CREATE USER`: Sets up a database user (`app_user`) with privileges on `sample_db`.
- `GRANT`: Grants all necessary permissions to the user.
- `FLUSH PRIVILEGES`: Ensures privilege changes take effect.
---
## 🧪 **2. test_db_content.yml**

### **Description:**  
This playbook validates the database server's functionality by running test queries.

```yaml
---
- name: Test database content
  hosts: workstation
  become: true

  vars:
    db_test_queries:
      - "SHOW DATABASES;"
      - "SELECT NOW();"
    db_host: dbserver1.lab.example.com

  tasks:
    # Display system facts such as OS, hostname, memory, and CPU details
    - name: Display system facts
      ansible.builtin.debug:
        msg:
          - "Operating System: {{ ansible_facts['os_family'] }}"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Memory Total: {{ ansible_facts['memtotal_mb'] }} MB"
          - "CPU Cores: {{ ansible_facts['processor_cores'] }}"

    # Run database queries
    - name: Run database queries
      block:
        - name: Execute database queries
          ansible.builtin.shell:
            cmd: "mysql -h {{ db_host }} -e '{{ item }}'"
          register: query_result
      rescue:
        # Log error to a file if the query fails
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/review-cr2/db_error.log
            line: "Error executing query '{{ item }}': {{ query_result }}"
            create: true
      loop: "{{ db_test_queries }}"
```

---

## 📋 **3. site.yml**

### **Description:**  
This master playbook imports and executes `db_deploy.yml` and `test_db_content.yml` sequentially.

```yaml
---
# Deploy and configure database servers
- name: Deploy database servers
  ansible.builtin.import_playbook: db_deploy.yml

# Validate the database servers by running test queries
- name: Test database content
  ansible.builtin.import_playbook: test_db_content.yml
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
| dbserver1.lab.example.com | 7  | 6       | 0          | 0      | 0       | 0       | 0       |
| workstation           | 2  | 0       | 0          | 0      | 0       | 0       | 0       |

---

## 📖 **Key Concepts Used**

- **Facts:** Display system-specific information.
- **Variables:** Manage packages, ports, and services.
- **Handlers:** Restart services after changes.
- **Templates:** Jinja2 for dynamic configurations.
- **Error Handling:** `block` and `rescue` for graceful failure management.


---




## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
---
- name: Install and configure web servers
  hosts: webservers
  become: true

  vars:
    web_packages:
      - httpd
      - firewalld


  tasks:
    # Removed previously installed packages
    - name: Uninstall packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: absent
      loop: "{{ web_packages }}"

```
   ```bash
   ansible-navigator run -m stdout playbook_clean.yml
   ```
