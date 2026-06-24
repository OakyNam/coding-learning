# JSON

JSON is a lightweight, text-based, language-independent data interchange format. In Python work, you will see JSON in configuration files, saved data, web API requests and responses, logs, test fixtures, and service-to-service messages.

This topic teaches two connected skills:

- reading and writing valid JSON data
- locating values inside nested JSON documents, including JSONPath-style queries

JSON is the data format. JSONPath is a separate query language for selecting values from JSON.

## Prerequisites

You should already be comfortable with basic Python dictionaries, lists, strings, numbers, booleans, loops, and indexing.

Useful vocabulary:

- JSON text or document: serialized data written in JSON syntax.
- Python object: the in-memory value after Python parses JSON.
- Object member: a JSON object name/value pair.
- Array element: one value in an ordered JSON array.
- Query or path: a way to point at or select values inside a JSON document.

## What You Will Learn

- How JSON objects, arrays, strings, numbers, booleans, and `null` work.
- How JSON differs from Python literal syntax.
- How JSON maps to Python `dict`, `list`, `str`, `int`, `float`, `True`, `False`, and `None`.
- How to load and save JSON with Python's standard `json` module.
- How to navigate nested objects and arrays.
- How JSONPath can describe precise selections from JSON data.
- How to handle missing fields, `null`, duplicate-name risks, encoding issues, and common syntax mistakes.

## JSON Essentials

JSON values can be objects, arrays, strings, numbers, `true`, `false`, or `null`.

Objects use `{}` and contain string member names:

```json
{
  "name": "Mona",
  "active": true
}
```

Arrays use `[]` and preserve order:

```json
["setup", "structure", "queries"]
```

JSON is not exactly Python syntax. JSON strings and object names use double quotes. JSON uses lowercase `true`, `false`, and `null`. JSON does not allow comments or trailing commas.

For interoperability, avoid duplicate object member names and use UTF-8 when exchanging JSON between systems.

## Python and JSON

Python's standard `json` module parses JSON text into Python values and serializes Python values back to JSON text.

| JSON value | Python value |
| --- | --- |
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` or `float` |
| `true` | `True` |
| `false` | `False` |
| `null` | `None` |

Common standard-library operations:

- `json.loads()` parses a JSON string.
- `json.load()` parses JSON from a readable file-like object.
- `json.dumps()` serializes a Python object to a JSON string.
- `json.dump()` writes JSON to a writable file-like object.
- `python -m json` can validate and pretty-print JSON from the shell.

Be careful with edge cases. JSON object keys are strings, so non-string Python dictionary keys may not round-trip as expected. Python's `json` module can accept or emit `NaN` and infinities by default, but those values are not valid JSON in the standard format.

## Lessons

1. [Foundations](01_foundations.md)  
   Start here for what JSON is, where it appears, and how it differs from Python literals.

2. [JSON Structure](02_json_structure.md)  
   Learn objects, arrays, primitive values, nesting, valid syntax, and reading formatted JSON.

3. [JSONPath Basics](03_jsonpath_basics.md)  
   Introduce RFC 9535 JSONPath ideas: root `$`, child segments, bracket notation, dot shorthand, selectors, and query results as zero-or-more matches.

4. [Querying Nested Data](04_querying_nested_data.md)  
   Practice moving through nested objects and arrays, choosing dot or bracket notation, and understanding missing members as empty results.

5. [Querying Arrays](05_querying_arrays.md)  
   Cover zero-based indexes, slices, wildcards, and selecting many array elements.

6. [Filters and Conditions](06_filters_and_conditions.md)  
   Learn filter selectors such as existence checks and comparisons, with care around missing values, `null`, strings, and numbers.

7. [Querying API Responses](07_querying_api_responses.md)  
   Apply parsing and selection to realistic API-shaped JSON: metadata, lists of records, pagination-like structures, errors, and optional fields.

8. [Querying JSON in Languages](08_querying_json_in_languages.md)  
   Learn that languages parse JSON into native structures. In Python, use `json` plus normal `dict` and `list` access, or a separate JSONPath library when needed.

9. [Common Query Mistakes](09_common_query_mistakes.md)  
   Review invalid JSON syntax, Python/JSON literal mixups, wrong array indexes, missing fields, confusing `null` with missing data, duplicate names, and JSONPath dialect differences.

10. [Practice Project](10_practice_project.md)  
    Read an unfamiliar JSON document, identify useful fields, parse it, and write precise selections.

## Suggested Study Order

Complete lessons 1 and 2 before doing JSONPath work. Those lessons teach the data format.

Complete lessons 3 through 6 in order because each one adds query syntax. Do lesson 7 after arrays and filters. Do lesson 8 once you can already read paths manually. Use lesson 9 as review before the practice project.

## JSONPath Note

This topic prefers RFC 9535 terminology where possible. In that standard, JSONPath selects zero or more nodes from a JSON value, and a valid query begins with `$`.

A missing member or out-of-range index should produce an empty result, not a crash, in JSONPath semantics. Older libraries and tool-specific JSONPath dialects may behave differently, especially around filters and slices.

## Sources

- RFC 8259, The JavaScript Object Notation (JSON) Data Interchange Format: https://datatracker.ietf.org/doc/html/rfc8259
- ECMA-404, The JSON Data Interchange Syntax: https://tc39.es/ecma404/
- Python `json` module documentation: https://docs.python.org/3/library/json.html
- RFC 9535, JSONPath: Query Expressions for JSON: https://datatracker.ietf.org/doc/html/rfc9535/

## Completion Goal

By the end, you should be able to read valid JSON, parse it in Python, understand its Python representation, and write or explain precise paths and queries for data inside nested objects and arrays.
