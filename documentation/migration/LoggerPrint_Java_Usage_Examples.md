# LoggerPrint Java Usage Examples

This document provides practical examples of how to use the Java implementation of `LoggerPrint` with Logback in various scenarios. These examples are designed to help junior Java developers understand how to effectively use the logging framework in their applications.

## Basic Usage Examples

### Simple Logging

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;

import static com.yourcompany.logging.DynamicArguments.args;

public class SimpleExample {
    // Create a logger for this class
    private static final LoggerPrint logger = LoggerPrint.getLogger(SimpleExample.class);
    
    public void performTask() {
        // Create context with class and method information
        DynamicArguments context = args().in("SimpleExample").in("performTask");
        
        // Log at different levels
        logger.info("Starting task", context);
        
        // Some business logic
        boolean success = doSomething();
        
        if (success) {
            logger.info("Task completed successfully", context);
        } else {
            logger.error("Task failed", context);
        }
    }
    
    private boolean doSomething() {
        // Implementation
        return true;
    }
}
```

### Logging with Exception Information

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;

import static com.yourcompany.logging.DynamicArguments.args;

public class ExceptionExample {
    private static final LoggerPrint logger = LoggerPrint.getLogger(ExceptionExample.class);
    
    public void processData(String data) {
        DynamicArguments context = args().in("ExceptionExample").in("processData");
        
        try {
            logger.debug("Processing data: " + data, context);
            
            if (data == null) {
                throw new IllegalArgumentException("Data cannot be null");
            }
            
            // Process the data
            String result = data.toUpperCase();
            logger.info("Data processed: " + result, context);
            
        } catch (Exception e) {
            // Log the exception with details
            logger.error("Error processing data: " + e.getMessage() + 
                         "\nStack trace: " + getStackTraceAsString(e), context);
        }
    }
    
    private String getStackTraceAsString(Exception e) {
        StringBuilder sb = new StringBuilder();
        for (StackTraceElement element : e.getStackTrace()) {
            sb.append("\n\t").append(element.toString());
        }
        return sb.toString();
    }
}
```

### Conditional Logging

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import org.slf4j.LoggerFactory;
import ch.qos.logback.classic.Logger;

import static com.yourcompany.logging.DynamicArguments.args;

public class ConditionalExample {
    private static final LoggerPrint logger = LoggerPrint.getLogger(ConditionalExample.class);
    
    public void processLargeDataSet(List<String> items) {
        DynamicArguments context = args().in("ConditionalExample").in("processLargeDataSet");
        
        // Check if debug is enabled before constructing expensive log messages
        Logger underlyingLogger = (Logger) LoggerFactory.getLogger(ConditionalExample.class);
        boolean isDebugEnabled = underlyingLogger.isDebugEnabled();
        
        logger.info("Processing " + items.size() + " items", context);
        
        for (int i = 0; i < items.size(); i++) {
            // Only construct the detailed log message if debug is enabled
            if (isDebugEnabled) {
                logger.debug("Processing item " + i + ": " + items.get(i), context);
            }
            
            // Process the item
            processItem(items.get(i));
        }
        
        logger.info("Finished processing all items", context);
    }
    
    private void processItem(String item) {
        // Implementation
    }
}
```

## Spring Framework Integration Examples

### Configuration Class

```java
package com.yourcompany.config;

import com.yourcompany.logging.LoggerPrint;
import com.yourcompany.logging.LogLevelManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;

@Configuration
public class LoggingConfig {
    
    private final Environment env;
    
    public LoggingConfig(Environment env) {
        this.env = env;
    }
    
    @Bean
    public void configureLogLevels() {
        // Set default log level based on environment property
        String defaultLevel = env.getProperty("logging.level.default", "WARN");
        LogLevelManager.setRootLogLevel(convertStringLevelToInt(defaultLevel));
        
        // Set specific log levels
        LogLevelManager.setLogLevel("com.yourcompany.service", 
            convertStringLevelToInt(env.getProperty("logging.level.service", "INFO")));
        
        LogLevelManager.setLogLevel("com.yourcompany.repository", 
            convertStringLevelToInt(env.getProperty("logging.level.repository", "WARN")));
    }
    
