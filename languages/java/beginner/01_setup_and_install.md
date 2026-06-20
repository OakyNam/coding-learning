# 01 - Setup and Install

Install a JDK, such as the latest long-term support version from Eclipse Temurin, Microsoft Build of OpenJDK, or Oracle.

Check Java:

```bash
java --version
javac --version
```

Create `Hello.java`:

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

Compile and run:

```bash
javac Hello.java
java Hello
```

Practice: change the message and run it again.
