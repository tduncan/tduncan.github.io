---
title: "My Software Toolbox"
last_modified_at: 2020-06-25
version: 1.1.1
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
---
Like every good practitioner, a software engineer needs a set of high quality, reliable tools that empower us to build
software quickly and with minimal development friction. I primarily write backend software for the JVM using Java, and 
my personal toolbox quite clear reflects as much.

I also have a strong preference for proven commodities. Though the Java ecosystem is quite mature, it is always
evolving with new frameworks, libraries, and tools are releasing frequently. I need to ship working software, however, 
and I need to have confidence that my tools works, and that I know how to work my tools. For this reason I lean heavily
on those tools that have proven themselves time and time again that they can be relied on, have a sufficiently large 
community to lean on for help, and that include high quality, ideally example-driven, documentation. 

That's not to say my toolbox is set in stone, however. The world of software engineering changes too quickly and I
would quickly find my skills and knowledge lagging behind if I were to stop considering new tools to add or replace an
existing tool, or even drop altogether. The release of Java 8 would be a great example of this. At that time my
toolbox would have included the [Joda-Time](https://www.joda.org/joda-time/) library as the original data and time 
facilities in Java left a lot to be desired. Jodatime fixed basically all of that while also providing you a way to 
covert to and from the JDK date and time classes without much effort. Java 8 provided a new and vastly improved datetime
API written by the author of [Joda-Time](https://www.joda.org/joda-time/) [Stephen Colebourne](https://blog.joda.org/).
[Joda-Time](https://www.joda.org/joda-time/) is a great library, but so is the Java 8 datetime API. When faced with a
choice between a great JDK solution and a great third-party library solution I will almost always choose the JDK and
celebrate the opportunity to remove an external dependency from a project.

So, here's my current set of tools for building a Java application.

**IntelliJ IDEA**

IntelliJ may be the most popular Java IDE, and for very good reason. It just works, and the experience of writing code
in IntelliJ is super smooth. I absolutely adore the code completion in this IDE. When I'm working in a familiar
codebase the suggested completions are so frequently correct that, at times, I can produce entire blocks of code using
just a handful ot keystrokes. The feeling of being in that kind of flow is exhilarating, and not one I have known using
other Java IDEs. 

**Test Driven Development**

I believe TDD to be a productivity force multiplier, especially over time as the software matures and the behavior 
begins to stabilize. The process helps me work in small chunks, learning and improving both the code and my
understanding. It also defaults to impressively high code coverage numbers.

* Domain Driven Design
* Ports and Adapters
* Gradle
* Spring/Spring Boot
* JUnit 5
* Mockito
* AssertJ
* Jackson
* Caffeine
* Test Containers
* Docker
* Docker Compose
* JMH
* SLF4J
* Log4J2