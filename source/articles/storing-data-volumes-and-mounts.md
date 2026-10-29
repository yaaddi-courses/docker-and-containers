# Keeping data: volumes and bind mounts

**Covers:** -v, bind mount, volume

## Containers forget {#overview}
Everything a container writes lives in a thin writable layer on top of the image, and it disappears when the container is removed. For a web server that is fine. For a database, it is a disaster. To keep data, you attach storage from outside the container. Docker offers two main ways to do it, and the same flag handles both.

## -v {#v}
`-v` mounts storage into a container, written `source:destination`, so the data outlives the container. For example, `-v dbdata:/var/lib/postgresql/data` attaches a volume called `dbdata` at that path in the container. When the first part is a path on your computer, such as `-v /home/me/site:/usr/share/nginx/html`, the folder is shared directly. Add `:ro` at the end for read-only.

The long form is `--mount`. It is more explicit, and it gives clearer error messages when something is wrong.

## volume {#volume}
A volume is storage that Docker creates and manages for you, so you do not have to point at a folder yourself. Docker keeps it in its own area on the host. Its data survives when the container stops or is deleted, and you can attach the same volume to several containers, one after another or together.

Volumes are the recommended choice for application data such as databases. They are portable, easy to back up and work the same on every operating system. Create one with `docker volume create dbdata`, or let `-v dbdata:/path` create it on first use.

## volumes {#volumes}
"Volumes", in the plural, is the general name for this kind of storage. `docker volume ls` lists them, and `docker volume rm` deletes one. Notice that removing a container does not remove its volumes. Over time, unused volumes pile up, and cleaning them takes a separate step.

## bind mount {#bind-mount}
A bind mount links one specific folder on your host straight into a container, so the container reads and writes the real files that live on the host. You choose the exact path. Changes appear on both sides at once.

This is ideal for development. If you mount your source code folder into the container, each edit in your editor is visible at once inside it, with no rebuild. There are drawbacks. The path must exist on the host and is specific to your machine, the container may write files that your user cannot easily edit, and permissions can be confusing.

## A worked example {#example}
Let's run a database with a volume, delete the container and check that the data survived:

```bash
docker run -d --name db -e POSTGRES_PASSWORD=example -v dbdata:/var/lib/postgresql/data postgres:16
# ...create some tables and rows...
docker rm -f db
docker run -d --name db2 -e POSTGRES_PASSWORD=example -v dbdata:/var/lib/postgresql/data postgres:16
```

Here is what happens.

1. The first container writes its database files to `dbdata`, a volume that Docker creates.
2. `docker rm -f db` stops and deletes the container, and its own writable layer is gone.
3. The volume `dbdata` remains. You can see it in `docker volume ls`.
4. The second container mounts the same volume and finds the tables and rows, as if nothing had happened.

Now a bind mount for development:

```bash
docker run -d -p 8080:80 -v "$(pwd)/site:/usr/share/nginx/html:ro" nginx
```

Edit `site/index.html` on your computer and refresh the browser. The change is visible straight away, because the container is reading the same file.

Let's compare the two. If you had used no `-v` for the database, the removal would have deleted 100 percent of the data. With a volume, you lose none. With the bind mount, the cost is that the example only works if `./site` exists on your machine.

## A common mistake {#mistake}
A common slip is to run a database without any volume. It works until the container is replaced, and then every record is gone. Another slip is to use bind mounts for production data. They tie the container to one path on one machine. Use volumes there, and back them up.

## Key points {#key-points}
- Data inside a container is lost when it is removed. Mount storage to keep it.
- `-v source:destination` attaches a volume or a host folder.
- A volume is managed by Docker and is the best choice for application data.
- A bind mount shares a specific host folder, which is ideal for editing code in development.
