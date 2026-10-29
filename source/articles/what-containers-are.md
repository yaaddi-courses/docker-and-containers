# What a container is, and how it compares with a virtual machine

**Covers:** container, kernel, VM, OS

## Packaging software so it runs anywhere {#overview}
Anyone who has installed software knows the problem: it works on one computer and fails on another because of a different version of a library, a missing tool or a different setting. Containers solve this by packaging an application together with everything it needs, then running it in an isolated space. To see why this is useful, and why it is lighter than the older approach of virtual machines, you need a few basic ideas about how a computer is organized.

## OS {#os}
An OS, or operating system, is the software that manages a computer's hardware and runs other programs. Windows, macOS and Linux are operating systems. It decides which program uses the processor and for how long, hands out memory, reads and writes files and talks to devices. Every program runs on top of it.

## kernel {#kernel}
The kernel is the core part of an operating system. It talks directly to the hardware and manages processes, memory and files. Everything else in the OS, such as the desktop, the command shell and the tools, is built on top of it.

The kernel is important for understanding containers, because a container does not have its own kernel. It shares the kernel of the machine it runs on. That is the main reason that containers are small and fast.

## container {#container}
A container is a lightweight, isolated box that holds an application with everything it needs to work: its libraries, its settings and its files. Inside the box, the application sees its own private world: its own file system, its own address for reaching others and its own list of processes. Outside, it is an ordinary process on the host's kernel, which uses Linux features called namespaces, which limit what a process can see, and control groups, which limit what it can use.

Because the box carries its dependencies, the application runs the same on a laptop, a test server and a production machine. Containers start in a fraction of a second and take little memory, so one machine can host dozens or hundreds of them.

## containers {#containers}
"Containers", in the plural, means the whole practice and technology: a system might "live in containers", or a team might "containerize" an application. Tools such as Docker made the approach widely usable by providing a simple way to build, share and start them.

## VM {#vm}
A VM, or virtual machine, is a simulated computer, complete with its own full operating system, running on top of the real one. A program called a hypervisor divides the real hardware among several VMs. Each VM thinks it is a whole separate computer.

VMs give very strong isolation, because each has its own kernel. The cost is weight: each VM includes a complete operating system, often several gigabytes, and takes tens of seconds or minutes to start.

## A worked example {#example}
Compare running three copies of a small web application on one server.

| | Three virtual machines | Three containers |
|---|---|---|
| What each instance includes | App + libraries + a full operating system | App + libraries only |
| Typical size of each | About 2,000 MB | About 100 MB |
| Total size of three | 6,000 MB | 300 MB (and shared parts are stored once) |
| Start time | About 30 seconds | Under 1 second |
| Kernel | Three, one per VM | One, shared by all |
| Isolation | Very strong | Strong, but they share the host's kernel |

The arithmetic: three VMs at 2,000 MB use 6,000 MB. Three containers at 100 MB use 300 MB, which is about 20 times less. On a server with 16 GB of memory, you might fit 6 or 7 such VMs and many dozens of containers.

The trade-off is that containers share the kernel, so a kernel flaw affects them all, and a Linux container cannot host a different kernel type, such as Windows software, without a VM in between. For security-sensitive cases, some systems place containers inside VMs, combining both.

## A common mistake {#mistake}
Thinking of a container as a small virtual machine. It is a process with a restricted view of the system. That difference explains why containers start so fast, and why they are not as isolated as VMs. Another mistake is treating a container as a place to store important data. Containers are disposable, and their data needs to be kept elsewhere.

## Key points {#key-points}
- An OS manages the hardware, and its kernel is the core that does so directly.
- A container packages an app with its dependencies and keeps it isolated; other things on the host stay out of its way.
- Containers share the host's kernel. A VM carries its own OS and kernel.
- Containers are much smaller and faster to start than VMs, with slightly weaker isolation.
