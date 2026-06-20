# Docker Basics

Docker packages an application with the environment it needs to run.

## Core Ideas

- An image is a packaged template.
- A container is a running instance of an image.
- A `Dockerfile` describes how to build an image.

## Example

```dockerfile
FROM alpine
CMD ["echo", "Hello from Docker"]
```

## Practice

Explain why a team might use Docker to run the same app on multiple machines.
