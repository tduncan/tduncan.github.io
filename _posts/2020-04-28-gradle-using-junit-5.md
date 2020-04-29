---
title: "Gradle: Using JUnit 5"
last_modified_at: 2020-04-28
categories:
  - Gradle
tags:
  - Gradle
  - Junit 5
version: 1.0.2
---
[JUnit 5](https://junit.org/junit5/) is the newest release of
the popular JUnit testing framework and offers a number of
improvements over [JUnit 4](https://junit.org/junit4/).  We'll
cover many of those improvements, but here we're interested in
how setup Gradle to execute JUnit 5 tests.

Gradle makes it very simple to quickly execute a suite of 
automated tests for a Java project. Applying the `java`
plugin (as we did in our [Gradle: Starting Simple](/gradle/gradle-starting-simple/)
example) configures your project with a `test` Gradle task.
As of Gradle version 6.3, however, the default configuration
of the `test` task is to expect JUnit 4 tests to execute. So,
we need to do a bit of work for JUnit 5.

First we need to expand our `build.gradle.kts` script to depend
on the JUnit 5 library.

```kotlin
repositories {
    mavenCentral()
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.6.2")
}
```

There is a bit to unpack here, so let's take a moment to explain
what we're seeing. The `dependencies` block is the section of
the build script where we declare what libraries we will be using.
Gradle will use this information to download the libraries and
include them on our project classpath.  Gradle will also handle
downloading the including any libraries required by our 
dependencies.  These are referred to as transitive dependencies.

From where does Gradle download these libraries?  Great question!
Gradle has built-in support for all Maven compatible artifact
repositories, the biggest of which is [Maven Central](https://mvnrepository.com/),
which is where we've instructed Gradle to look.

Now, let's take another look at the JUnit 5 dependency itself.

```kotlin
testImplementation("org.junit.jupiter:junit-jupiter:5.6.2")
```

Here we are specifying the identifying features of the 
library we want Gradle to download.  Typically, this means
providing the group, module, and version which above work out to
`org.junit.jupiter`, `junit-jupiter`, and `5.6.2`. Collectively
these are known as the module coordinates and direct Gradle to 
the correct artifact in the repository.

Finally, let's direct our attention to the `testImplementation`
part of the dependency.  Here we are instructing Gradle to
only associate our JUnit 5 dependency with the test configuration.
We haven't touched on configurations before, but for now just
know that Gradle is associating dependencies with specific
sets of source code, of which we currently have main and test.

We're almost there!  With JUnit 5 downloaded and available on
the class path for our tests we have just one more thing to do.

```kotlin
tasks.test {
    useJUnitPlatform()
}
```

As I mentioned at the start of the post the default test library
for a Gradle Java project is JUnit 4.  To use JUnit 5 we need
to configure the `test` task to use the JUnit 5 platform for
running tests.  With this now done, we're ready to go!

Let's try running the tests.  From your terminal run the `test`
task using the Gradle Wrapper.
```
$ ./gradlew test

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

And that's it!  Now, for some actual tests!

Source Code: https://github.com/tduncan/gradle-using-junit-5