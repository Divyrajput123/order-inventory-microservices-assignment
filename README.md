# Order Service

A Spring Boot microservice for processing product orders with inventory management integration.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [API Documentation](#api-documentation)
- [Testing Instructions](#testing-instructions)
- [Architecture](#architecture)
- [Configuration](#configuration)

## Overview

The Order Service is a microservice that handles order placement and communicates with an Inventory Service to check product availability and update stock levels. It uses Spring Boot, Spring Data JPA, and H2 in-memory database.

### Features

- Order placement with inventory validation
- Integration with Inventory Service via REST API
- Order status tracking (CONFIRMED, FAILED, INSUFFICIENT_STOCK)
- H2 in-memory database for persistence
- Comprehensive unit and integration tests

## Prerequisites

Before you begin, ensure you have the following installed:

- **Java 25** (or compatible version)
- **Gradle 8.x** or higher (or use the Gradle Wrapper included in the project)
- **IDE** (IntelliJ IDEA, Eclipse, or VS Code recommended)

## Project Setup

### 1. Clone or Navigate to the Project

```bash
cd Order
```

### 2. Build the Project

Using Gradle Wrapper (recommended):

```bash
# On Unix/macOS
./gradlew build

# On Windows
gradlew.bat build
```

### 3. Run the Application

```bash
# Using Gradle Wrapper
./gradlew bootRun

# Or build and run the JAR
./gradlew build
java -jar build/libs/Order-0.0.1-SNAPSHOT.jar
```

The application will start on **http://localhost:8080**

### 4. Access H2 Database Console

Once the application is running, you can access the H2 console at:
- **URL**: http://localhost:8080/h2-console
- **JDBC URL**: `jdbc:h2:mem:orderdb`
- **Username**: `sa`
- **Password**: (leave empty)

### 5. Inventory Service Configuration

The Order Service communicates with an Inventory Service. By default, it expects the Inventory Service to be running on:
- **URL**: http://localhost:8081

You can configure this in `src/main/resources/application.properties`:

```properties
inventory.service.url=http://localhost:8081
```

**Note**: For testing purposes, the Inventory Service is mocked using `@MockBean` in integration tests.

## API Documentation

### Base URL

```
http://localhost:8080
```

### Endpoints

#### Place an Order

Creates a new order and updates inventory accordingly.

**Endpoint**: `POST /order`

**Request Body**:
```json
{
  "productId": 1,
  "quantity": 10
}
```

**Request Parameters**:
- `productId` (Long, required): The ID of the product to order
- `quantity` (Integer, required): The quantity to order (must be > 0)

**Success Response** (201 Created):
```json
{
  "orderId": 1,
  "productId": 1,
  "quantity": 10,
  "orderDate": "2024-01-15T10:30:00",
  "status": "CONFIRMED",
  "message": "Order placed successfully"
}
```

**Error Responses**:

1. **400 Bad Request** - Invalid request (missing fields, invalid quantity):
```json
{
  "orderId": 1,
  "productId": 1,
  "quantity": 10,
  "orderDate": "2024-01-15T10:30:00",
  "status": "INSUFFICIENT_STOCK",
  "message": "Insufficient stock. Available: 5, Requested: 10"
}
```

2. **400 Bad Request** - Product not found:
```json
{
  "orderId": 1,
  "productId": 1,
  "quantity": 10,
  "orderDate": "2024-01-15T10:30:00",
  "status": "FAILED",
  "message": "Product not found in inventory"
}
```

3. **400 Bad Request** - Inventory update failed:
```json
{
  "orderId": 1,
  "productId": 1,
  "quantity": 10,
  "orderDate": "2024-01-15T10:30:00",
  "status": "FAILED",
  "message": "Failed to update inventory"
}
```

**Order Status Values**:
- `CONFIRMED`: Order successfully placed and inventory updated
- `FAILED`: Order failed due to service error or product not found
- `INSUFFICIENT_STOCK`: Order failed due to insufficient inventory

### Example cURL Requests

**Successful Order**:
```bash
curl -X POST http://localhost:8080/order \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1,
    "quantity": 10
  }'
```

**Invalid Request** (missing quantity):
```bash
curl -X POST http://localhost:8080/order \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1
  }'
```

## Testing Instructions

### Running All Tests

```bash
./gradlew test
```

### Running Specific Test Classes

```bash
# Unit tests for OrderService
./gradlew test --tests OrderServiceTest

# Unit tests for OrderController
./gradlew test --tests OrderControllerTest

# Integration tests
./gradlew test --tests OrderControllerIntegrationTest
```

### Test Structure

The project includes comprehensive test coverage:

#### 1. Unit Tests

**OrderServiceTest** (`src/test/java/com/example/Order/service/OrderServiceTest.java`)
- Tests service layer logic in isolation
- Uses Mockito to mock dependencies (OrderRepository, RestTemplate)
- Covers all business scenarios:
  - Successful order placement
  - Product not found
  - Insufficient stock
  - Inventory update failures
  - Exception handling

**OrderControllerTest** (`src/test/java/com/example/Order/controller/OrderControllerTest.java`)
- Tests controller layer in isolation
- Uses `@WebMvcTest` for lightweight controller testing
- Covers:
  - Successful order placement
  - Failed order scenarios
  - Request validation
  - Error handling

#### 2. Integration Tests

**OrderControllerIntegrationTest** (`src/test/java/com/example/Order/integration/OrderControllerIntegrationTest.java`)
- Full-stack integration test
- Tests complete flow: HTTP Request → Controller → Service → Repository → Database
- Uses `@SpringBootTest` with H2 in-memory database
- Mocks external Inventory Service via `@MockBean`
- Verifies:
  - REST endpoint behavior
  - Service layer logic
  - Database persistence
  - Integration between all layers

### Test Coverage

The test suite covers:
- ✅ All service business logic scenarios
- ✅ All REST endpoints
- ✅ Request validation
- ✅ Error handling
- ✅ Database persistence
- ✅ Inter-service communication (mocked)

### Viewing Test Reports

After running tests, view the HTML report:

```bash
# Generate test report
./gradlew test

# View report (location may vary)
open build/reports/tests/test/index.html
```

## Architecture

### Project Structure

```
src/main/java/com/example/Order/
├── config/
│   └── RestTemplateConfig.java      # RestTemplate bean configuration
├── controller/
│   └── OrderController.java         # REST controller for order endpoints
├── dto/
│   ├── InventoryBatch.java          # DTO for inventory batch data
│   ├── InventoryUpdateRequest.java  # DTO for inventory update requests
│   ├── OrderRequest.java            # DTO for order requests
│   └── OrderResponse.java           # DTO for order responses
├── model/
│   └── Order.java                   # Order entity (JPA)
├── repository/
│   └── OrderRepository.java         # Spring Data JPA repository
├── service/
│   └── OrderService.java            # Business logic for order processing
└── OrderApplication.java            # Spring Boot main class

src/test/java/com/example/Order/
├── controller/
│   └── OrderControllerTest.java     # Controller unit tests
├── integration/
│   └── OrderControllerIntegrationTest.java  # Full-stack integration tests
└── service/
    └── OrderServiceTest.java        # Service unit tests
```

### Architecture Layers

1. **Controller Layer**: Handles HTTP requests and responses
2. **Service Layer**: Contains business logic and orchestrates operations
3. **Repository Layer**: Data access layer using Spring Data JPA
4. **Model Layer**: JPA entities representing database tables
5. **DTO Layer**: Data Transfer Objects for API communication

### Flow Diagram

```
Client Request
    ↓
OrderController (validates request)
    ↓
OrderService (business logic)
    ↓
RestTemplate → Inventory Service (check availability)
    ↓
OrderService (update inventory)
    ↓
OrderRepository (save order)
    ↓
H2 Database (persist order)
    ↓
Response to Client
```

## Configuration

### Application Properties

Key configuration in `src/main/resources/application.properties`:

```properties
# Server Configuration
server.port=8080

# H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:orderdb
spring.datasource.username=sa
spring.datasource.password=

# JPA Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Inventory Service URL
inventory.service.url=http://localhost:8081
```

### Environment Variables

You can override configuration using environment variables:

```bash
# Change server port
export SERVER_PORT=8082

# Change inventory service URL
export INVENTORY_SERVICE_URL=http://inventory-service:8081
```

## Dependencies

Key dependencies used in this project:

- **Spring Boot 3.5.7**: Core framework
- **Spring Data JPA**: Database access
- **H2 Database**: In-memory database
- **Lombok**: Reduces boilerplate code
- **JUnit 5**: Testing framework
- **Mockito**: Mocking framework for tests

## Troubleshooting

### Common Issues

1. **Port Already in Use**
   - Change `server.port` in `application.properties`
   - Or kill the process using port 8080

2. **Inventory Service Connection Failed**
   - Ensure Inventory Service is running on port 8081
   - Check `inventory.service.url` configuration
   - For testing, integration tests mock the Inventory Service

3. **Database Connection Issues**
   - H2 is in-memory, so data is lost on restart
   - Access H2 console at http://localhost:8080/h2-console
   - Verify JDBC URL: `jdbc:h2:mem:orderdb`

4. **Test Failures**
   - Ensure all dependencies are downloaded: `./gradlew build --refresh-dependencies`
   - Check Java version compatibility (Java 25)

## License

This project is part of a microservices demonstration.

## Contact

For questions or issues, please refer to the project documentation or create an issue in the repository.

