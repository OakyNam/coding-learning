# 08 - Structs and Objects

C does not have classes, but `struct` groups related data.

```c
struct Student {
    char name[50];
    int age;
};

struct Student student = {"Mona", 25};
printf("%s\n", student.name);
```

Structs are useful for modeling records.

Practice: create a `Book` struct with title, author, and page count.
