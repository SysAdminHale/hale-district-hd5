\## run healthcheck which lives on FS01 but run it from ADM01

powershell -ExecutionPolicy Bypass -File "\\HD4-FS01\\Scripts$\\HealthChecks\\HD4-HealthCheck-Core.ps1"



\## remove things

Remove-Item -Recurse -Force "C:\\Hale-district-HD4\\Scripts.claude\\worktrees\\great-kilby"
git -C "C:\\Hale-district-HD4" worktree prune

Remove-Item -Recurse -Force "C:\\Hale-district-HD4\\Scripts.claude"
git -C "C:\\Hale-district-HD4" worktree prune

git -C "C:\\Hale-district-HD4" worktree prune

## 

\## display firwall group and set it to false

Enable-NetFirewallRule -DisplayGroup "File and Printer Sharing"

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False



\## which interface Linux will use

ip route get 10.0.0.10



\##This created RT01 segmentation in HD4

\# WAN

\# eth0 receives DHCP from Hyper-V Default Switch / upstream NAT

ip addr show eth0



\# Server/LAN segment

sudo ip addr add 10.0.0.1/24 dev eth1

sudo ip link set eth1 up



\# Teachers segment

sudo ip addr add 10.0.20.1/24 dev eth3

sudo ip link set eth3 up



\# Admin segment

sudo ip addr add 10.0.10.1/24 dev eth4

sudo ip link set eth4 up



eth0 = WAN / Default Switch

eth1 = HD4-LAN        = 10.0.0.1/24

eth3 = HD4-Teachers   = 10.0.20.1/24

eth4 = HD4-AdminServers = 10.0.10.1/24



sudo iptables -P INPUT ACCEPT

sudo iptables -P FORWARD DROP

sudo iptables -P OUTPUT ACCEPT



sudo iptables -A INPUT -i eth1 -j ACCEPT



\# Legacy server/LAN subnet to internet

sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT



\# Return traffic from internet back to server/LAN

sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT



\# General established/related return traffic

sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT



\# Admin subnet: full access

sudo iptables -A FORWARD -s 10.0.10.0/24 -j ACCEPT



\# Teachers -> server subnet

sudo iptables -A FORWARD -s 10.0.20.0/24 -d 10.0.0.0/24 -j ACCEPT



\# Teachers -> internet

sudo iptables -A FORWARD -s 10.0.20.0/24 -o eth0 -j ACCEPT



\# Students -> internet

sudo iptables -A FORWARD -s 10.0.30.0/24 -o eth0 -j ACCEPT



\# Students blocked from servers

sudo iptables -A FORWARD -s 10.0.30.0/24 -d 10.0.0.0/24 -j DROP





\### To make persistent

sudo apt update

sudo apt install iptables-persistent -y



\### To save and verify IP tables

sudo iptables-save | sudo tee /etc/iptables/rules.v4

sudo cat /etc/iptables/rules.v4



\### Reload persistent rules

sudo netfilter-persistent reload



\### Verify Active Rules

sudo iptables -S



\###HD4 RT01 Segmentation Summary

Subnets

\- 10.0.0.0/24   Servers/LAN

\- 10.0.10.0/24  Admin

\- 10.0.20.0/24  Teachers

\- 10.0.30.0/24  Students



RT01 gateway IPs

\- 10.0.0.1

\- 10.0.10.1

\- 10.0.20.1

\- 10.0.30.1 (intended/final student gateway; verify NIC mapping before freezing doc)



Firewall policy

\- Admin -> full access

\- Teachers -> servers + internet

\- Students -> internet only

\- Students -> servers blocked

\- Default FORWARD policy = DROP



Persistence

\- iptables-persistent installed

\- rules saved to /etc/iptables/rules.v4

\- netfilter-persistent reload successful









