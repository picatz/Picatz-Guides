# L2R - Local to Remote 

This is a somewhat comprehensive guide on how to copy files locally stored on your machine to a remote server. 

---

Remote server's IP for this guide : 192.168.4.10
User for this guide : user 

---

### SSH

A lot of these tools kind of depend on SSH to work on the remote host; and also have a way to access SSH on the local host. In general I recommend OpenSSH. If you're not familiar with ssh, please lookup ssh and learn more about it before blindly doing things if possible. I dunno.   

### Install OpenSSH

Please be sure to configure SSH properly. 

#### Debian
```
# Install openssh-server:
$ sudo apt-get install openssh-server

# Install openssh-client:
$ sudo apt-get install openssh-client

# Install openssh-sftp-server:
$ sudo apt-get install openssh-sftp-server
```

#### RHEL/CentOS 
```
# Install openssh-server:
$ sudo yum install openssh-server

# Install openssh-client:
$ sudo yum install openssh-client

# Install openssh-sftp-server:
$ sudo apt-get install openssh-sftp-server
```

#### Windows 

I'm sorry... I recommend: 
[MobaXterm](http://mobaxterm.mobatek.net/)
[Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
[Cygwin](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
[OpenSSH for Windows](https://winscp.net/eng/docs/guide_windows_openssh_server)

---

## RSYNC 

It is super trivial to copy local files ( like /path/to/local/file.txt ) to a remote host ( 192.168.4.10 ). 

```
$ rsync -v -e ssh /path/to/local/file.txt user@192.168.4.10:~

# -e ssh : specify the ssh as remote shell
# -v     : verbose ( -vv for increased verbosity )

# ~      : path to store file om remote server
```

### Install Rsync

#### Debian
```
$ sudo apt-get install rsync
```

#### RHEL/CentOS 
```
$ sudo yum install rsync
```

---

## SFTP

If rsync is not an option, the next best bet would be SFTP ( SSH File Transfer Protocol ) -- SSH must be working for this to work. 

```
# Connect to remote host:
$ sftp user@192.168.4.10

# For help with sftp commands: 
sftp> help 

# To copy a file:
# Change to path you want to put local file:
sftp> cd /path/to/a/dir/on/remote/host

# Put local file there. 
sftp> put /path/to/local/file.txt

# Exit session:
sftp> exit 
```

---

## SCP

Similar to other concepts, this tool allows us to copy files securely using SSH. Much better than FTP.

```
# Copy file from the local host to a remote host
$ scp /path/to/local/file.txt user@192.168.4.10:/some/remote/directory
```

There are also a bunch of useful feature for this tool, here a few of them:

```
# Copy file from the remote host to the local host ( reverse of this guide ):
scp user@192.168.4.10:/path/to/remote/file.txt /path/to/local/directory

# Copy the directory "/path/to/local/directory" from the local host to a remote host's directory "/some/remote/directory":
scp -r /path/to/local/directory user@192.168.4.10:/some/remote/directory

# Specify a specific port to use ( 31337 ) :
scp -P 31337 /path/to/remote/file.txt user@192.168.4.10:/some/remote/directory

# scp uses the Triple-DES cipher to encrypt the data
# it is also possible ( and possibly faster ) to use Blowfish
scp -c blowfish /path/to/local/file.txt user@192.168.4.10:/some/remote/directory
```

---

## FTP

If sftp isn't an option, and scp ins't an option, and rsync isn't an option... and for some reason you're running normal FTP. Well, I guess do that: 

```
# Connect to remote host:
$ ftp 192.168.4.10

# Let ftp do its ftp things and then login using legit creds.:
Connecting to 192.168.4.10...
192.168.4.10 FTP server ready.
Name: user
Password:

# For help with ftp commands: 
ftp> help 

# To copy a file:
# Change to path you want to put local file:
ftp> cd /path/to/a/dir/on/remote/host

# Put local file there. 
ftp> put /path/to/local/file.txt

# Exit session:
ftp> exit 
```

### Install FTP

#### Debian
```
$ sudo apt-get install ftp
```

#### RHEL/CentOS 
```
$ sudo yum install ftp
```

---

## Git 

Do you have git for some reason? That's pretty cool. Let's install [Honey Cat](https://github.com/picatz/Honey-Cat) for an example. 

```
$ git clone https://github.com/picatz/Honey-Cat
```

## Curl + Github Raw

Do you at least have some sort access to the Internet? Is your local file you want to copy possibly on github too? For example, if you wanted to install [Honey Cat](https://github.com/picatz/Honey-Cat) using github raw. 

```
curl https://raw.githubusercontent.com/picatz/Honey-Cat/master/hcat.sh >> /path/to/local/hcat.sh
```

### Install Git

#### Debian
```
$ sudo apt-get install git-all
```

#### RHEL/CentOS 
```
$ sudo yum install git-all
