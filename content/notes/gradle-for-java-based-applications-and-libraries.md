+++
title = "Gradle for Java-Based applications and libraries"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "alain.bouchard@appdirect.com"
disableToc = "false"
+++

- [Project structure for gradle project in Java](#project-structure-for-gradle-project-in-java)
- [In build.gradle](#in-buildgradle)
  - [JAVA plugin](#java-plugin)
  - [APPLICATION plugin](#application-plugin)
- [Commands](#commands)
- [Maven Dependencies](#maven-dependencies)
- [Testing with Gradle](#testing-with-gradle)
  - [JUnit 5 dependencies](#junit-5-dependencies)

## Project structure for gradle project in Java

- single Java project with Gradle
  - example:

  ```text
    root
     + src
       + main
       + java
       + resources

       + test
         + java
       + resources
     + build
       + classes    -> compiled class files
       + libs       -> generated JAR files
  ```

- A multi-modules project file structure:
  - example:

  ```text
    root/
     + appA/
        + build.gradle
           + src/
           + main/
           + java/
           + resources/
           + test/
           + java/
           + resources/
        + build/
           + classes/    -> compiled class files
           + libs/       -> generated JAR files
     + appB/
        + build.gradle
        + src/
       ... (same as above subproject)
  ```

## In build.gradle

### JAVA plugin

- add plugins:

  ```groovy
    plugins {
      id 'java'
    }
  ```

- set the Java compatibility:

  ```groovy
    java {
      sourceCompatibility = JavaVersion.VERSION_11
      targetCompatibility = JavaVersion.VERSION_11
    }
  ```

- compiler arguments:

  ```groovy
    compileJava {
      // example: to terminate compilation if a warning occures)
      options.compilerArgs << '-Werror'
    }
  ```

- set JAR filename explicitly:

  ```groovy
    version = '1.0.0'

    jar {
      archiveBaseName = '<your-project-name>'
    }
  ```

Set javadoc options:

  ```groovy
  javadoc {
    options.header = '<your java doc title here>'
    options.verbose()
  }
  ```

### APPLICATION plugin

- in `build.gradle` file:

  ```groovy
    plugins {
      id 'application'
    }
  ```

- set base class to run for the application as **needed** by plugin:

  ```groovy
    application {
      mainClass = 'com.<organization>.<project>.Main'
    }
  ```

- in `settings.gradle` file:

  ```groovy
    rootProject.name = '<project_name>'
    include ':appA', ':appB'
  ```

## Commands

- `gradle wrapper`  -> Create Gradle Wrapper
- `gradle clean`    -> Clean dist output?

- with java plugin:
  - `./gradlew compileJava --console=verbose`
  - `./gradlew processResources --console=verbose`
- or both above command agragated command:
  - `./gradlew classes --console=verbose`

- see dependencies tree (and if it can be found)
  - `./gradlew dependencies`

- generate JAR file
  - `./gradlew jar` -> dropped JAR in build/libs directory

- with application plugin:
  - `./gradlew run --args="add 1 2"`  -> specifies arguments
  - `./gradlew installDist`           -> generate shippable application with scripts
  - `./gradlew distZip distTar`       -> bundle distributhe appliation
  - `./gradlew javadocs`              -> genetate java doc in build/docs/javadoc (index.html)

- project or multi-modules project:
  - `./gradlew project`               -> show project and show projects structure

## Maven Dependencies

- where to find? [https://search.maven.org]
- dependency coordinates:
  - example: `commons-cli:commons-cli:1.4` -> `<group>:<artifact>:<version>` or GAV

- set the repositories (where to get from) and dependencies:

  ```groovy
    repositories {
      mavenCentral()
    }

    dependencies {
      implementation 'commons-cli:commons-cli:1.4'  // get from maven repo search results
      implementation project(':appA')               // add project dependencies (other modules from this project)
      }
  ```

## Testing with Gradle

### JUnit 5 dependencies

- from search.maven.org:
- search org.junit.jupiter (latest version is 5)
  - minimum needed is junit-jupiter-api and junit-jupiter-engine
- declaring test dependencies:
  - testImplementation -> work on compilation and test execution
  - testRuntime -> work on runtime only
- adding dependencies:
  - copy & paste Gradle Groovy DSL path for both depenencies

  ```groovy
    dependencies {
      implementation 'commons-cli:commons-cli:1.4' // not a testImplementation!
      testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'  // test dependency
      testRuntime 'org.junit.jupiter:junit-jupiter-engine:5.7.0'  // test dependency on run time only
    }
  ```

- commands:
  - `./gradlew compileTestJava` -> compile tests
  - `./gradlew test`            -> run the tests (see useJUnitPlatform comment in order to use JUnit 5)
  - go to `build/reports/tests/test` the automaticaly gradle generated HTML report (index.html)
  - go to `build/test-reults/test` for the autmaticaly gradle generated XML test report
- adding test task in `build.gradle`:

  ```groovy
    test {
      useJUnitPlatform()  // Java plugin expect Java 4 by default. Add useJUnitPlatform to indicates to use JUnit5 instead

      testLogging {
        events 'started', 'skipped', 'failed'  // will display specified events on run time
        exceptionFormat 'full'  // show stack trace on failures
      }
    }
  ```
