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
> 1. ELF

This stands for "Executable and Linkable Format" and it's the standard binary format for Unix systems.