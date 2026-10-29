# Running, stopping and removing containers

**Covers:** docker run, docker stop, docker rm

## The life of a container {#overview}
A container goes through a predictable life: it is created from an image, it runs, it stops and it is eventually deleted. Three commands move it along this path. Knowing exactly what each one does, and what it leaves behind, prevents the most common beginner surprises, such as a stopped container that still uses disk space.

## docker run {#docker-run}
`docker run` creates and starts a new container from an image. If you do not have the image locally, it first pulls it from a registry. A simple example is `docker run nginx`, which starts a web server in the foreground and ties up your terminal until you stop it.

Each call to `docker run` makes a new container, even for the same image. Running it three times gives three separate containers. To give a container a name you can refer to later, use `--name web`. Many options change how it runs, such as running in the background or publishing a port, and they are covered in their own articles.

## docker stop {#docker-stop}
`docker stop` shuts a running container down without deleting it. Docker first asks the main process to exit politely by sending the signal SIGTERM, and waits, 10 seconds by default. If the process is still running after that, Docker kills it with SIGKILL. A well-behaved application uses the polite signal to save its work and close connections.

After the stop, the container still exists in an exited state, with its file system and its logs. You can start it again with `docker start`. Its data stays on disk until you remove the container.

## docker rm {#docker-rm}
`docker rm` permanently deletes a stopped container from your machine. The name and the container's own file system disappear. Adding `-f` force-removes a container even if it is still running, by killing it first. Data kept in storage that Docker manages separately survives. Anything written only inside the container is gone.

To remove a container automatically when it stops, start it with `docker run --rm`. That is useful for one-off commands, and stops stopped containers from piling up.

## The lifecycle {#lifecycle}
Put together, the states are:

| State | How it gets there | What exists |
|---|---|---|
| Created | `docker create` or the start of `docker run` | The container and its file system |
| Running | `docker run` or `docker start` | A live process |
| Exited | `docker stop`, or the process ends | The container, its logs and file system |
| Removed | `docker rm` | Nothing |

`docker ps` lists running containers, and `docker ps -a` lists all of them, including the exited ones.

## A worked example {#example}
A short session:

```bash
docker run -d --name web nginx
docker ps
docker stop web
docker ps -a
docker rm web
docker ps -a
```

Step by step:

1. `docker run -d --name web nginx` pulls the image if needed and starts a container called `web` in the background. It prints the container's long ID.
2. `docker ps` shows `web` as running.
3. `docker stop web` sends SIGTERM, and nginx exits within a second. The command prints `web`.
4. `docker ps -a` still lists `web`, now with a status such as "Exited (0) 5 seconds ago". The container has not gone.
5. `docker rm web` deletes it.
6. The final `docker ps -a` no longer shows it.

If you had skipped the stop and tried `docker rm web` while it was running, Docker would refuse with an error saying the container is running. You would either stop it first or use `docker rm -f web`.

A tip on the disk: a stopped container holds on to its writable area. If a test creates 200 containers that exit and are never removed, and each wrote 50 MB, that is 10 GB of wasted space. Starting them with `--rm` avoids that.

## A common mistake {#mistake}
Believing that `docker stop` deletes the container. It does not, and stopped containers accumulate quietly. Another mistake is storing important data only inside a container and then removing it. Use separately managed storage for anything that must survive.

## Key points {#key-points}
- `docker run` creates and starts a container, and pulls the image if needed. Each call makes a new one.
- `docker stop` shuts it down politely and keeps it on disk. `docker rm` deletes it.
- `docker ps -a` shows exited containers too. Use `--rm` for one-off runs.
- Data inside a removed container is lost unless it is kept in separately managed storage.
