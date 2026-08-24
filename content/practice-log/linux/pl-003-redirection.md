---
title: "PL - 003 — Process I/O Redirection, Shell Expansions, and Globs"
date: 2026-06-07
draft: false
---
### Process Grouping & Output Redirection

The shell manages process input/output via file descriptors (FDs). By default:
* `0`: Standard Input (`stdin`)
* `1`: Standard Output (`stdout`)
* `2`: Standard Error (`stderr`)

The redirection operator `>` modifies FD 1 for the immediate command preceding it. Command chaining via `;` runs execution sequentially without grouping output streams.
### Terminal Session
```
[aadarsha@labserver ~]$ date; cal ; ls
Sun Jun  7 06:48:59 AM +0545 2026
      June 2026     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30            
                    
dir1  dir2  dir3  file1  file2  file3

# In sequential execution, `>` applies ONLY to the final command (`ls`).
# `date` and `cal` send output to stdout (terminal); `ls` writes to `command_output`.

[aadarsha@labserver ~]$ date; cal; ls > command_output		# Only output of ls command is saved
Sun Jun  7 06:49:34 AM +0545 2026
      June 2026     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30            
                    
[aadarsha@labserver ~]$ cat command_output 
command_output
dir1
dir2
dir3
file1
file2
file3
[aadarsha@labserver ~]$						

# Command Grouping with Subshell: ()
# Parentheses spawn a child process subshell via fork(). 
# Redirection applies to the entire subshell's aggregated output stream.

[aadarsha@labserver ~]$ (date; cal; ls) > command_output      # using () ---> output of all commands is saved
 
[aadarsha@labserver ~]$ cat command_output 
Sun Jun  7 06:50:12 AM +0545 2026
      June 2026     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30            
                    
command_output
dir1
dir2
dir3
file1
file2
file3
[aadarsha@labserver ~]$

# Alternative: In-Process Grouping {}
# Braces group execution within the CURRENT shell (no subshell overhead).
# Note: Requires a space after `{` and a trailing semicolon before `}`.
[aadarsha@labserver ~]$ { date; cal; ls; } > command_output

# use of {} ---> use to define the range
# creating multiple Dir/Sub-dir at a time

[aadarsha@labserver ~]$ ls
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ mkdir dir{1..25}

[aadarsha@labserver ~]$ ls
dir1  dir10  dir11  dir12  dir13  dir14  dir15  dir16  dir17  dir18  dir19  dir2  dir20  dir21  dir22  dir23  dir24  dir25  dir3  dir4  dir5  dir6  dir7  dir8  dir9

[aadarsha@labserver ~]$ rmdir dir{10..25}

[aadarsha@labserver ~]$ ls
dir1  dir2  dir3  dir4  dir5  dir6  dir7  dir8  dir9

[aadarsha@labserver ~]$ touch file{10..20}

[aadarsha@labserver ~]$ ls
dir1  dir2  dir3  dir4  dir5  dir6  dir7  dir8  dir9  file10  file11  file12  file13  file14  file15  file16  file17  file18  file19  file20

[aadarsha@labserver ~]$ rm -f file{16..20} 

[aadarsha@labserver ~]$ ls
dir1  dir2  dir3  dir4  dir5  dir6  dir7  dir8  dir9  file10  file11  file12  file13  file14  file15

[aadarsha@labserver ~]$ rm -r *

[aadarsha@labserver ~]$ ls

[aadarsha@labserver ~]$ mkdir -p dir1/dir2/dir3/dir4/dir5

[aadarsha@labserver ~]$ ls
dir1

[aadarsha@labserver ~]$ tree
.
└── dir1
    └── dir2
        └── dir3
            └── dir4
                └── dir5

6 directories, 0 files

[aadarsha@labserver ~]$ ls
dir1
 
# use of {} and -p

[aadarsha@labserver ~]$ mkdir -p testcompany/{service/{development,devops/{cloud/{aws,azure,gcp},onpremise},security,AI},support/{technical,others},management}

[aadarsha@labserver ~]$ ls
dir1  testcompany

[aadarsha@labserver ~]$ tree testcompany/
testcompany/
├── management
├── service
│   ├── AI
│   ├── development
│   ├── devops
│   │   ├── cloud
│   │   │   ├── aws
│   │   │   ├── azure
│   │   │   └── gcp
│   │   └── onpremise
│   └── security
└── support
    ├── others
    └── technical

15 directories, 0 files
[aadarsha@labserver ~]$
 
# Creating a file/dir with special characters in their name

[aadarsha@labserver ~]$ ls
dir1  testcompany

[aadarsha@labserver ~]$ vi "my file"            # file name with space inside " "

[aadarsha@labserver ~]$ ls
 dir1  'my file'   testcompany

[aadarsha@labserver ~]$ cat my\ file		# escape sequence --> \
This is the test file.

# creating a hidden file

[aadarsha@labserver ~]$ touch .secretdata

[aadarsha@labserver ~]$ ls
 dir1  'my file'   testcompany

[aadarsha@labserver ~]$ ls -a
 .   ..   .bash_history   .bash_logout   .bash_profile   .bashrc   dir1  'my file'   .secretdata   testcompany

[aadarsha@labserver ~]$ vi .secretdata 

[aadarsha@labserver ~]$ cat .secretdata 
this is the hidden file

[aadarsha@labserver ~]$ ls -a -l   		# ( ls -al ) --> in the most cases options sequence doesn't matter
total 24
drwx------. 4 aadarsha aadarsha  148 Jun  7 07:54  .
drwxr-xr-x. 3 root     root       22 May 26 15:36  ..
-rw-------. 1 aadarsha aadarsha 3843 Jun  4 22:45  .bash_history
-rw-r--r--. 1 aadarsha aadarsha   18 Oct 29  2024  .bash_logout
-rw-r--r--. 1 aadarsha aadarsha  144 Oct 29  2024  .bash_profile
-rw-r--r--. 1 aadarsha aadarsha  522 Oct 29  2024  .bashrc
drwxr-xr-x. 3 aadarsha aadarsha   18 Jun  7 07:02  dir1
-rw-r--r--. 1 aadarsha aadarsha   23 Jun  7 07:29 'my file'
-rw-r--r--. 1 aadarsha aadarsha   24 Jun  7 07:54  .secretdata
drwxr-xr-x. 5 aadarsha aadarsha   54 Jun  7 07:24  testcompany
``` 

