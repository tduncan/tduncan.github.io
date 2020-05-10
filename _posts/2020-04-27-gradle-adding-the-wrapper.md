---
title: "Gradle: Adding the Gradle Wrapper"
last_modified_at: 2020-05-09
categories:
  - Gradle
tags:
  - Gradle
version: 1.1.0
author:
  name   : "Thomas Duncan"
  avatar : "/assets/images/photo.jpg"
  bio    : "Software Engineer.  TDD Enthusiast."
  location: "San Francisco Bay Area"
  links:
    - label: "Website"
      icon: "fas fa-fw fa-link"
      url: "https://tduncan.github.io/"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/tduncan"
    - label: "Source Code"
      icon: "fas fa-code"
      url: "https://github.com/tduncan/gradle-adding-wrapper"
---
[Previously](https://tduncan.github.io/gradle/gradle-starting-simple/)
we created a simple Java project using Gradle.  This is a great
first step, however it requires anyone (including our future selves!)
to have a compatible version of Gradle already installed 
to work with the project.

Fortunately, Gradle has a very elegant solution to this problem 
in the form of the [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html).
The Gradle Wrapper is a script that will use a version, specified
by the project itself, of Gradle.  In addition, if that version 
is not available the script will automatically download it first!
Using the Gradle Wrapper is absolutely the best and most user-friendly
way to interact with a Gradle project.

And more good news, adding the Gradle Wrapper to our existing
simpel Gradle project is also easy!  Every basic Gradle project
is already outfitted with a `wrapper` task that will convert the
project into a Gradle Wrapper project.  So, lets run the task!

```
$ gradle wrapper

BUILD SUCCESSFUL in 759ms
1 actionable task: 1 executed
```

That's it!

If we look back at our project we'll see that Gradle has added a
few new files for us.

```
gradle-template
|---gradle
|   |---wrapper
|   |   |---gradle-wrapper.jar**
|   |   |---gradle-wrapper.properties**
|---src
|   |---main
|   |   |---java
|   |   |   |---io.github.tduncan
|   |   |   |   |---Main.java
│---build.gradle.kts
|---gradlew**
|---gradlew.bat**
|---settings.gradle.kts  
```

We can see the new `/gradle/wrapper` directory has been added,
and of particular interest is the `gradle-wrapper.properties`
file that was automatically created for us.  Taking a peek
inside we'll find a few properties, but I want to specifically
point out the `distributionUrl` property.

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-6.3-bin.zip
```

The `distributionUrl` property is what the Gradle Wrapper will
use to download the Gradle distribution if it isn't already
installed.  It's also where we can quickly update what version
of Gradle our project uses.  Notice the Gradle version in the 
URL?  As new versions of Gradle are released simply updating
that number is all you, and by extension anyone, needs to do!
Super cool!

The actual Gradle Wrapper scripts are the `gradlew` and
`gradlew.bat` in the root directory of our project.  You should
not need to edit these files, but they do need to be stored in 
version control with the rest of the project source code as these
scripts are now what we will use to execute Gradle tasks.

To run our code now looks like this using the Gradle Wrapper:
```
$ ./gradlew run

> Task :run
Hello World!

BUILD SUCCESSFUL in 1s
2 actionable tasks: 2 executed
```

If you were to run the same task, but the Gradle distribution
was not installed, it would be automatically downloaded first.

```
$ ./gradlew run
Downloading https://services.gradle.org/distributions/gradle-6.3-bin.zip
.........10%..........20%..........30%.........40%..........50%..........60%.........70%..........80%..........90%..........100%

> Task :run
Hello World!

BUILD SUCCESSFUL in 12s
2 actionable tasks: 1 executed, 1 up-to-date
```

Where is the distribution downloaded to?  They can be found
in `~/.gradle/wrapper/dists` in case you want to clean them 
up from time-to-time.