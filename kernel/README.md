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

```
uname -a
Linux ubuntu-bionic 4.15.0-163-generic #171-Ubuntu SMP Fri Nov 5 11:55:11 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

```
sudo apt install \
build-essential bc bison flex libssl-dev \
libelf-dev libssl-dev libncurses5-dev
```

get source code
```
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.19.225.tar.xz
tar xvf ./linux-4.19.225.tar.xz
```

prepare config
```
cd linux-4.19.225
# make defconfig
# make menuconfig
cp /boot/config-4.15.0-163-generic .config
make syncconfig # all enter
```

build & install

```
make -j `nproc`
sudo make modules_install
sudo make install
```

failed in `make` step.

```
make[1]: *** No rule to make target 'debian/canonical-certs.pem', needed by 'certs/x509_certificate_list'.  Stop.
Makefile:1784: recipe for target 'certs' failed
make: *** [certs] Error 2
```

https://askubuntu.com/questions/1329538/compiling-the-kernel-5-11-11

- `make modules_install` -> built module files are installed in `/lib/modules/<kernel release number>`
- `make install` -> call `installkernel` command -> the kernel image file will be copied under `/boot`

```
uname -r
```

https://blog.masu-mi.me/post/2020/12/10/observe-linux-over-qemu-with-gdb/

やりなおし

https://app.vagrantup.com/boxomatic/boxes/debian-11

```
vagrant init boxomatic/debian-11
vagrant up
vagrant ssh
```

```
sudo apt-get update
sudo apt-get install -y kernel-package # failed
```

```
uname -rn
debian 5.10.0-10-amd64
sudo apt-get install linux-source-5.10
```

```
cd /usr/src
sudo tar xf linux-source-5.10.tar.xz
cd linux-source-5.10 # move to /usr/src/linux-source-5.10
```

Copy current config
```
sudo cp /boot/config-5.10.0-10-amd64 .config
```

Install ncurses

```
sudo apt-get install libncurses5-dev
sudo apt-get install libssl-dev libelf-dev
```

Check manuconfig
```
make menuconfig # exit
```

Change kernel version in Makefile

```
vi Makefile # EXTRAVERSION = -test-version
```

build

```
sudo make
```

Set `CONFIG_SYSTEM_TRUSTED_KEYS=""`

https://qiita.com/OPySPGcLYpJE0Tc/items/f0120d7274216172aedc#2%E3%82%AB%E3%83%BC%E3%83%8D%E3%83%AB%E3%81%AE%E8%A8%AD%E5%AE%9A

```
cat .config | grep "CONFIG_SYSTEM_TRUSTED_KEYS"
CONFIG_SYSTEM_TRUSTED_KEYS=""
```
## 4. Task Scheduler

Linux can process multiple tasks at the same time even if a machine just has one CPU. A task scheduler decides how long each CPU executes a task.

1. **Round-Robin Scheduling**: same time slice for all tasks
1. **O(1) Scheduler**: 2003 Linux 2.6.0 ~2007 Linux 2.6.22 based on the priority of a task (nice value and sleep time)
1. **Completely Fair Scheduler (CFS)**: Since Linux 2.6.23, choose a task with the minimum **vruntime**. Data structure is a red–black tree.
    - You can check vruntime value in `/proc/<process_id>/sched`.

## 5. Virtual Memory

### 5.1. 64 bit x86 processor memory address

Virtual memory follows its CPU memory management unit.

1. **linear address (virtual address)**: address assigned to virtual memory realized by paging.
    - if page size is 4k, page table is 4 layer.
1. **logical address**: address assign to virtual memory by segment (usually not used in 64-bit program)
1. **physical address**: address assign to physical memory (max 52 bit)

```
|--------| ffffffffffffff
| kernel |
|--------|
|        |
|        |
|--------|
| process|
|--------| 00000000000000
```

- a linear address is assigned to a process
- kernel area cannot be accessed by a process
- PTI (Page Table Isolation): assign kernel area only when switching to kernel mode
- In kernel area, there is an area that is straight mapping to the whole physical memory.
- physical memory maximum size can be checked by `MAX_PHYSMEM_BITS` in `arch/x86/include/asm/sparsemem.h`
- You can check
    - memory map of a process in `/proc/<process_id>/maps`.
    - memory for file read by yourself in `/proc/self/maps`.
    - mapping between virtual and physical memory in `/proc/<process_id>/pagemap`.
    - usage of memory for kernel: need to build with `CONFIG_X86_PTDUMP=y`...

### 5.3. get physical address by tracking page table

- cr3 -> PGD offset -> PUD offset -> PMD offset -> page table offset -> physical page offset
- the information of the location in PGD table is stored in `mm->pgd`
- to switch a task to execute, set the member variable's value to `cr3 register`, which switch the virtual space.
- convert virtual address into physical address: calculate by a module `pagetable`


## 6. Context Switch

When a task scheduler switches tasks to execute, it saves and restores the execution status of a task. This process is called **context switch**.

In this chapter:
1. **context switch**
1. user mode vs. kernel mode

### 6.1. what is context switch?

CPU level: the status of register -> store in task management area (task struct) or stack in memory:
1. **general register** that stores temporary data
1. **stack pointer** to store the location of a pointer
1. **program counter** to store execution point
1. **register** to store a location in page table.

### 6.2. Process of context switch

1. `__schedule()` in `kernel/sched/core.c` triggers context switch.
1. In `context_switch(struct rq *rq, struct task_struct *prev, struct task_struct *next, struct rq_flags *rf)`
    1. `switch_mm_irqs_off(prev->active_mm, next->mm, next)` in the context of switching to user (no need to call for switching to kernel): set the data pointing the pgd table for the task.
    1. `switch_to(prev, next, prev)`
1. Call `__switch_to_asm` in `switch_to` function
    1. store the data of register to the existing stack
    1. swtich stack (rsp resgister -> task struct thread.sp & next task struct thread.sp -> rsp register)
    1. load the data from stack to register
    1. jump `__switch_to`
1. `__switch_to`: save and restore register for segment or register for FPU(float processing unit)


### 6.3. Check register value while context switching

not completed.

prepare vagrant
```
vagrant init ubuntu/bionic64
vagrant up
vagrant ssh
```

install
```
sudo apt update
sudo apt install qemu gdb busybox-static
```
prepare
```
mkdir context-switch-1 && cd context-switch-1
```

```
vagrant@ubuntu-bionic:~/context-switch-1$ mkdir bin dev proc sbin sys
vagrant@ubuntu-bionic:~/context-switch-1$ cp /bin/bu
bunzip2  busybox
vagrant@ubuntu-bionic:~/context-switch-1$ cp /bin/bu
bunzip2  busybox
vagrant@ubuntu-bionic:~/context-switch-1$ cp /bin/busybox bin/
vagrant@ubuntu-bionic:~/context-switch-1$ cd bin/
vagrant@ubuntu-bionic:~/context-switch-1/bin$ ln -s busybox sh
vagrant@ubuntu-bionic:~/context-switch-1/bin$ ln -s busybox mount
vagrant@ubuntu-bionic:~/context-switch-1/bin$ cd ../sbin/
vagrant@ubuntu-bionic:~/context-switch-1/sbin$ cat <<EOF >init
> #!/bin/sh
> mount -t proc none /proc
> mount -t sysfs none /sys
> exec /bin/sh
> EOF
vagrant@ubuntu-bionic:~/context-switch-1/sbin$ cd ../dev
vagrant@ubuntu-bionic:~/context-switch-1/dev$ sudo mknod console c 5 1
vagrant@ubuntu-bionic:~/context-switch-1/dev$ sudo mknod null c 1 3
vagrant@ubuntu-bionic:~/context-switch-1/dev$ cd ..
vagrant@ubuntu-bionic:~/context-switch-1$ find . | cpio -o -H newc | gzip > ~/initramfs.img
4032 blocks
```

```
sudo apt install make
```

prepare `gdbcom` script file.

```
target remote localhost:12345

