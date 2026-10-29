# Safer images: a small attack surface and a non-root user

**Covers:** attack surface, non-root user

## Assume that something will go wrong {#overview}
No application is free of bugs, and some bugs are security holes. Good security does not rely on never having a bug. It limits the damage when one is found. For containers, two practices do most of the work: ship less, so there is less to attack, and run with fewer rights, so that a break-in can do less.

## attack surface {#attack-surface}
An attack surface is the sum of all the places where a system could be broken into or misused. Every program, library and open port in an image is part of it. A full operating system image contains hundreds of tools, such as a shell, a package manager, compilers and network utilities, and each one may have vulnerabilities. Worse, those tools are also what an attacker uses once inside.

A smaller image has a smaller surface. Ways to reduce it:
- Start from a slim or minimal base image, such as a slim variant, Alpine or a distroless image that contains little more than the runtime.
- Use a multi-stage build so that compilers and test tools never reach the final image.
- Install only what the application needs, and remove package caches.
- Publish only the ports that are required.

Smaller images also download faster and start sooner, so security and efficiency point the same way.

## non-root user {#non-root-user}
A non-root user is an ordinary account with limited permissions, as opposed to the all-powerful root account. Containers run as root by default. That means that if an attacker takes over the application, they have full rights inside the container, can change any file in it and, if a weakness in the container system exists, may be a step closer to the host.

The fix is to create a limited user in the Dockerfile and switch to it with the `USER` instruction, before the final command. Then a compromised application can only do what that account may do. Also keep the application files owned by root and read-only to it, where possible, so that the application cannot rewrite its own code.

## non-root users {#non-root-users}
In the plural, these are accounts without administrator rights, used inside containers so that a compromise can do less harm. Some platforms enforce the rule and can refuse to start a container that wants to run as root. Preparing your images to run as a non-root user from the start avoids surprises later.

## A worked example {#example}
A Dockerfile that applies both ideas to a Python service:

```dockerfile
FROM python:3.12-slim
RUN useradd --create-home --uid 10001 appuser
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=appuser:appuser . .
USER appuser
CMD ["python", "app.py"]
```

What each choice does:

1. `python:3.12-slim` is a smaller base than the full image, which removes many tools that the service never uses.
2. `useradd` creates an account with no admin rights and a fixed id, 10001.
3. `--no-cache-dir` stops pip from leaving a download cache in the layer.
4. `USER appuser` makes everything after it, including the running service, use the limited account.

Check it from outside: `docker run --rm myapp:1.0 id` prints `uid=10001(appuser)`, not `uid=0(root)`. The id 0 would mean root.

Estimate the benefit: the full Python image is around 1,000 MB, and the slim image is around 130 MB. That is 87 percent smaller, and far fewer packages, so if a vulnerability scanner reports 150 problems in the full image, it may report 30 or fewer in the slim one. Fewer problems means less to fix and less for an attacker to use.

The cost is small. Some tools you are used to are missing in a slim image, so debugging may need a separate debug image, or a temporary container.

## A common mistake {#mistake}
Believing that a container is already a security boundary strong enough to run as root. A container shares the host's kernel, so root in a container is closer to root on the host than people expect. Another mistake is choosing a base image for convenience and never updating it. Rebuild regularly to pick up fixes.

## Key points {#key-points}
- The attack surface is everything in an image that could be abused. Make it small.
- Use slim bases, multi-stage builds and only the packages that you need.
- Containers run as root by default. Switch to a non-root user with `USER`.
- Check with `docker run --rm image id` that the process is not root.
