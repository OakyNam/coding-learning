# Environment Variables

Environment variables store configuration outside source code.

## Core Ideas

- They are useful for secrets, URLs, ports, and feature flags.
- Different environments can use different values.
- Public repos should include examples, not real secrets.

## Example

```bash
API_URL=https://example.com
PORT=3000
```

## Practice

Create a sample `.env.example` with placeholder values for a local app.
