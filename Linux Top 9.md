# Linux

1. Kill networking and Firewall setup (iptables)
    a.	Service network stop
    b.	Ifconfig <interface-name> down
    c.	Iptables -F
    d.	Iptables –P INPUT DROP
    e.	Iptables –P OUTPUT ACCEPT
    f.	Iptables –P FORWARD DROP
    g.	Iptables –A INPUT –p tcp –s <ip-address of scoring engine> --dport <scored port number> -m state –state NEW,ESTABLISHED –j ACCEPT
  
2.	Rotate Creds and delete unnecessary accounts
    a.	Passwd <username>
    b.	Check users – cat /etc/passwd, /etc/shadow
    c.	Userdel –rf <username>

3.	What services and processes are running on the box and listening ports
    a.	Service –status-all 
    b.	Ps -aux
    c.	Ps –ef
    d.	Service <service-name> stop
    e.	Netstat –tulpan
    f.	Chkconfig --list

4.	Backup configs for scored service
    a.	Cp <file path> <new file path>

5.	Check environ for suspicious
    a.	Cron jobs
    b.	Vi ~/.bashrc
    c.	ENV
    d.	Echo $PATH
    e.	Vi /etc/rc.local

6.	Check service configs for vulns
    a.	~/.ssh/authorized_keys
    b.	/etc/dhcp/dhcpd.conf
    c.	/etc/named.conf or /var/named/

7.	Restart services and turn on interfaces
    a.	Service <service-name> restart
    b.	Ifconfig <interface-name> up

8.	Audit System
    a.	Github.com/ppil/nixaa.sh
    b.	Sticky Bits
    c.	Groups
    d.	Permissions
    e.	Inventory all executables or scripts

9.	Check for other bads
    a.	Netcat listener
