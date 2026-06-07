# Advanced Gradle Configuration Examples

This file contains advanced Gradle configuration examples for various publishing scenarios.

## 1. Publishing to Custom Nexus Repository

If you want to publish to a custom Nexus repository (like your example), you can extend the root `build.gradle`:

```groovy
// In build.gradle subprojects block, add or modify the publishing.repositories section:

publishing {
    repositories {
        // Sonatype OSSRH (default in current config)
        maven {
            name = 'OSSRH'
            url = version.endsWith('SNAPSHOT') ? 'https://oss.sonatype.org/content/repositories/snapshots' : 'https://oss.sonatype.org/service/local/staging/deploy/maven2/'
            credentials {
                username = System.getenv('MAVEN_USERNAME') ?: ''
                password = System.getenv('MAVEN_PASSWORD') ?: ''
            }
        }
        
        // Custom Nexus Repository (Example)
        maven {
            name = 'CustomNexus'
            url = version.endsWith('SNAPSHOT') ? 
                'http://vicky-nas.tail70d9.ts.net:8081/repository/maven-snapshots/' :
                'http://vicky-nas.tail70d9.ts.net:8081/repository/maven-releases/'
            isAllowInsecureProtocol = true
            credentials {
                username = System.getenv('MAVEN_USERNAME') ?: ''
                password = System.getenv('MAVEN_PASSWORD') ?: ''
            }
        }
    }
    
    // ... rest of publishing configuration
}
```

### Publishing to Custom Nexus:

```bash
# Export credentials
export MAVEN_USERNAME="your-username"
export MAVEN_PASSWORD="your-password"

# Publish snapshot
./gradlew publishMavenJavaPublicationToCustomNexusRepository

# Publish release
./gradlew publishAllPublicationsToCustomNexusRepository
```

## 2. Using Snapshot Suffix Property

Similar to the example you showed, add this to your root `build.gradle`:

```groovy
def snapshotSuffix = project.properties['snapshotSuffix']

subprojects {
    // ...
    publishing {
        publications {
            mavenJava(MavenPublication) {
                // ...
                version = snapshotSuffix != null ? 
                    "${property('VERSION_NAME')}-${snapshotSuffix}-SNAPSHOT" :
                    property('VERSION_NAME').toString()
            }
        }
    }
}

tasks.register("publishSnapshot") {
    doFirst {
        if (!project.hasProperty("snapshotSuffix")) {
            throw new GradleException("Please provide snapshotSuffix property. Usage: ./gradlew publishSnapshot -PsnapshotSuffix=<suffix>")
        }
    }
    dependsOn 'publish'
}

tasks.register("publishRelease") {
    dependsOn 'publish'
}
```

### Usage:

```bash
# Publish snapshot with suffix
./gradlew publishSnapshot -PsnapshotSuffix=rc1

# Publish release
./gradlew publishRelease
```

## 3. Version Management with Gradle Properties

Update `gradle.properties` to support snapshot versioning:

```properties
# Version Properties
VERSION_NAME=1.7.1
SNAPSHOT_SUFFIX=

# If you want to enable snapshots, use:
# VERSION_NAME=1.7.2-SNAPSHOT
```

Then reference in `build.gradle`:

```groovy
group = property('GROUP').toString()
version = property('VERSION_NAME').toString()

// Or with snapshot suffix:
def snapshotSuffix = project.findProperty('SNAPSHOT_SUFFIX')
version = snapshotSuffix ? "${property('VERSION_NAME')}-${snapshotSuffix}" : property('VERSION_NAME').toString()
```

## 4. Multi-Repository Publishing Strategy

Create a `build.gradle` configuration that supports multiple repositories:

```groovy
subprojects {
    publishing {
        repositories {
            maven {
                name = 'OSSRH'
                url = 'https://oss.sonatype.org/content/repositories/snapshots'
                credentials {
                    username = System.getenv('OSSRH_USERNAME')
                    password = System.getenv('OSSRH_PASSWORD')
                }
            }
            maven {
                name = 'CompanyNexus'
                url = 'http://nexus.company.com:8081/repository/maven-public/'
                credentials {
                    username = System.getenv('NEXUS_USERNAME')
                    password = System.getenv('NEXUS_PASSWORD')
                }
            }
            maven {
                name = 'LocalArtifactory'
                url = 'http://artifactory.local:8081/artifactory/libs-release'
                credentials {
                    username = System.getenv('ARTIFACTORY_USERNAME')
                    password = System.getenv('ARTIFACTORY_PASSWORD')
                }
            }
        }
    }
}
```

## 5. Conditional Publishing Based on Branch

Add to root `build.gradle`:

