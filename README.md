# How Linux Works

## Overview

### Kernel

- kernel
  - is _in memory_, telling CPU where to look for its next task
  - runs in **kernel mode**
- **kernel space** - the memory area only accessible by kernel
- **user space** - part of the main memory accessible by user processes
- kernel can run **kernel threads**
  - like processes but have access to kernel space
  - `kthreadd`
  - `kblockd`
- Kernel manages:
  - processes
    - which processes are allowed to use the CPU?
  - memory
  - device drivers
  - system calls and support
    - processes _normally_ use system calls to communicate with kernel

#### Process Management

- processes run _simultaneously_
  - however they do not run at _exactly_ the same time
  - each process uses the CPU for a **time slice**
  - **context switch** - kernel's responsibility
    - _when_ does the kernel run?
    - kernel runs _between_ process time slices during a **context switch**

#### Memory Management

- kernel manages memory during **context switch**
- modern CPUs include a **Memory Management Unit (MMU)**
  - enabling **virtual memory**
  - **page table**

#### System Calls and Support

- `fork()`
  - kernel creates a _nearly identical_ copy of the process
- `exec(program)`
  - kernel loads & starts `program`, replacing the current process
- other than init, _all_ new user processes start as a result of `fork()`
  - _then_, probably runs `exec()` to start a new program
- **pseudodevices**
  - look-alike devices to user processes
  - but implemented purely in software
  - e.g. `/dev/random` - the kernel random number generator device

### User Space

- memory for the entire collection of running processes
- **userland**

### Users

- every user-space process has a user **owner**

## Basic Commands and Directory Hierarchy

### Commands

- `/bin/sh` - the Bourne Shell
- `bash` - the **Bourne Again Shell**
- `Ctrl-D` vs. `Ctrl-C`
  - `Ctrl-D` stops the current standard input entry from terminal with an `EOF` message
  - `Ctrl-C` terminates program regardless of its input/output
- `rmdir` - removes a `dir`
  - fails when `dir` is _NOT_ empty
- **Globbing**
  - to match _all_ files: `*`
  - `?` - match exactly one character
  - `'*'` - if you do not want the shell to expand a glob
  - Shell performs expansions _before_ running commands, and only then
- `grep`
  - `grep root /etc/passwd` - print lines in `/etc/passwd` that contain `root`
  - handy when operating on multiple files - it prints filename in addition to the matching line
  - Options:
    - `-i` - case insensitive
    - `-v` - inverse search
    - `-e` & `-E` pattern
      - `.*` - any number of characters, including none
      - `.+` - 1 or more
      - `.` - exactly one
- `less`
  - one screenful at a time
  - search for text inside `less`
    - `/word` - search forward
    - `?word` - search backward
- `pwd`
  - `-P` - avoid all symlinks
- `diff` - difference between two text files
  - `diff -u`
- `file` - format of a file
- `find`
  - `$ find dir -name file -print`
    - find `file` in `dir`
    - `-name` - pattern, which should _NOT_ include a slash `/`
    - `-print` - print full file name on stdin, followed by newline
    - `-print0` - print followed by null (instead of newline)
- `locate` - similar to `find` but searches against a pre-built index
  - _not_ real time
- `tail` & `head`
- `sort` - put lines of a text file in alphanumeric order
- `chsh` - change your shell

### Environment and Shell Variables

- _ALL_ processes on Unix systems have environment variable storage
- OS passes _all_ shell's env variables to programs run by shell

### Command Path

- separated by `:`

### Manual

- Manual page followed by **section number**
- `man <section_number> passwd`
- `info <command>`

### Shell I/O

- pipe `|`
- stderr to stdout - `2>&1`

### Processes

- `ps`
  - `TTY` - the terminal device where the process is running
  - `ps x` - show all _your_ running processes
  - `ps ax` - show _ALL_ processes _on the system_
- `$$` - current shell's PID

#### Process Termination

- `kill` - sends `TERM`
- `kill -STOP <PID>` - freeze (instead of terminating)
  - process still in memory
- `kill -CONT <PID>` - run the stopped process _again_
- `Ctrl-C` sends an `INT` signal
- `kill -KILL`
  - send the `KILL` signal
  - `kill -9`

#### Job Control

- `TSTP` & `CONT` signals
- If a program tries to read from stdin while it's in the background, it can freeze/terminate
  - try to `fg` to bring it back

### File Modes and Permissions

#### Permissions

- `s` permission (instead of `x`): the executable is **setuid**
  - when the program is executed, it runs as though the file owner is the user instead of you
  - run as root in order to get the privileges needed to change system files
  - e.g. `paswd` - needs to change `/etc/passwd`
- To change,
  - `chmod <group/user>+<permission> file`
  - `chmod <group/user>-<permission> file`
- You can _only_ access a file in a directory _if_ the directory is executable
- `umask` - applies a predefined set of permissions to any new file created by you
  - `<mask>` - permission bits that should _NOT_ be set on a newly created file
  - **logical complement**!!!
  - sets to `<mask> & 0777`
  - Use `umask 022` if you want everyone to be able to see all files & directories
  - Use `umask 077` if you do _NOT_ want anyone to be able to see all files & directories

#### Symbolic Links

- `<file_pointed_to>` does _NOT_ have to mean anything
  - does _NOT_ need to exist
- from `target` to `linkname`
  - `ln -s target linkname`
  - `target` - file/directory `linkname` points to

### Archiving and Compressing Files

- `gzip` and `tar`

#### `gzip`

- _ONLY_ compress, does _NOT_ archive
  - i.e. does _NOT_ pack multiple files/directories into a single one
- to compress, `gzip <file>`
- to unzip, `gunzip <file.gz>`

#### `tar`

- to create an archive - `tar cvf <archive>.tar file1 file2 ...`
- Flags
  - `c` - create mode
  - `v` - verbose
  - `f` - file option
    - followed by the file name to create
    - to use stdin/stdout, set filename to `-`
  - `x` - extract mode
- To unpack `.tar` file, `tar xvf <archive>.tar`
  - does _NOT_ remove the archived `.tar` file after extraction
- **Table-of-Contents Mode**
  - using flag `t` instead of `x`
  - check the contents of a `.tar` file _before_ unpacking
- Consider using `-p` when _unpacking_
  - preserves permissions
  - default under superuser

#### `.tar.gz` - Compressed Archives

- `gunzip` first
- then `tar xvf`

#### `zcat`

- Combine archival and compression functions with a pipeline
- `zcat file.tar.gz | tar xvf -`
- `zcat` - `gunzip -dc`
  - `-d` - decompress
  - `-c` - send result to standard output
- `tar` has a shortcut for `zcat`
  - `tar ztvf file.tar.gz`
- `.tgz` file === `.tar.gz` file

### Linux Directory Hierarchy

- `/usr` - where most of the user space programs and data reside
  - `/usr/local`
  - `/usr/share`
- Kernel location
  - `/vmlinuz` or `/boot/vmlinuz`
  - boot loader loads this file into memory and sets it in motion when system boots
- Once boo loader starts the kernel, the main kernel file is no longer used by the running system
  - however, `loadable kernel modules` - modules loaded/unloaded by kernel
  - `/lib/modules`

### Superuser

- `/etc/sudoers`
- Use `visudo` to edit `/etc/sudoers`
- checks for syntax errors _after_ saving the file
- To check `sudo` logs, `journalctl SYSLOG_IDENTIFIER=sudo`
-
