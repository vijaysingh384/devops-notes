# Maven Plugins & Execution

## Maven Plugins Overview

Plugins provide goals (tasks) that execute during build lifecycle phases.

### Plugin Syntax

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <!-- Plugin configuration -->
            </configuration>
        </plugin>
    </plugins>
</build>
```

## Compiler Plugin

Compiles Java source code.

**Configuration:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>11</source>
        <target>11</target>
        <encoding>UTF-8</encoding>
        <compilerArgs>
            <arg>-parameters</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

**Usage:**
```bash
mvn compiler:compile
mvn compiler:testCompile
```

## Surefire Plugin (Unit Testing)

Runs unit tests during the test phase.

**Configuration:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <!-- Include/exclude tests -->
        <includes>
            <include>**/*Test.java</include>
            <include>**/*Tests.java</include>
        </includes>
        <excludes>
            <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
        
        <!-- Parallel execution -->
        <parallel>methods</parallel>
        <threadCount>4</threadCount>
        
        <!-- System properties -->
        <systemPropertyVariables>
            <propertyName>value</propertyName>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

**Usage:**
```bash
# Run all tests
mvn test

# Run specific test
mvn test -Dtest=UserServiceTest

# Run specific method
mvn test -Dtest=UserServiceTest#testCreateUser

# Skip tests
mvn install -DskipTests

# Skip test compilation and execution
mvn install -Dmaven.test.skip=true
```

## Shade Plugin (Uber JAR)

Creates an executable JAR with all dependencies.

**Configuration:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.example.Main</mainClass>
                    </transformer>
                </transformers>
                <finalName>${project.artifactId}-${project.version}-uber</finalName>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**Result:**
```bash
mvn package
# Creates: target/my-app-1.0.0-uber.jar

# Run uber JAR
java -jar target/my-app-1.0.0-uber.jar
```

## Maven Wrapper (mvnw)

Allows project to use specific Maven version without requiring Maven installation.

### Generate Wrapper

```bash
mvn wrapper:wrapper
```

**Generated files:**
```
.mvn/
└── wrapper/
    ├── maven-wrapper.jar
    └── maven-wrapper.properties
mvnw          # Unix script
mvnw.cmd      # Windows script
```

### Usage

```bash
# Unix/Mac
./mvnw clean install

# Windows
mvnw.cmd clean install
```

**Benefits:**
- No Maven installation required
- Consistent Maven version across team
- CI/CD friendly

## Other Useful Plugins

### JAR Plugin

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.example.Main</mainClass>
                <addClasspath>true</addClasspath>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

### WAR Plugin

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.1</version>
    <configuration>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </configuration>
</plugin>
```

### Failsafe Plugin (Integration Tests)

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>2.22.2</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Usage:**
```bash
mvn verify
```

### Exec Plugin

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <mainClass>com.example.Main</mainClass>
    </configuration>
</plugin>
```

**Usage:**
```bash
mvn exec:java
```

## Plugin Execution

### Binding to Phases

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
        <execution>
            <id>custom-task</id>
            <phase>package</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <target>
                    <echo>Building ${project.artifactId}</echo>
                </target>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Running Plugin Goals

```bash
# Run specific goal
mvn plugin-prefix:goal

# Examples
mvn compiler:compile
mvn surefire:test
mvn shade:shade
```

## Best Practices

✅ Always specify plugin versions
✅ Use Maven Wrapper for consistency
✅ Configure Surefire for parallel test execution
✅ Use Shade plugin for executable JARs
✅ Separate unit and integration tests
✅ Document custom plugin configurations
