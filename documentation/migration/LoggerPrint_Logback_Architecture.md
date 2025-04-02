# LoggerPrint Logback Architecture

This document provides architectural diagrams for the Java implementation of the LoggerPrint class using Logback.

## Class Diagram

```mermaid
classDiagram
    class LoggerPrint {
        -Logger logger
        -DateTimeFormatter formatter
        -String instanceName
        +LoggerPrint(String loggerName)
        +LoggerPrint(Class<?> clazz)
        +assertLog(String msg, DynamicArguments id)
        +error(String msg, DynamicArguments id)
        +warn(String msg, DynamicArguments id)
        +info(String msg, DynamicArguments id)
        +debug(String msg, DynamicArguments id)
        +verbose(String msg, DynamicArguments id)
        -getModuleName(DynamicArguments id) String
        -formatMessage(String msg, DynamicArguments id) String
        +static getLogger(Class<?> clazz) LoggerPrint
        +static getLogger(String name) LoggerPrint
        +getInstanceName() String
    }
    
    class DynamicArguments {
        -List~Object~ args
        +in(Object value) DynamicArguments
        +count() int
        +toString(String separator) String
        +toArray() Object[]
        +static args() DynamicArguments
        +get(int index) Object
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
        +static getEffectiveLevel(String loggerName) int
    }
    
    class Logger {
        <<interface>>
        +isTraceEnabled() boolean
        +isDebugEnabled() boolean
        +isInfoEnabled() boolean
        +isWarnEnabled() boolean
        +isErrorEnabled() boolean
        +trace(String format, Object... args) void
        +debug(String format, Object... args) void
        +info(String format, Object... args) void
        +warn(String format, Object... args) void
        +error(String format, Object... args) void
    }
    
    class MDC {
        <<utility>>
        +static put(String key, String val) void
        +static remove(String key) void
        +static clear() void
        +static getCopyOfContextMap() Map
    }
    
    LoggerPrint --> Logger : uses
    LoggerPrint --> DynamicArguments : uses
    LoggerPrint --> MDC : uses
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
        MDC[MDC Context]
        
        subgraph Appenders
            ConsoleAppender[Console Appender]
            FileAppender[File Appender]
            RollingFileAppender[Rolling File Appender]
        end
        
        subgraph Layouts
            PatternLayout[Pattern Layout]
            JsonLayout[JSON Layout]
        end
        
        subgraph Filters
            ThresholdFilter[Threshold Filter]
            LevelFilter[Level Filter]
        end
    end
    
    AppCode --> LoggerPrint
    LoggerPrint --> DynamicArgs
    LoggerPrint --> SLF4J
    LoggerPrint --> MDC
    LogLevelMgr --> LogbackClassic
    
    SLF4J --> LogbackClassic
    LogbackClassic --> LogbackCore
    LogbackClassic --> Config
    LogbackClassic --> MDC
    
    Config --> Appenders
    Config --> Layouts
    Config --> Filters
    
    Appenders --> Layouts
    Appenders --> Filters
    
    ConsoleAppender --> Console[Console Output]
    FileAppender --> LogFile[Log File]
    RollingFileAppender --> ArchiveFiles[Archive Log Files]
```

## Sequence Diagram for Logging Process

```mermaid
sequenceDiagram
    participant App as Application
    participant LP as LoggerPrint
    participant DA as DynamicArguments
    participant MDC as SLF4J MDC
    participant SLF4J
    participant LB as Logback
    participant Appenders
    
    App->>DA: create context (args().in("module").in("method"))
    App->>LP: info("message", context)
    
    LP->>DA: getModuleName(context)
    DA-->>LP: formatted module name
    
    LP->>LP: formatMessage("message", context)
    LP->>MDC: MDC.put("context", moduleName)
    
    alt isInfoEnabled
        LP->>SLF4J: logger.info("[INFO] {}", formattedMessage)
        SLF4J->>LB: process log event
        LB->>LB: apply pattern layout
        LB->>Appenders: write formatted log
        Appenders-->>App: log written
    else
        LP->>LP: skip logging (level not enabled)
    end
    
    LP->>MDC: MDC.remove("context")
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
        Repository[DataRepository]
        Interceptor[LoggingInterceptor]
        Aspect[LoggingAspect]
    end
    
    subgraph "Logging Components"
        LP1[LoggerPrint Bean: applicationLogger]
        LP2[LoggerPrint Bean: userServiceLogger]
        LP3[LoggerPrint Bean: productServiceLogger]
        LP4[LoggerPrint Bean: dataLogger]
    end
    
    subgraph "Spring Context"
        PropSource[PropertySource]
        Profiles[Spring Profiles]
    end
    
    Config -->|@Bean| LP1
    Config -->|@Bean| LP2
    Config -->|@Bean| LP3
    Config -->|@Bean| LP4
    
    Service1 -->|@Autowired| LP2
    Service2 -->|@Autowired| LP3
    Controller -->|@Autowired| LP1
    Repository -->|@Autowired| LP4
    
    Interceptor -->|uses| LP1
    Aspect -->|uses| LP1
    
    PropSource -->|configure| Config
    Profiles -->|activate| Config
```

## Logback Configuration Hierarchy

```mermaid
flowchart TB
    root[Root Logger]
    app[com.example]
    service[com.example.service]
    controller[com.example.controller]
    util[com.example.util]
    
    root --> app
    app --> service
    app --> controller
    app --> util
    
    service -->|inherits| service1[UserService]
    service -->|inherits| service2[ProductService]
    
    controller -->|inherits| ctrl1[UserController]
    controller -->|inherits| ctrl2[AdminController]
    
    classDef default fill:#f9f9f9,stroke:#333,stroke-width:1px
    classDef root fill:#f8cecc,stroke:#b85450
    classDef app fill:#d5e8d4,stroke:#82b366
    classDef inherited fill:#dae8fc,stroke:#6c8ebf
    
    class root root
    class app,service,controller,util app
    class service1,service2,ctrl1,ctrl2 inherited
```

## MDC Context Flow

```mermaid
sequenceDiagram
    participant Client
    participant Filter as RequestFilter
    participant Controller
    participant Service
    participant Repository
    participant LoggerPrint
    
    Client->>Filter: HTTP Request
    Filter->>Filter: Generate requestId
    Filter->>MDC: put("requestId", uuid)
    Filter->>MDC: put("clientIP", ip)
    Filter->>Controller: forward request
    
    Controller->>LoggerPrint: info("Request received", args().in("endpoint"))
    LoggerPrint->>MDC: put("context", "controller")
    LoggerPrint->>Log: write log with MDC context
    LoggerPrint->>MDC: remove("context")
    
    Controller->>Service: process()
    Service->>LoggerPrint: info("Processing data", args().in("service"))
    LoggerPrint->>MDC: put("context", "service")
    LoggerPrint->>Log: write log with MDC context
    LoggerPrint->>MDC: remove("context")
    
    Service->>Repository: findData()
    Repository->>LoggerPrint: debug("Executing query", args().in("repository"))
    LoggerPrint->>MDC: put("context", "repository")
    LoggerPrint->>Log: write log with MDC context
    LoggerPrint->>MDC: remove("context")
    
    Repository-->>Service: return data
    Service-->>Controller: return result
    Controller-->>Filter: return response
    Filter->>MDC: clear()
    Filter-->>Client: HTTP Response
```