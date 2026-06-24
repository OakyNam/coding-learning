# 02 - JSON Structure

## Learning Goal

Recognize and write valid JSON objects, arrays, and primitive values.

By the end of this lesson, you should be able to:

- Tell whether a JSON value is an object, array, string, number, boolean, or `null`.
- Read nested JSON by following braces, brackets, keys, and values.
- Avoid common syntax mistakes such as single quotes, trailing commas, and Python-style `True`, `False`, or `None`.
- Explain how JSON values map to Python values after parsing.

## Why Structure Comes First

Before you can query JSON, parse it in Python, or use it from an API response, you need to see its shape.

JSON structure tells you whether data is stored as:

- a named field in an object,
- an item in an ordered array,
- another nested object or array,
- or a primitive value such as a string, number, boolean, or `null`.

That shape determines how you access the data later.

## JSON Values

JSON is built from values. A JSON value can be:

- an object
- an array
- a string
- a number
- `true`
- `false`
- `null`

Objects and arrays are container values. They hold other values.

Primitive values do not contain child values:

```json
"JSON"
42
3.14
true
false
null
```

## Objects

A JSON object uses curly braces and stores name/value pairs:

```json
{
  "name": "Mona",
  "active": true
}
```

In this object:

- `"name"` is a member name, often called a key.
- `"Mona"` is the value for `"name"`.
- `"active"` is another key.
- `true` is the value for `"active"`.
- A colon separates each key from its value.
- A comma separates object members.

Object member names are strings, so they must use double quotes.

## Arrays

A JSON array uses square brackets and stores ordered values:

```json
["intro", "json", "practice"]
```

Arrays keep their order. The first item is `"intro"`, the second is `"json"`, and the third is `"practice"`.

Arrays can contain objects too:

```json
[
  {
    "title": "Variables",
    "minutes": 20
  },
  {
    "title": "Lists",
    "minutes": 35
  }
]
```

This shape is common in API responses: a list of records, where each record is an object.

## Reading a Nested Document

Here is a small JSON document:

```json
{
  "course": {
    "title": "Python Basics",
    "published": true,
    "lesson_count": 10,
    "tags": ["python", "json", "data"],
    "instructor": null
  }
}
```

Walk through it from the outside in:

- The whole document is an object because it starts with `{` and ends with `}`.
- The object has one top-level key: `"course"`.
- The value of `"course"` is another object.
- `"title"` has a string value.
- `"published"` has a boolean value.
- `"lesson_count"` has a number value.
- `"tags"` has an array value.
- `"instructor"` has the value `null`, meaning there is intentionally no value there.

Formatted JSON uses indentation to make the structure visible. The spaces and newlines help humans read it, but the braces, brackets, colons, commas, strings, and values are what define the data.

## JSON Syntax Rules

- Object keys must be strings in double quotes.
- Strings must use double quotes, not single quotes.
- A colon separates a key from its value.
- Commas separate object members and array elements.
- Arrays keep their order.
- Objects are name/value collections; do not use key order to carry meaning.
- Use lowercase `true`, `false`, and `null`.
- Do not add a trailing comma after the last object member or array item.
- JSON does not support comments.

Valid JSON:

```json
{
  "name": "Mona",
  "active": true,
  "scores": [10, 12, 15]
}
```

Invalid JSON:

```json
{
  name: "Mona"
}
```

The key is not in double quotes.

Invalid JSON:

```json
{
  "active": True
}
```

JSON uses `true`, not Python's `True`.

Invalid JSON:

```json
{
  "scores": [10, 12, 15,]
}
```

The trailing comma after `15` is not valid JSON.

## JSON and Python Values

When Python reads JSON with `json.loads()` or `json.load()`, JSON values become Python values:

| JSON value | Python value |
| --- | --- |
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` or `float` |
| `true` | `True` |
| `false` | `False` |
| `null` | `None` |

Example:

```python
import json

text = '''
{
  "name": "Mona",
  "skills": ["Python", "JSON"],
  "active": true,
  "mentor": null
}
'''

data = json.loads(text)

print(data["skills"][0])
print(data["mentor"])
```

Expected output:

```text
Python
None
```

The JSON text contains `true` and `null`. After parsing, Python sees `True` and `None`.

## Common Mistakes

- Writing Python syntax instead of JSON syntax.
- Forgetting double quotes around object keys.
- Using single quotes around JSON strings.
- Using `True`, `False`, or `None` instead of `true`, `false`, or `null`.
- Adding a trailing comma.
- Confusing an object `{}` with an array `[]`.
- Assuming every array item must have the same type. JSON allows mixed arrays, though consistent arrays are usually easier to work with.
- Reusing the same object key more than once. JSON syntax allows this in some parsers, but duplicate names cause interoperability problems.

## Exercise

Create a JSON document that describes a small learning playlist.

Requirements:

1. The top-level value must be an object.
2. Include a string field named `"title"`.
3. Include a boolean field named `"public"`.
4. Include a number field named `"lesson_count"`.
5. Include a null field named `"reviewed_by"`.
6. Include an array named `"lessons"`.
7. Each lesson in `"lessons"` should be an object with:
   - `"title"` as a string
   - `"minutes"` as a number
   - `"completed"` as a boolean
8. After writing the JSON, label each value type used in the document.

## Worked Answer

```json
{
  "title": "JSON Basics",
  "public": true,
  "lesson_count": 3,
  "reviewed_by": null,
  "lessons": [
    {
      "title": "What JSON Is",
      "minutes": 8,
      "completed": true
    },
    {
      "title": "Objects and Arrays",
      "minutes": 12,
      "completed": false
    },
    {
      "title": "Common Syntax Mistakes",
      "minutes": 10,
      "completed": false
    }
  ]
}
```

Type labels:

- The top-level value is an object.
- `"title"` is a string.
- `"public"` is a boolean.
- `"lesson_count"` is a number.
- `"reviewed_by"` is `null`.
- `"lessons"` is an array.
- Each item inside `"lessons"` is an object.
- Inside each lesson object, `"title"` is a string, `"minutes"` is a number, and `"completed"` is a boolean.

## Sources

- RFC 8259, The JavaScript Object Notation (JSON) Data Interchange Format: https://datatracker.ietf.org/doc/html/rfc8259
- ECMA-404, The JSON Data Interchange Syntax: https://tc39.es/ecma404/
- Python `json` module documentation: https://docs.python.org/3/library/json.html
- JSON.org syntax overview: https://www.json.org/json-en.html

## Next Step

Next, learn how JSONPath selects values from JSON documents in [JSONPath Basics](03_jsonpath_basics.md).
