---
layout: post
title:  "Things learned while studying for RHCSA"
date:   2019-10-7
categories: linux
---

I signed up to take the RHCSA exam the Friday after next. 
Let's see how much I can learn before in 10 days.
I'll be really busy the week of the exam, so I better make this week count. 
This is my second attempt at the exam, a few years later. 
It turns out administrating a system is a bit more complicated than goofing around as a hobbiest.
Systemd kinda threw everything I learned playing with Arch and Gentoo out the window. 

Some of these notes will be really spare for things I (mostly) already know. If you see a paragraph, its for something I needed a refresher on.

I'm taking a spaced repetition approach. Going to try to read all the way through first without doing labs and exercises, then going back and doing all the labs. This will freshen it my mind and help reinforce the "finger thinking" for the commands and tasks. But time is running short.

# Command line basics

`! string` shell builtin expands to last command beginning with `string`.

You can get a list of previous commands with `history`, then `! number` with a number from the list to re-execute that command.

`Alt+.` expands to last word (usually argument) of previous command. Repeat it to go through earlier commands. 

`Ctrl+U` clear from cursor back to start of command line.

# File basics

`/bin`, `/sbin`, `/lib`, and `/lib64` are now symlinks to `/usr/bin`, `/usr/sbin`, `/usr/lib`, and `/usr/lib64`

`/var/run` and `/var/lock` are now combined under `/run`

For `/tmp` everything not touched in 10 days is deleted.
For `/var/tmp`, 30 days. 

## Hard links

Create a hard link with `ln source_file_name new_file_name`

Every file starts with single "hard link" mapping its file path to the real data in the file system. 
New hard links are new file paths pointing to same data. 
Each hard link is effectively the same, no preference to "original" file path. 
Deleting hard links does not remove data until ALL hard links are gone. 
Files have unique inode numbers so you can tell if two file names refer to the same file by `ls -i`. 
Doing `ls -l` will also list the link count for a file. 
Hard linked files share all link count, access permission, ownerships, timestamps and of course content.

Hardlinks must be in same file system. You cannot create a hard link for a directory or special file, just regular files. 

Example use case: an admin creating a read only file in all users home directory which they can update as needed. 

## Soft links

Create a soft link with `ln -s source_file_name new_file_name`

Soft links can point to files across file systems and can point to directories and special files.

Exampe use case: You have a 2nd hard drive just for storing large video files so you create a symbolic link from `/home/bart/Videos` to `/dev/sdb1/Videos`.  

`ls -l` will show the first letter of the long listing to be `l` to denote link. (`d` would be for directory, `-` would be for normal files).

Soft links only point to a name, so if you delete the original file then the link with be "dangling". Creating a new file with the same name as the original file would "re-attach" the dangling link. 

## Filename tricks

You can glob with more than just `*`.

```
`?` for any single character.
`[ abc... ]` for any one character in the brackets
`[^ abc... ]` for any character NOT in the brackets
`[! abc... ]` for any character NOT in the brackets
```

### Brace Expansion

Braces let you create sequence expansions.

```
$ echo {Homer,Marge,Bart,Lisa,Maggie}.log
Homer.log Marge.log Bart.log Lisa.log Maggie.log

$ echo moe{1..3}.txt
moe1.txt moe2.txt moe3.txt

$ echo barney{A..C}.txt
barneyA.txt BarneyB.txt BarneyC.txt

$ echo friend_{lenny,carl}_{1,2}
friend_lenny_1 friend_lenny_2 friend_carl_1 friend_carl_2

$ echo food_{donut{Glazed,Sprinkles},apple,cake}
food_donutGlazed food_donutSprinkles food_apple food_cake
```

## Help

`pinfo` is more helpful than man pages. Good for time killing later. Like an Oreilly book in your terminal. Looks like `pinfo` added color over `info` and some other nice features. 


Do `firefox /usr/share/doc` to read package documentation not in `man` or `info` pages. Mostly READMEs and FAQs, but some other cool docs. There's a whole book under `curl`!


## File I/O tricks

The I/O streams have numbered *channels*.

```
0	stdin
1	stdout
2	stderr
3+	filenames
```

Here are those redirection commands you always forget:

`2> filename` redirect `stderr` to `filename`

`2> /dev/null` discard error message

`> file 2>&1` redirect stdout and stderr to overwrite same file

`&>file` redirect stdout and stderr to overwrite same file (new syntax)

`>> file 2>&1` redirect stdout and stderr to appended to same file

`&>>file` redirect stdout and stderr to appended to same file (new syntax)

## Editing shell

`$LANG` sets locale for the shell. Do `man locale` to find file path for list of different locales if you ever need to change it. 

```
$ date
Tue 08 Oct 2019 08:06:28 AM EDT
$ LANG=fr_FR.UTF-8
$ date
mar. oct.  8 08:06:53 EDT 2019
```

`export` makes a variable an environment variable. A regular shell variable can just be used by the shell but an *environmnet* variable can be used by programs running from the shell too.

`$EDITOR` sets the editor for shell. Change it back to `vim` if some dorks ever sets it to `nano`.

`unset $SOME_VARIABLE_NAME` removes variables

`export -n $SOME_VARIABLE_NAME` to just unexport a variable without unsetting it for the current shell

# Users and Groups

`id` shows info for current user

`id some-user-name` gives info for some-user-name

`ls -l file` give user for a file.

`ls -ld dir` gives user for a directory

`ps -au` gives user for a process

Users are displayed by name, but handled by OS as UID. `/etc/passwd` maps users to UIDs. 
Example line:

`userA:x:1000:1000:User Alpha:/home/userA:/bin/bash`

username : password (moved to `/etc/shadow`, just an `x`) : UID : primary GID : real name : home dir : shell

If user has no login shell, it would be `/sbin/nologin`. 

When creating a new user, also create a new group with same name to be the primary group for that user. That user is the only member of that *user private group*. This simplifies file permissions.

*Supplemnetary groups* are mapped to users in `/etc/group`. 

## Superuser

Use `su -` instead of `su` because `su -` because it starts a new shell with fresh environment. `su` will also start a shell but will retain current environment variables.

`sudo` is even better because all commands run are logged to `/var/log/secure`.
All Members of group `wheel` have full admin access with `sudo`. This includes running commands as any other user. 

A non-admin account can `sudo su -` to gain interactive root shell. Or run `sudo -i` or `sudo -s`.

