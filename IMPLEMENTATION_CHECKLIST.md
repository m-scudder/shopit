# Implementation Checklist - Order Management System

## ✅ Phase 1: Infrastructure Setup

### Step 1.1: Start Docker Services
- [✅] Run `docker-compose up -d` to start all services
- [✅] Verify PostgreSQL is running on port 5432
- [✅] Verify MongoDB is running on port 27017
- [✅] Verify Kafka is running on port 9092
- [✅] Verify Redis is running on port 6379
- [✅] Access Kafka UI at http://localhost:8080 (optional)
- [✅] Access Mongo Express at http://localhost:8081 (optional)

### Step 1.2: Update pom.xml
- [✅] Add Spring Data JPA dependency
- [✅] Add PostgreSQL driver dependency
- [✅] Add OAuth2 Resource Server dependency
- [✅] Add Validation dependency
- [✅] Add Web dependency
- [✅] Add OpenFeign dependency (for service communication)
- [✅] Run `mvn clean install` to verify dependencies

### Step 1.3: Configure application.properties
- [✅] Add PostgreSQL connection properties
- [✅] Add MongoDB connection properties
- [✅] Add Kafka bootstrap server configuration
- [✅] Add Redis connection properties (if using)
- [✅] Configure JPA/Hibernate settings

---

## ✅ Phase 2: Authentication Service Setup

### Step 2.1: Database Schema (PostgreSQL)
- [ ] Create `User` entity with fields:
  - id (Long, @Id, @GeneratedValue)
  - username (String, unique, not null)
  - email (String, unique, not null)
  - password (String, not null) - will be encrypted
  - roles (Set<String> or @ManyToMany with Role entity)
  - enabled (Boolean, default true)
  - createdAt (LocalDateTime)
- [ ] Create `Role` entity (optional, or use enum)
- [ ] Create UserRepository interface extending JpaRepository
- [ ] Create custom query methods: findByUsername, findByEmail

### Step 2.2: Security Configuration
- [ ] Create `SecurityConfig` class with @Configuration
- [ ] Configure `SecurityFilterChain` bean
- [ ] Set up `PasswordEncoder` bean (BCryptPasswordEncoder)
- [ ] Configure authentication manager
- [ ] Set up JWT filter for token validation
- [ ] Configure CORS if needed
- [ ] Set up public endpoints (register, login)

### Step 2.3: JWT Utilities
- [ ] Create `JwtTokenUtil` class
- [ ] Implement method to generate JWT token (with username, roles, expiration)
- [ ] Implement method to validate JWT token
- [ ] Implement method to extract username from token
- [ ] Implement method to extract roles from token
- [ ] Add secret key configuration in application.properties

### Step 2.4: Authentication Service Layer
- [ ] Create `UserService` interface and implementation
- [ ] Implement `registerUser()` method:
  - Validate input
  - Check if user exists
  - Encrypt password
  - Save user with default role (ROLE_CUSTOMER)
  - Return user DTO
- [ ] Implement `authenticateUser()` method:
  - Validate credentials
  - Generate JWT token
  - Return token and user info
- [ ] Implement `loadUserByUsername()` for Spring Security

### Step 2.5: Authentication Controller
- [ ] Create `AuthController` with @RestController
- [ ] Create `POST /api/auth/register` endpoint:
  - Accept RegisterRequest DTO
  - Call UserService.registerUser()
  - Return ResponseEntity with success message
- [ ] Create `POST /api/auth/login` endpoint:
  - Accept LoginRequest DTO
  - Call UserService.authenticateUser()
  - Return ResponseEntity with JWT token
- [ ] Add validation annotations (@Valid)
- [ ] Add exception handling (@ExceptionHandler)

### Step 2.6: DTOs and Validation
- [ ] Create `RegisterRequest` DTO:
  - username (with @NotBlank, @Size)
  - email (with @Email, @NotBlank)
  - password (with @NotBlank, @Size(min=8))
- [ ] Create `LoginRequest` DTO:
  - username
  - password
- [ ] Create `AuthResponse` DTO:
  - token
  - username
  - roles
- [ ] Create `UserResponse` DTO (for registration response)

### Step 2.7: Testing Authentication
- [ ] Test user registration via Postman/curl
- [ ] Test user login and verify JWT token is returned
- [ ] Test JWT token validation
- [ ] Test accessing protected endpoint with valid token
- [ ] Test accessing protected endpoint without token (should fail)

---

## ✅ Phase 3: Order Service (SQL - PostgreSQL)

### Step 3.1: Database Schema
- [ ] Create `Order` entity:
  - id (Long)
  - userId (Long, foreign key to users)
  - orderDate (LocalDateTime)
  - status (OrderStatus enum: PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED)
  - totalAmount (BigDecimal)
  - shippingAddress (String or separate Address entity)
  - createdAt, updatedAt (LocalDateTime)
