# 08 - Maps and Objects

Maps store key-value pairs.

```java
import java.util.HashMap;

HashMap<String, Integer> ages = new HashMap<>();
ages.put("Ali", 25);
ages.put("Mona", 30);

System.out.println(ages.get("Ali"));
```

Objects come from classes:

```java
class Book {
    String title;
    int pages;
}
```

Practice: create a map of product names to prices.