source .

symbol-file vmlinux

b __switch_to_asm
la src

c
```

### 6.4. kernel mode vs. user mode

1. program in user mode cannot access memory for kernel
1. program in kernel mode can access to any memory area

switching mode:
1. process executes systemcall: user mode -> kernel mode -> user mode
1. when task scheduler changes tasks to execute: user mode -> kernel mode -> user mode


```
|--------|
| kernel |
|--------|
|        |
|        |
|--------|
| process|
|--------|
```

### 6.5. PTI (Page Table Isolation)

From Linux 4.15

- In user mode, memory area for kernel is not mapped to virtual memory space.
- Prepar two PGD tables (for user mode and kernel mode)

## 7. Physical memory management

Reason for necessity of physical memory management:
- direct memory mapping (kernel area)
- Direct Memory Access (DMA)
- Memory fragmentation (internal: small data in page, external: fragmented page location)

Buddy system (memory allocator):
- memory with different size 2^n (e.g. 1-block, 2-block, 4-block) and allocate proper sized one when requested.

## 8. Filesystem

- bytearray
- file info other than filename is stored in `inode`
- file types:
    - `-`: normal
    - `d`: directory <- this is also a file
    - `l`: symbolic link
    - `p`: pipe
    - ...
## References
- [The Linux Kernel documentation](https://www.kernel.org/doc/html/v4.13/index.html)
