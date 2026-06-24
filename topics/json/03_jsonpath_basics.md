# 03 - JSONPath Basics

## Learning Goal

Use JSONPath to select values from JSON objects and arrays using the root `$`, child properties, array indexes, wildcards, slices, descendant search, and simple filters.

By the end of this lesson, you should be able to:

- Explain what `$` means in a JSONPath query.
- Select object members with dot notation and bracket notation.
- Select array elements by index, wildcard, and slice.
- Use descendant search carefully.
- Read simple filters that use `@` for the current item.
- Expect a list of zero or more matches from a query.

## Why JSONPath Matters

JSON often contains nested objects and arrays. JSONPath gives you a compact way to describe the value or values you want without manually walking every level in prose.

You may see JSONPath-like expressions in API tools, test assertions, data pipelines, log tools, monitoring systems, and configuration tooling.

This lesson follows the vocabulary and behavior from RFC 9535 where possible. Older JSONPath libraries may differ, so always check the tool you are using.

## Starter JSON

Use this JSON document throughout the lesson:

```json
{
  "course": {
    "title": "Python Basics",
    "level": "beginner",
    "instructor": {
      "name": "Amina",
      "timezone": "UTC"
    },
    "lessons": [
      {
        "id": 1,
        "title": "Variables",
        "minutes": 25,
        "published": true
      },
      {
        "id": 2,
        "title": "Lists",
        "minutes": 40,
        "published": true
      },
      {
        "id": 3,
        "title": "Dictionaries",
        "minutes": 35,
        "published": false
      }
    ]
  }
}
```

## The Root `$`

A JSONPath query starts at the root value, written as `$`.

JSONPath treats the JSON document as a tree of nodes. Each segment narrows the current list of matching nodes. The final result is a list of zero or more matches.

Query:

```jsonpath
$.course.title
```

Result:

```json
["Python Basics"]
```

Step by step:

- `$` means the whole JSON document.
- `.course` selects the `course` member value.
- `.title` selects the `title` member value inside `course`.
- The result is a list containing the matching value.

## Dot Notation and Bracket Notation

Dot notation is short and readable for simple member names:

```jsonpath
$.course.instructor.name
```

Result:

```json
["Amina"]
```

Bracket notation is more general:

```jsonpath
$['course']['instructor']['name']
```

Result:

```json
["Amina"]
```

Use bracket notation when a member name has spaces, punctuation, or characters that would be awkward or invalid in dot notation.

## Arrays and Indexes

JSON arrays are ordered. JSONPath array indexes start at `0`.

Query:

```jsonpath
$.course.lessons[0].title
```

Result:

```json
["Variables"]
```

Query:

```jsonpath
$.course.lessons[1].minutes
```

Result:

```json
[40]
```

If an index is out of range, a valid JSONPath query returns no matches:

```jsonpath
$.course.lessons[99].title
```

Result:

```json
[]
```

## Wildcards

A wildcard `*` selects all children at that level.

Query:

```jsonpath
$.course.lessons[*].title
```

Result:

```json
["Variables", "Lists", "Dictionaries"]
```

Here, `lessons[*]` selects every lesson object in the array. Then `.title` selects the `title` value from each selected lesson object.

## Array Slices

A slice selects part of an array.

Query:

```jsonpath
$.course.lessons[:2].title
```

Result:

```json
["Variables", "Lists"]
```

The slice `:2` selects items from the start up to, but not including, index `2`. This is useful for "first few items" queries.

## Descendant Search

The descendant operator `..` searches below the current node, not just one level down.

Query:

```jsonpath
$..title
```

Result:

```json
["Python Basics", "Variables", "Lists", "Dictionaries"]
```

This finds every member named `title` anywhere in the document.

Descendant search is powerful, but less precise than a full path. Prefer a specific path when you know the structure.

## Simple Filters

A filter selects array or object children that pass a test. Inside a filter, `@` means the current item being tested.

Query:

```jsonpath
$.course.lessons[?@.published == true].title
```

Result:

```json
["Variables", "Lists"]
```

Query:

```jsonpath
$.course.lessons[?@.minutes >= 35].title
```

Result:

```json
["Lists", "Dictionaries"]
```

In these filters:

- `@` is the lesson currently being tested.
- `@.published` means the `published` value of that lesson.
- `@.minutes` means the `minutes` value of that lesson.
- `$` still means the whole input document.

## Common Mistakes

- Forgetting the root `$`: write `$.course.title`, not `course.title`.
- Expecting a single value every time. JSONPath returns a list of matches.
- Forgetting zero-based indexes: `[0]` is the first array item.
- Using `*` or `..` too early when a specific path would be clearer.
- Treating a missing field as a syntax error. A valid query with no matching data returns an empty result.
- Confusing `$` and `@`: `$` is the whole input document; `@` is the current item inside a filter.
- Assuming all JSONPath libraries behave identically. Prefer RFC 9535 behavior when writing portable examples.

## Exercise

Use the starter JSON. Write JSONPath queries for:

1. The course title.
2. The instructor name.
3. The title of the first lesson.
4. The titles of all lessons.
5. The titles of the first two lessons.
6. Every `title` value anywhere in the document.
7. The titles of published lessons.
8. The titles of lessons that take at least 35 minutes.
9. A path that returns an empty result because the field does not exist.

For each query, write the expected result list.

## Worked Answer

Course title:

```jsonpath
$.course.title
```

```json
["Python Basics"]
```

Instructor name:

```jsonpath
$.course.instructor.name
```

```json
["Amina"]
```

Title of the first lesson:

```jsonpath
$.course.lessons[0].title
```

```json
["Variables"]
```

Titles of all lessons:

```jsonpath
$.course.lessons[*].title
```

```json
["Variables", "Lists", "Dictionaries"]
```

Titles of the first two lessons:

```jsonpath
$.course.lessons[:2].title
```

```json
["Variables", "Lists"]
```

Every `title` value anywhere in the document:

```jsonpath
$..title
```

```json
["Python Basics", "Variables", "Lists", "Dictionaries"]
```

Titles of published lessons:

```jsonpath
$.course.lessons[?@.published == true].title
```

```json
["Variables", "Lists"]
```

Titles of lessons that take at least 35 minutes:

```jsonpath
$.course.lessons[?@.minutes >= 35].title
```

```json
["Lists", "Dictionaries"]
```

An empty result for a missing field:

```jsonpath
$.course.lessons[*].quiz
```

```json
[]
```

## Sources

- RFC 9535, JSONPath: Query Expressions for JSON: https://www.rfc-editor.org/rfc/rfc9535.html
- RFC 8259, The JavaScript Object Notation (JSON) Data Interchange Format: https://www.rfc-editor.org/rfc/rfc8259.html
- ECMA-404, The JSON Data Interchange Syntax: https://tc39.es/ecma404/
- JSON.org syntax overview: https://www.json.org/json-en.html

## Next Step

Next, practice paths through deeper objects and arrays in [Querying Nested Data](04_querying_nested_data.md).
