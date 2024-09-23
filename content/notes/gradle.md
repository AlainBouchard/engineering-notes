+++
title = "Gradle"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

## Gradle (6.6.1)

- requires JDK
- download from [https://gradle.org/releases/]
- can use either `Groovy` or `Kotlin` DSL (Domain-Specific Language)

## Terminology

- Project: models a software component
- Build script: contains automation instructions for a project
- Task: defines executable automation instructions
  - Ad hoc task: implements one-off, simplistic action code by defining doFirst or doLast, automatically extends DefaultTaslk without having to declare it
  - Typed task: Explicitly declares type (for example, Copy); does not need to define actions as they are already provided by type
- Wrapper:
  - set of files checked into SCM alongside source code
  - standardizes compatible Gradle version for a project
  - automatically downloads the Gradle distribution with defined version

## Actions

- doLast, doFirst, ...

## Files structure for a "Multi-module build"

```text
  root
   + build.gradle
   + moduleA/
   |  + build.gradle
   |   + src/
   + moduleB/
      + build.gradle
     + src/
```

## Commands

- `gradle <task name>`  -> run the task from the build.gradle
- `gradle wrapper`      -> create the wrapper files
  - creates `gradlew`, `gradlew.bat` and the gradle directories...
  - `gradle/wrapper/gradle-wrapper.properties`
  - example:

    ```text
      distributionBase=GRADLE_USER_HOME
      distributionPath=wrapper/dists
      distributionUrl=https\://services.gradle.org/distributions/gradle-6.6.1-bin.zip
      zipStoreBase=GRADLE_USER_HOME
      zipStorePath=wrapper/dists
    ```

  - `./gradlew <task name>`       -> download the wrapper gradle version and execute the task with it
- `gradle projects`               -> create settings for the project
  - Use `settings.gradle` do define information, example:

    ```groovy
      rootProject.name = "gradle-training"
    ```

- `gradle <task name> --dry-run`  -> show the tasks from the build.gradle but does not execute
- `gradle tasks --all`            -> show the full list of available tasks

## Example of Typed Task

- example:

  ```groovy
    task copyFiles(type: Copy) {
      from "sourceFiles"
      into "target"
      include "**/*md"
      includeEmptyDirs = false
    }

    task createZip(type: Zip) {
      from "build/docs"
      archiveFileName = "docs.zip"
      destinationDirectory = file("build/dist")
      dependsOn CopyFiles
    }
  ```

## Plugins

1. create a reusable or sharable `<plugin-name>.gradle` file with defined tasks
1. apply the plugin to the local project build.gradle file, example:
    1. plugin with tasks: `myPlugin.gradle`
    1. in `build.gradle`:

  ```groovy
    apply from: "myPlugin.gradle"
  ```

- Notes:
  - available plugins are Core Plugins (from Gradle) and Community Plugins (not from Gradle)
  - to apply Core plugin Base (for example), in `build.gradle`:

  ```groovy
    apply plugin: 'base'
  ```
