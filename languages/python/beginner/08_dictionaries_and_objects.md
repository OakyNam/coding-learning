# 08 - Dictionaries and Objects

Dictionaries store key-value pairs.

```python
student = {
    "name": "Mona",
    "age": 25,
    "active": True
}

print(student["name"])
```

Add or update values:

```python
student["city"] = "Chicago"
student["age"] = 26
```

Loop through keys and values:

```python
for key, value in student.items():
    print(key, value)
```

Practice: create a dictionary for a movie with title, year, and rating.
