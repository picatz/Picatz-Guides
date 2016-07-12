# OpenSSH Guide

This is a somewhat comprehensive guide for OpenSSH a suite of tools that serve as alternatives to unencrypted network protocols such as FTP and rlogin via tuneeling communications through SSH. Before continuing with this guide, I highly reccomend you read the [ssh](http://man.openbsd.org/ssh) and [sshd](http://man.openbsd.org/OpenBSD-current/man8/sshd.8) man pages. 

---

#### OpenSSH Suite

```
Breakdown of OpenSSH Suite, visually: 

─── iptables
  ├── ssh
  │ └──> A replacement for rlogin, rsh and telnet to allow shell access to a remote machine.
  ├── scp 
  │ └──> A replacement for rcp.
  ├── sftp
  │ └──> A replacement for ftp to copy files between computers.
  ├── sshd
  │ └──> The SSH server daemon.
  ├── ssh-keygen
  │ └──>  A tool to inspect and generate keys that are used for user and host authentication.
  ├── ssh-agent + ssh-add
  │ └──> Utilities help ease authentication by holding keys ready. 
  └── ssh-keyscan
    └──> Scans a list of hosts and collects their public keys.
```

---

#### OpenSSH Config Files

```
Breakdown of OpenSSH Config, visually ( also actually files in this case ): 

─── ~
  └── .ssh/ ( hidden file )
    └── authorized_keys
      └──> Lists the public keys (RSA or DSA) that can be used to log into the user’s account.
─── /etc/
  ├── ssh/
  │ ├── sshd_config
  │ │ └──> OpenSSH server configuration file.
  │ └── ssh_config
  │   └──> OpenSSH client configuration file.
  ├── nologin 
  │ └──> If this file exists, sshd refuses to let anyone except root log in.
  ├── hosts.allow 
  │ └──> Access controls list ( to allow ) enforced by tcp-wrappers are defined here.
  └── hosts.deny
    └──> Access controls list ( to deny ) enforced by tcp-wrappers are defined here.
```

---

## Installing 

You man want to check if you have OpenSSH already installed ( or mabye even already running ) on your machiene. To do so is very simple: 

### Check Version

#### NetCat Check Port 22

By default, OpenSSH uses port 22. You can check quickly if a computer allows connections on this port with netcat. For example, we can check our localhost: 

```
$ nc -v -z 127.0.0.1 22
```

#### OpenSSH Client Version Check

Check which version is running of the OpenSSH client via -v option: 

```
$ ssh -v 
```

#### OpenSSH Daemon Version Check

Check which version is running of the OpenSSH daemon via -v option: 

```
$ sshd -v 
```

#### Debian

Check if openssh packages are installed via dpkg :

```
$ dpkg --list | grep openssh
```

#### RHEL/CentOS

Check if the openssh packages are installed via yum : 

```
$ yum list installed openssh\*
```

Hint : Before blindly running SSH, be sure to check your client a server configurations if possible. 

### Install OpenSSH Server

Do you actually need an SSH Server on the host? Ok. Well, this is how you do that:

#### Debian

Install on Debian:

```
$ sudo apt-get install openssh-server
```

Reinstall on Debian:

```
$ sudo apt-get --reinstall openssh-server
```

#### RHEL/CentOS

Install on RHEL/CentOS ( yum ):

```
$ sudo yum install openssh-server
```

Reinstall on RHEL/CentOS ( yum ):

```
$ sudo yum install openssh-server
```

### Install OpenSSH Client

Do you actually need an SSH client on the host? Ok, that makes a little more sense. 

#### Debian

Install on Debian:

```
$ sudo apt-get install openssh-client
```

Reinstall on Debian:

```
$ sudo apt-get --reinstall openssh-client
```

#### RHEL/CentOS

Install on RHEL/CentOS ( yum ):

```
$ sudo yum install openssh-client
```

Reinstall on RHEL/CentOS ( yum ):

```
$ sudo yum install openssh-client
```

### Check Running Status

#### PS + Grep

We can use the [ps](http://linux.die.net/man/1/ps) and [grep](http://linux.die.net/man/1/grep) command to check if SSH is running on our host:

```
$ ps -aux | grep ssh
```

#### Service

Sometimes we can use the [service](http://linux.die.net/man/8/service) command to check if SSH is running on our host. Service runs a System V init script in as predictable environment as possible, removing most environment variables and with current working directory set to /.

Check if OpenSSH Daemon status : 

##### Debian

```
$ sudo service ssh status
```

##### RHEL/CentOS

```
$ sudo service sshd status
```

Turn on OpenSSH Daemon :

##### Debian

```
$ sudo service ssh start
```

##### RHEL/CentOS

```
$ sudo service sshd start
```

Turn off OpenSSH Daemon :

##### Debian

```
$ sudo service ssh stop
```

##### RHEL/CentOS

```
$ sudo service sshd stop
```

#### init.d

/etc/init.d contains scripts used by the System V init tools (SysVinit).

Check if OpenSSH Daemon status : 

##### Debian

```
$ sudo /etc/init.d/ssh status
```

##### RHEL/CentOS

```
$ sudo /etc/init.d/sshd status
```

Turn on OpenSSH Daemon :

##### Debian

```
$ sudo /etc/init.d/ssh start
```

##### RHEL/CentOS

```
$ sudo /etc/init.d/sshd start
```

Turn off OpenSSH Daemon :

##### Debian

```
$ sudo /etc/init.d/ssh stop
```

##### RHEL/CentOS

```
$ sudo /etc/init.d/sshd stop
```

#### Chkconfig

Sometimes we can use the [chkconfig](http://linux.die.net/man/8/chkconfig) command to check if SSH is running on our host. The chkconfig command-line tool that allows you to specify in which runlevel to start a selected service, as well as to list all available services along with their current setting. It's is advised you ensure you know what you're doing before blindly turning things on with this. Well, for everything really. 

Check setting for the OpenSSH Daemon. 

```
$ sudo chkconfig --list sshd
```

Enable OpenSSH Daemon. 

```
$ sudo chkconfig sshd on
```

---

## Configuring OpenSSH Server

Configuration for you OpenSSH Server should come in roughly four phases:

- 1. You Install OpenSSH Server ( only if needed )
- 2. You Configure Daemon
- 4. You Add Banner ( optional )
- 4. You Restart Service

### Disabling OpenSSH Server

If you do not specifically need not to provide the remote login and file transfer capabilities of SSH to your host, please disable and remove the OpenSSH Server.

#### RHEL/CentOS

Disable sshd via chkconfig at boot time:

```
$ sudo chkconfig sshd off
```

Disable sshd via service command :

```
$ sudo service sshd stop
```

### Remove OpenSSH Server

```
$ sudo yum remove openssh-server
```

#### Debian/Ubuntu

Disable ssh daemon ( service referred to as simply "ssh" in this case ). 

```
$ sudo service ssh stop
```

### /etc/ssh/sshd_config

The following are configurations for your OpenSSH Server to help keep it secure. '/etc/ssh/sshd_config'; and the SSH service will have to be restarted to apply changes. 

OpenSSH server configuration guidelines / examples :

#### Only Use SSH Protocol 2

SSH protocol version 1 (SSH-1) has security vulnerabilities. Protocol version 1 should be considered obsolete and should be avoided at all cost. 

```
Protocol 2
```

#### AllowUsers SSH Access

We can only allow root, kent and jeff user to use the system via SSH, add the following to sshd_config:

```
AllowUsers root kent jeff
```

#### DenyUsers SSH Access

We can also use ssh to deny certain users ( kent, jeff, peter ) pretty simply:

```
DenyUsers root kent jeff
```

#### Configure Idle Log Out Timeout Interval

User can login to server via ssh and you can set an idle timeout interval to avoid unattended ssh session. Imagine how annoying this could be if it  was set to something really, really unreasonably low ( in this example it is 300 secs = 5 minutes ):

```
ClientAliveInterval 300
ClientAliveCountMax 0
```

#### Disable .rhosts Files

Don’t read the user’s ~/.rhosts and ~/.shosts files. But be sure to mess with them if you're red team. SSH can emulate the behavior of the obsolete rsh command. It is recommended to disable this.

```
IgnoreRhosts yes
RhostsAuthentication no
RhostsRSAAuthentication no
```

#### Disable host-based authentication

Host-based authentication in SSH is used when a user on a trusted host wants to log on to a remote machine, which could be an untrusted system. This method does not require a password. Blue team no likey. Red team likey. SSH's cryptographic host-based authentication is slightly more secure than .rhosts authentication, since hosts are cryptographically authenticated. However, it is not recommended that hosts unilaterally trust one another, even within an organization.

```
HostbasedAuthentication no
```

#### Disable root Login via SSH

There is ( almost ) no need to login as root via ssh over a network. Normal users can use su or sudo (recommended) to gain root level access. When this is the case it is also easier to track multiple users for when they do login in to a system and use root, we know which ones are doing what because it's been isolated more. 

```
PermitRootLogin no
```

#### Change Listening Port

Change the default listening port to something cool ( may have to specify port via ssh command to login though ): 

```
Port 31337
```

#### Use Public Key Based Authentication

Use public/private key pair with password protection for the private key.

##### How to Setup SSH Public Key Based Authentication

On your localhost ( probably the one not running OpenSSH Server ), you're going to want to generate your ssh keys. These come in two flavors: public and private. Public key is copied ot the remote server; and the private key will be used to identify your computer and grant access to the remote host. I recommend you set a pass phrase for your ssh keys despite not needing to. 

Generate ssh keys:
```
$ ssh-keygen -t rsa
```

Copy public key to remote host ( 192.313.3.7 ) :

```
$ ssh-copy-id user@192.313.3.7 
```

Alternative you can do so using ssh itself to do the same thing :

```
cat ~/.ssh/id_rsa.pub | ssh user@192.313.3.7  "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```

Be sure to set PermitRootLogin without a password as described below once your keys are setup properly:

#### Setup RSAAuthentication

Setup to use RSA authentication, to make sure the keys we generated work and everything in coherent. 

```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile %h/.ssh/authorized_keys
```
#### Ensure login with SHH key 

Set it so users can only login with SSH key by turning off PasswordAuthentication. 

```
PasswordAuthentication no
```

#### Turn of Root Login

Set it so users can only login with SSH key.

```
PermitRootLogin no
```

#### Change Login Grace Time (LoginGraceTime)

2 minutes time to login successfully is too much. You should consider changing it to 30 seconds, or may be 1 minute.

```
LoginGraceTime 1m
```

#### Use TCP Wrappers

TCP Wrapper is a host-based Networking ACL system, used to filter network access to Internet. OpenSSH does supports TCP wrappers. Just update your '/etc/hosts.allow' file as follows to allow SSH only from 192.168.1.2 172.16.23.12 :

```
sshd : 192.168.1.2 172.16.23.12 
```

#### Disable Empty Passwords

You need to explicitly disallow remote login from accounts with empty passwords, update sshd_config with the following line:

```
PermitEmptyPasswords no
```

#### Chroot SSHD (Lock Down Users To Their Home Directories)

By default users are allowed to browse the server directories such as /etc/, /bin and so on. You can protect ssh, using os based chroot since the release of OpenSSH 4.8p1 or 4.9p1. [Here is a guide to configure that.](https://debian-administration.org/article/590/OpenSSH_SFTP_chroot_with_ChrootDirectory)

#### Enable a Warning Banner

Create a file called '/etc/issue.net', and put in a warning banner. 

```
Banner /etc/issue.net
```

#### Do Not Allow Users to Set Environment Options

To prevent users from being able to present environment options to the SSH daemon and potentially bypass some access restrictions, add or correct the following line:

```
PermitUserEnvironment no
```

#### Show last login

If you want to make sure it shows who logged last when a user logins.

```
PrintLastLog yes
```

#### Restrict SSH Access by IP

If you want to allow SSH connection to be accepted from specific IP addresses, you can add the ListenAddress:

```
PrintLastLog yes
```

#### Disable UseDNS

This might create a latency between the client and the server when trying to establish the connection.

```
UseDNS no
```

#### Set a login grace timeout

This might create a latency between the client and the server when trying to establish the connection.

```
LoginGraceTime 300
```

#### Set maximum startup connections

Specifies the maximum number of concurrent unauthenticated connections to the SSH daemon. This setting can be helpful against a brute-force script that performs forking.

```
MaxStartups 2
```

#### Disable Forwarding

Specifies the maximum number of concurrent unauthenticated connections to the SSH daemon. This setting can be helpful against a brute-force script that performs forking.

```
AllowTcpForwarding no
X11Forwarding no
```

#### Log More Information

By default, OpenSSH logs everything at the INFO level. If you want to record more information like failed login attempts, you can change the value of this to VERBOSE.

```
LogLevel VERBOSE
```

#### Strict Mode

Prevent the use of insecure home directory and key file permissions.

```
StrictModes yes
```

#### Super Strict Authentication

Automatically block connections after 10 attempts to authenticate. Be careful with this! But it's hilarious for red team :)

```
MaxAuthTries 10
```

#### Syslog facility code for logging SSH messages:

Setup authentication related messages so that they are logged and authorization related commands.

```
SyslogFacility AUTH
```

#### Print the Message of the Day

This ensures the OpenSSH daemon should print the contents of the /etc/motd file when a user logs in interactively.

```
PrintMotd yes
```

#### Privilege Separation

Separates privileges by creating an unprivileged child process to deal with incoming network traffic.The goal of privilege separation is to prevent privilege escalation by containing any corruption within the unprivileged processes.  The argument must be ``yes'', ``no'', or ``sandbox''.  If UsePrivilegeSeparation is set to ``sandbox'' then the pre-authentication unprivileged process is subject to additional restrictions.  The default is ``sandbox''.

```
UsePrivilegeSeparation sandbox
```

#### Disable Tunneling

Specifies that [tun(4)](https://www.freebsd.org/cgi/man.cgi?query=tun&sektion=4&apropos=0&manpath=FreeBSD+10.3-RELEASE+and+Ports) device forwarding is not allowed. May need to be enabled to setup port forwarding, or left disabled to disable port forwarding. 

```
PermitTunnel no
```

#### Pid File

Specifies the file  that contains the process ID of the SSH daemon, or ``none'' to not write one. The default is /var/run/sshd.pid.

```
PidFile /var/run/sshd.pid
```

#### Compression

Specifies whether compression is allowed, or delayed until  the user has authenticated successfully.  The argument must be ``yes'', ``delayed'', or ``no''.  The default is ``delayed''.

```
Compression delayed
```

#### FingerprintHash

Specifies the hash  algorithm used when logging key fingerprints. Valid options are: ``md5'' and ``sha256''.  The default is ``sha256''.

```
FingerprintHash sha256
```

#### MACs

Specifies the available MAC (message authentication code) algorithms.  The MAC algorithm is used for data integrity protection. Multiple algorithms must be comma-separated.  If the specified value begins with a `+' character, then the specified algorithms will be appended to the default set instead of replacing them.

```
MACs hmac-sha2-512 
```

#### MaxStartups

Specifies the maximum number of concurrent  unauthenticated connections to the SSH daemon.  Additional connections will be dropped until authentication succeeds or the LoginGraceTime expires for a connection.  The default is 10:30:100. Be careful as this could potentially lock yourself out.

#### PubkeyAcceptedKeyTypes

Specifies the key types that will be accepted for public key authentication as a comma-separated pattern list.  Alternately if the specified value begins with a `+' character, then the specified key types will be appended to the default set instead of replacing them.  The default for this option is:

```
PubkeyAcceptedKeyTypes ssh-rsa
```

#### Log SFTP

Log sftp level file access (read/write/etc.) that would not be easily logged otherwise.

```
Subsystem sftp  /usr/lib/ssh/sftp-server -f AUTHPRIV -l INFO
```

#### Ciphers

Specifies the ciphers allowed.  Multiple ciphers must be comma-separated.  If the specified value begins with a `+' character, then the specified ciphers will be appended to the default set instead of replacing them.

```
Ciphers aes256-ctr
```

---

## Configuring OpenSSH Client

Configuration for you OpenSSH Server should come in roughly four phases:

- 1. You Install OpenSSH Client
- 2. You Configure Client
- 3. You Restart Any Connections using Client

### /etc/ssh/ssh_config

The following are configurations for your OpenSSH Client to help keep it secure / make your life easier. '/etc/ssh/ssh_config'; and the SSH connection may will have to be restarted to apply changes. 

#### Restrict to IPv4 Only

Specifies which address family to use when  connecting. Valid arguments are ``any'', ``inet'' (use IPv4 only), or ``inet6'' (use IPv6 only).  The default is ``any''.

```
AddressFamily inet
```

#### CanonicalizeMaxDots

Specifies the maximum number of dot characters in a hostname before canonicalization is disabled.  The default, ``1'', allows a single dot (i.e. hostname.subdomain).

```
CanonicalizeMaxDots 1
```

#### CheckHostIP

If this flag is set to ``yes'', ssh(1) will additionally check the host IP address in the known_hosts file. This allows ssh to detect if a host key changed due to DNS spoofing and will add addresses of destination hosts to ~/.ssh/known_hosts in the process, regardless of the setting of StrictHostKeyChecking. If the option is set to ``no'', the check will not be executed. The default is ``no''.

```
CheckHostIP yes
```

#### Ciphers

Specifies the ciphers allowed for protocol  version 2 in order of preference.  Multiple ciphers must be comma-separated. If the specified value begins with a `+' character, then the specified ciphers will be appended to the default set instead of replacing them.

```
Ciphers aes256-ctr
```

#### Ciphers

Specifies whether to use compression.  The  argument must be ``yes'' or ``no''.  The default is ``no''.

```
Compression yes
```

#### CompressionLevel

Specifies the compression level to  use if compression is enabled. The argument must be an integer from 1 (fast) to 9 (slow, best). The default level is 6, which is good for most applications. The meaning of the values is the same as in gzip(1). Note that this option applies to protocol version 1 only.

```
CompressionLevel 5
```

#### ConnectionAttempts

Specifies the number of tries (one  per second) to make before exiting. The argument must be an integer. This may be useful in scripts if the connection sometimes fails.  The default is 1.

```
ConnectionAttempts 1
```

#### LogLevel

Gives the verbosity level that is used when logging messages from the ssh client. 

```
LogLevel VERBOSE
```

#### MACs

Specifies the available MAC (message authentication code) algorithms.  The MAC algorithm is used for data integrity protection. Multiple algorithms must be comma-separated.  If the specified value begins with a `+' character, then the specified algorithms will be appended to the default set instead of replacing them.

```
MACs hmac-sha2-512 
```

#### NumberOfPasswordPrompts

Specifies the number of password prompts before giving up. The argument to this keyword must be an integer. The default is 3.

```
NumberOfPasswordPrompts 3
```

#### NumberOfPasswordPrompts

Specifies the number of password prompts before giving up. The argument to this keyword must be an integer. The default is 3.

```
NumberOfPasswordPrompts 3
```

#### PasswordAuthentication

Specifies whether to use password authentication. The argument to this keyword must be ``yes'' or ``no''.  The default is ``yes''. Can be funny to set this to no for red team if people don't have ssh keys setup. 

```
PasswordAuthentication yes
```

#### PreferredAuthentications

Specifies the order in which the client should try authentication methods. 

```
PreferredAuthentications publickey,hostbased,password,hostbased,keyboard-interactive
```





---

## Helpful Links

- [How to Install and Configure OpenSSH Server In Linux](http://www.tecmint.com/install-openssh-server-in-linux/)
- [SSH Public Key Based Authentication](http://www.cyberciti.biz/tips/ssh-public-key-based-authentication-how-to.html)
- [Debian Linux Stop SSH User Hacking / Cracking Attacks with DenyHosts Software](http://www.cyberciti.biz/faq/block-ssh-attacks-with-denyhosts/)
- [Red Hat / Centos Install Denyhosts To Block SSH Attacks / Hacking](http://www.cyberciti.biz/faq/rhel-linux-block-ssh-dictionary-brute-force-attacks/)
- [Security/Guidelines/OpenSSH](https://wiki.mozilla.org/Security/Guidelines/OpenSSH)