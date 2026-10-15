# WordPress with Docker

This repository contains a self-hosted WordPress installation running in Docker containers.

The setup consists of a WordPress container based on the official WordPress Apache image and a MySQL database container. Docker Compose is used to run the WordPress application, configure the database connection through environment variables, expose WordPress on port 8080, persist WordPress and database data using named volumes, connect both services through a dedicated Docker network, and automatically restart the containers if they terminate unexpectedly.

The project was created to practice containerization with Docker and Docker Compose, including service communication, environment-based configuration, persistent data, and container networking.

## Table of Contents

- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)
  - [Project Structure](#project-structure)
  - [Configuration](#configuration)
  - [Port Configuration](#port-configuration)
  - [Database Connection](#database-connection)
  - [Docker Network](#docker-network)
  - [Persistent Data](#persistent-data)
  - [Container Dependencies](#container-dependencies)
  - [Automatic Restart](#automatic-restart)
  - [Stopping and Restarting](#stopping-and-restarting)
- [Technologies](#technologies)

## Quickstart

### Prerequisites

Before starting the application, make sure the following software is installed:

- [Docker](https://www.docker.com/)
- Docker Compose (included with current Docker installations)
- Git

### Installation

Clone the repository:

```bash
git clone git@github.com:DaveHannemann/Wordpress.git
cd Wordpress
```

Create the local environment configuration from the provided example:

```bash
cp example.env .env
```

> [!NOTE]
> Adjust the values in `.env` according to your requirements. The `.env` file contains local configuration and should not be committed to Git.

Pull the required images:

```bash
docker compose pull
```

Start the containers in the background:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker compose ps
```

Check the container logs:

```bash
docker compose logs -f
```

WordPress is then available on the configured host port.

When the project is deployed on a cloud VM, WordPress can be accessed using:

```text
http://<your-vm-ip>:8080
```
> [!NOTE]
> Make sure that TCP port `8080` is allowed by the cloud VM firewall.


## Usage

### Project Structure

The main project files are organized as follows:

```text
.
├── docker-compose.yaml
├── example.env
├── .env
├── .gitignore
└── README.md
```

- `docker-compose.yaml` defines and configures the `wordpress` and `db` services.
- `example.env` provides an example configuration for the required environment variables.
- `.env` contains the local environment configuration and should not be committed to Git.
- `.gitignore` contains files and directories that should not be committed to Git.
- `README.md` contains the project documentation.

## Configuration

The application is configured through environment variables.

Create the local `.env` file from the provided example:

```bash
cp example.env .env
```

The available configuration variables include:

```env
HOST_PORT=8080
CONTAINER_PORT=80

WORDPRESS_DB_HOST=db:3306
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wordpress
WORDPRESS_DB_PASSWORD=CHANGE_ME

MYSQL_DATABASE=wordpress
MYSQL_USER=wordpress
MYSQL_PASSWORD=CHANGE_ME
MYSQL_ROOT_PASSWORD=CHANGE_ME
```

### Host Port

The port configuration is defined using:

```env
HOST_PORT=8080
```

`HOST_PORT` defines the port exposed by Docker on the host machine.

### Container Port

The WordPress container listens on port 80:

```env
CONTAINER_PORT=80
```

Port 80 is the standard HTTP port used by Apache inside the WordPress container.

Docker Compose maps the ports using:

```yaml
ports:
  - "${HOST_PORT}:${CONTAINER_PORT}"
```

>[!TIP]
>With the default configuration, the mapping is:

```text
8080 → 80
```
> [!IMPORTANT]  
>For a local installation, WordPress can therefore be accessed through:

```text
http://localhost:8080
```
> [!IMPORTANT]  
>For a cloud VM:

```text
http://<your-vm-ip>:8080
```

## Database Connection

WordPress uses MySQL as its database.

The database connection is configured using:

```env
WORDPRESS_DB_HOST=db:3306
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wordpress
WORDPRESS_DB_PASSWORD=CHANGE_ME
```

`WORDPRESS_DB_HOST` points to the Docker Compose service named `db`.

The MySQL service is configured using:

```env
MYSQL_DATABASE=wordpress
MYSQL_USER=wordpress
MYSQL_PASSWORD=CHANGE_ME
MYSQL_ROOT_PASSWORD=CHANGE_ME
```

The database name, user, and password used by WordPress must correspond to the database configuration provided to MySQL.

The database container uses the official MySQL 8.0 image:

```yaml
db:
  image: mysql:8.0
```

## Docker Network

Both services are connected to the same dedicated Docker network:

```yaml
networks:
  wordpress-network:
```

The WordPress service uses:

```yaml
networks:
  - wordpress-network
```

The database service uses the same network:

```yaml
networks:
  - wordpress-network
```

This allows the WordPress container to communicate directly with the MySQL container using the service name:

```text
db
```

The database does not need to expose port 3306 to the host because the communication between WordPress and MySQL takes place inside the Docker network.

## Persistent Data

Both WordPress and MySQL use Docker-managed named volumes.

The WordPress volume is mounted to:

```yaml
volumes:
  - wordpress-data:/var/www/html
```

The MySQL volume is mounted to:

```yaml
volumes:
  - db-data:/var/lib/mysql
```

The volumes are defined in the Compose configuration:

```yaml
volumes:
  wordpress-data:
  db-data:
```

The WordPress volume stores persistent WordPress data such as:

- WordPress files
- uploaded media
- installed plugins
- installed themes
- generated WordPress files

The MySQL volume stores persistent database data such as:

- WordPress users
- posts and pages
- WordPress settings
- plugin data
- other database records

The data remains available when the containers are stopped, restarted, or recreated.

The available Docker volumes can be viewed with:

```bash
docker volume ls
```

> [!WARNING]
> Do not use `docker compose down -v` unless the persistent application and database data should be deleted. The `-v` option removes the Docker volumes.

## Container Dependencies

The WordPress service depends on the database service:

```yaml
depends_on:
  - db
```

This ensures that Docker Compose starts the database service before starting WordPress.

`depends_on` controls the service startup order. It does not guarantee that MySQL is already fully ready to accept connections when the WordPress container starts.

## Automatic Restart

Both services use:

```yaml
restart: unless-stopped
```

This causes Docker to automatically restart a container if its process terminates unexpectedly.

The policy applies to both:

- WordPress
- MySQL

The containers remain stopped if they are intentionally stopped by the user.

## Stopping and Restarting

To stop the application:

```bash
docker compose down
```

The containers are removed, but the named volumes remain available.

To start the application again:

```bash
docker compose up -d
```

To restart the running containers:

```bash
docker compose restart
```

To view the container status:

```bash
docker compose ps
```

To view the logs:

```bash
docker compose logs -f
```

The persistent data stored in the named volumes is not removed when the containers are stopped or recreated.

## Technologies

- Docker
- Docker Compose
- WordPress
- Apache
- PHP
- MySQL 8.0
- Docker Named Volumes
- Docker Networks
- Environment Variables
- Git / GitHub