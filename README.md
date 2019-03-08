# Cloud Foundry Smoke Spring Boot Application

## Purpose

Serve as a [ REALLY DUMB AND SIMPLE ] Smoke application that can be used by Cloud Foundry Platform Operators to validate applications can be bound to services properly. The application has been tweaked to use 256MB of Memory (I haven't done extensive tests to check it reliability over time).

The application is designed to the following services :

- RabbitMQ
- PostgreSQL
- Redis

## Building the application

- `./mvnw clean install`

## Pushing the application to Cloud Foundry

### Creating the services

You might need to adapt those based on your available plans (use `cf marketplace`) :

- `cf create-service p.mysql db-small mysql`
- `cf create-service p-rabbitmq standard rabbit`
- `cf create-service p-redis shared-vm redis`

### Binding

- `cf bind-service cf-smoke-app mysql`
- `cf bind-service cf-smoke-app rabbit`
- `cf bind-service cf-smoke-app redis`

### Pushing

- `cf push`

## Swagger

Navigate to `/swagger-ui.html` to access the Swagger UI

## Actuator

Health is the only Actuator endpoint exposed. Navigate to `/actuator` to view the available sub endpoints.

### Endpoints

| Endpoint                | Result |
|-------------------------|--------|
| /actuator/health        |   `{"status":"UP","details":{"rabbit":{"status":"UP","details":{"version":"3.7.11"}},"diskSpace":{"status":"UP","details":{"total":1073741824,"free":888221696,"threshold":10485760}},"db":{"status":"UP","details":{"database":"MySQL","hello":1}},"redis":{"status":"UP","details":{"version":"5.0.2"}}}}` |
| /actuator/health/rabbit | `{"status":"UP","details":{"version":"3.7.11"}}` |
| /actuator/health/mysql  | `{"status":"UP","details":{"database":"MySQL","hello":1}}` |

### Behavior when one service is down

| Endpoint                | Result | HTTP Return Code
|-------------------------|--------|-------------------|
| /actuator/health        |  `{"status":"DOWN","details":{"rabbit":{"status":"DOWN","details":{"error":"org.springframework.amqp.AmqpAuthenticationException: com.rabbitmq.client.AuthenticationFailureException: ACCESS_REFUSED - Login was refused using authentication mechanism PLAIN. For details see the broker logfile."}},"diskSpace":{"status":"UP","details":{"total":1073741824,"free":888221696,"threshold":10485760}},"db":{"status":"UP","details":{"database":"MySQL","hello":1}},"redis":{"status":"UP","details":{"version":"5.0.2"}}}}` | 503 |
| /actuator/health/rabbit  | `{"status":"DOWN","details":{"error":"org.springframework.amqp.AmqpAuthenticationException: com.rabbitmq.client.AuthenticationFailureException: ACCESS_REFUSED - Login was refused using authentication mechanism PLAIN. For details see the broker logfile."}}` | 503 |

### actuator/info

Shows the commit information that the app is using.

`{"build":{"version":"1.0.0","artifact":"cf-smoke-app","name":"Cloud Foundry Smoke Application","group":"com.jmichelgarcia","time":"2019-03-08T15:06:30.094Z"}}`