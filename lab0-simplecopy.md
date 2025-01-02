# 🚀 **Ansible Playbooks: Simple File Operations with Copy and Lineinfile Modules**

This project contains three simple Ansible playbooks demonstrating the use of the `copy` and `lineinfile` modules.

## 📂 **Project Structure**

```
/simple-playbooks
├── playbook1.yml   # Write a simple line to a file
├── playbook2.yml   # Write a line with system facts
├── playbook3.yml   # Replace a specific string in a file
├── playbook_clean.yml   # clean up the environment
├── inventory       # Inventory file for target hosts
```

---

## 🛠️ **1. Playbook 1: Write a Simple Line to a File**

**Playbook:** `playbook1.yml`

```yaml
---
- name: Write a simple line to a file
  hosts: all
  become: true

  tasks:
    - name: Write a static line to a file
      ansible.builtin.copy:
        dest: /tmp/simple_file.txt
        content: "This is a static line written by Ansible."
        owner: root
        group: root
        mode: '0644'
```

### **Explanation:**
- Creates a file `/tmp/simple_file.txt`.
- Writes a static line to the file.
- Ensures the correct ownership and permissions.

### **Run the Playbook:**
```bash
ansible-playbook -i inventory playbook1.yml
```

---

## 🧪 **2. Playbook 2: Write a Line with System Facts**

**Playbook:** `playbook2.yml`

```yaml
---
- name: Write system facts to a file
  hosts: all
  become: true

  tasks:
    - name: Write CPU, Memory, and IP information to a file
      ansible.builtin.copy:
        dest: /tmp/facts_file.txt
        content: |
          CPU: {{ ansible_processor[0] }}
          Memory: {{ ansible_memtotal_mb }} MB
          IP Address: {{ ansible_default_ipv4.address }}
        owner: root
        group: root
        mode: '0644'
```

### **Explanation:**
- Gathers system facts (`ansible_processor`, `ansible_memtotal_mb`, `ansible_default_ipv4.address`).
- Writes CPU, Memory, and IP Address information to `/tmp/facts_file.txt`.
- Ensures proper ownership and permissions.

### **Run the Playbook:**
```bash
ansible-playbook -i inventory playbook2.yml
```

---

## ✏️ **3. Playbook 3: Replace a Specific String in a File**

**Playbook:** `playbook3.yml`

```yaml
---
- name: Replace a specific string in a file
  hosts: all
  become: true

  tasks:
    - name: Replace 'static' with 'dynamic' in the file
      ansible.builtin.lineinfile:
        path: /tmp/simple_file.txt
        regexp: 'static'
        line: 'This is a dynamic line written by Ansible.'
```

### **Explanation:**
- Searches for the word `static` in `/tmp/simple_file.txt`.
- Replaces it with the word `dynamic`.

### **Run the Playbook:**
```bash
ansible-playbook -i inventory playbook3.yml
```

---

## ✅ **Expected Results**

1. **Playbook 1:** Creates `/tmp/simple_file.txt` with a static line.
2. **Playbook 2:** Creates `/tmp/facts_file.txt` with system facts.
3. **Playbook 3:** Updates `/tmp/simple_file.txt`, replacing `static` with `dynamic`.

---

## 📖 **Key Concepts Used**

- **Copy Module:** Write static or dynamic content to files.
- **System Facts:** Retrieve CPU, memory, and IP information.
- **Lineinfile Module:** Replace or modify specific lines in files.

---

## 🚦 **4. Clean Up the Environment**

After testing, clean up the environment:
**Playbook:** `playbook_clean.yml`
```yaml
---
- name: Clean nodes
  hosts: all
  become: true

  tasks:
    - name: Remove previously created files
      ansible.builtin.file:
        path: "{{ item }}"
        state: absent
      loop:
      - /tmp/simple_file.txt
      - /tmp/facts_file.txt
```
### **Run the Playbook:**
```bash
ansible-playbook -i inventory playbook_clean.yml
```
---

Happy Automating! 🚀✨
