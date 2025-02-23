# 🚀 Vagrant Virtual Machine Automation - Multi-VM Setup

---

## 📖 Overview
This project automates the setup of multiple virtual machines using Vagrant and VirtualBox. The setup includes a **scriptbox** VM and three web servers (**web01, web02, web03**) with SSH-based authentication and remote access configurations.

---

## 📑 Table of Contents
- [Prerequisites](#-prerequisites) 🔑
- [Architecture](#-architecture) 🗺️
- [Setup & Installation](#-setup-and-installation) 🛠️
- [Vagrant Setup](#-vagrant-setup) 🐳
- [Key-Based Authentication](#-key-based-authentication) 🔑
- [Cleaning Up Resources](#-cleaning-up-resources) 🧹
- [Conclusion](#-conclusion) ✅

---

## 🔑 Prerequisites
Before starting, ensure you have the following installed:

- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- A terminal or command prompt with administrative privileges

---

## 🗺️ Architecture
This project sets up four virtual machines with the following specifications:

- **scriptbox (192.168.10.12)** - Controls remote access to web servers.
- **web01 (192.168.10.13)** - Web server 1 (CentOS Stream 9)
- **web02 (192.168.10.14)** - Web server 2 (CentOS Stream 9)
- **web03 (192.168.10.15)** - Web server 3 (Ubuntu Jammy 64-bit)

### Workflow:
1. **Initialize Vagrant** – Start the VMs.
2. **Modify /etc/hosts** – Map hostnames to IPs.
3. **Create Users & Grant Sudo Access** – Add `devops` user with password-less sudo.
4. **Setup SSH Key Authentication** – Enable key-based access for `devops` user.
5. **Verify Remote Access** – SSH into the web servers from `scriptbox`.

---

## 🛠️ Setup & Installation

### 1⃣ Initialize Vagrant:
```bash
vagrant up
```

### 2⃣ Modify `/etc/hosts` on scriptbox:
```bash
sudo vim /etc/hosts
```
Add the following lines:
```plaintext
192.168.10.13 web01  
192.168.10.14 web02  
192.168.10.15 web03  
```

### 3⃣ Create `devops` User & Grant Sudo Access
For each web server, execute:
```bash
ssh vagrant@web01
sudo useradd devops
sudo passwd devops
sudo visudo
- devops ALL=(ALL) NOPASSWD: ALL (add the line in configuration file.)  
```
Repeat for `web02` and `web03`.

### 4⃣ Resolve SSH Issues for `web03`
Edit `/etc/ssh/sshd_config` on `web03`:
```bash
sudo vim /etc/ssh/sshd_config
```
Modify this line:
```plaintext
PasswordAuthentication yes
```
Restart SSH service:
```bash
sudo systemctl restart sshd
```
Alternatively, modify the `Vagrantfile`:
```ruby
web03.ssh.insert_key = false
```

---

## 🐳 Vagrant Setup
Below is the `Vagrantfile` used for this project:
```ruby
Vagrant.configure("2") do |config|
  config.vm.define "scriptbox" do |scriptbox|
    scriptbox.vm.box = "eurolinux-vagrant/centos-stream-9"
    scriptbox.vm.network "private_network", ip: "192.168.10.12"
    scriptbox.vm.hostname = "scriptbox"
    scriptbox.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
  end

  config.vm.define "web01" do |web01|
    web01.vm.box = "eurolinux-vagrant/centos-stream-9"
    web01.vm.network "private_network", ip: "192.168.10.13"
    web01.vm.hostname = "web01"
  end
  
  config.vm.define "web02" do |web02|
    web02.vm.box = "eurolinux-vagrant/centos-stream-9"
    web02.vm.network "private_network", ip: "192.168.10.14"
    web02.vm.hostname = "web02"
  end

  config.vm.define "web03" do |web03|
    web03.vm.box = "ubuntu/jammy64"
    web03.vm.network "private_network", ip: "192.168.10.15"
    web03.vm.hostname = "web03"
    web03.ssh.insert_key = false
  end
end
```

---

## 🔑 Key-Based Authentication
### 1⃣ Generate SSH Key:
```bash
ssh-keygen
```
### 2⃣ Copy Public Key to Web Servers:
```bash
ssh-copy-id devops@web01
ssh-copy-id devops@web02
ssh-copy-id devops@web03
```
### 3⃣ Test SSH Access Without Password:
```bash
ssh devops@web01 uptime
ssh devops@web02 uptime
ssh devops@web03 uptime
```

---

## 🧹 Cleaning Up Resources

To remove the VMs and free up system resources:

### 1⃣ Destroy the Virtual Machines:
```bash
vagrant destroy -f
```
### 2⃣ Remove Unused Vagrant Instances:
```bash
vagrant global-status --prune
```

---

## ✅ Conclusion
This project automates the setup of multiple VMs using Vagrant and VirtualBox, configuring remote SSH access and key-based authentication for easy management.

---

## 👨‍🏫 Instructor
This project was guided by **Imran Teli**, who provided valuable mentorship throughout the process.