- [ ] Create `OrderItem` entity:
  - id (Long)
  - orderId (Long, @ManyToOne with Order)
  - productId (String, reference to inventory service)
  - quantity (Integer)
  - price (BigDecimal)
  - subtotal (BigDecimal)
- [ ] Create OrderRepository and OrderItemRepository
- [ ] Add custom queries: findByUserId, findByStatus

### Step 3.2: Service Layer
- [ ] Create `OrderService` interface and implementation
- [ ] Implement `createOrder()`:
  - Validate user exists
  - Validate products exist (call inventory service)
  - Calculate total amount
  - Create order and order items
  - Publish "order.created" event to Kafka
  - Return order DTO
- [ ] Implement `getOrderById()`:
  - Find order by id
  - Verify user has access (or is admin)
  - Return order DTO
- [ ] Implement `getUserOrders()`:
  - Find all orders for user
  - Return list of order DTOs
- [ ] Implement `updateOrderStatus()`:
  - Update order status
  - Publish "order.updated" event to Kafka

### Step 3.3: Kafka Integration
- [ ] Create Kafka producer configuration
- [ ] Create `OrderEvent` class (DTO for events)
- [ ] Publish events:
  - order.created
  - order.updated
  - order.cancelled
- [ ] Configure Kafka topic: `order-events`

### Step 3.4: Controller
- [ ] Create `OrderController` with @RestController
- [ ] Add `@PreAuthorize` for role-based access
- [ ] Create `POST /api/orders` - Create order
- [ ] Create `GET /api/orders/{id}` - Get order by ID
- [ ] Create `GET /api/orders` - Get user's orders
- [ ] Create `PUT /api/orders/{id}/status` - Update status (admin only)
- [ ] Add JWT authentication filter

### Step 3.5: DTOs
- [ ] Create `OrderRequest` DTO:
  - List of OrderItemRequest (productId, quantity)
  - shippingAddress
- [ ] Create `OrderResponse` DTO:
  - All order fields
  - List of OrderItemResponse
- [ ] Create `OrderItemRequest` and `OrderItemResponse` DTOs

---

## ✅ Phase 4: Payment Service (SQL - PostgreSQL)

### Step 4.1: Database Schema
- [ ] Create `Payment` entity:
  - id (Long)
  - orderId (Long, foreign key)
  - amount (BigDecimal)
  - paymentMethod (PaymentMethod enum: CREDIT_CARD, DEBIT_CARD, UPI, WALLET)
  - status (PaymentStatus enum: PENDING, PROCESSING, SUCCESS, FAILED, REFUNDED)
  - transactionId (String, unique)
  - paymentDate (LocalDateTime)
  - createdAt, updatedAt
- [ ] Create PaymentRepository
- [ ] Add queries: findByOrderId, findByStatus, findByTransactionId

### Step 4.2: Service Layer
- [ ] Create `PaymentService` interface and implementation
- [ ] Implement `processPayment()`:
  - Validate order exists
  - Validate order amount
  - Create payment record with PENDING status
  - Simulate payment processing (or integrate payment gateway)
  - Update payment status (SUCCESS/FAILED)
  - Publish "payment.processed" event to Kafka
  - Return payment DTO
- [ ] Implement `getPaymentByOrderId()`
- [ ] Implement `refundPayment()`:
  - Validate payment exists and is successful
  - Create refund record
  - Update payment status to REFUNDED
  - Publish "payment.refunded" event

### Step 4.3: Kafka Consumer
- [ ] Create Kafka consumer configuration
- [ ] Create consumer for "order.created" events
- [ ] Auto-trigger payment processing when order is created
- [ ] Handle payment processing asynchronously

### Step 4.4: Controller
- [ ] Create `PaymentController`
- [ ] Create `POST /api/payments` - Process payment
- [ ] Create `GET /api/payments/order/{orderId}` - Get payment by order
- [ ] Create `POST /api/payments/{id}/refund` - Refund payment (admin)

---

## ✅ Phase 5: Inventory Service (NoSQL - MongoDB)

### Step 5.1: MongoDB Document Model
- [ ] Create `Product` document class:
  - id (String, @Id)
  - name (String)
  - description (String)
  - price (BigDecimal)
  - stock (Integer)
  - category (String)
  - attributes (Map<String, Object> - flexible for different product types)
  - createdAt, updatedAt
- [ ] Create `ProductRepository` extending MongoRepository
- [ ] Add custom queries: findByCategory, findByStockLessThan

### Step 5.2: Service Layer
- [ ] Create `InventoryService` interface and implementation
- [ ] Implement `addProduct()`:
  - Validate input
  - Save product to MongoDB
  - Return product DTO
