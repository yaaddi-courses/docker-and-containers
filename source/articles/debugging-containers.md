# Debugging a container: logs, a shell and inspection

**Covers:** docker logs, docker exec, docker inspect, JSON, CSV

## Looking inside something designed to be closed {#overview}
A container is built to be isolated, and that makes problems harder to see. The application is not behaving, and you do not know why. Three commands cover most investigations, from the quickest check to the deepest. Use them in order: read what the application said, look around inside, and check how it was configured.

## docker logs {#docker-logs}
`docker logs` shows the console output of a container: everything the application has printed to its standard output and standard error since it started. It is the first place to look, because most applications print their errors there. Useful options: `-f` follows the output live, like watching a log file grow, `--tail 50` shows only the last 50 lines and `--since 10m` shows the last ten minutes.

For this to work, your application should log to the console and not to a file inside the container. Many images are configured this way on purpose.

## docker exec {#docker-exec}
`docker exec` runs a command inside a container that is already running. It attaches to the existing container instead of starting a new one. The most common use is to open an interactive shell: `docker exec -it web sh`. The flags mean interactive and with a terminal. Once inside, you can list files, read configuration, try a network request and check which processes run.

Remember that you changed nothing in the image. Anything you edit by hand inside the container is lost when it is removed, so use the shell to investigate, not to fix permanently. Some slim images have no shell at all, in which case `docker exec` of `sh` will fail, and you would debug through logs or a debug image instead.

## docker inspect {#docker-inspect}
`docker inspect` returns the full configuration and state of a container as JSON, including its network settings, its mounted volumes, its environment variables, its automatic-restart setting and why it exited. It answers questions such as "which port is really published?" and "what exit code did it finish with?".

The output is long, so filter it with `--format` and a template: `docker inspect --format '{{.State.ExitCode}}' web` prints just the exit code.

## JSON {#json}
JSON, or JavaScript Object Notation, is a text format for structured data, written as keys and values in curly braces, for example `{"name": "web", "running": true}`. Many tools, Docker among them, use it for output that programs can read. Because it is plain text with a regular shape, tools such as `jq` can search and reshape it.

## CSV {#csv}
CSV, or comma-separated values, is a plain-text table format where each line is a row and commas separate the columns. It opens directly in a spreadsheet. With a format template, Docker can print lists such as `docker ps --format '{{.ID}},{{.Names}},{{.Status}}'`, which you can save as a CSV file for a report.

## A worked example {#example}
A container named `shop` keeps stopping. Investigate in order:

```bash
docker ps -a --format '{{.Names}},{{.Status}}'
docker logs --tail 20 shop
docker inspect --format '{{.State.ExitCode}} {{.State.OOMKilled}}' shop
docker exec -it shop sh
```

Step by step:

1. The first command lists every container as CSV-style lines. You see `shop,Exited (137) 2 minutes ago`.
2. The logs show the last 20 lines. They stop suddenly in the middle of work, with no error message from the application.
3. `docker inspect` prints `137 true`: the exit code is 137, and the OOMKilled flag is true.
4. The meaning: exit code 137 means the process was killed, and OOMKilled means the cause was running out of memory, so the kernel stopped it. Without that check, you could have lost an hour looking for a bug in the code.
5. The last command would open a shell in a running container, but this one has exited, so you would first start it again, or run a fresh container and look around.

The fix is outside the container: give it more memory, or find why it uses so much. The investigation took three short commands and a minute.

## A common mistake {#mistake}
Opening a shell and changing files by hand to "fix" a problem, then forgetting that the change vanishes when the container is replaced. Make the fix in the Dockerfile or the configuration. Another mistake is skipping the logs. Most problems are explained in the last 20 lines.

## Key points {#key-points}
- `docker logs` shows what the application printed. Start there.
- `docker exec` runs a command, usually a shell, inside a running container.
- `docker inspect` returns the full configuration and state as JSON. Filter it with `--format`.
- Fixes belong in the Dockerfile or the configuration, not in a shell session.
