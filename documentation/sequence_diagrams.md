# LotusScript Developer Layer (LSDL) - Sequence Diagrams

## Overview

This document provides sequence diagrams for key operations in the LotusScript Developer Layer (LSDL) framework, illustrating how different components interact during various processes.

## Module Registration Sequence

When a LotusScript module initializes, it registers itself with the framework to enable proper identification in logs, traces, and error reports.

```mermaid
sequenceDiagram
    participant Module as LotusScript Module
    participant DLBase as libDLBase
    
    Module->>Module: Sub Initialize()
    Module->>DLBase: registerModule("ModuleName")
    DLBase->>DLBase: gl_modules(GetThreadInfo(11)) = "ModuleName"
    Note over DLBase: Module is now registered
```

## Error Handling Sequence

### Basic Error Handling

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Throwable as Throwable Class
    participant Error as libError
    participant ErrorDetails as ErrorDetails Class
    participant ErrorReport as ErrorStackReport Class
    
    App->>App: On Error GoTo catch
    Note over App: Error occurs
    App->>App: GoTo catch
    App->>Throwable: throwException()
    Throwable->>Error: getErrorStack("", GetThreadInfo(10), GetThreadInfo(11), TypeName(Me))
    Error->>ErrorDetails: New ErrorDetails(module, proc)
    Error->>Error: Build error stack
    Error->>ErrorReport: errorStackToDetailed(stack)
    ErrorReport->>ErrorReport: Format error stack
    ErrorReport-->>Error: Return formatted stack
    Error-->>Throwable: Return error stack
    Throwable->>App: Error Err, formattedStack
    Note over App: Error handler processes error
```

### Detailed Error Handling

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Throwable as Throwable Class
    participant Error as libError
    
    App->>App: On Error GoTo catch
    Note over App: Error occurs
    App->>App: GoTo catch
    App->>Throwable: throwExceptionDetailed("Additional context")
    Throwable->>Error: getErrorStack("Additional context", GetThreadInfo(10), GetThreadInfo(11), TypeName(Me))
    Error->>Error: Process as in basic error handling
    Error-->>Throwable: Return error stack with additional context
    Throwable->>App: Error Err, formattedStack
    Note over App: Error handler processes error
```

## Logging Sequence

### Direct Logging

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Log as libLog
    participant LogConfig as libLogConfig
    participant Logger as Logger Implementation
    
    App->>Log: logInfo("Message")
    Log->>Log: proc = GetThreadInfo(10)
    Log->>Log: module = getModuleName(GetThreadInfo(11))
    Log->>Log: isLoggingAllowed(module, proc, LL_INFO)
    Log->>Log: getLoggingLevel(module, proc)
    Note over Log: Check if logging level allows this message
    alt Logging allowed
        Log->>Log: L()
        Log->>LogConfig: libLogConfigInit() (if first log)
        LogConfig->>Log: Configure log levels
        Log->>Logger: info("Message", args.in(module).in(proc))
        Logger->>Logger: Format message
        Logger->>Logger: Output message
    end
```

### Class-based Logging

```mermaid
sequenceDiagram
    participant App as Application Class
    participant Loggable as Loggable Class
    participant Log as libLog
    participant Logger as Logger Implementation
    
    App->>Loggable: logInfo("Message")
    Loggable->>Loggable: proc = GetThreadInfo(10)
    Loggable->>Loggable: module = getModuleName(GetThreadInfo(11))
    Loggable->>Loggable: className = TypeName(Me)
    Loggable->>Loggable: isLoggingAllowed(module, className, proc, LL_INFO)
    Loggable->>Loggable: getLoggingLevel(module, className, proc)
    alt Logging allowed
        Loggable->>Log: L.info("Message", args.in(module).in(className).in(proc))
        Log->>Logger: info("Message", context)
        Logger->>Logger: Format message
        Logger->>Logger: Output message
    end
```

## Tracing Sequence

### Function Entry and Exit

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Traceable as Traceable Class
    participant Tracer as libTracer
    participant Stopwatch as Stopwatch Implementation
    
    App->>Traceable: traceIn()
    Traceable->>Tracer: traceIn()
    Tracer->>Tracer: module = getModuleName(GetThreadInfo(11))
    Tracer->>Tracer: proc = GetThreadInfo(10)
    Tracer->>Tracer: Push to trace stack
    Tracer->>Stopwatch: start()
    
    Note over App: Function execution
    
    App->>Traceable: traceOut()
    Traceable->>Tracer: traceOut()
    Tracer->>Stopwatch: lap()
    Stopwatch-->>Tracer: Return elapsed time
    Tracer->>Tracer: Pop from trace stack
    Tracer->>Tracer: Record execution time
```

### Trace Reporting

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Tracer as libTracer
    
    App->>Tracer: getTraceReport()
    Tracer->>Tracer: Format trace stack
    Tracer-->>App: Return formatted trace report
    App->>App: Display or log trace report
