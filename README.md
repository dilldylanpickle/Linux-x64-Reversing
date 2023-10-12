# Linux x64 Reversing

Here is a step-by-step process on how I would reverse an x86-64 Linux binary!

## Step 0. Example Binary

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