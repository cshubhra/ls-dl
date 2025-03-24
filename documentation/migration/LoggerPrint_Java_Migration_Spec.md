# LoggerPrint Java Migration Specification

## Overview

This document provides detailed specifications for migrating the Lotus Notes `libLoggerPrint.lss` class to a Java implementation. The `LoggerPrint` class is part of the LSDL (LotusScript Developer Layer) framework and is responsible for logging messages to the status bar or log files.

## Original LotusScript Implementation

The `LoggerPrint` class in LotusScript:
- Extends the `Logger` class
- Redirects all logging to the status bar (or to "log.nsf" if running on a server)
- Can be configured to log to a file using Notes.ini variables
- Formats log messages with timestamps and log level indicators

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

## Java Implementation Requirements

### 1. Class Structure

Create a Java class called `LoggerPrint` that implements a `Logger` interface:

```java
public class LoggerPrint implements Logger {
    // Implementation details will follow
}
```

### 2. Logger Interface

Create a Java interface called `Logger` that defines the logging methods:

```java
public interface Logger {
    void assert_(String msg, DynamicArguments id);
    void error(String msg, DynamicArguments id);
    void warn(String msg, DynamicArguments id);
    void info(String msg, DynamicArguments id);
    void debug(String msg, DynamicArguments id);
    void verbose(String msg, DynamicArguments id);
}
```

Note: Since `assert` is a reserved keyword in Java, we use `assert_` with an underscore.

### 3. DynamicArguments Class

Create a Java class called `DynamicArguments` that mimics the functionality of the LotusScript `DynamicArguments` class:

```java
public class DynamicArguments {
    private List<Object> args = new ArrayList<>();
    
    public int count() {
        return args.size();
    }
    
    public DynamicArguments in(Object v) {
        args.add(v);
        return this;
    }
    
    public boolean equals(DynamicArguments args2) {
        if (args2 == null) {
            return false;
        }
        
        if (this.count() != args2.count()) {
            return false;
        }
        
        if (this.count() == 0) {
            return true;
        }
        
        for (int i = 0; i < this.count(); i++) {
            if (!this.args.get(i).equals(args2.args.get(i))) {
                return false;
            }
        }
        
        return true;
    }
    
    public Object[] toArray() {
        return args.toArray();
    }
    
    public String toString(String separator) {
        return String.join(separator, args.stream()
            .map(Object::toString)
            .collect(Collectors.toList()));
    }
}
```

### 4. LoggerPrint Implementation

Implement the `LoggerPrint` class with the following methods:

```java
public class LoggerPrint implements Logger {
    @Override
    public void assert_(String msg, DynamicArguments id) {
        output("[ASSERT]     " + getModuleName(id) + msg);
    }
    
    @Override
    public void error(String msg, DynamicArguments id) {
        output("[ERROR]      " + getModuleName(id) + msg);
    }
    
    @Override
    public void warn(String msg, DynamicArguments id) {
        output("[WARN]        " + getModuleName(id) + msg);
    }
    
    @Override
    public void info(String msg, DynamicArguments id) {
        output("[INFO]           " + getModuleName(id) + msg);
    }
    
    @Override
    public void debug(String msg, DynamicArguments id) {
        output("[DEBUG]      " + getModuleName(id) + msg);
    }
    
    @Override
    public void verbose(String msg, DynamicArguments id) {
        output("[VERBOSE] " + getModuleName(id) + msg);
    }
    
    private void output(String msg) {
        System.out.println(LocalDateTime.now() + " " + msg);
    }
    
    private String getModuleName(DynamicArguments id) {
        return "[" + id.toString("->") + "] ";
    }
}
```

### 5. Configuration Options

Implement configuration options to mimic the Notes.ini settings:

```java
public class LoggerPrint implements Logger {
    private String logFilePath;
    private boolean logToConsole = true;
    private boolean logToFile = false;
    
    public LoggerPrint() {
        // Default constructor
    }
    
    public LoggerPrint(String logFilePath) {
        this.logFilePath = logFilePath;
        this.logToFile = true;
    }
    
    public void setLogToConsole(boolean logToConsole) {
        this.logToConsole = logToConsole;
    }
    
    public void setLogToFile(boolean logToFile) {
        this.logToFile = logToFile;
    }
    
    public void setLogFilePath(String logFilePath) {
        this.logFilePath = logFilePath;
        this.logToFile = true;
    }
    
    // ... other methods
    
    private void output(String msg) {
        String formattedMsg = LocalDateTime.now() + " " + msg;
        
        if (logToConsole) {
            System.out.println(formattedMsg);
        }
        
        if (logToFile && logFilePath != null) {
            try (FileWriter fw = new FileWriter(logFilePath, true);
                 BufferedWriter bw = new BufferedWriter(fw);
                 PrintWriter out = new PrintWriter(bw)) {
                out.println(formattedMsg);
            } catch (IOException e) {
                System.err.println("Error writing to log file: " + e.getMessage());
            }
        }
    }
}
```

