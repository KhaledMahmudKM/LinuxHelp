# Useful Linux Commands and Procedures


Khaled Mahmud

Khaled.mahmud@sheridancollege.ca

# Linux Servers: DHCP, DNS, NAT, Web

Useful Linux Commands


_Last Updated: May 25, 2020_

# Contents

[0. Conventions 2](#_Toc71706800)

[1. Some Basic Linux Commads/Tasks 3](#_Toc71706801)

[Change hostname 3](#_Toc71706802)

[Change Machine ID 4](#_Toc71706803)

[IP Setting: Static and Dynamic 4](#_Toc71706804)

[SSH Server Installation 5](#_Toc71706805)

[Checking Status of Services 6](#_Toc71706806)

[2. DHCP Server 8](#_Toc71706807)

[Install DHCP Server Package 8](#_Toc71706808)

[DHCP Server Configuration 9](#_Toc71706809)

[Running DHCP6 Service 12](#_Toc71706810)

[3. DNS Server Installation and Configuration 15](#_Toc71706811)

[Verify Current Name Server(s) 15](#_Toc71706812)

[Install DNS Server: Bind9 16](#_Toc71706813)

[Verify Bind Configuration Directory 18](#_Toc71706814)

[DNS Configuration (Primary Server) 19](#_Toc71706815)

[Verify DNS Operation 26](#_Toc71706816)

[DNS Configuration (Secondary Server) 28](#_Toc71706817)

[4. Create NAT 33](#_Toc71706818)

<hr>
<hr>

# 1. Some Basic Linux Commads/Tasks

## Change Hostname

1. Change the hostname using hostnamectl command.
2. Then update the /etc/hosts file.

```
vmadmin@LS02:~$sudo hostnamectl set-hostname ns1

vmadmin@LS02:~$sudo nano /etc/hosts
```

```
vmadmin@ns1:~$ hostnamectl

   Static hostname: ns1
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 62447195dfa24f14b70431654bceefbd
           Boot ID: 48f9e70f60dd4a7da20ca1fbf7332320
    Virtualization: oracle
  Operating System: Ubuntu 20.04 LTS
            Kernel: Linux 5.4.0-26-generic
      Architecture: x86-64
```

```
vmadmin@ns1:~$ cat /etc/hosts

127.0.0.1 localhost
127.0.1.1 ns1
#__ The following lines are desirable for IPv6 capable hosts_
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

```
vmadmin@ns1:~$ cat /etc/hostname

ns1
```


<hr>

## Change Machine ID

After cloning from an existing you may find that the machines ID of the new image is same as the old one. Change the machine ID. It&#39;s stored in /etc/machine-id file.

```
vmadmin@LS02:~$sudo rm /etc/machine-id

vmadmin@LS02:~$sudo systemd-machine-id-setup
```

If this does not create unique ID, just manually change the value (e.g. using nano).
<hr>

## IP Setting: Static and Dynamic

1. Create/Modify the .yaml file in /etc/netplan directory to configure IP addresses in the interfaces. The exact name of the yaml file is fixed. There may be more then one file in this directory. Example configuration is given below. Here the renderer is **networkd**.

```
user1@muserv1:~$ cat /etc/netplan/01-netcfg.yaml

# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.144.12/24]
      gateway4: 192.168.144.1
      nameservers:
        addresses: [192.168.144.1, 8.8.8.8]
    enp0s8:
      dhcp4: yes
```

2. Use **sudo netplan apply** command to apply the changes. Verify the change using **ip address** command.

```
vmadmin@LS02:~$sudo netplan apply
vmadmin@LS02:~$ip address
```

3. To start/restart/stop to check status of networkd service, use the following.

```
vmadmin@LS02:~$sudo systemctl status system-networkd
```
<hr>

## SSH Server Installation

To get SSH access to a linux machine, you can install openssh-server. You can see the status of the server with systemctl command.

vmadmin@LS02:~$sudo apt-get install opensh-server

vmadmin@LS02:~$sudo systemctl status ssh

![](RackMultipart20210525-4-tcjlx1_html_6d1df207ab2198d5.png)

### Checking Status of Services

1. You can use **service --status-all** command or **systemctl list-unit-files** command to view the status all the installed services.

vmadmin@LS02:~$sudo service --status-all

vmadmin@LS02:~$sudo systemctl list-unit-files

![](RackMultipart20210525-4-tcjlx1_html_38971b5b1eda0b3.png)

![](RackMultipart20210525-4-tcjlx1_html_35317513c88252c2.png)

1. Use **systemctl status**  **_servicename_** command or **service**  **_servicename_**  **status** command to see the status of a service. The following example shows the status of ssh service.

vmadmin@LS02:~$ sudo service ssh status

vmadmin@LS02:~$sudo systemctl status ssh

vmadmin@LS02:~$sudo systemctl list-unit-files | grep -i ssh.service

![](RackMultipart20210525-4-tcjlx1_html_6d1df207ab2198d5.png)

# 2. DHCP Server

### Install DHCP Server Package

The following steps show how to install and configure isc-dhcp-server.

1. Make sure to update and upgrade the linux. The install the dhcp server package.

vmadmin@LS02:~$sudo apt update

vmadmin@LS02:~$sudo apt apt upgrade

vmadmin@LS02:~$ sudo apt-get install isc-dhcp-server

1. Verify that all the files are installed in /etc/dhcp/ directory.

![](RackMultipart20210525-4-tcjlx1_html_5217f2f6a51450a5.png)

### DHCP Server Configuration

1. Modify/update /etc/default/isc-dhcp-server file to indicate which interface DHCP service will be provided. Find the interface name using ip address command. Interface name is **ens160** in the example below.

![](RackMultipart20210525-4-tcjlx1_html_68744d4dcd604931.png)

1. Enter all the server configuration information in the /etc/dhcp/dhcpd.conf file. This file, by default, comes with a lot of configuration examples. You can use the following pattern to configure a range of IP addresses for dynamic allocation, along with domain-name, DNS server and gateway router settings. Also, you can reserve an IP address for static allocation (with MAC address binding).

![](RackMultipart20210525-4-tcjlx1_html_568054b0b2538292.png)

![](RackMultipart20210525-4-tcjlx1_html_b7050f8b41b4ccb1.png)

1. Now, restart the dhcp service. Then you can verity the status of the service.

![](RackMultipart20210525-4-tcjlx1_html_cb677c3457133748.png)

1. Once the server starts distributing (leasing) IP addresses (and other information) to the clients, you can verify the leasing status using dhcp-lease-list command. You can also view the /var/lib/dhcp/dhcpd.leases file.

![](RackMultipart20210525-4-tcjlx1_html_e2b4da1b56e4d0e5.png)

![](RackMultipart20210525-4-tcjlx1_html_394bfe13f429fbf3.png)

### Running DHCP6 Service

The isc-dhcp-server also provide DHCP6 service. Follow the steps below to make server provide IPv6 addresses to the clients.

1. Make sure /etc/default/isc-dhcp-server file contains **INTERAFCESv6** parameter, set to the interface where the server is supposed to allocated IPv6 addresses. In the example above, it set to **ens160**.
2. Modify the yaml file in /etc/netplan/ directory to set IPv6 address to the server, in proper interface.

user1@muserv1:~$ cat /etc/netplan/01-netcfg.yaml

# Generated by VMWare customization engine.

network:

version: 2

renderer: networkd

ethernets:

ens160:

dhcp4: no

dhcp6: no

addresses:

- 192.168.144.12/24

- 2001:db8:90::12/64

gateway4: 192.168.144.1

gateway6: 2001:db8:90::1

nameservers:

addresses:

- 142.55.100.25

- 142.55.44.25

- 2001:db8:90::11

user1@muserv1:~$

1. Use **sudo netplan apply** command to apply the changes. Verify the change using **ip address** command.

vmadmin@LS02:~$sudo netplan apply

vmadmin@LS02:~$ip address

1. Modify the provided /etc/dhcp/dhcpd6.conf file to include desired parameters. A sample is given below.

![](RackMultipart20210525-4-tcjlx1_html_cbfbd66b96c46e03.png)

1. Restart the DHCP6 service. Verify the status of the service.

![](RackMultipart20210525-4-tcjlx1_html_ebd0354e73f7f063.png)

1. When a client gets a lease of IPv6 address from the server, you can verify the lease list.

![](RackMultipart20210525-4-tcjlx1_html_f1be769deff74a7b.png)

# 3. DNS Server Installation and Configuration

Mostly used DNS server in linux is bind9. The following instructions show you how to install and configure it.

### Verify Current Name Server(s)

There are many places in Linux, you can find DNS related information.

1. The file resolv.conf contains IP address of the internal (stub) resolver of the host. You don&#39;t need to this file. A sample output is given here.

vmadmin@ns1:~$ cat /etc/resolv.conf

_# This file is managed by man:systemd-resolved(8). Do not edit._

_#_

_# This is a dynamic resolv.conf file for connecting local clients to the_

_# internal DNS stub resolver of systemd-resolved. This file lists all_

_# configured search domains._

_#_

_# Run &quot;resolvectl status&quot; to see details about the uplink DNS servers_

_# currently in use._

_#_

_# Third party programs must not access this file directly, but only through the_

_# symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a different way,_

_# replace this symlink by a static file or a different symlink._

_#_

_# See man:systemd-resolved.service(8) for details about the supported modes of_

_# operation for /etc/resolv.conf._

nameserver 127.0.0.53

options edns0

vmadmin@ns1:~$

1. Your actual configuration can be found in the yaml file of netplan folder.

vmadmin@ns1:~$ cat /etc/netplan/00-installer-config.yaml

_# This is the network config written by &#39;subiquity&#39;_

network:

  version: 2

  renderer: networkd

  ethernets:

    enp0s3:

      dhcp4: false

      addresses:

        - 192.168.144.11/24

      gateway4: 192.168.144.1

      nameservers:

![](RackMultipart20210525-4-tcjlx1_html_3ac43594d0cda7c9.gif)        addresses:

          - 192.168.144.1

    enp0s8:

      dhcp4: true

vmadmin@ns1:~$

1. You can also check the runtime resolv.conf file to see what actual outside server is being used by the OS.

vmadmin@ns1:~$ cat /run/systemd/resolve/resolv.conf

_# This file is managed by man:systemd-resolved(8). Do not edit._

_#_

_# This is a dynamic resolv.conf file for connecting local clients directly to_

_# all known uplink DNS servers. This file lists all configured search domains._

_#_

_# Third party programs must not access this file directly, but only through the_

_# symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a different way,_

_# replace this symlink by a static file or a different symlink._

_#_

_# See man:systemd-resolved.service(8) for details about the supported modes of_

_# operation for /etc/resolv.conf._

![](RackMultipart20210525-4-tcjlx1_html_3ac43594d0cda7c9.gif)

nameserver 192.168.144.1

vmadmin@ns1:~$

### Install DNS Server: Bind9

1. Make sure to upgrade your system before you start.
2. Then install bind9 and bind9utils. This will install the servers run (start it).
3. Verify if the server is properly and running. Notice that the daemon is called named.
4. Check the listening ports, using ss command.

vmadmin@ns1:~$sudo apt update

vmadmin@ns1:~$sudo apt upgrade

vmadmin@ns1:~$sudo apt install bind9 bind9utils

vmadmin@ns1:~$

vmadmin@ns1:~$sudo systemctl status named

● named.service - BIND Domain Name Server

     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)

     Active: active (running) since Tue 2020-06-09 02:24:27 UTC; 12min ago

       Docs: man:named(8)

   Main PID: 30139 (named)

      Tasks: 8 (limit: 2282)

     Memory: 19.4M

     CGroup: /system.slice/named.service

             └─30139 /usr/sbin/named -f -u bind

\&lt;…output omitted…\&gt;

Jun 09 02:24:27 ns1 named[30139]: managed-keys-zone: Initializing automatic trust anchor management fo\&gt;

Jun 09 02:24:27 ns1 named[30139]: resolver priming query complete

vmadmin@ns1:~$

vmadmin@ns1:~$ ss -ltun

Netid  State   Recv-Q  Send-Q  Local Address:Port     Peer Address:Port  Process

udp    UNCONN  0       0    192.168.56.109:53            0.0.0.0:\*

udp    UNCONN  0       0    192.168.56.109:53            0.0.0.0:\*

udp    UNCONN  0       0       192.168.144.11:53            0.0.0.0:\*

udp    UNCONN  0       0       192.168.144.11:53            0.0.0.0:\*

udp    UNCONN  0       0       127.0.0.1:53             0.0.0.0:\*

udp    UNCONN  0       0       127.0.0.1:53             0.0.0.0:\*

udp    UNCONN  0       0       127.0.0.53%lo:53             0.0.0.0:\*

udp    UNCONN  0       0       192.168.56.109%enp0s8:68     0.0.0.0:\*

udp    UNCONN  0       0       [::1]:53                [::]:\*

udp    UNCONN  0       0       [::1]:53                [::]:\*

udp    UNCONN  0       0       [fe80::a00:27ff:feb1:9849]%enp0s3:53     [::]:\*

udp    UNCONN  0       0       [fe80::a00:27ff:feb1:9849]%enp0s3:53    [::]:\*

udp    UNCONN  0       0       [fe80::a00:27ff:fe37:cc3e]%enp0s8:53    [::]:\*

udp    UNCONN  0       0       [fe80::a00:27ff:fe37:cc3e]%enp0s8:53    [::]:\*

tcp    LISTEN  0       4096    127.0.0.1:953           0.0.0.0:\*

tcp    LISTEN  0       10      192.168.56.109:53       0.0.0.0:\*

tcp    LISTEN  0       10      192.168.144.11:53       0.0.0.0:\*

tcp    LISTEN  0       10      127.0.0.1:53            0.0.0.0:\*

tcp    LISTEN  0       4096    127.0.0.53%lo:53        0.0.0.0:\*

tcp    LISTEN  0       128     0.0.0.0:22             0.0.0.0:\*

tcp    LISTEN  0       4096    [::1]:t              [::]:\*

tcp    LISTEN  0       10      [fe80::a00:27ff:fe37:cc3e]%enp0s8:53    [::]:\*

tcp    LISTEN  0       10      [fe80::a00:27ff:feb1:9849]%enp0s3:53    [::]:\*

tcp    LISTEN  0       10      [::1]:53               [::]:\*

tcp    LISTEN  0       128     [::]:22                [::]:\*

vmadmin@ns1:~$

### Verify Bind Configuration Directory

List the files in the configuration directory of bind9, which is /etc/bind. This directory contains configuration files as well as template zone database (resource record) files. Some template zone files are provided by default. We will modify these files, as required, to customize out DNS server.

vmadmin@ns1:~$ cd /etc/bind

vmadmin@ns1:/etc/bind$ ls -l

total 48

-rw-r--r-- 1 root root 1991 May 15 12:03 bind.keys

-rw-r--r-- 1 root root  237 Apr 15 17:59 db.0

-rw-r--r-- 1 root root  271 Apr 15 17:59 db.127

-rw-r--r-- 1 root root  237 Apr 15 17:59 db.255

-rw-r--r-- 1 root root  353 Apr 15 17:59 db.empty

-rw-r--r-- 1 root root  270 Apr 15 17:59 db.local

-rw-r--r-- 1 root bind  463 Apr 15 17:59 named.conf

-rw-r--r-- 1 root bind  498 Apr 15 17:59 named.conf.default-zones

-rw-r--r-- 1 root bind  165 Apr 15 17:59 named.conf.local

-rw-r--r-- 1 root bind  846 Apr 15 17:59 named.conf.options

-rw-r----- 1 bind bind  100 Jun  9 02:24 rndc.key

-rw-r--r-- 1 root root 1317 Apr 15 17:59 zones.rfc1918

vmadmin@ns1:/etc/bind$

vmadmin@ns1:/etc/bind$ cat db.local

;

; BIND data file for local loopback interface

;

$TTL    604800

@       IN      SOA     localhost. root.localhost. (

                              2         ; Serial

                         604800         ; Refresh

                          86400         ; Retry

                        2419200         ; Expire

                         604800 )       ; Negative Cache TTL

;

@       IN      NS      localhost.

@       IN      A       127.0.0.1

@       IN      AAAA    ::1

vmadmin@ns1:/etc/bind$

### DNS Configuration (Primary Server)

For DNS configurations, we will modify only (i) named.conf.options, and (ii) named.conf.local. For database files, we will create, using the templates, (i) forward zone file, and (ii) reverse zone file. Follow the steps below.

1. Create a directory to store our zone database files.
2. Backup the provided template file before we modify the file.

vmadmin@ns1:/etc/bind$ sudo mkdir zones

vmadmin@ns1:/etc/bind$ sudo cp named.conf.options named.conf.options.org

vmadmin@ns1:/etc/bind$ sudo cp named.conf.local named.conf.local.org

1. Customize named.conf.options file. The content may look like the example given below.

vmadmin@ns1:/etc/bind$cat named.conf.options

options {

        directory &quot;/var/cache/bind&quot;;

        // If there is a firewall between you and nameservers you want

        // to talk to, you may need to fix the firewall to allow multiple

        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable

        // nameservers, you probably want to use them as forwarders.

        // Uncomment the following block, and insert the addresses replacing

        // the all-0&#39;s placeholder.

        //Edit the options file for add the following

        //Add forwarders, use ISP&#39;s or some public DNS server

![](RackMultipart20210525-4-tcjlx1_html_215c8632289810f9.gif)        forwarders {

                192.168.144.1; 8.8.8.8;

        };

        //========================================================================

        // If BIND logs error messages about the root key being expired,

        // you will need to update your keys.  See https://www.isc.org/bind-keys

        //========================================================================

        //Allow query only from own machine or local subnet hosts; or any intended subnet

        //You can specify both IPv4 and IPv6 hosts/subnets

![](RackMultipart20210525-4-tcjlx1_html_20f28ca41878bfd3.gif)        allow-query { localhost; 192.168.144.0/24; };

        //Allow recursive DNS lookup service for the specified hosts.

![](RackMultipart20210525-4-tcjlx1_html_a7dc75987534b3.gif)//Generally, for authoritative name servers, we would not allow recursion

        allow-recursion { localhost; 192.168.144.0/24; };

        //

        dnssec-validation auto;

        //Allow IPv6 (or use &#39;none&#39; to allow only IPv4)

![](RackMultipart20210525-4-tcjlx1_html_d0a15377fd9aaf36.gif)        listen-on-v6 { any; };

};

1. Restart the server and check the status.

vmadmin@ns1:/etc/bind$ sudo systemctl restart named

vmadmin@ns1:/etc/bind$ sudo systemctl status named

1. Update named.conf.local file. It should look something like shown below. Here a forward zone and a reverse is created. Note the specified database file names and locations. IP address of secondary (planned) DNS server is provided to allow zone transfer fixed server.

vmadmin@ns1:/etc/bind$ cat named.conf.local

/ Do any local configuration here

//

// Consider adding the 1918 zones here, if they are not used in your

// organization

//include &quot;/etc/bind/zones.rfc1918&quot;;

// Create forward and reverse zones

![](RackMultipart20210525-4-tcjlx1_html_3c4bca744062c64e.gif)

Zone name, for forward zone

// This is for forward zone

zone &quot;khaledmahmud.net&quot; {

![](RackMultipart20210525-4-tcjlx1_html_3c4bca744062c64e.gif)

It is a primary server

// It is a primary server

  type master;

![](RackMultipart20210525-4-tcjlx1_html_61abf2c082629753.gif)

Zone database file location

// Location of forward zone file for this domain

  file &quot;/etc/bind/zones/db.for.khaledmahmud.net&quot;;

![](RackMultipart20210525-4-tcjlx1_html_c6f4b9c556033b05.gif)

Do not allow dynamic update

// Do not allow dynamic update

  allow-update { none; };

// Allow zone transfer from our secondary server only

![](RackMultipart20210525-4-tcjlx1_html_990224ceaef1af5e.gif)

Allow zone transfer requests from specified server(s)

  allow-transfer { 192.168.144.12 ;};

also-notify {192.168.144.12; };

};

//This is for reverse zone

//For Zone name, use first three octets of the subnet, in reverse order, then append &#39;in-add.arpa&#39;

![](RackMultipart20210525-4-tcjlx1_html_830aa1986dfd48e5.gif)

Zone name, for reverse zone

zone &quot;144.168.192.in-addr.arpa&quot; {

  type master;

![](RackMultipart20210525-4-tcjlx1_html_4b89d2471c3ff636.gif)

Reverse zone database file location

  file &quot;/etc/bind/zones/db.rev.khaledmahmud.net&quot;;

  allow-transfer { 192.168.144.12 ;};

also-notify {192.168.144.12; };

  allow-update { none; };

};

1. Create zone database file for forward zone. The name and location must match the named.conf.local file.

vmadmin@ns1:/etc/bind$ cd zones

vmadmin@ns1:/etc/bind/zones$ sudo nano db.for.khaledmahmud.net

vmadmin@ns1:/etc/bind/zones$ cat db.for.khaledmahmud.net

;

; BIND data file for khaledmahmud.net.

;

$TTL    604800          ; default TTL, 1W

; SOA record specifies parameters for the server and the records

@       IN      SOA     khaledmahmud.net. dnsadmin.khaledmahmud.net. (

                         2020060402     ; Serial, use date of change and an increasing number

                         604800         ; Refresh, 1W

                          86400         ; Retry, 1D

                        2419200         ; Expire, 4W

                         604800 )       ; Negative Cache TTL, 1W

        IN      A       192.168.144.11

;

; Add name servers

@       IN      NS      ns1.khaledmahmud.net.

@       IN      NS      ns2.khaledmahmud.net.

ns1     IN      A       192.168.144.11

ns2     IN      A       192.168.144.12

; Add mail server

@       IN      MX  10  mail1.khaledmahmud.net. ;First priority

@       IN      MX  20  mail2.khaledmahmud.net. ;Next priority

mail1   IN      A       192.168.144.13

mail2   IN      A       192.168.144.14

; Add other A records

www     IN      A       192.168.144.13

; Add the gateway router, just for fun

R1      IN      A       192.168.144.1

; Add aliases

ftp     IN      CNAME       www.khaledmahmud.net.

1. Create zone database file for reverse zone. Again, The name and location must match the named.conf.local file.

vmadmin@ns1:/etc/bind/zones$ sudo nano db.rev.khaledmahmud.net

vmadmin@ns1:/etc/bind/zones$ cat db.rev.khaledmahmud.net

;

; BIND reverse data file for khaledmahmud.net 192.168.144 subnet

;

$TTL    604800          ; default TTL, 1W

@       IN      SOA     ns1.khaledmahmud.net. dnsadmin.khaledmahmud.net. (

                         2020060402     ; Serial

                         604800         ; Refresh, 1W

                          86400         ; Retry, 1D

                        2419200         ; Expire, 4W

                         604800 )       ; Negative Cache TTL, 1W

;

@       IN      NS      ns1.khaledmahmud.net.

; Add desired reverse references (pointers)

11      IN      PTR     ns1.khaledmahmud.net.

13      IN      PTR     mail1.khaledmahmud.net.

14      IN      PTR     mail2.khaledmahmud.net.

vmadmin@ns1:/etc/bind/zones$

1. Now, update the netplanyaml file with the new DNS server IP address (192.168.144.11). Apply netplan change. Add search item under nameservers item.

vmadmin@ns1:/etc/bind/zones$ cat /etc/netplan/00-installer-config.yaml

_# This is the network config written by &#39;subiquity&#39;_

network:

  version: 2

  renderer: networkd

  ethernets:

    enp0s3:

      dhcp4: false

      addresses:

        - 192.168.144.11/24

      gateway4: 192.168.144.1

      nameservers:

        addresses:

          - 192.168.144.11

        search: [khaledmahmud.net]

    enp0s8:

      dhcp4: true

vmadmin@ns1:/etc/bind/zones$

vmadmin@ns1:/etc/bind/zones$ sudo netplan apply

vmadmin@ns1:/etc/bind/zones$ sudo systemctl restart named

vmadmin@ns1:/etc/bind/zones$ sudo systemctl status named

1. Before you restart the server again, check of the configuration is OK. Also, check the zone files are defined properly. Use named-checkconf and named-checkzone utilities.

![](RackMultipart20210525-4-tcjlx1_html_127389d0908a943.gif)

Check your domain name against the forward zone file.

_#Check the configuration_

vmadmin@ns1:~$sudo named-checkconf

_#Check the zone syntax_

vmadmin@ns1:~$sudo named-checkzone khaledmahmud.net /etc/bind/zones/db.for.khaledmahmud.net

![](RackMultipart20210525-4-tcjlx1_html_626c8b131511887.gif)

Check your subnet against the reverse zone file.

vmadmin@ns1:~$sudo named-checkzone 144.168.192.in-addr.arpa

/etc/bind/zones/db.rev.khaledmahmud.net

1. Restart bind9 (named). Check the status of the server to make sure it&#39;s running properly.

### Verify DNS Operation

Let us verify if our DNS server is properly working.

1. Ping a domain name in the internet.
2. Use host utility to resolve a domain name using our server: 192.168.144.11.

See the sample output here.

vmadmin@ns1:~$ ping www.google.com

PING www.google.com (172.217.164.228) 56(84) bytes of data.

64 bytes from yyz12s05-in-f4.1e100.net (172.217.164.228): icmp\_seq=1 ttl=54 time=14.6 ms

64 bytes from yyz12s05-in-f4.1e100.net (172.217.164.228): icmp\_seq=2 ttl=54 time=14.9 ms

64 bytes from yyz12s05-in-f4.1e100.net (172.217.164.228): icmp\_seq=3 ttl=54 time=17.2 ms

^C

--- www.google.com ping statistics ---

3 packets transmitted, 3 received, 0% packet loss, time 2003ms

rtt min/avg/max/mdev = 14.637/15.556/17.177/1.149 ms

vmadmin@ns1:~$

vmadmin@ns1:~$ host google.com 192.168.144.11

Using domain server:

Name: 192.168.144.11

Address: 192.168.144.11#53

Aliases:

google.com has address 172.217.0.238

google.com has IPv6 address 2607:f8b0:400b:802::200e

google.com mail is handled by 30 alt2.aspmx.l.google.com.

google.com mail is handled by 10 aspmx.l.google.com.

google.com mail is handled by 40 alt3.aspmx.l.google.com.

google.com mail is handled by 50 alt4.aspmx.l.google.com.

google.com mail is handled by 20 alt1.aspmx.l.google.com.

vmadmin@ns1:~$

1. Use dig utility to see the resource records.

vmadmin@ns1:~$ dig google.com

; \&lt;\&lt;\&gt;\&gt; DiG 9.16.1-Ubuntu \&lt;\&lt;\&gt;\&gt; google.com

;; global options: +cmd

;; Got answer:

;; -\&gt;\&gt;HEADER\&lt;\&lt;- opcode: QUERY, status: NOERROR, id: 53637

;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 65494

;; QUESTION SECTION:

;google.com.                    IN      A

;; ANSWER SECTION:

google.com.             36      IN      A       172.217.0.238

;; Query time: 24 msec

;; SERVER: 127.0.0.53#53(127.0.0.53)

;; WHEN: Tue Jun 09 05:20:53 UTC 2020

;; MSG SIZE  rcvd: 55

vmadmin@ns1:~$

### DNS Configuration (Secondary Server)

Once than a Primary server has been configured, secondary servers can be added to the naming domain. But the primary server is the master for the zone, so a reference to the secondary server must be included in the primary server before anything else. Let us assume that the secondary server IPv4 address is 192.168.144.12.

1. In the primary server&#39;s configuration file (/etc/bind/named.conf.local), make sure to allow transfer request from the secondary server. Modify both the forward zone and reverse zone.

// Allow zone transfer from our secondary server only

  allow-transfer { 192.168.144.12 ;};

also-notify {192.168.144.12; };

We do not create or updates zone databases in the secondary server. A secondary name server uses zone transfer to get resource records from a primary server. Follow the following steps to install secondary DNS server in linux.

1. Make sure the server has proper (intended) hostname (e.g. ns2).

vmadmin@ns2:~$ hostnamectl

   Static hostname: ns2

         Icon name: computer-vm

           Chassis: vm

        Machine ID: f3b9d08af19e4b239828da46edde00ae

           Boot ID: e7b90fd9bedd465694580e3f716ee508

    Virtualization: oracle

  Operating System: Ubuntu 20.04 LTS

            Kernel: Linux 5.4.0-37-generic

      Architecture: x86-64

vmadmin@ns2:~$ cat /etc/hosts

127.0.0.1       localhost

127.0.1.1       ns2

_# The following lines are desirable for IPv6 capable hosts_

::1     localhost ip6-localhost ip6-loopback

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

vmadmin@ns2:~$

1. Install bind9 and bind9utils.

vmadmin@ns2:~$ sudo apt install bind9 bind9utils

vmadmin@ns2:~$ sudo systemctl status named

1. Edit /etc/bind/named.conf.local file. Add zones corresponding to the primary server.

vmadmin@ns2:~$ cat /etc/bind/named.conf.local

//

// Do any local configuration here

//

// Consider adding the 1918 zones here, if they are not used in your

// organization

//include &quot;/etc/bind/zones.rfc1918&quot;;

//Forward zone

zone &quot;khaledmahmud.net&quot; {

        type slave;

        file &quot;db.for.khaledmahmud.net&quot;;

        masters { 192.168.144.11; };

};

//Reverse zone

zone &quot;144.168.192.in-addr.apra&quot; {

        type slave;

        file &quot;db.rev.khaledmahmud.net&quot;;

        masters { 192.168.144.11; };

};

vmadmin@ns2:~$

1. Check the configuration using named-checkconf tool.

vmadmin@ns2:~$ sudo named-checkconf

vmadmin@ns2:~$

1. Restart the server and check the status to make sure it is active.

vmadmin@ns2:~$ sudo systemctl restart named

vmadmin@ns2:~$ sudo systemctl status named

There is no need to configure the databases because the secondary servers do not maintain such files. Instead, they receive all the naming information from the primary servers which are the only authorized to create new entries in the naming databases. The databases are stored in /var/cache/bind directory.

1. Verify the server&#39;s operation using nslookup, host and dig tools.

vmadmin@ns2:~$ nslookup ns1

Server:         127.0.0.53

Address:        127.0.0.53#53

Non-authoritative answer:

Name:   ns1.khaledmahmud.net

Address: 192.168.144.11

vmadmin@ns2:~$ nslookup 192.168.144.13

13.144.168.192.in-addr.arpa     name = mail1.khaledmahmud.net.

Authoritative answers can be found from:

vmadmin@ns2:~$

vmadmin@ns2:~$ dig www.gmail.com

; \&lt;\&lt;\&gt;\&gt; DiG 9.16.1-Ubuntu \&lt;\&lt;\&gt;\&gt; www.gmail.com

;; global options: +cmd

;; Got answer:

;; -\&gt;\&gt;HEADER\&lt;\&lt;- opcode: QUERY, status: NOERROR, id: 4455

;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 65494

;; QUESTION SECTION:

;www.gmail.com.                 IN      A

;; ANSWER SECTION:

www.gmail.com.          21599   IN      CNAME   mail.google.com.

mail.google.com.        7199    IN      CNAME   googlemail.l.google.com.

googlemail.l.google.com. 298    IN      A       172.217.1.165

;; Query time: 144 msec

;; SERVER: 127.0.0.53#53(127.0.0.53)

;; WHEN: Mon Jun 15 00:03:18 EDT 2020

;; MSG SIZE  rcvd: 111

vmadmin@ns2:~$

vmadmin@ns2:~$ dig google.com MX

; \&lt;\&lt;\&gt;\&gt; DiG 9.16.1-Ubuntu \&lt;\&lt;\&gt;\&gt; google.com MX

;; global options: +cmd

;; Got answer:

;; -\&gt;\&gt;HEADER\&lt;\&lt;- opcode: QUERY, status: NOERROR, id: 1776

;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 65494

;; QUESTION SECTION:

;google.com.                    IN      MX

;; ANSWER SECTION:

google.com.             238     IN      MX      50 alt4.aspmx.l.google.com.

google.com.             238     IN      MX      10 aspmx.l.google.com.

google.com.             238     IN      MX      30 alt2.aspmx.l.google.com.

google.com.             238     IN      MX      20 alt1.aspmx.l.google.com.

google.com.             238     IN      MX      40 alt3.aspmx.l.google.com.

;; Query time: 20 msec

;; SERVER: 127.0.0.53#53(127.0.0.53)

;; WHEN: Mon Jun 15 00:06:06 EDT 2020

;; MSG SIZE  rcvd: 147

vmadmin@ns2:~$

vmadmin@ns2:~$ dig khaledmahmud.net NS

; \&lt;\&lt;\&gt;\&gt; DiG 9.16.1-Ubuntu \&lt;\&lt;\&gt;\&gt; khaledmahmud.net NS

;; global options: +cmd

;; Got answer:

;; -\&gt;\&gt;HEADER\&lt;\&lt;- opcode: QUERY, status: NOERROR, id: 26240

;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 65494

;; QUESTION SECTION:

;khaledmahmud.net.              IN      NS

;; ANSWER SECTION:

khaledmahmud.net.       604800  IN      NS      ns2.khaledmahmud.net.

khaledmahmud.net.       604800  IN      NS      ns1.khaledmahmud.net.

;; Query time: 0 msec

;; SERVER: 127.0.0.53#53(127.0.0.53)

;; WHEN: Mon Jun 15 00:06:46 EDT 2020

;; MSG SIZE  rcvd: 81

vmadmin@ns2:~$

1. You can setup a client PC which configured with only the secondary DNS server. You can verify that the secondary DNS server correctly resolves the dns query. The following example is from VPCS in GNS3.

PC1\&gt; show ip

NAME        : PC1[1]

IP/MASK     : 192.168.144.80/24

GATEWAY     : 192.168.144.1

DNS         : 192.168.144.12  192.168.144.12

DHCP SERVER : 192.168.144.12

DHCP LEASE  : 349, 600/300/525

DOMAIN NAME : khaledmahmud.net

MAC         : 00:50:79:66:68:00

LPORT       : 10034

RHOST:PORT  : 127.0.0.1:10035

MTU:        : 1500

PC1\&gt; ping www.yahoo.com

www.yahoo.com -\&gt;\&gt; new-fp-shed.wg1.b.yahoo.com

new-fp-shed.wg1.b.yahoo.com resolved to 72.30.35.10

84 bytes from 72.30.35.10 icmp\_seq=1 ttl=127 time=31.124 ms

84 bytes from 72.30.35.10 icmp\_seq=2 ttl=127 time=108.838 ms

84 bytes from 72.30.35.10 icmp\_seq=3 ttl=127 time=26.307 ms

84 bytes from 72.30.35.10 icmp\_seq=4 ttl=127 time=25.554 ms

84 bytes from 72.30.35.10 icmp\_seq=5 ttl=127 time=25.334 ms

PC1\&gt;

# 4. Enable NAT in Linux

To set up a linux server to perform NAT service, follow the steps below.

### Enable IP Forwarding (Routing)

For just short term (until next boot), set the value of /proc/sys/net/ipv4/ip\_forward file to 1.

vmadmin@natbox:~$sudo echo 1 \&gt; /proc/sys/net/ipv4/ip\_forward

vmadmin@natbox:~$

For permanent change (persistent after reboot), edit /etc/sysctl.conf file. Uncomment the line containing net.ipv4.ip\_forward = 1.

### Configure IPTABLES

1. Add a POSTROUTING rule to nat table to translate IP addresses coming from the given source IP range. Specify the output interface.

vmadmin@natbox:~$sudo iptables -t nat -A POSTROUTING -s 10.99.99.0/24 -o enp0s3 -j MASQUERADE

vmadmin@natbox:~$

1. Check the new rule.

vmadmin@natbox:~$sudo iptables -t nat -L

vmadmin@natbox:~$

1. Make the rule persistent with boot. First install the package called iptables-persistent. The save the rule(s) in /etc/iptables/rules.v4 file. The iptables-persistent tool reads this file during boot.

vmadmin@natbox:~$sudo apt install iptables-persistent

vmadmin@natbox:~$sudo iptables-save \&gt; /etc/iptables/rules.v4