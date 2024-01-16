![](https://blog.apnic.net/wp-content/uploads/2020/03/Kea_Banner_2-555x202.png?v=34a58cff1683c10626a743d3592e8043)

In my [previous post,](https://blog.apnic.net/2020/03/06/how-to-getting-started-with-kea-dhcp-for-ipv4-ipv6/) I introduced Kea, an open-source Dynamic Host Configuration Protocol (DHCP) server.

In this post I’ll dive deeper into its more advanced features, including its database support, hooks, High Availability and API support capabilities, as well as touch on a new dashboard we are developing to make it easier to use.

## Kea database support

Any DHCP server that deals with data has different life cycles. In the case of Kea, the information being handled is split into three types:

1.  **Leases** — this is the run-time information that covers who currently uses which address or prefix, for how long, and when the addresses/prefixes should be renewed or expired.
2.  **Host reservations** — this is the aspect of the configuration that defines what kind of device-specific configuration should take place, such as an address being reserved, or a device getting some extra options or being assigned to a specific client class.
3.  **Configuration** — this is the remaining part of the configuration — subnets, classes, options, pools, networks, timers, network interfaces, and more.

Kea stores each type of information independently. If you prefer your server installation to be small, you can use memfile — our most compact and simplest configuration and generally the fastest to use. Your configuration will be stored in a JSON file on disk and will contain host reservations; the lease information will be stored on a local disk in a plain-text CSV file.

[![Figure 1 — Kea supports many database backends.](https://blog.apnic.net/wp-content/uploads/2020/03/Kea-support-1024x213.png)](https://blog.apnic.net/wp-content/uploads/2020/03/Kea-support.png)

Figure 1 — Kea supports many database backends.

However, Kea offers much more than that. The code can point leases, hosts, and the configuration backends (also called config backends) to other storage locations, namely databases. Leases and hosts can optionally be stored in MySQL, PostgreSQL, and even Cassandra. 

Kea version 1.6.0 has added the ability to also store most of the configuration (such as networks, subnets, and pools) in a database. MySQL is currently the only supported configuration backend, but if you’re interested in PostgreSQL, please get in touch with ISC.

While the SQL database backends are [decidedly slower than the standard memfile](https://kb.isc.org/docs/kea-performance-tests-140-vs-150) option, they have a number of advantages that still make them a very popular choice. 

First, you can connect multiple Kea servers to the same database and they all will share the information. While there’s some overhead with each new Kea instance, this method is currently the only way to provide a redundancy of more than two servers handling the same part of your network. 

Another advantage is that if you decide to use MySQL, MariaDB, or PostgreSQL, you can use any of the third-party apps, monitoring tools, reporting tools, DB optimizations, and redundancy and clustering mechanisms that are available on the market. You can also tweak the database and Kea will pick up your changes instantly; in most cases, there’s no need to restart or notify Kea.

One particular example of such a mechanism is the config backend. You can set up a central database with the configuration information for your whole network, as well as your remote Kea instances with very basic configuration (mostly database credentials, logging information, and local network interface names), and then instruct them to retrieve their actual configuration (subnets, pools, and so forth) from the central database. Kea servers can be told to poll periodically for any changes in the configuration. When set up that way, you can push configuration updates to the central server to be retrieved automatically by your remote servers. For more details, see [Kea ARM, Section 4.3](https://kea.readthedocs.io/en/kea-1.6.1/arm/admin.html#supported-backends).

## Hooks overview

Kea offers a flexible and customizable extension mechanism called **hooks**. This feature lets Kea load one or more hooks libraries and, at various points in its processing (‘hook points’), call functions in them. These functions perform whatever custom processing is required. This mechanism is similar to the modules mechanism in Apache.

ISC provides a well-documented API for writing hook libraries, and there are some libraries developed by third parties. Some operators choose to develop their own applications, for example to integrate with their internal systems. However, by far the most popular hook libraries are those provided by ISC.

Here are several of the most powerful hook libraries available from ISC:

-   **High Availability** — once loaded, this library allows two Kea servers to cooperate, providing a High Availability redundancy solution. Load-balancing, hot-standby, and backup servers are supported. (More on that later in this blog post.)
-   **Lease commands** — Kea provides a powerful RESTful, JSON-based API to control almost every aspect of its state and behaviour. The initial list of commands is fairly basic; to enable many optional features, a hooks library has to be loaded. Lease commands are an example of such a hooks library. It provides the ability to manage leases.
-   **Forensic logging** — Kea provides a powerful logging environment that can be configured to address regular operations. The log messages are highly technical and are intended to be read by network engineers. However, there is often a need to provide information about a specific IP address in response to a request from law enforcement. For this purpose, Kea has a dedicated, optional mechanism, where forensic logging provides an independent logging mechanism with the essential information typically requested (when an IP address was assigned, renewed, and released; what MAC was used; which switch ports were used, and so forth).
-   **Host Commands** — As mentioned earlier, Kea stores host reservations in a database. Inserting data directly into the database using SQL commands is possible, but it requires care and precision. Many users choose to use the Host Commands hooks library instead, as these commands provide a convenient way to manage reservations, such as querying for existing entries, adding new ones, and removing those that are no longer needed. Moreover, they execute consistency checks that are more thorough than the database schema could enforce. For example, Kea can check if a host reservation belongs to a subnet that is indeed defined in the currently running configuration.

While most of the ISC-created hooks are open-source, some of them are provided as a premium package that can be purchased for a modest price. Several hook libraries are available only to operators who hold a support contract with ISC — this is one of the ways we fund open-source software.

For a complete list of available hooks libraries from ISC, please see [Kea ARM, Section 15.4](https://kea.readthedocs.io/en/kea-1.6.1/arm/hooks.html#available-hooks-libraries).

## High Availability

High Availability is a desired property of almost any solution in a robust network. One of the key concepts of providing any highly available service is to avoid a single point of failure. In terms of the DHCP service, this means providing more than one DHCP server that does not share any infrastructure elements.

It is worth noting that ISC DHCP, an earlier DHCP implementation, offered a somewhat similar mechanism called _failover_. On a fundamental level, both provide similar operations — two servers can cooperate and jointly provide service, even if one of the servers becomes unavailable for any reason. However, there are several [notable differences](https://kb.isc.org/docs/aa-01617):

-   ISC DHCP provided failover for DHCPv4 only. When ISC DHCP was in active development, there was no IETF draft for DHCPv6 failover. There is now (see [RFC 8156](https://tools.ietf.org/html/rfc8156)), but it was never implemented.
-   The failover protocol available in ISC DHCP is complex. Among other things, it implements the concept of Maximum Client Lead Time (MCLT), which implies that each partner’s and client’s notion of the lease time is only roughly coupled and usually differs. This design assumption causes a plethora of various complexities that work correctly when configured properly, but which are a source of much confusion.
-   The standardization work on the DHCPv4 failover draft was never completed in the IETF; as such, it is not a standard.
-   The communication between ISC DHCP servers uses on-wire encoding and thus is somewhat difficult to follow.

All of the above reasons gave the general perception that the failover feature in ISC DHCP is complex and difficult to use. To address these concerns, ISC came up with a new solution called High Availability.

Kea implements the High Availability concept in the form of a High Availability hooks library, which can be loaded on a pair of DHCPv4 or DHCPv6 servers to increase the reliability of the DHCP service in the event of an outage of one of the servers. Two modes of operation are supported: load-balancing and hot-standby.

In the load-balancing configuration, one of the servers must be designated as ‘primary’ and the other as ‘secondary’. Functionally, there is no difference between the two during normal operation; however, this distinction is required when the two servers are started at (nearly) the same time and have to synchronize their lease databases. The primary server synchronizes the database first. The secondary server waits for the primary server to complete its lease database synchronization before it starts its own synchronization.

[![Figure 2 — Kea High Availability running in load-balancing mode.](https://blog.apnic.net/wp-content/uploads/2020/03/Kea-High-Availability_bi-directional_diagram-1024x582.png)](https://blog.apnic.net/wp-content/uploads/2020/03/Kea-High-Availability_bi-directional_diagram.png)

Figure 2 — Kea High Availability running in load-balancing mode.

In the hot-standby configuration, one of the servers is also designated as ‘primary’ and the second as ‘secondary’. However, during normal operation, the primary server is the only one that responds to DHCP requests.

The secondary or ‘standby’ server receives lease updates from the primary over the control channel; however, it does not respond to any DHCP queries as long as the primary is running or, more accurately, until the secondary considers the primary to be offline. If the secondary server detects the failure of the primary, it starts responding to all DHCP queries.

[![Figure 3 — Kea High Availability running in hot-standby mode.](https://blog.apnic.net/wp-content/uploads/2020/03/Kea-High-Availability_diagram-1024x567.png)](https://blog.apnic.net/wp-content/uploads/2020/03/Kea-High-Availability_diagram.png)

Figure 3 — Kea High Availability running in hot-standby mode.

Finally, Kea implements the concept of backup servers. It’s possible to configure additional servers that will receive updates; those servers will not be put into service automatically, but this option allows you to have servers that are almost a ‘drop-in’ replacement with an up-to-date database in the event of catastrophic failure(s) that take out both your primary and secondary servers. There is no specific limit on the number of backup servers, but each of them introduces a slight performance degradation to your primary-secondary pair. For more details, see [Kea ARM, Section 15.13](https://kea.readthedocs.io/en/kea-1.6.1/arm/hooks.html#ha-high-availability).

## Managing Kea with the API

A classic approach to daemon configuration assumes that the server’s configuration is stored in a configuration file and when the configuration is changed the daemon is restarted. This approach has the significant disadvantage of introducing periods of downtime, during restart, when client traffic is not handled. Another risk is that if the new configuration is invalid, for any reason, the server may refuse to start, which will further extend the downtime period until the issue is resolved.

[![Figure 4 — Kea API can be used over UNIX sockets or the HTTP/HTTPS interface.](https://blog.apnic.net/wp-content/uploads/2020/03/Kea_API_diagram-1024x738.png)](https://blog.apnic.net/wp-content/uploads/2020/03/Kea_API_diagram.png)

Figure 4 — The Kea API can be used over UNIX sockets or the HTTP/HTTPS interface.

To avoid such problems, the DHCPv4, DHCPv6, and DHCP-DDNS servers in Kea include support for a mechanism that allows online reconfiguration without requiring server shutdown. Those daemons can open UNIX sockets and receive JSON-based commands locally. Kea also provides a Control Agent (CA), which exposes the API interface over HTTP or HTTPS.

The most basic command is _list-commands_, which returns the current list of supported commands. Other very useful ones are _config-get_ (which returns the current running configuration), _config-set_ (which applies a new configuration), _shutdown_ (which instructs the server to shut down), and _version-get_ (which returns detailed information about the Kea version). 

As of version 1.6.2, Kea supports close to 150 different commands. Many of them are related to various subsystems that are provided by hooks; therefore, it’s often required to load certain hooks before commands become available. For example, to use _lease4-get_, _lease6-add_, and similar _lease-manipulation_ commands, you need to load the _lease\_cmds_ hooks library first.

Sending commands to Kea is easy. In the UNIX socket, a socat tool (available in most Linux and BSD distributions) can be used:

```
<strong>echo ‘{ “command”: “list-commands” }’ | socat UNIX:/tmp/kea4-ctrl-socket -,ignoreeof</strong>
```

Sending commands over the HTTP interface is only marginally more complex. Kea provides an example client written in python called _kea-shell_:

```
<strong>kea-shell --host 192.0.2.1 --port 8001 --service dhcp4 list-commands</strong>
```

Alternatively, curl can be used to send the commands:

```
<strong>curl -X POST -H "Content-Type: application/json" -d '</strong>
<strong>{ "command": "config-get", "service": [ "dhcp4" ] }' http://localhost:8000/</strong>
```

For more information, see [Kea ARM, Section 17](https://kea.readthedocs.io/en/kea-1.6.1/arm/ctrl-channel.html#management-api).

## New dashboard coming soon

Given that the Kea daemon is command-line only, ISC is working on a graphical interface called Stork that aims to make Kea (and BIND) monitoring and management much easier. The first release, intended for broader public experiment, is expected in April 2020. 

Stork integrates with Grafana to provide visibility into pool utilization, High Availability status, platform and application CPU utilization and lease activity. More details will be shared during the webinar.

![](https://blog.apnic.net/wp-content/uploads/2020/03/Stork_Screenshot-1024x780.png)

Figure 5 — Stork is a graphical interface to make Kea (and BIND) monitoring and management easier.

## Want to know more?

Please visit the [Kea website](https://kea.isc.org/) for more information, and watch a recording of the [APNIC webinar on Kea](https://academy.apnic.net/intro-to-kea-dhcp-recording/) where these and other Kea-related topics were covered in more detail.

_Tomek Mrugalski is the Director of DHCP Engineering at the Internet Systems Consortium (ISC) and one of the original authors and engineers of Kea DHCP._

<table><tbody><tr><td><nobr>Rate this article</nobr></td><td></td></tr></tbody></table>

___

The views expressed by the authors of this blog are their own and do not necessarily reflect the views of APNIC. Please note a [Code of Conduct](https://blog.apnic.net/?p=395) applies to this blog.
