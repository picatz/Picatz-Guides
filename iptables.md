# Iptables Guide

This is a somewhat comprehensive guide for iptables, a command-line application to configure IP packet filter rules in the Linux kernel. Before reading this guide, it's preferred if you read the [iptables man page](http://linux.die.net/man/8/iptables) on your specific system and check that it is installed ( though usually pre-installed on most Linux distributions ). 

---

#### Contributed By

[Kent 'picat' Gruber](https://github.com/picatz)

---

#### Visual Representation

```
Breakdown of iptables logic, visually: 

─── iptables
	├── Table 1
	│	├──> Built-in chains.
	│	│	└──> Rules
	│	└── Maybe User-defined chains.
	│		└──> Rules
	│
	├── Table 2
	│	├──> Built-in chains.
	│	│	└──> Rules
	│	└── Maybe User-defined chains.
	│		└──> Rules
	│
	└── Table X 
		├──> Built-in chains.
		│	└──> Rules
		└── Maybe User-defined chains.
			└──> Rules

```

##### TLDR / GRAPH HARD / ? / ELI5 

There can be more than 1 Table. There are built-in chains. Chains be be built by users. There can be more than on Chain. Chains contain Rules. There can be more than one Rule. 

---

## Installing Iptables

Iptables is probably already on your Linux distribution. To quickly check if you have iptables installed ( assuming you're working in a BASH shell for all of this guide ):

```
$ which iptables
```

#### Debian

```
# Install on Debian:
$ sudo apt-get install iptables

# Reinstall on Debian:
$ sudo apt-get --reinstall iptables

```

#### RHEL/CentOS

Note: you may want to [check if you're running firewalld](https://www.digitalocean.com/community/tutorials/how-to-migrate-from-firewalld-to-iptables-on-centos-7) and adjust your settings accordingly. [This is another helpful guide](http://tecadmin.net/install-and-use-iptables-on-centos-rhel-7/#)

```
# Install on RHEL/CentOS ( yum ):
$ sudo yum install iptables-services 
or 
$ sudo yum install iptables

# Reinstall on RHEL/CentOS ( yum ):
$ sudo yum reinstall iptables-services

```

### Check Version

Check iptables version:

#### Debian

```
# Debian / all:
$ iptables --version
```

#### RHEL/CentOS

```
# RHEL/CentOS ( yum ):
$ yum info iptables

# RHEL/CentOS ( rpm ):
$ rpm -qi iptables

# RHEL/CentOS ( rpm ) with grep:
$ rpm -qi iptables | grep "Version"

```

### Overview 

Run on a Debian 8 box, starting from a user without sudo/root privilege: 

<img src="resources/iptables/SetupCheckIptables.gif">


---


## 3 Built-in Chains ( INPUT, FORWARD, OUTPUT )

Iptables chains comes in three default flavors: INPUT, FORWARD, and OUTPUT. These chains are made up of a list of rules to define how a packet might be filtered. Each rule specifies what to do with a packet that matches, and this is called a "target". A target may also be a jump to a user-defined chain in the same table. Which makes a lot of sense, since we're essentially mananging the flow of packets coming in, being forwarded, or being outputed to/from/for a destination. So we can logically break down these chains to understand they manage certain tasks for packet filtering:

### INPUT

The INPUT chain manages incoming connections and the behavior that will act on the packets coming in from other hosts. 

### FORWARD

The FORWARD chain manages incoming connections which aren't meant for the host, but is being forwarded to anouther. In general this rule is mostly for [NAT-related](https://en.wikipedia.org/wiki/Network_address_translation) tasks. 

### OUTPUT

The OUTPUT chain is used for, obviously, outgoing connectons from your host to anouther host. 

#### Connections, (sometimes) a two way street. 

It is an important note that connections between two hosts may require both input and output rules to be configured properly such as the ping command which requires the target host to be able to reach the host trying to ping successfully. 


### Check Chain Inormation

Check all rules, showing the interface name, the rule options (if any), and the TOS masks. Packet and byte counters should also be listed to check, for example, if your host is being used for any forwarding or being used as a pass-through device. 

```
# Check all rules, verbose option included:
$ iptables -L -v
```

Check the chain default behavior ( or when iptables has no rules for a packet ) can be checked via a simple grep pipe:

```
# Check all rules, verbose option included:
$ iptables -L | grep "policy"
```

### Modify Chain Default Behavior 

Change default policy for all chains to accept connections ( usually the default, not exactly secure ):

#### Default to ACCEPT connections.

```
# Change iptables input chain to accept connections by default:
$ iptables --policy INPUT ACCEPT

# Change iptables output chain to accept connections by default:
$ iptables --policy OUTPUT ACCEPT

# Change iptables forward chain to accept connections by default:
$ iptables --policy FORWARD ACCEPT
```

Change default policy for all chains to drop ( deny / not allow ) connections ( can break things ):

#### Default to DROP connections.

```
# Change iptables input chain to drop connections by default:
$ iptables --policy INPUT DROP

# Change iptables output chain to drop connections by default:
$ iptables --policy OUTPUT DROP

# Change iptables forward chain to drop connections by default:
$ iptables --policy FORWARD DROP
```

### Understanding Policy Responses 

Quickly we'll go over the top three used connection-specific responses for policies. 

#### ACCEPT

Will allow connections for the specific rule in a chain. 

#### DROP

Will drop connections for the specific rule in a chain acting like it never existed. Useful for stealthy purposes as an error wont be sent back to the originating host of the connection. ( Request timed out. )

#### REJECT

Will drop connections but will send back an error; and will probably indicate a firewall is being used. ( Destination port unreachable. ) 

### Rules 

Rules are what specify what to do with a packet. If the packet does not match, the next rule in the chain is applied to the packet; if it does match, then the next rule is specified by the value of the target, which can be the name of a user-defined chain or one of the special values ACCEPT, DROP, QUEUE, or RETURN.

### Targets

A rule specifies the criteria for a packet, and also a target. Packets are matched against rules consecutively until there is a match. When there is a match, the packet then the next rule is specified by the value of the target, which can be the name of a user-defined chain or one of the special values ACCEPT, DROP, QUEUE, or RETURN. 

---

## Leveraging to Your Advantage

The following are some examples of how to use iptables to your advantage in a competition situation. 

#### Syntax 

For learning iptables, I highly recommend you use the longer syntax variation to make things a little more human readable.

### Initial Setup 

It is recommended that you use the iptables setup script by Peter Pilarski [iptsetup](https://github.com/ppil/iptsetup) to setup your iptables configuration for a competition. 

To install [iptsetup](https://github.com/ppil/iptsetup) is very simple ( assuming git is possible ): 

```
$ git clone https://github.com/ppil/iptsetup
```

Or, quite possible you don't have git readily available to you ( but you should have some sort of host machine to work with ); and if you have a USB or some way of having the file on your host machine, and if SSH is possible on the host you want to secure ( example 192.168.4.10 ; assuming user is called "user" ... probably root ):

```
$ rsync -v -e ssh /path/to/local/iptsetup.sh user@192.168.4.10:~

# -e ssh : specify the ssh as remote shell
# -v     : verbose ( -vv for increased verbosity )
```

If neither of those two options are working, I've written [another guide](/path/to/L2R) about copying files stored locally to remote locations.  

### Rules

#### Block Single IP

You can easily block ( DROP ) all connections from a single IP address 192.168.0.1:

```
$ iptables --append INPUT --source 192.168.0.1 --jump DROP

	or 

$ iptables -A INPUT -s 192.168.0.1 -j DROP
```

Simply, for incoming connections ( hence, appenending to the INPUT chain ) where there is the a source IP of 192.168.0.1, the target of the rule is the built-in chain/special value DROP which drops the packet on the floor. 

#### Block IP Range

You may also find it helpful to be able to block connetion with a range of IP addresses. Iptables supports CIDR and netmask notation which allows you to easily specify a range of IPv4 addresses. To learn more about working with IP addresses you may want to check out [IpIn-Fu](https://github.com/picatz/IpIn-Fu) to get a little more comfortable.

So we can easily block ( DROP ) all connection from the IP address range 192.168.0.1/24 ( 192.168.0.0 - 192.168.0.255 ) :

```
$ iptables --append INPUT --source 192.168.0.1/24 --jump DROP

	or 

$ iptables -A INPUT -s 192.168.0.1/24 -j DROP
```

We can also use a netmask ( 255.255.255.0 ) instead of a CIDR range to effectively do the same thing, block ( DROP ) all connection from the IP address range 192.168.0.1/24:

```
$ iptables --append INPUT --source 192.168.0.1/255.255.255.0 --jump DROP

	or 

$ iptables -A INPUT -s 192.168.0.1/255.255.255.0 -j DROP
```

#### Block TCP Connections on Specific Port

In some situations, disabling SSH for an IP can be a fun / important thing to do to secure a server depending on the situation. This example shows how to block SSH connections from 192.168.0.1 on port 22. 

```
$ iptables --append INPUT --protocol tcp --destination-port 22 -s 192.168.0.1 -j DROP

	or 

$ iptables -A INPUT -p tcp --dport 22 -s 192.168.0.1 -j DROP
```

#### Allow TCP Connections on Specific Port 

Also an example on specifying the interface as eth0 in this case:

```
$ iptables --append INPUT --in-interface eth0 --protocol tcp --destination-port 22 --source 192.168.0.1 --match state --state NEW,ESTABLISHED --jump ACCEPT

	or 

$ iptables -A INPUT -i eth0 -p tcp --dport 22 -s 192.168.0.1 - state --state NEW,ESTABLISHED -j ACCEPT
```

#### Block UDP Connections on Specific Port from a range of IPs

If you're going to block someone's tcp traffic, maybe you want to block their udp traffic?

```
$ iptables --append INPUT --protocol udp --destination-port 22 -s 192.168.0.1/24 -j DROP

	or 

$ iptables -A INPUT -p udp --dport 22 -s 192.168.0.1 -j DROP
```

#### Allow UDP Connections on Specific Port for a specific IP 

Also an example on specifying the interface as eth0 in this case:

```
$ iptables --append INPUT --in-interface eth0 --protocol udp --destination-port 22 --source 192.168.0.1 --match state --state NEW,ESTABLISHED --jump ACCEPT

	or 

$ iptables -A INPUT -i eth0 -p udp --dport 22 -s 192.168.0.1 - state --state NEW,ESTABLISHED -j ACCEPT
```

#### Load Balance Incoming Web Traffic

You have to deal with a lot of web traffic? Iptables has lodabalancing. It relies on the ptables nth extension; and this example load balances the HTTPS traffic to three different ip-address ( 192.168.1.101, 192.168.1.102, 192.168.1.103 ). For every 3rd packet, it is load balanced to the appropriate server (using the counter 0).

```
$ iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 0 -j DNAT --to-destination 192.168.1.101:443
$ iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 1 -j DNAT --to-destination 192.168.1.102:443
$ iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 2 -j DNAT --to-destination 192.168.1.103:443
``` 

#### Allow Ping from Outside to Inside

The following rules allow outside users to be able to ping your servers.

```
$ iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
$ iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
```

#### Allow Ping from Inside to Outside

```
$ iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
$ iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
```

#### Allow Loopback Access 

Allow full loopback access on your servers. i.e access using 127.0.0.1

```
$ iptables -A INPUT -i lo -j ACCEPT
$ iptables -A OUTPUT -o lo -j ACCEPT
```

#### Allow Internal Network to External network.

On the firewall server where one ethernet card is connected to the external, and another ethernet card connected to the internal servers, use the following rules to allow internal network talk to external network.

In this example, eth1 is connected to external network (internet), and eth0 is connected to internal network (For example: 192.168.1.x).

```
$ iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```

#### Allow outbound DNS

```
$ iptables -A OUTPUT -p udp -o eth0 --dport 53 -j ACCEPT
$ iptables -A INPUT -p udp -i eth0 --sport 53 -j ACCEPT
```

#### Allow Rsync From a Specific Network

If we only wanted to restrict access to rsync to the 192.168.101.0/24 range. 

```
$ iptables -A INPUT -i eth0 -p tcp -s 192.168.101.0/24 --dport 873 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 873 -m state --state ESTABLISHED -j ACCEPT
```

#### Allow incoming HTTP

Allow incoming HTTP connections on port 80, using a specific interface. 

```
$ iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```


#### Allow MySQL connection only from a specific network

If you are running MySQL, typically you don’t want to allow direct connection from outside. In most cases, you might have web server running on the same server where the MySQL database runs.

However DBA and developers might need to login directly to the MySQL from their laptop and desktop using MySQL client. In those case, you might want to allow your internal network to talk to the MySQL directly as shown below.

```
$ iptables -A INPUT -i eth0 -p tcp -s 192.168.100.0/24 --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
```

#### Allow Sendmail or Postfix Traffic

The following rules allow mail traffic. It may be sendmail or postfix.

```
$ iptables -A INPUT -i eth0 -p tcp --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 25 -m state --state ESTABLISHED -j ACCEPT
```

#### Allow IMAP and IMAPS

The following rules allow IMAP/IMAP2 traffic:

```
$ iptables -A INPUT -i eth0 -p tcp --dport 143 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 143 -m state --state ESTABLISHED -j ACCEPT
```

The following rules allow IMAPS traffic:

```
$ iptables -A INPUT -i eth0 -p tcp --dport 993 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 993 -m state --state ESTABLISHED -j ACCEPT
```

#### Allow POP3 and POP3S

The following rules allow POP3 access.

```
$ iptables -A INPUT -i eth0 -p tcp --dport 110 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 110 -m state --state ESTABLISHED -j ACCEPT
```

The following rules allow POP3S access.

```
$ iptables -A INPUT -i eth0 -p tcp --dport 995 -m state --state NEW,ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -o eth0 -p tcp --sport 995 -m state --state ESTABLISHED -j ACCEPT
```

#### Prevent DoS Attack

The following iptables rule will help you prevent the Denial of Service (DoS) attack on your webserver.

```
$ iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

#         -m limit: This uses the limit iptables extension
# –limit 25/minute: This limits only maximum of 25 connection per minute. Change this value based on your specific requirement
# –limit-burst 100: This value indicates that the limit/minute will be enforced only after the total number of connection have reached the limit-burst level.
```

#### Port Forwarding

The following example routes all traffic that comes to the port 442 to 22. This means that the incoming ssh connection can come from both port 22 and 422.

```
$ iptables -t nat -A PREROUTING -p tcp -d 192.168.102.37 --dport 422 -j DNAT --to 192.168.102.37:22
```

If you do the above, you also need to explicitly allow incoming connection on the port 422.

iptables -A INPUT -i eth0 -p tcp --dport 422 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp --sport 422 -m state --state ESTABLISHED -j ACCEPT

#### Drop SYN-FIN TCP scan packets. 

```
$ iptables -t raw -A PREROUTING -p tcp --tcp-flags FIN,SYN FIN,SYN -j DROP
```

#### Drop SYN-RST TCP scan packets. 

```
$ iptables -t raw -A PREROUTING -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
```

#### Drop X-mas TCP scan packets. 

```
$ iptables -t raw -A PREROUTING -p tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG FIN,PSH,URG -j DROP
```

#### Drop FIN TCP scan packets. 

```
$ iptables -t raw -A PREROUTING -p tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG FIN -j DROP
```

#### Drop NULL TCP scan packets. 

```
$ iptables -t raw -A PREROUTING -p tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP
```

#### Drop shellshock attempts (packet contains "() {")

```
$ iptables -A INPUT -m string --algo bm --hex-string '|28 29 20 7B|' -j DROP
```

#### Drop invalid packets (not associated with a known connection)

```
$ iptables -A INPUT -m state --state INVALID -j DROP
$ iptables -A FORWARD -m state --state INVALID -j DROP
$ iptables -A OUTPUT -m state --state INVALID -j DROP
```

#### Default drop rule

```
$ iptables -A INPUT -j DROP
```

#### Raw table prerouting gets priority

```
$ iptables -t raw -A PREROUTING -m recent --name banned --rcheck --seconds 259200 -j DROP
```

#### Ban port 

I hate this port "$port" in particular.

```
# Drop anything in this list for 3 days
# Raw table prerouting gets priority, other rules be damned.
$ iptables -t raw -A PREROUTING -m recent --name banned --rcheck --seconds 259200 -j DROP

# Log the event, add the IP to this list
$ iptables -t raw -A PREROUTING -p tcp --dport $port -m recent --name banned --set -j LOG --log-prefix "Banned: " --log-level 7
$ iptables -t raw -A PREROUTING -p tcp --dport $port -m recent --name banned --set -j DROP 
```

#### Ping of Death

Discard After followed ping is 10 times more than once per second.

```
# Creates chain named named "PING_OF_DEATH"
$ iptables -N PING_OF_DEATH 
$ iptables -A PING_OF_DEATH -p icmp --icmp-type echo-request \
         -m hashlimit \
         --hashlimit 1/s \
         --hashlimit-burst 10 \
         --hashlimit-htable-expire 300000 \
         --hashlimit-mode srcip \
         --hashlimit-name t_PING_OF_DEATH \
         -j RETURN

# Discard the limit was exceeded ICMP 
$ iptables -A PING_OF_DEATH -j LOG --log-prefix "ping_of_death_attack: "
$ iptables -A PING_OF_DEATH -j DROP

# ICMP is "PING_OF_DEATH" jump in the chain
$ ptables -A INPUT -p icmp --icmp-type echo-request -j PING_OF_DEATH

```

#### SYN Flood Attack

Enabled Syn Cookie in addition to this measure .

Creates chain named named "SYN_FLOOD"

```
$ iptables -N SYN_FLOOD 
$ iptables -A SYN_FLOOD -p tcp --syn \
         -m hashlimit \
         --hashlimit 200/s \
         --hashlimit-burst 3 \
         --hashlimit-htable-expire 300000 \
         --hashlimit-mode srcip \
         --hashlimit-name t_SYN_FLOOD \
         -j RETURN
```

Discard the connections that exceeded the limit and log that info 

```
$ iptables -A SYN_FLOOD -j LOG --log-prefix "syn_flood_attack: "
iptables -A SYN_FLOOD -j DROP
```

Incoming SYN packets should jump to the SYN_FLOOD chain

```
$ iptables -A INPUT -p tcp --syn -j SYN_FLOOD
```

#### HTTP DoS/DDoS Attack

Creates chain named named "HTTP_DOS"

```
$ iptables -N HTTP_DOS 
$ iptables -A HTTP_DOS -p tcp -m multiport --dports $HTTP \
         -m hashlimit \
         --hashlimit 1/s \
         --hashlimit-burst 100 \
         --hashlimit-htable-expire 300000 \
         --hashlimit-mode srcip \
         --hashlimit-name t_HTTP_DOS \
         -j RETURN
```

Discard the connections that exceeded the limit and log that info 

```
$ iptables -A HTTP_DOS -j LOG --log-prefix "http_dos_attack: "
$ iptables -A HTTP_DOS -j DROP
```

Incoming HTTP packets ( port 80 ) should  jump to the HTTP_DOC chain

```
$ iptables -A INPUT -p tcp -m multiport --dports 80 -j HTTP_DOS
```

#### Reject Ident

Reject ident probes with a tcp reset. May need to do this for a broken mail host that won't accept mail if I just drop its ident probe.

```
$ iptables -A INPUT -p tcp --dport 113 -j REJECT --reject-with tcp-reset
```

#### Allow Telnet ( you hilarious kitten you )

Allow telnet outbound.

```
$ iptables -A INPUT  -p tcp --sport 23 -m state --state ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -p tcp --dport 23 -m state --state NEW,ESTABLISHED -j ACCEPT
```

#### Allow FTP ( you hilarious kitten you )

Allow ftp outbound.

```
$ iptables -A INPUT  -p tcp --sport 23 -m state --state ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -p tcp --dport 23 -m state --state NEW,ESTABLISHED -j ACCEPT
```

#### Allow www outbound to 80.

Allow www outbound to 80.

```
$ iptables -A INPUT -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
```

Allow www outbound to 443.

```
$ iptables -A INPUT -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
$ iptables -A OUTPUT -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
```

#### Mitigate SSH brute force attempts

If you are using password authentication for SSH, prepare for the password brute-force attacks. 

If there are 4 or more login attempts within 2 minutes, packets from that IP address get dropped.

```
iptables -N AUTOBAN
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j AUTOBAN
iptables -A AUTOBAN -m recent --set --name SSH
iptables -A AUTOBAN -m recent --update --seconds 120 --hitcount 4 --name SSH -j DROP
```

Log this information:

```
$ iptables -A AUTOBAN -j LOG --log-prefix "ssh_autobanned: "
```

If you login from a constant IP address or range, you could add a -s flag to exempt it from the rule. So something like:

```
$ iptables -A INPUT -p tcp -s ! 192.168.1.0/24 --dport 22 -m state --state NEW -j AUTOBAN
```

#### Mitigate FTP brute force attempts

If you are using password authentication for FTP, prepare for the password brute-force attacks. 

```
$ iptables -A INPUT -p tcp --syn -m multiport --dports $FTP -m recent --name ftp_attack --set
$ iptables -A INPUT -p tcp --syn -m multiport --dports $FTP -m recent --name ftp_attack --rcheck --seconds 60 --hitcount 5 -j LOG --log-prefix "ftp_brute_force: "
$ iptables -A INPUT -p tcp --syn -m multiport --dports $FTP -m recent --name ftp_attack --rcheck --seconds 60 --hitcount 5 -j REJECT --reject-with tcp-reset
```

#### Drop Broadcast

Drop packets going to ( broadcast, multicast ) addreses.

```
$ iptables -A INPUT -d 192.168.1.255   -j LOG --log-prefix "drop_broadcast: "
$ iptables -A INPUT -d 192.168.1.255   -j DROP
$ iptables -A INPUT -d 255.255.255.255 -j LOG --log-prefix "drop_broadcast: "
$ iptables -A INPUT -d 255.255.255.255 -j DROP
$ iptables -A INPUT -d 224.0.0.1       -j LOG --log-prefix "drop_broadcast: "
$ iptables -A INPUT -d 224.0.0.1       -j DROP
```

#### All communication once a session established is allowed.

```
$ iptables -A INPUT  -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT
```

#### Mitigate Stealth Scan

Create a chain with the name STEALTH_SCAN

```
$ iptables -N STEALTH_SCAN 
$ iptables -A STEALTH_SCAN -j LOG --log-prefix "stealth_scan_attack: "
$ iptables -A STEALTH_SCAN -j DROP
```

Incoming sneaky packets should jump to the STEALTH_SCAN chain.

```
$ iptables -A INPUT -p tcp --tcp-flags SYN,ACK SYN,ACK -m state --state NEW -j STEALTH_SCAN
$ iptables -A INPUT -p tcp --tcp-flags ALL NONE -j STEALTH_SCAN

$ iptables -A INPUT -p tcp --tcp-flags SYN,FIN SYN,FIN         -j STEALTH_SCAN
$ iptables -A INPUT -p tcp --tcp-flags SYN,RST SYN,RST         -j STEALTH_SCAN
$ iptables -A INPUT -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -j STEALTH_SCAN

$ iptables -A INPUT -p tcp --tcp-flags FIN,RST FIN,RST -j STEALTH_SCAN
$ iptables -A INPUT -p tcp --tcp-flags ACK,FIN FIN     -j STEALTH_SCAN
$ iptables -A INPUT -p tcp --tcp-flags ACK,PSH PSH     -j STEALTH_SCAN
$ iptables -A INPUT -p tcp --tcp-flags ACK,URG URG     -j STEALTH_SCAN
```

#### Prevent Fragmented Packets

Help prevent against port scan by fragment packet + DOS attacks. 

```
$ iptables -A INPUT -f -j LOG --log-prefix 'fragment_packet:'
$ iptables -A INPUT -f -j DROP
```


#### Flush current tables

Save past bans before flushing raw table.

```
cat /proc/net/xt_recent/banned 2>/dev/null | cut -d" " -f1 | cut -d= -f2 > ./banned_IPs_save.tmp
```

Flush current tables.

```
$ iptables -F
$ iptables -F -t raw
```


#### Flush pre-exisiting user defined chains

```
$ iptables -X
```

#### Flush pre-exisiting user defined chains

```
$ iptables -X
```

#### Zero Iptables counters

```
$ iptables -Z
```


### Logging

#### Defaults 

Iptables will use /var/log/messages to log things. If you want to change this to your own custom log file add the following line to /etc/syslog.conf:

```
kern.warning   /var/log/custom.log
```

#### Log All Dropped Packets (both Incoming and Outgoing)

```
$ iptables --new-chain LOGGING
$ iptables --append OUTPUT --jump LOGGING
$ iptables --append INPUT --jump LOGGING
$ iptables --append LOGGING -match limit --limit 2/min --jump LOG --log-prefix "IPTables-Dropped: " --log-level 7
$ iptables --append LOGGING --jump DROP
```

#### Seperate Logging

Any udp not already allowed is logged and then dropped.

```
$ iptables -A INPUT  -i $IFACE -p udp -j LOG --log-prefix "IPTABLES UDP-IN: "
$ iptables -A INPUT  -i $IFACE -p udp -j DROP
$ iptables -A OUTPUT -o $IFACE -p udp -j LOG --log-prefix "IPTABLES UDP-OUT: "
$ iptables -A OUTPUT -o $IFACE -p udp -j DROP
```

Any icmp not already allowed is logged and then dropped.

```
$ iptables -A INPUT  -i $IFACE -p icmp -j LOG --log-prefix "IPTABLES ICMP-IN: "
$ iptables -A INPUT  -i $IFACE -p icmp -j DROP
$ iptables -A OUTPUT -o $IFACE -p icmp -j LOG --log-prefix "IPTABLES ICMP-OUT: "
$ iptables -A OUTPUT -o $IFACE -p icmp -j DROP
```

Any tcp not already allowed is logged and then dropped.

```
$ iptables -A INPUT  -i $IFACE -p tcp -j LOG --log-prefix "IPTABLES TCP-IN: "
$ iptables -A INPUT  -i $IFACE -p tcp -j DROP
$ iptables -A OUTPUT -o $IFACE -p tcp -j LOG --log-prefix "IPTABLES TCP-OUT: "
$ iptables -A OUTPUT -o $IFACE -p tcp -j DROP
```

Anything else not already allowed is logged and then dropped. It will be dropped by the default policy anyway, but let's be paranoid.

```
$ iptables -A INPUT  -i $IFACE -j LOG --log-prefix "IPTABLES PROTOCOL-X-IN: "
$ iptables -A INPUT  -i $IFACE -j DROP
$ iptables -A OUTPUT -o $IFACE -j LOG --log-prefix "IPTABLES PROTOCOL-X-OUT: "
$ iptables -A OUTPUT -o $IFACE -j DROP
```

### Set aggressive default drop log prefix

Discard those that did not fit pre-defined rules with logging:

```
$ iptables -A INPUT  -j LOG --log-prefix "drop: "
$ iptables -A INPUT  -j DROP
```

### Save Changes

Ubuntu:

```
$ sudo /sbin/iptables-save
```

RHEL/CentOS:

```
$ sudo /sbin/iptables-save
```

Or: 
```
$ sudo/etc/init.d/iptables save
```

### List Current Rules:

```
$ iptables -L
```

### Save Current Rules:

```
$ iptables-save -c > /etc/iptables.rules
```

### Restore Saved Rules:

Assuming rules are saved in /etc/iptables.rules

```
$ iptables-restore < /etc/iptables.rules
```

---

## Helpful BASH Things.

### Get Current Established Connections

Works on linux and OSX. 

```
$ netstat -na|grep ESTABLISHED|awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -n -r
```

### Monitor Current Connections

Works on linux and OSX. 

```
$ watch netstat -ntla
```

### Drop Apache Resource Hogs

```
#!/bin/bash
# By ppabc
# github.com/ppabc/cc_iptables
num=100
cd /home/wwwlogs
# Read the latest 1000 records , if more than 100, let's DROP that connection.
for i in tail access.log -n 1000|awk '{print $1}'|sort|uniq -c|sort -rn|awk '{if ($1>$num){print $2}}'; do
	iptables -I INPUT -p tcp -s $i --dport 80 -j DROP
done
```

---

## Helpful links.

- [The Beginner’s Guide to iptables, the Linux Firewall](http://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/)
- [20 Iptables Examples](http://www.cyberciti.biz/tips/linux-iptables-examples.html)
- [How to Log Linux IPTables Firewall Dropped Packets to a Log File](http://www.thegeekstuff.com/2012/08/iptables-log-packets/)
- [25 Most Frequently Used Linux IPTables Rules Examples](http://www.thegeekstuff.com/2011/06/iptables-rules-examples/)
- [dns-iptables-rules](https://github.com/smurfmonitor/dns-iptables-rules)
- [iptables.sh](https://github.com/suin/iptables/blob/master/iptables.sh)
- [James Stephens Iptables Example Ruleset](http://www.sns.ias.edu/~jns/files/iptables_ruleset)
- [iptables_torify](https://github.com/ericpaulbishop/iptables_torify)
- [cc_iptables](https://github.com/ppabc/cc_iptables)
- [eamonnkillian's iptables](https://github.com/eamonnkillian/iptables)
- [log2iptables.sh](https://github.com/theMiddleBlue/log2iptables)
- [Spamhaus DROP List](https://github.com/cowgill/spamhaus)
