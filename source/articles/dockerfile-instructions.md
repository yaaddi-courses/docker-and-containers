# Writing a Dockerfile: FROM, COPY, RUN, CMD and WORKDIR

**Covers:** Dockerfile, FROM, COPY, COPYing, RUN, CMD, WORKDIR

## A recipe for an image {#overview}
A Dockerfile is the recipe that Docker follows to build an image. It is a plain text file with one instruction per line, and Docker runs the instructions from top to bottom. Only a handful of instructions cover most real Dockerfiles. Learn what each one does, and when it does it, and you can read almost any of them.

## Dockerfile {#dockerfile}
A Dockerfile is a text file, named `Dockerfile` with no extension, that lists the exact steps Docker follows to build an image: what to start from, what files to add and what to run. Keeping the recipe in a file, in the project's repository, makes the build repeatable. Anyone, or any build server, can produce the same image.

Lines starting with `#` are comments. Instruction names such as FROM are written in capitals by convention, though Docker does not require it.

## FROM {#from}
`FROM` sets the base image that everything else builds on. Every Dockerfile needs one, and it is usually the first line, for example `FROM python:3.12-slim`. Everything the base provides, such as the operating system files and the language runtime, comes with it. Choose a small, official base, and a specific version label, not `latest`.

## WORKDIR {#workdir}
`WORKDIR` sets the working directory for the instructions that follow, and for the container when it starts. `WORKDIR /app` means later COPY, RUN and CMD instructions treat `/app` as the current folder. If the folder does not exist, Docker creates it. Prefer it to repeated `cd` commands, since it makes paths simple and clear.

## COPY {#copy}
`COPY` copies files from your project folder into the image being built, for example `COPY . /app`. The application's source code gets baked into the image, so it is available when a container starts. The source is relative to the folder you pass to `docker build`, and it cannot reach files outside it.

## COPYing {#copying}
People use "COPYing" as a verb: "COPYing the whole project too early breaks the cache". It refers to using the COPY instruction. A tip: copy the dependency list first, install, and then copy the rest of the code, so that a change in your code does not repeat the slow install.

## RUN {#run}
`RUN` executes a command while the image is being built, such as installing a package, and bakes the result permanently into the image as a new layer. Example: `RUN pip install -r requirements.txt`. It runs once, at build time. It does not run when a container starts.

## RUNs {#runs}
Several RUN instructions create several layers, one each. Combine related commands with `&&` in a single RUN to reduce layers, and clean up in the same step, for example `RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*`. Cleaning in a later RUN would not shrink the image, because the earlier layer already holds the files.

## CMD {#cmd}
`CMD` sets the command that a container runs automatically when it starts. Unlike RUN, which acts while the image is built, CMD acts when a container is launched from the finished image. Example: `CMD ["python", "app.py"]`. A Dockerfile has only one effective CMD, and anything you type after the image name in `docker run` replaces it.

Prefer the list form, called exec form, as shown. It runs your program directly, so signals from `docker stop` reach it and it can shut down cleanly.

## A worked example {#example}
A complete Dockerfile for a small Python application:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

Read it line by line:

1. `FROM` starts from a slim Python image.
2. `WORKDIR /app` makes `/app` the current folder.
3. The first `COPY` brings in only the dependency list.
4. `RUN` installs the dependencies at build time. This layer is cached until the list changes.
5. The second `COPY` brings in the source code, which changes often.
6. `CMD` says what to run when a container starts.

Count the effect on a rebuild after a one-line code change: steps 1 to 4 come from the cache, and only step 5 and the final metadata are redone, so it takes about a second. If steps 3 and 5 were one `COPY . .` before the install, the install would run again every time, perhaps for 40 seconds.

Build and run it with `docker build -t myapp .` and `docker run myapp`. The first command executes FROM to RUN. The second executes CMD.

## A common mistake {#mistake}
Confusing RUN and CMD. A command that should happen once, at build time, belongs in RUN. The command that starts your application belongs in CMD. Putting `RUN python app.py` makes the build run the server forever. Another mistake is copying everything before installing dependencies, which spoils the cache.

## Key points {#key-points}
- A Dockerfile lists the build steps, one instruction per line.
- FROM sets the base, WORKDIR the folder, COPY adds files, RUN executes at build time and CMD runs at start.
- Copy the dependency list first, install, then copy the code.
- Combine related RUN commands and clean up in the same step.
