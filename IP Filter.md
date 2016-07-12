Solaris IP Filter
=================

Open source IP Filter terms
---------------------------
/usr/lib/lpf/IPFILTER.LICENCE

Managed by the SMF services: svc:/network/pfil and svc:/network/ipfilter

Provides protection against network-based attacks -> Filters by:

-IP address
-Port
-Protocol
-Network interface
-Traffic direction
-Individual source IP address
-Destination IP address
-Range of IP addresses
-Address pools

-Network Address Translation: Translation of a private IP address to a different public IP address, muliple private addresses to a single public one.

-IP accounting: Input and output rules can be set up, recording number of bytes that pass through. Each time a rule match occurs, the byte count of
 the packet is added to the rule and allows for the collection of statistics

-Fragment Cache Check: Fragmented packets are cached by default. If "set defrag off" appears in the rules file. Fragments not cached.

-Packet State Check: If "keep state" is included in a rule, all packets in a specified session are passed or blocked automatically

-Firewall Check: Input/Output rules separately set up. Packet is allowed through IP Filter, into kerne's TCP/IP routines, or out onto the network.

-Groups: Write your rule set in a tree fashion.

-Function: Action to be taken. Block, pass, literal, and send ICMP response.

-Fast-route: Signals IP Filter to not pass the packet into the UNIX IP stack for routing. Results in a TTL decrement.

-IP Authentication: Only passed throught he firewall loops once to prevent double-processing.

IP Filter includes a directory called "/etc/ipf
 * ipf.conf
 * ipnat.conf
 * ippool.conf

________________________________________________________________________________________________________________________________

EDITING SOLARIS IP FILTER CONFIGURATION FILES
---------------------------------------------
To create a configuration file for packet filtering rules edit the "ipf.conf" file. /etc/ipf/ipf.conf
 * Uses the packet filtering rules that you put in to the ipf.conf file.
 * If you do not want the filtering rules loaded at boot-time, put the ipf.conf file in a location of your choice then activate with
 
"ipf" command

--If the ipf.conf file is empty this means that the filtering rules are set as:
pass in all
pass out all

To create a configuration file for NAT rules edit the "ipnat.conf" file. /etc/ipf/ipnat.conf
If you do not want the NAT rules loaded at boot-time, put the ipnat.conf file in a location of your choice 
  then activate with "ipnat" command

To create a configuration file for address pools edit the "ippool.conf" file. /etc/ipf/ippool.conf
If you do not want the pool of addresses loaded at boot-time, put the ippool.conf file in a location of your choice 
  then activate with "ippool" command

________________________________________________________________________________________________________________________________

Example: IP FILTER HOST CONFIGURATION
-------------------------------------

 *#* pass and log everything by default
pass in log on elx10 all
pass out log on elx10 all

 *#* block, but don't log, incoming packets from other reserved addresses
block in quick on elx10 from 10.0.0.0/8 to any
block in quick on elx10 from 172.16.0.0/12 to any

 *#* block and log untrusted internal IPs. 0/32 is notation that replaces
 *#* address of the machine running Solaris IP Filter.
block in log quick from 192.168.1.15 to <thishost>
block in log quick from 192.168.1.43 to <thishost>

 *#* block and log X11 (port 6000) and remote procedure call
 *#* and portmapper (port 111) attempts
block in log quick on elx10 proto tcp from any to elx10/32 port = 6000 keep state
block in log quick on elx10 proto tcp/udp from any to elx10/32 port = 111 keep state
___________________________________________________________________________________
1. Allow everything to pass into and out of the elxl interface.
2. Blocks any incoming packets from the private address spaces 10.0.0.0 and 172.16.0.0 from entering the firewall.
3. Blocks specific internal addresses form the host machine.
4. Blocks packets coming in on port 6000 and port 1111.


Configuring Packet Filtering Rules
----------------------------------

action [in|out] option keyword, keyword...

 * Each rule begins with an action. Applies action to the packet if the packet matches the rule.

###block

	Prevents the packet form passing through the filter.
	
###pass

	Allows the packet through the filter.
	
###log

	Logs the packet but does not determine if the packet is blocked or passed. Use "ipmon" command to view the log.
	
###count

	Includes the packet in the filter statistics. Use "ipfstat" command to view the statistics.
	
###skip {number}

	Makes the filter skip over {number} filtering rules.
	
###auth

	Requests that packet authentication be performed by a user program that validates packet information. 
	
###preauth

	Requests that the filter look at a pre-authenticated list to determine what to do with the packet.

 * Next word must be "in" or "out." Determines whether the packet filtering rule is applied to an incoming packet or an outgoing packet.

 * Choose from a list of options. If you use more than one option, they must be in the order shown:

###log

	Logs the packet if the rule is the last matching rule. "ipmon" command to view the log.
###quick

	Executes the rule containing the "quick" option if there is a packet match. Further rule checking stops.
	
###on {interface name}

	Applies the rule only if the packet is moving in or out of the specified interface.
	
###dup-to {interface-name}

	Copies the packet and sends the duplicate out on {interface name} to an optionally specified IP address
###to {interface name}

	Moves the packet to an outbound queue on {interface name}

 * After specifying the options, you can choose from a list of keywords that determine whether the packet matches the rule.

Any packet that does not match any rule in the configuration file is passed through the filter.

###tos

	Filters the packet based on the type-of-service value expressed as either a hexadecimal or decimal integer.
###ttl

	Matches the packet based on its time-to-live value. The time-to-live value stored in a packet indicates the length of time 
	a packet can be on the network before being discarded.
