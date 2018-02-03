# Linux

1. Kill networking and Firewall setup (iptables)
    1.	`service network stop`
    2.	`ifconfig <interface-name> down`
    3.	`iptables -F`
    4.	`iptables –P INPUT DROP`
    5.	`iptables –P OUTPUT ACCEPT`
    6.	`iptables –P FORWARD DROP`
    7.	`iptables –A INPUT –p tcp –s <ip-address of scoring engine> --dport <scored port number> -m state –state NEW,ESTABLISHED –j ACCEPT`
  
2.	Rotate Creds and delete unnecessary accounts
    1.	`passwd <username>`
    2.	Check users – `cat /etc/passwd, /etc/shadow`
    3.	`userdel –rf <username>`

3.	What services and processes are running on the box and listening ports
    1.	`service –status-all`
    2.	`ps -aux`
    3.	`ps –ef`
    4.	`service <service-name> stop`
    5.	`sudo netstat –tulpan`
    6.	`chkconfig --list`

4.	Backup configs for scored service
    1.	`cp <file path> <new file path>`

5.	Check environ for suspicious
    1.	Cron jobs
    2.	`vi ~/.bashrc`
    3.	ENV
    4.	`echo $PATH`
    5.	`vi /etc/rc.local`

6.	Check service configs for vulns
    1.	~/.ssh/authorized_keys
    2.	/etc/dhcp/dhcpd.conf
    3.	/etc/named.conf or /var/named/

7.	Restart services and turn on interfaces
    1.	`service <service-name> restart`
    2.	`ifconfig <interface-name> up`

8.	Audit System
    1.	Github.com/ppil/nixaa.sh
    2.	Sticky Bits
    3.	Groups
    4.	Permissions
    5.	Inventory all executables or scripts

9.	Check for other bads
    1.	Netcat listener
