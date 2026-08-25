---
title: "PL - 007 — Working with Vi and Vim Editor"
date: 2026-06-11
draft: false
---

### Vim's Editing Model

Vim's editing model is based on combining a count, an operator, and a motion:
```
    [count] [operator] [motion]
```
Examples:
```
dw       → delete to the start of the next word
d$       → delete to the end of the line
3dw      → delete three words
caw      → change a word
y$       → yank from the cursor to the end of the line
```
This operator-and-motion model is one of the fundamental concepts that makes Vim efficient for text editing.

#### Essential Navigation
**Basic movement**
```
h        → move left
j        → move down
k        → move up
l        → move right
```
**Word and line movement**
```
w        → next word
b        → previous word
e        → end of word
0        → beginning of line
^        → first non-blank character of line
$        → end of line
```
**File movement**
```
gg       → first line of file
G        → last line of file
Ctrl-d   → scroll down approximately half a screen
Ctrl-u   → scroll up approximately half a screen
```
**Paragraph and matching-character movement**
```
{        → previous paragraph
}        → next paragraph
%        → jump between matching brackets
```
**Character search**
```
f{char}  → find character forward on the current line
F{char}  → find character backward on the current line
t{char}  → move to character before target, forward
T{char}  → move to character after target, backward
```
**Counts**

Many Vim commands accept a numeric count:
```
3w       → move forward three words
5j       → move down five lines
3dd      → delete three lines
2dw      → delete two words
```

A count can often be combined with an operator and motion:
```
[count] [operator] [motion]
```
**Operators**:

Common editing operators include:
```
d        → delete
c        → change
y        → yank
```

Examples:
```
dw       → delete a word
cw       → change a word
y$       → yank to end of line
d}       → delete through the next paragraph
```
**Text Objects**

Text objects allow operations to be performed on logical units of text.

Common examples:
```
iw       → inner word
aw       → a word
i"       → inside double quotes
a"       → double-quoted text including quotes
i(       → inside parentheses
a(       → parentheses and their contents
```

Examples:
```
ci"      → change text inside quotes
di(      → delete text inside parentheses
caw      → change a word
daw      → delete a word
```
**Basic Editing Commands**
```
i        → insert before cursor
a        → insert after cursor
c        → change
d        → delete
y        → yank
x        → delete character under cursor
X        → delete character before cursor
r        → replace one character
R        → enter Replace mode
o        → open a new line below
O        → open a new line above
J        → join the current line with the next line
.        → repeat the last change
```

The . command is particularly useful because it allows a previous change to be repeated.

**Copy, Delete, and Paste**

Vim commonly uses the term yank rather than copy.
```
yy       → yank the current line
Nyy      → yank N lines

dd       → delete the current line
Ndd      → delete N lines

p        → put after the cursor
P        → put before the cursor
```
Deleted and yanked text is stored in Vim registers and can subsequently be put back into the buffer.

**Registers**
Registers provide storage locations for yanked and deleted text.

Named registers
```
"ayy     → yank into register a
"ap      → put contents of register a
```
**System clipboard**

When Vim has clipboard support:
```
"+y      → yank to the system clipboard
"+p      → paste from the system clipboard
```

Inspect registers with:
```
:reg
```

Vim also provides numbered registers, including the commonly useful:
```
"0       → yank register
"1       → most recent deletion/change register
```
**Marks**

Marks allow locations in a file to be remembered and revisited.
```
Set a mark
m{a-z}
```

Example:
```
ma       → set mark a

Jump to a mark
`{mark}  → jump to the exact position of the mark
'{mark}  → jump to the beginning of the marked line
```

Example:
```
`a       → jump to the exact position stored in mark a
'a       → jump to the beginning of the marked line

**Search**
/pattern     → search forward
?pattern     → search backward

n            → repeat search in the same direction
N            → repeat search in the opposite direction

*            → search forward for the word under the cursor
#            → search backward for the word under the cursor
```

Vim search supports regular-expression patterns, making it useful for locating structured text in large files.

**Search and Replace**
Vim's substitution command follows this general form:
```
:[range]s/{pattern}/{replacement}/[flags]
```

Examples:
```
:%s/old/new/g
```

Replace all matches in the file.
```
:%s/old/new/gc
```

Replace all matches while asking for confirmation.
```
:5,15s/old/new/g
```

Replace matches within lines 5 through 15.

**Important flags:**
```
g        → replace all matches on each line
c        → confirm each replacement
```
**Undo and Redo**
```
u        → undo
Ctrl-r   → redo
```

Vim maintains an extensive undo history and supports an undo tree, allowing changes to be navigated through different editing states.

**File and Buffer Management**
Open and save files
```
:e file          → edit/open a file
:w               → write/save the current file
:w filename      → write to a specified filename
:saveas filename → save under a new filename
```
Exit Vim
```
:q       → quit
:q!      → quit without saving changes
:wq      → write and quit
:x       → write if necessary and quit
```
**Buffers**

