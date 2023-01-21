---
layout: post
title:  "Java Log Framework suggestion by ChatGPT"
date:   2023-01-21 14:30:00 -0300
categories: Java Log ChatGPT |AI
---

<div class="me_quote">
Hi!
</div>

Hello! How can I help you today?

<div class="me_quote">
What are the most used Java log frameworks?
</div>

The most commonly used Java logging frameworks are:

    Log4j
    java.util.logging (built-in Java logging)
    Logback
    Apache Commons Logging
    SLF4J (Simple Logging Facade for Java)

SLF4J is a facade that allows the user to plug in the desired logging framework at deployment time. Log4j and Logback are both created by the same author and share similar features, Log4j is considered more stable and widely used.

<div class="me_quote">
what about log4j2?
</div>

Log4j2 is also a popular logging framework for Java, it is an upgraded version of log4j, which was the most popular logging framework for Java before. Log4j2 provides several significant improvements over Log4j 1.x such as:

    Asynchronous Logging
    Automatic Reloading of Configuration
    Garbage-free Logging
    Pluggable Layouts and Filters
    Java 8 Lambda support
    Support for various output destinations

Many projects that used log4j still use log4j2. It's recommended to use log4j2 if you are starting a new project or if you are willing to upgrade your current project.

<div class="me_quote">
How to use Log4j2 in my Java project?
</div>

To use Log4j2 in a Java project, you will need to perform the following steps:

Add the Log4j2 dependencies to your project. You can add the dependencies in your pom.xml file (if you are using Maven) or in your build.gradle file (if you are using Gradle). The dependencies you will need are:

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.13.3</version>
</dependency>
```

Create a log4j2.xml or log4j2.properties configuration file in the classpath of your application. This file contains the configuration settings for Log4j2, such as the appenders (where the log messages will be written), the log level, and the layout (how the log messages will be formatted).

In your Java code, import the following package:

```java
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;
```

In your class, create a static logger instance:

```java
private static final Logger logger = LogManager.getLogger(MyClass.class);
```

Use the logger instance to log messages:

```java
logger.info("Application started");
logger.error("An error has occurred", ex);
```

Optionally you can use annotations to configure the Logger

```java
@Log4j2
public class MyClass {
}
```

And that's it! Your Java application should now be configured to use Log4j2 for logging.

Note: Log4j2 uses different configuration files for different environments, it's recommended to use different configurations for development, testing, and production environments.

<div class="me_quote">
Please create a pom.xml file that uses log4j2
</div>

Sure, here is an example of a pom.xml file that includes the Log4j2 dependency:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.13.3</version>
        </dependency>
    </dependencies>

</project>
```

Make sure you have the correct version of log4j2, it's always recommended to use the latest version, you can check the latest version on the maven central repository, https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core

Also, make sure you have log4j2 configuration file under the classpath of your application, you can put it under the resources directory, and make sure the name of the configuration file is log4j2.xml or log4j2.properties.

Additionally, you may want to add a plugin to make sure the resources are copied to the classpath during packaging:

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-resources-plugin</artifactId>
            <version>3.2.0</version>
            <executions>
                <execution>
                    <id>copy-resources</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>copy-resources</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${basedir}/target/classes</outputDirectory>
                        <resources>
                            <resource>
                                <directory>src/main/resources</directory>
                            </resource>
                        </resources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

Let me know if you have any other questions.

<div class="me_quote">
Create log4j2.xml, please
</div>

Sure, here is an example of a log4j2.xml configuration file that sets up a file appender to write log messages to a file named application.log in the logs directory, and also sets up a console appender to write log messages to the console:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="ERROR">
    <Appenders>
        <File name="File" fileName="logs/application.log">
            <PatternLayout>
                <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n</pattern>
            </PatternLayout>
        </File>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout>
                <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n</pattern>
            </PatternLayout>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="File"/>
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

This configuration is using the default log level (debug), you can change it to the level you want, you have options like : trace,debug, info, warn, error, fatal and off.

The %d in the pattern is a date, %p is the level of the log, %c{1} is the name of the logger, %L is the line of the code that generated the log, %m is the message of the log and %n is the new line.

You also have other options to configure the appender like :

    BufferedIO
    ImmediateFlush
    FileName
    Append
    Locking
    MaxFileSize
    MaxBackupIndex
    CompressionType
    Advertiser

You can check the official documentation for more information and options :
https://logging.apache.org/log4j/2.x/manual/configuration.html

Please let me know if you have any questions or if there is anything else I can help you with.

<div class="me_quote">
When should I use BufferedIO?
</div>

BufferedIO is a feature that allows Log4j2 to buffer log events before they are written to the appender's output. This can be useful in certain situations because it can improve performance by reducing the number of I/O operations that need to be performed.

There are a few situations where you might want to use BufferedIO:

- When you are logging to a remote location: If your appender is configured to write log events to a remote location, such as a remote file system or a remote syslog server, buffering log events before they are written can help to reduce the number of network operations that need to be performed.

- When you are logging to a slow storage: If you are writing log events to a storage that is slow, such as a network-attached storage device or a cloud-based storage service, buffering log events before they are written can help to reduce the time it takes to write log events to the storage.

- When you are logging a high volume of data: If your application generates a high volume of log events, buffering log events before they are written can help to reduce the performance impact of writing log events to the appender's output.

It's worth noting that buffering log events can also have some downsides, such as potentially losing log events if the application crashes before they are written.

To use BufferedIO, you need to set the bufferedIO attribute to true on your appender, and also set the bufferSize attribute to specify the size of the buffer in bytes. For example:

```xml
<File name="File" fileName="logs/application.log" bufferedIO="true" bufferSize="8192">
```

It's important to test the buffering feature with your specific use case and data volume, to make sure it's beneficial for your scenario.

<div class="me_quote">
Is bufferSize in KB?
</div>

The bufferSize attribute in log4j2 refers to the size of the buffer in bytes.

In the example I provided earlier, you can set the buffer size to 8192 bytes which is 8 KB, this means that log4j2 will buffer log events until the buffer reaches 8KB before they are written to the appender's output.

It's important to note that the buffer size should be carefully chosen to balance the performance benefits of buffering log events with the risk of losing log events if the application crashes before they are written.

You can also set the buffer size to a value like "8KB" or "8192" and log4j2 will automatically recognize it.

```xml
<File name="File" fileName="logs/application.log" bufferedIO="true" bufferSize="8KB">
```

<div class="me_quote">
Create a basic Java class that make use of this log4j2 that we configured.
</div>

