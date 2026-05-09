# Microservice Development and Integration Guidelines

## 1. Repository and Naming Conventions

To maintain consistency across the microservices ecosystem, all new repositories and modules must adhere strictly to the following naming conventions:

* **Repository/Module Name:** Must follow the `<domain>-service` suffix pattern (e.g., `inventory-service`, `booking-service`, `notification-service`).

* **Maven Group ID:** `com.us.<author>`

* **Maven Artifact ID:** Must exactly match the repository/module name.

* **Base Package:** `com.us.<author>.<domain>` (e.g., `com.us.quy.inventory`).

## 2. Architectural Prerequisites

Before initializing a new service, developers must thoroughly review the architectural documentation to understand inter-service communication and network topology:

1. **Service Registry:** Read `eureka/README.md`. Every new microservice must be configured as a Eureka Client to participate in dynamic service discovery.

2. **API Routing:** Read `api-gateway/README.md`. External-facing endpoints of the new service must be explicitly routed and secured through the API Gateway configuration.

## 3. Maven Configuration (pom.xml) Standards

Every new microservice must standardize its build configuration, dependencies, and code quality enforcement mechanisms. Base your new `pom.xml` on the following structured guidelines.

### 3.1. Parent and Core Properties

Inherit from the designated Spring Boot Starter Parent and define the mandatory properties, including Java 21 and the centralized Checkstyle configuration paths.

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.5</version>
    <relativePath/>
</parent>

<properties>
    <java.version>21</java.version>
    <spring-cloud.version>2023.0.2</spring-cloud.version>
    <maven-checkstyle-plugin.version>3.4.0</maven-checkstyle-plugin.version>
    <checkstyle.version>10.17.0</checkstyle.version>
    <!-- Assumes a monorepo/multi-module structure where code quality configs are one level up -->
    <checkstyle.config.location>${project.basedir}/../checkstyle/checkstyle.xml</checkstyle.config.location>
    <checkstyle.check.skip>false</checkstyle.check.skip>
</properties>
```

### 3.2. Dependency Management

Declare the Spring Cloud dependencies BOM (Bill of Materials) to ensure version compatibility across all Spring Cloud components (like Eureka).

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 3.3. Mandatory Dependencies

Include the following core dependencies in every service to ensure consistency in web exposure, service discovery, and API documentation:

* **`spring-boot-starter-web`**: Required for REST API exposure and core web functionalities.
* **`spring-cloud-starter-netflix-eureka-client`**: Required for Service Discovery registration.
* **`springdoc-openapi-starter-webmvc-ui`** (Version `2.6.0`): Required for standardized Swagger/OpenAPI documentation.
* **`lombok`**: Required for boilerplate code reduction (marked as optional in `<optional>true</optional>`).

*Note: Include Domain-specific dependencies (e.g., `spring-boot-starter-data-jpa`, `postgresql`, `spring-boot-starter-security`) only if the microservice requires database access or internal security configurations.*

### 3.4. Build Plugins and Code Quality Enforcement

All microservices must enforce strict code quality and test coverage. The `<build><plugins>` section must contain the following components:

**A. JaCoCo Maven Plugin (Test Coverage)**
Enforces a minimum instruction coverage ratio of 70%. The build will fail if coverage drops below this threshold. Exclude non-business-logic packages (DTOs, Entities, Configurations) as follows:

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>report</goal>
            </goals>
            <configuration>
                <formats>
                    <format>HTML</format>
                    <format>XML</format>
                </formats>
            </configuration>
        </execution>
    </executions>
    <configuration>
        <excludes>
            <exclude>**/configs/**</exclude>
            ...
        </excludes>
        <rules>
            <rule>
                <element>BUNDLE</element>
                <limits>
                    <limit>
                        <counter>INSTRUCTION</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.70</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</plugin>
```

**B. Checkstyle Plugin**
Enforces coding standards. Ensure the path to `checkstyle.xml` and `suppressions.xml` resolves correctly relative to the new service's directory.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>${maven-checkstyle-plugin.version}</version>
    <configuration>
        <configLocation>${project.basedir}/../checkstyle/checkstyle.xml</configLocation>
        <suppressionsLocation>${project.basedir}/../checkstyle/suppressions.xml</suppressionsLocation>
        <failsOnError>true</failsOnError>
    </configuration>
</plugin>
```

**C. SpotBugs Plugin**
Performs static analysis to identify potential bugs. Threshold is set to `Medium`.

```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.6.2</version>
    <configuration>
        <threshold>Medium</threshold>
        <failOnError>true</failOnError>
        <excludeFilterFile>${project.basedir}/../spotbugs-exclude.xml</excludeFilterFile>
    </configuration>
</plugin>
```

## 4. Integration Checklist

Before raising a Pull Request for a newly initialized service, verify the following:

1. Does the service successfully compile and pass the 70% JaCoCo coverage gate (`mvn clean verify`)?

2. Does the service successfully register with the local Eureka instance upon startup?

3. Is the Swagger UI accessible at `http://localhost:<port>/swagger-ui.html`?

4. Have you added the routing configuration for this new service into the `api-gateway` module?