A buffer is Vim's in-memory representation of an opened file.
```
:ls       → list buffers
:bnext    → move to the next buffer
:bprev    → move to the previous buffer
:bdelete  → delete/unload a buffer
```
**Windows**

Vim can divide the screen into multiple windows.
```
:split       → horizontal split
:vsplit      → vertical split
```

Move between windows:
```
Ctrl-w h     → move left
Ctrl-w j     → move down
Ctrl-w k     → move up
Ctrl-w l     → move right
```

Resize windows:
```
Ctrl-w =     → equalize window sizes
Ctrl-w _     → maximize height
Ctrl-w |     → maximize width
```
**Tabs**

Vim tabs contain collections of windows.
```
:tabnew      → open a new tab
gt           → move to the next tab
gT           → move to the previous tab
```

A useful mental model is:
```
Buffer  → opened/editable text
Window  → view into a buffer
Tab     → collection of windows
```

These are related concepts but are not interchangeable.

**Vim Configuration**

Personal Vim configuration is commonly stored in:
```
~/.vimrc
```

For example:
```
set number
```

**enables line numbers.**

After modifying the configuration, it can be reloaded with:
```
:source ~/.vimrc
```

**Useful configuration options can include:**
```
set number
set relativenumber
set incsearch
set hlsearch
set ignorecase
set smartcase
```
**Movement:**
```
h j k l
w b e
0 ^ $
gg G
Ctrl-d Ctrl-u
{ }
%
f F t T
```
**Editing:**
```
i a
c d y
x X
r R
o O
J
.
```
**Copy/Delete/Paste:**
```
yy
dd
p P
```
**Visual:**
```
v
V
Ctrl-v
```
**Search:**
```
/
?
n N
* #
```
**Substitution:**
```
:s
:%s/old/new/g
:%s/old/new/gc
```
**Undo/Redo:**
```
u
Ctrl-r
```
**Files:**
```
:e
:w
:q
:q!
:wq
:x
```
**Buffers:**
```
:ls
:bnext
:bprev
:bdelete
```
**Windows:**
```
:split
:vsplit
Ctrl-w h/j/k/l
```
**Tabs:**
```
:tabnew
gt
gT
```
**Registers:**
```
"ayy
"ap
"+y
"+p
:reg
```
**Marks:**
```
ma
`a
'a
```

**Vim's own documentation from inside Vim:**
```
:help
:help {topic}
:help :w
:help yy
:help visual-mode
```
---
### Terminal Session

```
# Modes of vi editor
  # Command Mode
  # Text Input Mode

# Important Vim modes
    # 1. Normal mode    — navigation and commands
    # 2. Insert mode    — insert text
    # 3. Visual mode    — select text
    # 4. Command-line mode — : commands, searches, etc.
    # 5. Replace mode   — replace existing text
# Vim also has Select, Ex, Terminal, and other internal modes.

# 1. Insert Mode
# i
# ESC
# :wq   OR   :x     -->  Save & Exit


# 2. Command Mode
# cursor movement

# G --> move to the bottom of the file
# gg --> Move to the top 

# 'N'G --> Move to the n(th) line

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  testfile  words.gz
dira  impfiles.tar  testcompany         words

[aadarsha@labserver ~]$ cp /etc/passwd .

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  testcompany  words
dira  impfiles.tar  passwd              testfile     words.gz

[aadarsha@labserver ~]$ vi passwd

[aadarsha@labserver ~]$ cd 

[aadarsha@labserver ~]$ vi .vimrc

[aadarsha@labserver ~]$ source .vimrc 

[aadarsha@labserver ~]$ sudo su
[sudo] password for aadarsha: 
[root@labserver aadarsha]# 
 
# vi
# vim = vi + extra features

[root@labserver aadarsha]# yum list vim*
...
[root@labserver aadarsha]# 

[root@labserver aadarsha]# exit
exit
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  testcompany  words
dira  impfiles.tar  passwd              testfile     words.gz

[aadarsha@labserver ~]$ ls -la
total 18544
drwx------. 6 aadarsha aadarsha     4096 Jun  9 05:19 .
drwxr-xr-x. 3 root     root           22 May 26 15:36 ..
-rw-------. 1 aadarsha aadarsha    16303 Jun  8 22:34 .bash_history
-rw-r--r--. 1 aadarsha aadarsha       18 Oct 29  2024 .bash_logout
-rw-r--r--. 1 aadarsha aadarsha      144 Oct 29  2024 .bash_profile
-rw-r--r--. 1 aadarsha aadarsha      609 Jun  8 07:20 .bashrc
drwxr-xr-x. 3 aadarsha aadarsha       33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha       65 Jun  8 12:07 dira
drwxr-xr-x. 4 root     root           28 Jun  8 20:16 extracted
-rw-r--r--. 1 root     root     11704320 Jun  8 16:38 impfiles.tar
-rw-------. 1 aadarsha aadarsha       51 Jun  8 21:54 .lesshst
-rw-r--r--. 1 root     root       784897 Jun  8 16:33 newimpfiles.tar.gz
-rw-r--r--. 1 aadarsha aadarsha     1080 Jun  9 05:17 passwd
-rw-r--r--. 1 aadarsha aadarsha       24 Jun  7 07:54 .secretdata
drwxr-xr-x. 5 aadarsha aadarsha       54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha       56 Jun  7 21:33 testfile
-rw-------. 1 aadarsha aadarsha     7727 Jun  8 08:47 .viminfo
-rw-r--r--. 1 aadarsha aadarsha       11 Jun  9 05:18 .vimrc
-rw-r--r--. 1 aadarsha aadarsha  4953598 Jun  8 09:52 words
-rw-r--r--. 1 aadarsha aadarsha  1476067 Jun  8 12:09 words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vim .vimrc 

