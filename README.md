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

### a. Determine the File Type

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