```groovy
// Publish to different repositories based on git branch
subprojects {
    publishing {
        repositories {
            maven {
                def branch = "git rev-parse --abbrev-ref HEAD".execute().text.trim()
                name = branch == 'main' ? 'ProductionNexus' : 'DevelopmentNexus'
                url = branch == 'main' ? 
                    'http://nexus.prod.com:8081/repository/maven-releases/' :
                    'http://nexus.dev.com:8081/repository/maven-snapshots/'
                credentials {
                    username = System.getenv('NEXUS_USERNAME')
                    password = System.getenv('NEXUS_PASSWORD')
                }
            }
        }
    }
}
```

## 6. GPG Signing Configuration

Full GPG signing setup:

```groovy
// In subprojects block:

signing {
    sign publishing.publications.mavenJava
    
    // Method 1: Using GPG_KEY_ID environment variable
    useGpgCmd()
    
    // Method 2: Using in-memory signing (requires GPG_KEY and GPG_PASSPHRASE)
    def signingKey = System.getenv("GPG_KEY")
    def signingPassword = System.getenv("GPG_PASSPHRASE")
    if (signingKey && signingPassword) {
        useInMemoryPgpKeys(signingKey, signingPassword)
    }
}
```

### Environment variables for signing:

```bash
# Method 1: GPG command-line
export GPG_KEY_ID="your-key-id"

# Method 2: In-memory signing
export GPG_KEY="<ascii-armored-key>"
export GPG_PASSPHRASE="your-passphrase"
```

## 7. Dependency Management with BOM (Bill of Materials)

```groovy
// In gradle.properties
BOM_VERSION=3.5.5-0
BOM_GROUP=com.garverp

// In build.gradle subprojects
dependencies {
    implementation platform("${property('BOM_GROUP')}:bom:${property('BOM_VERSION')}")
    
    // These will inherit versions from BOM
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'
}
```

## 8. Testing Configuration Examples

```groovy
// In individual module build.gradle

// JUnit 5 configuration
test {
    useJUnitPlatform()
    
    // Test configuration
    testLogging {
        events "passed", "skipped", "failed"
        exceptionFormat "full"
    }
}

// Code coverage with JaCoCo
jacoco {
    toolVersion = "0.8.13"
}

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        html.required = true
        csv.required = false
    }
}

// Generate coverage report after tests
check.dependsOn jacocoTestReport
```

## 9. Custom Gradle Tasks

Add to `build.gradle`:

```groovy
// Task to show project info
tasks.register('projectInfo') {
    doLast {
        println "Project: ${rootProject.name}"
        println "Group: ${project.group}"
        println "Version: ${project.version}"
        println "Java: ${System.getProperty('java.version')}"
    }
}

// Task to prepare for release
tasks.register('prepareRelease') {
    dependsOn 'clean', 'build', 'jacocoTestReport'
    doLast {
        println "Release preparation complete!"
        println "Run './gradlew publish' to publish to Maven Central"
    }
}

// Task to list artifacts
tasks.register('listArtifacts') {
    doLast {
        fileTree("${buildDir}/libs") {
            include '**/*.jar'
        }.files.each { File file ->
            println "- ${file.name}"
        }
    }
}
```

## 10. Publishing to Maven Local for Testing

Before publishing to a remote repository, test locally:

```bash
# Publish to local Maven repository (~/.m2/repository)
./gradlew publishToMavenLocal

# Then in another project, test with:
# Add to your build.gradle:
# repositories { mavenLocal() }
# dependencies { implementation 'dev.samstevens.totp:totp:1.7.1' }
```

## Quick Reference: Publishing Commands

```bash
# Build everything
./gradlew clean build

# Generate artifacts
./gradlew jar sourcesJar javadocJar

# Verify before publishing
./gradlew check

# Publish to local Maven repository
./gradlew publishToMavenLocal

# Publish to OSSRH (Sonatype)
./gradlew publish

# Publish with custom properties
./gradlew publish -PsnapshotSuffix=beta1

# View all available tasks
./gradlew tasks --all

# View only publishing tasks
./gradlew tasks --group publishing
```

## Useful Environment Variables

```bash
# Maven Publishing
export MAVEN_USERNAME="username"
export MAVEN_PASSWORD="password"

# GPG Signing
export GPG_KEY_ID="key-id"
export GPG_PASSPHRASE="passphrase"

# Custom Nexus
export NEXUS_USERNAME="username"
export NEXUS_PASSWORD="password"
export NEXUS_URL="http://nexus.example.com"

# Build Configuration
export GRADLE_OPTS="-Xmx4g"
export JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
```

## Migrating from Maven to Gradle

If you need to migrate other projects from Maven to Gradle:

```bash
# Generate initial Gradle build from pom.xml
gradle init --type pom

# Or manually:
# 1. Create build.gradle in project root
# 2. Run: gradle wrapper --gradle-version 8.5
# 3. Test build: ./gradlew build
# 4. Gradually move configuration from pom.xml to build.gradle
```


