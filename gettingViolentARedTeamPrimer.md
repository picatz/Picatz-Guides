# Getting Violent - A Red Team Primer

This is a somewhat comprehensive guide on red team activities and where to use them 

---

- Attackers IP : 192.168.1.20
- Attackers reverse shell port : 4444
- Targets IP : 192.168.1.10
- User for this guide : user

---

#### Visual Representation

```
Breakdown of logic for this guide 

--- You are here
	+-- Ports to look for
	¦	+--> 7 Echo
	¦	¦	+--> Rules
	¦	+--> 20 - 21 FTP
	¦	¦	+--> Rules
	¦	+--> 22 SSH
	¦	¦	+--> Rules
	¦	+--> 23 Telnet
	¦	¦	+--> Rules
	¦	+--> 53 DNS
	¦	¦	+--> Rules
	¦	+--> 57 / 68 DHCP / BOOTP
	¦	¦	+--> Rules
	¦	+--> 80 HTTP
	¦	¦	+--> Rules
	¦	+--> 389 LDAP
	¦	¦	+--> Rules
	¦	+--> 443 HTTP over SSL
	¦	¦	+--> Rules
	¦	+--> 445 Microsoft DS
	¦	¦	+--> Rules
	¦	+--> 902 VMware Server
	¦	¦	+--> Rules
	¦	+--> 1080 SOCKS Proxy
	¦	¦	+--> Rules
	¦	+--> 1194 OpenVPN
	¦	¦	+--> Rules	
	¦	+--> 1337 WASTE
	¦	¦	+--> Rules
	¦	+--> 1080 SOCKS Proxy
	¦	¦	+--> Rules
	¦	+--> 5432 PostgreSQL
	¦	¦	+--> Rules
	¦	+--> 5800 VNC over HTTP
	¦	¦	+--> Rules
	¦	+--> 5900+ VNC Server
	¦	¦	+--> Rules
	¦	+--> 6665 - 6669 IRC
	¦	¦	+--> Rules	
	¦	+--> 6679/6697 IRC over SSL
	¦	¦	+--> Rules
	¦	+--> 8080 HTTP Proxy
	¦		+--> Rules
	+-- Basic reverse shells
	¦	+--> Bash
	¦	¦
	¦	+--> Perl
	¦	¦
	¦	+--> Python
	¦	¦
	¦	+--> PHP
	¦	¦
	¦	+--> Ruby
	¦	¦
	¦	+--> Netcat
	¦	¦
	¦	+--> Java
	¦	¦
		+--> xterm

		Before we begin into the service attacks please know that these ports are not set in stone. An example of this is running a HTTP server on port 1 if you so pleased. 
```