# 01 - Setup and Install

C programs must be compiled before they run. Install a compiler such as GCC, Clang, or MSVC.

Check GCC:

```bash
gcc --version
```

Create `hello.c`:

```c
#include <stdio.h>

int main(void) {
    printf("Hello, C!\n");
    return 0;
}
```

Compile and run:

```bash
gcc hello.c -o hello
./hello
```

Practice: change the message and compile again.
