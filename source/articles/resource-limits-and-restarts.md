# Limiting resources and restarting automatically

**Covers:** --memory, --cpus, --restart, restart policy, CPU, RAM

## Good neighbours and self-healing {#overview}
A container, left alone, may use as much of the machine as it wants, so one runaway program can slow or crash everything else. And when a container crashes at three in the morning, nobody wants to log in to start it again. Two groups of settings deal with this: limits that keep each container within its share, and rules that bring it back up when it stops.

## CPU {#cpu}
The CPU, or central processing unit, is the processor that executes a program's instructions. A machine has several cores, and containers share them. A container that loops forever may use a whole core or more, and slow its neighbours. CPU is a resource that can be shared out by time: a container that is limited does not crash, it just runs more slowly.

## RAM {#ram}
RAM, or random-access memory, is the fast working memory that a program uses for its data. Memory is different from CPU: it cannot be slowed down, only used or not. When a container tries to use more than its memory limit allows, the kernel kills the process, and Docker reports it as out of memory. Setting a limit therefore protects the machine, at the cost of a possible crash in the limited container.

## --cpus {#cpus}
`--cpus` limits how much processor time a container may use. `--cpus 1.5` means at most one and a half cores' worth of time, however many cores the machine has. A container with `--cpus 0.5` may use half of one core. Below the limit, it runs at full speed. Over it, it is slowed down.

## --memory {#memory}
`--memory` limits how much RAM a container may use, for example `--memory 512m` or `--memory 2g`. If the process tries to go beyond it, it is killed with an out-of-memory error. A related option, `--memory-swap`, controls whether it may also use disk swap space. Choose a limit with some room above what the application normally uses.

## --restart {#restart}
`--restart` sets when Docker restarts a container automatically, for example `--restart unless-stopped`. It is the flag form of a restart policy. The effect also applies when the machine reboots and the Docker service starts again.

## restart policy {#restart-policy}
A restart policy is the rule that tells Docker whether to start a container again after it stops or after a reboot. The options are:

| Policy | Behavior |
|---|---|
| `no` | Never restart (the default) |
| `on-failure` | Restart only if it exits with an error, optionally up to a number of times (`on-failure:5`) |
| `always` | Restart whatever the reason, including after a reboot |
| `unless-stopped` | Like always, but not if you stopped the container yourself |

For servers, `unless-stopped` is a sensible default. A restart policy is not a cure for a bug: a program that crashes at once will be restarted again and again, with growing delays.

## A worked example {#example}
Run a service with limits and a policy:

```bash
docker run -d --name api \
  --cpus 1 --memory 512m \
  --restart unless-stopped \
  myapp:1.0
docker stats --no-stream api
```

Walk through it:

1. `--cpus 1` caps the container at one core, so it cannot starve its neighbours.
2. `--memory 512m` caps its RAM at 512 megabytes.
3. `--restart unless-stopped` brings it back after a crash or a reboot.
4. `docker stats --no-stream` prints a snapshot of the usage, such as `CPU 12% MEM 310MiB / 512MiB`.

A sizing check: a server has 4 cores and 8 GB of RAM. If you run 6 containers with `--memory 1g`, they can use at most 6 GB together, leaving 2 GB for the system. If you had set no limits and one leaked memory, it could eat all 8 GB and take the machine down. With limits, only the leaking container is killed, and the restart policy starts it again within seconds.

Memory use of 310 MiB out of 512 is about 61 percent, which is healthy. If it showed 500 of 512, the next traffic peak would probably trigger a kill, so that is a signal to raise the limit or to fix the leak.

## A common mistake {#mistake}
Using `--restart always` to hide a crash loop. The service keeps starting and failing, and nobody notices. Check the logs and fix the cause. Another mistake is setting the memory limit lower than the application needs, which causes repeated out-of-memory kills.

## Key points {#key-points}
- CPU can be slowed by a limit. RAM cannot, so exceeding it kills the process.
- `--cpus` and `--memory` keep one container from harming the others.
- A restart policy such as `unless-stopped` starts a container again after a crash or reboot.
- A restart policy keeps applications up but does not fix bugs.
