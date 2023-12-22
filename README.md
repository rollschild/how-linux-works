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

## Devices

### Device Files

- **device files** a.k.a. **device nodes**
  - under `/dev`
- `echo blah > /dev/null`
- File modes for devices:
  - `b` - block
  - `c` - character
  - `p` - pipe
  - `s` - socket
- Major vs. minor device numbers
- _NOT all_ devices have device files
  - e.g. network interfaces

#### Block Device

- data accessed in fixed chunks
- quick random access
- fixed size
- disks

#### Character Device

- data streams
- no fixed size
- printer

#### Pipe Device

- like character devices, but
  - with _another process_ at the other end of the I/O stream instead of kernel driver

#### Socket Device

- special-purpose interfaces for interprocess communication
- outside of `/dev`

### sysfs

- kernel assigns devices in the order in which devices are found
  - a device may have different names between reboots!
- the **sysfs** interface - provided by kernel
  - to provide uniform view for attached devices based on their _actual_ hardware attributes
  - path under `/sys/devices/`
- `/dev/` enables user processes to use the device
- `/sys/devices/` is used to view information and manage the device
- Use `udevadm` to show the sys path of a device under `/dev/`
  - `udevadm info --query=all --name=/dev/sda`

### `dd`

- `dd`
  - read from an input file/stream
  - write to an output file/stream
- `dd` copies data in blocks of fixed size
- uses an old IBM Job Control Language (JCL) syntax

### Device Name Summary

- To find the name of a device (when partitioning a disk)
  - query udevd using `udevadm` - the _ONLY_ reliable way
  - look for the device under `/sys/`
  - guess from the output of `journalctl -k`
    - prints kernel messages
  - guess from the output of kernel system log
  - if disk device already visible to system, check output of `mount`
  - run `cat /proc/devices` to see block/character devices for which the system has drivers

#### Hard Disks `/dev/sd*`

- `/dev/sda`, `/dev/sdb` etc - _entire_ disks
- `/dev/sda1`, `/dev/sda2`, etc - **partitions**
- `sd` - SCSI disk
  - **Small Computer System Interface (SCSI)**
- `lsscsi` - list SCSI devices
- Linux uses **Universally Unique Identifier (UUID)** and **Logical Volume Manager (LVM)** to maintain stable disk device mapping

#### Virtual Disks `/dev/xvd*` & `/dev/vd*`

- for virtual machines

#### Non-Volatile Memory Devices: `/dev/nvme*`

- talking to solid state storage
- `nvme list`

#### Device Mapper `/dev/dm-*` & `/dev/mapper/`

- **LVM**: a level up from disks and other direct block storage on some systems

#### CD/DVD drives `/dev/sr*`

- optical storage drives _might_ show up as PATA devices
- `/dev/sr*` devices are _read only_
- To write/rewrite optical devices, use "generic" SCSI devices such as `/dev/sg0`

#### PATA Hard Disks `/dev/hd*`

- **PATA (Parallel ATA)**
- `/dev/hda`, `/dev/hdb`, `/dev/hdc`, `/dev/hdd`
- If a SATA device recognized as PATA - it's running in compatibility mode
  - hindered performance
  - check your BIOS

#### Terminals: `/dev/tty*`, `/dev/pts/`, `/dev/tty`

- **Terminal** - device for moving characters between a user process and an I/O device
- _Most_ terminals are **pseudoterminal** devices
- Two common terminals:
  - `/dev/tty1` - the first virtual console
  - `/dev/pts/0` - the first pseudoterminal device
- `/dev/tty` - the controlling terminal of the current process
- use `getty` to launch a virtual console?
- force changing console: `# chvt 1` - switch to `tty1`

#### Audio Devices: `/dev/snd/*`, `/dev/dsp`, `/dev/audio`, etc

- Two sets of audio devices
  - **Advanced Linux Sound Architecture (ALSA)** - in `/dev/snd/`
  - **Open Sound System (OSS)**
    - computer will play any WAV file sent to `/dev/dsp`

#### Device File Creation

- You normally do _NOT_ create device files
  - created by **devtmpfs** and **udev**
- To manually create:
  - `mknod` - creates one device
    - `# mknod /dev/sda1 b 8 1`
      - block
      - major number 8
      - minor number 1
- Each system has a `MAKEDEV` program in `/dev/` to create groups of devices

### udev

#### devtmpfs

- the **devtmpfs filesystem** developed in response to the problem of device availability during boot
- kernel create device files as necessary, but also notifies udevd a new device is available
- udevd, upon receiving the signal, does _not_ create the device files, but
  - performs device initialization
  - sets permissions
  - notifies other processes that new devices are available
  - creates a number of symbolic links in `/dev/`
    - look for them in `/dev/disk/by-id/`
- the **tmp** in devtmpfs:
  - the filesystem resides in main memory,
  - with read/write capability by user-space processes

#### udevadm

- admin tool for udevd
- search for and explore system devices
- monitor uevents as udevd receives them from kernel

#### Device Monitoring

- `udevadm monitor`
- `udevadm monitor --kernel --subsystem-match=scsi`
  - see only kernel messages pertaining to changes in SCSI subsystem
- **udisksd** - daemon that listens for events in order to
  - automatically attach disks
  - notify other processes that new disks are available

### SCSI and Linux Kernel

- computer <-> SCSI Host Aapter <-> Devices
- **Serial Attached SCSI (SAS)**
  - newer version of SCSI
  - better performance
- Most likely USB storage devices that use SCSI commands
- SATA disks appear as SCSI devices
  - but most of them communicate through a translation layer
- NVMe devices are _NOT_ SCSI
  - but they could show up in `lsscsi` as adapter number `N`
- For any given device file on the system, kernel _almost always_ uses
  - one top-layer driver, and
  - one lower-layer driver

#### USB Storage and SCSI

- Linux kernel includes a three-layer USB subsystem closely resembling the SCSI subsystem
  - device-class driver
  - bus management core
  - host controller driver
- `lsusb`

#### Generic SCSI Devices

- `lsscsi -g`
