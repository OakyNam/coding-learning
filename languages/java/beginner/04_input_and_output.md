# 04 - Input and Output

Use `System.out.println` for output.

```java
System.out.println("Hello");
```

Use `Scanner` for beginner console input.

```java
import java.util.Scanner;

Scanner scanner = new Scanner(System.in);
System.out.print("Name: ");
String name = scanner.nextLine();
System.out.println("Hello, " + name);
```

Practice: ask for an age and print the age next year.
