Description
===========

This project is meant to showcase a discrepancy between how sbt and coursier
resolve dependencies. 

If you try to compile the project, you'll see the below error reported:

```bash
[info] Compiling 1 Scala source to /Users/mirco/tmp/coursier-test/target/scala-2.10/classes...
[error] /Users/mirco/tmp/coursier-test/src/main/scala/App.scala:5: value configure is not a member of object ch.qos.logback.classic.BasicConfigurator
[error]   BasicConfigurator.configure(new LoggerContext())
[error]                     ^
[error] one error found
[error] (compile:compileIncremental) Compilation failed
[error] Total time: 2 s, completed Jun 7, 2017 3:12:29 PM
```

The problem has to do with the `logback-classic` JAR version that is in the classpath. The project's build specifies `logback-classic` version `1.1.1`, which does contain the static method `configure` in the class `BasicConfigurator`. Surprisingly, it's not version `1.1.1` that is going to be in the compile classpath. The `logback-classic` version used is actually `1.1.7`, and compilation fails because there is no static method named `configure` in the `BasicConfigurator` class.

Why is that happening? If you look [here](https://github.com/dotta/coursier-deps-resolution-bug/blob/master/project/src/main/scala/sbt/MyPlugin.scala#L13-L16), you'll see there is a sbt plugin that is scoping the `logback` dependency version `1.1.7` to a custom configuration `Foo`. The configuration `Foo` doesn't extend any other configuration, so there should be no interaction between the dependency scoped to `Foo` and the project ones that are scoped to `Compile`. Unfortunately, that doesn't seem to be true when using coursier, as this project demonstrates. However, if the coursier plugin is removed, then the project will compile just fine.

An additional discrepancy I noticed between the stock sbt behavior and coursier is that if [this line](https://github.com/dotta/coursier-deps-resolution-bug/blob/master/project/src/main/scala/sbt/MyPlugin.scala#L17) is removed, resolution will fail with the stock sbt, while it will work just fine with coursier. The error reported by sbt after removing the mentioned line is:

```bash
> compile
[info] Updating {file:/Users/mirco/tmp/coursier-test/}coursier-test...
[trace] Stack trace suppressed: run last *:update for the full output.
[error] (*:update) java.lang.IllegalArgumentException: Cannot add dependency 'org.slf4j#slf4j-api;1.7.0' to configuration 'foo' of module default#coursier-test_2.10;0.1-SNAPSHOT because this configuration doesn't exist!
```

Not sure if the two issues are connected.
