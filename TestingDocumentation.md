# Testing Documentation

This document provides comprehensive documentation of all tests implemented in the e-commerce microservices application, including End-to-End (E2E) tests, Unit tests, Integration tests, and Performance tests.

## Table of Contents

1. [End-to-End (E2E) Tests](#end-to-end-e2e-tests)
2. [Unit Tests](#unit-tests)
3. [Integration Tests](#integration-tests)
4. [Performance Tests with Locust](#performance-tests-with-locust)
5. [Test Execution](#test-execution)
6. [CI/CD Integration](#cicd-integration)

---

## End-to-End (E2E) Tests

E2E tests validate complete user workflows across multiple services, ensuring the entire system works together correctly.

### Location
- **Path**: `src/test/java/com/selimhorri/app/e2e/`
- **Framework**: Spring Boot Test with `TestRestTemplate`
- **Profile**: `test`

### Test Classes

#### 1. CompleteShoppingFlowE2ETest
**File**: `CompleteShoppingFlowE2ETest.java`

**Purpose**: Validates the complete shopping flow from user registration to payment processing.

**Test Flow**:
1. Register a new user
2. Browse available products
3. Add product to favourites
4. Create an order from a cart
5. Process payment for the order

**Key Assertions**:
- User registration succeeds
- Products can be browsed
- Favourites can be added
- Orders can be created
- Payments can be processed

**Test Method**:
- `testCompleteShoppingFlow()` - Complete end-to-end shopping experience

---

#### 2. OrderCreationAndPaymentE2ETest
**File**: `OrderCreationAndPaymentE2ETest.java`

**Purpose**: Validates the order creation and payment processing flow.

**Test Flows**:

**Flow 1: Order Creation and Payment**
1. Create a cart (or use existing cart)
2. Create an order from the cart
3. Update order status to `ORDERED`
4. Create payment for the order
5. Verify order status updates to `IN_PAYMENT`

**Flow 2: Order Cancellation**
1. Create an order
2. Cancel the order (soft delete)
3. Verify order is not in active orders list

**Test Methods**:
- `testOrderCreationAndPaymentFlow()` - Complete order to payment flow
- `testOrderCancellationFlow()` - Order cancellation workflow

**Key Assertions**:
- Orders can be created successfully
- Order status can be updated
- Payments can be created for orders
- Orders can be cancelled
- Cancelled orders are excluded from active lists

---

#### 3. ProductBrowseAndFavouriteE2ETest
**File**: `ProductBrowseAndFavouriteE2ETest.java`

**Purpose**: Validates product browsing and favourite management functionality.

**Test Flow**:
1. Create a user
2. Browse all available products
3. Add a product to favourites
4. Retrieve user's favourites list

**Test Method**:
- `testProductBrowseAndFavouriteFlow()` - Product browsing and favourites workflow

**Key Assertions**:
- Products can be browsed
- Products can be added to favourites
- User favourites can be retrieved
- All operations return successful HTTP status codes

---

#### 4. UserProfileManagementE2ETest
**File**: `UserProfileManagementE2ETest.java`

**Purpose**: Validates user profile creation and update functionality.

**Test Flows**:

**Flow 1: Profile Update**
1. Create a new user
2. Update user profile information
3. Retrieve user to verify changes

**Flow 2: Profile Update by ID**
1. Create a user
2. Update user by ID
3. Verify changes are persisted

**Test Methods**:
- `testUserProfileUpdateFlow()` - Complete profile update workflow
- `testUserProfileUpdateByIdFlow()` - Profile update by ID workflow

**Key Assertions**:
- Users can be created
- User profiles can be updated
- Updates are persisted correctly
- User data can be retrieved after updates

---

#### 5. UserRegistrationAndLoginE2ETest
**File**: `UserRegistrationAndLoginE2ETest.java`

**Purpose**: Validates user registration and authentication flow.

**Test Flow**:
1. Register a new user
2. Create credentials for the user
3. Perform login with credentials

**Test Method**:
- `testUserRegistrationAndLoginFlow()` - Registration and login workflow

**Key Assertions**:
- Users can be registered
- Credentials can be created
- Login functionality works correctly

---

### Running E2E Tests

```bash
# Run all E2E tests
mvn test -Dtest=*E2ETest

# Run specific E2E test
mvn test -Dtest=CompleteShoppingFlowE2ETest

# Run with test profile
mvn test -Dspring.profiles.active=test
```

---

## Unit Tests

Unit tests validate individual service methods in isolation using mocks and stubs.

### Framework
- **JUnit 5** (`@Test`, `@DisplayName`)
- **Mockito** (`@Mock`, `@InjectMocks`, `@ExtendWith(MockitoExtension.class)`)
- **Assertions**: JUnit 5 assertions

### Test Classes by Service

#### User Service

**Location**: `user-service/src/test/java/com/selimhorri/app/service/UserServiceUnitTest.java`

**Test Coverage**:
- `testFindById_Success()` - Find user by ID when user exists
- `testFindById_UserNotFound()` - Throw exception when user not found
- `testFindById_UserWithoutCredentials()` - Throw exception when user has no credentials
- `testFindByUsername_Success()` - Find user by username
- `testFindByUsername_NotFound()` - Throw exception when username not found
- `testFindAll_Success()` - Find all users with credentials
- `testFindAll_FiltersUsersWithoutCredentials()` - Filter users without credentials
- `testSave_Success()` - Save new user successfully
- `testUpdate_Success()` - Update user successfully
- `testDeleteById_Success()` - Delete user and credentials successfully
- `testDeleteById_UserWithoutCredentials()` - Throw exception when deleting user without credentials

**Mocked Dependencies**:
- `UserRepository`
- `CredentialRepository`

**Key Test Patterns**:
- Given-When-Then structure
- Mock verification
- Exception testing
- Edge case handling

---

#### Order Service

**Location**: `order-service/src/test/java/com/selimhorri/app/service/OrderServiceUnitTest.java`

**Test Coverage**:
- `testFindById_Success()` - Find order by ID when order exists and is active
- `testFindById_OrderNotFound()` - Throw exception when order not found
- `testFindAll_Success()` - Find all active orders
- `testSave_Success()` - Save order successfully when cart exists
- `testSave_WithoutCart()` - Throw exception when saving order without cart
- `testSave_CartNotFound()` - Throw exception when cart not found
- `testUpdateStatus_CreatedToOrdered()` - Update order status from CREATED to ORDERED
- `testUpdateStatus_OrderedToInPayment()` - Update order status from ORDERED to IN_PAYMENT
- `testUpdateStatus_InPayment_ThrowsException()` - Throw exception when updating status of order in payment
- `testDeleteById_Success()` - Delete order successfully when status allows
- `testDeleteById_InPayment_ThrowsException()` - Throw exception when deleting order in payment
- `testUpdate_Success()` - Update order successfully

**Mocked Dependencies**:
- `OrderRepository`
- `CartRepository`

**Key Test Scenarios**:
- Order lifecycle management
- Status transitions
- Business rule validation
- Error handling

---

#### Favourite Service

**Location**: `favourite-service/src/test/java/com/selimhorri/app/service/FavouriteServiceUnitTest.java`

**Test Coverage**:
- `testFindById_Success()` - Find favourite by ID when favourite exists
- `testFindById_FavouriteNotFound()` - Throw exception when favourite not found
- `testFindAll_Success()` - Find all favourites with user and product details
- `testFindAll_FiltersInvalidFavourites()` - Filter out favourites when user or product not found
- `testSave_Success()` - Save favourite successfully when user and product exist
- `testSave_UserNotFound()` - Throw exception when user not found during save
- `testSave_ProductNotFound()` - Throw exception when product not found during save
- `testSave_DuplicateFavourite()` - Throw exception when favourite already exists
- `testDeleteById_Success()` - Delete favourite successfully
- `testDeleteById_FavouriteNotFound()` - Throw exception when deleting non-existent favourite

**Mocked Dependencies**:
- `FavouriteRepository`
- `RestTemplate` (for external service calls to User and Product services)

**Key Test Scenarios**:
- Cross-service communication
- Duplicate prevention
- External service error handling
- Data filtering

---

#### User Resource (Controller)

**Location**: `user-service/src/test/java/com/selimhorri/app/resource/UserResourceUnitTest.java`

**Test Coverage**: Controller-level unit tests for REST endpoints.

---

### Running Unit Tests

```bash
# Run all unit tests
mvn test

# Run tests for a specific service
cd user-service && mvn test

# Run specific test class
mvn test -Dtest=UserServiceUnitTest

# Run with coverage
mvn test jacoco:report
```

---

## Integration Tests

Integration tests validate service interactions with real repositories and databases.

### Framework
- **Spring Boot Test** (`@SpringBootTest`)
- **Test Containers** (for database testing)
- **Active Profile**: `test`
- **Transaction Management**: `@Transactional`

### Test Classes by Service

#### User Service Integration

**Location**: `user-service/src/test/java/com/selimhorri/app/service/UserServiceIntegrationTest.java`

**Test Coverage**:
- `testSave_IntegrationWithRepositories()` - Integrate with UserRepository and CredentialRepository to save user
- `testFindById_IntegrationWithRepositories()` - Integrate with repositories to find user with credentials
- `testFindAll_IntegrationWithRepositories_FiltersUsersWithoutCredentials()` - Filter users without credentials
- `testDeleteById_IntegrationWithRepositories()` - Integrate with repositories to delete user and credentials
- `testUpdate_IntegrationWithRepositories()` - Integrate with repositories to update user information

**Key Features**:
- Uses real database (test profile)
- Tests repository interactions
- Validates data persistence
- Tests transaction handling

---

#### Order Service Integration

**Location**: `order-service/src/test/java/com/selimhorri/app/service/OrderServiceIntegrationTest.java`

**Test Coverage**: Integration tests for order service with real repositories.

---

#### Favourite Service Integration

**Location**: `favourite-service/src/test/java/com/selimhorri/app/service/FavouriteServiceIntegrationTest.java`

**Test Coverage**: Integration tests for favourite service with real repositories.

---

#### Payment Service Integration

**Location**: `payment-service/src/test/java/com/selimhorri/app/service/PaymentServiceIntegrationTest.java`

**Test Coverage**: Integration tests for payment service with real repositories.

---

#### Shipping Service Integration

**Location**: `shipping-service/src/test/java/com/selimhorri/app/service/ShippingServiceIntegrationTest.java`

**Test Coverage**: Integration tests for shipping service with real repositories.

---

### Running Integration Tests

```bash
# Run all integration tests
mvn test -Dtest=*IntegrationTest

# Run with test profile
mvn test -Dspring.profiles.active=test

# Run specific integration test
mvn test -Dtest=UserServiceIntegrationTest
```

---

## Performance Tests with Locust

Performance tests using Locust to simulate load and measure system performance under various conditions.

### Location
- **Path**: `ci-pipelines/performance-tests/`
- **File**: `locustfile.py`
- **Requirements**: `requirements.txt`
- **Documentation**: `README.md`

### Prerequisites

```bash
# Install Python 3.7+
# Install Locust
pip install -r ci-pipelines/performance-tests/requirements.txt
```

### Test Scenarios

#### 1. EcommerceUser (Main User Class)
**Purpose**: Simulates typical e-commerce user behavior with realistic task distribution.

**Task Distribution**:
- User Registration: 10%
- Product Browsing: 40%
- Favourite Management: 20%
- Order Creation: 20%
- Payment Processing: 10%

**Wait Time**: 1-3 seconds between tasks

**Use Cases**:
- Real-world user simulation
- Balanced load distribution
- Typical shopping patterns

---

#### 2. HighLoadUser
**Purpose**: Stress testing with rapid API calls to test system resilience.

**Characteristics**:
- Very short wait time (0.1-0.5 seconds)
- Rapid API calls
- Tests system under extreme load

**Tasks**:
- Frequent product browsing (weight: 3)
- Frequent product viewing (weight: 2)
- User listing (weight: 1)

**Use Cases**:
- Stress testing
- Load capacity testing
- System resilience validation

---

#### 3. ShoppingFlowUser
**Purpose**: Complete shopping flow simulation from registration to payment.

**Flow**:
1. Register user
2. Browse products
3. Create order
4. Process payment

**Wait Time**: 2-5 seconds between tasks

**Use Cases**:
- End-to-end performance testing
- Complete user journey validation
- Sequential workflow testing

---

#### 4. UserServicePerformanceTest
**Purpose**: Focused performance testing on user service endpoints.

**Tasks**:
- Get all users (weight: 5)
- Get user by ID (weight: 3)
- Create user (weight: 2)

**Wait Time**: 0.5-2 seconds

**Use Cases**:
- Service-specific performance testing
- User service optimization
- CRUD operation performance

---

#### 5. OrderServicePerformanceTest
**Purpose**: Focused performance testing on order service endpoints.

**Tasks**:
- Get all orders (weight: 5)
- Get order by ID (weight: 3)
- Create order (weight: 2)

**Wait Time**: 0.5-2 seconds

**Use Cases**:
- Order service optimization
- Order processing performance
- Database query performance

---

#### 6. UserRegistrationTaskSet
**Purpose**: Sequential task set for user registration flow.

**Tasks**:
- Register new user with random data

---

#### 7. ProductBrowseTaskSet
**Purpose**: Sequential task set for product browsing.

**Tasks**:
- Browse all products
- View specific product by ID

---

#### 8. FavouriteManagementTaskSet
**Purpose**: Sequential task set for managing favourites.

**Tasks**:
- Add product to favourites
- View user favourites

---

#### 9. OrderCreationTaskSet
**Purpose**: Sequential task set for order creation.

**Tasks**:
- Create new order
- View all orders

---

#### 10. PaymentProcessingTaskSet
**Purpose**: Sequential task set for payment processing.

**Tasks**:
- Create payment for order
- View all payments

---

### Running Performance Tests

#### Basic Usage

```bash
# Run with default settings
locust -f ci-pipelines/performance-tests/locustfile.py --host=http://localhost:8080
```

#### With Web UI

```bash
# Access Locust web UI
locust -f ci-pipelines/performance-tests/locustfile.py \
  --host=http://localhost:8080 \
  --web-host=0.0.0.0 \
  --web-port=8089
```

Then open browser to: `http://localhost:8089`

#### Headless Mode (No UI)

```bash
# Run without web UI
locust -f ci-pipelines/performance-tests/locustfile.py \
  --host=http://localhost:8080 \
  --headless \
  -u 100 \
  -r 10 \
  -t 60s
```

**Parameters**:
- `-u 100`: 100 concurrent users
- `-r 10`: 10 users per second spawn rate
- `-t 60s`: Run for 60 seconds

#### Specific Test Scenarios

```bash
# High Load Stress Test
locust -f ci-pipelines/performance-tests/locustfile.py \
  --host=http://localhost:8080 \
  -u 1000 \
  -r 50 \
  -t 5m \
  --class HighLoadUser

# Shopping Flow Test
locust -f ci-pipelines/performance-tests/locustfile.py \
  --host=http://localhost:8080 \
  -u 50 \
  -r 5 \
  -t 10m \
  --class ShoppingFlowUser

# User Service Performance Test
locust -f ci-pipelines/performance-tests/locustfile.py \
  --host=http://localhost:8080 \
  -u 200 \
  -r 20 \
  -t 5m \
  --class UserServicePerformanceTest

# Order Service Performance Test
locust -f ci-pipelines/performance-tests/locustfile.py \
  --host=http://localhost:8080 \
  -u 200 \
  -r 20 \
  -t 5m \
  --class OrderServicePerformanceTest
```

### Performance Metrics

The tests measure:
- **Response Time**: Average, min, max response times
- **Requests per Second (RPS)**: Throughput measurement
- **Failure Rate**: Percentage of failed requests
- **User Count**: Number of concurrent users
- **Request Distribution**: Breakdown by endpoint

### Expected Results

#### Normal Load
- Response time < 200ms for 95% of requests
- Failure rate < 1%
- RPS > 100

#### High Load
- Response time < 1s for 95% of requests
- Failure rate < 5%
- RPS > 500

#### Stress Test
- System remains responsive
- Graceful degradation under extreme load
- No memory leaks or crashes

---

## Test Execution

### Running All Tests

```bash
# Run all tests (unit, integration, E2E)
mvn clean test

# Run with coverage
mvn clean test jacoco:report

# Skip tests during build
mvn clean package -DskipTests
```

### Running Tests by Category

```bash
# Unit tests only
mvn test -Dtest=*UnitTest

# Integration tests only
mvn test -Dtest=*IntegrationTest

# E2E tests only
mvn test -Dtest=*E2ETest
```

### Running Tests by Service

```bash
# User service tests
cd user-service && mvn test

# Order service tests
cd order-service && mvn test

# Favourite service tests
cd favourite-service && mvn test
```

### Test Profiles

Tests use the `test` profile for configuration:
- Test database (H2 or TestContainers)
- Mock external services
- Reduced logging
- Test-specific properties

```bash
# Run with test profile
mvn test -Dspring.profiles.active=test
```

---

## CI/CD Integration

### GitHub Actions Workflows

#### E2E Test Execution

**Workflow**: `ci-pipelines/.github/workflows/run-e2e.yml`

**Purpose**: Runs E2E tests against deployed services in Kubernetes.

**Steps**:
1. Authenticate with Azure
2. Get AKS cluster credentials
3. Get API Gateway IP address
4. Verify services in Eureka
5. Update Postman collection with dynamic IP
6. Run Newman (Postman) tests
7. Run K6 performance tests

**Trigger**: 
- Manual workflow dispatch
- Called from PR pipelines

#### Build Pipeline Integration

**Workflow**: `ci-pipelines/.github/workflows/build.yml`

**Includes**:
- Docker image scanning with Trivy
- Unit and integration tests run during build
- Image push to Azure Container Registry

#### PR to Stage Pipeline

**Workflow**: `ci-pipelines/.github/workflows/pr-stage.yaml`

**Includes**:
- Build and deploy to stage environment
- Run E2E tests after deployment
- Verify service health

---

## Test Coverage Summary

### By Test Type

| Test Type | Count | Location |
|-----------|-------|----------|
| E2E Tests | 5 | `src/test/java/com/selimhorri/app/e2e/` |
| Unit Tests | 3+ | `{service}/src/test/java/.../service/*UnitTest.java` |
| Integration Tests | 5+ | `{service}/src/test/java/.../service/*IntegrationTest.java` |
| Performance Tests | 10 scenarios | `ci-pipelines/performance-tests/locustfile.py` |

### By Service

| Service | Unit Tests | Integration Tests |
|---------|------------|-------------------|
| User Service | ✅ | ✅ |
| Order Service | ✅ | ✅ |
| Favourite Service | ✅ | ✅ |
| Payment Service | - | ✅ |
| Shipping Service | - | ✅ |

### Common Issues

#### E2E Tests Failing
- **Issue**: Services not available
- **Solution**: Ensure all services are running and registered in Eureka

#### Integration Tests Failing
- **Issue**: Database connection issues
- **Solution**: Check test profile configuration and database setup

#### Performance Tests Not Running
- **Issue**: Locust not installed or host unreachable
- **Solution**: Install dependencies and verify service URLs

#### Tests Timing Out
- **Issue**: Long-running tests or slow services
- **Solution**: Increase timeout settings or optimize test data

---

## Additional Resources

- [JUnit 5 Documentation](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Locust Documentation](https://docs.locust.io/)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)
- [TestContainers Documentation](https://www.testcontainers.org/)

---

## Maintenance

### Regular Tasks
- Review and update test data
- Update test dependencies
- Add tests for new features
- Review test coverage reports
- Update performance benchmarks

### Test Maintenance Schedule
- **Weekly**: Review failing tests
- **Monthly**: Update test dependencies
- **Quarterly**: Review and update test coverage
- **As needed**: Add tests for new features

---

*Last Updated: 2024*
*Documentation Version: 1.0*

