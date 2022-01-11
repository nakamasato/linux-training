# 動かしながら学ぶLinuxカーネルの教科書

## 1. Basics of Linux Kernel

### 1.1. What is Linux kernel?

> the main component of a Linux operating system (OS) and is the core interface between a computer’s hardware and its processes.

Main kernel's tasks:
1. **Memory management**: Keep track of how much memory is used to store what, and where
1. **Process management**: Determine which processes can use the central processing unit (CPU), when, and for how long
1. **Device drivers**: Act as mediator/interpreter between the hardware and processes
1. **System calls and security**: Receive requests for service from the processes

https://www.redhat.com/en/topics/linux/what-is-the-linux-kernel

`man syscall` <- You can check system calls by this command.

Linux is a name of **kernel**.

Kernel + libraries, commands, and GUI = Linux distributiion.

### 1.2. The meaning of studying Linux kernel

Efficiently use Linux.

### 1.3. Kernel is event-driven

Event types:
1. Hardware [Interrupts](https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html#)
1. Software Interrupts
1. System calls

interrupts vector -> interrupts handler -> the previous execution flow is resumed.

### 1.4. Task management

**Time sharing system (TSS) (時分割処理)**: One CPU is assigned to each of running processes for a certain amount of time.

### 1.5. Memory Management

- **Virtual Address Space** for each process. Processes cannot access to another process's information.
- **Physical Address Space**
- **Paging**: Linux allocates memory to processes by dividing the physical memory into pages, and then mapping those physical pages to the virtual memory needed by a process. `x86` processor uses 4K bytes.
    - **Page Table**: Mapping of virtual address and physical address
    - Physical page is assigned when actually needed to save memory.
    - Memory swapping: Write infrequently used contents in memory to disk based on LRU to free up memory
- **OOM Killer**

### 1.6. Device Management

Abstrat all devices as a **file**.

application <-> device file <-> device driver

- character: in/output by 1 byte. no buffer
- block: data with a certain length as a unit. with buffer

### 1.7. Filesystem

Provide **file** interface to users and applications

user/application <-> VFS (Virtual File System) <-> File system

### 1.8. Networking

### 1.9. From loading to startup

- BIOS (Basic Input/Output System)
- UEFI (Unified Extensible Firmware Interface)
- Bootloader: load the kernel image file to memory e.g. GNU GRUB
- init disk image

## 2. Linux kernel module management

### 2.1. module

**loadable kernel module**: Separate each function as a module file, which can be loaded when needed

kernel image (code) -> kernel space (memory) -> execute

1. The size of kernel image can be kept small with loadable kernel modules.
1. Can choose which functions anytime

### 2.2. storage for module file

**Module directory**: `/lib/modules/<kernel release number>`

`uname -r`: you can check the release number of the running kernel

`.ko`: extension for module files

```find /lib/modules `--name -r` -name "*.ko"```

### 2.3. manage modules with command

- [kmod](https://man7.org/linux/man-pages/man8/kmod.8.html)
    - [modinfo](https://linux.die.net/man/8/modinfo) (install by `sudo apt install module-init-tools` in old distribution)
    - `lsmod`: loaded modules
    - `insmod`: e.g. ```sudo insmod /lib/modules/`uname -r`/kernel/driver/char/lp.ko```
    - `rmmod`: e.g. `sudo rmmod lp`. Not considering the dependencies.
    - `modprobe`:
    - `depmod`: check dependency of a module

### 2.4. Module Options

`sudo modprobe softdog soft_margin=300`

### 2.5. Auto loading

Modules are automatically loaded by the kernel itself or programms like **udevd** (userspace devicee management daemon)

Examples:
- A music player accesses to a device file, the kernel loads the device driver module if not loaded
- A USB is connected to a computer, the udevd receives the event and loads the device module automatically.

In either case, eventually `modprobe` is used to load a module.

### 2.6. Relate device and module

Check product info of connected devices

- `lspci`: PCI

    ```
    lspci -n
    00:00.0 0600: 8086:1237 (rev 02)
    00:01.0 0601: 8086:7000
    00:01.1 0101: 8086:7111 (rev 01)
    00:02.0 0300: 80ee:beef
    00:03.0 0200: 8086:100e (rev 02)
    00:04.0 0880: 80ee:cafe
    00:05.0 0401: 8086:2415 (rev 01)
    00:07.0 0680: 8086:7113 (rev 08)
    00:14.0 0100: 1000:0030
    ```

- `lsusb`: USB

## 3. Build a Linux kernel

You can customize kernel if you know how to build it. <- Not necessary for now.

## References
- [The Linux Kernel documentation](https://www.kernel.org/doc/html/v4.13/index.html)
