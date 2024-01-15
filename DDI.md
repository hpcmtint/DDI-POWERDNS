## Introduction

![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-blog-02.png)

[![Enterprise version](https://pandorafms.com/manual/!772/_media/wiki/icono-modulo-enterprise.png?w=23&h=23&tok=7d25c8 "Enterprise version")](https://pandorafms.com/en/prices/?o=dwpfms "https://pandorafms.com/en/prices/?o=dwpfms")With the IPAM extension (NG 731 version or later) you can manage the IPs of your networks, discover the hosts of a subnet and detect their availability changes (whether they respond to **ping** or not) or hostname (obtained through dns). You can also detect their operating system.

![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-youtube_ormuhteswdw-01.png)

The IPAM extension uses a recon script (dependent on the [Recon server](https://pandorafms.com/manual/!772/en/documentation/01_understanding/02_architecture#the_recon_server "en:documentation:01_understanding:02_architecture")) to perform all the underlying logic. IP management is independent of whether it has agents installed on those machines or an agent with remote monitors on that IP or not. You may optionally “associate” an agent to the IP and manage that IP, but it does not affect the monitoring being performed.

More information can be found in the video tutorial «[PFMS Reviews IPAM: IP Address Management](https://www.youtube.com/watch?v=-ByuHaNnrDs "https://www.youtube.com/watch?v=-ByuHaNnrDs")».

## IP Detection

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam.png")

A network can be configured (via network and network mask) so that address recognition is executed automatically from time to time or only manually. This system uses underneath the [Recon server (Netscan)](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/04_discovery#netscan "en:documentation:03_monitoring:04_discovery"), but it manages it automatically.

-   For its correct operation, it is important to make sure that you have the **xprobe** and **fping** packages installed. To find out more, check the documentation about [installing Pandora FMS](https://pandorafms.com/manual/!772/en/documentation/02_installation/01_installing "en:documentation:02_installation:01_installing") for more information.
    
-   The detection of operating systems is always approximate and is based on **xprobe**. For more accurate results use **nmap**.
    
-   Detection in virtual environments is difficult because the hypervisor used has to forward the packets accurately and correctly to the hosted device (virtual machine).
    
-   In **Ubuntu server 22** this PFMS IPAM feature is still in experimental phase.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-youtube_ormuhteswdw-02.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-youtube_ormuhteswdw-02.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-youtube_ormuhteswdw-02.png")

## IPs with installed agents

The first time the network is detected, after creating it in the IPAM control panel, Pandora FMS will look for the IPs of that network. If it detects that the IP is operational, it will manage it. If it does not respond to **ping**, it will be left unmanaged. Any managed IP that changes state (stops responding to ping) will generate an event in the system. You may manually manage as many IPs as you want, editing them to give you an alias/hostname, a description or even force their operating system.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-youtube_ormuhteswdw-03.png?w=800&tok=f140c7)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-youtube_ormuhteswdw-03.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-youtube_ormuhteswdw-03.png")

Special mention should be made of the fact that when IPAM detects an IP that has a software agent installed and has that IP assigned to it, it makes it possible to identify it explicitly, as in the case of IP `.125` in this screenshot:

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms_ipam_ip_address_with_agent.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms_ipam_ip_address_with_agent.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms_ipam_ip_address_with_agent.png")

And if you click on the detail view of the agent:

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam-agent-detail.png?w=550&tok=3435b9)](https://pandorafms.com/manual/!772/_detail/wiki/ipam-agent-detail.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam-agent-detail.png")

## Views

From right to left:

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-default-menu.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-default-menu.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-default-menu.png")

### Sites

It allows you to edit network sites (by clicking on name, column **Name**), delete with the corresponding trash can icon and create new network sites with the **Create** button.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-creating-sites.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-creating-sites.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-creating-sites.png")

To create a new network location type the name, by default the **Parent** field will be unselected indicating that it is a _root site_. In case it is a node select either a _root site_ or another node. Click the **Create** button again to save the new network site. The editing process is similar but uses the **Update** button.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-editing-sites.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-editing-sites.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-editing-sites.png")

Note that deleting a root site or a subnode with another subnode(s) will break the entire related chain.

If you repeat any names (case insensitive) it will be duly noted when saving or updating a record.

### Network locations

It allows editing network locations (click on name, **Name** column), deleting with the corresponding trash can icon (or multiple deletion by selecting each line and then pressing the **Delete** button) and creating new network locations with the **Create** button.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-network_location_config.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-network_location_config.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-network_location_config.png")

To create a new network location type the name and press the **Create** button again. The editing process is similar but uses the **Update** button.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-network_location_config-create.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-network_location_config-create.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-network_location_config-create.png")

To know the identifier of each location place the pointer over the location name and look at the last number of the link:

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-network_location_id.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-network_location_id.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-network_location_id.png")

If you repeat a name (case insensitive) it will be duly indicated when saving or updating a record.

### Operation view

It allows you to see the networks created, see their IP addresses, modify or delete them.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-operation_view.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-operation_view.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-operation_view.png")

By clicking on each of the items in the first column **Network** or on its corresponding icon in the **Action** column you can enter the **[Addresses view](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/11_ipam#addresses_view "en:documentation:03_monitoring:11_ipam")** ; to delete click on the trash can icon in the same column.

You can search by text in the **Search** field (by name, CIDR network address or description) and/or by [network location](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/11_ipam#network_locations "en:documentation:03_monitoring:11_ipam") (**Location**) and/or by [network site](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/11_ipam#sites "en:documentation:03_monitoring:11_ipam") (**Site**) and/or by [virtual network](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/11_ipam#vlan_ipam "en:documentation:03_monitoring:11_ipam") (**Vlan**) and then click on the **Search** button to refine the results.

#### Creation of an IPAM network

-   Operating system detection is always approximate and is based on **xprobe**. For more accurate results use **nmap**.
    
-   In Ubuntu server 22 this PFMS IPAM feature is still in experimental phase.
    

-   To create a new network click **Create** and fill in the following fields:
    

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-create.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-create.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-create.png")

-   **Network**: Network in IP/mask (CIDR) adress format.
    
-   **Name**: Network name.
    
-   **Discovery server**: Server in charge of this task.
    

-   **Vrf**: [Virtual routing and forwarding](https://en.wikipedia.org/wiki/Virtual_routing_and_forwarding "https://en.wikipedia.org/wiki/Virtual_routing_and_forwarding") by means of a PFMS agent (this allows, among other things, IP address overlapping). Type in at least two letters to select the agent.
    
-   **Monitoring**: Include statistical monitors.
    
-   **Lightweight mode**: Much faster network scanning without performing hostname or operating system detection of the detected hosts.
    
-   **Group**: Target group for the monitoring agent.
    
-   **Scan interval**: Time period (days) for automatic or manual check. Set zero for manual check.
    
-   **Operator users**: [Network operator](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/11_ipam#acl_users "en:documentation:03_monitoring:11_ipam") users. Only [superadmin](https://pandorafms.com/manual/!772/en/documentation/01_understanding/03_glossary#superadmin "en:documentation:01_understanding:03_glossary") type users or users with Pandora Administrator (PM) rights can create or modify the networks. See also [ACL Enterprise](https://pandorafms.com/manual/!772/en/documentation/04_using/11_managing_and_administration#acl_enterprise_system "en:documentation:04_using:11_managing_and_administration").
    

Click **Create** to save the network.

-   To edit an existing network click the wrench icon [![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=b710e9&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_config.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_config.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_config.png?id=es:documentation:03_monitoring:11_ipam") :
    

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-edit.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-edit.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-edit.png")

Click **Update** button to save changes of the IPAM network.

#### Import through CSV file

From version NG 758 onwards, such information can be imported from comma-separated values in `.csv` format. The order is the following:

```
network,network name,description,location(ID),group(ID),monitoring(0 or 1),lightweight mode(0 or 1),scan interval(days),recon server(ID)
```

Detailed description of each of the fields:

1.  **network**: Network in IP address/mask format (CIDR).
    
2.  **network name**: Name of the network.
    
3.  **description**: Description (_optional_)
    
4.  **location(ID)**: [Network location](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/11_ipam#network_locations "en:documentation:03_monitoring:11_ipam") (numeric identifier). _Must have at least one defined network location._
    
5.  **group(ID)**: Target group for monitoring agents. Zero for group `All`; to find out the numerical identifier of each group, go to **Profiles** → **Manage agent groups** ([defined groups](https://pandorafms.com/manual/!772/en/documentation/04_using/11_managing_and_administration#group_creation "en:documentation:04_using:11_managing_and_administration")).
    
6.  **monitoring(0 or 1)**: Include statistical monitors, zero (`0`) no, one (`1`) yes, any other value will be taken as zero.
    
7.  **lightweight mode(0 or 1)**: It is much faster scanning the network without performing hostname or operating system detection of the detected hosts; zero (`0`) no, one (`1`) yes, any other value will be taken as zero.
    
8.  **scan interval(days)**: Interval period (days), zero (`0`) if manual.
    
9.  **recon server(ID)**: Server in charge of this task (numeric identifier; zero if using Satellite server). Go to **Servers** → **[Manage servers](https://pandorafms.com/manual/!772/en/documentation/04_using/11_managing_and_administration#servers "en:documentation:04_using:11_managing_and_administration")** and locate Discovery server:
    

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-discovery_server_id.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-discovery_server_id.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-discovery_server_id.png")

Click on the **Edit** icon and take note of the numeric Discovery server identifier in the address bar (URL):

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-discovery_server_id-url.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-discovery_server_id-url.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-discovery_server_id-url.png")

-   You must add exactly nine (9) fields for each line, no more and no less.
    
-   The only optional field is the description: **all other fields are mandatory**.
    

It is extremely important that you **select the field separator correctly**: comma (default) or semicolon, and so on.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-upload_csv_file.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-upload_csv_file.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-upload_csv_file.png")

Click **Upload File**, select the file in CSV format, choose the appropriate separator in **Separator** and click **Upload** to finish the import.

#### Filters

On both views, you can sort by IP, Hostname and last update.

You can filter by a text substring, which will look for substrings in IP, hostname or comments. Enabling the checkbox near to search box, it will force an **exact** match search by IP.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-network-filter.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-network-filter.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-network-filter.png")

Not responding hosts are not shown by default, but the filter can be customized.

It can show only the managed IP addresses too.

### Addresses view

Network IP address management and operation are splitted in two views: [edition view](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/11_ipam#edit_view "en:documentation:03_monitoring:11_ipam") and icon view.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam2.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam2.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam2.png")

This view reports information on the network, including statistics on the percentage and number of occupied IP addresses (only for managed addresses). The filtered list can also be exported to comma-separated format (CSV) that you may open with any spreas sheet for its editing.

IP addresses will be shown as icons, large or small.

Each IP adress will be a big icon (if the IP adress is reserved it will have a clear blue background and if not, white) that offers the following information:

Each IP address has at the bottom right a link to edit it (with administration rights). At the bottom left, there is a small icon showing the detected OS. On disabled addresses, instead of the OS icon, this icon will be shown:

[![](https://pandorafms.com/manual/!772/_media/wiki/disabled.png)](https://pandorafms.com/manual/!772/_detail/wiki/disabled.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:disabled.png")

When clicking on the main icon, a modal window will be opened showing all the IP information, including an associated agent and OS, the setup for that IP and other information, like the creation date, the last user edition or the last time it was checked by a server. This view allows doing a ping to that address.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam3.png?w=600&tok=c35816)](https://pandorafms.com/manual/!772/_detail/wiki/ipam3.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam3.png")

This ping is sent from the machine where the Pandora FMS Console is installed.

Also, for an easier management of free IPs, there is a button that will show a dialogue box with the next free IP to set aside or manage.

[![](https://pandorafms.com/manual/!772/_media/wiki/next_free_ipam.png)](https://pandorafms.com/manual/!772/_detail/wiki/next_free_ipam.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:next_free_ipam.png")

### Edit view

If you have enough permissions, you will have access to the setup view, where IP addresses are shown as a list. You can filter them to see only the IPs you are interested into, modify them and update all of them at once.

Some fields, are automatically filled by the recon script, like hostname, if it has a Pandora FMS agent and the operating system. You can mark those fields as “manual” and edit them.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-youtube_ormuhteswdw-04.png?w=600&tok=39d9e6)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-youtube_ormuhteswdw-04.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-youtube_ormuhteswdw-04.png")

Fields marked as “manual” will not be updated by the recon script.

Other fields you can modify are:

-   Activate events on an IP address: When availability on this address changes (answers or stops answering) or the hostname changes, a new event will be generated. When an address is created, it will always generate an event.
    
-   Mark an IP Address as managed: These addresses that will be acknowledged as assigned in the network and managed in the system: The IPs will be filtered to show only those that have been marked as managed.
    
-   Disable: Disabled IP addresses are not checked by the recon script.
    
-   Comments: A field free to add comments on each address.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam4.png?w=800&tok=784891)](https://pandorafms.com/manual/!772/_detail/wiki/ipam4.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam4.png")

### Mass operations view

There is another option to manage IPs in a massive way, helping the user managing big groups of IPs.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam7.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam7.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam7.png")

## Subnetwork calculator

IPAM includes a tool to calculate IPv4 and IPv6 subnetworks.

In this tool, you can, using an IP address and a netmask, obtain the information of that network:

-   Network (Address/Bitmask)
    
-   Netmask
    
-   The Wildcard mask
    
-   The network Address
    
-   Broadcast Address
    
-   First valid IP
    
-   Last valid IP
    
-   Number of IPs in the network
    

These fields are given in address format (decimal for IPv4 and hexadecimal for IPv6) and binary format.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam22.png?w=600&tok=e7ca19)](https://pandorafms.com/manual/!772/_detail/wiki/ipam22.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam22.png")

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam11.png?w=600&tok=4fff4e)](https://pandorafms.com/manual/!772/_detail/wiki/ipam11.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam11.png")

## Recon / Discovery server task creation

The IPAM module uses the Discovery server Net Scan. The IPAM-type tasks that can be seen on the recon server are created by the IPAM module and should not be “manually” created or deleted.

For more information about how to carry out a recon task, check the [Discovery](https://pandorafms.com/manual/!772/en/documentation/03_monitoring/04_discovery#netscan "en:documentation:03_monitoring:04_discovery") section.

## VLAN IPAM

The VLAN (virtual private local area network or Virtual LAN) administration view allows you to create or update VLANs in a simple way. To create a new VLAN, a unique name is mandatory and a description is optional.[![](https://pandorafms.com/manual/!772/_media/wiki/ipam33.png?w=600&tok=10996c)](https://pandorafms.com/manual/!772/_detail/wiki/ipam33.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam33.png")

For versions NG 758, 759 and 760 you can import such information from `.csv` (in order):

-   `VLAN network`, `VLAN description`
    
    From version NG 761 onwards you can import such information from `.csv` (in order):
    

-   `VLAN network`, `VLAN description`, `VLAN custom ID`
    

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-upload_csv_file.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-upload_csv_file.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-upload_csv_file.png")

Once created, it can be consulted from the list of created VLANs, where the following information is shown_:_.

-   **Name**: VLAN name.
    
-   **Description**: VLAN description.
    
-   **Networks**: Networks assigned to VLANs. If no network is assigned, a message is displayed indicating so.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam44.png?w=700&tok=470909)](https://pandorafms.com/manual/!772/_detail/wiki/ipam44.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam44.png")

Operations:

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=b710e9&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_config.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_config.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_config.png?id=es:documentation:03_monitoring:11_ipam") Update VLAN data.

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=f1ab06&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_trash.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_trash.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_trash.png?id=es:documentation:03_monitoring:11_ipam") Delete VLAN. If a VLAN is deleted, a confirmation message will be displayed.

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=17fb75&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_statistics.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_statistics.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_statistics.png?id=es:documentation:03_monitoring:11_ipam") Statistics: link to VLAN statistics view.

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=3d05d1&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_plus.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_plus.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_plus.png?id=es:documentation:03_monitoring:11_ipam") Add networks to VLAN.

-   **If there are available networks:** A selector like the one shown below will appear where you can select one or more networks.
    

**NOTE:** It is important to know that a network **cannot** belong to two different VLANs.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_3.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_3.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_3.png")

From the selector it will be possible to create a new network to add to the list by means of the **create network** option.

-   **If there are no available networks:** An informative message will appear.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_3_1.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_3_1.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_3_1.png")

### IPAM VLAN Statistics

To get information from a VLAN, there is a view that shows the statistics.

-   Name and description.
    
-   Statistical data:
    
    -   Total available IPs.
        
    -   IP occupation and availability.
        
    -   Managed IPs.
        
    -   Reserved IPs.
        

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_4.png?w=500&tok=140f49)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_4.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_4.png")

Additionally, for each of the networks that are part of the VLAN, the following statistics and information will be displayed:

-   Name.
    
-   Recon Interval.
    
-   Localization.
    
-   Description.
    
-   Network scan progress.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_5.png?w=700&tok=1209ea)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_5.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_5.png")

These statistics can be exported to CSV format selecting the button at the top:

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam55.png?w=800&tok=f6487b)](https://pandorafms.com/manual/!772/_detail/wiki/ipam55.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam55.png")

### IPAM VLAN Wizard

This view will allow to create a VLAN easily and quickly via SNMP.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam66.png?w=600&tok=039f15)](https://pandorafms.com/manual/!772/_detail/wiki/ipam66.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam66.png")

In order to perform the SNMP query, it is required to enter address, community and version. Once entered, it will show a list with all the VLANs available for that address, detailing the following data:

-   Name of the VLAN. When there are no interfaces assigned to a VLAN, the default name is 'default'.
    
-   Interfaces.
    
-   Description.
    
-   Status. If the status is 'default', this field will be empty. If the VLAN is not created, a checkbox will appear to select it for later creation, adding as description the address and its interfaces as shown in the example:
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_8.png?w=800&tok=132da3)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_8.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_8.png")

## IPAM Supernet

The SuperNet Administration view allows to create or update a supernet in a simple way.

To create a new supernet, enter:

-   **Supernet**: Name of the supernet. This field is required and must be unique.
    
-   **Addres**: Network: address and mask. Field required.
    
-   **Mask**: Net mask. Field required.
    
-   **Subnetting Mask**: Subneting mask. This field is optional.
    
-   **Description**. Optional.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam8.png?w=600&tok=8ecb8f)](https://pandorafms.com/manual/!772/_detail/wiki/ipam8.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam8.png")

From version NG 758 onwards you can import such information from `.csv` (in order):

-   `name`, `description`, `address`, `mask`, `subnetting mask`
    

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-ipam-upload_csv_file.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-ipam-upload_csv_file.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-ipam-upload_csv_file.png")

Once created, it will be possible to check it from the list of created supernets, where the following information is shown:

-   **Name**: Supernet name.
    
-   **Address/Masks**: Supernet address and mask
    
-   **Subnetting mask**.
    
-   **Networks**: Networks assigned to Supernet. In case of not having any network assigned, a message is shown indicating so.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_10.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_10.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_10.png")

Operations:

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=b710e9&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_config.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_config.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_config.png?id=es:documentation:03_monitoring:11_ipam")Update Supernet data.

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=f1ab06&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_trash.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_trash.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_trash.png?id=es:documentation:03_monitoring:11_ipam")Delete Supernet. In case of deleting a supernet, a confirmation message will be displayed.

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=17fb75&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_statistics.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_statistics.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_statistics.png?id=es:documentation:03_monitoring:11_ipam")Statistics: link to the Supernet statistics view.

[![](https://pandorafms.com/manual/!772/lib/exe/fetch.php?w=21&h=21&tok=3d05d1&media=https%3A%2F%2Fprewebs.pandorafms.com%2Fmanual%2F_media%2Fwiki%2Ficon_plus.png)](https://prewebs.pandorafms.com/manual/_detail/wiki/icon_plus.png?id=es:documentation:03_monitoring:11_ipam "https://prewebs.pandorafms.com/manual/_detail/wiki/icon_plus.png?id=es:documentation:03_monitoring:11_ipam")Add networks to Supernet.

-   **If there are available networks:** A selector like the one shown below will appear where you can select one or more networks.
    

NOTE: It is important to know that a network cannot belong to two different Supernets.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_10_1.png?w=497&tok=40e202)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_10_1.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_10_1.png")

A new network can be created from the selector by selecting **next network**. If a subneting mask has been added, the next available network will be selected by default.

-   **If there are no available networks:** An informative message will appear.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_10_2.png?w=502&tok=acd5bc)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_10_2.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_10_2.png")

### IPAM Supernet Statistics

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-supernet_config-statistics.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-supernet_config-statistics.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-supernet_config-statistics.png")

To get information from a Supernet, there is a view that shows the statistics.

-   Name and description.
    
-   Statistical data:
    
    -   Total available IPs.
        
    -   IP occupation and availability.
        
    -   Managed IPs.
        
    -   Reserved IPs.
        

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_11.png?w=500&tok=6ca2fb)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_11.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_11.png")

Additionally, for each of the networks that are part of the Supernet, the following statistics and information will be displayed:

-   Name.
    
-   Recon Interval.
    
-   Localization.
    
-   Description.
    
-   Network scan progress.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_12.png?w=650&tok=14fb19)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_12.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_12.png")

These statistics can be exported to a spreadsheet selecting the button at the top:

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam77.png?w=800&tok=23b17d)](https://pandorafms.com/manual/!772/_detail/wiki/ipam77.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam77.png")

### IPAM Supernet Map

A map with all the created Supernets will be shown:

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam88.png?w=800&tok=e69abc)](https://pandorafms.com/manual/!772/_detail/wiki/ipam88.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam88.png")

Networks and Supernets will be represented as nodes. The difference between the two is that Supernets have a thicker edge.

The following information will be displayed inside each node:

-   Net or Supernet name.
    
-   Occupation percentage.
    
-   Number of available IPs.
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_15.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_15.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_15.png")

In the Pandora FMS **setup** in the Enterprise section, critical and warning thresholds can be configured, showing nodes in red for critical and orange for warning:

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_16.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_16.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_16.png")

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_17.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_17.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_17.png")

Statistics will be shown by clicking on a node:

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_18.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_18.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_18.png")

### Supernet treeview

The supernet tree view shows all created supernets in a simplified graphical form, and clicking on the respective icon will display a pop-up window with additional information and the possibility to modify the item in another tab of the web browser.

[![](https://pandorafms.com/manual/!772/_media/wiki/pfms-admin_tools-ipam-supernet_treeview.png)](https://pandorafms.com/manual/!772/_detail/wiki/pfms-admin_tools-ipam-supernet_treeview.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:pfms-admin_tools-ipam-supernet_treeview.png")

## IPAM Network Use Monitoring

IPAM's new system allows creating reports, graphs, alerts, etc.

In order to do this, the network to be monitored must have the monitoring option activated, as well as the group assignment option.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam99.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam99.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam99.png")

This will create an agent in Pandora whose name will be **IPAM\_<network name>**, and whose modules will have the following info:

-   Total number of available IPs.
    
-   Total number of free (unassigned) IPs.
    
-   Total number of occupied IPs (assigned, reserved).
    
-   Total number of reserved IPs.
    
-   % of free IPs (free/available).
    

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam222.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam222.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam222.png")[![](https://pandorafms.com/manual/!772/_media/wiki/ipam111.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam111.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam111.png")

## IPAM for DHCP Server (Windows):

The [Pandora FMS IPAM DHCP](https://pandorafms.com/library/ipam-dhcp-tool/ "https://pandorafms.com/library/ipam-dhcp-tool/") tool provides DHCP monitoring modules for a MS Windows® DHCP server and complements the information shown in the IPAM extension.

First, a collection must be created in Pandora FMS console. For example, a custom short name like _IPAM_ can be used.

Secondly, the IPAM agent tool is uploaded to the collection and the collection is rebuilt.

Thirdly, the collection is assigned to the Pandora FMS agent of the Windows DHCP server®.

Finally, the execution is registered in the **Complements** tab in the Pandora FMS agent administration:

```
%ProgramFiles%\pandora_agent\collections\ipam\ipam_agent_tool.exe
```

After a while, the file will be transferred to the agent and executed, providing the following modules:

-   \[network\] DHCP usage.
    
-   \[network\] available DHCP IPs.
    
-   \[network\] free DHCP IPs.
    
-   \[network\] assigned DHCP IPs.
    
-   \[network\] reserved DHCP IPs.
    

The information provided in the IPAM extension is not overwritten if the destination IP addresses are in “managed” status.

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_22.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_22.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_22.png")

[![](https://pandorafms.com/manual/!772/_media/wiki/ipam_23.png)](https://pandorafms.com/manual/!772/_detail/wiki/ipam_23.png?id=en%3Adocumentation%3A03_monitoring%3A11_ipam "wiki:ipam_23.png")

[Go back to Pandora FMS documentation index](https://pandorafms.com/manual/!772/en/documentation/start "en:documentation:start")