Sure, here's a basic example of a Java class that makes use of the Log4j2 configuration we previously discussed:

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class MyClass {
    private static final Logger logger = LogManager.getLogger(MyClass.class);

    public void doSomeWork() {
        logger.debug("Starting doSomeWork method");
        // Do some work here
        logger.info("Work completed successfully");
    }

    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.doSomeWork();
    }
}
```

In this example, we import the necessary Log4j2 classes and create a static logger instance for the MyClass class. Then, in the doSomeWork() method, we use the logger to log a debug message at the start of the method and an info message at the end, indicating that the work was completed successfully.

When the doSomeWork() method is called and the main method is executed, it will generate the following output in the console and in the application.log file:

DEBUG Starting doSomeWork method
INFO Work completed successfully

You can also use the logger to log messages at different levels like:

logger.trace("Trace message");
logger.debug("Debug message");
logger.info("Info message");
logger.warn("Warn message");
logger.error("Error message", ex);
logger.fatal("Fatal message");

It's important to note that the log level set on the log4j2.xml configuration file will determine which messages will be logged, messages with a lower level than the one set on the configuration file will be ignored.

I hope this example is helpful. Let me know if there's anything else I can help with.

<div class="me_quote">
what is Configuration status="ERROR" in log4j2 configuration file?
</div>

In Log4j2, the status attribute on the Configuration element in the configuration file is used to specify the level at which configuration errors should be reported. The attribute accepts one of four values: "trace", "debug", "info", and "error".

When the status attribute is set to "trace", all configuration messages will be logged. When set to "debug", all configuration messages except for trace messages will be logged. When set to "info", only configuration error and warning messages will be logged. And when set to "error", only configuration error messages will be logged.

In the example I provided earlier, the status attribute is set to "ERROR", This means that only configuration error messages will be logged. This is useful in production environments where you don't want to see too many log messages but you want to detect if there's a problem with the configuration of your logger.

It's important to note that this attribute is optional, if it's not set, the default value is "ERROR"
You can check the official documentation for more information and options : https://logging.apache.org/log4j/2.x/manual/configuration.html

Please let me know if you have any other questions.

<div class="me_quote">
How to run a maven project?
</div>

To run a Maven project, you will need to have the Maven build tool installed on your system. Once you have Maven installed, you can use the following command to run your project:

```sh
mvn compile
```

This command will compile your project, and it will create the target directory with all the compiled files.

You can also use the following command to run the test cases of your project:

```sh
mvn test
```

This command will execute all the test cases present in your project, and it will generate the test-report.

The most common command to run a Maven project is

```sh
mvn clean install
```

This command will clean any existing compiled files and resources, then it will download all the dependencies and plugins defined in the pom.xml file, it will compile the source code and run the test cases, then it will package the application and install it in the local repository.

You can also use the following command to run the main class of your project:

```sh
mvn exec:java -Dexec.mainClass="com.example.MyMainClass"
```

This command will execute the main class of your project, make sure you replace "com.example.MyMainClass" with the actual main class of your project.

You can also use the following command to generate the JavaDocs for your project:

```sh
mvn javadoc:javadoc
```

This command will generate the JavaDocs in the target/site/apidocs directory.

You can also use the following command to generate the project documentation using the maven-site-plugin

```sh
mvn site
```

This command will generate the project documentation in the target/site directory, it includes the JavaDocs, test report, and other project information.

It's recommended to check the official documentation for more information and options.
https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html

Please let me know if you have any other questions.

<div class="me_quote">
Include source and target versions in pom.xml for Java 17, please
</div>

To specify the source and target versions for Java 17 in a Maven project, you will need to include the maven-compiler-plugin in your pom.xml file and configure it to use Java 17 for both the source and target versions. Here's an example of how you can do this:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

You can replace version 3.8.0 with the latest version of the plugin.

It's important to note that you also have to have the Java 17 installed on your machine and set as the default JDK to be able to use it.

In addition, you might want to add the javax.annotation-api to your dependencies, as it is not included in Java 17.

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
```

<div class="me_quote">
what is the difference between source and target?
</div>

In the context of a Maven project, the source and target versions refer to the version of the Java programming language that the project's source code should be compiled against and the version that the compiled code should be compatible with, respectively.

The source version is the version of Java that the compiler uses to validate the syntax and semantics of the source code. The compiler will raise errors if any language features are used that are not available in the specified source version.

The target version is the version of Java that the compiler generates bytecode for. If the target version is lower than the source version, the compiler will translate the source code into a version of bytecode that can run on an older version of the Java runtime.

It's important to note that specifying a higher source version than the target version may cause the code to not run on the older JREs. Also, specifying a lower source version than the target version may prevent the use of new language features that are available in the newer version.

In general, it's a good practice to set the source version to the highest version of Java that you are using to develop your application, and the target version to the lowest version of Java that you want your application to support.

Also, it's recommended to check the official documentation for more information and options : https://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-source-and-target.html
Please let me know if you have any other questions.

<div class="me_quote">
Is it possible to dynamically change the log level?
</div>

Yes, it is possible to dynamically change the log level in Log4j2.

One way to do this is by using the Log4j2's configuration API to programmatically update the configuration of the loggers. Here is an example of how you can change the log level of a logger to debug:

```java
import org.apache.logging.log4j.core.LoggerContext;
import org.apache.logging.log4j.core.config.Configuration;
import org.apache.logging.log4j.core.config.LoggerConfig;

public class LogLevelChanger {
    public static void setLogLevel(String loggerName, String level) {
        LoggerContext ctx = (LoggerContext) LogManager.getContext(false);
        Configuration config = ctx.getConfiguration();
        LoggerConfig loggerConfig = config.getLoggerConfig(loggerName);
        loggerConfig.setLevel(Level.valueOf(level));
        ctx.updateLoggers();
    }
}
```

You can call the setLogLevel method and pass the logger name and the level that you want to set to the logger.
You can also use the following code snippet to change the root logger level

```java
LoggerContext ctx = (LoggerContext) LogManager.getContext(false);
Configuration config = ctx.getConfiguration();
config.getRootLogger().setLevel(Level.valueOf(level));
```

<div class="me_quote">
Is it possible to change using a Java parameter or environment variable?
</div>

Yes, it is possible to change the log level using a Java parameter or an environment variable.

One way to do this is by reading the value of the parameter or the environment variable in your code and passing it to the setLogLevel method. Here's an example of how you can read a Java parameter and use it to set the log level:

```java
public static void main(String[] args) {
    String logLevel = System.getProperty("log.level", "info");
    setLogLevel("com.example.MyClass", logLevel);
    // rest of the code
}
```

You can then set the log level when running your application by passing the -Dlog.level=debug parameter to the JVM.

Another way to do this is by reading the value of an environment variable and use it to set the log level:

```java
public static void main(String[] args) {
    String logLevel = System.getenv("LOG_LEVEL");
    if (logLevel != null) {
        setLogLevel("com.example.MyClass", logLevel);
    }
    // rest of the code
}
```

You can then set the log level by setting the LOG_LEVEL environment variable in your system.

It's important to note that you have to validate the value passed in the parameter or the environment variable to make sure it's a valid log level.

Also, you can use a combination of both methods to set the log level, first trying to read the environment variable and if it's not set, read the Java parameter.

