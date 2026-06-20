# 10 - Headers and Libraries

Headers declare functions, types, and constants.

```c
#include <stdio.h>
#include <math.h>
```

Your own header:

```c
// calculator.h
int add(int a, int b);
```

Implementation:

```c
// calculator.c
int add(int a, int b) {
    return a + b;
}
```

Practice: split an `add` function into a header and source file.
