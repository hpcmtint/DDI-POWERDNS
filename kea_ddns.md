# In the video below, we show you how to setup Dynamic DNS (DDNS) using Kea and Bind 9 on Ubuntu or Debian

___

What we’ll be covering is how to install the Kea DDNS module and how to configure Bind 9 and Kea to support Dynamic DNS using the TSIG protocol

Once this is setup, DNS will then be updated by the DHCP server as IP addresses are allocated to and released by computers

This can save a lot of admin work and time compared to manually maintaining DNS

Useful links:  
[https://bind9.readthedocs.io/en/v9.18.15/chapter6.html#dynamic-update](https://bind9.readthedocs.io/en/v9.18.15/chapter6.html#dynamic-update)  
[https://bind9.readthedocs.io/en/v9.18.15/chapter7.html#tsig](https://bind9.readthedocs.io/en/v9.18.15/chapter7.html#tsig)  
[https://bind9.readthedocs.io/en/v9.18.15/reference.html#dynamic-update-policies](https://bind9.readthedocs.io/en/v9.18.15/reference.html#dynamic-update-policies)  
[https://kea.readthedocs.io/en/latest/arm/ddns.html](https://kea.readthedocs.io/en/latest/arm/ddns.html)

_**Assumptions:**_  
You already have a Kea DHCP server or know how to install and configure this

And you already have a Bind 9 DNS server or know how to install and configure this

_**Create DNS Key**_  
We’ll start with the DNS server and because I want to cover both Debian and Ubuntu, I’ll switch to the root account

On Ubuntu you could use

Then supply your password

On Debian you could use

Then supply the root password

Next we need to create a key that can be used for authentication because we want to restrict which computers can issue DNS changes

Both Kea and Bind support the TSIG protocol, which stands for Transaction Signature, and so we’ll use that to create the key

We’ll switch to the Bind folder

And then create our key by running the following command

```bash
tsig-keygen dhcp1-ns1 > dhcp1-ns1.key
```

The naming convention is up to yourself, but like the one in the example provided by ISC, I’ve named this dhcp1-ns1 as it will be used by dhcp1 to send updates to ns1

The default algorithm is HMAC-SHA256, which is fine for me, but you can change this using the -a parameter

Your results will vary but the file would look something like this

```bash
key "dhcp1-ns1" { algorithm hmac-sha256; secret "M5Pi+XEzhSbdh1vfr5xnv/jw5jTY3eyUhrTUgRE9HJw="; };
```

This contains a private key, so I’ll change this to something different before this video is published, but I wanted to show you this one as an example

Now the bind group should have read access to the /etc/bind folder but check the perimissions of the file to make sure the Bind service will be able to read this file

If necessary you should change the ownership

```bash
chown root:bind dhcp1-ns1.key
```

But we should change the permissions so only root and bind users have access to it

_**Move Zone Files**_  
As previously mentioned, the Bind service has read access to files in the /etc/bind folder

This is deliberate so that if the Bind service is compromised the configuration and zone files can’t be changed

The problem though is for DDNS to work, Bind needs to be able to modify zone files

It’s recommended therefore to place dynamic zone files in the /var/lib/bind folder so that the Bind service can modify them

TIP: Even if you change the permissions for the /etc/bind folder, which we don’t want to do, DDNS still won’t work because Debian distros for instance use a security module known as AppArmor which will still block access

More sensitive, static zone files should remain in /etc/bind so they can only be controlled by someone with root access

In my case this is just a lab and I only have one domain, but if I was concerned about security I could split my network into sub-domains such as infra.homelab.lan and servers.homelab.lan or maybe have different zones such as labservers.lan and labcomputers.lan

In any case, I need to move the forward and reverse lookup files

```bash
mv db.homelab.lan /var/lib/bind/ mv db.192.168 /var/lib/bind/
```

_**Update DNS Server**_  
Because we’ve moved the zone files, we need to update the configuration for Bind to tell it where the files now are

We also need to allow zone updates, but restrict access

First we’ll make a backup copy of the existing file

```bash
cp named.conf.local named.conf.local.old
```

And then then apply our changes

```bash
include "/etc/bind/dhcp1-ns1.key"; zone "homelab.lan" { type master; file "/var/lib/bind/db.homelab.lan"; update-policy { grant dhcp1-ns1 wildcard *.homelab.lan A DHCID; }; }; zone "168.192.in-addr.arpa" { type master; file "/var/lib/bind/db.192.168"; update-policy { grant dhcp1-ns1 wildcard *.168.192.in-addr.arpa PTR DHCID; }; };
```

What we’ve done is to add a new line at the top to tell Bind to include the key file

This is an alternative option to pasting keys into the config file because there’s less risk of leaking a key’s contents i.e. if someone looks at this config file they won’t know what the secret is

My forward lookup zone is homelab.lan and the reverse lookup zone is 168.192.in-addr.arpa

For each of these I’ve updated the file location to point to the /var/lib/bind folder where the files are now stored

I’ve also added an update-policy command which allows someone or something to make DNS updates, as long as they know they key, but I’ve restricted what those updates can be

In each policy I’ve referenced the name of the key, in this case dhcp1-ns1

And I’ve used the wildcard name parameter along with an actual wildcard of \*. followed by the zone name

This is because the DHCP server will need to be able to modify multiple host entries and so I need to let it update any host entry

I also want to limit which record types can be modified

For the forward lookup zone, the DHCP server needs to update A record types i.e. host entries, whereas for the reverse lookup zone it needs to update PTR records

In both zones it needs to add DHCID records which is for the DHCP server itself to store information in DNS

By using update-policy instead of allow-update, more important record types like NS (Name Server) and MX (Email Server) cannot be changed

Now we’ll save the changes and exit

The next thing to do is to run a syntax check

If nothing is wrong you won’t get any feedback and we can then restart the Bind 9 service so that the changes take effect

But do check the service is working

Also double check that DNS is still resolving your static entries correctly

```bash
host dhcp1.homelab.lan host 192.168.201.10
```

_**Install DDNS Module**_  
As with the DNS server we’ll switch to root

Kea is modular by design, so unless you’ve installed everything then chances are you’ll need to install the DDNS module

```bash
apt update apt install isc-kea-dhcp-ddns-server -y
```

_**Create DDNS Key**_  
The DDNS module will need to authenticate to the DNS server using TSIG and we need to create a key file for that

We’ll first switch to the Kea service folder

And then create our file

```bash
"tsig-keys": [ { "name": "dhcp1-ns1", "algorithm": "hmac-sha256", "secret": "M5Pi+XEzhSbdh1vfr5xnv/jw5jTY3eyUhrTUgRE9HJw=" } ],
```

Then save and exit the file

This format is different to what was used for the DNS server but the data is the same

As before we want to keep private keys separate from the main configuration file but Kea defines things in blocks or section, so all of our keys would be defined in this single file, hence why I’ve nameed this tsig-keys.json

We do need to change the ownership

```bash
chown _kea:root tsig-keys.json
```

And also the permissions so only root and Kea have access to it

_**Configure DDNS**_  
As part of the installation, a default configuration file will be created and the DDNS service will be autoamtically started

There’s a lot of useful information in there, but it’s better to create your own file so we’ll take a backup of this one

```bash
mv kea-dhcp-ddns.conf kea-dhcp-ddns.conf.bak
```

Then we’ll create our own configuration

```bash
{ "DhcpDdns": { "ip-address": "127.0.0.1", "port": 53001, "control-socket": { "socket-type": "unix", "socket-name": "/tmp/kea-ddns-ctrl-socket" }, <?include "/etc/kea/tsig-keys.json"?> "forward-ddns" : { "ddns-domains" : [ { "name": "homelab.lan.", "key-name": "dhcp1-ns1", "dns-servers": [ { "ip-address": "192.168.100.10" } ] } ] }, "reverse-ddns" : { "ddns-domains" : [ { "name": "168.192.in-addr.arpa.", "key-name": "dhcp1-ns1", "dns-servers": [ { "ip-address": "192.168.100.10" } ] } ] }, "loggers": [ { "name": "kea-dhcp-ddns", "output_options": [ { "output": "stdout", "pattern": "%-5p %m\n" } ], "severity": "INFO", "debuglevel": 0 } ] } }
```

The first section is for the API setup, so I’ve left that as is

Then I’ve added a line to include the key file we created

Then we have the forward lookup zone defined followed by the reverse lookup zone

In each of these sections we define the zone name, the name of the TSIG key to use and the DNS server(s) to update

NOTE: Pay close attention to the period at the end of the zone name otherwise DDNS will fail, complaining it couldn’t find a matching FQDN. The period represents the root domain as exists in DNS

NOTE: If the key-name is blank then TSIG will not be used and DDNS in our case will fail because the DNS server expects it to be used

You can define multiple DNS servers if you have them but I only have one master server that can update zone files

The logging section I’ve left as is

Save the changes and exit

Next we’ll change the onwership so that Kea can access the file

```bash
chown _kea:root kea-dhcp-ddns.conf
```

Then we’ll check the file for possible syntax errors

```bash
kea-dhcp-ddns -t kea-dhcp-ddns.conf
```

And then restart the service for the changes to take effect

```bash
systemctl restart isc-kea-dhcp-ddns-server
```

With a final status check, just to be sure

```bash
systemctl status isc-kea-dhcp-ddns-server
```

_**Update DHCP Server**_  
As previously mentioned, Kea uses modules and by default, the DHCP server will not inform the DDNS module, so we need to update the DHCP configuration

Adding additional entries like these

```bash
"dhcp-ddns": { "enable-updates": true }, "ddns-qualifying-suffix": "homelab.lan", "ddns-override-client-update": true,
```

The first thing we do is to enable DDNS updates

Then we tell the DHCP server what the default DNS suffix will be

If the client only provides it’s hostname, this causes problems for DDNS because it won’t know what the FQDN is and so it won’t know what zone to update

In other words, the ddns-qualifying-suffix command is used to specify what domain name should be appended to the hostname

TIP: You can also define this on a per-subnet basis

The ddns-override-client-update setting is then set to true, because we don’t want the client deciding which records to update which is the default behaviour of Kea

Typically that would involve the client updating the forward zone while Kea updates the reverse zone

What we want is for Kea to update both so we can reduce our security risk by limiting which computers can make DNS updates

Save the changes and exit

Next we’ll check the file for possible syntax errors

```bash
kea-dhcp4 -t kea-dhcp4.conf
```

The restart the service for the changes to take effect

```bash
systemctl restart isc-kea-dhcp4-server
```

And then check the status, just to be sure

```bash
systemctl status isc-kea-dhcp4-server
```

_**Testing**_  
To make sure this is working properly, we want a computer to request a new IP address, otherwise DDNS may not be used

First we’ll check for an existing allocation

```bash
more /var/lib/kea/kea-leases4.csv
```

If one or even several records exist for this computer, then delete them and restart the DHCP service

Now either start up the computer, or bounce it’s NIC

Once it has an IP address, check the leases again

```bash
more /var/lib/kea/kea-leases4.csv
```

Then we’ll test DNS resolution for this DHCP client, but from a different computer

```bash
host popos.homelab.lan host 192.168.101.100
```

Assuming both are resolved by DNS then DDNS is working

_**Maintenance**_  
Going forward, the DNS server will now update the relevant zone files although it creates journal files as well to keep track of changes, similar to log files for databases

And if you look in the /var/lib/bind folder you’ll see new files that have a .jnl extension but they aren’t in a readable format and should be left untouched

In addition, you many not see entries present in the zone files but DNS resolution is working. This is because the changes haven’t been committed to the zone files yet

If you need to make manual changes to dynamic zones, you’ll neeed to pause dynamic updates first

Make your changes in the zone file(s), remembering to increment the serial number by at least one

You can then unpause dynamic updates

Any dynamic update attempt during this time period should be refused so it’s best done during a quiet period when IP addresses are least likely to be dynamically changed

Eventually things should catch up, if for instance a PC hostname couldn’t be added, it will be the next time the computer leases an IP address for example

_**Troubleshooting**_  
Because we’re using TSIG, make sure the clocks on your servers are accurate, otherwise DDNS will fail

One way to do that is to set up an NTS server, as I covered in another video, and configure your computers to use that

If you run into issues with your DNS entries not being updated then delete entries in the Kea database file and restart the DHCP service to make sure Kea will lease an unused IP address; If it already has an entry it seems to skip the DDNS part

You can run the following command to monitor the syslog file

Do this on both servers if DHCP and DNS are on different computers

Now reload a client that leases an IP address through DHCP and monitor the outputs

The logs should tell you what isn’t working and also why

For instance, if the TSIG key is wrong then the DNS server will show BADSIG in the logs

If the clocks are not in synch you’ll see BADTIME

On the other hand the DNS server may show an ouptut of REFUSED for instance and that could be due to the DHCP server asking to update a record type that it isn’t allowed to modify so check the update-policy command

TIP: Comment out the update-policy command and use the allow-update command instead. If that works then check the zonefile to see what records were added. Update your policy accordingly and revert to using the update-policy command

<iframe src="https://www.youtube.com/embed/wU7Wbbppaqg" allowfullscreen="" title="YouTube Video" data-darkreader-inline-border-top="" data-darkreader-inline-border-right="" data-darkreader-inline-border-bottom="" data-darkreader-inline-border-left=""></iframe>
