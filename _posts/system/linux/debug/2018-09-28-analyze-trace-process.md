---
layout: post
title: Analyze and trace linux program
description: debug method
tags: debug linux reverse-design
---
# analyze and trace a linux process

linux has utilities - strace, readelf, valgrind, objdump that can help us to debug or tracking a process

## 1. process or dynamic libraries analyze

use the readelf to analyze a executable or dynamic libraries, the readelf cat read information from the process/libraries:

* **header information** - contains machine, type, ... information
* **import libraries** - which libraries is used in the process/libraries
* **code sections** - some elf specific section information in the process/libraries
* **export symbols** - to show the export symbols information in the libraries

### 1.1. header information
use command to show header information:
```shell
readelf -h <file>
```

and it shows like:
```shell
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2''s complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x420560
  Start of program headers:          64 (bytes into file)
  Start of section headers:          1035672 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         29
  Section header string table index: 28
```
in this example, it shows the process is a executable file, machine is x86-64 and it is a 64 bits process

### 1.2. import libraries and code sections
use command as:
```shell
readelf -d <file>
```

and the output shell like:
```shell
Dynamic section at offset 0xf3e08 contains 26 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libtinfo.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libdl.so.2]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0x41d5e0
 0x000000000000000d (FINI)               0x4ba284
 0x0000000000000019 (INIT_ARRAY)         0x6f3df0
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x6f3df8
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x400298
 0x0000000000000005 (STRTAB)             0x412230
 0x0000000000000006 (SYMTAB)             0x404b38
 0x000000000000000a (STRSZ)              35923 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x6f4000
 0x0000000000000002 (PLTRELSZ)           5088 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x41c200
 0x0000000000000007 (RELA)               0x41c140
 0x0000000000000008 (RELASZ)             192 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffffe (VERNEED)            0x41c070
 0x000000006fffffff (VERNEEDNUM)         3
 0x000000006ffffff0 (VERSYM)             0x41ae84
 0x0000000000000000 (NULL)               0x0
```

it shows the process/libraries import the libtinfo.so, libdl.so, libc.so libraries and the code section likes init, fini, init_array, ... start address, that means we can read the process actions before and after main function entry.

e.g. we can use the hexdump command to read the more details for sections in process

### 1.3. export symbols

it usually be used in shared objects (dynamic libraries), use the command as:
```shell
readelf -s <file>
```
it shall show likes:
```shell
Symbol table '.dynsym' contains 121 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __snprintf_chk@GLIBC_2.3.4 (15)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND free@GLIBC_2.2.5 (16)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __errno_location@GLIBC_2.2.5 (16)
     4: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
     5: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND write@GLIBC_2.2.5 (16)
     6: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strlen@GLIBC_2.2.5 (16)
     7: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail@GLIBC_2.4 (17)
     8: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND snprintf@GLIBC_2.2.5 (16)
     9: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memset@GLIBC_2.2.5 (16)
    10: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND close@GLIBC_2.2.5 (16)
    11: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memchr@GLIBC_2.2.5 (16)
    12: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND read@GLIBC_2.2.5 (16)
    13: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    14: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memcpy@GLIBC_2.14 (18)
    15: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND malloc@GLIBC_2.2.5 (16)
    16: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __vsnprintf_chk@GLIBC_2.3.4 (15)
    17: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND open@GLIBC_2.2.5 (16)
    18: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND lseek64@GLIBC_2.2.5 (16)
    19: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    20: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strerror@GLIBC_2.2.5 (16)
    21: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize@GLIBC_2.2.5 (16)
    22: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  ABS ZLIB_1.2.2
    23: 000000000000d300   118 FUNC    GLOBAL DEFAULT   13 inflateEnd
    24: 000000000000ad30   240 FUNC    GLOBAL DEFAULT   13 inflateInit2_
    25: 0000000000002b20  1115 FUNC    GLOBAL DEFAULT   13 crc32_z@@ZLIB_1.2.9
    26: 0000000000006050  6439 FUNC    GLOBAL DEFAULT   13 deflate
    27: 0000000000005e40   150 FUNC    GLOBAL DEFAULT   13 deflateTune@@ZLIB_1.2.2.3
    28: 0000000000005cf0   328 FUNC    GLOBAL DEFAULT   13 deflatePrime@@ZLIB_1.2.0.8
.......
```

