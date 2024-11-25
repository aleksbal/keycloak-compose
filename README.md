# Keycloak and PostgreSQL Docker Compose Setup

This repository contains a Docker Compose configuration for setting up **Keycloak**, an open-source identity and access management solution, along with a **PostgreSQL** database. This setup facilitates the deployment of Keycloak with a persistent PostgreSQL backend, enabling secure authentication and authorization services for your applications.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Overview](#overview)
- [Services](#services)
    - [PostgreSQL Service](#postgresql-service)
    - [Keycloak (Auth) Service](#keycloak-auth-service)
- [Networks](#networks)
- [Volumes](#volumes)
- [Environment Variables](#environment-variables)
- [Getting Started](#getting-started)
- [Security Considerations](#security-considerations)
- [Additional Best Practices](#additional-best-practices)
- [License](#license)

## Prerequisites

Before deploying the services, ensure that you have the following installed on your system:

- [Docker](https://www.docker.com/get-started) (version 20.10.0 or higher)
- [Docker Compose](https://docs.docker.com/compose/install/) (version 1.27.0 or higher)

## Overview

The Docker Compose configuration defines two primary services:

1. **PostgreSQL (`postgres`)**: Serves as the database backend for Keycloak, storing all necessary data.
2. **Keycloak (`auth`)**: Provides identity and access management functionalities.

These services are connected via a dedicated Docker network and use Docker volumes to persist data.

## Services

### PostgreSQL Service

```yaml
postgres:
  image: postgres:15.5-alpine3.19
  volumes:
    - postgres_data:/var/lib/postgresql/data
  environment:
    POSTGRES_DB: ${POSTGRES_DB}
    POSTGRES_USER: ${POSTGRES_USER}
    POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
  networks:
    - keycloak-network
  healthcheck:
    test: ["CMD", "pg_isready", "-U", "${POSTGRES_USER}"]
    interval: 10s
    timeout: 5s
    retries: 5
  restart: unless-stopped
```

Description:

Image: Uses the official PostgreSQL image version 15.5-alpine3.19, which is a lightweight Alpine-based build.
Volumes:
postgres_data:/var/lib/postgresql/data: Utilizes a named Docker volume postgres_data to ensure data persistence across container restarts.
Environment Variables:
POSTGRES_DB: Sets the default database, defined in the .env file.
POSTGRES_USER: Creates a user, defined in the .env file.
POSTGRES_PASSWORD: Sets the password for the user, defined in the .env file.
Networks:
Connects to the keycloak-network Docker network, allowing communication with other services on the same network.
Healthcheck:
Uses pg_isready to check if PostgreSQL is ready to accept connections.
Restart Policy:
unless-stopped: Ensures the service restarts automatically unless explicitly stopped.

Keycloak (Auth) Service

```yaml
auth:
  image: quay.io/keycloak/keycloak:23.0.3
  ports:
    - "9990:8080"
    - "9991:8443"
  environment:
    KEYCLOAK_ADMIN: ${KEYCLOAK_ADMIN}
    KEYCLOAK_ADMIN_PASSWORD: ${KEYCLOAK_ADMIN_PASSWORD}
    KC_HOSTNAME_ADMIN: ${KC_HOSTNAME_ADMIN}
    KC_PROXY: edge
    KC_HOSTNAME: ${KC_HOSTNAME}
    KC_METRICS_ENABLED: "true"
    KC_HEALTH_ENABLED: "true"
    KC_DB: postgres
    KC_DB_PASSWORD: ${POSTGRES_PASSWORD}
    KC_DB_SCHEMA: public
    KC_DB_USERNAME: ${POSTGRES_USER}
    KC_DB_URL_HOST: postgres
    KC_DB_URL_DATABASE: ${POSTGRES_DB}
    KC_LOG_LEVEL: INFO
  depends_on:
    postgres:
      condition: service_healthy
  command:
    - start
  volumes:
    - keycloak_data:/opt/keycloak/data
    - ./auth/import:/opt/keycloak/data/import
  networks:
    - keycloak-network
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
    interval: 30s
    timeout: 10s
    retries: 3
  restart: unless-stopped
```

Description:

Image: Utilizes Keycloak version 23.0.3 from Quay.io.
Ports:
"9990:8080": Maps Keycloak's HTTP port 8080 to host port 9990.
"9991:8443": Maps Keycloak's HTTPS port 8443 to host port 9991.
Environment Variables:
KEYCLOAK_ADMIN: Administrator username, defined in the .env file.
KEYCLOAK_ADMIN_PASSWORD: Administrator password, defined in the .env file.
KC_HOSTNAME_ADMIN: Hostname for the admin console, defined in the .env file.
KC_PROXY: Sets the proxy mode to edge, indicating that Keycloak is behind a reverse proxy that handles TLS termination.
KC_HOSTNAME: External hostname for Keycloak, defined in the .env file.
KC_METRICS_ENABLED: Enables metrics collection (true).
KC_HEALTH_ENABLED: Enables health checks (true).
KC_DB: Specifies the database type as postgres.
KC_DB_PASSWORD: Database password, matching the PostgreSQL service, defined in the .env file.
KC_DB_SCHEMA: Uses the public schema in PostgreSQL.
KC_DB_USERNAME: Database username, matching the PostgreSQL service, defined in the .env file.
KC_DB_URL_HOST: Database host, referring to the PostgreSQL service (postgres).
KC_DB_URL_DATABASE: Database name, matching the PostgreSQL service, defined in the .env file.
KC_LOG_LEVEL: Sets the logging level to INFO for standard logging.
Depends On:
postgres: Ensures that the PostgreSQL service starts and is healthy before Keycloak starts.
Command:
start: Initiates Keycloak in start mode.
Volumes:
keycloak_data:/opt/keycloak/data: Utilizes a named Docker volume keycloak_data to ensure data persistence.
./auth/import:/opt/keycloak/data/import: Mounts a local auth/import directory to Keycloak's import directory, allowing the import of predefined configurations or realms.
Networks:
Connects to the keycloak-network Docker network, facilitating communication with the PostgreSQL service.
Healthcheck:
Uses curl to verify if Keycloak's health endpoint is accessible.
Restart Policy:
unless-stopped: Ensures the service restarts automatically unless explicitly stopped.
Networks

```yaml
networks:
  keycloak-network:
    driver: bridge
 ```

Description:

Name: The network is named keycloak-network.
Driver: Uses the bridge network driver, which is suitable for standalone containers on a single host.
Purpose:

Facilitates secure and efficient communication between the postgres and auth services.
Isolates these services from other Docker networks, enhancing security.

Volumes
```yaml
volumes:
  postgres_data:
    driver: local
  keycloak_data:
    driver: local
```

Description:

postgres_data:
Driver: Utilizes the local volume driver, storing PostgreSQL data on the host machine.
keycloak_data:
Driver: Utilizes the local volume driver, storing Keycloak data on the host machine.
Purpose:

Ensures that both PostgreSQL and Keycloak data persist across container restarts and recreations.
Decouples data storage from the container lifecycle, preventing data loss.
Environment Variables
Environment variables are managed using a .env file to separate configuration from code, enhancing security and maintainability.

.env File
Create a .env file in the same directory as your docker-compose.yml with the following content:

env
Copy code
# PostgreSQL Configuration
POSTGRES_DB=keycloak
POSTGRES_USER=keycloak
POSTGRES_PASSWORD=your_secure_password_here

# Keycloak Configuration
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=your_secure_admin_password_here
KC_HOSTNAME_ADMIN=localhost
KC_HOSTNAME=some-machine.somewhere
Explanation of Variables:

PostgreSQL Service
POSTGRES_DB: Name of the default database (keycloak).
POSTGRES_USER: Username for the database (keycloak).
POSTGRES_PASSWORD: Password for the database user (your_secure_password_here).
Keycloak (Auth) Service
KEYCLOAK_ADMIN: Administrator username (admin).
KEYCLOAK_ADMIN_PASSWORD: Administrator password (your_secure_admin_password_here).
KC_HOSTNAME_ADMIN: Hostname for the admin console (localhost).
KC_HOSTNAME: External hostname for Keycloak (some-machine.somewhere).
Getting Started
Follow these steps to set up and run the services:

1. Clone the Repository
bash
Copy code
git clone https://github.com/your-username/keycloak-postgres-docker-compose.git
cd keycloak-postgres-docker-compose
2. Create Necessary Directories
Ensure that the directories for data persistence and configuration imports exist:

bash
Copy code
mkdir -p auth/import
auth/import: Contains Keycloak import files (e.g., realm configurations).
Note: The postgres_data and keycloak_data directories are managed as Docker named volumes and do not require manual creation.

3. Configure Environment Variables
Create a .env file as described in the Environment Variables section with secure credentials.

4. Configure Import Files (Optional)
If you have predefined Keycloak configurations or realms, place them in the auth/import directory. Keycloak will automatically import these configurations on startup.

5. Start the Services
Launch the services using Docker Compose:

bash
Copy code
docker-compose up -d
The -d flag runs the containers in detached mode.
6. Verify Service Status
Check the status of the services to ensure they are running and healthy:

bash
Copy code
docker-compose ps
7. Access Keycloak
Admin Console (HTTP): http://localhost:9990
Admin Console (HTTPS): https://localhost:9991
Log in using the administrator credentials defined in your .env file:

Username: As specified by KEYCLOAK_ADMIN (e.g., admin)
Password: As specified by KEYCLOAK_ADMIN_PASSWORD
8. Access PostgreSQL
PostgreSQL is not exposed to the host by default for security reasons. To connect to PostgreSQL from the host, you can:

Option 1: Temporarily add port mapping in the docker-compose.yml:

yaml
Copy code
postgres:
  ...
  ports:
    - "5432:5432"
Then, restart the services:

bash
Copy code
docker-compose up -d
Option 2: Use Docker's networking features or connect from another container within the keycloak-network.

Connection Details:

Host: localhost
Port: 5432 (if port mapping is enabled)
Database: As specified by POSTGRES_DB (e.g., keycloak)
Username: As specified by POSTGRES_USER (e.g., keycloak)
Password: As specified by POSTGRES_PASSWORD
Security Considerations
Passwords: Ensure that the .env file contains strong, unique passwords for both PostgreSQL and Keycloak admin accounts. Avoid using simple or hardcoded passwords.

Network Exposure: By default, PostgreSQL is not exposed to the host. Only expose necessary ports and restrict access using firewall rules or Docker's network configurations.

Data Persistence: Docker volumes (postgres_data and keycloak_data) ensure data persistence. Regularly back up these volumes to prevent data loss.

Health Checks: Health checks ensure that services are running correctly and dependencies are met before starting dependent services.

Restart Policies: Services are configured to restart automatically unless explicitly stopped, enhancing reliability and uptime.

Additional Best Practices
Logging and Monitoring:

Implement centralized logging and monitoring solutions to track the performance and health of your services.
Consider integrating with tools like Prometheus, Grafana, or the ELK stack.
Backup Strategies:

Regularly back up PostgreSQL data to prevent data loss.
Implement backup routines for Keycloak configurations and data.
Secure Communication:

Use SSL/TLS for securing communications between services and clients.
Consider using reverse proxies like Nginx or Traefik for managing SSL termination and routing.
Access Control:

Limit access to the Docker host and management interfaces.
Use Docker user namespaces and other isolation mechanisms to enhance security.
Documentation and Version Control:

Keep your docker-compose.yml, .env, and related configurations under version control (e.g., Git) for tracking changes and collaboration.
Maintain clear documentation for setup, configuration, and maintenance procedures.
Regular Updates:

Keep Docker images up to date to benefit from the latest security patches and features.
Regularly review and update your .env file and configurations as needed.
License
This project is licensed under the MIT License.
