# Flask App with MySQL — Docker Two-Tier Application

A simple **two-tier web application** built with **Flask and MySQL**, containerized using Docker.

The Flask application allows users to submit messages, stores them in MySQL, and displays the stored messages through the frontend.

## Architecture

```text
                    User
                     |
                     | HTTP :80
                     v
              +-------------+
              |    Flask    |
              |  Container  |
              +-------------+
                     |
                     | Docker Network
                     | MySQL :3306
                     v
              +-------------+
              |    MySQL    |
              |  Container  |
              +-------------+
                     |
                     v
              +-------------+
              | mysql_data  |
              |   Volume    |
              +-------------+
```

## Technologies

- Python
- Flask
- MySQL 8.4
- Docker
- Docker Compose
- Docker Networks
- Docker Volumes
- MySQL Healthcheck
- Docker `depends_on`

## Project Structure

```text
two-tier-flask-app/
│
├── app.py
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── message.sql
├── README.md
└── templates/
    └── ...
```

## Prerequisites

Make sure you have:

- Docker
- Docker Compose
- Git

Check Docker:

```bash
docker --version
```

Check Docker Compose:

```bash
docker compose version
```

## Clone the Repository

```bash
git clone https://github.com/Ahmadraza9091/docker_practice.git
```

Navigate into the project:

```bash
cd docker_practice
```

## Run Using Docker Compose

Build and start the application:

```bash
docker compose up -d --build
```

Check running containers:

```bash
docker compose ps
```

Expected services:

```text
flask-app
mysql-db
```

## Docker Compose Services

### Flask

The Flask container is built from the project's `Dockerfile`.

It connects to MySQL using:

```text
MYSQL_HOST=mysql-db
MYSQL_USER=Ahmad
MYSQL_PASSWORD=root
MYSQL_DB=devops
```

The Flask application is exposed on:

```text
Port 80
```

### MySQL

The application uses:

```text
MySQL 8.4
```

MySQL configuration:

```text
Database: devops
User: Ahmad
Password: root
Port: 3306
```

The MySQL container is named:

```text
mysql-db
```

## Docker Network

Both containers communicate through a Docker bridge network:

```text
two-tier
```

Check the network:

```bash
docker network ls
```

Inspect it:

```bash
docker network inspect two-tier
```

The Flask application connects to MySQL using:

```text
mysql-db
```

The Flask application should **not** use `localhost` for MySQL because `localhost` refers to the Flask container itself.

## MySQL Healthcheck

The MySQL service has a healthcheck to verify that MySQL is ready to accept queries.

Check the MySQL health status:

```bash
docker inspect --format='{{.State.Health.Status}}' mysql-db
```

Expected:

```text
healthy
```

## Flask Dependency on MySQL

The Flask container depends on MySQL being healthy:

```yaml
depends_on:
  mysql:
    condition: service_healthy
```

This makes Docker Compose wait for the MySQL service to pass its healthcheck before starting Flask.

## Persistent MySQL Data

MySQL uses a Docker named volume:

```text
mysql_data
```

The volume is mounted at:

```text
/var/lib/mysql
```

Check volumes:

```bash
docker volume ls
```

Inspect the volume:

```bash
docker volume inspect two-tier-flask-app_mysql_data
```

Because MySQL data is stored in the volume, removing the containers does not remove the database data.

For example:

```bash
docker compose down
```

Then:

```bash
docker compose up -d
```

The database data will remain.

### WARNING

Do not use:

```bash
docker compose down -v
```

unless you intentionally want to delete the MySQL volume and its data.

## Access the Application

If running locally:

```text
http://localhost
```

If running on AWS EC2:

```text
http://EC2-PUBLIC-IP
```

Make sure the EC2 Security Group allows inbound HTTP traffic on:

```text
TCP 80
```

## Check Container Logs

View all logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

Flask logs:

```bash
docker compose logs flask
```

MySQL logs:

```bash
docker compose logs mysql
```

## Access the Containers

Enter the Flask container:

```bash
docker exec -it flask-app bash
```

Enter the MySQL container:

```bash
docker exec -it mysql-db bash
```

Open MySQL directly:

```bash
docker exec -it mysql-db mysql -uAhmad -p
```

Then:

```sql
USE devops;
```

Show tables:

```sql
SHOW TABLES;
```

## Create the Messages Table

If the table does not already exist:

```sql
CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT
);
```

Check the table:

```sql
SHOW TABLES;
```

Check stored messages:

```sql
SELECT * FROM messages;
```

The project also contains:

```text
message.sql
```

which can be used for database setup.

## Stop the Application

Stop the containers:

```bash
docker compose stop
```

Start them again:

```bash
docker compose start
```

## Stop and Remove Containers

```bash
docker compose down
```

This removes the containers but keeps the MySQL volume.

Start the project again:

```bash
docker compose up -d
```

## Rebuild the Application

If you modify the Dockerfile, Python dependencies, or application code:

```bash
docker compose up -d --build
```

For a completely fresh image build:

```bash
docker compose build --no-cache
```

Then:

```bash
docker compose up -d
```

## Useful Docker Commands

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

List images:

```bash
docker image ls
```

List networks:

```bash
docker network ls
```

List volumes:

```bash
docker volume ls
```

View container logs:

```bash
docker logs <container>
```

Execute a command inside a container:

```bash
docker exec -it <container> bash
```

## Run Without Docker Compose

The application can also be run manually without Docker Compose.

### Create the Network

```bash
docker network create two-tier
```

### Build the Flask Image

```bash
docker build -t flask-app:01 .
```

### Run MySQL

```bash
docker run -d \
    --name mysql-db \
    --network two-tier \
    -v mysql_data:/var/lib/mysql \
    -e MYSQL_DATABASE=devops \
    -e MYSQL_ROOT_PASSWORD=root \
    -e MYSQL_USER=Ahmad \
    -e MYSQL_PASSWORD=root \
    mysql:8.4
```

### Run Flask

```bash
docker run -d \
    --name flask-app \
    --network two-tier \
    -e MYSQL_HOST=mysql-db \
    -e MYSQL_USER=Ahmad \
    -e MYSQL_PASSWORD=root \
    -e MYSQL_DB=devops \
    -p 80:80 \
    flask-app:01
```

Check:

```bash
docker ps
```

## Troubleshooting

### Check Flask logs

```bash
docker compose logs flask
```

### Check MySQL logs

```bash
docker compose logs mysql
```

### Check MySQL health

```bash
docker inspect --format='{{.State.Health.Status}}' mysql-db
```

### Check network connectivity

```bash
docker network inspect two-tier
```

Both Flask and MySQL should be connected to the same network.

### Test MySQL hostname from Flask

Enter the Flask container:

```bash
docker exec -it flask-app bash
```

Then:

```bash
getent hosts mysql-db
```

## Security Note

This project is intended for Docker and DevOps practice.

The example credentials:

```text
MYSQL_USER=Ahmad
MYSQL_PASSWORD=root
```

are for development/testing only.

For production deployments:

- Do not hard-code passwords.
- Use environment variables or Docker secrets.
- Do not expose MySQL port `3306` publicly.
- Use a production WSGI server such as Gunicorn.
- Use HTTPS.
- Validate and sanitize user input.
- Follow least-privilege access.

## Learning Objectives

This project demonstrates:

- Creating Docker images
- Writing a Dockerfile
- Running Docker containers
- Docker Compose
- Multi-container applications
- Docker bridge networks
- Container-to-container communication
- MySQL containerization
- Persistent Docker volumes
- Docker healthchecks
- `depends_on`
- Container logs
- Container debugging
- Deploying Docker applications on AWS EC2

## Author

**Ahmad Raza**

GitHub:

https://github.com/Ahmadraza9091