you can see the symbols type (FUNC is function), bind (global is global scope), ndx has number is the export symbols

## 2. objects (code instance, symbols table, ...)

some times we need trace a error from an address, in this case we can use the objdump to help us:

* **disassemble** - dump a disassemble source code, help us to narrow down the root cause
* **dump resource** - dump the string resource from the binary

### 2.1. disassemble

it can get the function address and disassemble code, use the command as:
```shell
objdump -gD <file>
```

and the output will very large, it will contain about function address and disassembly code

### 2.2. dump string resource

sometimes we need extract the string from binrary, use the command as:
```shell
objdump -sj .rodata <file>
```

the output shall like:
```shell
/bin/bash:     file format elf64-x86-64

Contents of section .rodata:
 cf4a0 01000200 64656275 67676572 00474e55  ....debugger.GNU
 cf4b0 20626173 682c2076 65727369 6f6e2025   bash, version %
 cf4c0 732d2825 73290a00 7838365f 36342d70  s-(%s)..x86_64-p
 cf4d0 632d6c69 6e75782d 676e7500 474e5520  c-linux-gnu.GNU
 cf4e0 6c6f6e67 206f7074 696f6e73 3a0a0009  long options:...
 cf4f0 2d2d2573 0a005368 656c6c20 6f707469  --%s..Shell opti
 cf500 6f6e733a 0a00092d 2573206f 72202d6f  ons:...-%s or -o
 cf510 206f7074 696f6e0a 0072756e 5f6f6e65   option..run_one
 cf520 5f636f6d 6d616e64 002d6300 72626173  _command.-c.rbas
 cf530 68004920 68617665 206e6f20 6e616d65  h.I have no name
 cf540 21003f3f 686f7374 3f3f0042 4153485f  !.??host??.BASH_
 cf550 454e5600 504f5349 584c595f 434f5252  ENV.POSIXLY_CORR
 cf560 45435400 504f5349 585f5045 44414e54  ECT.POSIX_PEDANT
 cf570 4943005c 732d5c76 5c242000 3e20007e  IC.\s-\v\$ .> .~
 cf580 2f2e6261 73687263 0025733a 20696e76  /.bashrc.%s: inv
 cf590 616c6964 206f7074 696f6e00 25632563  alid option.%c%c
 cf5a0 3a20696e 76616c69 64206f70 74696f6e  : invalid option
 cf5b0 006c6f67 696e5f73 68656c6c 00494e53  .login_shell.INS
 cf5c0 4944455f 454d4143 53002c74 65726d3a  IDE_EMACS.,term:
 cf5d0 00202874 65726d3a 0064756d 6200656d  . (term:.dumb.em
 cf5e0 61637300 50533100 50533200 5353485f  acs.PS1.PS2.SSH_
 cf5f0 434c4945 4e540053 5348325f 434c4945  CLIENT.SSH2_CLIE
 cf600 4e54002f 6574632f 62617368 2e626173  NT./etc/bash.bas
 cf610 68726300 2f657463 2f70726f 66696c65  hrc./etc/profile
 cf620 007e2f2e 70726f66 696c6500 7e2f2e62  .~/.profile.~/.b
 cf630 6173685f 70726f66 696c6500 7e2f2e62  ash_profile.~/.b
...
```

then, you can extract string from this output

## 3. trace system call

in case, run a process segmentation fault, we can use some methods to trace, one of them is use strace utility, it can do:

* **system call** - trace system call
* **network** - trace network feature
* **signals** - trace systeem signals

### 3.1. trace system call

