# How containers talk to each other: networks, IPs and DNS

**Covers:** network, networking, DNS, IP

## Private networks inside one machine {#overview}
A container is isolated, but most applications need to talk to others: a web service to a database, a worker to a queue. Docker provides virtual networks for this. If you understand how addresses and names work, you can solve the most common problem in this area: "my app cannot find my database". Let's begin with addresses.

## IP {#ip}
An IP address, short for Internet Protocol address, is the number that identifies a device or a container on a network, such as `172.18.0.3`. Every container on a Docker network gets its own address. The numbers are assigned automatically and are not fixed: when you remove and recreate a container, it may get a different address. For that reason you should not write IP addresses into your configuration.

## DNS {#dns}
DNS, the Domain Name System, is the service that turns names such as `example.com` into IP addresses. It works like a phone book: you look up a name and get a number. Docker runs a small DNS service of its own for user-defined networks, so containers can look each other up by name, such as `db`, and receive the current IP address. The names update automatically when containers change.

## network {#network}
A network, in Docker, is a virtual network that connects chosen containers so that they can reach each other. Containers on the same network can communicate. Containers on different networks are separated from each other, which is a simple and effective way to limit what can talk to what.

Docker has several types. The default `bridge` network connects containers on one host, but on the default one, containers can reach each other only by IP address. A user-defined bridge network, which you create yourself, adds the automatic name lookup. Other types include `host`, where the container shares the machine's own network, and `none`, with no network at all.

## networks {#networks}
You can create as many networks as you need with `docker network create`, and list them with `docker network ls`. A common design uses one network for the front end and the database, and another for the front end and the outside world, so the database is not reachable from outside.

## networking {#networking}
Networking is the general topic of how containers communicate with each other, with the host and with the outside world. Three directions matter. One is container to container, over a shared network, by name. Another is the host or the outside to a container, through a published port. The third is container to the outside, and outgoing requests work by default.

## A worked example {#example}
A web app needs a database. Let's wire them together with a user-defined network:

```bash
docker network create appnet
docker run -d --name db --network appnet -e POSTGRES_PASSWORD=example postgres:16
docker run -d --name web --network appnet -p 8080:8000 -e DB_HOST=db myapp:1.0
docker exec web ping -c 1 db
```

Here is what happens.

1. `docker network create appnet` makes a private virtual network.
2. The database joins it with the name `db`. The web app also joins it, and is told that the database is at the host name `db`.
3. When the web app connects to `db`, Docker's DNS looks the name up and returns the database container's current IP address, for example `172.18.0.2`.
4. Only the web app has a published port (`-p 8080:8000`). The database has none, so nothing outside can reach it, but the web app can.
5. The `ping` command inside `web` proves that the name `db` resolves.

Now suppose the database container is recreated and gets the address `172.18.0.5`. The app needs no change, because it uses the name, and DNS now returns the new IP address. If the address had been written into the configuration, the connection would have broken.

Count the exposure. The setup publishes 1 port out of 2 applications. If you had published the database port too, the database would be reachable from the host and possibly the internet, doubling the places an attacker could probe.

## A common mistake {#mistake}
A common slip is to use the default bridge network and expect names to work. They do not work there, so the connection fails with "name not resolved". Create your own network. Another slip is to use `localhost` to reach another container. Inside a container, `localhost` means that container itself.

## Key points {#key-points}
- Each container on a network gets an IP address, which can change.
- DNS lets containers on a user-defined network find each other by name.
- Containers on the same network can talk. Different networks are isolated.
- Publish only the ports that the outside needs, and use names, not addresses.