###proto

	Matches a specific protocol. You can use any of the procol name specified in the "/etc/protocols" file, or use a decimal
	number to represent the protocol. The keywork tcp/udp can be used to match either a TCP or a UDP packet.
	
###from/to/all/any

	Matches any or all of the following: the source IP address, and the port number. The "all" keyword is used to accept
	packets from all sources and to all destinations.
###with

	Matches specified attributes associated with the packet. Insert either the word "not" or the word "no" in front of the
	keyword in order to match the packet only if the option is not present.
###flags

	Used for TCP to filter based on TCP flags that are set. For more info. on the TCP flags, see the ipf(4) man page.
	
###icmp-type

	Filters according to ICMP type. Used only when the "proto" option is set to "icmp" and is not used if the "flags"
	option is used.
	
###keep keep-options

	Determines the info. that is kept for a packet. Include the "state" option and the "frags" option. "state" keeps
	information on packet fragments and applies the information to later fragments. "keep-options" allow matching
	packets to pass without going through the access control list.
	
###head {number}

	Creates a new group for filtering rules, denoted by the number {number}.
	
###group {number}

	Adds the rule to group number {number} instead of the default group. Placed in group 0 if no other group is specified.

EX: Block incoming traffic from the IP address 192.168.0.0/16
block in quick from 192.168.0.0/16 to any
________________________________________________________________________________________________________________________________
IP Filter's Nat Feature
-----------------------

NAT sets up mapping rules that translate source and destination IP addresses into other Internet or intranet addresses. These rules
Modify the source and destination addresses of incoming or outgoing IP packets and send the packets on. Use NAT to redirect traffic
from one port to another port.

Use "ipnat" command to work with NAT rule list.
You can create NAT rules either at the command line using the "ipnat" command or in the NAT config. file. 
"/etc/ipf/ipnat.conf"

 * Each rule begins with the following commands:

###map

	Maps one IP address or network to another IP address or network in an unregulated round-robin process.
	
###rdr

	Redirects packets from one IP address and port pair to another IP address and port pair.
	
###bimap

	Establishes a bidirectional NAT between an external IP address and an internal IP address.
	
###map-block

	Establishes static IP-address-based translation. Based on an algorithm that forces addresses to be translation
	into a destination range.

 * Following the command, the next word is the interface name, such as "hme0"

 * Next, you can choose form a variety of parameters, which determine the NAT configuration:

###ipmask

	Designates the network mask
	
###dstipmask

	Designates the address that ipmask is translated to
	
###mapport

	Designates "tcp, udp" or "tcp/udp" protocols, along with a range of port numbers.

EX: Rewrite a packet that goes out on the "de0" device with a source address of 192.168.1.0/24 and to externally show its source
address as 10.1.0.0/16
map de0 192.168.1.0/24 -> 10.1.0.0/16

________________________________________________________________________________________________________________________________

IP Filter's Address Pools Feature
---------------------------------

Address pools establish a single reference that is used to name a group of address/netmask pairs. Provide processes to reduce the time
needed to math IP addresses with rules.

Reside with the ippool.conf file
Manually activate packet filtering with the ippool command

Configuring Address Pools

table role = {role-name} type = {storage-format} number = {reference-number}

	Specifies the reference number that is used by the filtering rule.

*#* ipfstat -io
empty list for ipfilter(out)
block in from pool/13(!) to any

Even if you add the pool later, the addition of the pool does not update the kernel rule set. You also need to reload the rules
	file that references the pool. Check "ippool(4)" man page for more info.

________________________________________________________________________________________________________________________________

Solaris 10 7/07 release:
-----------------------

Packet filter hooks replace the "pfil" module to enable Oracle Soalris IP filter. The use of packet filter hooks streamlines the procedure
	to enable Oracle Solaris IP filter. Through these hooks, Oracle Solaris IP Filter uses pre-reouting (input) and post-routing (output)
	filter taps to control packet flow into and out of the Oracle Solaris system.

Packet filter hooks eliminate the need for the "pfil" module. The following have been removed:
 * pfil driver
 * pfil daemon
 * svc:/network/pfil SMF service

________________________________________________________________________________________________________________________________

MAN PAGES
---------

###ipf(1M)

	Use the ipf command to complete the following tasks:
		Work with packet filtering rule sets.
		Disable and enable filtering.
		Reset statistics and resynchronize the in-kernel interface list with the current interface list.
###ipf(4)

	Contains the grammar and syntax for creating Oracle Solaris IP filter packet filtering rules.
	
###ipfilter(5)

	Provides open source IP Filter licensing information.
	
###ipfs(1M) 

	Use the "ipfs" command to save and restore NAT information and satate table information across reboots.
	
###ipfstat(1M)

	Use the "ipfstat" command to retrieve and display statistics on packet processing.
	
###ipmon(1M) 

	Use the "ipmon" command to open the log device and view logged packets for both packet filtering and NAT.
	
###ipnat(1M)

	Use the "ipnat" command to complete the following tasks:
		Work with NAT rules.
		Retrieve and display NAT statistics.
###ipnat(4)

	Contains the grammar and syntax for creating NAT rules.
	
###ippool(1M)

	Use the "ippool" command to create and manage address pools.
	
###ippool(4)

	Contains the grammar and syntax for creating Oracle Solaris IP Filter address pools.
	
###ndd(1M) 

	Displays current filtering parameters of the "pfil" STREAMS module and the current values of the tuanble paramaters.


 "It was not possible to think except with ones brain, no one could stand outside himself in order to check the functioning of his inner processes."
 	Stanislaw Lem, Solaris
