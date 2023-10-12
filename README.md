# Linux x64 Reversing

Here is a step-by-step process on how I would reverse an x86-64 Linux binary!

## Example Binary

Here is the binary we are going to reverse today:

```c
#include <stdio.h>
#include <string.h>

#define LICENSE_KEY "uT54_cOn5O1E_COw60ys"

int check_license(const char *input_key) {
    return strcmp(input_key, LICENSE_KEY) == 0;
}

int main() {
    char input[50];
    
    printf("Enter your license key: ");
    fgets(input, sizeof(input), stdin);

    size_t len = strlen(input);
    if (len > 0 && input[len - 1] == '\n') {
        input[len - 1] = '\0';
    }
    
    if (check_license(input)) {
        printf("License key is valid!\n");
    } else {
        printf("License key is invalid.\n");
    }
    
    return 0;
}
```

### Static Analysis

#### a. Determine the File Type

First, we need to determine whether we are dealing with an executable, a shared library, or some other type of file.

***Terminal:***
```bash
dilldylanpickle@archlinux:~/Linux-x64-Reversing$ file license-key
license-key: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ded5e8f145e4e45c471b080d253dd02bfe98d056, for GNU/Linux 3.2.0, with debug_info, not stripped
```

Let's break down the output of the `file` command for our `license-key` binary:
> 1. ELF:

This stands for "Executable and Linkable Format" and it's the standard binary format for Unix systems.

> 2. 64-bit:

This indicates the architecture's bit-width. In this case, the binary is compiled for a 64-bit system.

> 3. LSB:

This stands for "Least Significant Byte" and it refers to the endianness of the data in the binary. LSB (or little-endian) means the least significant byte is stored first.

> 4. pie:

This stands for "Position Independent Executable" and this binary can be loaded into any memory address and still execute correctly. PIE is often used alongside ASLR (Address Space Layout Randomization) to increase security by randomizing the location of the process in memory.

> 5. x86-64:

This refers to the CPU architecture for which the binary was compiled. x86-64 (or AMD64) is the 64-bit version of the x86 instruction set.

> 6. version 1 (SYSV):

This specifies the ELF version and ABI (Application Binary Interface) and "SYSV" refers to the System V ABI.

> 7. dynamically linked:

The binary requires external shared libraries at runtime. This is in contrast to a statically linked binary, which contains all required libraries.

> 8. interpreter /lib64/ld-linux-x86-64.so.2:

This is the dynamic linker/loader's path, responsible for loading the binary's dependencies when it starts.

> 9. BuildID[sha1]=ded5e8f145e4e45c471b080d253dd02bfe98d056:

A unique identifier for the binary, which can be useful for debugging or referring to specific versions of binaries.

> 10. for GNU/Linux 3.2.0:

Indicates the minimum kernel version required to execute the binary.

> 11. with debug_info:

The binary contains debugging information. This makes it easier to analyze, reverse engineer, or debug using tools like `gdb`.

> 12. not stripped:

Symbol and debugging information hasn't been removed from the binary. Stripping a binary makes it smaller and removes function names and other symbols, but it also makes it harder to reverse engineer or debug.

#### b. Identify ASCII Strings

The `strings` command in Linux is used to extract and display printable strings from a binary or any file. When analyzing binaries, especially in the context of reverse engineering, security research, or malware analysis, the `strings` command provides a quick and valuable insight into the potential behavior, functionality, or origin of a binary.