### 6. Factory Method

Create a factory method to instantiate the logger:

```java
public class LoggerFactory {
    private static Logger instance;
    
    public static Logger getLogger() {
        if (instance == null) {
            instance = new LoggerPrint();
        }
        return instance;
    }
    
    public static void setLogger(Logger logger) {
        instance = logger;
    }
}
```

### 7. Log Level Management

Implement log level management similar to the original LotusScript implementation:

```java
public class LogLevel {
    public static final byte LL_NONE = 0;    // silence
    public static final byte LL_ASSERT = 1;
    public static final byte LL_ERROR = 2;
    public static final byte LL_WARN = 3;
    public static final byte LL_INFO = 4;
    public static final byte LL_DEBUG = 5;
    public static final byte LL_VERBOSE = 6;
    public static final byte LL_ALL = (byte) 255;
    
    private static Map<String, Byte> loggingLevels = new HashMap<>();
    
    public static void setLogLevel(String module, byte level) {
        if (module == null || module.isEmpty()) {
            return;
        }
        
        if ("ALL".equals(module)) {
            loggingLevels.clear();
        } else {
            // Remove any nested modules
            loggingLevels.entrySet().removeIf(entry -> 
                entry.getKey().toLowerCase().startsWith(module.toLowerCase()));
        }
        
        loggingLevels.put(module.toLowerCase(), level);
    }
    
    public static byte getLoggingLevel(String module, String className, String proc) {
        String id = (module + "|" + className + "|" + proc).toLowerCase();
        
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = (module + "|" + className).toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = module.toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = (className + "|" + proc).toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = className.toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = proc.toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = "all";
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        return LL_INFO; // Default level
    }
    
    public static boolean isLoggingAllowed(String module, String className, String proc, byte level) {
        return getLoggingLevel(module, className, proc) >= level;
    }
}
```

## Complete Java Implementation

Here's the complete Java implementation for the junior developer to follow:

```java
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public interface Logger {
    void assert_(String msg, DynamicArguments id);
    void error(String msg, DynamicArguments id);
    void warn(String msg, DynamicArguments id);
    void info(String msg, DynamicArguments id);
    void debug(String msg, DynamicArguments id);
    void verbose(String msg, DynamicArguments id);
}

public class DynamicArguments {
    private List<Object> args = new ArrayList<>();
    
    public int count() {
        return args.size();
    }
    
    public DynamicArguments in(Object v) {
        args.add(v);
        return this;
    }
    
    public boolean equals(DynamicArguments args2) {
        if (args2 == null) {
            return false;
        }
        
        if (this.count() != args2.count()) {
            return false;
        }
        
        if (this.count() == 0) {
            return true;
        }
        
        for (int i = 0; i < this.count(); i++) {
            if (!this.args.get(i).equals(args2.args.get(i))) {
                return false;
            }
        }
        
        return true;
    }
    
    public Object[] toArray() {
        return args.toArray();
    }
    
    public String toString(String separator) {
        return String.join(separator, args.stream()
            .map(Object::toString)
            .collect(Collectors.toList()));
    }
}

public class LogLevel {
    public static final byte LL_NONE = 0;    // silence
    public static final byte LL_ASSERT = 1;
    public static final byte LL_ERROR = 2;
    public static final byte LL_WARN = 3;
    public static final byte LL_INFO = 4;
    public static final byte LL_DEBUG = 5;
    public static final byte LL_VERBOSE = 6;
    public static final byte LL_ALL = (byte) 255;
    
    private static Map<String, Byte> loggingLevels = new HashMap<>();
    
    public static void setLogLevel(String module, byte level) {
        if (module == null || module.isEmpty()) {
            return;
        }
        
        if ("ALL".equals(module)) {
            loggingLevels.clear();
        } else {
            // Remove any nested modules
            loggingLevels.entrySet().removeIf(entry -> 
                entry.getKey().toLowerCase().startsWith(module.toLowerCase()));
        }
        
        loggingLevels.put(module.toLowerCase(), level);
    }
    
    public static byte getLoggingLevel(String module, String className, String proc) {
        String id = (module + "|" + className + "|" + proc).toLowerCase();
        
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = (module + "|" + className).toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = module.toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = (className + "|" + proc).toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = className.toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = proc.toLowerCase();
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        id = "all";
        if (loggingLevels.containsKey(id)) {
            return loggingLevels.get(id);
        }
        
        return LL_INFO; // Default level
    }
    
    public static boolean isLoggingAllowed(String module, String className, String proc, byte level) {
        return getLoggingLevel(module, className, proc) >= level;
    }
}

public class LoggerPrint implements Logger {
    private String logFilePath;
    private boolean logToConsole = true;
    private boolean logToFile = false;
    
    public LoggerPrint() {
        // Default constructor
    }
    
    public LoggerPrint(String logFilePath) {
        this.logFilePath = logFilePath;
        this.logToFile = true;
    }
    
    public void setLogToConsole(boolean logToConsole) {
        this.logToConsole = logToConsole;
    }
    
    public void setLogToFile(boolean logToFile) {
        this.logToFile = logToFile;
    }
    
    public void setLogFilePath(String logFilePath) {
        this.logFilePath = logFilePath;
        this.logToFile = true;
    }
    
    @Override
    public void assert_(String msg, DynamicArguments id) {
        output("[ASSERT]     " + getModuleName(id) + msg);
    }
    
    @Override
    public void error(String msg, DynamicArguments id) {
        output("[ERROR]      " + getModuleName(id) + msg);
    }
    
    @Override
    public void warn(String msg, DynamicArguments id) {
        output("[WARN]        " + getModuleName(id) + msg);
    }
    
    @Override
    public void info(String msg, DynamicArguments id) {
        output("[INFO]           " + getModuleName(id) + msg);
    }
    
    @Override
    public void debug(String msg, DynamicArguments id) {
        output("[DEBUG]      " + getModuleName(id) + msg);
    }
    
    @Override
    public void verbose(String msg, DynamicArguments id) {
        output("[VERBOSE] " + getModuleName(id) + msg);
    }
    
    private void output(String msg) {
        String formattedMsg = LocalDateTime.now() + " " + msg;
        
        if (logToConsole) {
            System.out.println(formattedMsg);
        }
        
        if (logToFile && logFilePath != null) {
            try (FileWriter fw = new FileWriter(logFilePath, true);
                 BufferedWriter bw = new BufferedWriter(fw);
                 PrintWriter out = new PrintWriter(bw)) {
                out.println(formattedMsg);
            } catch (IOException e) {
                System.err.println("Error writing to log file: " + e.getMessage());
            }
        }
    }
    
    private String getModuleName(DynamicArguments id) {
        return "[" + id.toString("->") + "] ";
    }
}

public class LoggerFactory {
    private static Logger instance;
    
    public static Logger getLogger() {
        if (instance == null) {
            instance = new LoggerPrint();
        }
        return instance;
    }
    
    public static void setLogger(Logger logger) {
        instance = logger;
    }
}

// Helper class to create DynamicArguments instances
public class Args {
    public static DynamicArguments create() {
        return new DynamicArguments();
    }
}
```

