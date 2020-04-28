---
title: "Gradle: Starting Simple"
last_modified_at: 2020-04-27
categories:
  - Gradle
tags:
  - Gradle
  - Kotlin
version: 1.0.1
---
If you are not already familiar with [Gradle](https://www.gradle.org), its an open source 
build tool that is commonly used in software projects, particularly
Java and other JVM-based programming languages like [Kotlin](https://kotlinlang.org/) or [Groovy](https://groovy-lang.org/).

Let's take a look at a simple Gradle build script using the Kotlin
DSL.  First, the project structure.  If you have experience using Gradle 
there shouldn't be any surprises here.  If you haven't, this is 
the standard Gradle boilerplate.

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

Our `settings.gradle.kts` file is very basic, defining only the 
name of our root project.

```kotlin
rootProject.name = "gradle-template"
```
The `settings.gradle.kts` is how we teach Gradle about the specific
projects that need to be evaluated and built.  Our example is very
simple; we only need a top-level project, but more complicated 
projects likely have a number of sub projects that also need to be
built by Gradle.

Finally, `build.gradle.kts` script itself.  This is where we will
define how Gradle should evaluate our project and, by extension,
what Gradle tasks are available to execute.
```kotlin
plugins {
    java
    application
}

application {
    mainClassName = "io.github.tduncan.Main"
}
```
At the top of the build script we immediately inform Gradle what
type of application we are building by the plugins being applied
to the project.  Among other things, using the `java` plugin will 
configure Gradle to expect Java source code in the 
`src/main/java` directory.

Similarily, the `application` configures Gradle to expect a fully
qualified class name to be provided as the `mainClassName`.  This
class needs to contain a valid Java main method, and be located
within the `src/main/java` directory.  As long as we do this, we
can take advantage the `run` task included by default by the `application`
plugin to quickly run our program from the command line.

`Main.java` is, as the name implies, a very simple "Hello World!"
main method.
```java
package io.github.tduncan;

class Main {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

Now, open up your favorite terminal and execute the `run` command.

```
$ gradle run                                 
                                             
> Task :run                                  
Hello World!                                 
                                             
BUILD SUCCESSFUL in 838ms                    
2 actionable tasks: 1 executed, 1 up-to-date 
```

Source Code: https://github.com/tduncan/gradle-template