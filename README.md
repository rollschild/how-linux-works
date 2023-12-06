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
