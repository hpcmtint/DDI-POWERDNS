# DDNS with ISC-DHCP and PowerDNS on Debian 10 Buster - The Dummit

[DDNS with ISC-DHCP and PowerDNS on Debian 10 Buster](https://thedummit.com/ddns-with-isc-dhcp-and-powerdns-on-debian-10-buster/)
Powered by OpenAI

##### Links

*   [Carll On Medium](https://carll.medium.com/get-up-and-running-on-with-your-own-dns-and-dhcp-server-from-scratch-powerdns-isc-dhcp-server-4b9d6185d275)
*   [How to compile, install and configure openLDAP in Centos 8](https://thedummit.com/how-to-compile-install-and-configure-openldap-in-centos-8/)

##### Marks

Posted on 3 November 2021 in [Tutorial](https://thedummit.com/category/tutorial/)

![](https://thedummit.com/wp-content/webpc-passthru.php?src=https://thedummit.com/wp-content/uploads/2020/12/ddns3-900x563.jpg&nocache=1)

Before we begin, there is something important thing to do:

1\. If your recursor and authoritative run under 1 machine, you need to split them because isc-dhcp cannot define dns port in it’s configuration file .

for example :your recursor reserve port 53 and you authoritative use 5300) this cannot work for isc-dhcp, because it need to commit the record to authorization server which is currently using non standard dns port (5300). isc-dhcp **only** able to communicate with authoritative server using port 53

2\. it is mandatory for zone domain name **must have equal name** to ddns-domainname , otherwise it won’t work (you’ll see error on log file)

So, i’m going to jump start to ISC-DHCP installation since i’ve already make tutorial for PowerDNS installation [here](https://thedummit.com/powerdns-how-to-setup-and-configure-for-local-dns/).

In this tutorial i will use Debian 10 as base OS for DHCP Server (you can also use ubuntu server too). The reason i’m using debian base linux because there pre-compiled command for dhcp that only available on debian base OS (like _**dhcp-lease-list**_ that only available on Debian base linux and not in other linux distro).

**Prerequisite**

*   **Creating TSIG Key**

You need to generate TSIG key on powerdns server in order to make ISC-DHCP able to communicate with powerdns (because isc-dhcp need to update records inside PowerDNS database).

\# Create a key called 'dhcp-key' using hmac-sha256
root@dns1:~\# pdnsutil generate-tsig-key dhcp-key hmac-sha256
Generating new key with 64 bytes
Create new TSIG key dhcp-key hmac-sha256 eNgfPSPxyN3xX4Z91Ba+UfQaBXYa7l67Nw/F4iuKOEnOllI+A1ZB5MK8qpemjSThO3riCtnJc01HPMKDFMqADA==\# Enable the key on your DNS zone
root@dns1:~\# pdnsutil activate-tsig-key mydomain.com dhcp-key master
Enabled TSIG key dhcp-key for mydomain.com\# To verify you can list your keys
root@dns1:~\# pdnsutil list-tsig-keys
dhcp-key. hmac-sha256. eNgfPSPxyN3xX4Z91Ba+UfQaBXYa7l67Nw/F4iuKOEnOllI+A1ZB5MK8qpemjSThO3riCtnJc01HPMKDFMqADA==\# Verify the key is enabled for your domain
root@dns1:~\# pdnsutil show-zone mydomain.com
This is a Native zone
Zone is not actively secured
Zone has following allowed TSIG key(s): dhcp-key
Metadata items:
        TSIG\-ALLOW\-AXFR dhcp-key
No keys for zone 'mydomain.com'.

Shell session

*   **Enable DNS Update**

Next you to tell powerDNS to allow dns-update from outside, in this case from isc-dhcp-server.

root@dns1:/etc/dhcp\# nano /etc/powerdns/pdns.conf#Allow dnsupdates from localhost
allow-dnsupdate-from=127.0.0.0/8\# Enable DNS updates
dnsupdate=yes

Shell session

NB : in this case, I’m putting dhcp-server and powerdns authoritative under one roof. If you choose to install isc-dhcp separately, you can change ip on the **_allow-dnsupdate-from_** with you dhcp-server ip address.

Once you complete, restart powerdns service.

**Installation**

Make sure debian package is up-to-date before continuing.

root@dhcpsvr:~#apt update
root@dhcpsvr:~#apt upgrade

Shell session

next, you can install isc-dhcp package with this command

root@dhcpsvr:~#apt-get install isc-dhcp-server

Shell session

after the installation, isc-dhcp-server will run automatically. So you need to to stop the service before you can configure and use it.

root@dhcpsvr:/etc/dhcp\# service isc-dhcp-server stop

Shell session

If you have encountered any error, don’t worry, it just need to be configured. So, next step is to open isc-dhcp configuration file :

root@dhcpsvr:~#nano /etc/dhcp/dhcpd.conf

Shell session

you can copy from an example configuration file located in share doc directory to current dhcpd configuration folder. Then change configuration suited to your needs.

root@dhcpsvr:~\# cp /usr/share/doc/isc-dhcp-server/examples/ /etc/dhcp

Shell session

Once it is done, you can continue to setup ddns.

**DDNS Configuration**

You need to edit you dhcp config file to enable ddns mode.

authoritative;
default\-lease-time 720000;
max-lease-time 2160000;
ping-check true;
update-static\-leases on;

ddns-updates on;
ddns-update-style standard;
ddns-ttl 300;
ddns-domainname "<your-domain-name>";

\# Add the TSIG key generated with pdnsutil
key "dhcp-key" {
        algorithm hmac-sha256;
        secret "eNgfPSPxyN3xX4Z91Ba+UfQaBXYa7l67Nw/F4iuKOEnOllI+A1ZB5MK8qpemjSThO3riCtnJc01HPMKDFMqADA==";
};

\# Specify the zone name and the DNS server on the localhost
zone <your-domain-name> {
        primary 127.0.0.1;
        key dhcp-key;
}
#-------------------------------
\# Internal Network Scope
#-------------------------------
subnet 10.11.88.0 netmask 255.255.255.224 {
        option domain-name-servers 172.32.55.111, 172.32.55.222;
        option domain-search "<your-domain-name>";
        option routers 10.11.88.254;

        pool {
        range 10.11.88.1 10.11.88.50;
    }
}

Shell session

You need to download the latest OUI to make dhcp-server able to show which device’s manufacture is connected to you DHCP-Server right now (make things easier if you have problem with WIFI Card)

\# Download the latest OUI list
wget -O /usr/local/etc/oui.txt http://standards-oui.ieee.org/oui.txt
\# included with isc-dhcp-server is a command that shows your dhcp clients, and the manufacturer of their MAC addresses. 
root@dns1:~\# dhcp-lease-list
Reading leases from /var/lib/dhcp/dhcpd.leases
MAC      IP      hostname        manufacturer
====================================================
00:0c:1..  10.2.   ups1            CyberPower Systems, Inc.
00:0c:1..  10.2.   ups2            CyberPower Systems, Inc.
18:66:d..  10.2.   idrac-srv1      Dell Inc.

Shell session

So all the requirement is checked, all you have to is start the dhcp service

root@dns1:~# service isc-dhcp-server start
root@dns1:~# service isc-dhcp-server status
● isc-dhcp-server.service — LSB: DHCP server
 Loaded: loaded (/etc/init.d/isc-dhcp-server; generated)
 Active: active (running) since Mon 2020–06–15 11:31:36 JST; 3s ago
 Docs: man:systemd-sysv-generator(8)
 Process: 10794 ExecStart=/etc/init.d/isc-dhcp-server start (code=exited, status=0/SUCCESS)
 Tasks: 1 (limit: 4700)
 Memory: 12.0M
 CGroup: /system.slice/isc-dhcp-server.service
 └─10806 /usr/sbin/dhcpd \-4 \-q \-cf /etc/dhcp/dhcpd.conf ens18
Jun 15 11:31:34 dns1 systemd\[1\]: Starting LSB: DHCP server…
Jun 15 11:31:34 dns1 isc-dhcp-server\[10794\]: Launching IPv4 server only.
Jun 15 11:31:34 dns1 dhcpd\[10806\]: Wrote 2 leases to leases file.
Jun 15 11:31:34 dns1 dhcpd\[10806\]: Server starting service.
Jun 15 11:31:36 dns1 isc-dhcp-server\[10794\]: Starting ISC DHCPv4 server: dhcpd.
Jun 15 11:31:36 dns1 systemd\[1\]: Started LSB: DHCP server.

Shell session

**Verify**

To verify everything is working add a dhcp client to the network and monitor the logs on your new dns/dhcp server.

\# We can also verify the DNS record is updated from the syslog
root@dns1:/etc/dhcp# grep dhcpd /var/log/syslog
dns1 dhcpd\[8861\]: DHCPRELEASE of 10.88.77.21 from ca:fe:ff:81:4c:33 (test2) via ens18 (found)
dns1 dhcpd\[8861\]: DHCPDISCOVER from ca:fe:ff:81:4c:33 via ens18
dns1 dhcpd\[8861\]: DHCPOFFER on 10.88.77.21 to ca:fe:ff:81:4c:33 (test2) via ens18
dns1 dhcpd\[8861\]: DHCPREQUEST for 10.88.77.21 (10.88.77.30) from ca:fe:ff:81:4c:33 (test2) via ens18
dns1 dhcpd\[8861\]: DHCPACK on 10.88.77.21 to ca:fe:ff:81:4c:33 (test2) via ens18
dns1 dhcpd\[8861\]: Added new forward map from test2.expresso-addict.com to 10.88.77.21
# pdnsutil list-zone will start to show records for dhcp clients. 
root@dns1:/etc/dhcp# pdnsutil list-zone expresso-addict.com
$ORIGIN .
dns1.expresso-addict.com 3600 IN A 10.88.77.30
expresso-addict.com 3600 IN NS dns1.expresso-addict.com.
expresso-addict.com 3600 IN SOA dns1.expresso-addict.com. hostmaster.expresso-addict.com. 2020061501 10800 3600 6048
00 3600
test2.expresso-addict.com 300 IN A 10.88.77.21
test2.expresso-addict.com 300 IN DHCID AAIBhkP+9vwMjeINBpbOj+uAI3l7fClXxGksBGRsmSror6Y=

Shell session

**Additional**

If you want more security, you can install ufw firewall (if not yet present in your server) and add the following port to allow dhcp server response

dns1:~\# ufw allow dns
dns1:~\# ufw allow ssh
dns1:~\# ufw allow 67/udp comment “allow isc-dhcp-server respones”
dns1:~\# ufw enable
dns1:~\# ufw status
Status: active

To                         Action      From
--                         ------      ----
DNS                        ALLOW       Anywhere
67/udp                     ALLOW       Anywhere
22/tcp                     ALLOW       Anywhere
DNS (v6)                   ALLOW       Anywhere (v6)
67/udp (v6)                ALLOW       Anywhere (v6)   
22/tcp (v6)                ALLOW       Anywhere (v6)

Shell session

Credit : this tutorial is based on this marvelous post from [Carll On Medium](https://carll.medium.com/get-up-and-running-on-with-your-own-dns-and-dhcp-server-from-scratch-powerdns-isc-dhcp-server-4b9d6185d275)

[

#### How to compile, install and configure openLDAP in Centos 8

](https://thedummit.com/how-to-compile-install-and-configure-openldap-in-centos-8/)

3 1 vote

Article Rating

Subscribe

Notify of

Label

Name\*

Email\*

Website

Δ

Label

Name\*

Email\*

Website

Δ

0 Comments

Inline Feedbacks

View all comments
