# LoggerPrint Logback Architecture

This document provides architectural diagrams for the Java implementation of the LoggerPrint class using Logback.

## Class Diagram

```mermaid
classDiagram
    class LoggerPrint {
        -Logger logger
        -DateTimeFormatter formatter
        +LoggerPrint(String loggerName)
        +LoggerPrint(Class<?> clazz)
        +assertLog(String msg, DynamicArguments id)
        +error(String msg, DynamicArguments id)
        +warn(String msg, DynamicArguments id)
        +info(String msg, DynamicArguments id)
        +debug(String msg, DynamicArguments id)
        +verbose(String msg, DynamicArguments id)
        -getModuleName(DynamicArguments id) String
        +static getLogger(Class<?> clazz) LoggerPrint
        +static getLogger(String name) LoggerPrint
    }
    
    class DynamicArguments {
        -List~Object~ args
        +in(Object value) DynamicArguments
        +count() int
        +toString(String separator) String
        +toArray() Object[]
        +static args() DynamicArguments
    }
    
    class LogLevelManager {
        +static LL_NONE: int
        +static LL_ASSERT: int
        +static LL_ERROR: int
        +static LL_WARN: int
        +static LL_INFO: int
        +static LL_DEBUG: int
        +static LL_VERBOSE: int
        +static LL_ALL: int
        +static setLogLevel(String loggerName, int level)
        +static setRootLogLevel(int level)
        -static convertToLogbackLevel(int level) Level
    }
    
    class Logger {
        <<interface>>
    }
    
    LoggerPrint --> Logger : uses
    LoggerPrint --> DynamicArguments : uses
    LogLevelManager --> Logger : configures
```

## Component Diagram

```mermaid
flowchart TB
    subgraph Application
        AppCode[Application Code]
        LoggerPrint[LoggerPrint]
        DynamicArgs[DynamicArguments]
        LogLevelMgr[LogLevelManager]
    end
    
    subgraph Logback
        SLF4J[SLF4J API]
        LogbackCore[Logback Core]
        LogbackClassic[Logback Classic]
        Config[logback.xml]
        
        subgraph Appenders
            ConsoleAppender[Console Appender]
            FileAppender[File Appender]
        end
    end
    
    AppCode --> LoggerPrint
    LoggerPrint --> DynamicArgs
    LoggerPrint --> SLF4J
    LogLevelMgr --> LogbackClassic
    
    SLF4J --> LogbackClassic
    LogbackClassic --> LogbackCore
    LogbackClassic --> Config
    
    Config --> ConsoleAppender
    Config --> FileAppender
    
    ConsoleAppender --> Console[Console Output]
    FileAppender --> LogFile[Log File]
```

## Sequence Diagram for Logging Process

```mermaid
sequenceDiagram
    participant App as Application
    participant LP as LoggerPrint
    participant DA as DynamicArguments
    participant SLF4J
    participant LB as Logback
    participant Appenders
    
    App->>DA: create context (args().in("module").in("method"))
    App->>LP: info("message", context)
    LP->>DA: getModuleName(context)
    DA-->>LP: formatted module name
    LP->>SLF4J: MDC.put("context", moduleName)
    LP->>SLF4J: logger.info("[INFO] {}", message)
    SLF4J->>LB: process log event
    LB->>LB: apply pattern layout
    LB->>Appenders: write formatted log
    LP->>SLF4J: MDC.remove("context")
```

## Log Level Mapping

```mermaid
flowchart LR
    subgraph "LotusScript Levels"
        LS_NONE[LL_NONE = 0]
        LS_ASSERT[LL_ASSERT = 1]
        LS_ERROR[LL_ERROR = 2]
        LS_WARN[LL_WARN = 3]
        LS_INFO[LL_INFO = 4]
        LS_DEBUG[LL_DEBUG = 5]
        LS_VERBOSE[LL_VERBOSE = 6]
        LS_ALL[LL_ALL = 255]
    end
    
    subgraph "Logback Levels"
        LB_OFF[OFF]
        LB_ERROR[ERROR]
        LB_WARN[WARN]
        LB_INFO[INFO]
        LB_DEBUG[DEBUG]
        LB_TRACE[TRACE]
        LB_ALL[ALL]
    end
    
    LS_NONE --> LB_OFF
    LS_ASSERT --> LB_ERROR
    LS_ERROR --> LB_ERROR
    LS_WARN --> LB_WARN
    LS_INFO --> LB_INFO
    LS_DEBUG --> LB_DEBUG
    LS_VERBOSE --> LB_TRACE
    LS_ALL --> LB_ALL
```

## Spring Integration

```mermaid
flowchart TB
    subgraph "Spring Application"
        Config[LoggingConfig]
        Service1[UserService]
        Service2[ProductService]
        Controller[WebController]
    end
    
    subgraph "Logging Components"
        LP1[LoggerPrint Bean: applicationLogger]
        LP2[LoggerPrint Bean: userServiceLogger]
        LP3[LoggerPrint Bean: productServiceLogger]
    end
    
    Config -->|@Bean| LP1
    Config -->|@Bean| LP2
    Config -->|@Bean| LP3
    
    Service1 -->|@Autowired| LP2
    Service2 -->|@Autowired| LP3
    Controller -->|@Autowired| LP1
```