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

<details>
<summary><strong>Click to expand playbook</strong></summary>

```yaml
---
- name: Install and configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Display system facts
      ansible.builtin.debug:
        msg:
          - "Operating System: {{ ansible_facts['os_family'] }}"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Memory Total: {{ ansible_facts['memtotal_mb'] }} MB"
          - "CPU Cores: {{ ansible_facts['processor_cores'] }}"

    - name: Install required packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop:
        - httpd
        - firewalld

    - name: Start services
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop:
        - httpd
        - firewalld

    - name: Deploy configuration template
      ansible.builtin.template:
        src: templates/vhost.conf.j2
        dest: /etc/httpd/conf.d/vhost.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart httpd

    - name: Copy index.html
      ansible.builtin.copy:
        src: files/
        dest: "/var/www/vhosts/{{ ansible_facts['hostname'] }}/"
        owner: root
        group: root
        mode: '0644'

    - name: Ensure web server port is open
      ansible.posix.firewalld:
        state: enabled
        permanent: true
        immediate: true
        service: http
      when: ansible_facts['os_family'] == 'RedHat'

  handlers:
    - name: Restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted
```

</details>

---

## 🧪 **2. get_web_content.yml**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

<details>
<summary><strong>Click to expand playbook</strong></summary>

```yaml
---
- name: Test web content
  hosts: workstation
  become: true

  tasks:
    - name: Display system facts
      ansible.builtin.debug:
        msg:
          - "Operating System: {{ ansible_facts['os_family'] }}"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Memory Total: {{ ansible_facts['memtotal_mb'] }} MB"
          - "CPU Cores: {{ ansible_facts['processor_cores'] }}"

    - name: Retrieve web content and write to error log on failure
      block:
        - name: Retrieve web content
          ansible.builtin.uri:
            url: http://{{ item }}
            return_content: true
          register: content
      rescue:
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/review-cr2/error.log
            line: "Error retrieving content from {{ item }}: {{ content }}"
            create: true
      loop:
        - servera.lab.example.com
        - serverb.lab.example.com
```

</details>

---

## 📋 **3. site.yml**

### **Description:**  
This master playbook imports and executes `dev_deploy.yml` and `get_web_content.yml` sequentially.

<details>
<summary><strong>Click to expand playbook</strong></summary>

```yaml
---
- name: Deploy web servers
  ansible.builtin.import_playbook: dev_deploy.yml

- name: Retrieve web content
  ansible.builtin.import_playbook: get_web_content.yml
```

</details>

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

## 📖 **Key Concepts Used**

- **Facts:** Display system-specific information.
- **Loops:** Repeat tasks for multiple items.
- **Conditions:** Tasks execute based on OS family.
- **Handlers:** Restart services after changes.
- **Templates:** Jinja2 for dynamic configurations.
- **Error Handling:** `block` and `rescue` for graceful failure management.

---

Happy Automating! 🚀✨