[aadarsha@labserver ~]$ vi passwd 

[aadarsha@labserver ~]$ cat .vimrc 
set number
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -a
.              .bash_logout   dir1       impfiles.tar        passwd       testfile  words
..             .bash_profile  dira       .lesshst            .secretdata  .viminfo  words.gz
.bash_history  .bashrc        extracted  newimpfiles.tar.gz  testcompany  .vimrc
[aadarsha@labserver ~]$ vi .vimrc 

[aadarsha@labserver ~]$ vi passwd 

[aadarsha@labserver ~]$ vi .vimrc 

[aadarsha@labserver ~]$ vi .vimrc

[aadarsha@labserver ~]$ cat .vimrc 
set nu
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi passwd 

# not working above --> set number permanently 

# ~/.vimrc  --> is the user's Vim configuration file. Vim reads it during startup under normal circumstances.

# reload it with    --> :source ~/.vimrc   ( within the file)
# or from the shell --> vim ~/.vimrc



[aadarsha@labserver ~]$ vi +10 passwd          # set cursor in line number 10

[aadarsha@labserver ~]$ vi +18 passwd          # set cursor in line number 18

# Set cursor at a maching pattern

  vim +/pattern file



```
### Delete, Copy and Paste
```
yy       → yank the current line
Nyy      → yank N lines

dd       → delete the current line
Ndd      → delete N lines

p        → put after cursor/line
P        → put before cursor/line

u        → undo
CTRL-R   → redo
```
```
[aadarsha@labserver ~]$ vi passwd 

# Ex Mode
```
### Searching Text
```
/pattern      → search forward
?pattern      → search backward

n             → repeat search in same direction
N             → repeat search in opposite direction

*             → search forward for word under cursor
#             → search backward for word under cursor

:set hlsearch → highlight search matches
:nohlsearch   → clear search highlighting
```
```
# same as in man page

# Search and Replace Text

# :%s/<old>/<new>/g   --> search and replace in the whole file with confirmation

# Where:

    % = entire file
    s = substitute
    g = replace all matches on each line
    c = confirm each replacement

# :%s/<old>/<new>/gc  --> search and replace in the whole file without confirmation

# For a range:

# :%5,15s/<old>/<new>/g or gc --> search and replace in the whole file with or without confirmation in the given lines

```
### Visual mode
```
v        → characterwise Visual mode
V        → linewise Visual mode
CTRL-V   → blockwise Visual mode

y        → yank/copy selection
d        → delete selection
p        → put/paste
```
```
[aadarsha@labserver ~]$ vi passwd 

# Password Protect a file

[aadarsha@labserver ~]$ vi secret_data

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  secret_data  testfile  words.gz
dira  impfiles.tar  passwd              testcompany  words

[aadarsha@labserver ~]$ cat secret_data 
this is secret file
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vim passwd 

[aadarsha@labserver ~]$ vim passwd 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ alias vi='vim'

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vi secret_data 
 
[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vim .bashrc 

[aadarsha@labserver ~]$ sudo su
[sudo] password for aadarsha: 
[root@labserver aadarsha]# 

[root@labserver aadarsha]# vim .bashrc 
[root@labserver aadarsha]# exit
exit
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi secret_data 

# File Recovery after crash

[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  secret_data  testfile  words.gz
dira  impfiles.tar  passwd              testcompany  words

[aadarsha@labserver ~]$ rm -r newimpfiles.tar.gz words.gz impfiles.tar 
rm: remove write-protected regular file 'newimpfiles.tar.gz'? 
rm: remove regular file 'words.gz'? y
rm: remove write-protected regular file 'impfiles.tar'? y

[aadarsha@labserver ~]$ ls
dir1  dira  extracted  passwd  secret_data  testcompany  testfile  words
 
[aadarsha@labserver ~]$ ls
dir1  dira  extracted  passwd  secret_data  testcompany  testfile  words

# swap file or guard file --> created to protect the currently opening file when sudden incident like poweroff or terminate

# Vim uses a swap file while editing a buffer. It helps detect concurrent/stale editing sessions and provides recovery data after a crash or unexpected termination.

# recover from .swp file

# vim -r passwd.swp

[aadarsha@labserver ~]$ 
```