<div class="me_quote">
how to use a different log4j file at runtime?
</div>

There are several ways to use a different log4j2 configuration file at runtime:

Using the -Dlog4j.configurationFile system property: You can specify the location of the log4j2 configuration file by setting the -Dlog4j.configurationFile system property to the file path when starting your application. For example:

```sh
java -Dlog4j.configurationFile=file:/path/to/log4j2.xml -jar myapp.jar
```

Using the log4j.configurationFile system property in the log4j2.xml file: you can set the log4j.configurationFile system property in the log4j2.xml file, this way you can change the configuration file location without modifying the command line or the code.

```xml
<Configuration status="WARN" monitorInterval="30" 
               properties="file:${sys:log4j.configurationFile}">
```

Using the log4j.configurationFactory system property: You can specify a different configuration factory by setting the log4j.configurationFactory system property to the fully qualified class name of the factory class when starting your application.

```sh
java -Dlog4j.configurationFactory=mypackage.MyConfigurationFactory -jar myapp.jar
```

Using the log4j.configurator system property: You can specify a different configurator by setting the log4j.configurator system property to the fully qualified class name of the configurator class when starting your application.

```sh
java -Dlog4j.configurator=mypackage.MyConfigurator -jar myapp.jar
```

It's worth noting that the first option is the most common and recommended way, the other options are used in specific scenarios like when the application needs to use different log4j2 configurations in different environments, or when you want to use a different configuration factory or configurator.

<div class="me_quote">
Does Log4j2 have support for memory mapped file?
</div>

Yes, Log4j2 has support for memory-mapped files through the RandomAccessFileAppender class. The RandomAccessFileAppender class is an appender that writes log events to a file using memory-mapped file I/O for maximum performance.

Here is an example of how you can configure a RandomAccessFileAppender in your log4j2.xml file:

```xml
<RandomAccessFile name="MyFile" fileName="application.log"
    immediateFlush="false" bufferSize="8192">
    <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
</RandomAccessFile>
```

In this example, we are creating a RandomAccessFileAppender named "MyFile" that writes to a file named "application.log", with a buffer size of 8192 bytes. It is also using a PatternLayout to format the log messages.

It's important to note that the RandomAccessFileAppender uses memory-mapped file I/O, which allows for fast and efficient file access, but it also has some limitations. For example, it's only supported on certain platforms and it can cause the JVM to run out of memory if the file size exceeds the available memory.

You should also check the official documentation for more information and options: https://logging.apache.org/log4j/2.x/manual/appenders.html#RandomAccessFileAppender

<div class="me_quote">
Show me a full example using RandomAccessFileAppender, please
</div>

Here is an example of a log4j2.xml file that configures a RandomAccessFileAppender:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="30">
    <Appenders>
        <RandomAccessFile name="MyFile" fileName="logs/application.log"
            immediateFlush="false" bufferSize="8KB">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </RandomAccessFile>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="MyFile"/>
        </Root>
    </Loggers>
