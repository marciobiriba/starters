

# Spring Boot Starter Reverse Engineering – spring-boot-starter-web

## A. Dependency tree proof

[INFO] com.example:demo:jar:0.0.1-SNAPSHOT
[INFO] \- org.springframework.boot:spring-boot-starter-web:jar:3.5.7:compile
[INFO]    +- org.springframework.boot:spring-boot-starter:jar:3.5.7:compile
[INFO]    |  +- org.springframework.boot:spring-boot:jar:3.5.7:compile
[INFO]    |  +- org.springframework.boot:spring-boot-autoconfigure:jar:3.5.7:compile
[INFO]    |  +- org.springframework.boot:spring-boot-starter-logging:jar:3.5.7:compile
[INFO]    |  |  +- ch.qos.logback:logback-classic:jar:1.5.20:compile
[INFO]    |  |  |  +- ch.qos.logback:logback-core:jar:1.5.20:compile
[INFO]    |  |  |  \- org.slf4j:slf4j-api:jar:2.0.17:compile
[INFO]    |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.24.3:compile
[INFO]    |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.24.3:compile
[INFO]    |  |  \- org.slf4j:jul-to-slf4j:jar:2.0.17:compile
[INFO]    |  +- jakarta.annotation:jakarta.annotation-api:jar:2.1.1:compile
[INFO]    |  +- org.springframework:spring-core:jar:6.2.12:compile
[INFO]    |  |  \- org.springframework:spring-jcl:jar:6.2.12:compile
[INFO]    |  \- org.yaml:snakeyaml:jar:2.4:compile
[INFO]    +- org.springframework.boot:spring-boot-starter-json:jar:3.5.7:compile
[INFO]    |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.19.2:compile
[INFO]    |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.19.2:compile
[INFO]    |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.19.2:compile
[INFO]    |  +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.19.2:compile
[INFO]    |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.19.2:compile
[INFO]    |  \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.19.2:compile
[INFO]    +- org.springframework.boot:spring-boot-starter-tomcat:jar:3.5.7:compile
[INFO]    |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:10.1.48:compile
[INFO]    |  +- org.apache.tomcat.embed:tomcat-embed-el:jar:10.1.48:compile
[INFO]    |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:10.1.48:compile
[INFO]    +- org.springframework:spring-web:jar:6.2.12:compile
[INFO]    |  +- org.springframework:spring-beans:jar:6.2.12:compile
[INFO]    |  \- io.micrometer:micrometer-observation:jar:1.15.5:compile
[INFO]    |     \- io.micrometer:micrometer-commons:jar:1.15.5:compile
[INFO]    \- org.springframework:spring-webmvc:jar:6.2.12:compile
[INFO]       +- org.springframework:spring-aop:jar:6.2.12:compile
[INFO]       +- org.springframework:spring-context:jar:6.2.12:compile
[INFO]       \- org.springframework:spring-expression:jar:6.2.12:compile

## B. spring.factories analysis

WebMvcAutoConfiguration is enabled only by classpath conditions (Servlet, DispatcherServlet, WebMvcConfigurer) and by the absence of a user-defined WebMvcConfigurationSupport. There is no property key that enables it.

org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration

So spring.factories sticks around in Boot 3 because Boot still needs an SPI registry, while spring.components is a separate, framework-level optimization for component scanning, not a replacement for all uses of spring.factories.

## C. Key auto-configuration classes explained

WebMvcAutoConfiguration conditions: Applied only when servlet + MVC classes are present and no custom WebMvcConfigurationSupport exists.

WebMvcAutoConfigurationAdapter role: Applies Boot’s MVC defaults by implementing WebMvcConfigurer.

Where DispatcherServlet is created: In DispatcherServletAutoConfiguration as the dispatcherServlet bean.

How Boot chooses Tomcat: Only Tomcat classes are on the classpath, so its @ConditionalOnClass + @ConditionalOnMissingBean block wins.

Exact condition: @ConditionalOnClass({Servlet.class, Tomcat.class, UpgradeProtocol.class}) with @ConditionalOnMissingBean(ServletWebServerFactory.class).

Why you get a ThreadPoolTaskExecutor: Because TaskExecutionAutoConfiguration runs and no Executor bean is defined.

## D. Actual bean creation order

dispatcherServlet
dispatcherServletRegistration
tomcatServletWebServerFactory
tomcatServletWebServerFactoryCustomizer
tomcatWebServerFactoryCustomizer

## E. How to make Netty win by default (my own starter)

No clue.