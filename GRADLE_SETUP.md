# Gradle Build Setup Guide

This project has been fully converted from Maven to Gradle with Groovy DSL and includes comprehensive publishing configuration for Sonatype OSSRH (Maven Central).

## Project Structure

```
java-totp/
├── build.gradle              # Root build configuration
├── gradle.properties          # Project properties and metadata
├── settings.gradle            # Project structure definition
├── totp/                      # Main TOTP library module
│   └── build.gradle           # TOTP module configuration
└── totp-spring-boot-starter/  # Spring Boot starter module
    └── build.gradle           # Spring Boot starter configuration
```

## Building the Project

### Clean Build
```bash
./gradlew clean build
```

### Build Without Tests
```bash
./gradlew clean build -x test
```

### Run Tests Only
```bash
./gradlew test
```

### Build Single Module
```bash
./gradlew :totp:build
./gradlew :totp-spring-boot-starter:build
```

## Project Configuration

### gradle.properties

The `gradle.properties` file contains all project metadata:

```properties
# Project Properties
GROUP=dev.samstevens.totp
VERSION_NAME=1.7.1

# POM Properties
POM_NAME=java-totp
POM_DESCRIPTION=A library to help implement time-based one time passwords to enable MFA
POM_URL=https://github.com/samdjstevens/java-totp

# License Properties
POM_LICENSE_NAME=MIT License
POM_LICENSE_URL=http://www.opensource.org/licenses/mit-license.php

# SCM Properties
POM_SCM_URL=https://github.com/samdjstevens/java-totp
POM_SCM_CONNECTION=scm:git:git://github.com/samdjstevens/java-totp.git
POM_SCM_DEV_CONNECTION=scm:git:ssh://github.com/samdjstevens/java-totp.git
```

## Publishing to Maven Central (OSSRH)

### Prerequisites

1. **Sonatype OSSRH Account**: Create an account at https://oss.sonatype.org
2. **GPG Key**: Generate a GPG key for signing artifacts
3. **Environment Variables**: Set credentials as environment variables

### Environment Variables

Export the following environment variables:

```bash
export MAVEN_USERNAME="your-ossrh-username"
export MAVEN_PASSWORD="your-ossrh-password"
export GPG_KEY_ID="your-gpg-key-id"
```

### Publishing Tasks

#### Generate POM Files
```bash
./gradlew generatePomFileForMavenJavaPublication
```

#### Publish to Local Maven Repository
```bash
./gradlew publishToMavenLocal
```

#### Publish to OSSRH (Staging)
```bash
./gradlew publish
```

This will:
- Publish the main JAR
- Publish sources JAR (-sources)
- Publish Javadoc JAR (-javadoc)
- Sign all artifacts (if GPG is configured)

#### Custom Publish Tasks
```bash
# Publish snapshot version
./gradlew publishSnapshot

# Publish release version
./gradlew publishRelease
```

### Snapshot vs Release Versions

- **SNAPSHOT version**: Deployed to snapshots repository
  - Version format: `1.7.1-SNAPSHOT`
  - URL: `https://oss.sonatype.org/content/repositories/snapshots`

- **Release version**: Deployed to staging repository
  - Version format: `1.7.1`
  - URL: `https://oss.sonatype.org/service/local/staging/deploy/maven2/`

### Signing Configuration

To enable GPG signing during publication, ensure `GPG_KEY_ID` environment variable is set:

```bash
export GPG_KEY_ID="your-gpg-key-id"
./gradlew publish -Psigning.gnupg.keyName=your-gpg-key-id
```

## Available Gradle Commands

### List All Tasks
```bash
./gradlew tasks --all
```

### List Publishing Tasks
```bash
./gradlew tasks --group publishing
```

### Check Project Structure
```bash
./gradlew projects
```

### View Dependencies
```bash
./gradlew dependencies
```

### Generate Javadoc
```bash
./gradlew javadoc
```

### Code Coverage Report (JaCoCo)
```bash
./gradlew jacocoTestReport
```
Reports available at: `build/reports/jacoco/test/html/index.html`

## Modules

### 1. totp (Main Library)
- Core TOTP implementation
- Dependencies: commons-codec, commons-net (optional), google-zxing
- Test dependencies: JUnit 5, Mockito

### 2. totp-spring-boot-starter (Spring Boot Integration)
- Spring Boot auto-configuration for TOTP
- Depends on: totp module, spring-boot-autoconfigure
- Includes: Spring Boot configuration processor for IDE support

## Gradle Properties

### Java Configuration
- **sourceCompatibility**: Java 21
- **targetCompatibility**: Java 21
- **encoding**: UTF-8

### Gradle Daemon
- Enabled for faster builds
- JVM args: `-Xmx4g`
- Java home: `/usr/lib/jvm/java-21-openjdk-amd64`

### Parallel Build
- Enabled for multi-module builds

## Artifacts Generated

After a successful build, the following artifacts are created:

### Main JAR
- `totp/build/libs/totp-1.7.1.jar`
- `totp-spring-boot-starter/build/libs/totp-spring-boot-starter-1.7.1.jar`

### Sources JAR
- `totp/build/libs/totp-1.7.1-sources.jar`

### Javadoc JAR
- `totp/build/libs/totp-1.7.1-javadoc.jar`

## Troubleshooting

### Build Fails with "classifier" Error
- Use `archiveClassifier` instead of `classifier` in newer Gradle versions

### GPG Issues During Publishing
- Ensure GPG is installed: `gpg --version`
- List available keys: `gpg --list-keys`
- Export GPG_KEY_ID: `export GPG_KEY_ID="your-key-id"`

### Credentials Not Found
- Verify `MAVEN_USERNAME` and `MAVEN_PASSWORD` environment variables are set
- Check credentials have access to OSSRH

### OSSRH Published But Not on Maven Central
1. Log in to https://oss.sonatype.org
2. Navigate to "Staging Repositories"
3. Find your staged repository
4. Click "Release" to promote to Maven Central
5. Wait for synchronization (typically 10-30 minutes)

## CI/CD Integration

### GitHub Actions Example
```yaml
name: Publish to Maven Central

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '21'
          distribution: 'openjdk'
      - name: Publish
        env:
          MAVEN_USERNAME: ${{ secrets.OSSRH_USERNAME }}
          MAVEN_PASSWORD: ${{ secrets.OSSRH_PASSWORD }}
          GPG_KEY_ID: ${{ secrets.GPG_KEY_ID }}
        run: ./gradlew publish
```

## Migration Notes

This project was migrated from Maven (pom.xml) to Gradle (build.gradle) with the following improvements:

- **Groovy DSL**: More readable and maintainable than XML
- **Better Publishing**: Cleaner POM generation with property-based configuration
- **Consistent Configuration**: gradle.properties for centralized metadata
- **Environment Variables**: Secure credential handling via environment variables
- **JaCoCo Integration**: Built-in code coverage reporting
- **Multi-module Support**: Proper handling of parent and child modules
- **Faster Builds**: Gradle's incremental compilation and caching

## Additional Resources

- [Gradle Documentation](https://docs.gradle.org)
- [Maven Publishing Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html)
- [Sonatype OSSRH Guide](https://central.sonatype.org/publish/publish-guide/)
- [Java-TOTP GitHub](https://github.com/samdjstevens/java-totp)


