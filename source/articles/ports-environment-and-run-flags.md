# Run flags: ports, environment variables and background mode

**Covers:** -p, --publish, -d, -e, -t

## Changing how a container runs {#overview}
A plain `docker run` starts a container with default behavior, which is rarely what you want. It ties up your terminal, and nothing outside can reach the application. A few short flags change that. Each flag is a small setting you add after `docker run`, and a handful of them cover most daily use. Let's go through them.

## -p {#p}
`-p` publishes a container's port on the host machine, written as `host:container`, for example `-p 8080:80`. A container has its own private network, so a web server listening on port 80 inside it cannot be reached from outside until you publish the port. With `-p 8080:80`, requests to port 8080 on your machine are forwarded to port 80 in the container.

Nothing is published by default. You can choose any free host port, and the container port is whatever the application listens on. Only the first number can be changed freely. To restrict access to your own machine, bind to the loopback address with `-p 127.0.0.1:8080:80`.

## --publish {#publish}
`--publish` is the long form of `-p` and does exactly the same: `--publish 8080:80`. Long forms are easier to read in scripts and documentation. Many flags have both a short and a long form.

## -d {#d}
`-d` runs the container in the background, which is called detached mode. Docker starts it, prints its ID and gives your terminal back. Without it, the container runs in the foreground and shows its output until you press Ctrl+C, which also stops it. Use `-d` for servers, and then use `docker logs` to see the output.

## -e {#e}
`-e` sets an environment variable inside the container, for example `-e MODE=production`. Environment variables let you configure a container without changing its image, so the same image can run in test and production with different values. For several variables, repeat the flag or use `--env-file` with a file. Do not put real secrets in a command that you will keep in your shell history or scripts.

## -t {#t}
`-t` allocates a terminal for the container, so interactive programs behave properly, with colours and line editing. You nearly always use it together with `-i`, which keeps the input open, as `-it`. For example, `docker run -it ubuntu bash` gives you an interactive shell in a new container.

## A worked example {#example}
Let's start a web server, configure it and reach it from a browser:

```bash
docker run -d -p 8080:80 -e GREETING=hello --name web nginx
curl http://localhost:8080
docker logs web
docker stop web
```

Here is what each part does.

1. `-d` runs the server in the background, so the terminal is free for the next command.
2. `-p 8080:80` forwards port 8080 on your computer to port 80 inside the container.
3. `-e GREETING=hello` sets a variable that the application inside can read.
4. `--name web` gives the container a name to use in the later commands.
5. `curl http://localhost:8080` reaches the server through the published port, and returns the nginx welcome page.

Now count the effect of leaving out `-p`. The container runs, but `curl` fails with "connection refused", because nothing listens on host port 8080. And without `-d`, the first command would hold your terminal, and `curl` would need a second window.

Next, two copies of the same server. They need different host ports, since two things cannot listen on one port: `-p 8080:80` for the first and `-p 8081:80` for the second. Both containers use port 80 inside, which is fine, because each has its own private network.

## A common mistake {#mistake}
A common slip is to reverse the order and write `-p 80:8080` when the application listens on 80 inside. The first number is the host and the second is the container. Another slip is to pass secrets with `-e` in a command that gets saved in logs or history.

## Key points {#key-points}
- `-p host:container` (or `--publish`) makes a container's port reachable from outside.
- `-d` runs a container in the background. `-it` gives an interactive terminal.
- `-e` sets environment variables so one image can serve many settings.
- The first number in a port mapping is the host and the second is the container.
