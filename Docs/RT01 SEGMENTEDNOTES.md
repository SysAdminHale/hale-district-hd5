\############################################################

\# HALEDISTRICT HD4 / HD5 STUDY SHEET

\# RT01 SEGMENTED NETWORK + FIREWALL BASELINE

\#

\# Purpose:

\# - Build a multi-subnet routed network on RT01

\# - Enable internet access through WAN

\# - Enforce basic district-style firewall policy

\# - Persist rules across reboot

\#

\# Assumptions:

\# - eth0 = WAN / upstream / Default Switch

\# - eth1 = Servers/LAN

\# - eth3 = Teachers

\# - eth4 = Admin

\# - Students interface must be verified before final use

\#

\# IMPORTANT:

\# Linux interface names are determined by MAC/interface discovery,

\# not by Hyper-V switch names. Always verify with:

\#   ip addr

\# and match MAC addresses in Hyper-V before assigning final IPs.

\############################################################





\############################

\# 1. VERIFY INTERFACES

\############################



ip addr

ip route



\# Confirm final mapping before proceeding.

\# Expected working HD4 mapping:

\#   eth0 = WAN

\#   eth1 = 10.0.0.1/24   (Servers/LAN)

\#   eth3 = 10.0.20.1/24  (Teachers)

\#   eth4 = 10.0.10.1/24  (Admin)

\#   Students interface = verify before assigning 10.0.30.1/24





\############################

\# 2. ASSIGN GATEWAY IPs

\############################



\# Server/LAN subnet

sudo ip addr add 10.0.0.1/24 dev eth1

sudo ip link set eth1 up



\# Teachers subnet

sudo ip addr add 10.0.20.1/24 dev eth3

sudo ip link set eth3 up



\# Admin subnet

sudo ip addr add 10.0.10.1/24 dev eth4

sudo ip link set eth4 up



\# Students subnet

\# REPLACE <student-iface> after verifying the correct interface

\# Example only:

\# sudo ip addr add 10.0.30.1/24 dev <student-iface>

\# sudo ip link set <student-iface> up





\############################

\# 3. ENABLE ROUTING

\############################



\# Enable IPv4 forwarding immediately

sudo sysctl -w net.ipv4.ip\_forward=1



\# Persist IPv4 forwarding

sudo sed -i 's/^#\\?net.ipv4.ip\_forward=.\*/net.ipv4.ip\_forward=1/' /etc/sysctl.conf

sudo sysctl -p





\############################

\# 4. ENABLE NAT TO INTERNET

\############################



\# Masquerade all internal traffic out the WAN interface

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE





\############################

\# 5. SET FIREWALL DEFAULTS

\############################



sudo iptables -P INPUT ACCEPT

sudo iptables -P FORWARD DROP

sudo iptables -P OUTPUT ACCEPT





\############################

\# 6. INPUT RULES

\############################



\# Allow router management/traffic arriving on server/LAN interface

sudo iptables -A INPUT -i eth1 -j ACCEPT





\############################

\# 7. FORWARD RULES

\############################



\# Allow Server/LAN subnet out to internet

sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT



\# Allow return traffic from internet back to Server/LAN

sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT



\# Allow established / related return traffic generally

sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT



\# Admin -> anywhere

sudo iptables -A FORWARD -s 10.0.10.0/24 -j ACCEPT



\# Teachers -> Servers

sudo iptables -A FORWARD -s 10.0.20.0/24 -d 10.0.0.0/24 -j ACCEPT



\# Teachers -> Internet

sudo iptables -A FORWARD -s 10.0.20.0/24 -o eth0 -j ACCEPT



\# Students -> Internet

sudo iptables -A FORWARD -s 10.0.30.0/24 -o eth0 -j ACCEPT



\# Students blocked from Servers

sudo iptables -A FORWARD -s 10.0.30.0/24 -d 10.0.0.0/24 -j DROP





\############################

\# 8. VERIFY ACTIVE RULES

\############################



sudo iptables -S

sudo iptables -L -v

sudo iptables -t nat -S





\############################

\# 9. SAVE RULES PERSISTENTLY

\############################



\# Install persistence package

sudo apt update

sudo apt install iptables-persistent -y



\# Save current IPv4 rules

sudo iptables-save | sudo tee /etc/iptables/rules.v4



\# Verify saved file

sudo cat /etc/iptables/rules.v4



\# Reload persistent rules

sudo netfilter-persistent reload



\# Confirm active rules still loaded

sudo iptables -S





\############################

\# 10. BASIC VALIDATION TESTS

\############################



\# On RT01

ping -c 4 8.8.8.8

ping -c 4 10.0.0.10



\# On Admin workstation

\# ping 10.0.0.10

\# ping 8.8.8.8



\# On Teacher workstation

\# ping 10.0.0.10

\# ping 8.8.8.8



\# On Student workstation

\# ping 8.8.8.8

\# ping 10.0.0.10   <-- should fail if policy is working





\############################

\# 11. QUICK DIAGNOSTIC COMMANDS

\############################



\# Show interface/IP mapping

ip addr



\# Show route selected for a destination

ip route get 10.0.0.10



\# Show ARP/neighbors

ip neigh

arp -n



\# Show current firewall rules

sudo iptables -S



\# Show NAT rules

sudo iptables -t nat -S





\############################

\# 12. POLICY SUMMARY

\############################



\# Admin    -> full access

\# Teachers -> servers + internet

\# Students -> internet only

\# Students X servers

\# Default FORWARD policy = DROP

\############################################################