```

## Unit Testing Sequence

### Test Execution

```mermaid
sequenceDiagram
    participant Agent as agUnitsTester
    participant TestCase as libTestCase
    participant TestLib as Test Library
    participant AppLib as Application Library
    
    Agent->>Agent: Find test libraries
    Agent->>TestLib: Create test class instance
    TestLib->>TestCase: Inherit test capabilities
    TestLib->>TestLib: runTest()
    loop For each test method
        TestLib->>AppLib: Call function under test
        TestLib->>TestCase: assert methods
        TestCase->>TestCase: Record test results
    end
    Agent->>Agent: Generate test report
    Agent->>Agent: Display test results
```

### Assertion Process

```mermaid
sequenceDiagram
    participant Test as Test Class
    participant TestCase as TestCase Class
    
    Test->>TestCase: assertTrue(condition)
    alt Condition is true
        TestCase->>TestCase: Record success
    else Condition is false
        TestCase->>TestCase: Record failure
        TestCase->>TestCase: Capture stack trace
    end
    TestCase-->>Test: Continue execution
```

## Complex Scenarios

### Error in Traced Function

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Traceable as Traceable Class
    participant Tracer as libTracer
    participant Throwable as Throwable Class
    participant Error as libError
    participant Log as libLog
    
    App->>Traceable: traceIn()
    Traceable->>Tracer: traceIn()
    
    Note over App: Error occurs
    
    App->>Throwable: throwException()
    Throwable->>Error: getErrorStack()
    Error-->>Throwable: Return error stack
    Throwable->>App: Error Err, formattedStack
    
    App->>Log: logError("Error occurred")
    Log->>Log: Process log message
    
    App->>Traceable: traceOut()
    Traceable->>Tracer: traceOut()
    Tracer->>Tracer: Record execution time
    Tracer->>Tracer: Mark trace as failed
```

### Logger Configuration and Initialization

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Log as libLog
    participant DLBase as libDLBase
    participant Config as libConfig
    participant LogConfig as libLogConfig
    participant LoggerPrint as LoggerPrint Class
    
    App->>Log: First log call
    Log->>Log: L()
    Log->>Log: gl_logger Is Nothing
    Log->>DLBase: Execute libLogConfig code
    DLBase->>LogConfig: libLogConfigInit()
    LogConfig->>Log: setLogLevel("ALL", LL_WARN)
    Log->>DLBase: classOverloadFactory(CLASS_LOGGER)
    DLBase->>Config: Check for logger overrides
    Config->>DLBase: overloadDefaultClass("Logger", "libLoggerPrint", "LoggerPrint")
    DLBase->>DLBase: Create LoggerPrint instance
    DLBase-->>Log: Return logger instance
    Log->>Log: Set gl_logger
    Log->>LoggerPrint: Process log message
```

## Component Interaction for Key Operations

### Application Startup

```mermaid
sequenceDiagram
    participant App as Application
    participant DLBase as libDLBase
    participant Config as libConfig
    participant LogConfig as libLogConfig
    
    App->>DLBase: Use "libDLBase"
    DLBase->>DLBase: Initialize()
    DLBase->>DLBase: registerModule("libDLBase")
    
    App->>App: Initialize()
    App->>DLBase: registerModule("AppName")
    
    Note over App: First log or error operation
    
    App->>DLBase: Try to load libConfig
    alt libConfig exists
        DLBase->>Config: libConfigInit()
        Config->>DLBase: Configure overrides
    end
    
    App->>DLBase: Try to load libLogConfig
    alt libLogConfig exists
        DLBase->>LogConfig: libLogConfigInit()
        LogConfig->>LogConfig: Configure log levels
    end
```

### Error Handling with Logging

```mermaid
sequenceDiagram
    participant App as Application Code
    participant Throwable as Throwable Class
    participant Error as libError
    participant Log as libLog
    participant Logger as Logger Implementation
    
    Note over App: Error occurs
    App->>Throwable: throwException()
    Throwable->>Error: getErrorStack()
    Error-->>Throwable: Return error stack
    Throwable->>App: Error Err, formattedStack
    
    App->>Log: logError("Error details")
    Log->>Log: Check log level
    Log->>Logger: error("Error details", context)
    Logger->>Logger: Format and output message
```

## Best Practices for Component Interaction

1. **Proper Initialization**: Always call `registerModule` in the `Initialize` sub of each module.

2. **Error Handling Pattern**: Use the standard error handling pattern with `On Error GoTo catch` and `throwException`.

3. **Tracing Discipline**: Always pair `traceIn` with `traceOut`, even in error paths.

4. **Log Level Selection**: Choose appropriate log levels for different types of messages.

5. **Class Inheritance**: Use the appropriate base class (`Throwable`, `Loggable`, `Traceable`) based on the functionality needed.

## Implementation Considerations

1. **Thread Safety**: The framework uses thread-specific information to track module names and trace stacks.

2. **Performance Impact**: Tracing and detailed logging can impact performance, especially in tight loops.

3. **Error Recovery**: The framework is designed to continue functioning even if parts of it encounter errors.

4. **Extension Points**: The framework provides several extension points through class inheritance and overrides.