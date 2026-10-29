# How Docker fits together: daemon, CLI, API and registries

**Covers:** Docker daemon, CLI, API, registry, Docker Hub, private registry, docker pull, docker push

## Two programs and a library of images {#overview}
When you type a Docker command, very little of the work happens in the program you are typing into. Docker is made of a client, a background program that does the real work, and a place where images are stored and shared. If you know the parts, you can understand error messages such as "cannot connect to the Docker daemon", and you can see how images travel from one machine to another. Let's meet the parts in the order a command meets them.

## CLI {#cli}
The CLI, or command-line interface, is the `docker` command you type in a terminal. It is the client. It does not run containers by itself. It turns what you type, such as `docker run nginx`, into a request and sends it on. Because the client is a separate program, it can also control a daemon on a different machine.

## API {#api}
An API, or application programming interface, is an agreed way for one program to ask another for something. Docker's daemon offers an API, usually over a local socket file or over the network. The CLI is only one program that uses it. Other tools, such as graphical dashboards and automation scripts, call the same API.

## Docker daemon {#docker-daemon}
The Docker daemon is the background program that actually creates, runs and stops your containers. On Linux, it runs as a service named `dockerd`. It also manages images, networks and storage. It waits for requests from the API and carries them out, so you do not talk to it directly. The CLI does that for you.

If the daemon is not running, every command fails with a connection error. On Windows and macOS, Docker Desktop starts a small Linux machine in which the daemon runs. So starting the Docker Desktop application is the first fix for that error.

## registry {#registry}
A registry is a server that stores container images, so you can upload (push) or download (pull) them instead of rebuilding from scratch every time. An image is stored under a name and a tag, such as `nginx:1.27`. Registries are how a build on your laptop reaches a server in the cloud: you push it once and the server pulls it.

## Docker Hub {#docker-hub}
Docker Hub is Docker's default public registry, an online library of ready-made images. When you run `docker pull nginx` without naming a registry, Docker looks in Docker Hub. It holds official images maintained by trusted publishers, such as nginx, redis and postgres, and millions of community images. Prefer official images or those from known publishers, because anybody can upload to the public library.

## private registry {#private-registry}
A private registry restricts who may see or download its images, unlike Docker Hub, which is public by default. Companies use one so that only their own servers and staff can get their application images. Options include a hosted service from a cloud provider, GitHub's container registry and a registry you run yourself. You log in with `docker login` before pushing or pulling.

## docker pull {#docker-pull}
`docker pull` downloads an image from a registry onto your machine without starting a container. For example, `docker pull postgres:16` fetches that image and stores it locally. If you run an image that you do not have, `docker run` pulls it for you. A separate pull is useful to prepare ahead of time, or to update to a newer version of a tag.

## docker push {#docker-push}
`docker push` uploads a local image to a registry so that others, or other machines, can pull it. The image's name must include where it is going, for example `registry.example.com/team/shop:1.4`. If you try to push `shop:1.4` with no registry in the name, Docker assumes Docker Hub and refuses without the right account name.

## A worked example {#example}
Let's move an image from a laptop to a server:

```bash
docker build -t registry.example.com/team/shop:1.4 .
docker login registry.example.com
docker push registry.example.com/team/shop:1.4
```

On the server:

```bash
docker pull registry.example.com/team/shop:1.4
docker run -d registry.example.com/team/shop:1.4
```

Here is the flow.

1. The CLI on the laptop sends the build request to the local daemon through the API. The daemon builds the image.
2. The push request makes the daemon upload the image layers to the private registry.
3. On the server, a different CLI asks a different daemon to pull. The daemon downloads the layers, skipping any it already has.
4. `docker run` asks the daemon to start a container from the image.

If the second command had failed with "Cannot connect to the Docker daemon", the CLI is fine but the daemon is not running or not reachable. If the push had failed with "denied", the CLI and the daemon work, but the registry does not accept your login.

## A common mistake {#mistake}
A common slip is to push to a public registry by accident. If you tag your company's image with only a short name and push, it may go to Docker Hub, where it is public. Always include the full registry address in the name for company images. Another slip is to trust any image on Docker Hub. Check the publisher and prefer official images.

## Key points {#key-points}
- The CLI is the client you type in. The daemon is the background program that does the work. They communicate through an API.
- A registry stores images. Docker Hub is the public default, and private registries restrict access.
- `docker pull` downloads an image, and `docker push` uploads one. The image name must include the registry.
- Connection errors usually mean that the daemon is not running.