    @Bean
    public LoggerPrint applicationLogger() {
        return LoggerPrint.getLogger("com.yourcompany.application");
    }
    
    @Bean
    public LoggerPrint serviceLogger() {
        return LoggerPrint.getLogger("com.yourcompany.service");
    }
    
    @Bean
    public LoggerPrint repositoryLogger() {
        return LoggerPrint.getLogger("com.yourcompany.repository");
    }
    
    private int convertStringLevelToInt(String level) {
        switch (level.toUpperCase()) {
            case "OFF": return LogLevelManager.LL_NONE;
            case "ERROR": return LogLevelManager.LL_ERROR;
            case "WARN": return LogLevelManager.LL_WARN;
            case "INFO": return LogLevelManager.LL_INFO;
            case "DEBUG": return LogLevelManager.LL_DEBUG;
            case "TRACE": return LogLevelManager.LL_VERBOSE;
            case "ALL": return LogLevelManager.LL_ALL;
            default: return LogLevelManager.LL_INFO;
        }
    }
}
```

### Service Class with Injected Logger

```java
package com.yourcompany.service;

import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import org.springframework.stereotype.Service;

import static com.yourcompany.logging.DynamicArguments.args;

@Service
public class ProductService {
    private final LoggerPrint logger;
    
    // Constructor injection of the logger
    public ProductService(LoggerPrint serviceLogger) {
        this.logger = serviceLogger;
    }
    
    public void createProduct(String name, double price) {
        DynamicArguments context = args().in("ProductService").in("createProduct");
        
        logger.info("Creating product: " + name + " with price: " + price, context);
        
        try {
            // Validate input
            if (name == null || name.trim().isEmpty()) {
                logger.warn("Attempted to create product with empty name", context);
                throw new IllegalArgumentException("Product name cannot be empty");
            }
            
            if (price <= 0) {
                logger.warn("Attempted to create product with invalid price: " + price, context);
                throw new IllegalArgumentException("Product price must be positive");
            }
            
            // Create the product
            // ...
            
            logger.info("Product created successfully with ID: 12345", context);
            
        } catch (Exception e) {
            logger.error("Failed to create product: " + e.getMessage(), context);
            throw e;
        }
    }
}
```

### Controller with Logging

```java
package com.yourcompany.controller;

import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import com.yourcompany.service.ProductService;
import org.springframework.web.bind.annotation.*;

import static com.yourcompany.logging.DynamicArguments.args;

@RestController
@RequestMapping("/api/products")
public class ProductController {
    private final LoggerPrint logger;
    private final ProductService productService;
    
    public ProductController(LoggerPrint applicationLogger, ProductService productService) {
        this.logger = applicationLogger;
        this.productService = productService;
    }
    
    @PostMapping
    public ResponseEntity<ProductResponse> createProduct(@RequestBody ProductRequest request) {
        DynamicArguments context = args().in("ProductController").in("createProduct");
        
        logger.info("Received request to create product: " + request.getName(), context);
        
        try {
            productService.createProduct(request.getName(), request.getPrice());
            
            ProductResponse response = new ProductResponse(/* ... */);
            
            logger.info("Product creation request completed successfully", context);
            return ResponseEntity.ok(response);
            
        } catch (IllegalArgumentException e) {
            logger.warn("Invalid product data: " + e.getMessage(), context);
            return ResponseEntity.badRequest().body(new ErrorResponse(e.getMessage()));
            
        } catch (Exception e) {
            logger.error("Unexpected error creating product: " + e.getMessage(), context);
            return ResponseEntity.status(500).body(new ErrorResponse("Internal server error"));
        }
    }
}
```

## Advanced Usage Examples

### Custom MDC Context

```java
package com.yourcompany.logging;

import org.slf4j.MDC;

import java.util.UUID;

