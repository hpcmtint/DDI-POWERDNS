# FW NAT PORT 

I'm running a local dns server on port 5300 to develop a software. I need my machine to use that dns but I wasn't able to tell /etc/resolv.conf to check on a different port. I searched a bit on google and I didn't find a solution.

I set 127.0.0.1 as nameserver on /etc/resolv.conf. This is my whole /etc/resolv.conf:

nameserver 127.0.0.1

Could you please tell me how can I redirect outbound traffic on port 53 to another port?

[ARTICLE](https://superuser.com/posts/1806299/timeline)

You can specify a different for DNS (for instance BIND) to LISTEN on, as that's controlled via `/etc/named.conf`, but the issue is getting the DNS client to connect to the DNS server on that port rather than the default port 53.

The `/etc/resolv.conf` config file doesn't support any form of alternative port number, so will only connect natively via port 53.

The only way to achieve what you're looking for is to have something in the middle to take the port 53 request from the client, change it to use port 5353 and then deliver that request to the server.

The way others have achieved that is to use `iptables` to reroute that internal request as needed. An example thread discussing it can be found here [https://serverfault.com/questions/401489/redirect-traffic-from-127-0-0-1-to-127-0-0-1-on-port-53-to-port-5300-with-iptabl](https://serverfault.com/questions/401489/redirect-traffic-from-127-0-0-1-to-127-0-0-1-on-port-53-to-port-5300-with-iptabl)

But the crucial points are to 1) update your `/etc/resolv.conf` to include an entry for `127.0.0.1` and then add a rule in iptables to handle the redirect as :

```
iptables -t nat -A OUTPUT -p tcp --dport domain -j DNAT --to-destination 127.0.0.1:5300
iptables -t nat -A OUTPUT -p udp --dport domain -j DNAT --to-destination 127.0.0.1:5300
```

[Share](https://superuser.com/a/1806299 "Short permalink to this answer")

[Improve this answer](https://superuser.com/posts/1806299/edit)

Follow
