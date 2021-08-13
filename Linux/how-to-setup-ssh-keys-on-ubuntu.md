**Table of Contents**
- [1. How to Setup SSH Keys on Ubuntu](#1-how-to-setup-ssh-keys-on-ubuntu)
  - [1.1. Install SSH Server](#11-install-ssh-server)
  - [1.2. Connecting to SSH](#12-connecting-to-ssh)
  - [1.3. Disable SSH on Ubuntu](#13-disable-ssh-on-ubuntu)
  - [1.4. Connecting to SSH Using Public Key](#14-connecting-to-ssh-using-public-key)
    - [1.4.1. Generate Public/Private Keypair on Your Ubuntu Machine](#141-generate-publicprivate-keypair-on-your-ubuntu-machine)
    - [1.4.2. Upload Your Public Key to Remote Linux Server(Ubuntu)](#142-upload-your-public-key-to-remote-linux-serverubuntu)
- [2. Probelm](#2-probelm)
  - [2.1. Error: ssh connection closed by \<ip-address\> port 22](#21-error-ssh-connection-closed-by-ip-address-port-22)

# 1. How to Setup SSH Keys on Ubuntu

## 1.1. Install SSH Server
Install `openssh-server`
```
sudo apt update
sudo apt install openssh-server
```

Check ssh status
```
sudo systemctl status ssh
```

You should see the output: `Active: active (running)`
```bash
# Output
#
# ● ssh.service - OpenBSD Secure Shell server
#    Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-03-22 08:51:18 UTC; 15min ago
#   Process: 862 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
#  Main PID: 932 (sshd)
#     Tasks: 1 (limit: 2317)
#    CGroup: /system.slice/ssh.service
#            └─932 /usr/sbin/sshd -D
#
# Mar 22 08:51:18 ubuntu-server-18-04 systemd[1]: Starting OpenBSD Secure Shell server...
# Mar 22 08:51:18 ubuntu-server-18-04 sshd[932]: Server listening on 0.0.0.0 port 22.
# Mar 22 08:51:18 ubuntu-server-18-04 sshd[932]: Server listening on :: port 22.
# Mar 22 08:51:18 ubuntu-server-18-04 systemd[1]: Started OpenBSD Secure Shell server.
# Mar 22 09:03:15 ubuntu-server-18-04 sshd[1346]: Accepted password for jiahao from 192.168.0.203 port 54807 ssh2
# Mar 22 09:03:15 ubuntu-server-18-04 sshd[1346]: pam_unix(sshd:session): session opened for user jiahao by (uid=0)
```

Ubuntu comes with a firewall configuration tool called UFW. If the firewall is enabled on your system, make sure to open the SSH port:
```
sudo ufw allow ssh
```

## 1.2. Connecting to SSH
```bash
# ssh username@ip_address
ssh jiahao@192.168.0.189
```

When you connecting through SSH for the first time, you will see the following message
Type `yes` and will prompt to enter password to login
```bash
# Output
# 
# The authenticity of host '192.168.0.189 (192.168.0.189)' can't be established.
# ECDSA key fingerprint is SHA256:f/r8ZXfnZfrDg8TzXtCDjPghicx+sGMAIuXC9zcvCNQ.
# Are you sure you want to continue connecting (yes/no)? yes
# 
# Warning: Permanently added '192.168.0.189' (ECDSA) to the list of known hosts.
# jiahao@192.168.0.189's password:
```

## 1.3. Disable SSH on Ubuntu
If you want to disable SSH, you can run the command below to stop SSH service
```
sudo systemctl stop ssh
```

To start SSH
```
sudo systemctl start ssh
```

To disable SSH
```
sudo systemctl disable ssh
```

To enable SSH
```
sudo systemctl enable ssh
```

## 1.4. Connecting to SSH Using Public Key
### 1.4.1. Generate Public/Private Keypair on Your Ubuntu Machine

Run the command to generate SSH keypair
```
ssh-keygen -t rsa -b 4096
```

> - `-t` : Type. Keypair type, RSA is the default type.
> - `-b` : Bits. Default key is 3072 bits long. 4096 bits key for stronger security

```bash
# Output
#
# Generating public/private rsa key pair.
# Enter file in which to save the key (/home/jiahao/.ssh/id_rsa):
# Enter passphrase (empty for no passphrase):
# Enter same passphrase again:
# Your identification has been saved in /home/jiahao/.ssh/id_rsa.
# Your public key has been saved in /home/jiahao/.ssh/id_rsa.pub.
# The key fingerprint is:
# SHA256:******************************************* jiahao@ubuntu-server-18-04
# The key's randomart image is:
# +---[RSA 4096]----+
# ...
# +----[SHA256]-----+
```

Check the keypair
```
file ~/.ssh/id_rsa
```

If you see `(No such file or directory)` error, that mean the SSH keypair isn't create, your can run `ssh-keygen -t rsa - b 4096`
```bash
# Output
# 
# /home/jiahao/.ssh/id_rsa: PEM RSA private key
```

### 1.4.2. Upload Your Public Key to Remote Linux Server(Ubuntu)
Run the command on your client machine with `openssh-client` package
```bash
# For linux machine
# ssh-copy-id remote-user@server-ip

ssh-copy-id jiahao@192.168.0.189

# For windows machine
# type $env:USERPROFILE\.ssh\id_rsa.pub | ssh {IP-ADDRESS-OR-FQDN} "cat >> .ssh/authorized_keys"

type $env:USERPROFILE\.ssh\id_rsa.pub | ssh jiahao@192.168.0.189 "cat >> .ssh/authorized_keys"
#jiahao@192.168.0.189's password:
```

> **Note**: If your client machine dont have SSH keypair, can run `ssh-keygen -t rsa -b 4096` to generate 

The public key will stored in `~/.ssh/authorized_keys`, run the command to login your server again
```bash
# ssh username@ip_address
# ssh jiahao@192.168.0.189
```

Check the authorized keys added on your server
```
cat ~/.ssh/authorized_keys
```

# 2. Probelm

## 2.1. Error: ssh connection closed by \<ip-address\> port 22

Run command below to remove all `ssh_host_` file and reconfigure ssh
```
sudo rm /etc/ssh/ssh_host_*
sudo dpkg-reconfigure openssh-server
```

