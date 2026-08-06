# Smart15 Installation

This guide explains how to install and start Smart15 on a Linux server or in a Windows + WSL2 environment.

Smart15 runs with Docker. This means its components — GeoLibre interface, Smart15 plugin, and processing backend — are prepared and started in a standardized way without manually installing Python, Node.js, or geospatial libraries.

## Quick overview

Once the prerequisites are installed, setup typically comes down to three commands:

```bash
git clone --recurse-submodules <REPOSITORY_URL>
cd Smart15-Acessibilidade
./scripts/start.sh
```

The first run may take a few minutes because Docker images need to be built.

## Concepts used

- **Terminal:** window used to type commands into the operating system. On Ubuntu, open it with `Ctrl` + `Alt` + `T`.
- **Git:** tool used to download and update the project source code.
- **Docker:** tool that runs the application in isolated containers with all required dependencies.
- **Docker Compose:** Docker component that starts, stops, and manages Smart15 services together.
- **WSL2:** Linux environment inside Windows. To use it, Docker Desktop must be installed and integrated with the Linux distribution.

## 1. Prerequisites

Required:

- Git;
- Docker;
- Docker Compose;
- `curl`;
- `ss`;
- `timeout`.

The user running the installation must also have permission to use Docker.

### Installation on Ubuntu or Debian

Update the package list and install the basic tools:

```bash
sudo apt update

sudo apt install -y \
  git \
  curl \
  iproute2 \
  coreutils \
  docker.io \
  docker-compose-plugin
```

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

After this command, log out of Linux and log back in. In WSL2, close the distribution and open it again.

> Without this step, you may need to use `sudo` before Docker commands.

### Verify the environment

Run the commands below. Each should display an installed version:

```bash
git --version
docker --version
docker compose version
```

You can also check if Docker is working:

```bash
docker run hello-world
```

## 2. Download the project

In the terminal, go to the folder where you want to keep the system and run:

```bash
git clone --recurse-submodules <REPOSITORY_URL>
cd Smart15-Acessibilidade
```

Replace `<REPOSITORY_URL>` with the Git repository address provided by the project team.

The `--recurse-submodules` option also downloads GeoLibre, the component that provides the geospatial interface used by Smart15.

## 3. Start the application

Inside the `Smart15-Acessibilidade` folder, run:

```bash
./scripts/start.sh
```

The script automates environment preparation. It performs:

1. prerequisite validation;
2. creation of the `.env` configuration file when needed;
3. GeoLibre initialization;
4. Smart15 plugin compilation;
5. plugin synchronization with GeoLibre;
6. Docker Compose configuration validation;
7. port conflict checks;
8. Docker image build;
9. container startup;
10. backend and interface health checks;
11. presentation of access addresses.

At the end, the application will be available at the default addresses:

- **Smart15 application:** <http://localhost:8080>
- **Technical backend:** <http://localhost:8000>
- **API technical documentation:** <http://localhost:8000/docs>

When the application is deployed on a server, replace `localhost` with the server IP or hostname. Example:

```text
http://192.168.1.50:8080
```

For normal use, access the application on port `8080`.

## 4. Port configuration

External ports are defined in the `.env` file:

```dotenv
SMART15_BACKEND_PORT=8000
SMART15_WEB_PORT=8080
```

On first run, the `.env` file is created automatically from `.env.example`.

If the default port is already in use, you can temporarily start on other ports:

```bash
SMART15_BACKEND_PORT=8001 \
SMART15_WEB_PORT=8081 \
./scripts/start.sh
```

In this example, the application will be accessible at:

```text
http://localhost:8081
```

## Daily operation

### Check if the system is running

```bash
docker compose \
  --env-file .env \
  --file deploy/compose/docker-compose.yml \
  ps
```

Expected services are:

- `backend`: service that runs simulation processing;
- `geolibre`: Smart15 web interface.

The `running` or `Up` state indicates the service is active.

### Follow system logs

```bash
docker compose \
  --env-file .env \
  --file deploy/compose/docker-compose.yml \
  logs -f
```

To exit log view, press `Ctrl` + `C`.

### Stop the application

```bash
docker compose \
  --env-file .env \
  --file deploy/compose/docker-compose.yml \
  down
```

This command stops the application but preserves project files and Docker images.

### Start again

```bash
./scripts/start.sh
```

### Rebuild the environment

Use this procedure when the project is updated or containers need to be rebuilt:

```bash
docker compose \
  --env-file .env \
  --file deploy/compose/docker-compose.yml \
  down --remove-orphans

./scripts/start.sh
```

## Common issues

### `git`, `docker`, or `docker compose` command not found

This indicates the corresponding tool is not installed or the terminal needs to be reopened after installation. Repeat the [Prerequisites](#1-prerequisites) section.

### Permission error running Docker

If a permission denied message appears when running Docker, check whether the user was added to the `docker` group:

```bash
sudo usermod -aG docker $USER
```

Then log out and log back into the system.

### Port in use

Change the ports in the `.env` file or use temporary values:

```bash
SMART15_BACKEND_PORT=8001 \
SMART15_WEB_PORT=8081 \
./scripts/start.sh
```

### The application does not open from another computer on the network

Check:

1. whether the application started correctly;
2. whether the server IP address was used in the browser;
3. whether the configured port is allowed by the firewall and local network;
4. whether services appear as `running` in the `docker compose ... ps` command.

### Outdated plugin in the browser

This guidance is intended for the technical team after the plugin is updated:

1. open the browser console;
2. run the command below;
3. reload the page with `Ctrl` + `Shift` + `R`.

```javascript
localStorage.removeItem('geolibre.pluginBundlePins')
```

### Verificar os logs de um único serviço

**Logs do backend:**

```bash
docker compose \
  --env-file .env \
  --file deploy/compose/docker-compose.yml \
  logs -f backend
```

**Logs da interface GeoLibre:**

```bash
docker compose \
  --env-file .env \
  --file deploy/compose/docker-compose.yml \
  logs -f geolibre
```
