# Maven Build Lifecycle and Dependencies

## Table of Contents
1. [Build Lifecycle Phases](#build-lifecycle-phases)
2. [Parent POM](#parent-pom)
3. [Dependency Scope](#dependency-scope)
4. [Transitive Dependencies](#transitive-dependencies)
5. [Version Conflicts & Resolution](#version-conflicts--resolution)
6. [Using Dependency Management](#using-dependency-management)

---

## Build Lifecycle Phases

### Maven Build Lifecycles

Maven has **three built-in build lifecycles:**

1. **default**: Main build lifecycle (compile, test, package, deploy)
2. **clean**: Cleans the project
3. **site**: Generates project documentation

### Default Lifecycle Phases

**Complete Phase Sequence:**
```
validate
  ↓
initialize
  ↓
generate-sources
  ↓
process-sources
  ↓
generate-resources
  ↓
process-resources
  ↓
compile
  ↓
process-classes
  ↓
generate-test-sources
  ↓
process-test-sources
  ↓
generate-test-resources
  ↓
process-test-resources
  ↓
test-compile
  ↓
process-test-classes
  ↓
test
  ↓
prepare-package
  ↓
package
  ↓
pre-integration-test
  ↓
integration-test
  ↓
post-integration-test
  ↓
verify
  ↓
install
  ↓
deploy
```


### Key Phases Explained

#### 1. validate
**Purpose**: Validate the project is correct and all necessary information is available

**What it does:**
- Checks if POM is valid
- Verifies required properties are set
- Validates project structure

**Command:**
```bash
mvn validate
```

**Example Output:**
```
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------------< com.example:my-app >-------------------
[INFO] Building my-app 1.0.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

#### 2. compile
**Purpose**: Compile the source code of the project

**What it does:**
- Compiles Java files from `src/main/java`
- Outputs to `target/classes`
- Processes resources from `src/main/resources`

**Command:**
```bash
mvn compile
```

**Example:**
```
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ my-app ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 10 source files to /project/target/classes
```

