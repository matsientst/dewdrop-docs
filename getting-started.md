---
layout: default
title: Getting Started
nav_order: 2
description: "Getting started with dewdrop"
permalink: /getting-started/
---

# Getting Started
First, you'll need add dewdrop to your pom:

```xml
<dependency>
  <groupId>events.dewdrop</groupId>
  <artifactId>dewdrop</artifactId>
  <version>1.0.0</version>
</dependency>
```

Next, you need to make sure you're running EventStore locally. To do this you can go download the [EventStore](https://www.eventstore.com/downloads) client and run it. Or, you can run a docker instance which is included in the repository.
To start the docker instance, run the following command in the dewdrop directory:

`docker-compose up -d`

We are also assuming that most people getting going with the project are running it inside a dependency injected framework like Spring Boot. If this is the case you need to create a class that wraps your DI application context. For the case of Spring Boot you'd create a class that implements `DependencyInjectionAdapter` and expose it as a bean.

```java
public class DewdropDependencyInjection implements DependencyInjectionAdapter {
        private ApplicationContext applicationContext;
        public DewdropDependencyInjection(ApplicationContext applicationContext) {
                this.applicationContext = applicationContext;
        }

        @Override
        public <T> T getBean(Class<?> clazz) {
            return (T) applicationContext.getBean(clazz);
        }
}
```
This lets Dewdrop know that it should use the application context to get the spring managed beans it needs.

The next step is to create a `DewdropConfiguration` class that will be used to configure the Dewdrop framework.

```java
import java.beans.BeanProperty;

public class DewdropConfiguration {
    @Autowired
    ApplicationContext applicationContext;
    
    @Bean 
    public DewdropDependencyInjection dependencyInjection() {
        return new DewdropDependencyInjection(applicationContext);
    }
    
    @Bean
    public DewdropProperties dewdropProperties() {
        return DewdropProperties.builder()
            .packageToScan("events.dewdrop")
            .packageToExclude("events.dewdrop.fixture.customized")
            .connectionString("esdb://localhost:2113?tls=false")
            .create();
    }
    
    @Bean 
    public Dewdrop dewdrop() {
        return DewdropSettings.builder()
            .properties(dewdropProperties())
            .dependencyInjectionAdapter(dependencyInjection())
            .create()
            .start();
    }
}
``` 
And that is it! You can now run the application and it will start up the Dewdrop framework.

### Next, learn how to use Dewdrop:

[Using Dewdrop](/using-dewdrop){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }