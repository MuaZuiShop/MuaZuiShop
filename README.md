# Mua Zui Shop - Microservices Platform

[![Microservices](https://img.shields.io/badge/Architecture-Microservices-blue.svg)](#)
[![Java](https://img.shields.io/badge/Java-21-orange.svg)](#)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen.svg)](#)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED.svg)](#)

This repository serves as the central coordination point for the Mua Zui Shop microservices architecture. It utilizes Git submodules to manage independent service repositories and provides unified container orchestration configurations via Docker Compose to provision the complete system locally.

---

## System Architecture

The platform is constructed using independent, loosely coupled services communicating via Apache Kafka and leveraging Spring Cloud Netflix Eureka for service discovery. The architectural design is heavily inspired by established [Microservices Patterns](https://microservices.io/patterns/index.html).

### Submodules (Core Services)

* [**API Gateway**](https://github.com/MuaZuiShop/api-gateway)**:** Edge service for request routing, filtering, cross-cutting concerns, and unified API documentation (Port 8000).

* [**Eureka Server**](https://github.com/MuaZuiShop/eureka-server)**:** Service registry for dynamic discovery and load balancing (Port 8761).

* [**Auth Service**](https://github.com/MuaZuiShop/auth-service)**:** Identity and access management, handling JWT generation and validation (Port 8001).

* [**Customer Service**](https://github.com/MuaZuiShop/customer-service)**:** Domain service managing customer profiles and related operational logic (Port 8002).

* *Note: Shared domain models and utility logic are maintained in a private GitHub Package: `common-libraries`.*

### Infrastructure Dependencies

* **Relational Database:** PostgreSQL 17 (Alpine)

* **In-Memory Data Store:** Redis 7.2.4 (Alpine)

* **Event Streaming Platform:** Confluent Kafka 7.6.1 (KRaft mode)

* **Monitoring/UI:** Provectus Kafka-UI (Port 8080)

## Container & Network Infrastructure

All services are orchestrated using Docker Compose and reside within a unified custom bridge network (`mua-zui-shop-net`), allowing seamless inter-service communication using container names as hostnames.

### Service Registry & Ports

| Service Name | External Port | Internal Port | Protocol | Description |
| :--- | :--- | :--- | :--- | :--- |
| **api-gateway** | `8000` | `8000` | HTTP | Entry point for all client requests |
| **eureka-server** | `8761` | `8761` | HTTP | Service Discovery & Health Monitoring |
| **auth-service** | `8001` | `8001` | HTTP | Security, JWT & User Identity |
| **customer-service** | `8002` | `8002` | HTTP | Business logic for Customer domain |
| **postgres** | `5432` | `5432` | TCP | Primary relational database |
| **redis** | `6379` | `6379` | TCP | Caching and session management |
| **kafka** | `9092` | `9092` | TCP | Event streaming (KRaft mode) |
| **kafka-ui** | `8080` | `8080` | HTTP | Web UI for managing Kafka topics/clusters |

## Initialization Procedure

### 1. Clone the Repository

This project heavily relies on Git Submodules. It is mandatory to include the recursive flag during the initial clone to fetch the source code of all child services.

```bash
git clone --recursive [https://github.com/MuaZuiShop/MuaZuiShop.git](https://github.com/MuaZuiShop/MuaZuiShop.git)
cd MuaZuiShop
```

*(For existing clones missing the submodules, execute: `git submodule update --init --recursive`)*

### 2. Registry Authentication Configuration

The services depend on internal artifacts (`common-libraries`) hosted on GitHub Packages. Prior to initiating the Docker build process, the host environment must be configured with appropriate authentication credentials.

*Note: To request the required authentication credentials, please contact the administrative team at `muazuishopp@gmail.com`.*

Export your GitHub credentials into the environment variables:

**Linux/macOS:**

```bash
export GITHUB_ACTOR=your_github_username
export GITHUB_TOKEN=ghp_your_personal_access_token
```

**Windows (PowerShell):**

```powershell
$env:GITHUB_ACTOR="your_github_username"
$env:GITHUB_TOKEN="ghp_your_personal_access_token"
```

*Requirement: The Personal Access Token must possess the `read:packages` scope.*

### 3. Provisioning the Platform

A comprehensive `docker-compose.yml` is provided at the root directory to provision all infrastructure dependencies and compile the application services sequentially.

```bash
docker-compose up -d --build
```

## API Documentation

The platform utilizes OpenAPI 3 (Swagger) to provide comprehensive API documentation. Instead of accessing individual service documentations separately, the **API Gateway** acts as an aggregator.

Once the platform is running, you can access the unified Swagger UI containing endpoints for all registered services at:

👉 [**http://localhost:8000/swagger-ui.html**](http://localhost:8000/swagger-ui.html) *(assuming the API Gateway is running on its default port)*

Use the dropdown menu within the Swagger UI to switch between different service definitions (e.g., Auth Service, Customer Service).

## Code Quality & Security Verification

To maintain codebase integrity and prevent security vulnerabilities, the project enforces specific verification checks.

### Code Style (Checkstyle)

We utilize Checkstyle to ensure consistent coding conventions across all Java services. To verify your code locally before committing:

```bash
# Navigate to the specific service directory (e.g., customer-service)
cd customer-service
./mvnw checkstyle:check
```

### Secret Scanning (TruffleHog)

To prevent accidental commits of sensitive information (like credentials, API keys, or tokens), we recommend using TruffleHog. Ensure your local environment is clear of leaked secrets before pushing:

```bash
# Run TruffleHog scan on the entire repository and submodules
trufflehog git file://. --only-verified
```

*(Requires [TruffleHog](https://github.com/trufflesecurity/trufflehog) to be installed on your host machine)*

## Optimized Local Development Environment

When iterating on specific application services (e.g., `api-gateway`, `auth-service`, or `customer-service`), executing a full Docker context rebuild and redownloading Maven dependencies within the isolated container layer is highly inefficient.

To optimize the development lifecycle, an alternative configuration is provided: `utils-local/docker-compose.yml`. This configuration is designed to mount the host machine's local Maven repository (`~/.m2`) into the containers, drastically reducing dependency resolution time during subsequent builds.

**Execution Workflow:**

1. Provision only the core infrastructure components and the service registry using the primary compose file:

   ```bash
   docker-compose up -d postgres redis kafka kafka-ui eureka-server
   ```

2. Build and execute the targeted services using the localized configuration:

   ```bash
   # Deploy API Gateway and Auth Service using local Maven cache
   docker-compose up -d --build
   ```

## Submodule Maintenance

To synchronize the parent repository with the latest upstream commits from the child repositories, execute the following procedure:

```bash
# Fetch and merge the latest remote pointers for all registered submodules
git submodule update --remote --merge

# Stage and commit the updated submodule references
git add .
git commit -m "chore: synchronize submodule pointers to latest upstream"
git push
```

## References

* [Microservices Patterns](https://microservices.io/patterns/index.html) - Foundational patterns applied in this architecture.