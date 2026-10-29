# Building images, build context and multi-stage builds

**Covers:** docker build, build context, multi-stage build

## From recipe to image {#overview}
Once you have a Dockerfile, a single command turns it into an image. What many people miss is what Docker sends to do it, and how to avoid shipping your build tools to production. This article covers the command, the folder that goes with it, and a technique that makes final images small.

## docker build {#docker-build}
`docker build` reads a Dockerfile and turns its instructions into an image. A typical call is `docker build -t myapp:1.0 .`. The `-t` flag names, or tags, the result. The final `.` is the location of the build context, which is here the current folder.

Docker prints each step as it runs, and marks steps taken from the cache. At the end, you have an image that you can run or push. If the Dockerfile has another name, point to it with `-f`, such as `docker build -f Dockerfile.dev .`.

## build context {#build-context}
The build context is the set of files in the folder you pass to `docker build`, which are sent to the Docker daemon before the build starts, so that instructions such as COPY have something to take from. Everything in that folder is sent, whether or not the Dockerfile uses it.

That makes a large folder a problem. If your project contains a 2 GB data folder or a `node_modules` directory, it is all uploaded for each build, which is slow. It can also leak files into the image by accident. A file named `.dockerignore` lists patterns to leave out, such as `.git`, `node_modules`, `*.log` and secrets. Use one in every project.

## multi-stage build {#multi-stage-build}
A multi-stage build is a Dockerfile technique that uses one stage to compile or build the application, with all its heavy build tools, and then copies only the finished result into a clean, small final stage. The first stage is thrown away. Only the last one becomes the image you ship.

This matters because the tools needed to build, such as compilers, package managers and test frameworks, are not needed to run. Leaving them in enlarges the image, slows downloads and gives attackers more to use. Each stage starts with its own FROM, and you copy between stages with `COPY --from=<stage>`.

## A worked example {#example}
A small Go program is built in one stage and run in another.

```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN go build -o /out/app .

FROM alpine:3.20
COPY --from=build /out/app /usr/local/bin/app
CMD ["app"]
```

How it works:

1. The first stage starts from an image with the full Go toolchain and builds the program into `/out/app`.
2. The second stage starts from a tiny Alpine image.
3. `COPY --from=build` takes only the compiled file from the first stage.
4. The final image does not contain the compiler or the source code.

Compare the sizes. The Go image is about 850 MB, and a single-stage build that ships it would be about 850 MB plus the program. The Alpine image is about 8 MB, and the compiled program is perhaps 10 MB. The multi-stage result is roughly 18 MB, which is about 2 percent of the other size, so it downloads in a moment and has far fewer files to attack.

Build it with `docker build -t shop:2.0 .`, run from the folder that holds the Dockerfile. If that folder also held a 1.5 GB `data` directory, the build context would send all of it, taking perhaps a minute before the first step even starts. A `.dockerignore` with the line `data` brings it back to a fraction of a second.

You can also stop at an earlier stage, for debugging, with `--target build`.

## A common mistake {#mistake}
Running `docker build` in a large folder with no `.dockerignore`. The build is slow and may copy secrets into the image. A second mistake is using a single stage, so the final image carries the compiler. Use multi-stage for any language with a separate build step.

## Key points {#key-points}
- `docker build -t name:tag .` builds an image from the Dockerfile in the build context.
- The build context is everything in the folder you pass, sent to the daemon, so keep it small with `.dockerignore`.
- A multi-stage build compiles in one stage and ships only the result from another.
- The result is a much smaller, safer final image.
