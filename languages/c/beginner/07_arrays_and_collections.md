# 07 - Arrays and Collections

C arrays store multiple values of the same type.

```c
int scores[] = {90, 85, 100};
printf("%d\n", scores[0]);
```

Track the length yourself or calculate it:

```c
int length = sizeof(scores) / sizeof(scores[0]);
```

Loop through an array:

```c
for (int i = 0; i < length; i++) {
    printf("%d\n", scores[i]);
}
```

Practice: create an array of five numbers and print their total.
