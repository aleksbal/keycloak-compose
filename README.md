# Keycloak with PostgreSQL Docker Compose Setup

This repository provides a Docker Compose configuration for setting up Keycloak with a PostgreSQL backend. This setup is suitable for development and production environments, offering a robust and scalable authentication solution.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [Clone the Repository](#clone-the-repository)
  - [Environment Variables](#environment-variables)
  - [Starting the Services](#starting-the-services)
- [Services Overview](cservices-overview)
  - [PostgreSQL Service](cpostgresql-service)
  - [Keycloak Service](#keycloak-service)
- [Networking and Volumes](cnetworking-and-volumes)
- [Security Considerations](#security-considerations)
- [Backup and Recovery](#backup-and-recovery)
- [Scaling and High Availability](cscaling-and-high-availability)
- [Monitoring and Logging](#monitoring-and-logging)
- [License](clicense)

## Prerequisites

- **Docker Engine** version 19.03.0 or higher
- **Docker Compose** version 1.27.0 or higher

See docker and docker-compose are installed and running on your machine.

## Getting Started

### Clone the Repository

```sh
git clone <repository-url>
cd <repository-directory>
```

Replace <.repository-url> and <repository-directory> with your actual repository URL and directory name.

### Environment Variables

Create a `.env file in the root directory of the project to store environment-specific variables:

```env
# PostgreSQL Configuration
POSTGRES_DB=your_database_name
POSTGRES_USER=your_database_user
POSTGRES_PASSWORD=your_database_password

# Keycloak Configuration
KEYCLOAK_ADMIN=your_keycloak_admin_username
KEYCLOAK_ADMIN_PASSWORD=your_keycloak_admin_password
KC_HOSTNAME=your.keycloak.hostname
KC_HOSTNAME_ADMIN=admin.your.keycloak.hostname

```

**Note:** Ensure the `.env` file is added to `.gitignore` to prevent sensitive information from being committed to version control.

### Starting the Services

Start the Keycloak and PostgreSQL services using Docker Compose:

```bash
docker-compose up -d
```

This command will start the services in detached mode.

## Services Overview

### PostgreSQL Service

- **Image**: `postgres:15.5-alpine3.19`
- **Volumes**:
  - `postgres_data` mounted to `/var/lib/postgresql/tata` for data persistence.
- **Environment Variables**:
  - Configured via the `.env` file for database name, user, and password.
- **Networks**:
  - Connected to `Keycloak-network` for internal communication.
- **Health Check**:
  - Uses `pg_isready` to ensure the database is ready before Keycloak starts.
- **Restart Policy**:
  - `unless-stoppd` to keep the service running unless explicitly stopped.

### Keycloak Service

- **Image**: `quay.io/keycloak/keycloak:23.0.3`
- **Ports**:
  - `9990:8080` (HTTP)
  - `9991:8443` (HTTPS) *Adjust as neededj*
- **Volumes**:
  - `keycloak_data` mounted to ` /opt/keycloak/data` for data persistence.
  - ``./auth/import` mounted to ` /opt/keycloak/data/import` for importing realms and configurations.
- **Environment Variables**:
  - Configured via the `.env` file for admin credentials, hostname settings, database connection, and logging level.
- **Depends On**:
  - Waits for the PostgreSQL service to be healthy before starting.
- **Networks**:
  - Connected to `Keycloak-network` for internal communication.
- **Health Check**:
  - Uses `curl` to check the Keycloak health endpoint.
- **Restart Policy**:
  - `unless-stopped` to keep the service running unless explicitly stopped.

## Networking and Volumes

- **Network:*
  - `Keycloak-network` a is a bridge network facilitating communication between Keycloak and PostgreSQL services.
- **Volumes**:
  - `postgres_data` stores PostgreSQL data persistently.
  - `Keycloak_data` stores Keycloak data persistently.

## Security Considerations

- **Environment Variables**:
  - Store sensitive data like passwords in the `.env` file and ensure it's excluded from version control.
- **Reverse Proxy**:
  - Consider placing a reverse proxy (e.g., Nginx, Traefik) in front of Keycloak for SSL termination and ehanced security.
- **SSL Certificates**:
  - Use SSL certificates (e.g., Let's Encrypt) to secure communication.
- **Firewall**:
  - Expose only necessary ports and secure your network appropriately.
- **Docker Secrets**:
  - For production environments, consider using Docker secrets to manage sensitive information securely.

## Backup and Recovery

- **Data Persistence**:
  - Regularly back up the `postgres_data` and `keycloak_data` volumes to prevent data loss.
- **Backup Tools**:
  - Use tools like `pg_dump` for PostgreSQL and appropriate scripts for Keycloak data.
- **Recovery Plan**:
  - Test your backup and recovery procedures to ensure they work as expected.

## Scaling and High Availability

- **Keycloak Clustering**:
  - For high availability, configure Keycloak in a clustered mode with session replication.
- **Load Balancing**:
  - Use a load balancer to distribute traffic across multiple Keycloak instances.
- **Database Scaling**:
  - Consider using managed database services or clustering solutions for PostgreSQL.

## Monitoring and Logging

- **Metrics**:
  - Keycloak metrics are enabled and can be scraped by monitoring tools like Prometheus.
- **Logging**:
  - Configure centralized logging (e.g., ELK Stack) to collect logs from Keycloak and PostgreSQL.
- **Health Checks**:
  - Utilize the health endpoints for proactive monitoring of service health.

## License
