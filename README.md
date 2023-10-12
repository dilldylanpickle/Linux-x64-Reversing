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
> 1. `ELF`:

This stands for "Executable and Linkable Format" and it's the standard binary format for Unix systems.

> 2. `64-bit`:

This indicates the architecture's bit-width. In this case, the binary is compiled for a 64-bit system.

> 3. `LSB`:

This stands for "Least Significant Byte" and it refers to the endianness of the data in the binary. LSB (or little-endian) means the least significant byte is stored first.

> 4. `pie`:

This stands for "Position Independent Executable" and this binary can be loaded into any memory address and still execute correctly. PIE is often used alongside ASLR (Address Space Layout Randomization) to increase security by randomizing the location of the process in memory.

> 5. `x86-64`:

This refers to the CPU architecture for which the binary was compiled. x86-64 (or AMD64) is the 64-bit version of the x86 instruction set.

> 6. `version 1 (SYSV)`:

This specifies the ELF version and ABI (Application Binary Interface) and "SYSV" refers to the System V ABI.

> 7. `dynamically linked`:

The binary requires external shared libraries at runtime. This is in contrast to a statically linked binary, which contains all required libraries.

> 8. `interpreter /lib64/ld-linux-x86-64.so.2`:

This is the dynamic linker/loader's path, responsible for loading the binary's dependencies when it starts.

> 9. `BuildID[sha1]=ded5e8f145e4e45c471b080d253dd02bfe98d056`:

A unique identifier for the binary, which can be useful for debugging or referring to specific versions of binaries.

> 10. `for GNU/Linux 3.2.0`:

Indicates the minimum kernel version required to execute the binary.

> 11. `with debug_info`:

The binary contains debugging information. This makes it easier to analyze, reverse engineer, or debug using tools like `gdb`.

> 12. `not stripped`:

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
/home/dylan/Test
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