
***************************************************************************
HOW TO MAKE UBUNTU SERVER 22.04.x LTS use Static IP instead of DCHP Address
***************************************************************************


STEP 1: Login to the ubuntu server via ssh using existing DHCP IP 

 ssh username@server-ip-addr 

 Password: *********


STEP 2:  Determine the version of the distro 

cat lsb_release -a 

Output: 

Distributor ID: Ubuntu 

Description:    Ubuntu 22.04.5 LTS

Release:        22.04

Codename:       jammy

STEP 3:  Identify the network interface, IP Address, Subnet Mask and gatway of the server

command: ip addr show

cp1@workernode-01:~$ ip add show

_1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever_
       
_2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:50:69:16 brd ff:ff:ff:ff:ff:ff
    altname enp2s0
    inet 192.168.214.138/24 metric 100 brd 192.168.214.255 scope global *dynamic* ens160
       valid_lft 1536sec preferred_lft 1536sec
    inet6 fe80::20c:29ff:fe50:6916/64 scope link 
       valid_lft forever preferred_lft forever_

       

Note: The network interface is ens160, IP Address is 192.168.214.138/24, 
and the "dynamic" -keyword indicates that the IP address is dynamically assigned by a DHCP server.

STEP 3: Identify the network interface configuration file located in /etc/netplan directory

command: ls /etc/netplan

if the configuration file "01-network-manage-all.yaml" is missing create using the following command:

sudo touch /etc/netplan/01-network-manage-all.yaml

change the file permissionn to 644 using: chmod 644 /etc/netplan/01-network-manage-all.yaml

STEP 4: Edit the network interface configuration file using the following command:
# configs below are in ymal indentation

network:
  version: 2
  renderer: networkd
  ethernets:
    ens160:
      addresses:
        - 192.168.214.134/24
      nameservers:
        addresses: [192.168.214.1,1.1.1.1]
      routes:
        - to: default
          via: 192.168.214.2


SETP 5: Create a cloud.cfg.d/99-disable-network-config.cfg and write to it (network: {config: disabled})
To disable cloud-init's

** network configuration capabilities, write a file **

** /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:**

network: {config: disabled}


STEP 6: Now apply the new configuration:

sudo netplan apply

STEP 7: Backup the network config in /etc/netplan for cloud-init configuration before deleting the file

node-01: /etc/netplan$ ls -ll

total 12

-rw------- 1 root root 241 Mar 23 15:33 01-network-manage-all.yaml
-rw------- 1 root root 391 Mar 23  2025 50-cloud-init.yaml
-rw------- 1 root root 391 Mar 23 15:32 50-cloud-init.yaml.copy

$ _sudo rm -rf 50-cloud-init.yaml_

STEP 8: Reboot the server to apply the new configuration

$ sudo reboot

STEP 9 : Login to the server using the new static IP address

$ ip address show 
