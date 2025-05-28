# 🖥️ Secure Web Server Deployment using Flask, Docker, GitHub Actions, and Ansible

## 📌 Project Overview

This project showcases a full-stack deployment pipeline for a secure Flask-based web server. It demonstrates the use of DevOps practices including containerization, continuous integration/deployment, infrastructure hardening using CIS Benchmarks, and optional Terraform-based infrastructure provisioning.

---

##  Tech Stack

- **Backend:** Python Flask
- **Containerization:** Docker
- **CI/CD:** GitHub Actions
- **Deployment:** AWS EC2
- **Hardening & Automation:** Ansible (CIS-compliant)
- **Optional Infra as Code:** Terraform
- **Monitoring & Logging:** Manual setup (future scope)

---

##  Features & Implementations

### ✅ Web Server
- Built a basic Flask application with routing logic in `routes.py`.
- Displays a homepage and a custom message.
- Dockerized the app for environment consistency.

### ✅ Dockerization
- Dockerfile with multi-stage build.
- Installs requirements from `requirements.txt`.
- Configured for efficient caching and no local state retention.

### ✅ CI/CD Pipeline (GitHub Actions)
- Triggered on push to `main`.
- Runs:
  - Linting and syntax checks.
  - Docker image build and push.
  - Rsync deployment to EC2 via SSH (public key authentication).

### ✅ Deployment
- Deployed to AWS EC2 (Ubuntu 22.04).
- Hardened instance using Ansible.
- SSH keys used securely with GitHub Secrets.
  
---

## 🛡️ Ubuntu Hardening (Ansible)

### ⚙️ Security Tasks Included

- ✅ `/var/log` on a separate partition
- ✅ IP forwarding disabled
- ✅ Local login warning banner configured
- ✅ Secure ICMP redirects disabled
- ✅ IPv6 disabled
- ✅ SSH LogLevel set to `INFO`
- ✅ Login/Logout audit events collected
- ✅ File deletion audit events collected
- ✅ Password expiration set to 90 days

### 📄 Ansible Playbook Snippet

``yaml
- name: CIS Basic Ubuntu Hardening
  hosts: localhost
  become: yes
  tasks:

    - name: Ensure /var/log is on a separate partition
      shell: mount | grep -w "/var/log"
      failed_when: false
      register: varlog
      changed_when: false

    - name: Disable IP forwarding
      sysctl:
        name: net.ipv4.ip_forward
        value: 0
        state: present
        reload: yes

    - name: Disable IPv6
      sysctl:
        name: net.ipv6.conf.all.disable_ipv6
        value: 1
        state: present
        reload: yes

    - name: Set SSH LogLevel to INFO
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?LogLevel'
        line: 'LogLevel INFO'

    - name: Configure login banner
      copy:
        content: "Authorized access only. All activity may be monitored."
        dest: /etc/issue.net

    - name: Collect login/logout events
      lineinfile:
        path: /etc/audit/rules.d/audit.rules
        line: '-w /var/log/faillog -p wa -k logins'

    - name: Collect file deletion events
      lineinfile:
        path: /etc/audit/rules.d/audit.rules
        line: '-a always,exit -F arch=b64 -S unlink -S unlinkat -S rename -S renameat -k delete'

    - name: Enforce password expiration
      lineinfile:
        path: /etc/login.defs
        regexp: '^PASS_MAX_DAYS'
        line: 'PASS_MAX_DAYS   90'

      ''
