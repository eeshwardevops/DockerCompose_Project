# VProfile Docker Compose Project

A multi-tier Java web application (VProfile) containerized using Docker Compose. This project demonstrates containerization and orchestration of a complete web stack with database, caching, messaging, and search capabilities.

## 📋 Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Technologies](#technologies)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Services](#services)
- [Getting Started](#getting-started)
- [Database Setup](#database-setup)
- [Docker Images](#docker-images)
- [Configuration](#configuration)
- [Running the Application](#running-the-application)
- [Accessing the Application](#accessing-the-application)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Overview

VProfile is a web application built with Spring Framework that demonstrates the integration of multiple technologies in a containerized environment. The application is fully orchestrated using Docker Compose, allowing for easy deployment and scaling.

## Prerequisites

- **JDK 11** or higher (Project uses Java 17)
- **Maven 3** or higher (v3.9.9 recommended)
- **Docker** and **Docker Compose** (latest versions recommended)
- **MySQL 8** (included in Docker setup)
- At least 4GB of available RAM
- 10GB of disk space

## Technologies

### Backend & Framework
- **Spring Framework** (v6.0.11)
- **Spring Boot** (v3.1.3)
- **Spring MVC** - Web framework
- **Spring Security** (v6.1.2) - Authentication & Authorization
- **Spring Data JPA** (v3.1.2) - Data access layer
- **Hibernate ORM** (v7.0.0) - Object-relational mapping

### Presentation & View
- **JSP** (Jakarta Servlet JSP)
- **JSTL** - Tag libraries
- **Jakarta Servlet** (v6.1.0)

### Data & Persistence
- **MySQL 8** - Primary database
- **Hibernate Validator** (v6.2.0)

### Caching & Messaging
- **Memcached** - In-memory caching
- **RabbitMQ** - Message broker
- **Spring AMQP** (v3.1.6) - AMQP messaging

### Search
- **Elasticsearch** (v7.10.2) - Full-text search

### Application Server & Reverse Proxy
- **Apache Tomcat 10** with JDK 21
- **Nginx** - Reverse proxy & load balancer

### Build & Dependency Management
- **Maven** - Build automation
- **Apache Commons** - Utilities (FileUpload, IO, DBCP)
- **Jackson** (v2.13.0) - JSON processing
- **Log4j2** (v2.23.1) - Logging

### Testing
- **JUnit 4 & 5**
- **Mockito** (v5.5.0)
- **Spring Boot Test** - Integration testing
- **Hamcrest** - Matchers

## Architecture

```
┌─────────────────────────────────────────────┐
│         Nginx (Reverse Proxy)               │
│              Port: 80                       │
└────────────────────┬────────────────────────┘
                     │
┌────────────────────▼────────────────────────┐
│      Spring MVC Application (Tomcat)        │
│              Port: 8080                     │
└────────┬─────────────┬──────────┬───────────┘
         │             │          │
    ┌────▼──┐    ┌─────▼──┐  ┌──▼────┐
    │ MySQL │    │Memcache│  │RabbitMQ│
    │Port:3306   │Port:11211│ │Port:15672│
    └────────┘    └────────┘  └────────┘
         │
    ┌────▼──────┐
    │Elasticsearch
    │Port: 9200
    └───────────┘
```

## Project Structure

```
.
├── README.md                          # This file
├── compose.yaml                       # Docker Compose configuration
├── pom.xml                            # Maven configuration
├── Docker-files/                      # Docker build contexts
│   ├── db/                            # MySQL database Dockerfile
│   ├── app/                           # Application Dockerfile
│   └── web/                           # Nginx Dockerfile
├── src/                               # Application source code
│   ├── main/
│   │   ├── java/                      # Java source files
│   │   ├── webapp/                    # JSP files and web resources
│   │   └── resources/
│   │       └── db_backup.sql          # Database schema and initial data
│   └── test/                          # Test files
└── Screenshot *.png                   # Project screenshots
```

## Services

The Docker Compose setup defines the following services:

### 1. **vprodb** - MySQL Database
- **Image**: Custom image (eshwar933/vprofiledb)
- **Port**: 3306
- **Root Password**: vprodbpass
- **Volume**: vprodbdata (persists database)
- **Purpose**: Stores all application data

### 2. **vprocache01** - Memcached
- **Image**: memcached:latest
- **Port**: 11211
- **Purpose**: In-memory caching for performance optimization

### 3. **vpromq01** - RabbitMQ
- **Image**: rabbitmq:latest
- **Port**: 15672 (Management UI)
- **Credentials**: guest / guest
- **Purpose**: Message broker for asynchronous processing

### 4. **vproapp** - Spring Application
- **Image**: Custom image (eshwar933/vprofileapp)
- **Build Context**: ./Docker-files/app
- **Port**: 8080
- **Volume**: vproappdata (application deployment)
- **Purpose**: Main Java web application running on Tomcat

### 5. **vproweb** - Nginx Reverse Proxy
- **Image**: nginx:latest
- **Port**: 80
- **Build Context**: ./Docker-files/web
- **Purpose**: Routes traffic to the application

## Getting Started

### 1. Clone the Repository
```bash
git clone https://github.com/eeshwardevops/DockerCompose_Project.git
cd DockerCompose-Project
```

### 2. Verify Docker Installation
```bash
docker --version
docker-compose --version
```

### 3. Build Custom Docker Images
```bash
docker-compose build
```

### 4. Start Services
```bash
docker-compose up -d
```

### 5. Verify Services are Running
```bash
docker-compose ps
```

## Database Setup

### Import Database Schema

The project includes a MySQL dump file at `src/main/resources/db_backup.sql` containing the database schema and initial data.

#### Option 1: Automatic Import (via Docker)
The database is automatically initialized when the `vprodb` container starts with the dump file.

#### Option 2: Manual Import
```bash
# Access the MySQL container
docker exec -it vprodb mysql -u root -pvprodbpass

# Inside MySQL:
mysql> source /src/main/resources/db_backup.sql;
mysql> exit;
```

#### Option 3: From Host Machine
```bash
# Ensure vprodb service is running
docker-compose up -d vprodb

# Wait a few seconds for MySQL to start, then:
docker exec -i vprodb mysql -u root -pvprodbpass < src/main/resources/db_backup.sql
```

### Database Credentials
- **Host**: vprodb
- **Port**: 3306
- **User**: root
- **Password**: vprodbpass
- **Database**: accounts

## Docker Images

The following Docker images are used:

| Service | Image | Version | Purpose |
|---------|-------|---------|---------|
| Database | eshwar933/vprofiledb | Custom | MySQL database |
| Cache | memcached | latest | In-memory caching |
| Messaging | rabbitmq | latest | Message broker |
| Application | eshwar933/vprofileapp | Custom | Spring MVC application |
| Web Server | nginx | latest | Reverse proxy |
| Build Base | maven | 3.9.9-eclipse-temurin-21 | Build environment |
| App Base | tomcat | 10-jdk21 | Application server |

## Configuration

### Environment Variables

The following environment variables are configured in `compose.yaml`:

```yaml
MYSQL_ROOT_PASSWORD: vprodbpass
RABBITMQ_DEFAULT_USER: guest
RABBITMQ_DEFAULT_PASS: guest
```

### Application Properties

Update application properties in the Spring application context for:
- Database connection pooling (Commons DBCP)
- Memcached server address
- RabbitMQ broker settings
- Elasticsearch connection
- Logging configuration (Log4j2)

### Volumes

Two named volumes persist data across container restarts:
- **vprodbdata**: MySQL database files
- **vproappdata**: Tomcat application deployments

## Running the Application

### Start All Services
```bash
docker-compose up -d
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f vproapp
```

### Stop Services
```bash
# Stop all services
docker-compose stop

# Stop and remove containers
docker-compose down

# Remove volumes as well (careful!)
docker-compose down -v
```

### Rebuild Services
```bash
# Rebuild all custom images
docker-compose build --no-cache

# Rebuild and restart
docker-compose up -d --build
```

## Accessing the Application

Once all services are running:

### Web Application
- **URL**: http://localhost
- **Port**: 80 (Nginx reverse proxy)

### Application Direct
- **URL**: http://localhost:8080
- **Note**: Use this if Nginx is not configured

### RabbitMQ Management Console
- **URL**: http://localhost:15672
- **Credentials**: guest / guest

### MySQL Database
- **Host**: localhost:3306
- **User**: root
- **Password**: vprodbpass
- **Connection Tools**: MySQL Workbench, DBeaver, or CLI

### Memcached
- **Host**: localhost:11211
- **Tool**: telnet or specialized clients

## Troubleshooting

### Services Won't Start
```bash
# Check logs for errors
docker-compose logs

# Verify Docker daemon is running
docker info

# Check port availability
netstat -an | grep LISTEN
```

### Database Connection Issues
```bash
# Test MySQL connectivity
docker exec -it vprodb mysql -u root -pvprodbpass -e "SELECT 1"

# Check database imports
docker exec -it vprodb mysql -u root -pvprodbpass accounts -e "SHOW TABLES;"
```

### Application Won't Deploy
```bash
# Check Tomcat logs
docker exec -it vproapp tail -f /usr/local/tomcat/logs/catalina.out

# Verify application war file
docker exec -it vproapp ls -la /usr/local/tomcat/webapps/
```

### Port Conflicts
```bash
# Find process using a port (e.g., 80)
lsof -i :80

# Change ports in compose.yaml if needed
# Example: Change "80:80" to "8000:80"
```

### Rebuild After Code Changes
```bash
# Rebuild the application image
docker-compose build --no-cache vproapp

# Restart the service
docker-compose up -d vproapp
```

## Performance Optimization Tips

1. **Memcached**: Configure cache keys and TTL appropriately
2. **Database**: Add indexes on frequently queried columns
3. **RabbitMQ**: Tune prefetch count and queue settings
4. **Elasticsearch**: Configure shards and replicas based on data volume
5. **Tomcat**: Adjust thread pools in `server.xml`

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## Project Statistics

- **Language Composition**:
  - CSS: 41.2%
  - Java: 28.2%
  - SCSS: 15.2%
  - Less: 15%
  - Other: 0.4%

## License

This project is open source and available under the MIT License.

## Support & Contact

For issues, questions, or suggestions:
- Create an issue on [GitHub Issues](https://github.com/gajulaeshwar9/DockerCompose-Project/issues)
- Contact the repository owner

## Acknowledgments

- Built with Spring Framework ecosystem
- Docker and Docker Compose for containerization
- Open source community contributions

---

**Last Updated**: May 22, 2026
**Repository**: [gajulaeshwar9/DockerCompose-Project](https://github.com/gajulaeshwar9/DockerCompose-Project)
