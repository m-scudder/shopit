# Shopit - Order Management System

A comprehensive microservices-based order management system built with Spring Boot, featuring event-driven architecture, hybrid database strategy (SQL + NoSQL), and secure authentication.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Services Overview](#services-overview)
- [API Documentation](#api-documentation)
- [Development](#development)
- [Testing](#testing)
- [Contributing](#contributing)

## 🎯 Overview

Shopit is a modern order management system designed to handle the complete order lifecycle from creation to delivery. The system is built using a microservices architecture pattern, enabling scalability, maintainability, and independent service deployment.

### Key Highlights

- **Microservices Architecture**: Loosely coupled services communicating via REST APIs and event-driven messaging
- **Hybrid Database Strategy**: PostgreSQL for transactional data, MongoDB for flexible document storage
- **Event-Driven Communication**: Kafka-based asynchronous messaging for inter-service communication
- **Secure Authentication**: JWT-based authentication with OAuth2 resource server
- **Role-Based Access Control**: Support for multiple user roles (Customer, Admin)

## 🏗️ Architecture

The system follows a microservices architecture with the following components:

```
┌─────────────────┐
│   API Gateway   │ (Optional - Future Enhancement)
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼────┐ ┌──────────┐ ┌──────────┐ ┌─────────────┐
│ Auth  │ │ Order │ │ Payment  │ │Inventory │ │Notification │
│Service│ │Service│ │ Service  │ │ Service  │ │  Service    │
└───┬───┘ └───┬───┘ └────┬─────┘ └────┬─────┘ └──────┬──────┘
    │         │          │            │              │
    │         │          │            │              │
┌───▼─────────▼──────────▼────────────▼──────────────▼──────┐
│                      Kafka Message Broker                  │
└────────────────────────────────────────────────────────────┘
    │         │          │            │              │
┌───▼───┐ ┌───▼───┐ ┌───▼───┐ ┌──────▼──────┐ ┌────▼─────┐
│PostgreSQL│ │PostgreSQL│ │PostgreSQL│ │  MongoDB  │ │ MongoDB │
│ (Users) │ │ (Orders) │ │(Payments)│ │(Products) │ │(Notifs) │
└─────────┘ └─────────┘ └─────────┘ └───────────┘ └─────────┘
```

### Communication Patterns

- **Synchronous**: REST APIs with OpenFeign for immediate responses (e.g., inventory validation)
- **Asynchronous**: Kafka events for decoupled communication (e.g., order created → payment processing)

## 🛠️ Tech Stack

### Core Framework
- **Spring Boot** 4.0.0
- **Java** 17
- **Maven** (Build Tool)

### Databases
- **PostgreSQL** 15 - Transactional data (Users, Orders, Payments)
- **MongoDB** 7 - Document storage (Products, Notifications)

### Messaging & Caching
- **Apache Kafka** - Event-driven messaging
- **Redis** - Caching and session management

### Security
- **Spring Security** - Authentication and authorization
- **OAuth2 Resource Server** - JWT token validation
- **JWT** - Token-based authentication

### Service Communication
- **OpenFeign** - Declarative REST clients
- **Spring Cloud** - Microservices support (optional)

### Development Tools
- **Docker & Docker Compose** - Containerization and orchestration
- **Kafka UI** - Kafka monitoring (Port 8080)
- **Mongo Express** - MongoDB admin UI (Port 8081)

## ✨ Features

### Authentication Service
- User registration and login
- JWT token generation and validation
- Role-based access control (Customer, Admin)
- Password encryption with BCrypt

### Order Service
- Create and manage orders
- Order status tracking (PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED)
- Order history for users
- Event publishing for order lifecycle

### Payment Service
- Payment processing
- Multiple payment methods (Credit Card, Debit Card, UPI, Wallet)
- Payment status tracking
- Refund processing
- Event-driven payment processing

### Inventory Service (Admin)
- Product catalog management
- Stock management
- Flexible product schema (MongoDB)
- Product search and filtering
- Low stock alerts

### Notification Service
- Order confirmation notifications
- Payment status notifications
- Shipping updates
- Admin alerts (low stock, etc.)
- Notification history

## 📦 Prerequisites

Before you begin, ensure you have the following installed:

- **Java** 17 or higher
- **Maven** 3.6+ 
- **Docker** and **Docker Compose**
- **Git** (for version control)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd shopit
```

### 2. Start Infrastructure Services

Start all required services (PostgreSQL, MongoDB, Kafka, Redis) using Docker Compose:

```bash
docker-compose up -d
```

This will start:
- PostgreSQL on port `5432`
- MongoDB on port `27017`
- Kafka on port `9092`
- Redis on port `6379`
- Kafka UI on port `8080` (http://localhost:8080)
- Mongo Express on port `8081` (http://localhost:8081)

### 3. Verify Services

Check if all services are running:

```bash
docker-compose ps
```

### 4. Build the Project

```bash
mvn clean install
```

### 5. Run the Application

```bash
mvn spring-boot:run
```

The application will start on port `8080` (default Spring Boot port).

### 6. Access the Application

- **API Base URL**: http://localhost:8080
- **Kafka UI**: http://localhost:8080 (Kafka UI)
- **Mongo Express**: http://localhost:8081

## 📁 Project Structure

```
shopit/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/scudder/shopit/
│   │   │       ├── ShopitApplication.java
│   │   │       ├── admin/          # Admin service modules
│   │   │       ├── auth/           # Authentication service
│   │   │       ├── order/          # Order service
│   │   │       ├── payment/        # Payment service
│   │   │       ├── inventory/      # Inventory service
│   │   │       └── notification/   # Notification service
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/
├── compose.yaml                     # Docker Compose configuration
├── pom.xml                          # Maven dependencies
├── SETUP_GUIDE.md                   # Detailed setup instructions
├── IMPLEMENTATION_CHECKLIST.md      # Implementation roadmap
└── README.md                        # This file
```

## 🔧 Services Overview

### Authentication Service
- **Database**: PostgreSQL
- **Endpoints**:
  - `POST /api/auth/register` - User registration
  - `POST /api/auth/login` - User login (returns JWT token)

### Order Service
- **Database**: PostgreSQL
- **Endpoints**:
  - `POST /api/orders` - Create new order
  - `GET /api/orders/{id}` - Get order by ID
  - `GET /api/orders` - Get user's orders
  - `PUT /api/orders/{id}/status` - Update order status (Admin only)
- **Events Published**: `order.created`, `order.updated`, `order.cancelled`

### Payment Service
- **Database**: PostgreSQL
- **Endpoints**:
  - `POST /api/payments` - Process payment
  - `GET /api/payments/order/{orderId}` - Get payment by order ID
  - `POST /api/payments/{id}/refund` - Refund payment (Admin only)
- **Events Consumed**: `order.created`
- **Events Published**: `payment.processed`, `payment.refunded`

### Inventory Service
- **Database**: MongoDB
- **Admin Endpoints**:
  - `POST /api/admin/inventory/products` - Add product
  - `PUT /api/admin/inventory/products/{id}` - Update product
  - `PUT /api/admin/inventory/products/{id}/stock` - Update stock
  - `GET /api/admin/inventory/products` - Get all products
  - `DELETE /api/admin/inventory/products/{id}` - Delete product
- **Public Endpoints**:
  - `GET /api/products/{id}` - Get product details
  - `GET /api/products` - Search products
- **Events Consumed**: `order.created`
- **Events Published**: `inventory.updated`

### Notification Service
- **Database**: MongoDB
- **Endpoints**:
  - `GET /api/notifications` - Get user's notifications
  - `PUT /api/notifications/{id}/read` - Mark notification as read
  - `GET /api/notifications/unread` - Get unread count
- **Events Consumed**: `order.created`, `payment.processed`, `order.updated`, `inventory.updated`

## 📚 API Documentation

API documentation will be available via Swagger/OpenAPI once implemented. For now, refer to:
- `SETUP_GUIDE.md` - Detailed service setup and configuration
- `IMPLEMENTATION_CHECKLIST.md` - Complete implementation roadmap

## 🔐 Authentication

The system uses JWT (JSON Web Tokens) for authentication:

1. **Register**: `POST /api/auth/register` with username, email, and password
2. **Login**: `POST /api/auth/login` with username and password
3. **Use Token**: Include the JWT token in the `Authorization` header for protected endpoints:
   ```
   Authorization: Bearer <your-jwt-token>
   ```

### Roles

- **ROLE_CUSTOMER**: Can place orders, view own orders
- **ROLE_ADMIN**: Full access, can manage inventory, update order status, process refunds

## 🧪 Testing

### Unit Tests

```bash
mvn test
```

### Integration Tests

Integration tests are included in the test directory. Run with:

```bash
mvn verify
```

### Manual Testing

Use tools like Postman or curl to test the API endpoints:

```bash
# Register a user
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","email":"test@example.com","password":"password123"}'

# Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"password123"}'
```

## 🛠️ Development

### Configuration

Application configuration is in `src/main/resources/application.properties`:

- Database connections (PostgreSQL, MongoDB)
- Kafka broker settings
- JWT configuration
- OAuth2 settings

### Database Management

- **PostgreSQL**: Access via `psql` or any PostgreSQL client
  ```bash
  psql -h localhost -U postgres -d orderdb
  ```

- **MongoDB**: Access via Mongo Express UI (http://localhost:8081) or `mongosh`
  ```bash
  mongosh mongodb://admin:admin123@localhost:27017/shopit?authSource=admin
  ```

### Kafka Topics

The system uses the following Kafka topics:
- `order-events` - Order lifecycle events
- `payment-events` - Payment processing events
- `inventory-events` - Inventory update events
- `notification-events` - Notification events

Monitor topics via Kafka UI at http://localhost:8080

### Adding New Features

1. Follow the microservices pattern - each service should be independent
2. Use DTOs for API communication
3. Publish events for cross-service communication
4. Add proper validation and error handling
5. Update this README with new endpoints/features

## 📝 Implementation Status

See `IMPLEMENTATION_CHECKLIST.md` for detailed implementation status and roadmap.

Current Phase: Infrastructure setup complete ✅

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Code Style

- Follow Java naming conventions
- Use meaningful variable and method names
- Add comments for complex logic
- Write unit tests for new features

## 📄 License

This project is part of an educational/development project.

## 📞 Support

For issues, questions, or contributions, please refer to:
- `HELP.md` - Common issues and solutions
- `SETUP_GUIDE.md` - Detailed setup instructions
- `IMPLEMENTATION_CHECKLIST.md` - Development roadmap

## 🎯 Roadmap

- [ ] Complete authentication service implementation
- [ ] Implement order service with Kafka integration
- [ ] Add payment service with event-driven processing
- [ ] Build inventory management service
- [ ] Implement notification service
- [ ] Add API Gateway (optional)
- [ ] Implement service discovery (Eureka)
- [ ] Add comprehensive API documentation (Swagger)
- [ ] Add monitoring and logging
- [ ] Performance optimization
- [ ] Add comprehensive test coverage

---

**Built with ❤️ using Spring Boot and Microservices Architecture**

