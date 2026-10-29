# Cleaning up, and when you need an orchestrator

**Covers:** docker system prune, orchestrator, Kubernetes

## Housekeeping now, scale later {#overview}
Docker is generous with disk space. Every pulled image, stopped container, unused network and build leaves something behind, and after a few months a developer machine can hold tens of gigabytes of leftovers. At the other end of the scale, a single machine eventually stops being enough, and something must manage containers across many. This article covers the cleanup command and the idea that takes over when one host is too small.

## docker system prune {#docker-system-prune}
`docker system prune` is a cleanup command that removes unused Docker resources: stopped containers, networks that no container uses, dangling images, which are untagged leftovers of earlier builds, and the build cache. It asks for confirmation before it deletes anything, unless you add `-f`.

By default it leaves volumes alone, because they may hold data that you need. Add `--volumes` to remove unused ones, and `-a` to remove all images that are not used by a container, not only dangling ones. Both are more aggressive, so read the summary before you agree.

Use `docker system df` first to see how much space Docker is using and how much of it could be reclaimed.

## orchestrator {#orchestrator}
An orchestrator is a tool that manages containers across many machines for you: it decides where each container runs, restarts the ones that crash, scales the number up or down, moves traffic to healthy copies and rolls out new versions without downtime. You declare what you want, such as "run five copies of this image", and the orchestrator works to make it true and to keep it true.

You need one when the manual approach breaks down. On a single machine, a script and a restart policy are enough. With ten machines and fifty containers, nobody can track by hand which host has room, what died and what must move when a machine fails.

## Kubernetes {#kubernetes}
Kubernetes is the most widely used container orchestrator. It began at Google, was released as open source in 2014 and is now maintained under the Cloud Native Computing Foundation. It runs containers across a cluster of machines, restarts failed ones, scales them on demand, balances traffic, rolls out updates gradually and rolls back if something goes wrong.

It is powerful and complex, with many concepts and a long learning curve. Alternatives exist, including Docker Swarm, which is simpler, and HashiCorp Nomad. Many teams use a managed Kubernetes service from a cloud provider, which runs the control part for them.

## A worked example {#example}
First, find out what is wasting space and reclaim it:

```bash
docker system df
docker system prune
docker system prune -a --volumes
```

A realistic result:

| Command | What it shows or removes | Space |
|---|---|---|
| `docker system df` | Images 14 GB (9 GB reclaimable), containers 1 GB, volumes 3 GB, build cache 5 GB | Report only |
| `docker system prune` | Stopped containers, unused networks, dangling images, build cache | About 6 GB freed |
| `docker system prune -a --volumes` | Also every unused image and unused volume | Another 9 GB or more, possibly data too |

Be careful with the last one: a volume that no container uses at the moment may still be a database you care about. Check `docker volume ls` first.

Now the point where cleanup is not the answer. A shop runs 40 containers on one server and traffic is growing. If a single server can run 50 containers, there is little room left, and if that machine fails, everything is down. With three servers and an orchestrator, you declare "10 copies of the web container". Kubernetes spreads them across the machines, about 3 or 4 on each. If one server dies, it starts replacements on the other two within seconds. Capacity rises from 50 to 150 containers with no change to the images.

The images you have built with Docker run unchanged. The orchestrator is a layer on top, not a replacement.

## A common mistake {#mistake}
Running `docker system prune -a --volumes` without looking, and deleting a database volume that was temporarily not in use. Always read what will be removed. A second mistake is adopting Kubernetes for a single small application. It adds a lot of concepts, and a single host with Compose is often enough.

## Key points {#key-points}
- `docker system prune` removes unused containers, networks, dangling images and cache. Volumes are kept unless you ask.
- Use `docker system df` to see the space first.
- An orchestrator places, restarts and scales containers across many machines.
- Kubernetes is the most widely used orchestrator. Use it when one host is no longer enough.
