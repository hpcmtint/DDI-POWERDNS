![](https://bluecatnetworks.com/wp-content/uploads/2020/06/Network-servers-racks-blue-1600px-1024x682.jpg)

If you’re asking “What is IPAM,” here you go: IP address management (IPAM) is a method for planning, tracking, and managing IP address space on a network. IP, which stands for Internet Protocol, is how devices on a network communicate with each other.

IP addressing is a logical means of assigning addresses to devices on an IP network. Each device connected to a network requires a unique IP address.

Managing tens of thousands of IP addresses for large enterprise networks can quickly get complicated. It’s assigning IP addresses to devices, yes. But it’s also changing them, deleting them when devices leave the network, dealing with potential address conflicts, setting up subnets, and designating addresses for use by certain applications or clouds.

IPAM is a method to bring organization to what would otherwise be IP chaos.

In this glossary entry, we’ll define IPAM and then take a deeper look at the benefits of IPAM software tools. Furthermore, we’ll look at how it can integrate with [DNS](https://bluecatnetworks.com/glossary/what-is-dns/) and [DHCP](https://bluecatnetworks.com/glossary/what-is-dhcp/).

## What is IPAM?

### First, some networking basics

IP (version 4) addresses are 32-bit integers that can be expressed in hexadecimal notation. The more common format, known as dotted quad or dotted decimal, is x.x.x.x, where each x can be any value between 0 and 255. For example, 192.0.2.146 is a valid IPv4 address.

While [IPv4](https://bluecatnetworks.com/glossary/what-is-ipv4/) still routes most of today’s internet traffic, we’ve run out of address space. As a result, the internet is undergoing a gradual transition to IPv6. This latest version of IP is a 128-bit address space, with both letters and numbers expressed in hexadecimal format (for example, 2002:db8::8a3f:362:7897).

### Keeping track of all those IP addresses

If there are hundreds or thousands of devices on your network, imagine trying to keep track of each of their IP addresses yourself. It gets out of hand fast.

IPAM can give network admins a real-time inventory of both used and unassigned IP addresses, including details like their subnets, status, hostname, and associated hardware.

The challenge with tracking IP addresses gets even more complicated when addressing involves DHCP. This is how most large enterprise networks are run.

When a device such as a laptop or a smartphone joins a network, it typically asks for an IP address from a DHCP server. IP assignment from DHCP servers happens dynamically. This is sometimes called DHCP leasing.

This means that a device connected to the network doesn’t have a forever address unless the address is reserved specifically for it. The IP address can periodically change as its lease time expires unless the lease is successfully renewed.

This is especially true for devices that move around from subnet to subnet. Each IP address is only valid in one particular subnet, so roaming devices must change addresses as they roam.

![](https://bluecatnetworks.com/wp-content/uploads/2020/05/What-is-IPAM-1024x876.jpg)

## The benefits of IPAM tools

IPAM software tools are crucial for simplifying IP address management. They allow network admins to automatically discover unallocated and assignable IP addresses and easily provision IP addresses to devices on a network.

Some of the benefits in particular:

-   With all IP addresses in a central repository, you get a **consolidated view of your network**.
-   [Automating IPAM](https://bluecatnetworks.com/blog/secret-to-network-automation-dns/) can provide **faster service for end-users** and drastically reduce admin time spent on IP address space management.
-   Seeing your IP data regularly helps you to detect abnormal behavior, giving you **additional network security**.
-   IPAM **enhances operational efficiency**, saving admin time and brainpower for more important work.

### But what about spreadsheets?

Some enterprises still use IP address spreadsheets to manage all the IP addresses on their network. For small networks in a single geographic location without a lot to manage, spreadsheets may work just fine.

Small entities can at least step up from spreadsheets to [open-source IPAM solutions](https://packetpushers.net/virtual-toolbox/ip-address-managers/) like phpIPAM or netbox. These free options work for IP management on a smaller scale and usually don’t include more complex capabilities like integration with DNS and DHCP.

But gambling with spreadsheets to manage core business functions in large, global enterprises is fraught with risk. For example, consider:

-   A spreadsheet can get out sync with the network fast. Worse, fat-finger spreadsheet errors can bring down an entire network.
-   Typically, only one person can edit a spreadsheet at a time. While web-based spreadsheet solutions are available, they usually lack the automation and error checking needed to reliably manage IP addresses.
-   Automation in a spreadsheet requires someone to develop and maintain that code. This eats up valuable network admin time.
-   Access control gets sticky. Do you open spreadsheets up to non-admins to get more done but increase your chances of errors? Or do you strictly limit access to a few people, which hinders your ability to do things quickly?

In larger enterprise settings, the number of devices on a network can vary by the hour. Manual IP addressing tracking in that kind of environment isn’t feasible for any network team.

### Support for both IPv4 and IPv6

As enterprises [transition to IPv6 DNS](https://bluecatnetworks.com/blog/ease-your-transition-to-ipv6-dns/), IPAM tools can continue to provision 32-bit IPv4 addresses along with newer 128-bit IPv6 addresses.

To learn more about the difference between [IPv4 vs. IPv6](https://bluecatnetworks.com/blog/ipv4-vs-ipv6-whats-the-difference/) read this primer.

## IPAM integration with DNS and DHCP

IPAM also provides insight for network administrators who deploy and manage core DNS and DHCP services. In fact, while IPAM tools on their own can be helpful, they do not alone solve the underlying problems inherent in decentralized network infrastructure systems.

When you use IPAM software in isolation, the lack of integration with DNS and DHCP becomes a problem. These three core network functions are inextricably tied together. Keeping DNS and DHCP data synced with IPAM data makes your IPAM data much more accurate, and therefore much more valuable.

### Answering more complex questions

By integrating with [DNS servers](https://bluecatnetworks.com/glossary/what-is-a-dns-server/) and DHCP servers, IP address management software can provide information about details such as:  
![DDI integrates DNS, DHCP, and IPAM into one solution](https://bluecatnetworks.com//wp-content/uploads/2020/05/DDI-integrates-DNS-DHCP-and-IPAM.jpg)

-   IP addresses available in the address pool
-   Hostnames correlated to IP addresses
-   Devices assigned to IP addresses
-   Subnets use, including how large they are and who is using them
-   Permanent and temporary IP addresses
-   Default routers assignment to each network device

An IPAM solution also simplifies DNS and DHCP tasks, like writing a DNS record or configuring DHCP settings.

### The DDI triad

Together, DNS, DHCP, and IPAM make up a triad known as DDI.

Managing the three components the comprise the [DDI meaning](https://bluecatnetworks.com/glossary/what-is-ddi/) separately presents inherent risks. Bringing them together into one managed solution transforms network management. DDI provides core network services and enables communications across all points of the network.

And when it comes to managing it, admins get visibility and control of their network from a single pane of glass with the [BlueCat platform](https://bluecatnetworks.com/adaptive-dns/ddi/).

#### Related content