- [ ] Implement `updateProduct()`:
  - Find product by id
  - Update fields
  - Save to MongoDB
- [ ] Implement `updateStock()`:
  - Find product by id
  - Update stock quantity
  - Publish "inventory.updated" event if stock is low
- [ ] Implement `getProductById()`
- [ ] Implement `getAllProducts()` with pagination
- [ ] Implement `searchProducts()` - flexible search by various criteria

### Step 5.3: Kafka Consumer
- [ ] Create consumer for "order.created" events
- [ ] Decrease stock when order is created
- [ ] Validate stock availability before decreasing
- [ ] Publish "inventory.updated" event

### Step 5.4: Admin Controller
- [ ] Create `InventoryController` with admin-only endpoints
- [ ] Add `@PreAuthorize("hasRole('ADMIN')")` to all endpoints
- [ ] Create `POST /api/admin/inventory/products` - Add product
- [ ] Create `PUT /api/admin/inventory/products/{id}` - Update product
- [ ] Create `PUT /api/admin/inventory/products/{id}/stock` - Update stock
- [ ] Create `GET /api/admin/inventory/products` - Get all products
- [ ] Create `GET /api/admin/inventory/products/{id}` - Get product by ID
- [ ] Create `DELETE /api/admin/inventory/products/{id}` - Delete product

### Step 5.5: Public Controller (for Order Service)
- [ ] Create `ProductController` with public endpoints
- [ ] Create `GET /api/products/{id}` - Get product details (for order validation)
- [ ] Create `GET /api/products` - Search/browse products

---

## ✅ Phase 6: Notification Service (NoSQL - MongoDB)

### Step 6.1: MongoDB Document Model
- [ ] Create `Notification` document:
  - id (String, @Id)
  - userId (Long)
  - type (NotificationType enum: ORDER_CONFIRMED, PAYMENT_SUCCESS, PAYMENT_FAILED, ORDER_SHIPPED, LOW_STOCK)
  - message (String)
  - status (NotificationStatus enum: PENDING, SENT, FAILED)
  - metadata (Map<String, Object> - flexible data)
  - createdAt, sentAt
- [ ] Create NotificationRepository
- [ ] Add queries: findByUserId, findByStatus, findByType

### Step 6.2: Service Layer
- [ ] Create `NotificationService` interface and implementation
- [ ] Implement `sendNotification()`:
  - Create notification document
  - Send email/SMS (simulate or integrate)
  - Update notification status
  - Save to MongoDB
- [ ] Implement `getUserNotifications()`
- [ ] Implement `markAsRead()`

### Step 6.3: Kafka Consumers
- [ ] Create consumer for "order.created" → Send order confirmation
- [ ] Create consumer for "payment.processed" → Send payment confirmation/failure
- [ ] Create consumer for "order.updated" → Send shipping updates
- [ ] Create consumer for "inventory.updated" → Send low stock alerts (admin)

### Step 6.4: Controller
- [ ] Create `NotificationController`
- [ ] Create `GET /api/notifications` - Get user's notifications
- [ ] Create `PUT /api/notifications/{id}/read` - Mark as read
- [ ] Create `GET /api/notifications/unread` - Get unread count

---

## ✅ Phase 7: Integration & Testing

### Step 7.1: Service Communication
- [ ] Configure Feign clients for synchronous calls:
  - Order Service → Inventory Service (check product availability)
  - Order Service → Payment Service (verify payment)
- [ ] Test Kafka event flow:
  - Order created → Payment triggered
  - Order created → Inventory updated
  - Payment processed → Notification sent
  - Inventory low → Admin notification

### Step 7.2: Error Handling
- [ ] Add global exception handler (@ControllerAdvice)
- [ ] Handle service unavailable scenarios
- [ ] Implement retry logic for Kafka consumers
- [ ] Add circuit breaker pattern (optional, using Resilience4j)

### Step 7.3: Testing
- [ ] Test complete order flow:
  1. User registers/logs in
  2. User creates order
  3. Payment is processed
  4. Inventory is updated
  5. Notifications are sent
- [ ] Test admin inventory management
- [ ] Test error scenarios (out of stock, payment failure)
- [ ] Test concurrent requests

### Step 7.4: Documentation
- [ ] Add Swagger/OpenAPI documentation
- [ ] Document all endpoints
- [ ] Document authentication flow
- [ ] Document event flow

---

## 📝 Notes

- Use `@Transactional` for database operations
- Use `@Async` for long-running tasks
- Implement proper logging (SLF4J)
- Use DTOs to avoid exposing entities
- Validate all inputs
- Handle edge cases (null checks, empty collections)
- Consider adding pagination for list endpoints
- Add proper HTTP status codes
- Use appropriate HTTP methods (GET, POST, PUT, DELETE)