## Usage Examples

Here are some examples of how to use the migrated Java classes:

```java
// Basic usage
Logger logger = new LoggerPrint();
DynamicArguments args = new DynamicArguments().in("MyModule").in("MyClass").in("myMethod");
logger.info("This is an info message", args);

// Using the factory
Logger logger = LoggerFactory.getLogger();
logger.debug("Debug message", Args.create().in("Module").in("Class").in("method"));

// Setting log levels
LogLevel.setLogLevel("ALL", LogLevel.LL_WARN);
LogLevel.setLogLevel("MyModule", LogLevel.LL_DEBUG);

// Configuring file output
LoggerPrint fileLogger = new LoggerPrint("C:/logs/application.log");
LoggerFactory.setLogger(fileLogger);
```

## Integration with Spring Framework

For integration with Spring Framework, you can:

1. Define the Logger as a Spring Bean:

```java
@Configuration
public class LoggerConfig {
    @Bean
    public Logger logger() {
        LoggerPrint logger = new LoggerPrint();
        
        // Configure from application properties
        if (environment.getProperty("logging.file.path") != null) {
            logger.setLogFilePath(environment.getProperty("logging.file.path"));
        }
        
        return logger;
    }
}
```

2. Use Spring's dependency injection:

```java
@Service
public class MyService {
    private final Logger logger;
    
    @Autowired
    public MyService(Logger logger) {
        this.logger = logger;
    }
    
    public void doSomething() {
        logger.info("Operation performed", Args.create().in("MyService").in("doSomething"));
    }
}
```

## Testing

Create unit tests to verify the functionality:

```java
@Test
public void testLoggerPrint() {
    // Setup
    ByteArrayOutputStream outContent = new ByteArrayOutputStream();
    System.setOut(new PrintStream(outContent));
    
    // Test
    LoggerPrint logger = new LoggerPrint();
    logger.info("Test message", Args.create().in("TestModule").in("testMethod"));
    
    // Verify
    String output = outContent.toString();
    assertTrue(output.contains("[INFO]"));
    assertTrue(output.contains("[TestModule->testMethod]"));
    assertTrue(output.contains("Test message"));
}
```

## Conclusion

This specification provides a comprehensive guide for migrating the Lotus Notes `libLoggerPrint.lss` class to Java. The Java implementation maintains the same functionality while adapting to Java's language features and conventions. The junior developer should follow this specification to create a robust logging system that can be integrated with the new Angular and Java Spring application.