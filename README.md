# 🖥️ Ansible 

This project demonstrates configuring two AWS EC2 instances (VM1 & VM2) and deploying a webserver on each using **Ansible**.  
Each server listens on **port 8080** and serves a web page with:

- **VM1 →** `Hello World from SJSU-1`  
- **VM2 →** `Hello World from SJSU-2`

The playbook also supports **undeployment** (stopping/removing the webserver and configs).

---

## 📂 Project Structure
```
ansible/
├── inventory.ini            # Inventory file with VM IPs and SSH details
├── site.yml                 # Ansible playbook (deploy + undeploy)
├── group_vars/
│   └── all.yml              # Common variables (port, site root, etc.)
├── host_vars/
│   ├── vm1.yml              # Variables specific to VM1 (app_id=1)
│   └── vm2.yml              # Variables specific to VM2 (app_id=2)
└── templates/
    └── index.html.j2        # Jinja2 template for index.html
```
---

## ⚙️ Setup

### 1. Prerequisites
- Control machine: macOS (preferred) with **Ansible** installed via [Homebrew](https://brew.sh/)
- Two AWS EC2 Ubuntu 22.04 instances (`t2.micro` recommended)
- Security Group with ports **22 (SSH)** and **8080 (Custom TCP)** open
- SSH key pair (`sjsu-key.pem`) added to `~/.ssh/sjsu-key.pem` and registered in your GitHub account

### 2. Configure Inventory
Edit `ansible/inventory.ini` with your instance IPs and username (for Ubuntu use `ubuntu`):

```ini
[web]
vm1 ansible_host=54.215.94.80 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/sjsu-key.pem
vm2 ansible_host=52.53.227.102 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/sjsu-key.pem
```

### 3. Test Ansible Connectivity
```bash
cd ansible
ansible -i inventory.ini web -m ping
```

Expected output:
```
vm1 | SUCCESS => {"changed": false, "ping": "pong"}
vm2 | SUCCESS => {"changed": false, "ping": "pong"}
```

---

## 🚀 Deploy Webservers

Deploy webservers on both VMs (nginx on Ubuntu) listening on **port 8080**:

```bash
ansible-playbook -i inventory.ini site.yml --tags deploy
```

What happens:
- Installs **nginx**  
- Creates `/var/www/sjsu` folder  
- Places a templated `index.html` file that says *Hello World from SJSU-1* or *SJSU-2*  
- Configures nginx to serve on port 8080  
- Starts and enables the nginx service  

### Verify Deployment
Open in your browser:
- [http://54.215.94.80:8080](http://54.215.94.80:8080) → should show **Hello World from SJSU-1**  
- [http://52.53.227.102:8080](http://52.53.227.102:8080) → should show **Hello World from SJSU-2**  

Or test via curl:
```bash
curl -i http://54.215.94.80:8080
curl -i http://52.53.227.102:8080
```

---

## 🗑️ Undeploy Webservers

To stop and remove nginx + site configs:

```bash
ansible-playbook -i ansible/site.yml --tags undeploy
```

Verify:
```bash
curl -I http://54.215.94.80:8080
curl -I http://52.53.227.102:8080
```
You should see a **connection refused** or **404** after undeploy.



---


Repo link: [https://github.com/Mominuddin07/Ansible](https://github.com/Mominuddin07/Ansible)

---

## 👤 Author
- **Name:** _Mohammed Mominuddin_  
- **Course:** SJSU — Enterprice Technologies - Andrew Bond Sir  
