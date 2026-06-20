# 07 - Arrays, Lists, and Collections

Arrays have fixed size.

```java
int[] scores = {90, 85, 100};
System.out.println(scores[0]);
```

Lists can grow.

```java
import java.util.ArrayList;

ArrayList<String> names = new ArrayList<>();
names.add("Ali");
names.add("Mona");
```

Loop through a list:

```java
for (String name : names) {
    System.out.println(name);
}
```

Practice: create a list of cities and print each one.
