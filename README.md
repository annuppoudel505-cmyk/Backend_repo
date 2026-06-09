# Dockerizing a Maven-Based Spring Boot Application Using JDK 25

## Objective

This project demonstrates how to containerize a Maven-based Spring Boot application using Docker. Two approaches are implemented:

1. Basic Dockerfile using a full JDK image.
2. Optimized Multi-Stage Dockerfile using Maven for build and a lightweight JRE image for runtime.

---

## Technologies Used

- Java 25
- Spring Boot
- Maven
- Docker

---

## Project Structure

```text
java-maven-docker-app/
│
├── src/
├── pom.xml
├── Dockerfile
├── Dockerfile.multi
├── .dockerignore
└── README.md
```

---

## Requirement 1: Basic Dockerfile

Uses a standard JDK image for both build and runtime.

### Dockerfile

```dockerfile
FROM eclipse-temurin:25-jdk

WORKDIR /app

COPY . .

RUN ./mvnw clean package -DskipTests

CMD ["java", "-jar", "target/*.jar"]
```

### Build Image

```bash
docker build -t java-basic .
```

### Run Container

```bash
docker run -p 8080:8080 java-basic
```

---

## Requirement 2: Optimized Multi-Stage Dockerfile

Uses:

- Maven image for building
- Lightweight JRE image for runtime

### Dockerfile.multi

```dockerfile
# Build Stage
FROM maven:3.9.9-eclipse-temurin-25 AS builder

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

# Runtime Stage
FROM eclipse-temurin:25-jre-alpine

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

ENV SPRING_PROFILES_ACTIVE=prod

EXPOSE 8080

HEALTHCHECK CMD wget --spider -q http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Build Image

```bash
docker build -t java-optimized -f Dockerfile.multi .
```

### Run Container

```bash
docker run -p 8080:8080 java-optimized
```

---

## Verify Application

Open browser:

```text
http://localhost:8080
```

Expected Output:

```text
Hello from Spring Boot running in Docker!
```

---

## Image Size Comparison

Run:

```bash
docker images
```

Example Results:

| Image Type | Approximate Size |
|------------|------------------|
| Basic JDK Image | 500 MB – 800 MB |
| Multi-Stage Image | 80 MB – 150 MB |

---

## Why Multi-Stage Builds Are Better

Multi-stage builds separate the build environment from the runtime environment.

Benefits:

- Smaller final image
- Faster deployment
- Reduced attack surface
- No Maven or source code in production image

---

## Why JRE Images Are Smaller Than JDK Images

JDK contains:

- Java compiler
- Debugging tools
- Development utilities
- Runtime

JRE contains only:

- Java runtime libraries
- JVM

Since runtime images remove development tools, they are significantly smaller.

---

## Optional Optimizations Implemented

### Docker Ignore

```dockerignore
target
.git
.idea
.vscode
*.iml
*.log
README.md
```

Reduces build context size and improves build speed.

### Environment Variable

```dockerfile
ENV SPRING_PROFILES_ACTIVE=prod
```

Used to activate the production profile.

### Health Check

```dockerfile
HEALTHCHECK CMD wget --spider -q http://localhost:8080/actuator/health || exit 1
```

Allows Docker to monitor container health.

---

## Commands Summary

### Build Basic Image

```bash
docker build -t java-basic .
```

### Run Basic Container

```bash
docker run -p 8080:8080 java-basic
```

### Build Optimized Image

```bash
docker build -t java-optimized -f Dockerfile.multi .
```

### Run Optimized Container

```bash
docker run -p 8080:8080 java-optimized
```

### View Images

```bash
docker images
```

---

## Acceptance Criteria

- Application runs successfully in Docker.
- JDK 25 used correctly.
- Multi-stage build implemented.
- Image size comparison completed.
- Optimizations added.