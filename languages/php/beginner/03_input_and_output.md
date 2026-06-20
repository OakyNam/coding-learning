# 03 - Input and Output

## Learning Goal

Learn input and output in PHP well enough to read examples, edit them, and use the idea in a small program.

## Why It Matters

This lesson helps you move from recognizing the idea to using it in real programs. Read the example, trace what each line does, and then change the code so the idea becomes yours.

## Core Idea

In PHP, this topic shows up often at the beginner level. Focus on the shape of the problem first: what data enters, what work happens, and what result should come out.

## Example

```
<?php
$items = ["api", "json", "test"];
foreach ($items as $item) {
    echo strtoupper($item) . PHP_EOL;
}
```

## How To Think About It

- Name the input before writing the solution.
- Keep each step small enough to explain out loud.
- Check the result with simple values before trying harder cases.
- Prefer clear code while learning; clever code can wait.

## Common Mistakes

- Copying the example without changing it.
- Ignoring error messages instead of reading the first useful line.
- Mixing several new ideas in one experiment.
- Forgetting to run the program after each small change.

## Practice

1. Recreate the example from memory.
2. Change the names, values, or inputs and run it again.
3. Write a short note explaining what changed.
4. Connect the idea to one shared topic from the root README.

## Next Step

Return to this level's README and continue with the next numbered lesson.


