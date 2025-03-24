# LoggerPrint Migration Specification: LotusScript to Java with Logback

## Overview

This document provides detailed specifications for migrating the `LoggerPrint` class from LotusScript to Java using the Logback logging framework. This migration is part of the larger effort to reengineer a Lotus Notes/Domino application to an Angular and Java Spring application.

## Original LotusScript Implementation

The original `LoggerPrint` class in LotusScript is defined in `libLoggerPrint.lss` and has the following characteristics:

- Extends the `Logger` class
- Provides methods for different log levels: `assert`, `error`, `warn`, `info`, `debug`, and `verbose`
- Formats log messages with timestamps and module information
- Outputs logs to the status bar (or to "log.nsf" if running on a server)
- Can be configured to log to a file using Notes.ini variables

```lss
Class LoggerPrint As Logger
    Sub assert(msg As String, id As DynamicArguments)
        Call me.output({[ASSERT]     } & me.getModuleName(id) & msg)
    End Sub
    
    Sub Error(msg As String, id As DynamicArguments)
        Call me.output({[ERROR]      } & me.getModuleName(id) & msg)
    End Sub
    
    Sub warn(msg As String, id As DynamicArguments)
        Call me.output({[WARN]        } & me.getModuleName(id) & msg)
    End Sub
    
    Sub info(msg As String, id As DynamicArguments)
        Call me.output({[INFO]           } & me.getModuleName(id) & msg)
    End Sub
    
    Sub debug(msg As String, id As DynamicArguments)
        Call me.output({[DEBUG]      } & me.getModuleName(id) & msg)
    End Sub
    
    Sub verbose(msg As String, id As DynamicArguments)
        Call me.output({[VERBOSE] } & me.getModuleName(id) & msg)
    End Sub
    
    Private Sub Output(msg As String)
        Print CStr(Now()) & { } & msg
    End Sub
    
    Private Function getModuleName(id As DynamicArguments) As String
        getModuleName = {[} & id.toString({->}) & {] }
    End Function
End Class
```

## Java Implementation with Logback

### Dependencies

Add the following dependencies to your Maven `pom.xml` or Gradle build file:

```xml
<!-- For Maven -->
<dependencies>
    <!-- Logback Core -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <version>1.4.11</version>
    </dependency>
    
    <!-- Logback Classic (implements SLF4J) -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>
    
    <!-- SLF4J API -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>
</dependencies>
```

### Logback Configuration

Create a `logback.xml` file in the `src/main/resources` directory:

```xml
<configuration>
    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%level] [%logger{36}] %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/application.log</file>
        <append>true</append>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%level] [%logger{36}] %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- Root Logger -->
    <root level="WARN">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>
    
    <!-- Application Loggers -->
    <logger name="com.yourcompany.application" level="INFO" />
</configuration>
```

### DynamicArguments Class

First, we need to implement a Java equivalent of the `DynamicArguments` class:

```java
package com.yourcompany.logging;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

/**
 * Java implementation of the LotusScript DynamicArguments class.
 * Used to build context information for logging.
 */
public class DynamicArguments {
    private final List<Object> args = new ArrayList<>();
    
    /**
     * Adds a value to the arguments list.
     * 
     * @param value The value to add
     * @return This instance for method chaining
     */
    public DynamicArguments in(Object value) {
        args.add(value);
        return this;
    }
    
    /**
     * Returns the number of arguments.
     * 
     * @return The count of arguments
     */
    public int count() {
        return args.size();
    }
    
    /**
     * Converts the arguments to a string with the specified separator.
     * 
     * @param separator The separator to use between arguments
     * @return A string representation of the arguments
     */
    public String toString(String separator) {
        return args.stream()
                .map(Object::toString)
                .collect(Collectors.joining(separator));
    }
    
    /**
     * Converts the arguments to an array.
     * 
     * @return An array of the arguments
     */
    public Object[] toArray() {
        return args.toArray();
    }
    
    /**
     * Factory method to create a new DynamicArguments instance.
     * 
     * @return A new DynamicArguments instance
     */
    public static DynamicArguments args() {
        return new DynamicArguments();
    }
}
```

### LoggerPrint Implementation

Now, let's implement the `LoggerPrint` class using Logback:

