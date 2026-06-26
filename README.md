# DSO-Conduit

A backend - written in Python using Django - and a frontend - Written in TypeScript using Angular - containerized and managed in a single docker compose file, including a database.

- [Prerequisites](#prerequisites)
- [Quickstart](#quickstart)
- [Technical explanation](#technical-explanation)
    - [Database](#database)
    - [Backend](#backend)
    - [Frontend](#frontend)

## Prerequisites

- Git
- [Docker](https://docs.docker.com/engine/install)
- [Docker compose](https://docs.docker.com/compose/install/)

## Quickstart

1. Clone this project and it's submodules (backend & frontend repositories):
    ```bash
    git clone --recurse-submodules git@github.com:mickkc/dso-conduit.git
    ```
2. Enter the directory of the cloned repo:
    ```
    cd dso-conduit
    ```
3. Copy the example `.env`-file:
    ```
    cp .env.example .env
    ```
4. Change the secret key, api url, and optionally the other options:
    ```
    nano .env
    ```
    To generate a secure key, you can use `openssl rand -hex 32`.
5. Start the containers:
    ```
    docker compose up -d
    ```

## Technical explanation

This project contains two submodules: a frontend and a backend.

- The backend is written in Python, and uses Django. It provides an API that the frontend can use to fetch posts and tags. It also uses a postgresql database to store this data.
- The frontend is written in Typescript, and uses Angular. It connects to the backend to load its content.

Both projects include a `Dockerfile`, that is used in the `docker-compose.yml` to build the images and run the containers.

### Database

The database is a postgresql database. It is reachable by the backend through the docker compose's network. There isn't one explicitly specified, so Docker creates one automatically and adds the containers to it.

The database credentials (username, password, etc.) are read from the `.env` file. If they aren't specified there, default values will be used.
But you should at least specify a password.

It also includes a health check, so the backend only starts once the database is ready for connections. Otherwise, the backend fails and restarts a few times before finally connecting.

### Backend

The backend is available in the [conduit-backend submodule](/conduit-backend/). It depends on the database and has some more configuration options:

- `BACKEND_PORT`: This is the port the backend is reachable at from the host container. It needs to be accessible from the internet for reasons I will explain in the frontend section.
- `POSTGRES_HOST` & `POSTGRES_PORT`: The hostname and port to connect to the database. Because both the backend and the database are in the same network, the service name can be used for the host. The default postgresql port is `5432`.
- `POSTGRES_USER`, `POSTGRES_PASSWORD` & `POSTGRES_DB`: Used to connect and authenticate with the database. These values must be the same as in the database container, which is why they are read from the same options in the `.env` file.
- `API_SECRET_KEY`: A secret key used by Django. **Is option is required!** You can use `openssl rand -hex 32` to generate a secure one.
- `DJANGO_ALLOWED_HOSTS`: A comma separated list of hosts allowed to access your backend.
- `CORS_ORIGIN_WHITELIST`: A comma separated list of origins allowed to access your backend. Must include protocol, e.g. `http://127.0.0.1`.
- `DJANGO_SUPERUSER_USERNAME`, `DJANGO_SUPERUSER_EMAIL` & `DJANGO_SUPERUSER_PASSWORD`: Automatically creates a superuser if all of these variables are set. If a user with that name or email already exist, it is skipped.

### Frontend

The frontend is available in the [conduit-frontend submodule](/conduit-frontend/). It uses a pre-defined `API_URL` to connect to the backend.

This URL is injected into the source code before it's built and deployed. Because the frontend runs in the browser, that means that it's not part of the Docker network, and needs to access the backend through the internet.

Thats why you need two exposed ports: for the frontend to be accessible and for the backend to be used by the user's browser.

By default, the `API_URL` points to `http://localhost`, with the backend port and `/api` added after it (e.g. `http://localhost:8080/api`) for local deployments. When actually deploying, change this to you publicly accessible domain or ip.

You can also use `APP_PORT` to change the port the application is running and reachable on.