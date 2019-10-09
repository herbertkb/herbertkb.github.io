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


