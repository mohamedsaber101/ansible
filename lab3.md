# 🚀 **Ansible Playbook: WildFly Application Deployment and Nginx Load Balancer**

This project contains three Ansible playbooks to automate the deployment, configuration, and validation of WildFly application servers and an Nginx load balancer.

## 📂 **Project Structure**

```
/ansible-deployment
├── wildfly_deploy.yml   # Playbook for deploying WildFly application servers
├── nginx_deploy.yml     # Playbook for configuring Nginx as a load balancer
├── site.yml             # Master playbook to run both playbooks
├── roles/
│   ├── wildflyapp/
│   │   ├── tasks/main.yml
│   │   ├── files/
│   │   │   ├── example-jaxrs-war-swarm.jar
│   │   │   └── wildflyapp.service
│   └── nginx/
│       ├── tasks/main.yml
│       └── templates/nginx.conf.j2
├── inventory
```

---

## 🛠️ **1. wildfly_deploy.yml**

### **Description:**  
This playbook installs and configures WildFly application servers on managed hosts.

```yaml
---
- name: Install and configure WildFly application servers
  hosts: wildflyservers
  become: true

  vars:
    wildfly_package: java-1.8.0-openjdk-headless
    wildfly_service: wildflyapp

  tasks:
    # Install Java runtime
    - name: Install Java runtime
      ansible.builtin.package:
        name: "{{ wildfly_package }}"
        state: present

    # Create application directory
    - name: Create WildFly application directory
      ansible.builtin.file:
        path: /opt/wildflyapp
        state: directory

    # Copy application JAR file
    - name: Copy WildFly application JAR
      ansible.builtin.copy:
        src: example-jaxrs-war-swarm.jar
        dest: /opt/wildflyapp/example-jaxrs-war-swarm.jar
        mode: '0644'

    # Deploy WildFly systemd service
    - name: Deploy WildFly service
      ansible.builtin.copy:
        src: wildflyapp.service
        dest: /etc/systemd/system/wildflyapp.service
        mode: '0644'
      notify: Reload systemd

    # Enable and start WildFly service
    - name: Enable and start WildFly service
      ansible.builtin.systemd:
        name: "{{ wildfly_service }}"
        enabled: yes
        state: started

  handlers:
    - name: Reload systemd
      ansible.builtin.systemd:
        daemon_reload: yes
```

---

## 🧪 **2. nginx_deploy.yml**

### **Description:**  
This playbook installs and configures Nginx as a load balancer for WildFly servers.

```yaml
---
- name: Configure Nginx load balancer
  hosts: loadbalancer
  become: true

  vars:
    nginx_package: nginx
    nginx_config: /etc/nginx/nginx.conf

  tasks:
    # Install Nginx
    - name: Install Nginx
      ansible.builtin.package:
        name: "{{ nginx_package }}"
        state: present

    # Deploy Nginx configuration
    - name: Deploy Nginx configuration
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: "{{ nginx_config }}"
        mode: '0644'
      notify: Reload Nginx

    # Enable and start Nginx service
    - name: Enable and start Nginx service
      ansible.builtin.systemd:
        name: nginx
        enabled: yes
        state: started

  handlers:
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
This master playbook imports and executes `wildfly_deploy.yml` and `nginx_deploy.yml` sequentially.

```yaml
---
# Deploy WildFly application servers
- name: Deploy WildFly servers
  ansible.builtin.import_playbook: wildfly_deploy.yml

# Configure Nginx load balancer
- name: Configure Nginx load balancer
  ansible.builtin.import_playbook: nginx_deploy.yml
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

Happy Automating! 🚀✨
