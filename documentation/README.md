# LotusScript Developer Layer (LSDL) Documentation

## Overview

This documentation provides a comprehensive analysis of the LotusScript Developer Layer (LSDL) framework, which is designed to enhance Lotus Notes/Domino development by providing robust error handling, logging, tracing, and unit testing capabilities in LotusScript.

The documentation is intended to support the reengineering of this Lotus Notes application to an Angular and Java Spring application.

## Documentation Index

### 1. [Architecture Documentation](architecture.md)
Provides an overview of the LSDL framework architecture, including its core components, data flow, and usage patterns.

### 2. [Data Flow Documentation](data_flow.md)
Describes how information moves between different components during various operations such as error handling, logging, tracing, and unit testing.

### 3. [Class Structure Documentation](class_structure.md)
Details the key classes, their relationships, inheritance hierarchies, and responsibilities within the LSDL framework.

### 4. [Sequence Diagrams](sequence_diagrams.md)
Illustrates how different components interact during various processes through sequence diagrams for key operations.

### 5. [Source Code Documentation](source_code_documentation.md)
Provides detailed documentation for each source code file in the LSDL framework, explaining their purpose, functionality, and key components.

### 6. [Migration Strategy](migration_strategy.md)
Outlines the strategy for migrating the LSDL framework from Lotus Notes/Domino to a modern architecture using Angular for the frontend and Java Spring for the backend.

## Framework Summary

The LSDL framework consists of several key components:

1. **Error Handling**: Standardized exception handling and stack trace reporting
2. **Logging System**: Configurable logging with multiple levels
3. **Tracing**: Function entry/exit tracking with performance measurement
4. **Unit Testing**: Framework for defining and running tests

These components are implemented through a set of LotusScript libraries that work together to provide a comprehensive development layer for Lotus Notes applications.

## Key Features

- **Standardized Error Handling**: Consistent approach to error handling with detailed stack traces
- **Configurable Logging**: Multiple log levels with configurable verbosity
- **Performance Measurement**: Timing of function execution for performance analysis
- **Unit Testing Framework**: Structure for defining and running tests with various assertion methods
- **Extensibility**: Framework can be extended through class overrides and custom implementations

## Directory Structure

```
├── extensions/
│   └── libraries/
│       ├── libConfig.lss
│       ├── libLogConfig.lss
│       └── libLoggerPrint.lss
├── source/
│   ├── agents/
│   │   └── agUnitsTester.lss
│   ├── libraries/
│   │   ├── libDLBase.lss
│   │   ├── libDynamicArguments.lss
│   │   ├── libError.lss
│   │   ├── libLog.lss
│   │   ├── libStopwatchLSXLC.lss
│   │   ├── libTestCase.lss
│   │   └── libTracer.lss
│   └── toolbar-action.formula
└── test/
    └── libraries/
        ├── libDynamicArgumentsTest.lss
        ├── libErrorTest.lss
        ├── libLogTest.lss
        └── libTracerTest.lss
```

## Usage Patterns

### Error Handling Pattern

```lss
Sub someFunction()
    On Error GoTo catch
    
    ' Function code
    
    GoTo finally
catch:
    throwException
finally:
End Sub
```

### Tracing Pattern

```lss
Sub someFunction()
    traceIn
    On Error GoTo catch
    
    ' Function code
    
    GoTo finally
catch:
    traceOut
    throwException
finally:
    traceOut
End Sub
```

### Logging Pattern

```lss
logAssert "Critical assertion message"
logError "Error message"
logWarn "Warning message"
logInfo "Informational message"
logDebug "Debug message"
logVerbose "Verbose message"
```

### Class Inheritance Pattern

```lss
Class MyClass As Throwable
    ' Inherits error handling capabilities
End Class

Class MyLoggableClass As Loggable
    ' Inherits both error handling and logging capabilities
End Class

Class MyTraceableClass As Traceable
    ' Inherits error handling and tracing capabilities
End Class
```

## Migration Approach

The migration to Angular and Java Spring will follow these phases:

1. **Analysis and Planning**: Detailed code analysis, architecture design, and technology selection
2. **Core Framework Implementation**: Implementing the core functionality in the new technologies
3. **Feature Migration**: Incremental migration of features with validation
4. **Deployment and Transition**: User training and phased rollout

For more details, see the [Migration Strategy](migration_strategy.md) document.