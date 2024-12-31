Ansible Playbook: Web Server Deployment and Testing
This project contains three Ansible playbooks to automate the deployment, configuration, and validation of web servers.

📂 Project Structure
bash
Code kopieren
/review-cr2
├── dev_deploy.yml      # Playbook for deploying and configuring web servers
├── get_web_content.yml # Playbook for testing web server content
├── site.yml            # Master playbook to run both playbooks
├── templates/
│   └── vhost.conf.j2   # Jinja2 template for virtual host configuration
└── files/
    └── index.html      # Sample web content file
🛠️ 1. dev_deploy.yml
Description:
This playbook installs and configures Apache HTTP servers on managed hosts.

<details> <summary><strong>Click to expand playbook</strong></summary>
yaml
Code kopieren
---
- name: Install and configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install httpd package
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Start httpd service
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

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

  handlers:
    - name: Restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted
</details>
🧪 2. get_web_content.yml
Description:
This playbook validates the web server's functionality by retrieving its content.

<details> <summary><strong>Click to expand playbook</strong></summary>
yaml
Code kopieren
---
- name: Test web content
  hosts: workstation
  become: true

  tasks:
    - name: Retrieve web content and write to error log on failure
      block:
        - name: Retrieve web content
          ansible.builtin.uri:
            url: http://servera.lab.example.com
            return_content: true
          register: content
      rescue:
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/review-cr2/error.log
            line: "{{ content }}"
            create: true
</details>
📋 3. site.yml
Description:
This master playbook imports and executes dev_deploy.yml and get_web_content.yml sequentially.

<details> <summary><strong>Click to expand playbook</strong></summary>
yaml
Code kopieren
---
- name: Deploy web servers
  ansible.builtin.import_playbook: dev_deploy.yml

- name: Retrieve web content
  ansible.builtin.import_playbook: get_web_content.yml
</details>
🚦 How to Run the Playbooks
Navigate to the project directory:

bash
Code kopieren
cd /home/student/review-cr2
Run the playbooks using ansible-navigator:

bash
Code kopieren
ansible-navigator run -m stdout site.yml
Expected Output:

All tasks should complete successfully.
No errors should appear in the output.
✅ Expected Results
Host	OK	Changed	Unreachable	Failed	Skipped	Rescued	Ignored
servera.lab.example.com	7	6	0	0	0	0	0
serverb.lab.example.com	7	6	0	0	0	0	0
workstation	2	0	0	0	0	0	0
📖 Key Concepts Used
Handlers: To restart services after configuration changes.
Templates: Jinja2 templates for dynamic configurations.
Error Handling: block and rescue for graceful failure management.
Idempotency: Tasks ensure no unnecessary changes occur on repeated runs.