use strace can trace the system call (open, close, read, write, ioctl) in a process, use command as:
```shell
strace -x -e trace=open,close,write,read,ioctl <file>
```

it will show as:
```shell
read(3, "###\n### Sample Wget initializati"..., 4096) = 4096
read(3, "ruct = off\n\n# You can turn on re"..., 4096) = 846
read(3, "", 4096)                       = 0
close(3)                                = 0
close(3)                                = 0
read(3, "\x54\x5a\x69\x66\x32\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x05\x00\x00\x00\x05\x00\x00\x00\x00"..., 4096) = 790
read(3, "\x54\x5a\x69\x66\x32\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x05\x00\x00\x00\x05\x00\x00\x00\x00"..., 4096) = 485
close(3)                                = 0
write(2, "--2018-09-28 17:03:41--  http://"..., 47--2018-09-28 17:03:41--  http://www.hinet.net/
) = 47
read(3, "# Locale name alias data base.\n#"..., 4096) = 2995
read(3, "", 4096)                       = 0
close(3)                                = 0
write(2, "Resolving www.hinet.net (www.hin"..., 43Resolving www.hinet.net (www.hinet.net)... ) = 43
close(3)                                = 0
close(3)                                = 0
read(3, "# /etc/nsswitch.conf\n#\n# Example"..., 4096) = 556
read(3, "", 4096)                       = 0
close(3)                                = 0
read(3, "# The \"order\" line is only used "..., 4096) = 92
read(3, "", 4096)                       = 0
....
```

you can see the read, write function be called with partial of data content, that can help you to trace the process progress

### 3.2 trace network actions
it also can only trace network actions, use command as:
```shell
strace -x -e trace=network <file>
```
the output will be:
```Shell
Resolving www.hinet.net (www.hinet.net)... socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 3
connect(3, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 3
connect(3, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
socket(AF_INET, SOCK_DGRAM|SOCK_CLOEXEC|SOCK_NONBLOCK, IPPROTO_IP) = 3
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("127.0.0.53")}, 16) = 0
sendmmsg(3, [{msg_hdr={msg_name=NULL, msg_namelen=0, msg_iov=[{iov_base="\x85\xfb\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x03\x77\x77\x77\x05\x68\x69\x6e\x65\x74\x03\x6e\x65\x74\x00\x00\x01\x00\x01", iov_len=31}], msg_iovlen=1, msg_controllen=0, msg_flags=0}, msg_len=31}, {msg_hdr={msg_name=NULL, msg_namelen=0, msg_iov=[{iov_base="\x30\x03\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x03\x77\x77\x77\x05\x68\x69\x6e\x65\x74\x03\x6e\x65\x74\x00\x00\x1c\x00\x01", iov_len=31}], msg_iovlen=1, msg_controllen=0, msg_flags=0}, msg_len=31}], 2, MSG_NOSIGNAL) = 2
recvfrom(3, "\x85\xfb\x81\x80\x00\x01\x00\x09\x00\x00\x00\x00\x03\x77\x77\x77\x05\x68\x69\x6e\x65\x74\x03\x6e\x65\x74\x00\x00\x01\x00\x01\xc0"..., 2048, 0, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("127.0.0.53")}, [28->16]) = 186
recvfrom(3, "\x30\x03\x81\x80\x00\x01\x00\x09\x00\x00\x00\x00\x03\x77\x77\x77\x05\x68\x69\x6e\x65\x74\x03\x6e\x65\x74\x00\x00\x1c\x00\x01\xc0"..., 65536, 0, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("127.0.0.53")}, [28->16]) = 282
socket(AF_NETLINK, SOCK_RAW, NETLINK_ROUTE) = 3
bind(3, {sa_family=AF_NETLINK, nl_pid=0, nl_groups=00000000}, 12) = 0
getsockname(3, {sa_family=AF_NETLINK, nl_pid=14698, nl_groups=00000000}, [12]) = 0

```
it shows the actions for networks process
