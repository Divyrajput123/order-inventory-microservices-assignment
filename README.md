# Inventory Service

A Spring Boot microservice for managing inventory with batch tracking and expiry date management. The service implements the Factory Design Pattern for extensible inventory handling strategies.

## Features

- Product and inventory batch management
- Batch tracking with expiry dates
- FIFO (First In First Out) inventory handling based on expiry dates
- Factory Pattern implementation for extensible inventory strategies
- RESTful API endpoints
- H2 in-memory database
- Comprehensive unit and integration tests

## Prerequisites

- Java 17 or higher (Note: The project is configured for Java 25, but Java 17+ will work)
- Gradle 7.x or higher (or use the included Gradle Wrapper)

## Project Setup

### 1. Clone the Repository

```bash
git clone <repository-url>
cd Inventory
```

### 2. Build the Project

Using Gradle Wrapper (recommended):

```bash
# On Unix/Mac
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
java -jar build/libs/Inventory-0.0.1-SNAPSHOT.jar
```

The application will start on `http://localhost:8080` by default.

### 4. Access H2 Console

Once the application is running, you can access the H2 database console at:
- URL: `http://localhost:8080/h2-console`
- JDBC URL: `jdbc:h2:mem:inventorydb`
- Username: `sa`
- Password: (leave empty)

## API Documentation

### Base URL

```
http://localhost:8080
```

### Endpoints

#### 1. Get Inventory Batches by Product ID

Returns all inventory batches for a product, sorted by expiry date (earliest first).

**Endpoint:** `GET /inventory/{productId}`

**Path Parameters:**
- `productId` (Long, required) - The ID of the product

**Response:**
- **200 OK** - Returns list of inventory batches
- **404 Not Found** - Product not found
- **500 Internal Server Error** - Server error

**Example Request:**
```bash
curl -X GET http://localhost:8080/inventory/1
```

**Example Response:**
```json
[
  {
    "id": 1,
    "product": {
      "id": 1,
      "name": "Product A",
      "description": "Sample product"
    },
    "quantity": 10,
    "expiryDate": "2024-12-31",
    "batchNumber": "BATCH001",
    "receivedDate": "2024-01-15"
  },
  {
    "id": 2,
    "product": {
      "id": 1,
      "name": "Product A",
      "description": "Sample product"
    },
    "quantity": 20,
    "expiryDate": "2025-01-31",
    "batchNumber": "BATCH002",
    "receivedDate": "2024-01-20"
  }
]
```

#### 2. Update Inventory

Updates inventory after an order is placed. Uses the Factory Pattern to select the appropriate inventory handler strategy.

**Endpoint:** `POST /inventory/update`

**Request Body:**
```json
{
  "productId": 1,
  "quantity": 15,
  "handlerType": "STANDARD"
}
```

**Request Fields:**
- `productId` (Long, required) - The ID of the product
- `quantity` (Integer, required) - The quantity to deduct from inventory
- `handlerType` (String, optional) - The inventory handler strategy to use. Defaults to "STANDARD" if not provided

**Response:**
- **200 OK** - Returns list of updated inventory batches
- **400 Bad Request** - Invalid request or insufficient inventory
- **500 Internal Server Error** - Server error

**Example Request:**
```bash
curl -X POST http://localhost:8080/inventory/update \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1,
    "quantity": 15,
    "handlerType": "STANDARD"
  }'
```

**Example Response:**
```json
[
  {
    "id": 1,
    "product": {
      "id": 1,
      "name": "Product A"
    },
    "quantity": 0,
    "expiryDate": "2024-12-31",
    "batchNumber": "BATCH001",
    "receivedDate": "2024-01-15"
  },
  {
    "id": 2,
    "product": {
      "id": 1,
      "name": "Product A"
    },
    "quantity": 15,
    "expiryDate": "2025-01-31",
    "batchNumber": "BATCH002",
    "receivedDate": "2024-01-20"
  }
]
```

**Error Responses:**

Insufficient Inventory:
```json
"Insufficient inventory. Requested: 100, Available: 30"
```

Missing Required Fields:
```json
"Product ID and quantity are required"
```

Product Not Found:
```json
"Product not found with id: 999"
```