</Configuration>
```

In this example, we are creating a RandomAccessFileAppender named "MyFile" that writes to a file named "application.log" in the "logs" folder, with a buffer size of 8KB. It also uses a PatternLayout to format the log messages. The immediateFlush attribute is set to false, means that the log events will be buffered until the buffer size is reached before they are written to the file.

We also have a Root logger that is set to the debug level, which means that it will log all messages of level debug, info, warn, error, and fatal. The AppenderRef element refers to the "MyFile" appender, this means that all log events generated by the Root logger will be written to the "MyFile" appender.

It's important to note that you have to have the folder "logs" created in your project, if not the appender will fail to create the file.

It's also possible to customize the configuration to suit your needs, for example, you can add more appenders, configure different loggers with different levels, or use different layout patterns.

<div class="me_quote">
What does monitorInterval mean?
</div>

In Log4j2, the monitorInterval attribute on the Configuration element in the configuration file is used to specify the interval (in seconds) at which the configuration file should be checked for changes. If a change is detected, the new configuration will be applied automatically.

For example, if you set the monitorInterval to 30 seconds, Log4j2 will check the configuration file every 30 seconds to see if it has been modified. If it has been modified, the new configuration will be loaded and applied.

This feature is useful when you want to make changes to the configuration file without having to restart the application. It allows you to change the log level, add or remove appenders or even change the layout pattern in runtime.

It's important to note that when the monitorInterval attribute is not set, the default value is 30 seconds.

You should also note that this feature has an overhead on performance, if you don't need to change the configuration in runtime you should set the monitorInterval to a higher value or even set it to -1 to disable this feature.

You can check the official documentation for more information and options: https://logging.apache.org/log4j/2.x/manual/configuration.html

<div class="me_quote">
What's the performance impact that Log can have in an application?
</div>

Logging can have a performance impact on an application, depending on the amount of logging, the logging level, and the logging implementation.

- Quantity of logs: The more logs you have, the greater the performance impact will be. This is because each log statement generates a log event that needs to be processed. High-frequency logging, such as logging every time an iteration of a loop is executed, can cause a significant performance impact.

- Log level: The logging level also affects performance. Logging at higher levels (such as debug or trace) generates more log events than logging at lower levels (such as warning or error). So, if you are logging at a high level, you may want to consider reducing the logging level to reduce the performance impact.

- Logging implementation: The logging implementation can also have an impact on performance. For example, synchronous logging, where each log statement blocks until the log event has been written, can slow down an application. Asynchronous logging, where log events are written in a separate thread, can help to reduce the performance impact.

- Buffering: Buffering log events can also help to improve performance by reducing the number of I/O operations required to write log events to disk.

- Memory-mapped files: Memory-mapped files can improve performance by allowing for fast and efficient file access, but it also has some limitations. For example, it's only supported on certain platforms and it can cause the JVM to run out of memory if the file size exceeds the available memory.

Overall, it's important to measure the performance impact of logging in your specific application and make adjustments as needed.

It's also recommended to use a logging library that provides advanced features such as log filtering, buffering and memory-mapped files, like Log4j2.



## Code Repository based on the ChatGPT help =)

[https://github.com/tivrfoa/java-log4j-chatgpt](https://github.com/tivrfoa/java-log4j-chatgpt)


```log
$ mvn clean compile exec:java -Dexec.mainClass="org.acme.Main"
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------------< org.acme:log4j2 >---------------------------
[INFO] Building log4j2 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ log4j2 ---
[INFO] Deleting /home/abc/java-logging-tests/chatgpt1/target
[INFO] 
[INFO] --- maven-resources-plugin:3.2.0:copy-resources (copy-resources) @ log4j2 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Using 'null' encoding to copy filtered properties files.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-resources-plugin:3.2.0:resources (default-resources) @ log4j2 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Using 'null' encoding to copy filtered properties files.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.0:compile (default-compile) @ log4j2 ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/abc/java-logging-tests/chatgpt1/target/classes
[INFO] 
[INFO] --- exec-maven-plugin:3.1.0:java (default-cli) @ log4j2 ---
2023-01-21 18:06:44,169 org.acme.Main.main() DEBUG Apache Log4j Core 2.13.3 initializing configuration XmlConfiguration[location=/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml]
2023-01-21 18:06:44,178 org.acme.Main.main() DEBUG Installed 0 script engines
2023-01-21 18:06:44,179 org.acme.Main.main() DEBUG PluginManager 'Core' found 122 plugins
2023-01-21 18:06:44,180 org.acme.Main.main() DEBUG PluginManager 'Level' found 0 plugins
2023-01-21 18:06:44,186 org.acme.Main.main() DEBUG PluginManager 'Lookup' found 16 plugins
2023-01-21 18:06:44,192 org.acme.Main.main() DEBUG Building Plugin[name=layout, class=org.apache.logging.log4j.core.layout.PatternLayout].
2023-01-21 18:06:44,210 org.acme.Main.main() TRACE TypeConverterRegistry initializing.
2023-01-21 18:06:44,211 org.acme.Main.main() DEBUG PluginManager 'TypeConverter' found 26 plugins
2023-01-21 18:06:44,227 org.acme.Main.main() DEBUG PatternLayout$Builder(pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n", PatternSelector=null, Configuration(/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml), Replace=null, charset="null", alwaysWriteExceptions="null", disableAnsi="null", noConsoleNoAnsi="null", header="null", footer="null")
2023-01-21 18:06:44,229 org.acme.Main.main() DEBUG PluginManager 'Converter' found 44 plugins
2023-01-21 18:06:44,242 org.acme.Main.main() DEBUG Building Plugin[name=appender, class=org.apache.logging.log4j.core.appender.FileAppender].
2023-01-21 18:06:44,256 org.acme.Main.main() DEBUG FileAppender$Builder(fileName="logs/application.log", append="null", locking="null", advertise="null", advertiseUri="null", createOnDemand="null", filePermissions="null", fileOwner="null", fileGroup="null", bufferedIo="null", bufferSize="null", immediateFlush="null", ignoreExceptions="null", PatternLayout(%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n), name="File", Configuration(/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml), Filter=null, ={})
2023-01-21 18:06:44,259 org.acme.Main.main() DEBUG Starting FileManager logs/application.log
2023-01-21 18:06:44,260 org.acme.Main.main() DEBUG Building Plugin[name=layout, class=org.apache.logging.log4j.core.layout.PatternLayout].
2023-01-21 18:06:44,261 org.acme.Main.main() DEBUG PatternLayout$Builder(pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n", PatternSelector=null, Configuration(/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml), Replace=null, charset="null", alwaysWriteExceptions="null", disableAnsi="null", noConsoleNoAnsi="null", header="null", footer="null")
2023-01-21 18:06:44,263 org.acme.Main.main() DEBUG Building Plugin[name=appender, class=org.apache.logging.log4j.core.appender.ConsoleAppender].
2023-01-21 18:06:44,267 org.acme.Main.main() DEBUG ConsoleAppender$Builder(target="SYSTEM_OUT", follow="null", direct="null", bufferedIo="null", bufferSize="null", immediateFlush="null", ignoreExceptions="null", PatternLayout(%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n), name="Console", Configuration(/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml), Filter=null, ={})
2023-01-21 18:06:44,271 org.acme.Main.main() DEBUG Starting OutputStreamManager SYSTEM_OUT.false.false
2023-01-21 18:06:44,272 org.acme.Main.main() DEBUG Building Plugin[name=appenders, class=org.apache.logging.log4j.core.config.AppendersPlugin].
2023-01-21 18:06:44,273 org.acme.Main.main() DEBUG createAppenders(={File, Console})
2023-01-21 18:06:44,274 org.acme.Main.main() DEBUG Building Plugin[name=AppenderRef, class=org.apache.logging.log4j.core.config.AppenderRef].
2023-01-21 18:06:44,279 org.acme.Main.main() DEBUG createAppenderRef(ref="File", level="null", Filter=null)
2023-01-21 18:06:44,280 org.acme.Main.main() DEBUG Building Plugin[name=AppenderRef, class=org.apache.logging.log4j.core.config.AppenderRef].
2023-01-21 18:06:44,281 org.acme.Main.main() DEBUG createAppenderRef(ref="Console", level="null", Filter=null)
2023-01-21 18:06:44,282 org.acme.Main.main() DEBUG Building Plugin[name=root, class=org.apache.logging.log4j.core.config.LoggerConfig$RootLogger].
2023-01-21 18:06:44,284 org.acme.Main.main() DEBUG createLogger(additivity="null", level="DEBUG", includeLocation="null", ={File, Console}, ={}, Configuration(/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml), Filter=null)
2023-01-21 18:06:44,287 org.acme.Main.main() DEBUG Building Plugin[name=loggers, class=org.apache.logging.log4j.core.config.LoggersPlugin].
2023-01-21 18:06:44,289 org.acme.Main.main() DEBUG createLoggers(={root})
2023-01-21 18:06:44,291 org.acme.Main.main() DEBUG Configuration XmlConfiguration[location=/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml] initialized
2023-01-21 18:06:44,291 org.acme.Main.main() DEBUG Starting configuration XmlConfiguration[location=/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml]
2023-01-21 18:06:44,292 org.acme.Main.main() DEBUG Started configuration XmlConfiguration[location=/home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml] OK.
2023-01-21 18:06:44,294 org.acme.Main.main() TRACE Stopping org.apache.logging.log4j.core.config.DefaultConfiguration@33cff3c5...
2023-01-21 18:06:44,295 org.acme.Main.main() TRACE DefaultConfiguration notified 1 ReliabilityStrategies that config will be stopped.
2023-01-21 18:06:44,295 org.acme.Main.main() TRACE DefaultConfiguration stopping root LoggerConfig.
2023-01-21 18:06:44,296 org.acme.Main.main() TRACE DefaultConfiguration notifying ReliabilityStrategies that appenders will be stopped.
2023-01-21 18:06:44,297 org.acme.Main.main() TRACE DefaultConfiguration stopping remaining Appenders.
2023-01-21 18:06:44,298 org.acme.Main.main() DEBUG Shutting down OutputStreamManager SYSTEM_OUT.false.false-1
2023-01-21 18:06:44,298 org.acme.Main.main() DEBUG OutputStream closed
2023-01-21 18:06:44,299 org.acme.Main.main() DEBUG Shut down OutputStreamManager SYSTEM_OUT.false.false-1, all resources released: true
2023-01-21 18:06:44,300 org.acme.Main.main() DEBUG Appender DefaultConsole-1 stopped with status true
2023-01-21 18:06:44,301 org.acme.Main.main() TRACE DefaultConfiguration stopped 1 remaining Appenders.
2023-01-21 18:06:44,301 org.acme.Main.main() TRACE DefaultConfiguration cleaning Appenders from 1 LoggerConfigs.
2023-01-21 18:06:44,302 org.acme.Main.main() DEBUG Stopped org.apache.logging.log4j.core.config.DefaultConfiguration@33cff3c5 OK
2023-01-21 18:06:44,429 org.acme.Main.main() TRACE Reregistering MBeans after reconfigure. Selector=org.apache.logging.log4j.core.selector.ClassLoaderContextSelector@47989685
2023-01-21 18:06:44,430 org.acme.Main.main() TRACE Reregistering context (1/1): '6e4c0d8c' org.apache.logging.log4j.core.LoggerContext@32d8dd63
2023-01-21 18:06:44,432 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c'
2023-01-21 18:06:44,434 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c,component=StatusLogger'
2023-01-21 18:06:44,435 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c,component=ContextSelector'
2023-01-21 18:06:44,437 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c,component=Loggers,name=*'
2023-01-21 18:06:44,438 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c,component=Appenders,name=*'
2023-01-21 18:06:44,439 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c,component=AsyncAppenders,name=*'
2023-01-21 18:06:44,440 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c,component=AsyncLoggerRingBuffer'
2023-01-21 18:06:44,441 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=6e4c0d8c,component=Loggers,name=*,subtype=RingBuffer'
2023-01-21 18:06:44,446 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=6e4c0d8c
2023-01-21 18:06:44,454 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=6e4c0d8c,component=StatusLogger
2023-01-21 18:06:44,457 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=6e4c0d8c,component=ContextSelector
2023-01-21 18:06:44,461 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=6e4c0d8c,component=Loggers,name=
2023-01-21 18:06:44,464 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=6e4c0d8c,component=Appenders,name=Console
2023-01-21 18:06:44,466 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=6e4c0d8c,component=Appenders,name=File
2023-01-21 18:06:44,471 org.acme.Main.main() TRACE Using default SystemClock for timestamps.
2023-01-21 18:06:44,472 org.acme.Main.main() DEBUG org.apache.logging.log4j.core.util.SystemClock supports precise timestamps.
2023-01-21 18:06:44,476 org.acme.Main.main() TRACE Using DummyNanoClock for nanosecond timestamps.
2023-01-21 18:06:44,477 org.acme.Main.main() DEBUG Reconfiguration complete for context[name=6e4c0d8c] at URI /home/abc/java-logging-tests/chatgpt1/target/classes/log4j2.xml (org.apache.logging.log4j.core.LoggerContext@32d8dd63) with optional ClassLoader: null
2023-01-21 18:06:44,478 org.acme.Main.main() DEBUG Shutdown hook enabled. Registering a new one.
2023-01-21 18:06:44,481 org.acme.Main.main() DEBUG LoggerContext[name=6e4c0d8c, org.apache.logging.log4j.core.LoggerContext@32d8dd63] started OK.
2023-01-21 18:06:44 DEBUG Main:10 - Starting doSomeWork method
2023-01-21 18:06:44 INFO  Main:12 - Work completed successfully
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total ti 3.443 s
[INFO] Finished at: 2023-01-21T18:06:44-03:00
[INFO] ------------------------------------------------------------------------
```

## Changing the log4j configuration file when running the application

```log
$ mvn exec:java -Dexec.mainClass="org.acme.Main"  -Dlog4j.configurationFile=file:./src/main/resources/log4j2-bufferedio.xml 
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------------< org.acme:log4j2 >---------------------------
[INFO] Building log4j2 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- exec-maven-plugin:3.1.0:java (default-cli) @ log4j2 ---
2023-01-21 18:29:04,663 org.acme.Main.main() DEBUG Apache Log4j Core 2.13.3 initializing configuration XmlConfiguration[location=file:./src/main/resources/log4j2-bufferedio.xml]
2023-01-21 18:29:04,674 org.acme.Main.main() DEBUG Installed 0 script engines
2023-01-21 18:29:04,676 org.acme.Main.main() DEBUG PluginManager 'Core' found 122 plugins
2023-01-21 18:29:04,676 org.acme.Main.main() DEBUG PluginManager 'Level' found 0 plugins
2023-01-21 18:29:04,685 org.acme.Main.main() DEBUG PluginManager 'Lookup' found 16 plugins
2023-01-21 18:29:04,692 org.acme.Main.main() DEBUG Building Plugin[name=layout, class=org.apache.logging.log4j.core.layout.PatternLayout].
2023-01-21 18:29:04,710 org.acme.Main.main() TRACE TypeConverterRegistry initializing.
2023-01-21 18:29:04,711 org.acme.Main.main() DEBUG PluginManager 'TypeConverter' found 26 plugins
2023-01-21 18:29:04,725 org.acme.Main.main() DEBUG PatternLayout$Builder(pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n", PatternSelector=null, Configuration(file:./src/main/resources/log4j2-bufferedio.xml), Replace=null, charset="null", alwaysWriteExceptions="null", disableAnsi="null", noConsoleNoAnsi="null", header="null", footer="null")
2023-01-21 18:29:04,726 org.acme.Main.main() DEBUG PluginManager 'Converter' found 44 plugins
2023-01-21 18:29:04,740 org.acme.Main.main() DEBUG Building Plugin[name=appender, class=org.apache.logging.log4j.core.appender.FileAppender].
2023-01-21 18:29:04,752 org.acme.Main.main() DEBUG FileAppender$Builder(fileName="logs/buffered.log", append="null", locking="null", advertise="null", advertiseUri="null", createOnDemand="null", filePermissions="null", fileOwner="null", fileGroup="null", bufferedIo="true", bufferSize="8192", immediateFlush="null", ignoreExceptions="null", PatternLayout(%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n), name="File", Configuration(file:./src/main/resources/log4j2-bufferedio.xml), Filter=null, ={})
2023-01-21 18:29:04,755 org.acme.Main.main() DEBUG Starting FileManager logs/buffered.log
2023-01-21 18:29:04,756 org.acme.Main.main() DEBUG Building Plugin[name=appenders, class=org.apache.logging.log4j.core.config.AppendersPlugin].
2023-01-21 18:29:04,757 org.acme.Main.main() DEBUG createAppenders(={File})
2023-01-21 18:29:04,759 org.acme.Main.main() DEBUG Building Plugin[name=AppenderRef, class=org.apache.logging.log4j.core.config.AppenderRef].
2023-01-21 18:29:04,765 org.acme.Main.main() DEBUG createAppenderRef(ref="File", level="null", Filter=null)
2023-01-21 18:29:04,766 org.acme.Main.main() DEBUG Building Plugin[name=root, class=org.apache.logging.log4j.core.config.LoggerConfig$RootLogger].
2023-01-21 18:29:04,767 org.acme.Main.main() DEBUG createLogger(additivity="null", level="DEBUG", includeLocation="null", ={File}, ={}, Configuration(file:./src/main/resources/log4j2-bufferedio.xml), Filter=null)
2023-01-21 18:29:04,770 org.acme.Main.main() DEBUG Building Plugin[name=loggers, class=org.apache.logging.log4j.core.config.LoggersPlugin].
2023-01-21 18:29:04,772 org.acme.Main.main() DEBUG createLoggers(={root})
2023-01-21 18:29:04,774 org.acme.Main.main() DEBUG Configuration XmlConfiguration[location=file:./src/main/resources/log4j2-bufferedio.xml] initialized
2023-01-21 18:29:04,775 org.acme.Main.main() DEBUG Starting configuration XmlConfiguration[location=file:./src/main/resources/log4j2-bufferedio.xml]
2023-01-21 18:29:04,775 org.acme.Main.main() DEBUG Started configuration XmlConfiguration[location=file:./src/main/resources/log4j2-bufferedio.xml] OK.
2023-01-21 18:29:04,776 org.acme.Main.main() TRACE Stopping org.apache.logging.log4j.core.config.DefaultConfiguration@2475e5a3...
2023-01-21 18:29:04,777 org.acme.Main.main() TRACE DefaultConfiguration notified 1 ReliabilityStrategies that config will be stopped.
2023-01-21 18:29:04,777 org.acme.Main.main() TRACE DefaultConfiguration stopping root LoggerConfig.
2023-01-21 18:29:04,778 org.acme.Main.main() TRACE DefaultConfiguration notifying ReliabilityStrategies that appenders will be stopped.
2023-01-21 18:29:04,778 org.acme.Main.main() TRACE DefaultConfiguration stopping remaining Appenders.
2023-01-21 18:29:04,779 org.acme.Main.main() DEBUG Shutting down OutputStreamManager SYSTEM_OUT.false.false-1
2023-01-21 18:29:04,779 org.acme.Main.main() DEBUG OutputStream closed
2023-01-21 18:29:04,780 org.acme.Main.main() DEBUG Shut down OutputStreamManager SYSTEM_OUT.false.false-1, all resources released: true
2023-01-21 18:29:04,780 org.acme.Main.main() DEBUG Appender DefaultConsole-1 stopped with status true
2023-01-21 18:29:04,781 org.acme.Main.main() TRACE DefaultConfiguration stopped 1 remaining Appenders.
2023-01-21 18:29:04,781 org.acme.Main.main() TRACE DefaultConfiguration cleaning Appenders from 1 LoggerConfigs.
2023-01-21 18:29:04,782 org.acme.Main.main() DEBUG Stopped org.apache.logging.log4j.core.config.DefaultConfiguration@2475e5a3 OK
2023-01-21 18:29:04,923 org.acme.Main.main() TRACE Reregistering MBeans after reconfigure. Selector=org.apache.logging.log4j.core.selector.ClassLoaderContextSelector@22d58199
2023-01-21 18:29:04,924 org.acme.Main.main() TRACE Reregistering context (1/1): '68e62ca4' org.apache.logging.log4j.core.LoggerContext@33136fc8
2023-01-21 18:29:04,925 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4'
2023-01-21 18:29:04,926 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=StatusLogger'
2023-01-21 18:29:04,927 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=ContextSelector'
2023-01-21 18:29:04,928 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=*'
2023-01-21 18:29:04,929 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=Appenders,name=*'
2023-01-21 18:29:04,929 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=AsyncAppenders,name=*'
2023-01-21 18:29:04,930 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=AsyncLoggerRingBuffer'
2023-01-21 18:29:04,930 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=*,subtype=RingBuffer'
2023-01-21 18:29:04,933 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4
2023-01-21 18:29:04,938 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=StatusLogger
2023-01-21 18:29:04,940 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=ContextSelector
2023-01-21 18:29:04,943 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=
2023-01-21 18:29:04,945 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=Appenders,name=File
2023-01-21 18:29:04,949 org.acme.Main.main() TRACE Using default SystemClock for timestamps.
2023-01-21 18:29:04,949 org.acme.Main.main() DEBUG org.apache.logging.log4j.core.util.SystemClock supports precise timestamps.
2023-01-21 18:29:04,952 org.acme.Main.main() TRACE Using DummyNanoClock for nanosecond timestamps.
2023-01-21 18:29:04,953 org.acme.Main.main() DEBUG Reconfiguration complete for context[name=68e62ca4] at URI file:./src/main/resources/log4j2-bufferedio.xml (org.apache.logging.log4j.core.LoggerContext@33136fc8) with optional ClassLoader: null
2023-01-21 18:29:04,954 org.acme.Main.main() DEBUG Shutdown hook enabled. Registering a new one.
2023-01-21 18:29:04,956 org.acme.Main.main() DEBUG LoggerContext[name=68e62ca4, org.apache.logging.log4j.core.LoggerContext@33136fc8] started OK.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total ti 1.492 s
[INFO] Finished at: 2023-01-21T18:29:04-03:00
[INFO] ------------------------------------------------------------------------
```

## Using RandomAccessFileAppender

```log
$ mvn exec:java -Dexec.mainClass="org.acme.Main"  -Dlog4j.configurationFile=file:./src/main/resources/log4j2-RandomAccessFileAppender.xml 
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------------< org.acme:log4j2 >---------------------------
[INFO] Building log4j2 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- exec-maven-plugin:3.1.0:java (default-cli) @ log4j2 ---
2023-01-21 20:10:39,270 org.acme.Main.main() DEBUG Apache Log4j Core 2.13.3 initializing configuration XmlConfiguration[location=file:./src/main/resources/log4j2-RandomAccessFileAppender.xml]
2023-01-21 20:10:39,279 org.acme.Main.main() DEBUG Installed 0 script engines
2023-01-21 20:10:39,280 org.acme.Main.main() DEBUG PluginManager 'Core' found 122 plugins
2023-01-21 20:10:39,280 org.acme.Main.main() DEBUG PluginManager 'Level' found 0 plugins
2023-01-21 20:10:39,285 org.acme.Main.main() DEBUG PluginManager 'Lookup' found 16 plugins
2023-01-21 20:10:39,289 org.acme.Main.main() DEBUG Building Plugin[name=layout, class=org.apache.logging.log4j.core.layout.PatternLayout].
2023-01-21 20:10:39,298 org.acme.Main.main() TRACE TypeConverterRegistry initializing.
2023-01-21 20:10:39,298 org.acme.Main.main() DEBUG PluginManager 'TypeConverter' found 26 plugins
2023-01-21 20:10:39,308 org.acme.Main.main() DEBUG PatternLayout$Builder(pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n", PatternSelector=null, Configuration(file:./src/main/resources/log4j2-RandomAccessFileAppender.xml), Replace=null, charset="null", alwaysWriteExceptions="null", disableAnsi="null", noConsoleNoAnsi="null", header="null", footer="null")
2023-01-21 20:10:39,308 org.acme.Main.main() DEBUG PluginManager 'Converter' found 44 plugins
2023-01-21 20:10:39,316 org.acme.Main.main() DEBUG Building Plugin[name=appender, class=org.apache.logging.log4j.core.appender.RandomAccessFileAppender].
2023-01-21 20:10:39,323 org.acme.Main.main() DEBUG RandomAccessFileAppender$Builder(fileName="logs/random-access-file-appender.log", append="null", advertise="null", advertiseURI="null", bufferedIo="null", bufferSize="8192", immediateFlush="false", ignoreExceptions="null", PatternLayout(%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n), name="MyFile", Configuration(file:./src/main/resources/log4j2-RandomAccessFileAppender.xml), Filter=null, ={})
2023-01-21 20:10:39,325 org.acme.Main.main() DEBUG Starting RandomAccessFileManager logs/random-access-file-appender.log
2023-01-21 20:10:39,326 org.acme.Main.main() DEBUG Building Plugin[name=appenders, class=org.apache.logging.log4j.core.config.AppendersPlugin].
2023-01-21 20:10:39,326 org.acme.Main.main() DEBUG createAppenders(={MyFile})
2023-01-21 20:10:39,327 org.acme.Main.main() DEBUG Building Plugin[name=AppenderRef, class=org.apache.logging.log4j.core.config.AppenderRef].
2023-01-21 20:10:39,330 org.acme.Main.main() DEBUG createAppenderRef(ref="MyFile", level="null", Filter=null)
2023-01-21 20:10:39,330 org.acme.Main.main() DEBUG Building Plugin[name=root, class=org.apache.logging.log4j.core.config.LoggerConfig$RootLogger].
2023-01-21 20:10:39,331 org.acme.Main.main() DEBUG createLogger(additivity="null", level="DEBUG", includeLocation="null", ={MyFile}, ={}, Configuration(file:./src/main/resources/log4j2-RandomAccessFileAppender.xml), Filter=null)
2023-01-21 20:10:39,333 org.acme.Main.main() DEBUG Building Plugin[name=loggers, class=org.apache.logging.log4j.core.config.LoggersPlugin].
2023-01-21 20:10:39,334 org.acme.Main.main() DEBUG createLoggers(={root})
2023-01-21 20:10:39,335 org.acme.Main.main() DEBUG Configuration XmlConfiguration[location=file:./src/main/resources/log4j2-RandomAccessFileAppender.xml] initialized
2023-01-21 20:10:39,336 org.acme.Main.main() DEBUG Starting configuration XmlConfiguration[location=file:./src/main/resources/log4j2-RandomAccessFileAppender.xml]
2023-01-21 20:10:39,336 org.acme.Main.main() DEBUG Log4j2 ConfigurationScheduler starting 1 threads
2023-01-21 20:10:39,339 org.acme.Main.main() DEBUG Started configuration XmlConfiguration[location=file:./src/main/resources/log4j2-RandomAccessFileAppender.xml] OK.
2023-01-21 20:10:39,340 org.acme.Main.main() TRACE Stopping org.apache.logging.log4j.core.config.DefaultConfiguration@3329e482...
2023-01-21 20:10:39,341 org.acme.Main.main() TRACE DefaultConfiguration notified 1 ReliabilityStrategies that config will be stopped.
2023-01-21 20:10:39,341 org.acme.Main.main() TRACE DefaultConfiguration stopping root LoggerConfig.
2023-01-21 20:10:39,341 org.acme.Main.main() TRACE DefaultConfiguration notifying ReliabilityStrategies that appenders will be stopped.
2023-01-21 20:10:39,341 org.acme.Main.main() TRACE DefaultConfiguration stopping remaining Appenders.
2023-01-21 20:10:39,342 org.acme.Main.main() DEBUG Shutting down OutputStreamManager SYSTEM_OUT.false.false-1
2023-01-21 20:10:39,342 org.acme.Main.main() DEBUG OutputStream closed
2023-01-21 20:10:39,342 org.acme.Main.main() DEBUG Shut down OutputStreamManager SYSTEM_OUT.false.false-1, all resources released: true
2023-01-21 20:10:39,342 org.acme.Main.main() DEBUG Appender DefaultConsole-1 stopped with status true
2023-01-21 20:10:39,343 org.acme.Main.main() TRACE DefaultConfiguration stopped 1 remaining Appenders.
2023-01-21 20:10:39,343 org.acme.Main.main() TRACE DefaultConfiguration cleaning Appenders from 1 LoggerConfigs.
2023-01-21 20:10:39,343 org.acme.Main.main() DEBUG Stopped org.apache.logging.log4j.core.config.DefaultConfiguration@3329e482 OK
2023-01-21 20:10:39,421 org.acme.Main.main() TRACE Reregistering MBeans after reconfigure. Selector=org.apache.logging.log4j.core.selector.ClassLoaderContextSelector@3992e778
2023-01-21 20:10:39,422 org.acme.Main.main() TRACE Reregistering context (1/1): '68e62ca4' org.apache.logging.log4j.core.LoggerContext@51ef9023
2023-01-21 20:10:39,423 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4'
2023-01-21 20:10:39,423 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=StatusLogger'
2023-01-21 20:10:39,423 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=ContextSelector'
2023-01-21 20:10:39,424 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=*'
2023-01-21 20:10:39,425 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=Appenders,name=*'
2023-01-21 20:10:39,425 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=AsyncAppenders,name=*'
2023-01-21 20:10:39,426 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=AsyncLoggerRingBuffer'
2023-01-21 20:10:39,427 org.acme.Main.main() TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=*,subtype=RingBuffer'
2023-01-21 20:10:39,430 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4
2023-01-21 20:10:39,434 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=StatusLogger
2023-01-21 20:10:39,435 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=ContextSelector
2023-01-21 20:10:39,437 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=
2023-01-21 20:10:39,438 org.acme.Main.main() DEBUG Registering MBean org.apache.logging.log4j2:type=68e62ca4,component=Appenders,name=MyFile
2023-01-21 20:10:39,440 org.acme.Main.main() TRACE Using default SystemClock for timestamps.
2023-01-21 20:10:39,440 org.acme.Main.main() DEBUG org.apache.logging.log4j.core.util.SystemClock supports precise timestamps.
2023-01-21 20:10:39,442 org.acme.Main.main() TRACE Using DummyNanoClock for nanosecond timestamps.
2023-01-21 20:10:39,442 org.acme.Main.main() DEBUG Reconfiguration complete for context[name=68e62ca4] at URI file:./src/main/resources/log4j2-RandomAccessFileAppender.xml (org.apache.logging.log4j.core.LoggerContext@51ef9023) with optional ClassLoader: null
2023-01-21 20:10:39,442 org.acme.Main.main() DEBUG Shutdown hook enabled. Registering a new one.
2023-01-21 20:10:39,443 org.acme.Main.main() DEBUG LoggerContext[name=68e62ca4, org.apache.logging.log4j.core.LoggerContext@51ef9023] started OK.
[WARNING] thread Thread[Log4j2-TF-1-Scheduled-1,5,org.acme.Main] was interrupted but is still alive after waiting at least 15000msecs
[WARNING] thread Thread[Log4j2-TF-1-Scheduled-1,5,org.acme.Main] will linger despite being asked to die via interruption
[WARNING] NOTE: 1 thread(s) did not finish despite being asked to via interruption. This is not a problem with exec:java, it is a problem with the running code. Although not serious, it should be remedied.
[WARNING] Couldn't destroy threadgroup org.codehaus.mojo.exec.ExecJavaMojo$IsolatedThreadGroup[name=org.acme.Main,maxpri=10]
java.lang.IllegalThreadStateException
    at java.lang.ThreadGroup.destroy (ThreadGroup.java:803)
    at org.codehaus.mojo.exec.ExecJavaMojo.execute (ExecJavaMojo.java:319)
    at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo (DefaultBuildPluginManager.java:137)
    at org.apache.maven.lifecycle.internal.MojoExecutor.doExecute2 (MojoExecutor.java:370)
    at org.apache.maven.lifecycle.internal.MojoExecutor.doExecute (MojoExecutor.java:351)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:215)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:171)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:163)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:117)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:81)
    at org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder.build (SingleThreadedBuilder.java:56)
    at org.apache.maven.lifecycle.internal.LifecycleStarter.execute (LifecycleStarter.java:128)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:294)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:192)
    at org.apache.maven.DefaultMaven.execute (DefaultMaven.java:105)
    at org.apache.maven.cli.MavenCli.execute (MavenCli.java:960)
    at org.apache.maven.cli.MavenCli.doMain (MavenCli.java:293)
    at org.apache.maven.cli.MavenCli.main (MavenCli.java:196)
    at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0 (Native Method)
    at jdk.internal.reflect.NativeMethodAccessorImpl.invoke (NativeMethodAccessorImpl.java:77)
    at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke (DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke (Method.java:568)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced (Launcher.java:282)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launch (Launcher.java:225)
    at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode (Launcher.java:406)
    at org.codehaus.plexus.classworlds.launcher.Launcher.main (Launcher.java:347)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total ti 15.927 s
[INFO] Finished at: 2023-01-21T20:10:54-03:00
[INFO] ------------------------------------------------------------------------
2023-01-21 20:10:54,465 pool-1-thread-1 DEBUG Stopping LoggerContext[name=68e62ca4, org.apache.logging.log4j.core.LoggerContext@51ef9023]
2023-01-21 20:10:54,466 pool-1-thread-1 DEBUG Stopping LoggerContext[name=68e62ca4, org.apache.logging.log4j.core.LoggerContext@51ef9023]...
2023-01-21 20:10:54,466 pool-1-thread-1 TRACE Unregistering 1 MBeans: [org.apache.logging.log4j2:type=68e62ca4]
2023-01-21 20:10:54,467 pool-1-thread-1 TRACE Unregistering 1 MBeans: [org.apache.logging.log4j2:type=68e62ca4,component=StatusLogger]
2023-01-21 20:10:54,467 pool-1-thread-1 TRACE Unregistering 1 MBeans: [org.apache.logging.log4j2:type=68e62ca4,component=ContextSelector]
2023-01-21 20:10:54,467 pool-1-thread-1 TRACE Unregistering 1 MBeans: [org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=]
2023-01-21 20:10:54,468 pool-1-thread-1 TRACE Unregistering 1 MBeans: [org.apache.logging.log4j2:type=68e62ca4,component=Appenders,name=MyFile]
2023-01-21 20:10:54,468 pool-1-thread-1 TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=AsyncAppenders,name=*'
2023-01-21 20:10:54,468 pool-1-thread-1 TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=AsyncLoggerRingBuffer'
2023-01-21 20:10:54,468 pool-1-thread-1 TRACE Unregistering but no MBeans found matching 'org.apache.logging.log4j2:type=68e62ca4,component=Loggers,name=*,subtype=RingBuffer'
2023-01-21 20:10:54,469 pool-1-thread-1 TRACE Stopping XmlConfiguration[location=file:./src/main/resources/log4j2-RandomAccessFileAppender.xml]...
2023-01-21 20:10:54,470 pool-1-thread-1 TRACE XmlConfiguration notified 2 ReliabilityStrategies that config will be stopped.
2023-01-21 20:10:54,470 pool-1-thread-1 TRACE XmlConfiguration stopping 1 LoggerConfigs.
2023-01-21 20:10:54,471 pool-1-thread-1 TRACE XmlConfiguration stopping root LoggerConfig.
2023-01-21 20:10:54,471 pool-1-thread-1 TRACE XmlConfiguration notifying ReliabilityStrategies that appenders will be stopped.
2023-01-21 20:10:54,472 pool-1-thread-1 TRACE XmlConfiguration stopping remaining Appenders.
2023-01-21 20:10:54,473 pool-1-thread-1 DEBUG Shutting down RandomAccessFileManager logs/random-access-file-appender.log
2023-01-21 20:10:54,474 pool-1-thread-1 DEBUG Shut down RandomAccessFileManager logs/random-access-file-appender.log, all resources released: true
2023-01-21 20:10:54,474 pool-1-thread-1 DEBUG Appender MyFile stopped with status true
2023-01-21 20:10:54,475 pool-1-thread-1 TRACE XmlConfiguration stopped 1 remaining Appenders.
2023-01-21 20:10:54,475 pool-1-thread-1 TRACE XmlConfiguration cleaning Appenders from 2 LoggerConfigs.
2023-01-21 20:10:54,475 pool-1-thread-1 DEBUG Log4j2 ConfigurationScheduler shutting down threads in java.util.concurrent.ScheduledThreadPoolExecutor@40087898[Running, pool size = 1, active threads = 0, queued tasks = 1, completed tasks = 0]
2023-01-21 20:10:54,476 pool-1-thread-1 DEBUG Stopped XmlConfiguration[location=file:./src/main/resources/log4j2-RandomAccessFileAppender.xml] OK
2023-01-21 20:10:54,476 pool-1-thread-1 DEBUG Stopped LoggerContext[name=68e62ca4, org.apache.logging.log4j.core.LoggerContext@51ef9023] with status true
```

## Related Links

[https://logging.apache.org/log4j/2.x/manual/configuration.html](https://logging.apache.org/log4j/2.x/manual/configuration.html)

[Log4j MemoryMappedFile and RollingRandomAccessFile](https://stackoverflow.com/a/34902543/339561)

[How do I set log4j level on the command line?](https://stackoverflow.com/questions/7126709/how-do-i-set-log4j-level-on-the-command-line)