# 🚀 **Ansible Playbook: WildFly Application Deployment and Nginx Load Balancer**

This project contains three Ansible playbooks to automate the deployment, configuration, and validation of WildFly application servers and an Nginx load balancer.

## 📂 **Project Structure**

```
/ansible-deployment
├── site.yml             # Master playbook to run both roles
├── inventory
├── roles/
│   ├── wildflyapp/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── files/
│   │   │   ├── example-jaxrs-war-swarm.jar
│   │   │   └── wildflyapp.service
│   │   └── templates/
│   └── nginx/
│       ├── tasks/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       └── templates/
│           └── nginx.conf.j2
```

---

## 🛠️ **1. roles/wildflyapp**

### **Description:**  
This role installs and configures WildFly application servers on managed hosts.

**Tasks File:** `roles/wildflyapp/tasks/main.yml`

```yaml
---
- name: Install Java runtime
  ansible.builtin.package:
    name: java-1.8.0-openjdk-headless
    state: present

- name: Create WildFly application directory
  ansible.builtin.file:
    path: /opt/wildflyapp
    state: directory

- name: Copy WildFly application JAR
  ansible.builtin.copy:
    src: app.jar
    dest: /opt/wildflyapp/app.jar
    mode: '0644'

- name: Deploy WildFly service
  ansible.builtin.copy:
    src: wildflyapp.service
    dest: /etc/systemd/system/wildflyapp.service
    mode: '0644'
  notify: Reload systemd

- name: Enable and start WildFly service
  ansible.builtin.systemd:
    name: wildflyapp
    enabled: yes
    state: started
```

**Handlers File:** `roles/wildflyapp/handlers/main.yml`

```yaml
---
- name: Reload systemd
  ansible.builtin.systemd:
    daemon_reload: yes

- name: Restart WildFly service
  ansible.builtin.systemd:
    name: wildflyapp
    state: restarted
```

**Files:**
- `app.jar`: WildFly application JAR file.
  ```
   @ /home/student/artifacts/app.jar directory
  ```
- `wildflyapp.service`: Systemd service configuration.
  
```
[Unit]
Description=Wildfly Swarm Application Script
After=auditd.service systemd-user-sessions.service time-sync.target

[Service]
User=root
TimeoutStartSec=0
Type=simple
KillMode=process
WorkingDirectory=/opt/wildflyapp
ExecStart=/bin/java -jar /opt/wildflyapp/app.jar
Restart=always
RestartSec=2
LimitNOFILE=5555

[Install]
WantedBy=multi-user.target
```

---

## 🧪 **2. roles/nginx**

### **Description:**  
This role installs and configures Nginx as a load balancer for WildFly servers.

**Tasks File:** `roles/nginx/tasks/main.yml`

```yaml
---
- name: Install Nginx
  ansible.builtin.package:
    name: nginx
    state: present

- name: Deploy Nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    mode: '0644'
  notify: Reload Nginx

- name: Enable and start Nginx service
  ansible.builtin.systemd:
    name: nginx
    enabled: yes
    state: started
```

**Handlers File:** `roles/nginx/handlers/main.yml`

```yaml
---
- name: Reload Nginx
  ansible.builtin.systemd:
    name: nginx
    state: reloaded
```

**Template File:** `roles/nginx/templates/nginx.conf.j2`

```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    upstream wildfly_backend {
        server wildfly1.example.com:8080;
        server wildfly2.example.com:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://wildfly_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

---

## 📋 **3. site.yml**

### **Description:**  
This master playbook imports and executes the `wildflyapp` and `nginx` roles sequentially.

```yaml
---
- name: Deploy WildFly application servers
  hosts: wildflyservers
  become: true
  roles:
    - wildflyapp

- name: Configure Nginx load balancer
  hosts: loadbalancer
  become: true
  roles:
    - nginx
```

---

## 🚦 **How to Run the Playbooks**

1. **Navigate to the project directory:**
   ```bash
   cd /ansible-deployment
   ```

2. **Run the playbooks using `ansible-playbook`:**
   ```bash
   ansible-playbook -i inventory site.yml
   ```

3. **Expected Output:**
   - All tasks should complete successfully.
   - WildFly servers should be running.
   - Nginx should forward requests to the backend WildFly servers.

---

## ✅ **Expected Results**

| Host                 | OK | Changed | Unreachable | Failed | Skipped | Rescued | Ignored |
|-----------------------|----|---------|------------|--------|---------|---------|---------|
| wildfly1.example.com | 6  | 5       | 0          | 0      | 0       | 0       | 0       |
| wildfly2.example.com | 6  | 5       | 0          | 0      | 0       | 0       | 0       |
| loadbalancer         | 3  | 2       | 0          | 0      | 0       | 0       | 0       |

---

**Hitting the loadbalancer `ansible-playbook`:**
### **Description:**  
Execute the following command multiple times to verify that the load balancer is functioning properly with the backend servers.
   ```bash
   curl node1
   ```

## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
---
- name: Clean the environment
  hosts: all
  become: true

  vars:
    packages:
      - nginx
      -  java-headless


  tasks:
    # Removed previously installed packages
    - name: Uninstall packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: absent
      loop: "{{ packages }}"

```
   ```bash
   ansible-navigator run -m stdout playbook_clean.yml
   ```
