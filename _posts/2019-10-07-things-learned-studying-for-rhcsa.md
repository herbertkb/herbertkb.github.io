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