## Architecture

### Factory Pattern Implementation

The service uses the Factory Design Pattern to allow extensible inventory handling strategies:

- **`InventoryHandler`** - Interface defining inventory processing strategies
- **`StandardInventoryHandler`** - FIFO implementation based on expiry dates
- **`InventoryHandlerFactory`** - Factory for creating and managing handlers

To add a new inventory strategy:
1. Implement the `InventoryHandler` interface
2. Annotate with `@Component`
3. The factory will automatically register it

### Project Structure

```
src/
├── main/
│   ├── java/com/example/Inventory/
│   │   ├── controller/          # REST controllers
│   │   ├── service/             # Business logic
│   │   ├── repository/          # Data access layer
│   │   ├── entity/              # JPA entities
│   │   ├── factory/             # Factory pattern implementation
│   │   └── dto/                 # Data transfer objects
│   └── resources/
│       └── application.properties
└── test/
    └── java/com/example/Inventory/
        ├── controller/          # Integration tests
        ├── service/             # Service tests
        └── factory/             # Factory pattern tests
```

## Testing Instructions

### Running All Tests

```bash
./gradlew test
```

### Running Specific Test Classes

```bash
# Unit tests
./gradlew test --tests "InventoryServiceTest"
./gradlew test --tests "StandardInventoryHandlerTest"

# Integration tests
./gradlew test --tests "InventoryControllerIntegrationTest"
./gradlew test --tests "InventoryServiceIntegrationTest"
```

### Test Coverage

The project includes:

1. **Unit Tests** (JUnit 5 + Mockito):
   - `InventoryServiceTest` - Service layer unit tests
   - `StandardInventoryHandlerTest` - Handler logic tests
   - `InventoryHandlerFactoryTest` - Factory pattern tests

2. **Integration Tests** (@SpringBootTest):
   - `InventoryControllerIntegrationTest` - REST endpoint integration tests
   - `InventoryServiceIntegrationTest` - Service integration tests with H2 database

### Test Configuration

Tests use a separate H2 in-memory database configured in `src/test/resources/application-test.properties`. The test database is created fresh for each test run and cleaned up afterward.

### Viewing Test Results

After running tests, view the HTML report:
```bash
# Open in browser
open build/reports/tests/test/index.html
```

## Database Schema

### Products Table
- `id` (Long, Primary Key)
- `name` (String, Unique, Not Null)
- `description` (String)

### Inventory Batches Table
- `id` (Long, Primary Key)
- `product_id` (Long, Foreign Key)
- `quantity` (Integer, Not Null)
- `expiry_date` (LocalDate, Not Null)
- `batch_number` (String, Not Null)
- `received_date` (LocalDate, Not Null)

## Configuration

### Application Properties

Key configuration in `application.properties`:

- **Database**: H2 in-memory database
- **JPA**: Auto-update schema, SQL logging enabled
- **H2 Console**: Enabled at `/h2-console`

### Port Configuration

To change the server port, add to `application.properties`:
```properties
server.port=8081
```

## Development

### Adding New Inventory Handlers

1. Create a new class implementing `InventoryHandler`:
```java
@Component
public class CustomInventoryHandler implements InventoryHandler {
    @Override
    public List<InventoryBatch> processInventoryUpdate(
        List<InventoryBatch> batches, 
        Integer requestedQuantity
    ) {
        // Your custom logic here
    }
    
    @Override
    public String getHandlerType() {
        return "CUSTOM";
    }
}
```

2. The factory will automatically discover and register it
3. Use it by specifying `"handlerType": "CUSTOM"` in update requests

## Troubleshooting

### Build Issues

If you encounter Java version errors:
1. Update `build.gradle` line 13 to use a supported Java version (e.g., `JavaLanguageVersion.of(17)`)
2. Ensure your JAVA_HOME points to the correct Java version

### Database Connection Issues

- Verify H2 console is accessible at `http://localhost:8080/h2-console`
- Check that the JDBC URL matches: `jdbc:h2:mem:inventorydb`

### Test Failures

- Ensure all dependencies are downloaded: `./gradlew clean build`
- Check that the test profile is being used correctly

## License

This project is part of a microservices architecture demonstration.

