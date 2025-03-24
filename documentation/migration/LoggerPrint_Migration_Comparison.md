# LoggerPrint Migration: LotusScript vs Java Implementation Comparison

This document provides a side-by-side comparison of the original LotusScript `LoggerPrint` implementation and the new Java implementation using Logback.

## Class Structure Comparison

| Feature | LotusScript Implementation | Java Implementation |
|---------|---------------------------|---------------------|
| Base Class/Interface | `Class LoggerPrint As Logger` | Uses SLF4J `Logger` interface |
| Inheritance | Extends `Logger` class | Composition with SLF4J `Logger` |
| Log Levels | `assert`, `error`, `warn`, `info`, `debug`, `verbose` | `assertLog`, `error`, `warn`, `info`, `debug`, `verbose` |
| Context Management | Uses `DynamicArguments` | Uses `DynamicArguments` + SLF4J MDC |
| Output Mechanism | Direct `Print` statement | Configurable Logback appenders |
| Configuration | Notes.ini variables | XML/Groovy configuration files |

## Method Comparison

### LotusScript Methods

```lss
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
```

### Java Methods

```java
public void assertLog(String msg, DynamicArguments id) {
    try {
        MDC.put("context", getModuleName(id));
        logger.error("[ASSERT] {}", msg);
    } finally {
        MDC.remove("context");
    }
}

public void error(String msg, DynamicArguments id) {
    try {
        MDC.put("context", getModuleName(id));
        logger.error("[ERROR] {}", msg);
    } finally {
        MDC.remove("context");
    }
}

public void warn(String msg, DynamicArguments id) {
    try {
        MDC.put("context", getModuleName(id));
        logger.warn("[WARN] {}", msg);
    } finally {
        MDC.remove("context");
    }
}

public void info(String msg, DynamicArguments id) {
    try {
        MDC.put("context", getModuleName(id));
        logger.info("[INFO] {}", msg);
    } finally {
        MDC.remove("context");
    }
}

public void debug(String msg, DynamicArguments id) {
    try {
        MDC.put("context", getModuleName(id));
        logger.debug("[DEBUG] {}", msg);
    } finally {
        MDC.remove("context");
    }
}

public void verbose(String msg, DynamicArguments id) {
    try {
        MDC.put("context", getModuleName(id));
        logger.trace("[VERBOSE] {}", msg);
    } finally {
        MDC.remove("context");
    }
}

private String getModuleName(DynamicArguments id) {
    return "[" + id.toString("->") + "] ";
}
```

## Usage Comparison

### LotusScript Usage

```lss
' Setting log levels
Call setLogLevel("ALL", LL_WARN)
Call setLogLevel("libTracer", LL_DEBUG)

' Creating context and logging
Dim id As DynamicArguments
Set id = args().in("MyModule").in("MyProcedure")
Call logInfo("This is an info message")
```

### Java Usage

```java
// Setting log levels
LogLevelManager.setRootLogLevel(LogLevelManager.LL_WARN);
LogLevelManager.setLogLevel("com.yourcompany.tracer", LogLevelManager.LL_DEBUG);

// Creating context and logging
DynamicArguments context = args().in("MyModule").in("MyMethod");
logger.info("This is an info message", context);
```

## Configuration Comparison

### LotusScript Configuration (Notes.ini)

```
# Log to log.nsf on client
LogStatusBar=1

# Redirect logging to a file
Debug_Outfile=c:\temp\StatusBarLogging.txt
```

### Java Configuration (logback.xml)

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

## Key Differences and Improvements

1. **Thread Safety**: The Java implementation is thread-safe, using SLF4J's MDC for context management.

2. **Configurability**: Logback provides extensive configuration options through XML or Groovy files, allowing for:
   - Multiple appenders (console, file, database, etc.)
   - Custom layouts and patterns
   - Rolling file policies
   - Filtering based on log level or content

3. **Performance**: Logback includes performance optimizations:
   - Asynchronous logging
   - Conditional logging (checking log level before message construction)
   - Efficient parameter substitution

4. **Integration**: The Java implementation integrates with:
   - Spring Framework for dependency injection
   - Java EE/Jakarta EE environments
   - Testing frameworks for verification

5. **Extensibility**: Logback's architecture allows for:
   - Custom appenders
   - Custom layouts
   - Custom filters
   - Custom converters for pattern layouts

6. **Monitoring**: Logback provides:
   - JMX monitoring
   - Status monitoring
   - Conditional reloading of configuration

## Migration Path

1. **Direct Replacement**: The Java implementation provides a direct replacement for the LotusScript implementation, maintaining the same method signatures (except for `assert` → `assertLog`).

2. **Gradual Adoption**: The migration can be done gradually:
   - Start with core logging functionality
   - Add advanced features as needed
   - Integrate with Spring or other frameworks later

3. **Configuration Migration**:
   - Map Notes.ini settings to logback.xml
   - Preserve log formats for consistency
   - Add new capabilities as needed

## Log Level Mapping

| LotusScript Level | Value | Java/Logback Level |
|------------------|-------|-------------------|
| LL_NONE          | 0     | OFF               |
| LL_ASSERT        | 1     | ERROR             |
| LL_ERROR         | 2     | ERROR             |
| LL_WARN          | 3     | WARN              |
| LL_INFO          | 4     | INFO              |
| LL_DEBUG         | 5     | DEBUG             |
| LL_VERBOSE       | 6     | TRACE             |
| LL_ALL           | 255   | ALL               |