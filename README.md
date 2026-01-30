# 🚪 HealthConnect API Gateway

This service acts as the single **Entry Point** for the **HealthConnect** ecosystem. It utilizes **Spring Cloud Gateway** to centralize traffic, manage security, and route requests to the corresponding microservices (Java, Go, and Python).

![HealthConnect Architecture](./assets/dapi-gateway.jpg)

---

## 📊 Routing Architecture

The Gateway is configured to listen on port `8080` and orchestrate calls to the following services:

| Service | Technology | Port | Responsibility | Route Prefixes |
| :--- | :--- | :--- | :--- | :--- |
| **Patient Service** | Java (Spring) | `8081` | Data core and AES encryption. | `/api/v1/patients/**`, `/api/v1/doctype/**`, `/api/v1/medical-records/**` |
| **Storage Service** | Go | `8082` | File management and Vault storage. | `/api/v1/files/**` |
| **Audit Hub** | Python | `8000` | Audit logging and anomaly detection. | `/api/v1/audit/**` (StripPrefix=2) |

---

## 🛠️ Key Configuration

Routing is managed declaratively. A critical detail is the integration with the Python service (**Audit Hub**), where a path transformation filter is applied:

> **Note on Audit Hub:** The configuration uses `StripPrefix=2`. 
> Example: A request to `localhost:8080/api/v1/audit/logs` is redirected to `localhost:8000/audit/logs`.

```yaml
server:
  port: 8080

spring:
  application:
    name: healthconnect-api-gateway
  cloud:
    gateway:
      routes:
        - id: patient-service
          uri: http://localhost:8081
          predicates:
            - Path=/api/v1/patients/**, /api/v1/doctype/**, /api/v1/medical-records/**, /api/v1/replication/**

        - id: storage-service
          uri: http://localhost:8082
          predicates:
            - Path=/api/v1/files/**

        - id: audit-hub-service
          uri: http://localhost:8000
          predicates:
            - Path=/api/v1/audit/**
          filters:
            - StripPrefix=2