***Terminal:***
```bash
dilldylanpickle@archlinux:~/Linux-x64-Reversing$ strings license-key
/lib64/ld-linux-x86-64.so.2
mgUa
__cxa_finalize
fgets
__libc_start_main
strcmp
puts
strlen
stdin
__stack_chk_fail
printf
libc.so.6
GLIBC_2.4
GLIBC_2.2.5
GLIBC_2.34
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
PTE1
u+UH
uT54_cOn5O1E_COw60ys
Enter your license key: 
License key is valid!
License key is invalid.
:*3$"
GCC: (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
_IO_buf_end
input_key
_old_offset
_IO_save_end
short int
size_t
input
_IO_write_ptr
_flags
_IO_buf_base
_markers
_IO_read_end
_freeres_buf
fgets
_lock
long int
printf
_cur_column
_IO_FILE
unsigned char
_IO_marker
_shortbuf
_IO_write_base
_unused2
_IO_read_ptr
short unsigned int
GNU C17 11.4.0 -mtune=generic -march=x86-64 -g -fasynchronous-unwind-tables -fstack-protector-strong -fstack-clash-protection -fcf-protection
main
strlen
_freeres_list
__pad5
_IO_codecvt
long unsigned int
_IO_write_end
__off64_t
__off_t
_chain
_IO_wide_data
_IO_backup_base
stdin
strcmp
_flags2
_mode
_IO_read_base
_vtable_offset
check_license
_IO_save_base
_fileno
_IO_lock_t
license-key.c
/home/dilldylanpickle/Linux-x64-Reversing
/usr/lib/gcc/x86_64-linux-gnu/11/include
/usr/include/x86_64-linux-gnu/bits
/usr/include/x86_64-linux-gnu/bits/types
/usr/include
stddef.h
types.h
struct_FILE.h
string.h
stdio.h
Scrt1.o
__abi_tag
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.0
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
license-key.c
__FRAME_END__
_DYNAMIC
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_start_main@GLIBC_2.34
_ITM_deregisterTMCloneTable
puts@GLIBC_2.2.5
stdin@GLIBC_2.2.5
_edata
_fini
strlen@GLIBC_2.2.5
__stack_chk_fail@GLIBC_2.4
printf@GLIBC_2.2.5
fgets@GLIBC_2.2.5
__data_start
strcmp@GLIBC_2.2.5
__gmon_start__
__dso_handle
_IO_stdin_used
_end
__bss_start
main
check_license
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@GLIBC_2.2.5
_init
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
.debug_aranges
.debug_info
.debug_abbrev
.debug_line
.debug_str
.debug_line_str
```

Let's break down the typical kinds of strings we've extracted from the license-key binary:

> 1. Library and System References:

* `/lib64/ld-linux-x86-64.so.2`, `libc.so.6`, and other `.so` files: These are references to dynamic libraries. The binary might be using functions from these libraries.

> 2. Function Names:

* `fgets`, `strcmp`, `puts`, `strlen`, `printf`, `__libc_start_main`, etc.: These are standard library functions.
* Their presence suggests the operations the binary might perform, e.g., `fgets` for reading input, `strcmp` for comparing strings.

> 3. Custom or Non-Standard Strings:

* `uT54_cOn5O1E_COw60ys`: Looks like some kind of encoded or obfuscated string.
* `Enter your license key: `, `License key is valid!`, `License key is invalid.`: These strings are directly related to the program's functionality.

> 4. Compiler and Debug Info:

* GCC: (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0: Indicates the compiler version used.
* `/home/dilldylanpickle/Linux-x64-Reversing`, `license-key.c`: Gives insights into the development environment and source file names.

> 5. Standard Files and References:

* Various headers like `string.h`, `stdio.h`, and system paths: Indicates usage of standard libraries or functions.

> 6. Symbols and Relocations:

* `.symtab`, `.strtab`, `.shstrtab`, `.dynsym`, `.dynstr`, `.rela.dyn`, `.rela.plt`, etc.: These are sections in the ELF format that pertain to symbols, their string representations, and relocation entries.


> 7. Miscellaneous Information:

* Various references like `__gmon_start__`, `_ITM_registerTMCloneTable`, and `__cxa_finalize` indicate specific functionalities or attributes of the binary.
* Memory section names like `.text`, `.data`, `.bss` provide insight into the layout of the binary in memory.

#### c. Display information from any ELF file

THe `readelf` command will provides essential details about the format and structure of the binary.

***Terminal:***
```bash
dilldylanpickle@archlinux:~/Linux-x64-Reversing$ readelf -a license-key
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1100
  Start of program headers:          64 (bytes into file)
  Start of section headers:          16696 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         37
  Section header string table index: 36

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000000338  00000338
       0000000000000030  0000000000000000   A       0     0     8
  [ 3] .note.gnu.bu[...] NOTE             0000000000000368  00000368
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000000038c  0000038c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000000003b0  000003b0
       0000000000000028  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           00000000000003d8  000003d8
       0000000000000138  0000000000000018   A       7     1     8
  [ 7] .dynstr           STRTAB           0000000000000510  00000510
       00000000000000c9  0000000000000000   A       0     0     1
  [ 8] .gnu.version      VERSYM           00000000000005da  000005da
       000000000000001a  0000000000000002   A       6     0     2
  [ 9] .gnu.version_r    VERNEED          00000000000005f8  000005f8
       0000000000000040  0000000000000000   A       7     1     8
  [10] .rela.dyn         RELA             0000000000000638  00000638
       00000000000000d8  0000000000000018   A       6     0     8
  [11] .rela.plt         RELA             0000000000000710  00000710
       0000000000000090  0000000000000018  AI       6    24     8
  [12] .init             PROGBITS         0000000000001000  00001000
       000000000000001b  0000000000000000  AX       0     0     4
  [13] .plt              PROGBITS         0000000000001020  00001020
       0000000000000070  0000000000000010  AX       0     0     16
  [14] .plt.got          PROGBITS         0000000000001090  00001090
       0000000000000010  0000000000000010  AX       0     0     16
  [15] .plt.sec          PROGBITS         00000000000010a0  000010a0
       0000000000000060  0000000000000010  AX       0     0     16
  [16] .text             PROGBITS         0000000000001100  00001100
       00000000000001e0  0000000000000000  AX       0     0     16
  [17] .fini             PROGBITS         00000000000012e0  000012e0
       000000000000000d  0000000000000000  AX       0     0     4
  [18] .rodata           PROGBITS         0000000000002000  00002000
       0000000000000060  0000000000000000   A       0     0     4
  [19] .eh_frame_hdr     PROGBITS         0000000000002060  00002060
       000000000000003c  0000000000000000   A       0     0     4
  [20] .eh_frame         PROGBITS         00000000000020a0  000020a0
       00000000000000cc  0000000000000000   A       0     0     8
  [21] .init_array       INIT_ARRAY       0000000000003d90  00002d90
       0000000000000008  0000000000000008  WA       0     0     8
  [22] .fini_array       FINI_ARRAY       0000000000003d98  00002d98
       0000000000000008  0000000000000008  WA       0     0     8
  [23] .dynamic          DYNAMIC          0000000000003da0  00002da0
       00000000000001f0  0000000000000010  WA       7     0     8
  [24] .got              PROGBITS         0000000000003f90  00002f90
       0000000000000070  0000000000000008  WA       0     0     8
  [25] .data             PROGBITS         0000000000004000  00003000
       0000000000000010  0000000000000000  WA       0     0     8
  [26] .bss              NOBITS           0000000000004010  00003010
       0000000000000010  0000000000000000  WA       0     0     16
  [27] .comment          PROGBITS         0000000000000000  00003010
       000000000000002b  0000000000000001  MS       0     0     1
  [28] .debug_aranges    PROGBITS         0000000000000000  0000303b
       0000000000000030  0000000000000000           0     0     1
  [29] .debug_info       PROGBITS         0000000000000000  0000306b
       0000000000000370  0000000000000000           0     0     1
  [30] .debug_abbrev     PROGBITS         0000000000000000  000033db
       0000000000000159  0000000000000000           0     0     1
  [31] .debug_line       PROGBITS         0000000000000000  00003534
       00000000000000bd  0000000000000000           0     0     1
  [32] .debug_str        PROGBITS         0000000000000000  000035f1
       0000000000000290  0000000000000001  MS       0     0     1
  [33] .debug_line_str   PROGBITS         0000000000000000  00003881
       00000000000000d1  0000000000000001  MS       0     0     1
  [34] .symtab           SYMTAB           0000000000000000  00003958
       0000000000000408  0000000000000018          35    18     8
  [35] .strtab           STRTAB           0000000000000000  00003d60
       0000000000000267  0000000000000000           0     0     1
  [36] .shstrtab         STRTAB           0000000000000000  00003fc7
       000000000000016a  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x00000000000007a0 0x00000000000007a0  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x00000000000002ed 0x00000000000002ed  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x000000000000016c 0x000000000000016c  R      0x1000
  LOAD           0x0000000000002d90 0x0000000000003d90 0x0000000000003d90
                 0x0000000000000280 0x0000000000000290  RW     0x1000
  DYNAMIC        0x0000000000002da0 0x0000000000003da0 0x0000000000003da0
                 0x00000000000001f0 0x00000000000001f0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  NOTE           0x0000000000000368 0x0000000000000368 0x0000000000000368
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  GNU_EH_FRAME   0x0000000000002060 0x0000000000002060 0x0000000000002060
                 0x000000000000003c 0x000000000000003c  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000002d90 0x0000000000003d90 0x0000000000003d90
                 0x0000000000000270 0x0000000000000270  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt 
   03     .init .plt .plt.got .plt.sec .text .fini 
   04     .rodata .eh_frame_hdr .eh_frame 
   05     .init_array .fini_array .dynamic .got .data .bss 
   06     .dynamic 
   07     .note.gnu.property 
   08     .note.gnu.build-id .note.ABI-tag 
   09     .note.gnu.property 
   10     .eh_frame_hdr 
   11     
   12     .init_array .fini_array .dynamic .got 

Dynamic section at offset 0x2da0 contains 27 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0x1000
 0x000000000000000d (FINI)               0x12e0
 0x0000000000000019 (INIT_ARRAY)         0x3d90
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x3d98
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x3b0
 0x0000000000000005 (STRTAB)             0x510
 0x0000000000000006 (SYMTAB)             0x3d8
 0x000000000000000a (STRSZ)              201 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x3f90
 0x0000000000000002 (PLTRELSZ)           144 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x710
 0x0000000000000007 (RELA)               0x638
 0x0000000000000008 (RELASZ)             216 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000000000001e (FLAGS)              BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW PIE
 0x000000006ffffffe (VERNEED)            0x5f8
 0x000000006fffffff (VERNEEDNUM)         1
 0x000000006ffffff0 (VERSYM)             0x5da
 0x000000006ffffff9 (RELACOUNT)          3
 0x0000000000000000 (NULL)               0x0

Relocation section '.rela.dyn' at offset 0x638 contains 9 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000003d90  000000000008 R_X86_64_RELATIVE                    11e0
000000003d98  000000000008 R_X86_64_RELATIVE                    11a0
000000004008  000000000008 R_X86_64_RELATIVE                    4008
000000003fd8  000100000006 R_X86_64_GLOB_DAT 0000000000000000 __libc_start_main@GLIBC_2.34 + 0
000000003fe0  000200000006 R_X86_64_GLOB_DAT 0000000000000000 _ITM_deregisterTM[...] + 0
000000003fe8  000900000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0
000000003ff0  000a00000006 R_X86_64_GLOB_DAT 0000000000000000 _ITM_registerTMCl[...] + 0
000000003ff8  000b00000006 R_X86_64_GLOB_DAT 0000000000000000 __cxa_finalize@GLIBC_2.2.5 + 0
000000004010  000c00000005 R_X86_64_COPY     0000000000004010 stdin@GLIBC_2.2.5 + 0

Relocation section '.rela.plt' at offset 0x710 contains 6 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000003fa8  000300000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
000000003fb0  000400000007 R_X86_64_JUMP_SLO 0000000000000000 strlen@GLIBC_2.2.5 + 0
000000003fb8  000500000007 R_X86_64_JUMP_SLO 0000000000000000 __stack_chk_fail@GLIBC_2.4 + 0
000000003fc0  000600000007 R_X86_64_JUMP_SLO 0000000000000000 printf@GLIBC_2.2.5 + 0
000000003fc8  000700000007 R_X86_64_JUMP_SLO 0000000000000000 fgets@GLIBC_2.2.5 + 0
000000003fd0  000800000007 R_X86_64_JUMP_SLO 0000000000000000 strcmp@GLIBC_2.2.5 + 0
No processor specific unwind information to decode

Symbol table '.dynsym' contains 13 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _[...]@GLIBC_2.34 (2)
     2: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5 (3)
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
     5: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __[...]@GLIBC_2.4 (4)
     6: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
     7: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fgets@GLIBC_2.2.5 (3)
     8: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
     9: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    10: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
    11: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    12: 0000000000004010     8 OBJECT  GLOBAL DEFAULT   26 stdin@GLIBC_2.2.5 (3)

Symbol table '.symtab' contains 43 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS Scrt1.o
     2: 000000000000038c    32 OBJECT  LOCAL  DEFAULT    4 __abi_tag
     3: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
     4: 0000000000001130     0 FUNC    LOCAL  DEFAULT   16 deregister_tm_clones
     5: 0000000000001160     0 FUNC    LOCAL  DEFAULT   16 register_tm_clones
     6: 00000000000011a0     0 FUNC    LOCAL  DEFAULT   16 __do_global_dtors_aux
     7: 0000000000004018     1 OBJECT  LOCAL  DEFAULT   26 completed.0
     8: 0000000000003d98     0 OBJECT  LOCAL  DEFAULT   22 __do_global_dtor[...]
     9: 00000000000011e0     0 FUNC    LOCAL  DEFAULT   16 frame_dummy
    10: 0000000000003d90     0 OBJECT  LOCAL  DEFAULT   21 __frame_dummy_in[...]
    11: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS license-key.c
    12: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    13: 0000000000002168     0 OBJECT  LOCAL  DEFAULT   20 __FRAME_END__
    14: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS 
    15: 0000000000003da0     0 OBJECT  LOCAL  DEFAULT   23 _DYNAMIC
    16: 0000000000002060     0 NOTYPE  LOCAL  DEFAULT   19 __GNU_EH_FRAME_HDR
    17: 0000000000003f90     0 OBJECT  LOCAL  DEFAULT   24 _GLOBAL_OFFSET_TABLE_
    18: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_mai[...]
    19: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
    20: 0000000000004000     0 NOTYPE  WEAK   DEFAULT   25 data_start
    21: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5
    22: 0000000000004010     8 OBJECT  GLOBAL DEFAULT   26 stdin@GLIBC_2.2.5
    23: 0000000000004010     0 NOTYPE  GLOBAL DEFAULT   25 _edata
    24: 00000000000012e0     0 FUNC    GLOBAL HIDDEN    17 _fini
    25: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strlen@GLIBC_2.2.5
    26: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail[...]
    27: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND printf@GLIBC_2.2.5
    28: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fgets@GLIBC_2.2.5
    29: 0000000000004000     0 NOTYPE  GLOBAL DEFAULT   25 __data_start
    30: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strcmp@GLIBC_2.2.5
    31: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    32: 0000000000004008     0 OBJECT  GLOBAL HIDDEN    25 __dso_handle
    33: 0000000000002000     4 OBJECT  GLOBAL DEFAULT   18 _IO_stdin_used
    34: 0000000000004020     0 NOTYPE  GLOBAL DEFAULT   26 _end
    35: 0000000000001100    38 FUNC    GLOBAL DEFAULT   16 _start
    36: 0000000000004010     0 NOTYPE  GLOBAL DEFAULT   26 __bss_start
    37: 0000000000001219   199 FUNC    GLOBAL DEFAULT   16 main
    38: 00000000000011e9    48 FUNC    GLOBAL DEFAULT   16 check_license
    39: 0000000000004010     0 OBJECT  GLOBAL HIDDEN    25 __TMC_END__
    40: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
    41: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize@G[...]
    42: 0000000000001000     0 FUNC    GLOBAL HIDDEN    12 _init

Histogram for `.gnu.hash' bucket list length (total of 2 buckets):
 Length  Number     % of total  Coverage
      0  0          (  0.0%)
      1  2          (100.0%)    100.0%

Version symbols section '.gnu.version' contains 13 entries:
 Addr: 0x00000000000005da  Offset: 0x0005da  Link: 6 (.dynsym)
  000:   0 (*local*)       2 (GLIBC_2.34)    1 (*global*)      3 (GLIBC_2.2.5)
  004:   3 (GLIBC_2.2.5)   4 (GLIBC_2.4)     3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  008:   3 (GLIBC_2.2.5)   1 (*global*)      1 (*global*)      3 (GLIBC_2.2.5)
  00c:   3 (GLIBC_2.2.5)

Version needs section '.gnu.version_r' contains 1 entry:
 Addr: 0x00000000000005f8  Offset: 0x0005f8  Link: 7 (.dynstr)
  000000: Version: 1  File: libc.so.6  Cnt: 3
  0x0010:   Name: GLIBC_2.4  Flags: none  Version: 4
  0x0020:   Name: GLIBC_2.2.5  Flags: none  Version: 3
  0x0030:   Name: GLIBC_2.34  Flags: none  Version: 2

Displaying notes found in: .note.gnu.property
  Owner                Data size 	Description
  GNU                  0x00000020	NT_GNU_PROPERTY_TYPE_0
      Properties: x86 feature: IBT, SHSTK
	x86 ISA needed: x86-64-baseline

Displaying notes found in: .note.gnu.build-id
  Owner                Data size 	Description
  GNU                  0x00000014	NT_GNU_BUILD_ID (unique build ID bitstring)
    Build ID: ded5e8f145e4e45c471b080d253dd02bfe98d056

Displaying notes found in: .note.ABI-tag
  Owner                Data size 	Description
  GNU                  0x00000010	NT_GNU_ABI_TAG (ABI version tag)
    OS: Linux, ABI: 3.2.0
```