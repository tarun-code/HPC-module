# What is PXE?

```
Preboot execution environment(PXE), it is a method to boot other systems over the network, without any disk

packages required to configure the PXE bootserver

syslinux: this is bootloader which is used to boot the computenode by master    
    # we copy three file from syslinux package
    # 1. initrd.img 
    # 2. vmlinuz
    # copied from syslinux
    # 3. menu.c32
    # 4. pxelinux.0

dhcp: this package will be used to give ip to node which will be booting on the PXE boot network.

tftp-server: this package will be used to setup the TFTP server, which will be used tranfer the kernel to the node.

xinetd: xinetd (Extended Internet Service Daemon) is an open-source super-server daemon which runs on many Unix-like systems, and manages Internet-based connectivity.

httpd: this package will be used to host the pxeboot config

```

---
## How to configure PXE Boot server:

```
Pre-requisits for PXE_boot:

-> create new Vm's with two network adapters
    1. NAT : For Public network access
    2. Host-Only : For private network access
```


### Step 1 : setup the OS
---
```bash
# Disable the selinux  
setenforce 0;  
sed -i 's/^SELINUX=.*$/SELINUX=disabled/g' /etc/selinux/config;

# Disable the firewall  
systemctl stop firewalld;  
systemctl disable firewalld;
```
---
### Step 2 : Download the required packages
---
```bash
# Download the packages 
yum install epel-release -y; 
yum install tftp-server -y;  
yum install httpd -y;  
yum install xinetd -y;  
yum install dhcp -y;  
```
---
### Step 3 : setup the dhcp server
---
```bash
# create dhcpd.conf file   
cat << EOF > /etc/dhcp/dhcpd.conf 
subnet 10.10.10.0\24 netmask 255.255.255.0 {
    option domain-name-servers master;
    # option domain-name-servers 10.10.10.155;
    option routers 10.10.10.155
    default-lease-time 3600;
    max-lease-time 7200;
    range 10.10.10.210 10.10.10.230;
}
EOF 
# to check if dhcp server is configured properly or not
dhcpd -t
# start the services 
systemctl start dhcpd tftp.service xinetd httpd;
systemctl enable dhcpd tftp.service xinetd httpd;
```
### Step 4: setup the PXE boot directories
---
```bash
mkdir -p /var/pxe/centos7 
mkdir -p /var/lib/tftpboot/centos7
# list all the blocks of storages 
lsblk
# duplicate disk command  
dd if=/dev/sr0 of=/centos7.iso
# mount the disk  
mount -t iso9660 -o loop /centos7.iso /var/pxe/centos7
# copy the kernel from the installed image to destination folder  
cp /var/pxe/centos7/images/pxeboot/vmlinuz /var/lib/tftpboot/centos7/
cp /var/pxe/centos7/images/pxeboot/initrd.img /var/lib/tftpboot/centos7/
cp /usr/share/syslinux/menu.c32 /var/lib/tftpboot/
cp /usr/share/syslinux/pxelinux.0  /var/lib/tftpboot/
# create PXE boot config file
cat << EOF > /var/lib/tftpboot/pxelinux.cfg/default
# creating PXE defination
timeout 100
default menu.c32
menu title ######## PXE BOOT MENU #########
label 1
        menu label ^1) install Centos 7
        kernel centos7/vmlinuz
        append initrd=centos7/initrd.img method=http://10.0.0.155/centos7 devfs=nomount
label 2
        menu label ^2) Boot from local drive
        localboot 
EOF
```
### Step 5 : host the file for pxeboot
---
```bash
# create dhcpd.conf file   
cat << EOF > /etc/httpd/conf.d/pxeboot.conf \
# create new
        Alias /centos7 /var/pxe/centos7
        <Directory /var/pxe/centos7>
            Options Indexes FollowSymLinks 
            # Ip address that will be allowed to access
            Require ip 127.0.0.1 [network_ip]
        </Directory>
EOF
```