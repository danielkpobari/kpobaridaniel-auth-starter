# Spring Boot Security System - Authentication & Authorization

A production-ready, modular authentication and authorization system built with Spring Boot, demonstrating clean architecture principles through a reusable security starter library.

## 🏗️ Architecture Overview

This project implements a **multi-module Maven architecture** with clear separation of concerns:

```
security-system/
├── core-security-starter/     # Reusable Spring Boot Starter (Security Library)
└── sample-application/         # Consumer Application (Business Logic)
```

### Design Philosophy

- **Separation of Concerns**: All security logic lives in the starter; business logic lives in the application
- **Reusability**: The security starter can be used across multiple projects
- **Zero Configuration**: Auto-configuration via Spring Boot's `AutoConfiguration.imports`
- **Production-Ready**: Includes logging, exception handling, and standardized error responses

## 📦 Module Responsibilities

### core-security-starter

**Purpose**: Reusable Spring Boot Starter providing complete authentication and authorization infrastructure

**Contains**:
- ✅ JWT token generation and validation (`JwtTokenProvider`)
- ✅ JWT authentication filter (`JwtAuthenticationFilter`)
- ✅ Spring Security configuration (`SecurityConfig`)
- ✅ Custom authentication entry point (401 handler)
- ✅ Custom access denied handler (403 handler)
- ✅ Global exception handling with standardized error responses
- ✅ Configurable security properties (JWT secret, expiration, header)
- ✅ BCrypt password encoding
- ✅ Request logging (user, method, endpoint)

**Does NOT contain**: Business entities, repositories, controllers, or application-specific logic

### sample-application

**Purpose**: Demonstrates consumption of the security starter with minimal code

**Contains**:
- ✅ User entity and repository
- ✅ Authentication service (UserDetailsService implementation)
- ✅ REST controllers (public, user, admin endpoints)
- ✅ DTOs for requests/responses
- ✅ Application configuration

**Does NOT contain**: Any security configuration, filters, JWT logic, or exception handlers

## 🔐 Security Features

### Authentication
- Username/password authentication
- BCrypt password hashing (strength 10)
- JWT token generation with claims:
  - `sub`: username
  - `roles`: comma-separated roles
  - `iat`: issued at timestamp
  - `exp`: expiration timestamp
- Stateless session management

### Authorization
- **URL-based access control**:
  - `/api/public/**` - Public access
  - `/api/auth/**` - Public access (login)
  - `/api/admin/**` - Requires `ROLE_ADMIN`
  - All other endpoints - Requires authentication

- **Method-level security**: `@PreAuthorize` annotations supported

### Cross-Cutting Concerns
- **Logging**: Every authenticated request logs user, HTTP method, and endpoint
- **Error Handling**: Standardized JSON error responses with:
  - `timestamp`: ISO-8601 datetime
  - `status`: HTTP status code
  - `error`: Error type
  - `message`: Human-readable message
  - `path`: Request URI

## 🚀 Getting Started

### Prerequisites
- Java 17 or higher
- Maven 3.6+

### Build the Project

```bash
cd kpobari-fcmb
mvn clean install
```

This will:
1. Build the `core-security-starter` library
2. Install it to your local Maven repository
3. Build the `sample-application` with the starter as a dependency
4. Run all integration tests

### Run the Application

```bash
cd sample-application
mvn spring-boot:run
```

The application starts on `http://localhost:8080`

### Run Tests

```bash
mvn test
```

Tests verify:
- ✅ Successful login returns JWT token
- ✅ Failed login returns 401
- ✅ Public endpoints accessible without authentication
- ✅ Protected endpoints require valid JWT
- ✅ Admin endpoints require ROLE_ADMIN
- ✅ Missing token returns 401
- ✅ Insufficient role returns 403

## 📡 API Endpoints

### Public Endpoints

#### Health Check
```bash
curl -X GET http://localhost:8080/api/public/health
```

**Response**:
```json
{
  "status": "UP",
  "message": "Application is running"
}
```

### Authentication

#### Login
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "password": "password123"
  }'
```

**Response**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "username": "john",
  "roles": "ROLE_USER"
}
```

### Protected Endpoints

#### Get Current User (Authenticated)
```bash
curl -X GET http://localhost:8080/api/user/me \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response**:
```json
{
  "username": "john",
  "message": "You are authenticated"
}
```

### Admin Endpoints

#### Get All Users (Admin Only)
```bash
curl -X GET http://localhost:8080/api/admin/users \
  -H "Authorization: Bearer ADMIN_JWT_TOKEN"
