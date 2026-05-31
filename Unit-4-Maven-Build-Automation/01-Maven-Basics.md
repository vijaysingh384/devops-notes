# Maven Build Automation - Basics

## Table of Contents
1. [Why Build Tools Exist](#why-build-tools-exist)
2. [Problems Solved by Automated Builds](#problems-solved-by-automated-builds)
3. [Project Object Model (POM)](#project-object-model-pom)
4. [Directory Structure](#directory-structure)

---

## Why Build Tools Exist

### The Manual Build Problem

**Without Build Tools:**
```bash
# Compile all Java files manually
javac -d bin src/com/example/Main.java
javac -d bin src/com/example/Utils.java
javac -d bin src/com/example/Service.java
# ... hundreds more files

# Download dependencies manually
# Visit website, download JAR, place in lib/

# Run tests manually
java -cp bin:lib/* org.junit.runner.JUnitCore com.example.TestMain

# Package manually
jar cvf myapp.jar -C bin .

# Deploy manually
scp myapp.jar user@server:/opt/app/
```

**Problems:**
- ❌ Time-consuming and error-prone
- ❌ Difficult to reproduce
- ❌ Hard to share with team
- ❌ No dependency management
- ❌ Manual version tracking
- ❌ Inconsistent builds across environments

### What Build Tools Provide

**With Maven:**
```bash
mvn clean install
```

**Benefits:**
- ✅ Automated compilation
- ✅ Dependency management
- ✅ Automated testing
- ✅ Standardized project structure
- ✅ Reproducible builds
- ✅ Plugin ecosystem
- ✅ Multi-module support

### Evolution of Build Tools

**Timeline:**
```
Make (1976)
  ↓
Ant (2000)
  ↓
Maven (2004)  ← We are here
  ↓
Gradle (2012)
```

**Comparison:**

| Tool | Configuration | Dependency Mgmt | Convention |
|------|--------------|-----------------|------------|
| **Make** | Makefile | Manual | No |
| **Ant** | XML (build.xml) | Manual (Ivy) | No |
| **Maven** | XML (pom.xml) | Built-in | Yes |
| **Gradle** | Groovy/Kotlin | Built-in | Yes |

### Why Maven?

**Key Features:**
1. **Convention over Configuration**: Standard project structure
2. **Declarative**: Describe what, not how
3. **Dependency Management**: Automatic download and resolution
4. **Plugin Architecture**: Extensible functionality
5. **Repository System**: Central and local repositories
6. **Build Lifecycle**: Standardized build phases
7. **Multi-Module Support**: Manage complex projects

**Maven Philosophy:**
- Projects should follow standard structure
- Build process should be uniform
- Dependencies should be managed centrally
- Builds should be reproducible

---

## Problems Solved by Automated Builds

### 1. Dependency Management

**Problem:**
```
Project needs:
- Spring Framework 5.3.0
- Hibernate 5.6.0
- JUnit 5.8.0
- Each has its own dependencies (transitive)
```

**Manual Approach:**
- Download each JAR manually
- Find and download all transitive dependencies
- Manage version conflicts
- Update classpath

**Maven Solution:**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.0</version>
    </dependency>
</dependencies>
```

Maven automatically:
- Downloads spring-core
- Downloads all transitive dependencies
- Resolves version conflicts
- Updates local repository

### 2. Build Reproducibility

**Problem:**
- "Works on my machine"
- Different environments produce different results
- Difficult to debug build issues

**Maven Solution:**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Benefits:**
- Same build process everywhere
- Consistent Java version
- Reproducible artifacts
- CI/CD friendly

### 3. Project Standardization

**Problem:**
- Every project has different structure
- New developers need time to understand layout
- Tools need custom configuration

**Maven Solution:**
```
Standard Maven Project Structure
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
├── target/
└── pom.xml
```

**Benefits:**
- Instant familiarity
- IDE support
- Tool integration
- Team efficiency

### 4. Build Automation

**Problem:**
```bash
# Manual build steps
javac ...
run tests
generate docs
package JAR
create WAR
deploy
```

**Maven Solution:**
```bash
mvn clean install
```

**Automated Steps:**
1. Clean previous builds
2. Validate project
3. Compile source code
4. Run unit tests
5. Package into JAR/WAR
6. Run integration tests
7. Install to local repository

### 5. Version Management

**Problem:**
- Managing library versions
- Updating dependencies
- Tracking what's used where

**Maven Solution:**
```xml
<properties>
    <spring.version>5.3.0</spring.version>
    <hibernate.version>5.6.0</hibernate.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
</dependencies>
```

**Benefits:**
- Centralized version management
- Easy updates
- Consistent versions across modules

### 6. Multi-Module Projects

**Problem:**
- Large projects with multiple components
- Shared dependencies
- Build order management

**Maven Solution:**
```xml
<!-- Parent POM -->
<modules>
    <module>common</module>
    <module>service</module>
    <module>web</module>
</modules>
```

**Benefits:**
- Single command builds all modules
- Automatic dependency resolution
- Shared configuration
- Proper build order

### 7. Testing Integration

**Problem:**
- Running tests manually
- Skipping tests accidentally
- No test reports

**Maven Solution:**
```bash
mvn test  # Run all tests
mvn verify  # Run integration tests
```

**Benefits:**
- Automated test execution
- Test reports generation
- CI/CD integration
- Coverage reports

### 8. Artifact Management

**Problem:**
- Where to store built artifacts?
- How to share between teams?
- Version tracking

**Maven Solution:**
```xml
<distributionManagement>
    <repository>
        <id>releases</id>
        <url>http://nexus.company.com/releases</url>
    </repository>
</distributionManagement>
```

**Benefits:**
- Central artifact repository
- Version management
- Team collaboration
- Release management

---

## Project Object Model (POM)

### What is POM?

**Definition**: The Project Object Model (POM) is an XML file (pom.xml) that contains project information and configuration details used by Maven to build the project.

**Core Concept**: POM is the fundamental unit of work in Maven. It describes:
- What the project is
- What it depends on
- How to build it
- Where to deploy it

### Basic POM Structure

**Minimal POM:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project Coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    
</project>
```

### POM Coordinates (GAV)

**Three coordinates uniquely identify a project:**

**1. GroupId:**
- Organization or group identifier
- Usually reverse domain name
- Example: `com.example`, `org.springframework`

**2. ArtifactId:**
- Project name
- Should be lowercase with hyphens
- Example: `my-app`, `spring-core`

**3. Version:**
- Project version
- Semantic versioning recommended
- Example: `1.0.0`, `2.1.3-SNAPSHOT`

**Complete Coordinates:**
```xml
<groupId>com.example</groupId>
<artifactId>my-app</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>
```

### Complete POM Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project Coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-application</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <!-- Project Information -->
    <name>My Application</name>
    <description>A sample Maven project</description>
    <url>https://example.com/my-app</url>
    
    <!-- Properties -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <junit.version>5.8.0</junit.version>
    </properties>
    
    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <!-- Build Configuration -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
</project>
```

### POM Sections Explained

#### 1. Project Coordinates
```xml
<groupId>com.example</groupId>
<artifactId>my-app</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>  <!-- jar, war, pom, maven-plugin -->
```

#### 2. Properties
```xml
<properties>
    <!-- Encoding -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    
    <!-- Java Version -->
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    
    <!-- Dependency Versions -->
    <spring.version>5.3.0</spring.version>
    <junit.version>5.8.0</junit.version>
</properties>
```

#### 3. Dependencies
```xml
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
```

#### 4. Build Configuration
```xml
<build>
    <!-- Source Directory (default: src/main/java) -->
    <sourceDirectory>src/main/java</sourceDirectory>
    
    <!-- Output Directory (default: target) -->
    <outputDirectory>target/classes</outputDirectory>
    
    <!-- Plugins -->
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
        </plugin>
    </plugins>
</build>
```

#### 5. Repositories
```xml
<repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
    
    <repository>
        <id>spring-releases</id>
        <url>https://repo.spring.io/release</url>
    </repository>
</repositories>
```

### POM Inheritance

**Super POM:**
- All POMs inherit from Super POM
- Defines default values
- Located in Maven installation

**Effective POM:**
```bash
# View effective POM (merged with parent and super POM)
mvn help:effective-pom
```

### POM Best Practices

✅ **Use Properties for Versions**
```xml
<properties>
    <spring.version>5.3.0</spring.version>
</properties>

<dependency>
    <version>${spring.version}</version>
</dependency>
```

✅ **Specify Plugin Versions**
```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>  <!-- Always specify version -->
</plugin>
```

✅ **Use Dependency Management**
```xml
<dependencyManagement>
    <dependencies>
        <!-- Define versions here -->
    </dependencies>
</dependencyManagement>
```

✅ **Keep POM Clean and Organized**
- Group related dependencies
- Use comments for clarity
- Follow consistent formatting

---

## Directory Structure

### Standard Maven Directory Layout

**Complete Structure:**
```
my-app/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/              # Application source code
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── Main.java
│   │   ├── resources/         # Application resources
│   │   │   ├── application.properties
│   │   │   ├── log4j2.xml
│   │   │   └── static/
│   │   │       └── images/
│   │   ├── webapp/            # Web application files (for WAR)
│   │   │   ├── WEB-INF/
│   │   │   │   └── web.xml
│   │   │   └── index.html
│   │   └── filters/           # Resource filter files
│   │       └── filter.properties
│   ├── test/
│   │   ├── java/              # Test source code
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── MainTest.java
│   │   └── resources/         # Test resources
│   │       └── test.properties
│   ├── site/                  # Site documentation
│   │   └── site.xml
│   └── assembly/              # Assembly descriptors
│       └── distribution.xml
├── target/                    # Build output (generated)
│   ├── classes/               # Compiled classes
│   ├── test-classes/          # Compiled test classes
│   ├── my-app-1.0.0.jar      # Final artifact
│   └── surefire-reports/      # Test reports
├── .mvn/                      # Maven wrapper files
│   └── wrapper/
│       └── maven-wrapper.properties
├── mvnw                       # Maven wrapper script (Unix)
└── mvnw.cmd                   # Maven wrapper script (Windows)
```

### Directory Purposes

#### 1. Source Directories

**src/main/java/**
- Application Java source code
- Package structure mirrors groupId
```
src/main/java/
└── com/
    └── example/
        └── myapp/
            ├── Main.java
            ├── controller/
            ├── service/
            ├── repository/
            └── model/
```

**src/main/resources/**
- Configuration files
- Property files
- Static resources
```
src/main/resources/
├── application.properties
├── application.yml
├── log4j2.xml
├── db/
│   └── migration/
│       └── V1__init.sql
└── static/
    ├── css/
    ├── js/
    └── images/
```

**src/main/webapp/** (WAR projects only)
- Web application files
- JSP, HTML, CSS, JavaScript
```
src/main/webapp/
├── WEB-INF/
│   ├── web.xml
│   └── views/
│       └── home.jsp
├── index.html
├── css/
└── js/
```

#### 2. Test Directories

**src/test/java/**
- Unit test source code
- Integration test source code
```
src/test/java/
└── com/
    └── example/
        └── myapp/
            ├── MainTest.java
            ├── controller/
            │   └── UserControllerTest.java
            └── service/
                └── UserServiceTest.java
```

**src/test/resources/**
- Test configuration files
- Test data files
```
src/test/resources/
├── application-test.properties
├── test-data.sql
└── fixtures/
    └── sample-data.json
```

#### 3. Build Output Directory

**target/**
- Generated by Maven
- Should be in .gitignore
- Contains all build artifacts

```
target/
├── classes/                   # Compiled main classes
├── test-classes/              # Compiled test classes
├── generated-sources/         # Generated source code
├── generated-test-sources/    # Generated test sources
├── maven-status/              # Build status
├── surefire-reports/          # Unit test reports
├── failsafe-reports/          # Integration test reports
├── site/                      # Generated site
└── my-app-1.0.0.jar          # Final artifact
```

### Customizing Directory Structure

**Override Defaults in POM:**
```xml
<build>
    <!-- Source Directory -->
    <sourceDirectory>src/main/java</sourceDirectory>
    
    <!-- Test Source Directory -->
    <testSourceDirectory>src/test/java</testSourceDirectory>
    
    <!-- Resources -->
    <resources>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
    
    <!-- Test Resources -->
    <testResources>
        <testResource>
            <directory>src/test/resources</directory>
        </testResource>
    </testResources>
    
    <!-- Output Directory -->
    <outputDirectory>target/classes</outputDirectory>
    
    <!-- Test Output Directory -->
    <testOutputDirectory>target/test-classes</testOutputDirectory>
    
    <!-- Final Name -->
    <finalName>${project.artifactId}-${project.version}</finalName>
</build>
```

### Multi-Module Project Structure

```
parent-project/
├── pom.xml                    # Parent POM
├── module-common/
│   ├── pom.xml
│   └── src/
│       └── main/
│           └── java/
├── module-service/
│   ├── pom.xml
│   └── src/
│       └── main/
│           └── java/
└── module-web/
    ├── pom.xml
    └── src/
        └── main/
            ├── java/
            └── webapp/
```

**Parent POM:**
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>module-common</module>
        <module>module-service</module>
        <module>module-web</module>
    </modules>
</project>
```

### Best Practices

✅ **Follow Standard Structure**
- Don't customize unless necessary
- Tools expect standard layout
- Team familiarity

✅ **Keep Source and Resources Separate**
```
src/main/java/     # Code only
src/main/resources/ # Config only
```

✅ **Organize Packages Properly**
```
com.example.myapp/
├── controller/
├── service/
├── repository/
├── model/
├── dto/
├── config/
└── util/
```

✅ **Add target/ to .gitignore**
```bash
# .gitignore
target/
*.class
*.jar
*.war
```

✅ **Use Meaningful Package Names**
```
com.company.project.feature.layer
com.example.ecommerce.order.service
```

---

## Summary

**Maven Build Automation Basics:**

1. **Why Build Tools**: Automate compilation, testing, packaging, and deployment
2. **Problems Solved**: Dependency management, reproducibility, standardization
3. **POM**: Central configuration file defining project structure and dependencies
4. **Directory Structure**: Standardized layout for source, resources, and tests

**Key Takeaways:**
- Maven follows "Convention over Configuration"
- POM is the heart of Maven projects
- Standard directory structure enables tool integration
- Automated builds ensure consistency and reproducibility

**Next Steps:**
- Understanding build lifecycle phases
- Working with dependencies
- Configuring plugins
- Maven and Docker integration