### Wildcard Characters
```
# ?	      --> matches exactly one character
# *  	  --> matches zero or more characters

# [set]   --> matches one character within the set
   # [az]    --> matches within the set
   # [a-z]   --> matches within the range
   # eg: ls [b-p]*

# [!set] or [^set] --> Matches one character NOT in the set
    # [^az]   --> matches out of the set
    # [^a-z]  --> matches out of the range
    # eg: ls [!b-p]*

# [[:class:]]	--> Locale-safe POSIX character class	
    # eg: ls [[:lower:]]* 


[aadarsha@labserver ~]$ cd /

[aadarsha@labserver /]$ ls
afs  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

[aadarsha@labserver /]$ ls -d ???   					# to search on first level directory not inside
afs  bin  dev  etc  lib  mnt  opt  run  srv  sys  tmp  usr  var

[aadarsha@labserver /]$ ls -d b??
bin

[aadarsha@labserver /]$ ls -d b*
bin  boot

[aadarsha@labserver /]$ ls -d *t
boot  mnt  opt  root
 
[aadarsha@labserver /]$ ls -d *r
usr  var

[aadarsha@labserver /]$ ls -d r??t
root

[aadarsha@labserver /]$ ls -d [b-p]*
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc

[aadarsha@labserver /]$ ls -d [^b-p]*
afs  root  run  sbin  srv  sys  tmp  usr  var

[aadarsha@labserver /]$ ls -d [b-p]*
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc

[aadarsha@labserver /]$ ls -d [^b-p]*
afs  root  run  sbin  srv  sys  tmp  usr  var

[aadarsha@labserver /]$ ls -d *[c-z]
afs  bin  boot  dev  etc  home  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
 
[aadarsha@labserver /]$ ls -d *[c-f]
etc  home  proc

[aadarsha@labserver /]$ ls -d *[bh]
lib

[aadarsha@labserver /]$ ls -d [bh]*
bin  boot  home

[aadarsha@labserver /]$ ls -d [hs]*
home  sbin  srv  sys

[aadarsha@labserver /]$ ls -d [hs]*
home  sbin  srv  sys

[aadarsha@labserver /]$ ls -d [sh]*
home  sbin  srv  sys

[aadarsha@labserver /]$ ls -d [^she]*
afs  bin  boot  dev  lib  lib64  media  mnt  opt  proc  root  run  tmp  usr  var
 
[aadarsha@labserver /]$ ls -d [she]*
etc  home  sbin  srv  sys

[aadarsha@labserver /]$ pwd
/
[aadarsha@labserver /]$ cd ~			# home directory

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ ls
 dir1  'my file'   testcompany

# path navigation
  # Absolute Path: Fully qualified route evaluated from the root directory (/)
  # Relative Path: Route relative to the Current Working Directory (.) using parent (..)

[aadarsha@labserver ~]$ tree 
.
├── dir1
│   └── dir2
│       └── dir3
│           └── dir4
│               └── dir5
├── my file
└── testcompany
    ├── management
    ├── service
    │   ├── AI
    │   ├── development
    │   ├── devops
    │   │   ├── cloud
    │   │   │   ├── aws
    │   │   │   ├── azure
    │   │   │   └── gcp
    │   │   └── onpremise
    │   └── security
    └── support
        ├── others
        └── technical

21 directories, 1 file
[aadarsha@labserver ~]$ 
      
[aadarsha@labserver ~]$ cd /home/aadarsha/testcompany/service/security/

[aadarsha@labserver security]$ pwd
/home/aadarsha/testcompany/service/security

[aadarsha@labserver security]$ cd /home/aadarsha/testcompany/service/security/   	# from root: absolute path

[aadarsha@labserver security]$ cd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cd /home/aadarsha/testcompany/service/security/   		# from root: absolute path
[aadarsha@labserver security]$ 

[aadarsha@labserver security]$ cd ../../../dir1/dir2/dir3/dir4/  			# from current path: relative path

[aadarsha@labserver dir4]$ pwd
/home/aadarsha/dir1/dir2/dir3/dir4

[aadarsha@labserver dir4]$ ls
dir5
[aadarsha@labserver dir4]$ cd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
 dir1  'my file'   testcompany
 
# moving and renaming files/dirs : mv command

# mv [options] <source> <destination>

# [option]
  # -i  -->  ask for confirmation  (default for root) 
  # -f  -->  forcefully overwrite   (default for normal users)

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ ls
 dir1  'my file'   testcompany
 
[aadarsha@labserver ~]$ rm 'my file' 

[aadarsha@labserver ~]$ ls
dir1  testcompany
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cd dir1/dir2/dir3/
[aadarsha@labserver dir3]$ 

[aadarsha@labserver dir3]$ vi ~?
```
---
### Summary
- 
- Subshell () vs Grouping {}:
  () forks a new child process to run grouped commands;
  {} groups execution within the existing shell process.
  Process Control: Subshells () execute fork(), while {} executes in-process.

- Expansion Order: 
  Expansion Mechanics: Brace expansions {} evaluate before Glob expansions (*, ?, []).

- File Descriptors: 
  Standard redirection (>) acts on FD 1 (stdout). To capture error streams, use 2> (stderr) or &> (both stdout and stderr).

- POSIX Classes: 
  Range globs like [a-z] can be locale-sensitive. Use [[:lower:]] or [[:alpha:]] for deterministic cross-system behavior.
---