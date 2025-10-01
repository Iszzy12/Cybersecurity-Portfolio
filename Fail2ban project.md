## SSH Brute-Force Lab – From Zero to Ban
Build an SSH honeypot, attack it with Hydra, and watch Fail2Ban react in real time.

## 0. Lab sheet
| Role | OS | IP | Account |
|------|----|----|---------|
| Target | Ubuntu 22.04 | 192.168.1.44 | `ubuntu` (your normal user) |
| Attacker | Parrot OS 6.x | 192.168.1.237 | `israel` (any user) |

Network: Host-only (VirtualBox) – **never expose to internet**.

## 1. Target – install & harden SSH

## 1.1 update & install openssh-server
sudo apt update && sudo apt install -y openssh-server fail2ban

# 1.2 move SSH to non-default port (2020)
sudo sed -i 's/#Port 22/Port 2020/' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# 1.3 verify listening
sudo ss -tulnp | grep 2020

## 2. Target – configure Fail2Ban
Create a minimal jail:
sudo tee /etc/fail2ban/jail.local <<EOF
[DEFAULT]
bantime  = 600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port    = 2020
logpath = %(sshd_log)s
EOF

sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd   # should show “0 banned”


## 3. Attacker – build word-lists

cd ~
echo -e "root\nubuntu\nadmin" > users.txt
echo -e "123456\npassword\nqwerty\nletmein\nparrot" > passes.txt

## 4. Attacker – brute-force simulation

hydra -L users.txt -P passes.txt -s 2020 192.168.1.44 ssh -t 4 -V


## 5. Target – watch detection & ban

# live log
sudo tail -f /var/log/fail2ban.log

# another terminal – check active bans
sudo fail2ban-client status sshd
sudo iptables -L f2b-sshd -n


Expected output:

NOTICE  [sshd] Ban 192.168.1.237

Within **3 failed attempts** the attacker IP is **DROP-ed for 10 min**.

## 6. Verify ban is real
From attacker:

ssh -p 2020 192.168.1.44
# → ssh: connect to host 192.168.1.44 port 2020: Connection refused


## 7. Clean-up / repeat
Un-ban yourself:

sudo fail2ban-client unban 192.168.1.237


## 8. Repo contents
- `README.md` – this file  
- `Vagrantfile` – one-command Ubuntu lab (optional)  
- `LICENSE` – MIT

## 9. Legal
Only attack VMs you own. Lab performed inside isolated VirtualBox host-only network.
```




```

#### `LICENSE`
```
MIT License – free for educational use.
---

Now anyone can **reproduce your entire lab** by cloning the repo and following the numbered steps.