public class LoggingContext implements AutoCloseable {
    private final String requestId;
    private final String userId;
    
    public LoggingContext(String userId) {
        this.requestId = UUID.randomUUID().toString();
        this.userId = userId;
        
        // Set MDC values
        MDC.put("requestId", requestId);
        MDC.put("userId", userId);
    }
    
    @Override
    public void close() {
        // Clear MDC values
        MDC.remove("requestId");
        MDC.remove("userId");
    }
    
    public String getRequestId() {
        return requestId;
    }
    
    public String getUserId() {
        return userId;
    }
}
```

Usage:

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import com.yourcompany.logging.LoggingContext;

import static com.yourcompany.logging.DynamicArguments.args;

public class AdvancedExample {
    private static final LoggerPrint logger = LoggerPrint.getLogger(AdvancedExample.class);
    
    public void processUserRequest(String userId, String action) {
        // Create a logging context that will be automatically closed
        try (LoggingContext context = new LoggingContext(userId)) {
            DynamicArguments args = args().in("AdvancedExample").in("processUserRequest");
            
            logger.info("Processing user request: " + action + " (Request ID: " + context.getRequestId() + ")", args);
            
            // Process the request
            // ...
            
            logger.info("User request processed successfully", args);
        }
    }
}
```

### Aspect-Oriented Logging

```java
package com.yourcompany.aspect;

import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import static com.yourcompany.logging.DynamicArguments.args;

@Aspect
@Component
public class LoggingAspect {
    private final LoggerPrint logger = LoggerPrint.getLogger(LoggingAspect.class);
    
    @Around("execution(* com.yourcompany.service.*.*(..))")
    public Object logServiceMethods(ProceedingJoinPoint joinPoint) throws Throwable {
        String className = joinPoint.getTarget().getClass().getSimpleName();
        String methodName = joinPoint.getSignature().getName();
        DynamicArguments context = args().in(className).in(methodName);
        
        logger.debug("Entering method: " + methodName, context);
        
        long startTime = System.currentTimeMillis();
        
        try {
            Object result = joinPoint.proceed();
            
            long duration = System.currentTimeMillis() - startTime;
            logger.debug("Exiting method: " + methodName + " (duration: " + duration + "ms)", context);
            
            return result;
            
        } catch (Exception e) {
            logger.error("Exception in method: " + methodName + " - " + e.getMessage(), context);
            throw e;
        }
    }
}
```

### Testing with LoggerPrint

```java
package com.yourcompany.service;

import ch.qos.logback.classic.Logger;
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.read.ListAppender;
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.slf4j.LoggerFactory;

import static com.yourcompany.logging.DynamicArguments.args;
import static org.junit.jupiter.api.Assertions.*;

public class UserServiceTest {
    private UserService userService;
    private LoggerPrint logger;
    private ListAppender<ILoggingEvent> listAppender;
    
    @BeforeEach
    public void setUp() {
        // Get the Logback logger
        Logger logbackLogger = (Logger) LoggerFactory.getLogger("test");
        
        // Create and start a ListAppender
        listAppender = new ListAppender<>();
        listAppender.start();
        
        // Add the appender to the logger
        logbackLogger.addAppender(listAppender);
        
        // Create the LoggerPrint instance
        logger = new LoggerPrint("test");
        
        // Create the service with the logger
        userService = new UserService(logger);
    }
    
    @Test
    public void testCreateUser_Success() {
        // Arrange
        String username = "testuser";
        
        // Act
        userService.createUser(username);
        
        // Assert
        assertEquals(2, listAppender.list.size());
        
        // First log message should be info about creating user
        ILoggingEvent firstEvent = listAppender.list.get(0);
        assertEquals("INFO", firstEvent.getLevel().toString());
        assertTrue(firstEvent.getFormattedMessage().contains("Creating user: testuser"));
        
        // Second log message should be info about successful creation
        ILoggingEvent secondEvent = listAppender.list.get(1);
        assertEquals("INFO", secondEvent.getLevel().toString());
        assertTrue(secondEvent.getFormattedMessage().contains("User created successfully"));
    }
    
    @Test
    public void testCreateUser_NullUsername() {
        // Arrange
        String username = null;
        
        // Act & Assert
        Exception exception = assertThrows(IllegalArgumentException.class, () -> {
            userService.createUser(username);
        });
        
        assertEquals("Username cannot be null", exception.getMessage());
        
        // Should have logged a warning and an error
        assertEquals(2, listAppender.list.size());
        
        // First log message should be info about creating user
        ILoggingEvent firstEvent = listAppender.list.get(0);
        assertEquals("INFO", firstEvent.getLevel().toString());
        
        // Second log message should be a warning
        ILoggingEvent secondEvent = listAppender.list.get(1);
        assertEquals("WARN", secondEvent.getLevel().toString());
        assertTrue(secondEvent.getFormattedMessage().contains("Invalid username"));
    }
}
```

