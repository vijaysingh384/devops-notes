# Maven Build Lifecycle

## Build Lifecycle Phases

### Key Phases

**1. validate** - Validate project correctness
```bash
mvn validate
```

**2. compile** - Compile source code
```bash
mvn compile
# Compiles src/main/java → target/classes
```

**3. test** - Run unit tests
```bash
mvn test
# Runs tests from src/test/java
# Uses Surefire plugin
```

**4. package** - Package compiled code
```bash
mvn package
# Creates JAR/WAR in target/
```

**5. verify** - Run integration tests
```bash
mvn verify
# Runs integration tests
# Uses Failsafe plugin
```

**6. install** - Install to local repository
```bash
mvn install
# Installs to ~/.m2/repository
```

**7. deploy** - Deploy to remote repository
```bash
mvn deploy
# Deploys to Nexus/Artifactory
```

### Common Commands

```bash
# Clean and compile
mvn clean compile

# Clean, test, and package
mvn clean package

# Skip tests
mvn clean install -DskipTests

# Run specific phase
mvn test

# Clean build output
mvn clean
```

### Phase Execution

When you run a phase, all previous phases execute:
```bash
mvn package
# Executes: validate → compile → test → package
```

## Clean Lifecycle

```bash
# Remove target directory
mvn clean

# Clean and build
mvn clean install
```

## Site Lifecycle

```bash
# Generate project documentation
mvn site

# Deploy site
mvn site-deploy
```
