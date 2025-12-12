# Order Management System - Setup Guide

## Phase 1: Database Setup (SQL + NoSQL)

### Step 1: Update Docker Compose for Databases

Add the following services to your `compose.yaml`:

**SQL Database (PostgreSQL)** - For transactional data:
- Order service
- Payment service
- User/Authentication service

**NoSQL Database (MongoDB)** - For:
- Inventory service (flexible schema for product catalogs)
- Notification service (event logs, notifications)
- Analytics/logs

**Additional Services:**
- Kafka (for inter-service communication)
- Redis (for caching and session management)

### Step 2: Update pom.xml Dependencies

Add these dependencies to your `pom.xml`:

**For SQL (PostgreSQL):**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

**For NoSQL (MongoDB):**
- Already included: `spring-boot-starter-mongodb`

**For Authentication:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**For Microservices Communication:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

**For Service Discovery (Optional but recommended):**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### Step 3: Configure application.properties

Create separate profiles for each microservice or use a shared config:

**Common Database Configuration:**
```properties
# PostgreSQL (SQL)
spring.datasource.url=jdbc:postgresql://localhost:5432/orderdb
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# MongoDB (NoSQL)
spring.data.mongodb.uri=mongodb://localhost:27017/shopit
spring.data.mongodb.database=shopit

# Kafka
spring.kafka.bootstrap-servers=localhost:9092
```

---

## Phase 2: Authentication Setup

### Step 1: JWT Token Configuration

1. **Create JWT Utility Class:**
   - Generate JWT tokens
   - Validate JWT tokens
   - Extract claims from tokens

2. **Create Security Configuration:**
   - Configure `SecurityFilterChain`
   - Set up password encoder (BCrypt)
   - Configure authentication manager
   - Set up JWT filter for token validation

3. **Create User Entity (SQL):**
   - User table in PostgreSQL
   - Fields: id, username, email, password (encrypted), roles, enabled, created_at

4. **Create User Repository:**
   - JPA repository for user operations
   - Custom queries for finding by username/email

5. **Create Authentication Service:**
   - User registration
   - User login (returns JWT token)
   - Password validation
   - Role management

6. **Create Authentication Controller:**
   - `/api/auth/register` - POST
   - `/api/auth/login` - POST
   - `/api/auth/refresh` - POST (optional)

### Step 2: Role-Based Access Control (RBAC)

Define roles:
- `ROLE_CUSTOMER` - Can place orders, view own orders
- `ROLE_ADMIN` - Full access, inventory management
- `ROLE_PAYMENT_SERVICE` - Internal service role

### Step 3: Inter-Service Authentication

For microservices communication:
- Use service-to-service JWT tokens
- Or API keys for internal services
- Configure Feign clients with authentication headers

---

## Phase 3: Microservices Architecture

### Service Breakdown:

#### 1. **API Gateway Service** (Optional but recommended)
- Single entry point
- Route requests to appropriate services
- Handle authentication/authorization
- Load balancing

#### 2. **Authentication Service**
- User registration/login
- JWT token generation
- User management
- **Database:** PostgreSQL (SQL)

#### 3. **Order Service**
- Create orders
- Order status management
- Order history
- Order validation
- **Database:** PostgreSQL (SQL)
- **Communication:** Publishes events to Kafka (order created, order updated)

#### 4. **Payment Service**
- Process payments
- Payment status tracking
- Refund handling
- Payment history
- **Database:** PostgreSQL (SQL)
- **Communication:** Consumes order events, publishes payment events

#### 5. **Inventory Service** (Admin)
- Product catalog management
- Stock management
- Inventory updates
- Product search
- **Database:** MongoDB (NoSQL) - flexible schema for products
- **Communication:** Consumes order events, publishes inventory events

#### 6. **Notification Service**
- Email notifications
- SMS notifications (optional)
- Push notifications (optional)
- Notification history
- **Database:** MongoDB (NoSQL) - for notification logs
- **Communication:** Consumes events from all services

### Step 4: Service Communication Strategy

**Synchronous Communication:**
- Use REST APIs with Feign clients
- For immediate responses (e.g., checking inventory before order)

**Asynchronous Communication:**
- Use Kafka topics:
  - `order-events` - Order created, updated, cancelled
  - `payment-events` - Payment processed, failed, refunded
  - `inventory-events` - Stock updated, low stock alerts
  - `notification-events` - Send notifications

### Step 5: Database Strategy per Service

**SQL (PostgreSQL) - Transactional Data:**
- Authentication Service: Users, Roles, Sessions
- Order Service: Orders, OrderItems, OrderStatus
- Payment Service: Payments, Transactions, Refunds

**NoSQL (MongoDB) - Flexible/Document Data:**
- Inventory Service: Products (varying attributes), Categories, Stock
- Notification Service: Notification logs, Templates, User preferences

---

## Phase 4: Implementation Order

### Week 1: Foundation
1. Set up Docker Compose with all databases and services
2. Configure authentication service
3. Set up JWT and security
4. Create basic user registration/login

### Week 2: Core Services
1. Order Service (SQL)
2. Payment Service (SQL)
3. Basic integration between Order and Payment

### Week 3: Admin & Notifications
1. Inventory Service (MongoDB)
2. Admin endpoints for inventory management
3. Notification Service (MongoDB)
4. Kafka integration for event-driven communication

### Week 4: Integration & Testing
1. Connect all services via Kafka
2. End-to-end testing
3. Error handling and resilience
4. API documentation (Swagger/OpenAPI)

---

## Quick Start Commands

### Start Infrastructure:
```bash
docker-compose up -d
```

### Build and Run Services:
```bash
# Build
mvn clean install

# Run each service (you'll need separate run configurations or profiles)
mvn spring-boot:run -Dspring-boot.run.profiles=auth-service
```

---

## Database Schema Overview

### PostgreSQL Tables:

**users:**
- id, username, email, password, roles, enabled, created_at

**orders:**
- id, user_id, order_date, status, total_amount, shipping_address

**order_items:**
- id, order_id, product_id, quantity, price

**payments:**
- id, order_id, amount, payment_method, status, transaction_id, created_at

### MongoDB Collections:

**products:**
- Flexible schema: _id, name, description, price, stock, category, attributes (varies by product type)

**notifications:**
- _id, user_id, type, message, status, created_at, metadata

---

## Next Steps After Setup

1. Create entity classes for each service
2. Set up repositories
3. Create service layers
4. Implement REST controllers
5. Configure Kafka producers/consumers
6. Add exception handling
7. Implement validation
8. Add logging and monitoring

