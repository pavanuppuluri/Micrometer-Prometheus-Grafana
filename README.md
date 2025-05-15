# Micrometer-Prometheus-Grafana

Here is a complete end-to-end setup for **integrating Spring Boot, Micrometer, Prometheus, and Grafana** using a 
HotelBookingApp as an example. This includes configuration files, dependencies, Docker setup, and Docker Compose orchestration.

## 1. Spring Boot App Setup

✅ pom.xml (Micrometer + Actuator)

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
</dependencies>
```
✅ application.yml

```yml
management:
  endpoints:
    web:
      exposure:
        include: "health,info,prometheus"
  endpoint:
    prometheus:
      enabled: true
metrics:
  export:
    prometheus:
      enabled: true
server:
  port: 8080
```

**Controller Example**

```java
@RestController
public class BookingController {

    private final Counter requestCounter;

    public BookingController(MeterRegistry registry) {
        this.requestCounter = registry.counter("booking_requests_total");
    }

    @GetMapping("/book")
    public String bookRoom() {
        requestCounter.increment();
        return "Room booked successfully!";
    }
}
```

## 2. Dockerize Spring Boot App

✅ Dockerfile

```Dockerfile
CopyEdit
FROM openjdk:17-jdk-slim
ARG JAR_FILE=target/hotel-booking-app.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

## 3. Prometheus Config

✅ prometheus.yml

```yaml
global:
scrape_interval: 15s
evaluation_interval: 15s
scrape_configs:
- job_name: 'hotel-booking-app'
metrics_path: '/actuator/prometheus'
static_configs:
- targets: ['hotel-booking-app:8080']
```



