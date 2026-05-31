# Maven Dependencies

## Dependency Scope

### Scopes Explained

**1. compile** (default)
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.0</version>
    <scope>compile</scope>
</dependency>
```
- Available in all classpaths
- Transitive

**2. provided**
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```
- Provided by JDK or container
- Not included in package

**3. runtime**
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
    <scope>runtime</scope>
</dependency>
```
- Not needed for compilation
- Required for execution

**4. test**
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.0</version>
    <scope>test</scope>
</dependency>
```
- Only for testing
- Not transitive

**5. system**
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>custom-lib</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/custom-lib.jar</systemPath>
</dependency>
```
- Must provide explicit path
- Not recommended

**6. import** (only in dependencyManagement)
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.7.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## Transitive Dependencies

### What are Transitive Dependencies?

When you add a dependency, Maven automatically includes its dependencies.

**Example:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.0</version>
</dependency>
```

This automatically includes:
- spring-web
- spring-webmvc
- jackson-databind
- tomcat-embed-core
- And their dependencies...

### View Dependency Tree

```bash
# Show all dependencies
mvn dependency:tree

# Show specific artifact
mvn dependency:tree -Dincludes=org.springframework:spring-core
```

### Excluding Transitive Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## Version Conflicts & Resolution

### Dependency Mediation

Maven uses "nearest definition" strategy:

```
Project
├── A 1.0 → C 1.0
└── B 1.0 → C 2.0

Result: C 1.0 wins (shorter path from A)
```

### Resolving Conflicts

**1. Explicit Declaration**
```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>library-c</artifactId>
        <version>2.0</version>
    </dependency>
</dependencies>
```

**2. Dependency Management**
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>library-c</artifactId>
            <version>2.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**3. Exclusions**
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>library-a</artifactId>
    <version>1.0</version>
    <exclusions>
        <exclusion>
            <groupId>com.example</groupId>
            <artifactId>library-c</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### Analyze Dependencies

```bash
# Show dependency tree
mvn dependency:tree

# Analyze conflicts
mvn dependency:analyze

# Show effective POM
mvn help:effective-pom
```

## Using Dependency Management

### Parent POM with Dependency Management

**parent-pom.xml:**
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <properties>
        <spring.version>5.3.0</spring.version>
        <junit.version>5.8.0</junit.version>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

**child-pom.xml:**
```xml
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>child-module</artifactId>
    
    <dependencies>
        <!-- No version needed - inherited from parent -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
        </dependency>
    </dependencies>
</project>
```

### BOM (Bill of Materials)

```xml
<dependencyManagement>
    <dependencies>
        <!-- Import Spring Boot BOM -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.7.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- No version needed -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

## Best Practices

✅ Use dependency management for version control
✅ Minimize direct dependencies
✅ Exclude unnecessary transitive dependencies
✅ Use properties for version numbers
✅ Regularly update dependencies
✅ Use `mvn dependency:analyze` to find unused dependencies