```java
package com.yourcompany.logging;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * Java implementation of the LotusScript LoggerPrint class using Logback.
 * This class provides logging functionality similar to the original LotusScript implementation.
 */
public class LoggerPrint {
    private final Logger logger;
    private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    
    /**
     * Constructor that creates a logger with the specified name.
     * 
     * @param loggerName The name of the logger
     */
    public LoggerPrint(String loggerName) {
        this.logger = LoggerFactory.getLogger(loggerName);
    }
    
    /**
     * Constructor that creates a logger for the specified class.
     * 
     * @param clazz The class to create the logger for
     */
    public LoggerPrint(Class<?> clazz) {
        this.logger = LoggerFactory.getLogger(clazz);
    }
    
    /**
     * Logs an assertion message.
     * 
     * @param msg The message to log
     * @param id The context information
     */
    public void assertLog(String msg, DynamicArguments id) {
        try {
            MDC.put("context", getModuleName(id));
            logger.error("[ASSERT] {}", msg);
        } finally {
            MDC.remove("context");
        }
    }
    
    /**
     * Logs an error message.
     * 
     * @param msg The message to log
     * @param id The context information
     */
    public void error(String msg, DynamicArguments id) {
        try {
            MDC.put("context", getModuleName(id));
            logger.error("[ERROR] {}", msg);
        } finally {
            MDC.remove("context");
        }
    }
    
    /**
     * Logs a warning message.
     * 
     * @param msg The message to log
     * @param id The context information
     */
    public void warn(String msg, DynamicArguments id) {
        try {
            MDC.put("context", getModuleName(id));
            logger.warn("[WARN] {}", msg);
        } finally {
            MDC.remove("context");
        }
    }
    
    /**
     * Logs an info message.
     * 
     * @param msg The message to log
     * @param id The context information
     */
    public void info(String msg, DynamicArguments id) {
        try {
            MDC.put("context", getModuleName(id));
            logger.info("[INFO] {}", msg);
        } finally {
            MDC.remove("context");
        }
    }
    
    /**
     * Logs a debug message.
     * 
     * @param msg The message to log
     * @param id The context information
     */
    public void debug(String msg, DynamicArguments id) {
        try {
            MDC.put("context", getModuleName(id));
            logger.debug("[DEBUG] {}", msg);
        } finally {
            MDC.remove("context");
        }
    }
    
    /**
     * Logs a verbose message (maps to TRACE level in SLF4J).
     * 
     * @param msg The message to log
     * @param id The context information
     */
    public void verbose(String msg, DynamicArguments id) {
        try {
            MDC.put("context", getModuleName(id));
            logger.trace("[VERBOSE] {}", msg);
        } finally {
            MDC.remove("context");
        }
    }
    
    /**
     * Gets the module name from the dynamic arguments.
     * 
     * @param id The dynamic arguments
     * @return The formatted module name
     */
    private String getModuleName(DynamicArguments id) {
        return "[" + id.toString("->") + "] ";
    }
    
    /**
     * Factory method to create a LoggerPrint instance for the specified class.
     * 
     * @param clazz The class to create the logger for
     * @return A new LoggerPrint instance
     */
    public static LoggerPrint getLogger(Class<?> clazz) {
        return new LoggerPrint(clazz);
    }
    
    /**
     * Factory method to create a LoggerPrint instance with the specified name.
     * 
     * @param name The name of the logger
     * @return A new LoggerPrint instance
     */
    public static LoggerPrint getLogger(String name) {
        return new LoggerPrint(name);
    }
}
```

### LogLevel Management

To manage log levels programmatically (equivalent to `setLogLevel` in LotusScript):

```java
package com.yourcompany.logging;

import ch.qos.logback.classic.Level;
import ch.qos.logback.classic.Logger;
import org.slf4j.LoggerFactory;

/**
 * Utility class for managing log levels.
 */
public class LogLevelManager {
    // Constants for log levels (matching the original LotusScript constants)
    public static final int LL_NONE = 0;    // OFF in Logback
    public static final int LL_ASSERT = 1;  // ERROR in Logback (for assertions)
    public static final int LL_ERROR = 2;   // ERROR in Logback
    public static final int LL_WARN = 3;    // WARN in Logback
    public static final int LL_INFO = 4;    // INFO in Logback
    public static final int LL_DEBUG = 5;   // DEBUG in Logback
    public static final int LL_VERBOSE = 6; // TRACE in Logback
    public static final int LL_ALL = 255;   // ALL in Logback
    
    /**
     * Sets the log level for the specified logger.
     * 
     * @param loggerName The name of the logger
     * @param level The log level to set
     */
    public static void setLogLevel(String loggerName, int level) {
        Logger logger = (Logger) LoggerFactory.getLogger(loggerName);
        logger.setLevel(convertToLogbackLevel(level));
    }
    
    /**
     * Sets the log level for all loggers.
     * 
     * @param level The log level to set
     */
    public static void setRootLogLevel(int level) {
        Logger rootLogger = (Logger) LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);
        rootLogger.setLevel(convertToLogbackLevel(level));
    }
    
    /**
     * Converts a numeric log level to a Logback Level.
     * 
     * @param level The numeric log level
     * @return The corresponding Logback Level
     */
    private static Level convertToLogbackLevel(int level) {
        switch (level) {
            case LL_NONE:
                return Level.OFF;
            case LL_ASSERT:
            case LL_ERROR:
                return Level.ERROR;
            case LL_WARN:
                return Level.WARN;
            case LL_INFO:
                return Level.INFO;
            case LL_DEBUG:
                return Level.DEBUG;
            case LL_VERBOSE:
                return Level.TRACE;
            case LL_ALL:
                return Level.ALL;
            default:
                return Level.INFO; // Default level
        }
    }
}
```

### Spring Integration

To integrate with Spring Framework:

```java
package com.yourcompany.config;

import com.yourcompany.logging.LoggerPrint;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Spring configuration for LoggerPrint.
 */
@Configuration
public class LoggingConfig {
    
    /**
     * Creates a LoggerPrint bean for the application.
     * 
     * @return A LoggerPrint instance
     */
    @Bean
    public LoggerPrint applicationLogger() {
        return LoggerPrint.getLogger("com.yourcompany.application");
    }
    
    /**
     * Example of creating a logger for a specific component.
     * 
     * @return A LoggerPrint instance for the user service
     */
    @Bean
    public LoggerPrint userServiceLogger() {
        return LoggerPrint.getLogger("com.yourcompany.service.UserService");
    }
}
```

## Usage Examples

### Basic Usage

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;

import static com.yourcompany.logging.DynamicArguments.args;

public class ExampleService {
    private final LoggerPrint logger = LoggerPrint.getLogger(ExampleService.class);
    
    public void doSomething() {
        // Create context information
        DynamicArguments context = args().in("ExampleService").in("doSomething");
        
        // Log messages at different levels
        logger.info("Starting operation", context);
        
        try {
            // Perform some operation
            logger.debug("Operation details", context);
        } catch (Exception e) {
            logger.error("Operation failed: " + e.getMessage(), context);
        }
        
        logger.info("Operation completed", context);
    }
}
```

### Using with Spring Dependency Injection

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import org.springframework.stereotype.Service;

import static com.yourcompany.logging.DynamicArguments.args;

@Service
public class UserService {
    private final LoggerPrint logger;
    
    // Constructor injection
    public UserService(LoggerPrint userServiceLogger) {
        this.logger = userServiceLogger;
    }
    
    public void createUser(String username) {
        DynamicArguments context = args().in("UserService").in("createUser");
        logger.info("Creating user: " + username, context);
        
        // User creation logic
        
        logger.info("User created successfully", context);
    }
}
```

### Setting Log Levels Programmatically

```java
import com.yourcompany.logging.LogLevelManager;

public class LoggingExample {
    public static void main(String[] args) {
        // Set log level for all loggers
        LogLevelManager.setRootLogLevel(LogLevelManager.LL_WARN);
        
        // Set log level for specific loggers
        LogLevelManager.setLogLevel("com.yourcompany.service", LogLevelManager.LL_DEBUG);
        LogLevelManager.setLogLevel("com.yourcompany.repository", LogLevelManager.LL_INFO);
    }
}
```

## Testing

Here's how to test the LoggerPrint implementation:

```java
import ch.qos.logback.classic.Logger;
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.read.ListAppender;
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.slf4j.LoggerFactory;

import static com.yourcompany.logging.DynamicArguments.args;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class LoggerPrintTest {
    private LoggerPrint loggerPrint;
    private ListAppender<ILoggingEvent> listAppender;
    
    @BeforeEach
    public void setUp() {
        // Get the underlying Logback logger
        Logger logger = (Logger) LoggerFactory.getLogger("test");
        
        // Create and start a ListAppender
        listAppender = new ListAppender<>();
        listAppender.start();
        
        // Add the appender to the logger
        logger.addAppender(listAppender);
        
        // Create the LoggerPrint instance
        loggerPrint = new LoggerPrint("test");
    }
    
    @Test
    public void testInfoLogging() {
        // Create context
        DynamicArguments context = args().in("TestClass").in("testMethod");
        
        // Log a message
        loggerPrint.info("Test message", context);
        
        // Verify the log
        assertEquals(1, listAppender.list.size());
        ILoggingEvent event = listAppender.list.get(0);
        assertEquals("INFO", event.getLevel().toString());
        assertTrue(event.getFormattedMessage().contains("[INFO] Test message"));
    }
    
    // Add more tests for other log levels and functionality
}
```

## Migration Notes

1. **Log Levels**: The original LotusScript implementation uses custom log levels. In Logback, we map these to standard SLF4J levels:
   - `assert` → ERROR
   - `error` → ERROR
   - `warn` → WARN
   - `info` → INFO
   - `debug` → DEBUG
   - `verbose` → TRACE

2. **Context Information**: The original implementation uses `DynamicArguments` for context. We maintain this pattern but use SLF4J's MDC (Mapped Diagnostic Context) for thread-safe context management.

3. **Configuration**: Instead of Notes.ini variables, Logback uses XML or Groovy configuration files. The provided `logback.xml` includes both console and file appenders.

4. **Method Names**: The Java implementation uses the same method names as the original, except for `assert` which is renamed to `assertLog` to avoid conflict with Java's `assert` keyword.

5. **Spring Integration**: The Java implementation includes Spring integration for easy dependency injection in a Spring application.

## Conclusion

This migration specification provides a comprehensive guide for implementing the `LoggerPrint` class in Java using the Logback framework. The implementation maintains the functionality of the original LotusScript class while leveraging Java and Logback features for improved logging capabilities.

The junior Java developer should follow this specification to create the Java implementation, adapting it as needed to fit the specific requirements of the application being reengineered.