## Performance Optimization Examples

### Lazy Message Construction

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;
import org.slf4j.LoggerFactory;
import ch.qos.logback.classic.Logger;

import java.util.function.Supplier;

import static com.yourcompany.logging.DynamicArguments.args;

public class PerformanceExample {
    private static final LoggerPrint logger = LoggerPrint.getLogger(PerformanceExample.class);
    
    // Helper method for lazy message construction
    private static void logDebug(DynamicArguments context, Supplier<String> messageSupplier) {
        Logger underlyingLogger = (Logger) LoggerFactory.getLogger(PerformanceExample.class);
        if (underlyingLogger.isDebugEnabled()) {
            logger.debug(messageSupplier.get(), context);
        }
    }
    
    public void processComplexData(ComplexData data) {
        DynamicArguments context = args().in("PerformanceExample").in("processComplexData");
        
        logger.info("Processing complex data", context);
        
        // Use lazy message construction for expensive debug logging
        logDebug(context, () -> "Complex data details: " + data.generateDetailedReport());
        
        // Process the data
        // ...
        
        logger.info("Complex data processing complete", context);
    }
}
```

### Batch Logging

```java
import com.yourcompany.logging.DynamicArguments;
import com.yourcompany.logging.LoggerPrint;

import java.util.ArrayList;
import java.util.List;

import static com.yourcompany.logging.DynamicArguments.args;

public class BatchLoggingExample {
    private static final LoggerPrint logger = LoggerPrint.getLogger(BatchLoggingExample.class);
    private static final int BATCH_SIZE = 100;
    
    public void processLargeDataSet(List<String> items) {
        DynamicArguments context = args().in("BatchLoggingExample").in("processLargeDataSet");
        
        logger.info("Starting to process " + items.size() + " items", context);
        
        List<String> processingLog = new ArrayList<>();
        int count = 0;
        
        for (String item : items) {
            // Process the item
            String result = processItem(item);
            
            // Add to processing log
            processingLog.add("Item " + count + ": " + result);
            count++;
            
            // Log in batches to reduce overhead
            if (count % BATCH_SIZE == 0 || count == items.size()) {
                logger.debug("Processed batch: " + String.join(", ", processingLog), context);
                processingLog.clear();
            }
        }
        
        logger.info("Completed processing " + items.size() + " items", context);
    }
    
    private String processItem(String item) {
        // Implementation
        return "processed";
    }
}
```

## Conclusion

These examples demonstrate how to use the Java implementation of `LoggerPrint` with Logback in various scenarios. By following these patterns, junior Java developers can effectively implement logging in their applications while maintaining compatibility with the original LotusScript logging approach.

Remember these key principles:

1. **Create context information** using `DynamicArguments` to maintain the module/method structure
2. **Choose the appropriate log level** for each message
3. **Include relevant information** in log messages to aid debugging
4. **Consider performance** when logging in high-volume scenarios
5. **Use dependency injection** when working with Spring Framework
6. **Write tests** to verify logging behavior

For more advanced usage, refer to the Logback documentation at https://logback.qos.ch/documentation.html