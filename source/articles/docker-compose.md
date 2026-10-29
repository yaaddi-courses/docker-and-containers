# Docker Compose: a whole application in one file

**Covers:** Docker Compose, Docker-Compose, docker-compose.yml, services, YAML, -f

## From a dozen commands to one {#overview}
A real application is rarely one container. It might be a web app, a database and a cache, each needing its own `docker run` command with ports, variables, volumes and a network. Typing and remembering all of that is error-prone. Docker Compose lets you describe the entire application in one file and start it with a single command.

## YAML {#yaml}
YAML is a plain-text format for structured configuration, written with indentation instead of brackets. A list is a column of lines starting with a dash, and nested settings are indented under their parent. It is easy for people to read, and it is the format of the Compose file.

Two rules cause most of the trouble. Indentation matters, and you must use spaces, not tabs. And a colon followed by a space separates a key from its value. A wrong indent can silently change the meaning or cause an error.

## Docker Compose {#docker-compose}
Docker Compose is a tool for defining and running several related containers together, such as an app and its database, with a single command, instead of running separate commands for each. You describe the setup in a file, and Compose creates the network, the volumes and the containers for you, in the right order.

The command is `docker compose up -d` to start everything in the background, `docker compose down` to stop and remove it, and `docker compose logs` to see the output of all of the services.

## Docker-Compose {#docker-compose-hyphen}
With a hyphen, `docker-compose` is the name of the older, separate version of the tool, a standalone program. The newer one is built into Docker and is called as `docker compose`, with a space. The file format is the same, and many guides online still use the hyphenated command. If one is missing, try the other.

## docker-compose.yml {#docker-compose-yml}
`docker-compose.yml` is the YAML file where you describe a Compose setup. Newer versions also accept the shorter name `compose.yaml`. Under a top-level key named `services`, you list each container, with its image, ports, environment variables and other settings. Other top-level keys such as `volumes` and `networks` define shared storage and networks.

Keep the file in the project's repository, so that anyone who clones it can start the full application with one command.

## services {#services}
`services` is the top-level section of the Compose file where you list each container you want Compose to manage. Each entry has a name, such as `web` or `db`, which also becomes its network name, so one service can reach another by that name. Each service has its own image or build instructions, ports, variables and so on.

## -f {#f}
`-f` names a file to use. With Compose it picks a different Compose file: `docker compose -f compose.prod.yml up -d`. You can give it several times, and later files override earlier ones, which lets you keep a base file and a small file for each environment. With `docker build`, `-f` names a Dockerfile with a different name.

## A worked example {#example}
A web application with a database in one file:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:8000"
    environment:
      DB_HOST: db
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - dbdata:/var/lib/postgresql/data

volumes:
  dbdata:
```

Read it:

1. Two services are defined: `web`, built from the Dockerfile in this folder, and `db`, from a ready-made image.
2. The `web` service publishes port 8000 as 8080 and finds the database by the name `db`.
3. `depends_on` starts the database first.
4. The `db` service stores its data on the volume `dbdata`, which is declared at the bottom.
5. Compose automatically creates a network, so `web` and `db` can reach each other by name.

Start everything with `docker compose up -d`, check the state with `docker compose ps` and stop and remove it with `docker compose down`. Add `--volumes` to `down` only if you also want to delete the data.

To run a variant, create `compose.dev.yml` with only the differences, such as an extra published port, and start it with `docker compose -f compose.yml -f compose.dev.yml up -d`.

Compare the effort: starting this by hand needs a network command, two `docker run` commands with 8 flags between them and the right order. With the file, one command does all of it, and the setup is the same on every machine.

## A common mistake {#mistake}
Using tabs in the YAML file, or a wrong indent, which produces confusing errors. Use two spaces and an editor that shows whitespace. Another mistake is that `depends_on` only waits for the container to start, not for the database to be ready. Add a health check or make the application retry.

## Key points {#key-points}
- Compose describes a multi-container application in one YAML file and starts it with one command.
- The `services` section lists the containers. They reach each other by service name.
- `docker compose` (with a space) is the current tool. `docker-compose` is the older one.
- `-f` selects a file, and several files can be combined for different environments.