```

**Response**:
```json
[
  {
    "id": 1,
    "username": "john",
    "roles": "ROLE_USER"
  },
  {
    "id": 2,
    "username": "admin",
    "roles": "ROLE_USER,ROLE_ADMIN"
  }
]
```

## 👥 Test Users

The application comes with pre-configured test users:

| Username | Password     | Roles                    |
|----------|--------------|--------------------------|
| john     | password123  | ROLE_USER                |
| admin    | password123  | ROLE_USER, ROLE_ADMIN    |
| jane     | password123  | ROLE_USER                |

## ⚙️ Configuration

Configure JWT settings in `application.yml`:

```yaml
security:
  jwt:
    secret: MySecureJwtSecretKeyForProductionUse123456789012345678901234567890
    expiration: 86400000  # 24 hours in milliseconds
    header: Authorization
    prefix: "Bearer "
```

**Production Recommendations**:
- Use environment variables for the JWT secret
- Rotate secrets regularly
- Use longer expiration for refresh tokens
- Enable HTTPS in production

## 🧪 Testing Strategy

### Integration Tests
Located in `SecurityIntegrationTest.java`, covering:

1. **Public Access**: Health endpoint accessible without auth
2. **Successful Authentication**: Valid credentials return JWT
3. **Failed Authentication**: Invalid credentials return 401
4. **Token Validation**: Valid token grants access
5. **Missing Token**: Returns 401 Unauthorized
6. **Role-Based Access**: User role denied from admin endpoint (403)
7. **Admin Access**: Admin role can access admin endpoints

### Test Execution
```bash
mvn test -Dtest=SecurityIntegrationTest
```

## 🎯 Design Decisions & Trade-offs

### 1. Spring Boot Starter Pattern
**Decision**: Package security as a reusable starter
**Rationale**: 
- Promotes code reuse across microservices
- Enforces consistent security patterns
- Simplifies application code

**Trade-off**: Adds complexity for single-application projects

### 2. JWT Stateless Authentication
**Decision**: Use JWT tokens instead of session-based auth
**Rationale**:
- Scalable for distributed systems
- No server-side session storage
- Works well with microservices

**Trade-off**: Cannot revoke tokens before expiration (consider refresh token pattern for production)

### 3. Auto-Configuration
**Decision**: Use Spring Boot 3.x `AutoConfiguration.imports`
**Rationale**:
- Modern Spring Boot standard (replaces `spring.factories`)
- Automatic bean registration
- Zero configuration for consumers

**Trade-off**: Less explicit than manual configuration

### 4. BCrypt Password Encoding
**Decision**: Use BCrypt with default strength (10)
**Rationale**:
- Industry standard for password hashing
- Built-in salt generation
- Adaptive cost factor

**Trade-off**: Slower than plain hashing (intentional security feature)

### 5. Method-Level Security
**Decision**: Enable `@PreAuthorize` annotations
**Rationale**:
- Fine-grained authorization control
- Declarative security at method level
- Complements URL-based security

**Trade-off**: Can scatter security logic across codebase

### 6. Standardized Error Responses
**Decision**: Consistent JSON error format across all endpoints
**Rationale**:
- Predictable client error handling
- Includes timestamp, status, message, and path
- Professional API design

**Trade-off**: Slightly more verbose than simple error messages

### 7. H2 In-Memory Database
**Decision**: Use H2 for sample application
**Rationale**:
- Zero configuration
- Perfect for demos and testing
- Easy to replace with production database

**Trade-off**: Not suitable for production (use PostgreSQL, MySQL, etc.)

## 🔒 Security Best Practices Implemented

1. ✅ **Password Hashing**: BCrypt with salt
2. ✅ **Stateless Sessions**: JWT-based authentication
3. ✅ **CSRF Protection**: Disabled for stateless API (appropriate for JWT)
4. ✅ **Role-Based Access Control**: URL and method-level authorization
5. ✅ **Secure Error Handling**: No sensitive information in error messages
6. ✅ **Request Logging**: Audit trail of authenticated requests
7. ✅ **Token Expiration**: Configurable JWT expiration
8. ✅ **Separation of Concerns**: Security logic isolated in starter

## 📚 Technology Stack

- **Spring Boot 3.2.0**: Framework
- **Spring Security 6.x**: Authentication & Authorization
- **JJWT 0.12.3**: JWT implementation
- **Spring Data JPA**: Data access
- **H2 Database**: In-memory database
- **Lombok**: Boilerplate reduction
- **JUnit 5**: Testing framework
- **MockMvc**: Integration testing

## 🚀 Production Deployment Checklist

- [ ] Replace H2 with production database (PostgreSQL, MySQL)
- [ ] Externalize JWT secret to environment variables
- [ ] Enable HTTPS/TLS
- [ ] Implement refresh token mechanism
- [ ] Add rate limiting
- [ ] Configure CORS policies
- [ ] Set up centralized logging (ELK, Splunk)
- [ ] Implement token blacklisting for logout
- [ ] Add API documentation (Swagger/OpenAPI)
- [ ] Configure production logging levels

## 📄 License

This project is a demonstration of Spring Boot security architecture for educational and interview purposes.

## 👨‍💻 Author



**Kpobari** - [GitHub](#   k p o b a r i d a n i e l - a u t h - s t a r t e r  
 