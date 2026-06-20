# 06 - Filters and Conditions

## Learning Goal

Use filters to select array items that match a condition.

## Why It Matters

JSON Querying is useful because it appears across many languages, tools, and real projects. Learning the concept once makes it easier to recognize everywhere else.

## Core Idea

Start with the smallest working example, then add one detail at a time. When the result changes, pause and explain why.

## Example

```
$.store.book[?(@.price < 10)].title
```

## Practical Pattern

- Identify the input.
- Choose the smallest tool or syntax that solves the problem.
- Check the output before building on it.
- Save a working example you can reuse later.

## Common Mistakes

- Memorizing syntax without understanding the input and output.
- Skipping small tests.
- Using a complex solution before the simple one is clear.

## Practice

1. Write a JSON document with at least one nested object and one array.
2. Query one top-level field with JSONPath.
3. Query one nested value.
4. Query multiple array items with a wildcard or filter.

## Next Step

Return to the topic README and continue with the next lesson.

