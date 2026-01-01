# ikanOS v5 Teaching Operating System

(Pronounced ee-ka-NOS, formerly SimpleOS v1-4. Greek for "sufficient" or "capable".)

Dan O’Malley

This project is based on versions 1-4 of [SimpleOS](https://github.com/meuer26/SimpleOSv4_Student) by Dan O'Malley (built during 2023-2026) but was enhanced and written with Grok (xAI) for version 5. This was a large-scale attempt to see how team programming with an AI in late 2025, early 2026 could enhance a significant low-level project. Overall, Grok did well on high-level C code but required significant assistance with the hardware-software interface and low-level debugging. It also seemed to lose context on very large files and required significant assistance for large files. It took roughly 120 hours to move from v4 to v5 of this project with Grok's help.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Features

- 32-bit x86, monolithic, multi-tasking, Plan 9-inspired kernel (60% Plan 9 minimalism, 30% Unix file structure, 10% laziness/organic coding that should be refactored)
- 24 MB of physical RAM
- 16 MB virtual address space per process. 8 MB user space, 8 MB kernel space. Kernel space is identity mapped.
- EXT2 Filesystem, 2 KB blocks, 32 MB disk image, Fully associative, in-kernel LRU disk cache
- AP processor started (SMP)
- ELF loader loads all ELF sections that are marked for loading.
- Named pipes in kernel for IPC and network pipes (UDP send and UDP receive)
- VGA text mode 80x50, color, custom 8x8 character font (changeable)
- PC speaker sounds
- Allows 32 running processes
- Basic semaphores, scheduler, and kernel log for syscalls
- Basic shell, top, line text editor, cat, grep, and scripting language
- Kernel and File Explorer tool (view kernel global objects table, pipes, sockets (network pipes), file text viewer, user space and kernel buffer statuses, memory hex viewer and disassembler, disk and block viewer, inode and ELF header viewer)
- Custom C Library
- Minimal C compiler 
- Minimal assembler (that outputs in ELF format)
- Can code, edit, compile, assemble, and run basic programs (including calling C library functions) all within the OS
- Basic file operations, compilation, and assembly test suite
- Simplified Text Adventure game
- Basic dungeon crawler



## Key Architectural Features

- Kernel Design: Monolithic kernel with a focus on essential OS primitives. It includes basic semaphores for synchronization, a scheduler for task switching, and a kernel log for tracking syscalls (up to 768 events). The kernel supports up to 32 concurrent processes and uses a global object table (GOT) to manage open objects like files, buffers, pipes, and sockets uniformly.

## Memory Management
- Virtual memory with paging: 8 MB user space (0x0–0x7FFFFF) and 8 MB kernel space (0x800000+). User heap starts at 0x600000, stack at 0x7FE000.
- Frame allocation uses a page frame map with first-fit strategy Processes can request specific or available pages via mmap syscall.
- Supports ELF loading, mapping text/data sections to preferred addresses.
- Kernel caches (e.g., fully associative LRU disk cache) and semaphores for mutual exclusion on shared structures.

## File System
- EXT2-based with 2 KB blocks. Supports basic operations like create, delete, move, chmod, and directory navigation. Files, pipes, and network sockets are treated as unified "objects" in the GOT. Includes inode/directory structures, block group descriptors, and tools for viewing inodes, ELF headers, disk sectors/blocks.

## Processes and Scheduling
- Multi-tasking with context switching via interrupts. Each process has a task struct (192 bytes) tracking PID, PPID, state, registers, file descriptors (up to 32), and more.
- Scheduler is interrupt-driven (150 Hz via PIT timer) and supports priorities, nice values, sleep, and wait syscalls.
- Syscalls include exec (loads ELF binaries), exit (with codes), kill, fork-like exec with parent switching, and ps/top for monitoring.
Supports up to 32 processes; zombies are cleaned via free syscall.

## Interrupts and Hardware Interaction
- IDT setup with PIC remapping to avoid conflicts. Handles system interrupts, timer for scheduling/uptime, and keyboard input.
- Syscalls triggered via interrupts (e.g., int 0x80) from ring 3, with handlers for sound, I/O, memory dumps, etc.
- Ring 3 transition via IRET method for user programs.

## Networking
- Basic UDP over NE2000-compatible NIC (base port 0x300). Uses named pipes/sockets for IPC and network I/O.
Supports send/receive with IP/UDP headers, checksums. Tracks packets transmitted/received.
- Buffers: Incoming receive (20 KB) and payload (20 KB).

## User Interface and Hardware Support
- VGA text mode: 80x50 resolution, color, custom 8x8 font (loadable). Cursor control, screen clearing, and printing functions.
- PC speaker for basic sounds (via PIT channel 2).
- Keyboard input via scan codes, with ASCII mapping and shift handling.
- No graphics; all text-based.


## Built-in Tools and Applications
- Shell and Utilities: Basic shell with commands like cat, grep, dir/ls, cd, mv, rm, chmod. Includes a line text editor, scripting language, and test suites for file ops/compilation.
- Monitoring Tools: ps/top for processes, uptime, memory dump/disassembly, open files viewer, global objects (files/pipes/sockets) viewer, kernel log viewer, disk/sector/block explorer, inode/ELF header viewers.
- Development Environment: Self-hosting capabilities—edit, compile (minimal C compiler), assemble (ELF output), and run programs within the OS using custom libc. Supports calling libc functions in user programs.
Games/Ports: Simplified Adventure game and a basic dungeon crawler.
- LibC: Custom user-space library with string utils (strcmp, strlen, etc.), heap allocation (fixed-size objects), printf/scanf, time conversions (Unix epoch), rand, and wrappers for all syscalls (e.g., open, read, write, exec, mmap).

## Syscalls and API

- 39 syscalls (e.g., open/close/read/write, exec/exit/kill, mmap/free, net send/rcv, sound, dir/cd/mv/chmod, logs/dumps).
Parameters passed via structs (e.g., file/network params) for flexibility.


## Attribution

Versions 1-4 of this kernel (when it was named SimpleOS) were written by Dan O'Malley (during 2023-2026). For Version 5, Dan O'Malley had Grok (xAI) add to the C library, develop functions to easily display memory, disassemble machine code, enhance file system operations, as well as create the edlin text editor, the basic c compiler, the basic assembler, and some games. This allowed a much more full-featured userspace for students while Dan O'Malley's kernel has largely remained the same.

In Version 5, the following files were written by Dan O'Malley:
- bootloader-stage1.asm
- bootloader-stage2.cpp
- cat.cpp
- constants.h
- exceptions.cpp
- file.cpp
- frame-allocator.cpp
- init.cpp
- interrupts.cpp
- kernel.cpp
- myprog.cpp
- schedule.cpp
- sound.cpp
- top.cpp
- trap.cpp
- vm.cpp
- assembler.sh
- code.sh
- compiler.sh
- file.sh
- prog1.c
- prog2.c
- test1.s
- testall.sh

The following files are based on earlier versions of Dan O'Malley's code but extended with Grok:

- fs.cpp had functions created and extended with Grok. These are noted inline in that file.
- sh.cpp and ex.cpp were written by Dan O'Malley but argument checking tweaks/tokenization, keyboard enhancement as well as command history were written by Grok.
- keyboard.cpp was written by Dan O'Malley but extended with new keys by Grok.
- screen.cpp had one function written by Grok (switchTo80x50Mode), the rest by Dan O'Malley.
- second_proc_start.asm was based on bootloader-stage-1 but extended with Grok.
- Most syscall.cpp code written by Dan O'Malley. Grok wrote some functions and extended others. These are noted inline.
- Most x86.cpp code written by Dan O'Malley. Grok wrote one function (startApplicationProcessor) and it is noted inline.
- libc-main.cpp is a combination of Dan O'Malley and Grok. These are noted inline.
- Makefile was written by Dan O'Malley but extended with Grok to support incremental builds.

The following files/portions were entirely written by Grok:
- adventure.cpp
- al.cpp
- cc.cpp
- dungeon.cpp
- edlin.cpp
- grep.cpp
- net.cpp
- src.cpp
- filetst.cpp
- inputtst.cpp
- Most of the code comments and Markdown descriptions were generated by Grok.

## Installation

This project requires Ubuntu 22.04 LTS as well as the programs called in Makefile.