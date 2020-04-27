---
title: "Gradle"
last_modified_at: 2020-04-26T11:20:02-05:00
categories:
  - Gradle
  - Builds
tags:
  - Gradle
  - Kotlin
---
Gradle
------
If you are not already familiar with [Gradle](https://www.gradle.org), its an open source 
build tool that is commonly used in software projects, particularly
projects using Java and other JVM-based programming languages like 
Kotlin or Scala.

Initially Gradle build scripts utilized the Groovy programming language,
however starting with version 5.0 support for Kotlin-based build
scripts 

If you are familiar with Gradle you are likely used to the default
Groovy-based build script.

First, the project structure.  If you have experience using Gradle
in the past then there shouldn't be any surprises.

```
gradle-template
|---src
|   |---main
|   |   |---java
|   |   |   |---io.github.tduncan
|   |   |   |   |---Main.java
│---build.gradle.kts
|---settings.gradle.kts  
```

Our `settings.gradle.kts` file contains just a single line.

```kotlin
rootProject.name = "gradle-template"
```

```kotlin
plugins {
    java
    application
}

configure<JavaPluginConvention> {
    targetCompatibility = JavaVersion.VERSION_14
    sourceCompatibility = JavaVersion.VERSION_14
}

application {
    mainClassName = "io.github.tduncan.Main"
}

group = "io.github.tduncan"
version = "1.0.0-SNAPSHOT"
```