`sudo` is configed with `/etc/sudoers` which can only be edited with `visudo`. Each line specifies a user or a group (marked with `%`, which hosts with this file the rule applies, and which users sudo can be used to run *as*. `/etc/sudoers` is also a concanteation of files under `/etc/sudoers.d`. So admins can add or remove sudo rights by adding or removing files to that directory. 

```
userA	ALL=(ALL) ALL
%groupB ALL=(ALL) ALL
ansible ALL=(ALL) NOPASSWD:ALL	# not need a password
```

## Manage users

Make sure you always delete users with `userdel -r` to remove their files. Admins can run `find / -nouser -o -nogroup` to find all unowned files and directories. 

### UID ranges

UID 0 is root
UID 1-200 are "system users" for OS system processes
UID 201-999 are system users which don't own files. UIDs assigned dynamically from the pool of unused ids.
UID 1000+ are regular users.

## Manage groups

System groups have GIDs 0-999 and *user private groups* match user UIDs from 1000+, so supplementary groups should have a range much higher above. 

`usermod -g` changes a user's primary group. 

`usermod -aG` adds a user to a supplementary group.

## Manage passwords

`/etc/shadow` stores passwords. Only root can read it. It also has password aging and expiration.

`:` separated records.

```
username for password : encrypted password : day last changed : min number of since last change that user needs to update password or get locked out : days that can pass until password expires : warning period for when user is nagged to update password : inactivity period - days after expirateion for which password still works and after which account is locked : day password expires : last field empty for future use
```

encrypted password is `$` delimited. 
```
hashing algorithm $ salt $ encrpyted hash
```

`chage` can implement password aging policy for user. 

`usermod -L` locks an account from logging in. You can prevent an account from ever logging in by assigning it the shell `/sbin/nologin`

## Processes

Process is an address space of memory, security context of ownership and privileges, execution threads, and process state. Environment of a process is the local and global variables, schedulin context, and any resources allocated by OS (file descriptors, network ports).

Each process has unique process id (PID) and its parent process id (PPID) in its environment. All processes are children of `systemd` in RHEL8. 

`fork` creates child process, copying the parent process address space along with security, file descriptors, ports, resources, environment variables and running code. Parent *sleeps* while child runs, setting a *wait* to be signaled when child finishes. When it does, the child discards all resources except a *zombie* entry in the process table. The parent then removes the zombie from the table and resumes its execution. 

### Process states

```
Running		R	TASK_RUNNING		executing
Sleeping	S	TASK_INTERRUPTIBLE	waiting for hardware request, OS resource, or signal
Sleeping	D	TASK_UNINTERRUPTIBLE	as S but does not respond to signals
Sleeping	K	TASK_KILLABLE		as D but may be killed
Sleeping	I	TASK_REPORT_IDLE	as K but not counted for usage stats
Stopped		T	TASK_STOPPED		suspended until signaled to resume
Stopped		T	TASK_TRACED		as T but debugging
ZOMBIE		Z	EXIT_ZOMBIE		child process exiting
ZOMBIE		X	EXIT_DEAD		child process released. will not show in lists
```

Look for statuses in the `S` column of `top` and the `STAT` column of `ps aux`

### `ps` tricks

`ps` takes three option formats. POSIX options must have a dash in front. BSD options must not have the dash. GNU long options have two dashes, can't really be grouped. So, `ps -aux` is not t he same as `ps aux`.

`ps aux` is most common

`ps lax` has more details, slightly faster in that it doesn't look up user names from UIDs

`ps -ef` similar to `ps lax`

`ps` by itself gives processes for current shell with same effictive user id. 

Normally sorted by PID, which goes out of order as processes created/destroed. Use `-O` or `--sort`. 

## Job Control

One job per pipeline at terminal (single command is a short pipeline).
Only one job can read from terminal at a time --  the foregrounded process. 
Background a new process with `&` at the end of the pipeline. The last command's process will be te PID reported for that job, although all processes are still part of that job.

`jobs` lists currently running jobs for this terminal.
`fg` can bring a job back to foregound. 
`Ctrl+z` suspends a foregounded job.
`bg` resumes a job in the background.

```
$ echo 'hi' | sleep 1000 &
[1] 4572
$ fg %1
echo 'hi' | sleep 1000
^Z
[1]+  Stopped                 echo 'hi' | sleep 1000
$ bg %1
[1]+ echo 'hi' | sleep 1000 &
$ ps
  PID TTY          TIME CMD
 4531 pts/8    00:00:00 bash
 4572 pts/8    00:00:00 sleep
 4606 pts/8    00:00:00 ps
```

### Killing Processes

Process Management signals
Numbers are for `x86_64`. For cross platform, use the short names instead 
```
1	HUP	Hangup			terminate terminal control process or reload process config without termination
2	INT	Keyboard interrupt	terminate program, may be blocked or handled. Ctrl+c
3	QUIT	Keyboard quit		as 2, with process dump. Ctrl+\
9	KILL	Kill, unblockable	abrutly terminate program, cannot be handled, alway fatal
15	TERM	Terminate		"polite" terminate to allow self-cleanup.
18	CONT	Continue		resume process if stopped
19	STOP	Stop, unblockable	stops process, may not be handled or blocked
20	TSTP	keyboard stop		stops process, but may be handled or blocked. Ctrl+z
```

`killall` signals multiple process by command name. `pkill` allows signalling multiple processes by command name, UID, GID, PPID, or terminal. `pgrep` lists processes same way instead of signalling them. 


Force user logout: `pkill -SIGKILL -u bob`

`w` lists logged in users and stats on resource usage.

Lists users processes: `pgrep -l -u bob`

`pstree` views process tree for parent process or single user. Root parent process can be killed to kill all children in tree. 

### monitoring process

*load average* is rough gauge of pending system resource requests and if load is going up or down.
*load number* is collected every 5 seconds from processes in runnable and uninterruptible states. load number is reported as moving average for last 1, 5, and 15 minutes. These are the last three numbers in `uptime`. 

## Services and Daemons

`systemd` daemon manages service startup and resources at boot time and when system is running. By convention, daemon names end in `d`. 

A *service* is one or more daemons. `oneshot` changes to the system such as starting or stopping a service need not leave a daemon afterward. 

`systemd` has PID 1.

Improvements over the old init scripts:
- parallization, so boots up faster
- on-demand daemon starting without needed a separate service
- automatic service dependency management. Ie, don't run a network service if the network is down.
- track related processes by control groups (containers!!)

Systemd objects are managed by *units*. 
- `.service` units are system services to start frequently run daemons like webservers
- `.socket` units are interprocess communication (IPC) sockets. If client connects to socket, then systemd starts associated daemon and connects them.
- `.path` units are activated by file system changes

`systemctl` manages units. 
Display unit types with `systemctl -t help`
List all currently loaded service units: `systemctl list-units --type=service`
List all unit files: `systemctl list-unit-files`

`systemctl status name.type` to view status of a unit. If type is ommitted, shows service unit.

Verify status with 

`systemctl is-active some-service.service`

`systemctl is-enabled some-service.service`

`systemctl is-failed some-service.service`

To list all failed units: `systemctl --failed --type=service`

## Controlling System Services

### Starting and Stopping

To start:
`systemctl start some-service.service` or `systemctl start some-service`

To stop: `systemctl stop some-service`

To restart `systemctl restart some-service`

To reload configuration of service without restart: `systemctl reload some-service`

If you're not sure if service even has config files: `systemctl reload-or-restart some-service`

### Unit Dependencies

systemd and systemctl automatically start units when needed as dependencies for other services. To completely disable, remove all dependent units. 

List dependencies with: `systemctl list-dependencies some-service.service`

### Masking/Unmasking service

Masking a service prevents accidentally starting a service that conflicts with another by creating a link in config directories to `/dev/null`. Trying to start masked service will fail.

`systemctl mask serviceA`

To unmask:

`systemctl unmask serviceA`

### Start Service at Boot

To start on boot:
`systemctl enable serviceA`

This creates symlink from service unit file in `/usr/lib/system/system` to where `systemd` looksfor files: `/etc/systemd/system/some-service-name.target.wants`

This does not actually start the service. Be sure to run `systemctl start` and `systemctl enable` together.

## Managing SSH

When user connects to ssh server, `ssh` client checks for public key in known hosts file `/etc/ssh/ssh_known_hosts` or `~/.ssh/known_hosts

If match on hostname, then client compares public key to what received from ssh server. 
If those don't match, then network could be hijacked or server compromised. User must confirm to continue. 
Set `StrictHostKeyChecking` to `yes` in ~/.ssh/config` or `/etc/ssh/ssh_config` to always abort when keys don't match.

If hostname does not have a key `known_hosts` then user must confirm to continue and then an entry will be added for that host to the file.

To update the key for a host, update the third field in the `known_hosts` file to match the appropiate key for that host. You can find the public key for a host in its `/etc/ssh/` dir ending in `.pub` .

### key based auth

`ssh-keygen` generates public and private keys to `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`

If you overwrite existing key files, then you will need to replace on all servers currently logged with those keys. 

If you don't set a password for the keys, then anyone with your private key file could login as you. If you do set a password, then you'll have to enter a password for the keys (not the host) each time you login. 

Share the public key with host with `ssh-copy-id`.

`ssh-agent` caches the password so you only have to enter it once per session. It prints shell commands to set environment variables. `eval $(ssh-agent)` with execute the shell comands and display PID of ssh-agent running process. Once running, use `ssh-add` to load the ssh private keys into memory. 

You can use the `-i` flag to specify a non-default private key.
`ssh -i .ssh/non-default-key user@example.com`

### customize OpenSSH service config

OpenSSH service is daemon `sshd` configed with `/etc/ssh/sshd_config`.

#### Prohibit root ssh logins

`PermitRootLogin	no`

or to only allow with private key based logins,

`PermitRootLogin	without-password`

Reload the server: `systemctl reload ssh`

#### Private key only logins

`PasswordAuthentication		no`

Reload the server: `systemctl reload ssh`


## Logs

`systemd-journald` handles syslog messages. 
Collects event messages from kernel, early boot process, stdout and stderr from daemons, and syslog events. 
Restructures to standard format and writes to structured indexed system journal. By default, jounral is on filesystem that does not persist after rebooting.

`rsyslog` reads from `systemd-journald` journals and forwards them to other services and may write syslog messages to persistent log files in `/var/log`, sorted by the programs which sent each message and their priority. 

Log files in `/var/log`:

- /var/log/messages`: most non-debug syslog messages, except security related

- /var/log/secure`: security related syslog messages

- /var/log/maillog`: email syslog messages

- /var/log/cron`: syslog messages about scheduled jobs

- /var/log/boot.log`: non-syslog messages from system startup

### reviewing logs

syslog messages categorized by facility (type) and priority. See `rsyslog.conf` man page.

Message logging by rsyslog can be configured in  `/etc/rsyslog.conf` and any file in /etc/rsyslog.d` ending in `.conf`. 
Each syslog rule is a single line in the file. The left side is a filter for facility and priority. The right side is which file to save the message.
Multiple matches mean the message is written to multiple files. The keyword `none` can disable writing that message.

`logrotate` can be configured to rotate old log files with dates in the rotated filenames.

To manually log a syslog message, use `logger` to send that message to the `rsyslog` service. By default, it sends as the `user` facility with priority `notice`. Use `-p` to change this. This can be useful in testing the `ryslog` config. 

### reviewing system journals

`journalctl` displays all messages in the journal, or can be filtered. `root` has full access to all logs, regular uses may not see some messages. 

`notice` and `warning` message are in bold. `error` and higher are in red. 

`-n` limits the output like in `head`.

`-f` follows the output like in `tail -f`.

To just view errors: `journalctl -p err`

Other priority levels: `debug`, `info`, `notice`, `warning`, `err`, `crit`, `alert`, and `emerg` 
To limit by time, `--since` and `--until`

Ex. 

`--since today`

`--since "2019-02-10 20:35:55" --until "2019-02-13 12:00:00"

`--since "-1 hour"

To see even more details in logs, use `-o verbose` to unlock hidden fields. These fields can be used for further filtering. 

Most common fields to filter by:
```
 _COMM is the name of the command 
 _EXE is the path to the executable for the process 
 _PID is the PID of the process 
 _UID is the UID of the user running the process 
 _SYSTEMD_UNIT is the systemd unit that started the process 
```

Example filter: `journalctl _UID=1001 _COMM=cowsay`

### Preserve journal after reboot

System journals defaultly stored to `/run/log/journal/` which clears on reboot.

Change the `Storage` param in `/etc/systemd/journald.conf` . Options are:

`persistent`: stores to  persistent `/var/log/journal/` instead. 
`volatile`: stores to volatile `/run/log/journal/`
`auto`: rsyslog just does whatever it wants. The default.

## Accurate Time

`timedatectl` for current timezone settings.

`timedatectl list-timezones` for available timezone names

`tzselect` to help choose correct zone.

`timedatectl set-timezone America/NewYork` to set timezone

`timedatectl set-time` to set current time in ISO format

`timedatectl set-ntp true` to enable NTP syncing

### Chronyd

`chronyd` sync hardware clock with NTP clocks. 

*Stratum* of NTP source is how authoritive. `stratum 0` is the time source. `stratum 1` is an NTP server directly attached to it. `stratum 2` is a machine synchronizeing with the NTP server. 

`/etc/chrony.conf` has three fields: server, ip/host, and options. `iburst` uses 4 measurements to set the clock. 

`chronyc` is the client to the `chronyd` daemon. To test the NTP changes, use `chronyc sources -v`


## Networking

### Concepts

#### TCP/IP Layers

- Applicaton: protocols for apps. Ie: ssh, https, nfs, cifs, smtp, etc.

- Transport: TCP is reliable, with connections. UDP is unreliable, connectionless. Applications use tcp and udp ports, you can find a list in `/etc/services`.

- Internet/Network: IPv4 and IPv6 protocols route packets between hosts on different networks based on the host IP address and a prefix for the network address. 

- Link / Media Access: physical media. Ethernel (802.3) and Wireless WLAN (802.11). Hardware MAC address are destinations on local network segments


#### Network Interface Names

`eth0` and such are obsolete now because the numbering system was based on when devices were attached, whcih could vary between boots (PCI doesn't have an order) and normal usage. 

New names start with interface type
```
en	ethernet
wl	WLAN
ww	WWAN
```

Next are codes based on firmware or PCI topology

```
o N	on-board device, firmware index number N	eno1	onboard ethernet device 1	
s N	PCI hot-plug slot N				ens3	hotplug device 3
pMsN	PCI on bus M, slot N				wlp4s0	wireless bus 4, slot 0
pXsYfZ	multifunction cards, fZ denotes function #	enp0s1f1
```

#### IPv4 Networking

IP address is 32 bit number, broken into 4 8-bit octets. Address divided into host part and network part. All hosts on same network share same network part. Host part uniqly IDs each host on network. 

Size of networks are variable, so netmasks used see which part of IP address is the network part. Netmask is binary ANDed with IP address to get the network address.  

#### IPv6 Networking

128 bit number, broken into 8 colon separated groups of 4-bit nibbles. `::` denotes repeated `0` nibbles.  When using an IPv6 address with a port, always put the address in brackets. 
`[20:10:de:ad:f1:54:0:7]:80`

Two parts: network prefix and interace ID. Unlike IPv4 variable masks, IPv6 has a standard `/64` subnet mask. This split the address in half, the first part is the network prefix and the later part is the interface id. 

Often network provide gives `/48` as prefix, so remaining `/16` can be used for local subnets. 

Special Address and Networks
```
::1/128		localhost	as 127.0.0.1/8, loopback
::		unspecified	as 0.0.0.0, listening to all IP address
::/0		default route	default route for router to send to
2000::/3	global unicast
fd00::/8	unique local
fe80::/10	link local
ff00::/8	multicast
```

Link local addresses are unroutable, only for hosts on specific network link. Every interface automatically have link-local address to `fe80::/64`. The interface id portion of the address is unique by using the MAC address to help form it. 

To use it, provide the interface name as the scope identifier at end of address, denoted with `%`. 

For example, to `fe80::121:dead::1%eno0`

Multicast address allow sending to multiple specific hosts. No broadcast address in IPv6, so multicast more common in IPv6. 

#### host names

Dealing with ip addresses are a pain, so always name them in `/dev/hosts`

### Validating Network Configs

`ip link show` to list all network interfaces

`ip addr show eno1` to show details for interface `eno1`

`ip -s link show eno0` to show network performance stats

`ping` and new `ping6` to test connection to hosts, ip addresses. Must include scope identifier for link local addresses

`ip route` to show routing table

`ip -6 route` to show IPv6 routing table

`tracepath` and `tracepath6` to show hops to destination. Similar to `traceroute` and `traceroute6`. 

`ss` to display socket statistics, replacement for `netstat`. 

To show all TCP sockets: `ss -ta`

### Configuring Networking

NetworkManager is daemon to manage and monitor network settings. Tools interact with NetworkManager and save config files to `/etc/sysconfig/network-scripts`

A *device* is network interface. 

A *connection* is collection of settings for a device with a unique name.
Only one connection active for device at a time. Connections may be shared by devices, or same device can switch connections. For example, when swtching form a home wifi to work wifi to public wifi.
`nmcli` creates and edits connection files. 

`nmcli dev status` to show status for all devices. 

`nmcli con show` to list all connections. `--active` to filter for just active connections. 

`nmcli con add` to add new connection. 

`nmcli con add conn-name my-new-conn type ethernet ifname eno2` 
creates a new connection named `my-new-conn` for interface `eno2`, creates a new file `/etc/sysconfig/network-scripts/ifcfg-my-new-conn`. This connection gets its IPv4 address from DHCP and autoconnects on startup. It get IPv6 address by listening for router multicasts on local-link. 

`nmcli con add conn-name my-other-conn type ethernet ifname eno2 ipv4.address 192.168.0.5/24 ipv4.gateway 192.168.0.254` creates a `my-other-conn` ethernet connection on `eno2` with static IPv4 address/netmask and default gateway. Autoconnects on startup and saves config to `/etc/sysconfig/network-scripts/ifcfg-my-other-conn`.

`nmcli con add conn-name my-third-conn type ethernet ifname eno2 ipv6.address 2001:db8:0:1::c000:207/64 ipv6.gateway 2001:db8:0:1::1 ipv4.address 192.0.2.7/24 ipv4.gateway 192.0.2.1`
creates a `my-thid-conn` with static ipv4 and ipv6 address and gateways. 

`nmcli con up some-connection-name` activates the connection `some-connection-name` on the interface its bound to. Note that the connection is activated, not the interface. 

`nmcli dev dis eno1` disconnect interface `eno1` and brings it down. 

`nmcli dev dis eno1` abbreviates the above

`nmcli con down connection-name` usually not the best way because most connections are autoconnecting, so NetworkManager will immediately restart it. 

`nmcli con show connection-name` to show detailed status for connection. Some settings are *static* and set by admin, stored in config files. Others are *active* and not persisted, such as DCHP data. 

`nmcli con mod connection-name` to change settings. Static settings are backed up to that connections config file. 

`nmcli con mod static-eno1 ipv4.address 192.0.2.2./24 ipv4.gateway 192.0.2.254` to update static ip4 address and gateway for connection `static-eno1`

`nmcli con mod static-eno1 ipv6.address 2001::1 pv6.gateway 2001::0` to update static ip6 address and gateway for connection `static-eno1`

When changing a connection from DHCP to static, be sure to change `ipv4.method` from `auto` to `manual` and likewise when changing from SLAAC or DHCPv6 to static, change `ipv6.method` from `auto` or `dhcp` to `manual`.

`nmcli con del static-eno1` to delete connection `static-eno1`. This also disconnects the device and removes config file.

`nmcli con reload` to reload all config files, helpful after hand editing onfig files.

`root` can always make changes with `nmcli`. Users logged in at local consoles or desktop client can also make changes because it implies physical presence and need to update network connections as needed. Regular users `ssh`ed in do not have access to change network settings without root access. 

`nmcli gen permissions` to see what current permissions you have.

### Editing Network Config files

I'm not copying that whole table of config options. Check the docs if you need to do this!

### Configuring Hostname and Name Resolution

`hostname` displays the current host's name

You can update the hostname by editing `/etc/hostname`.
`hostnamectl` command can update this file and display fully qualified host name. If `/etc/hostname` is missing, it looks up the hostname by reverse DNS. 

`hostnamectl set-hostname host@example.com`

`hostnamectl status`

Name resolution is configed by `/etc/nsswitch.conf` and by default `/etc/hosts` is checked first to map ip addresses to hostnames. 

`getent hosts host-name` can test host name resolution from `/etc/hosts`. 

`/etc/resolv.conf` configs how the hostname query works. `search` followed by a list of short host names to check as DNS server. `nameserver` followed by IP address of nameserver to try. 

NetworkManager can update the `/etc/resolv.conf` with the `nmcli con mod IDNAME +ipv4.dns IP.ADDRESS`

Likewise for IPv6: `nmcli con mod static-eno1 +ipv6.dns 2001:2345::1`

`host HOSTNAME` can test DNS server connection

```
$ host 8.8.8.8
8.8.8.8.in-addr.arpa domain name pointer dns.google.
```

## Archiving and transferring files
### archiving

`tar`

`-c`, `--create` 

`-x`, `--extract`

`-t`, `--list`

`-v`, `--verbose`

`-f`, `--file=` filename, must be provided of archive to use or create

`-p`, `--preserve-permissions`

`-z`, `--gzip` use gzip compression to create/use `.tar.gz`

`-j`, `--bzip2` use bzip2 compression to create/use `.tar.bz2`. Better compression.

`-J`, `--xz` xz compression `.tar.xz`, even bettern than bz2 (!!)

`--xattrs` to save extended attributes like ACL or SELinux permissions

To create a tar file: 

`tar -cf archive.tar file1 file2`

`tar -cf archiveOfDir.tar fileDir/`

To extract

`tar -xf someArchive.tar`

To create compressed:

`tar -czf backup.bz bigDir/`

To verify:

`tar -tzf backup.bz`

To extract (don't forget to create new dir before unzipping):

`tar -xzf backup.bz`

### Transferring

Copy from here to there:
`scp [target files and dirs]+ user@hostname:/destination/dir`

Copy from there to here:
`scp [user@hostname:/target/dir/targetfile /local/destination/dir`

Copy remote directory recursively with `-r`:
`scp -r [user@hostname:/target/dir/targetfile /local/destination/dir`

Use `sftp` to interactively upload and download from remote server.
`sftp user@hostname`

Download a file: `sftp> get fileName`

Upload a file: `sftp> put reliative/to/fileName/from/home/dir`

### Synchronizing Files 

`rsync` copies files between systems by only sending the differences. Initially the same as regular copy but much faster after. `-n` to "dryrun" a sync to see which changes are sent and make sure nothing important is overwriten or deleted. `-a` archive mode preserves permissions, timestamps, symbolic links, ownerships, and works on device files as well. Use `-H` to preserve hard links as well.

`rsync -av /source/dir /dest/dir`

`rsync -av /source/dir user@hostname:/dest/dir`

Note: trailing `/` on source directory to sync only synchronizes to files,  not the directory itself.

## Installing software

`subscription-manager` to register your system with Red Hat support and unock secret bonus levels.

Entitlement certifications show that system has been subscribed to support. 

`/etc/pki/product`	which Red Hat products installed
`/etc/pki/consumer`	which Red Hat account is registered
`/etc/pki/entitlement`	which Red Hat subscriptions are attached to system

Usually, one one version of a package can be installed at a time, but if the package is designed so there is no conflicting file names, then multiple versions can be installed. See `kernel` and (maybe?) `java` and `python` with their version directories.

### rpm tricks

`rpm` for low level info on package files and installed packages. Use `-p` to refer to a package file, and not the installed package database. 

Queries

`rpm -q [select options] [query options]`

`rpm -qa` list all installed packages

`rpm -qf FILENAME` which package installed FILENAME

`rpm -q package-name` which version of `package-name` is installed

`rpm -qi package-name` more detailed info

`rpm -ql package-name` list files installed by package

`rpm -qc package-name` list just config files for package

`rpm -qc package-name` list just documentation files for package

`rpm -q --scripts package-name` displays any shell scripts run when package installed, removed

`rpm -q --changelog package-name` changes in package

Use `cpio` and `rpm2cpio` if you want to get files out of an rpm without installing it.

Extract all files:
`$ rpm2cpio somepackage-1.0.0.amiga.rpm | cpio -id` 

Extract some or all:

`$ rpm2cpio somepackage-1.0.0.amiga.rpm | cpio -id "*txt"` 

`$ rpm2cpio somepackage-1.0.0.amiga.rpm | cpio -id "README.txt"` 

### yum tricks

`yum help` to show everything I'm going to type about anyway.

`yum list 'cow*'` to list installed and available packages matching pattern

`yum search KEYWORD` to search in name and summary fields

`yum search all KEYWORD` to search in name, summary, and description

`yum info package-name` to get details on single package

`yum provides /path/name` to find which package created the file path

`sudo yum update` to update all packages

`yum update kernel` to actually install the latest kernel. `uname -r` to verify version. 

`yum group list` to see available package groups

`yum group list hidden` to see package groups hidden by environment groups

`yum group info "Group Name"` for mandatory, default and optional packages

`yum group install "Group Name"` to install all mandatory and default packages in group

#### Package transaction history

`/var/log/dnf.rpm.log` logs all install and remove transactions

`yum history` summarizes the log

`yum history undo` reverses a transaction by ID number

### Yum Repositories

`yum repolist all` to view all default available repos

`yum-config-manager --enable some-repo-name-rps` to enable/disable a repo

#### Third Party Repo

New repos added by creating a file in `/etc/yum.repos.d/` ending in `.repo` containing the repo URL, whether to use GPG for validting package signatures, and the URL to the GPG key to use. 
Automate with `yum-config-manager`

`yum-config-manager --add-repo="http://dl.example.org/path/to/some/repo/"

Then edit the file to include the GPG key if available. Best practice is to download the file to `/etc/pki/rpm-gpg/` and edit the repo file to have line `gpgkey=file:///etc/pki/rpm-gpg/REPO-GPG-KEY`

You can also add new repos with an `rpm` package.

`rpm --import http://dl.example.org/path/to/gpg/REPO-GPG-KEY`

`yum install http://dl.example.org/path/to/some/repo.noarch.rpm`

To disable searching a repository, but leave it defined, set `enabled=0` in its config. 

### Package Module Streams

*Modularity* lets a single repo host many versions of a package and its dependencies. Previously, managing multiple versions required separate repos for each. 


RHEL8 has two main software repos: 

BaseOS: core OS content as rpm packages. Traditional component lifecycle.

AppStream: content with varying lifecycles as both modules and packages. Some required components and many applications which were previously in Red Hat Software Collections and third party repos. 

Module is a set of packages working together, usually a specific large application or programming language. Typically includes the main application, dependency libraries, documentation, and help utilities. 

Module Streams each support a different version of the module, and can be updated independently. For each module, only one stream can be enabled at a time to provide packages for installation. 

Module Profiles are lists of certain packages to be installed for specific cases, ie. minimal, server, client, development, high availability, etc. If not specified, then module will provide default profile. 

`yum module list` to list available modules

`yum module list nodejs` to list streams for specific modules

`yum module info nodejs` to get details of module

`yum module info --profile nodejs:12` to see which packages are installed by which profile in module version

Install and update like normal packages and groups

`sudo yum module install nodejs`

or `yum install @nodejs`, the `@` denotes a module name

Verify module streams and profile with `module list nodejs`

To remove an installed module:
`yum module remove nodejs`

After removing, the module stream is still enabled.

To disable module stream:
`yum module disable nodejs`

Switching a module stream implies upgrading or downgrading to a different version. First remove all the modules proivded by that modules stream. This removes all the packages installed by that module and their dependencies. 

`sudo yum module remove postgresql`

`sudo yum module reset postgresql`

`sudo yum module install postgreql:10`

## Filesystem Access

Mostly review, looks like there's more device types than 15 years ago. 

```
/dev/sda, /dev/sdb, ...		SATA/SAS/USB storage
/dev/vda, /dev/vdb, ...		virtio-blk paravirtualized storage (some VMs)
/dev/nvme0, /dev/nvme1, ...	NVM storage (SSDs)
/dev/mmcblk0, /dev/mmcblk1, ...	SD/MMC/eMMC storage (SD cards)
```

Some VMs use `virtio-scsi` storage which mounts as `/dev/sd*` names

(*fbroke formatting)

LVM combines several block devices and partitions into *volume groups*, which can be split nto *logical volumes*, the equivalent of physical disk partitions. 

LVM makes a directory under `/dev` for the volume group and a file for each logical volume. So for volume group `vg` and logical volume `lvhome`, the logical device file name is `/dev/vg/lvhome`. 

Don't forget the `df -h` command to overview all mounted disks and space usage.

Use `du -h` for space usage on a directory tree, recursively.

### Mounting and Umounting 

`lsblk` to see details for block devices. Man, this is much nicer than `fdisk -l`. And it supprts LVM!

`# mount /dev/vdb1 /mnt/data` To mount a virtual device `/dev/vdb1` to *empty dir* `/mnt/data`. Don't forget to create the empty dir before mounting!

However, device names vary by the order in which added to system, may var between reboots. Safer to use UUID of filesystem on devices. 

`lsblk -fp` to list full path, UUID, and type of file system 

`# mount UUID="asfdas-asfasf-asdffasf-asfaf" /mnt/data` to mount by UUID.

If logged into most desktop environments, they will automount removable storage media at `/run/media/USERNAME/LABEL` where `USERNAME` is your username and `LABEL` is what what given to the filesystem when created. 

Always unmount the device manually before removing.

`umount /mnt/data`

Cannot unmount if any processes are using it. 

`lsof /mnt/data` to see all processes using a directory. 

Don't forget to `cd`out of a directory you are trying to `umount`!

### Locatin file tricks

`locate` uses a pregenerated index, `mlocate`, to instantly lookup filenames

Normally `mlocate` is regenerated daily, but can be manually regenerated with `updatedb`

`find` crawls through heirarchy in real time from some starting dir.

`find / -name cowsayd_config` to search entire system for `cowsayd_config`

`find / -name *cowsay*` to search for any files with `cowsay` in name`

`find / -iname *message*` to search names, ignoring case

`find -user homer` to search by user

`find -uid 1001` to search by UID

`find -gid 1001` to search by GID

`find / -user home -group moesbar`

`find /home -perm 764` to search for files with rwxrw-r-- permission in /home

#### Find files by size
`find -size 100M` find files approx 10M

`find -size +100M` find files greater than 10G

`find -size -100M` find files less than 10k

#### Find files by modification time

`-mmin` followed by minutes and `+` or `-` modifier

`find -mmin 120` files last modified 120 minutes ago

`find -mmin +240` find files last modified over 4 hours ago

`find -mmin -30` find files last modified less than 30 min ago

#### Find by type

`find -type d` to list directories

`find -type l` to list soft links

`find -type f` to list regular files

`find /dev -type b` to list block devices

`find / -type f -links +1` find all files with more than 1 hard link

## Server support

Web Console to monitor system in a browser, based on Cockpit project.

```
sudo yum install cockpit
sudo systemctl enable --now cockpit.socket

# open firewall if needed
sudo firewall-cmd --add-service=cockpit --permanent
sudo firewall-cmd --reload
```

Then login with browser to `https://servernam:9090`


You can also use the Red Hat Support Tool to search for help and file bug reports.

`redhat-support-tool`.

For automatic detection of issues, register your system with Red Hat Insights

`insights-client --register`

-----------------------------------------------------------

## Command line tricks

The `#!` at the beginning of a script is actually a "magic number" to indicate file type. See `magic(5)` for more. 

### bash loops

```bash
for VARIABLE in LIST; do
    COMMAND VARIABLE
done
```

Examples

`for HOST in host1 host2 host3; do echo $HOST; done`

`for HOST in host{1,2,3}; do echo $HOST; done`

`for HOST in host{1..3}; do echo $HOST; done`

`for FILE in file*; do ls $FILE; done` uses glob to make list from `filea fileb filec` in dir

`for PACKAGE in $(rpm -qa | grep kernel); do echo "$PACKAGE installed on $(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE))"; done`

`for EVEN in $(seq 2 2 10); do echo "$EVEN"; done`

#### exit codes

`exit` to leave script early. 

`exit 0` to finish without error

`exit 1` (or any other number 1-255) to finish with error

Error code is return to parent process, which stores it in `?` variable. This can be accessed with `$?`

```
$ false
$ echo $?
1
$ true
$ echo $?
0
```

#### conditionals

`test <TEST EXPRESSION>` checks a condition and stores value to `$?`, where `0` means test passed and `1` is test failed.

`test 1 -gt 0 ; echo $?`

`test 0 -gt 1 ; echo $?`

Bash provides builtin syntax `[ <TEST EXPRESSION ]` and more recently `[[ <TEST EXPRESSION> ]]` with glob pattern matching and regex pattern matching.

Bash's command parser is based on spaces, so don't forget to space out the brackets and other operators!

#### Bash if/then/else

```
if <condition>; then
	<statements>
fi
```

```
if <condition>; then
	<statements>
	else
	<statements>
fi
```

```
if <condition>; then
		<statements>
	elif <condition>; then
		<statements>
	else
		<statements>
fi
```

### Regex 

Use `man egex` as reference because I'm not typing all that out if I already know 95% of it. 

## Schedule future tasks

### scheduling deferred job

`at` package provides `atd` daemon and related tools `at`, `atq`. Installed by defualt.

Use `at TIMESPEC` to schedule jobs. 26 queues `a` to `z` arranged by priority with `z` being lowest. See `/usr/share/doc/at/timespec` for grammar. 

Examples:

`at now + 1 min`

`at teatime tomorrow`

`at noon + 4 days`

`at 02:00pm`

`at 15:59`

`at 6pm august 7 2029`

Use `atq` to view pending jobs or `at -l`

Use `at -c JOBNUMBER` to view commands the will run for job

Use `atrm JOBNUMBER` to remove job with JOBNUMBER

### scheduling recurring jobs

`crond` enabled and started by default for recurring jobs. 
`crond` reads user cron files and a set of system wide files for when jobs run. 
By default, `crond` will try to email output or errors from jobs when they run, but the output can instead be redirected to other files. 

Users should use `crontab` to manage jobs instead of writing cron files directly.

`crontab -i` list jobs for current user

`crontab -r` remove all jobs for current user

`crontab -e` edit jobs for user. 

`crontab filename` remove all jobs, replace with contents of *filename* or stdin

`roout` can use `crontab -u` to schedule jobs as a different user, but shouldn't use this to edit system cron files. 

#### Job format

fields in this order: minutes, hours, day of month, month, day of week, command to run

`*` for don't care/always

a number for minutes hours, dates, or day of week with Sunday being 0, Monday being 1, etc, and Sunday back at 7.

`x-y` range from x to y, inclusive.

`x,y` for lists. Can also include ranges. Ie. `2, 4, 8-10, 12` in the minutes column for job to  run at 2, 4, 8, 9, 10, and 12 minutes past the hour.

`*/x` for an interval of x. So `*/5` in minutes column means run the job every 5 minutes. 

Examples:

`0 0 12 25 * /usr/local/bin/merry_christmas` to run on christmass day every year

`*/5 9-17 * Jan-Mar Mon-Fri echo "do your taxes!"` 

`0 9 * * 1-5 mutt -s "Morning boss" boss@example.com % Hey there, just checking in` to send email every morning. 

You can always cat `/etc/crontab` for examples and `man crontabs`. 

System jobs should be in `/etc/crontab` or `/etc/cron.d`. Good practice to create new files to add under `/etc/cron.d` so they aren't overwritten by package updates to `/etc/crontab`.

Special dirs for scripts to run every hour, day, week, month: `/etc/cron.hourly/`,`/etc/cron.daily/`,`/etc/cron.weekly/`,`/etc/cron.monthly/`. Make sure scripts are executable.


The `/etc/cron.d/0hourly` file runs the `/etc/cron.d/hourly` scripts using the `run-parts` command. 

`/etc/anacrontab` calls `run-parts` for the daily, weekly, and monthly tasks. `/etc/anacrontab` makes sure that jobs aren't skipped if machine is off or hibernating and run as soon as system is back up. You can adjust the delay after rebooting to run the tasks with `Delay n minutes` param in `/etc/anacrontab`. 
`START_HOURS_RANGE` param sets a time interval so that resumed jobs are not restarted outside of that range and must wait until next day. 

#### Systemd Timer

systemd *timer units* activates other units (usually a service) with matching unit name.  

For example, `sysstat` package provides `sysstat-collect.timer` to collect system stats every 10 min. Configed with `/usr/lib/systemd/system/systat-collect.timer`. The every 10 min is set by `[Timer]` param `OnCalendar` which can be changed for much more specific timers. 
`OnUnitActiveSec` can delay the unit to activate. 

After edits, reload the config with `systemctl daemon-reload`
Then activate the timer unit: `systemctl enable --now <unitnae>.timer`

#### Managing temp files

`systemd-tmpfiles` helps purge old temp files to save space and clear outdated data.

On startup, `systemd` runs `systemd-tmpfiles --create --remove`  to clear any files marked for deletion in `/run/tmpfiles.d/*.conf`, `/usr/lib/tmpfiles.d/*.conf`, and `/etc/tmpfiles.d/*.conf` and touch/chown any files marked for creation with correct permissions. 

`systemd-tmpfiles-clean.timer` triggers `systemd-tmpfiles-clean.service` to run `systemd-tmpfiles --clean` to regular clear out old files.

Config the `[Timer]` section of these timer units to adjust when to run. 

```
[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
```

This config will run the the timer 15 min after boot, then again 24 hours since last activation.

After changing, always run `systemctl daemon-reload` to reload the config and enable the timer with `systemctl enable --now systemd-tmpfiles-clean.timer`

#### manual cleaning

`systemd-tmpfiles --clean` and `systemd-tmpfile --create` use same config, but `--clean` purges all files which haven't been accessed/modified more recently than as configged. 

Seven columns: type, path, mode, uid, gid, age, and argument. Type is which action to take. `d` to create a dir, `z` to recursivly restore SELinux context and file permissions and ownership. 

`d /run/systemd/seats 0755 root root -`
Create the `/run/systemd/seats/ dir with user and owner `root` and permissions `0755`

`D /home/studen 0700 student student 1d`
Create the `/home/student` dir. If it already exists, purge its contents. When `--clean` is run, remove all files not touched in more than one day. 

`L /run/fstablink - root root - /etc/fstab` create symbolic link `/run/fstablink` to `/etc/fstab`. Never automatically purge. 

#### config file precedence

`/usr/lib/tmpfiles.d/` are provided by RPM packages and shouldn't be edited manully. 

`/run/tmpfiles.d` are already volative and used by daemons to manage their own runtime temp files. 

`/etc/tmpfiles.d/` meant for admin to config custom temp locations and override defaults. 

`/etc` has highest priority, then `/run` then `/usr`. So you can overrite a vendors config by copying from `/usr/lib/tmpfiles.d/` to `/etc/tmpfiles.d` and changing it there   

### Performance tuning

`tuned` applies system settings according to profile to optimize for specific requirements. 

Static tuning: `tuned` applies kernel params for overall performance and does not change in response to activity. 

Dynamic tuning: `tuned` monitors system and adjusts in response to behavior changes. 

Install and enable
```
yum install tuned
systemctl enable --now tuned
```

`tuned-adm` helps change `tuned` settings. 

`tuned-adm active` to check current profile

`tuned-adm list` to list available profiles

`tuned-adm profilename` to switch to profile

`tuned-adm recomend` to recommend a tuning profile. This sets the  default after installation. 

`tuned-adm off` to revert tuned changes by current profile. 

In Web Console, can also adjust under System -> Performance Profile

### Process Scheduling

Nice levels range from -20 (highest priority) to 19 (lowest priority) with 0 as default. Niceness refers to how willing a process is to cede CPU time to other processes. 

Only `root` can *reduce* a process nice level. Normal users can *increase* the nice levels of their own processes. 

`top` and `ps` can display nice levels. In `top`, the nice levels are mapped from -20..19 to 0..40. This `ps` command gives nice levels by process:

`ps axo pid,comm,nice --sort=-nice`

`nice command` start `command` with nice level 10 by default. 

`nice -n15 command` to start `command` with nice level 15. 

`renice` can change nice level of active process, using its PID

`renice -n 19 1234`

## ACLs to control file access

*Access Control Lists* give more fine grained file permissions to different *named users* and *named groups*. Users can set ACLs on files and dirs they own. `CAP_FOWNER` usrs can set ACLs for any file and dir. New files subdirs inherit the ACL settings of their parent dir. 
Note the `X` permissions for directories allow the directory's contents to be searchable.

Filesystems need to be mounted with ACL support. ext3 and ext4 have the `acl` enable by default in RHEL. XFS have ACL built in. To enable support, toggle the ACL option when using `mount` to mount the filesystem or in the `/etc/fstab` file and reboot. 

`ls -l` will only show a `+` at the end of the permissions string to show that ACLs exist for that file/dir. 

`getfacl file-name` to display the ACL settings for file-name. 

`getfacl .` to display ACL settings for current dir

`getfacl -F /some/dir` to recursively display ACL details for `/some/dir`

`getfacl` output can be used as input for `setfacl`. So you could direct the output to a file as a backup, then restore later with `setfacl --set-file=backup-acl.txt`

Example output:
```
[user@host content]$ getfacl .
# file: .
# owner: user
# group: operators
# flags: -s-
user::rwx
user:consultant3:---
user:1005:rwx
group::rwx
group:consultant1:r-x
group:2210:rwx
mask::rwx
other::---
default:user::rwx
default:user:consultant3:---
default:group::rwx
default:group:consultant1:r-x
default:mask::rwx
default:other::---
```

First part is comments for targeted name, owner user, and owner group. If any extra dir flags `setuid`, `setgid`, `sticky`, then a list of flags. Here, `setgid`: `flags: -s-` 

Then list of user ACLs. Note the owner `user`, the named user `consultant3`, and the UID `1005`. 

Then the group ACLs. Note the owner group has `rwx`, but the group `consultant1` can only `r-w` and GID `2210` has `rwx`. 

The `default:mask:rwx` shows the initial maximum permission for any new files or subdirs. New subdirs can have execute permission, but not new files. 

`default:other::---` means all UIDs and GIDs not mentioned above have NO permissions to new files or subdirs. The named UID and GID above will not get initial ACL entries for new files and subdirs. But they can still create their own files and subdirs. 

#### ACL mask

ACL mask is max permissions to named users, group owner, and named groups. It doesn't affter permissions of file owner or `other` non-named users. All files and dirs with ACLs will have ACL mask. Mask can be viewed with `getfacl` and set with `setfacl` and will be recalculated when any relevent ACLs are updated. 

#### Permission Precedence

Given a process

Does the process's user own the file? Then file users ACL permissions apply

Is the process's user a named user for that file? then the named users ACL permissions apply.

Is the process running under a group that owns the file or is a named group? Then ACL for that group applies

Else, the file's `other` ACL permissions apply. 

#### Examples

`systemd-journald` uses ACL to allow read acces to `/var/log/journal/.../system.journal` to `adm` and `wheel` groups. This lets `adm` and `wheel` users view the journals without giving access to secure content in `/var/log`
```
$ getfacl /var/log/journal
getfacl: Removing leading '/' from absolute path names
# file: var/log/journal
# owner: root
# group: systemd-journal
# flags: -s-
user::rwx
group::r-x
group:adm:r-x
group:wheel:r-x
mask::r-x
other::r-x
default:user::rwx
default:group::r-x
default:group:adm:r-x
default:group:wheel:r-x
default:mask::r-x
default:other::r-x
```


`systemd-udev` uses `udev` rules to set ACLs so that users logged in with deskto GUIs can fully access devices like CD/DVD players, USB storage, sound cards, etc. ACLs active until user logs out, then new ACLs applied when another user logs in.


### securing with ACLs

`setfacl` to add/modify/remove ACLs on files and directories. Same permission syntax as `chmod`. 

`-m` to add/modify an ACL.  Other ACLs unaffected.

`-M` to add/modify with a file or STDIN (`-` as filename). Other ACLs unaffected.

`--set` or `--set-file` to completely replace all ACLs.

`setfacl -m u:some-user:rX some-filename` read and execute perms for `some-file` for `some-user`

If user field is blank, then assumes file owner. Similar to `chmod`, but `chmod` has no effect on named users.

`setfacl -m g:some-group:rw some-file` to give `some-group` read/write access to `some file`

`setfacl -m o::- file-name` to give no permissions to non-owner, non-named users and groups.

Can set multiple ACLs with comma seperated entries:

`setfacl -m u::rwx,g:consultants:rX,o::- file-name` to give owning user rwx, rx to the consultants group and no permissions to everyone else. 

#### Recursive ACL mods

`setfacl -R -m u:some-user:rX /some/dir` to recursively give some-user rX permissions to all files and subdirs in /some/dir

#### Deleting ACLs

`setfacl -x u:some-user,g:some-group some-file` to remove ACL for `some-user` and `some-group` from `some-file`.
Other ACLs for that file are unaffected.

Mask ACL cna only be deleted after all other ACLs are deleted. When mask is gone, then `+` will no long appear in `ls -l` output to show ACL presence. 

`setfacl -b some-file` to remove all ACLs at once

#### ACL inheritance

Set a default ACL on a directory so all files and subdirs inherit the ACL by default.
The directory itself still needs regular ACL, setting a default ACL just enables the inheritance.

`setfacl -m d:u:some-user:rx /some/dir/` to give some-user rx perms to /some/dir and subdirectories.

Same syntax as regular ACLs, just prefixed with `d:` or with `-d`

Delete a default ACL the same way, but with `d:`

`setfacl -x d:u:some-user /some/dir`

## SELinux Security

File permissions only control *who* can user a file, not *how* and to *do what*. 
If a hacker gains control of a process, they can execute actions as its user and spread throughout the system.
So if a hacker controls `apache`, which has write access to `/var` and `/tmp`, they can see all other files in those dirs. Not good!

SELinux is set of polcies for an executable on what files and operations it is allowed for execution, config, and data.

Modes

- Enforcing: blocks bad access. On by default.

- Permissive: not blocking access, but logs SELinux interactions. For testing and debugging.

- Disabled: don't do this. 

Display the current mode with `getenforce` and change with `setenforce`.
You can set at boot time with the kernel param `enforcing` where `enforcing=1` turns on Enforcing mode and `enforcing=0` is Permissive.
You can turn it off completely with boot param `selinux=0` or back on with `selinux=1`
Kernel params override this, but you can also change the file `/etc/selinux/config` to set enforcement mode. 

Every file, dir, process and port has a *SELinux context* label. 
This label is checked to see if a process can access it. 
Explicity access is required by default.

SELinux labels have contexts for user, role, type, and sensitivity level.  

SELinux user names are suffixed `_u`, roles `_r` and types `_t`. 

`someuser_u:object_r:httpd_sys_context_t:s0:/var/www/html/index.html`

By default in RHEL, the type context is used  for access rules. 

To display SELinux labels, use the `-Z` flag for `ps`, `ls`.
To set SELlinux labels, use the `-Z` flag for `cp` and `mkdir`.

### Controlling Contexts

All processes and files are labeled with security context. New files inherit their context from parent dir.

However, files copied from other directories may retain their original directory context, not the context of their destination. This happens with `cp -a`. 
(Copy, preserve all)

`chcon` changes SELinux context for a file or dir. It should only be used for testing because it does not backup to SELinux context database.
`restorecon` will reset the context to the default for that location. 

```bash
$ mkdir test
$ ls -Zd test
unconfined_u:object_r:user_home_t:s0 test

$ chcon -t httpd_sys_content_t test
$ ls -Zd test
unconfined_u:object_r:httpd_sys_content_t:s0 test

$ restorecon -v test
Relabeled /home/kh/develop/github.io/herbertkb.github.io/_posts/test from unconfined_u:object_r:httpd_sys_content_t:s0 to unconfined_u:object_r:user_home_t:s0
$ ls -Zd test
unconfined_u:object_r:user_home_t:s0 test
```

`semanage fcontext` changes context and affects the database `restorecon` uses to reset contexts to default.

`sudo semanage fcontext -l` lists the rules `restorecon` will apply to restore to default. 

`sudo semanage fcontext -a -t some-new-content_t` to add a context type

`sudo semanage fcontext -d` to delete a context type


### SELinux Booleans

SELinux Booleans are on/off rules to fine tune SELinux behavior. 

`man booleans` for an overview.

`man -k _selinux` to list all boolean man pages 

`getsebool -a` to list all booleans current values.

`getsebool httpd_enable_homedirs` to get current value of specific boolean

`sudo semanage boolean -l` as well to see current values and short description. requires root access.

`sudo semanage boolean -l -C` to list just the values that differ from default

`sudo setsebool httpd_enable_homedirs on` to turn a boolean on or off

`sudo setsebool -P httpd_enable_homedirs on` to turn a boolean on or off, persistent over rebooting

### Investigating SELinux Issues

SELinux is usually doing its job correctly by blocking access. Ie, a webserver trying to access `/home` when user files haven't been configured for sharing. 

Most common issue is wrong file context comes from when a file is created in a place with one context and then moved to a place with a different context. 

An SELinux Boolean may also need to be adjusted. Ie, `ftp_anon_write` to allow anonymous ftp users to upload files. 

#### Monitoring for SELinux violations

`setroubleshoot-server` sends SELinux error messages from `var/log/audit/audit.log` to `/var/log/messages`.

These messages have a UUID which can be searched for with `sealert -l UUID` to generate detailed summary around specific incident. 

`sealert -a /var/log/audit/audit.log` to generate report for all incidents. 

`ausearch` can also search the `/var/log/audit.log` for incidents and filter by incident type and time.

`ausearch -m AVC -ts recent`

Web Console also includes a friendly way to view SELinux reports. 

## Basic Storage

MBR boot scheme is obsolete because 32-bitness limited filesystem sizes to 2 TiB, which is a normal hard drive these days. 

GUID Partition Table (GPT) is new standard for UEFI (Unified Extensible Firmware Interface) devices. 64-bit for logical addresses and up to 128 partitions, from 15 for MBR (at best!). 
Each disk and partition has a globally unique identifier (GUID) and partition table stored and begining and end for redundany with primary at head of device and secondary at the end. Checksums detect errors in GPT header and partition table.

### Partition editing

`parted` replaces `fdisk`. Takes whole device name as argument and a command. Or leave the command off to start interactive session. Note: commands issued immediately (unlike fdisk which had a sperate apply step) so be careful. 

`parted /dev/sda print` to display partitions on `/dev/sda`.

`parted /dev/sda unit GB print` to display with sizes in gigs. Other unit options: `s` for sectors, `B` for bytes, `MiB` `GiB` `TiB` for powers of 2 sizes, `MB`, `GB` `TB` for SI sizes (powers of 10). 

To write a new partition table, begin with writing a label. Note that writing a label erasing the current partition table. 

`parted /dev/sda mklabel msdos` for MBR label

`parted /dev/sdb mklabel gpt` for GPT label

To create the partitions, you can start `parted /dev/name` without any other args to start interactive session. Or do it all at once:

`parted /dev/sda mkpart primary xfs 2048s 100GB` to create MBR primary partition starting at sector 2048s and continuing for 100GB.

`parted /dev/sda mkpart userdata xfs 2048s 100GB` to create GPT xfs partition named `userdata` starting at sector 2048s and continuing for 100GB.

After updating partition tables, run `udevadm settle` for `udev` daemon to recognize the new partitions and create entries under `/dev`

`parted /dev/sda rm N` to remove partition number `N` from `/dev/sda`

Format the new partition(s) with a filesystem. 

`mkfs.xfs /dev/sda1` to format /dev/sda1 with XFS

`mkfs.ext4 /dev/sdb2` to format /dev/sdb2 with EXT4

Mount with `mount /dev/sda1 /mnt/new/mountpoint`.

Display all mounted partitions with `mount` by itself. 

To persistenly mount after reboot, update `/etc/fstab`. Remember to use the UUID for the partition because device names could vary on startup. Use `lsblk -fs` to scan for device UUIDs. 

First column is device name or UUID. Second column is mount point. Third column is file system type. Fourth column is comma separated list of options. `default` is fine to put here. Fifth column is used by `dump` for backups. Can leave as `0`. Sixth column is used y `fsck` on startup to see if filesystem should be checked. `0` turns off. `1` turns on. `2` starts in parallel. Good practice to start root partition as `1` and other secondary partitions in parallel. 

To verify `/etc/fstab`, use `umount` to first unmount the partition, then `mount /mnt/point` which checks `/etc/fstab` to remount. Or use `findmnt --verify`. A bad `/etc/fstab` can make the system unbootable so always check.  

Always run `systemctl daemon-reload` to register the changes in systemd.

### Swap Space

To create a new swap partition, create a new partition but set the type to `linux-swap`. Load it with `udevadm settle`. Format it for swap usage with `mkswap /dev/sda2` where `/dev/sda2` is the new swap partition. Use `swapon /dev/sda2` to activate it. You can see currentl swap usage with `free`. You can turn off the swap aprtition with `swapoff`, which will try to make any active memory pages to another swap device first. If it cannot, the swap space stays active. 

To keep the swap partition persistently, add it to the `/etc/fstab` with mount point `swap` and filesystem `swap`. Always run `systemctl daemon-reload` to register the changes in systemd.

If you have multiple swap partitions, assign priorty with the `pri=` option in `/etc/fstab`. Default priority is `-2` and higher priorities used up first. `swapon --show` can display the current swap partition priorities. If partitions have same priority, pages are assigned round robin.  

## Logical Volume Management

*Physical devices* are block storage devices. Could be partitions or whole disks. Must be initialzed as LVM physical volume first. Whole disk will be used if specified. You can scan for devices with `lsblk`, or `cat /proc/partitions`.

*Phyisical volumes (PV)* are partitions formatted for LVM. Segmented into same sized *physical extents* (PEs). Scan with `pvdisplay`.

*Volume Groups (VGs)* are pools of one or more PVs. PV can only go to one VG. Scan with `vgdisplay`.

*Logical Volume (LV)* is like a logical partition of a volume group It can be resized with remainning free PEs in the VG. *Logical extents (LEs)* map directly to PEs. Mirroring maps a LE to 2 or more PEs. Scan with `lvdisplay`.

### Steps to create a logical volume

1. Create a partition with `parted` or `fdisk` and set the type to `Linux LVM` or `0x8e`. Use `partprobe` to register the new partition in the kernel. You can scan for partions with `lsblk`, or `cat /proc/partitions`

2. Make the partition a PV with `pvcreate`. Ex. `pvcreate /dev/sda2`

3. Create a volume group with `vgcreate vg-name /dev/sda2 /dev/sda3`

4. Create a logical volume with `lvcreate -n lv-name -L 20G vg-name` to create an LV named `lv-name` on VG `vg-name` of 20 GiB size. Use `-l` to assign number of PEs for size. 

5. Format with filesystem such as with `mkfs.xfs` or `mkfs.ext4` or `mkswap`. For example, `mkfs -t xfs /dev/vg-name/lv-name`

6. Mount `mount /dev/vg-name/lv-name /mnt/stuff`

7. Add to fstab: `/dev/vg-name/lv-name /nmt/stuff defaults 1 2`. Don't need to provide UUID because LVMs are already uniquely IDed with UUID by LVM itself.

### Removing LVM

1. Unmount the volume. `umount /mnt/stuff` or `umount /dev/vg-name/lv-name`

2. `lvremove /dev/vg-name/lv-name` to delete the logical volume. This frees the PEs mapped to the LEs for this volume and allow to be re=used for other LVs in same VG. 

3. `vgremove /dev/vg-name` to delete a VG and free all its PEs for other VGs to use.

4. `pvremove /dev/sda2  /dev/sda3` to delete PV metadata from the partitions

### Extending LVM

#### Expanding a VG

Assume a new phyical device available at `/dev/sdb` of 500GB. 

First we create the new physical volume

`parted -s /dev/sdb set 1 lvm on`

`pvcreate /dev/sdb1`

Then we extend the VG to include the new PV.

`vgextend vg-name /dev/sdb1`

Validate with `vgdisplay`.

#### Shrinking a VG

To shrink a VG by removing a PV, first we need to move all the data off that PV to other PVs in the VG. This assumes enough free PEs in the other PVs. 

`pvmove /dev/sdb1`

Then remove the PV from the VG

`vgreduce /dev/sdb1`

#### Expanding an LV

1. first verify that enough free PEs are in VG with `vgdisplay`

2. `lvextend -L +500G /dev/vg-name/lv-name` to expand `lv-name` by 500GB. If the `+` were missing, then the total size of `lv-name` would be set to 500GB. 

3. extend the filesystem on the LV to the rest of its new PEs.
	
	for XFS: `xfs_grow /mnt/stuff`

	for ext4: `resizefs /dev/vg-name/lv-name`

#### Extending a swap space LV

1. `vgdisplay vg-name` to verify enough PEs are available.

2. Deactivate swap space. Swap cannot be extended "live". `swapoff -v /dev/vg-name/lv-swap`

3. Extend the logical volume: `lvextend /dev/vg-name/lv-swap -l 100`

4. Reformat the LV with swap: `mkswap /dev/vg-name/lv-swap`

5. Reactivate the swap space: `swapon -va /dev/vg-name/lv-swap`

## Advanced Storage

### Multilayered Storage with Stratis

Stratis manages a pool of storage devices to create and manage volumes. Built on existing LVM and XFS tooling. File systems are "thinly provisioned" on pools. They do not have a fixed size or preallocate space. Stratis manages hidden LVM volumes to expand the filesystems as needed. Size of filesystem is how many blocks of storage currently being used. Space available is remaining blocks in the entire pool. Multiple filesystems cna be in same pool, but filesystems can also reserve extra space. 

Stratis uses metadata on the filesystems to function. Do not manage these filesystems outside of Stratis or you will overwrite/break the metadata and just have a bad day.

Devices in a pool can be in either data tier for large devices like hard disks or cache tier for high I/O devices like SSDs. 

#### Managing

Install Stratis `yum install stratis-cli stratisd`

Enable: `systemctl enable --now stratisd`

Operations

- Create device pools: `stratis pool create pool-name/dev/sda`

	Pools are subdirs under `/stratis`

- View pools: `stratis pool list`

- Add more devices to pool: `stratis pool add-data pool-name/dev/sdb`

- View block devices in pool: `stratis blockdev list pool-name`

- Create a new filesystem in pool: `stratis filesystem create pool-name filesystem-name`

	Filesystems are under `/stratis/pool-name`

- View filesystems: `stratis filesystem list`

- To mount in `/etc/fstab`, get UUID for new filesystem with `lsblk --output=UUID /stratis/pool-name/filesystem-name`. 
	MUST include option `x-systemd.requires=stratisd.service` for mounting after Stratis daemon is started or else you will boot into `emergency.target` if this is a root partition. 

### Compressing storage with Virtual Data Organizer (VDO)

Virtual Data Organizer is a device mapper driver that compresses files and checks for duplicate files to further save disk space and throughput. 

Uses kernel module `kvdo` to compress data and `uds` to deduplicate data.

Installed on top of block devices or RAID and below storage layers like LVM and filesystems. 

Three stages:

- zero-block elimination
- deduplication
- compression

#### VDO Volumes

VDO Volumes act like disk partitions. They can have filesystems written on them and used for LVM. They are thinly provisioned, so users only see *logical* size, not the *physical* size. 

When creating, you can give it a logical size greater than the phyiscal size of the block devices it is over. If you ommit the size, VDO will the create the volume with 1:1 correspondence to physical device, which is faster, but misses out on storage efficiency.

#### Usage

Installing: `yum install vdo kmod-kvdo`

Creating a VDO volume: `vdo create --name=my-vdo --device=/dev/sda --vdoLogicalSize=1000G`

Can then format and mount like regular filesystem. 

Check status with: `vdo status --name=my-vdo`

List VDOs active: `vdo list`

Stop a VDO: `vdo stop`

Start a VDO: `vdo start`

## Network Storage

### Mounting and Unmounting NFS shares

#### mount

To mount root of share and browse shared directories:

```bash
$ sudo mkdir /mnt/point
$ sudo server-name:/mnt/point
```

If you know the actual path on share you want,

```bash
$ sudo mount -p /mnt/point
$ sudo mount -t nfs -o rw,sync serverb:/share /mnt/point
```

`-o rw,sync` tell mount to immediately sync `/mnt/point` with share. 

`-t nfs` tells mount that its an NFS share. 

#### at boot with /etc/fstab

Example `/etc/fstab` entry:

`serverb:/share /mount/point  nfs  rw,soft  0 0`

Then `sudo mount /mnt/point`, if not already mounted. 

#### on demand

With automounting, user mounting does not need admin access.
Also, share is not permanently connected from `/etc/fstab`, saving newtork and cpu resources.  

Install: `sudo yum install autofs`

Enable: `sudo systemctl enable --now autofs`

Add a *master map* file to `/etc/auto.master.d`. 
This tells autofs what the base dir is for mapping shares and the automount mapping file.
`sudo echo "/shares /etc/auto.demo" > /etc/auto.master.d/master-map.autofs"`
`.autofs` extension required for autofs to recognize file. Can include multiple master map files. 

Create the mapping file. By convention, filename begins with `auto.` then its identifier. `sudo echo "work  -rw,sync server-name:/shares/work " > /etc/auto.work` To map `/shares/work` on `server-name` to `/shares/work` locally. Here, `work` is the "key" used as mount point by autofs. `/shares` and `/shares/work` are created and removed as needed by autofs.

##### Direct map

To mount to a preexisting dir that autofs won't create and delete at will, use this format for the master map file:

`/- /etc/auto.direct`

The `/-` point tells autofs to use an existing mount point. Then, in `/etc/auto.direct`, a normal entry as:

`/mnt/docs  -rw,sync  server-name:/shares/docs`

autofs will not remove `/mnt`. But autofs will create/destroy `/mnt/docs`as needed.

##### wildcard maps

`*  -rw,sync  serverb:/shares/&`

Here, everything under `/shares` on `serverb` is automounted when a user tries to access it. Ie, a user tries to access `/shares/dira/dirb` then `/shares/dira/dirb` is automatically created locally and mounted. 

#### unmount

`umount /mnt/point`

### nfsconfig

RHEL8 has new `nfsconfig` tool to manage client and server configs. Driven by `/etc/nfs.conf`. See `[nfsd]` section to config server. 

`nfsconfig --set section key value` to update config with tool instead of by file. Easier to automate.

`nfsconfig --get section key` to get value for a param in a given section

`nfsconfig --unset section key` to reset value 



## Booting Up!

## Network Security

## Install RHEL

lmao this is the last chapter? haha, nice.

