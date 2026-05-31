# Maven and Docker Integration

## dockerfile-maven-plugin

Plugin to build and push Docker images during Maven build.

### Basic Configuration

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.13</version>
    <configuration>
        <repository>${docker.image.prefix}/${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

## Dockerizing Maven Applications

### Multi-Stage Dockerfile

```dockerfile
# Build stage
FROM maven:3.8-openjdk-11 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Pushing to Registries

### Docker Hub
```bash
docker login
mvn clean package dockerfile:push
```

### Private Registry
```xml
<properties>
    <docker.registry>registry.company.com</docker.registry>
</properties>
```

### GitHub Container Registry
```bash
echo $TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

## Best Practices

✅ Use multi-stage builds
✅ Cache Maven dependencies
✅ Use .dockerignore
✅ Tag with version numbers
✅ Use slim base images
