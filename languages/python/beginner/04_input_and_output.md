# 04 - Input and Output

Use `print()` to show output and `input()` to read text.

```python
name = input("What is your name? ")
print(f"Hello, {name}!")
```

Input is always text at first. Convert it when you need a number.

```python
age_text = input("Age: ")
age = int(age_text)
print(f"Next year you will be {age + 1}")
```

Practice: ask for two numbers, convert them, and print their sum.
