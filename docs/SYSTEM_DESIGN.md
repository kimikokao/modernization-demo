# .NET to Java Modernization - System Design Document

## 📋 Document Information

- **Project**: .NET to Java Modernization Demo
- **Version**: 1.0.0
- **Date**: 2026-03-31
- **Status**: Design Phase
- **Related Issue**: [#1](https://github.com/kimikokao/modernization-demo/issues/1)
- **Reference Implementation**: [AI-SDLC-lab](https://github.com/kimikokao/AI-SDLC-lab)

---

## 🎯 Executive Summary

### Project Goal
Modernize a legacy .NET account management system to a cloud-native Java Spring Boot application with improved scalability, maintainability, and testability.

### Key Objectives
1. Migrate from .NET Framework to Java Spring Boot 3.x
2. Implement RESTful API architecture
3. Achieve 90%+ unit test coverage
4. Maintain functional parity with legacy system
5. Improve performance and scalability

### Success Criteria
- All legacy features successfully migrated
- API response time < 200ms for 95th percentile
- Zero data loss during migration
- Comprehensive test coverage (>90%)
- Complete API documentation

---

## 🏗️ System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Client Layer                        │
│  (Web Browser, Mobile App, External Systems)           │
└────────────────────┬────────────────────────────────────┘
                     │ HTTPS/REST
                     ▼
┌─────────────────────────────────────────────────────────┐
│                  API Gateway Layer                      │
│              (Spring Boot Application)                  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │         AccountController (REST API)             │  │
│  │  - GET /api/accounts                             │  │
│  │  - GET /api/accounts/{id}                        │  │
│  │  - POST /api/accounts                            │  │
│  │  - PUT /api/accounts/{id}                        │  │
│  │  - DELETE /api/accounts/{id}                     │  │
│  │  - POST /api/accounts/{id}/deposit               │  │
│  │  - POST /api/accounts/{id}/withdraw              │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                 Business Logic Layer                    │
│                  (Service Layer)                        │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │            AccountService                        │  │
│  │  - Business validation                           │  │
│  │  - Transaction management                        │  │
│  │  - Business rules enforcement                    │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                Data Access Layer                        │
│              (Repository Pattern)                       │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │         AccountRepository (JPA)                  │  │
│  │  - CRUD operations                               │  │
│  │  - Custom queries                                │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                  Database Layer                         │
│         (PostgreSQL / H2 for testing)                   │
└─────────────────────────────────────────────────────────┘
```

### Technology Stack Comparison

| Component | .NET (Legacy) | Java (Target) |
|-----------|---------------|---------------|
| Framework | .NET Framework 4.x | Spring Boot 3.2.0 |
| Language | C# | Java 17 |
| API Style | SOAP/WCF | REST/JSON |
| ORM | Entity Framework | Spring Data JPA |
| Database | SQL Server | PostgreSQL |
| Testing | NUnit | JUnit 5 + Mockito |
| Build Tool | MSBuild | Maven |
| Container | IIS | Embedded Tomcat |

---

## 📊 Data Model

### Account Entity

```java
@Entity
@Table(name = "accounts")
public class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String accountNumber;
    
    @Column(nullable = false)
    private String accountHolder;
    
    @Column(nullable = false)
    private BigDecimal balance;
    
    @Column(nullable = false)
    private String accountType;
    
    @Column(nullable = false)
    private LocalDateTime createdAt;
    
    @Column(nullable = false)
    private LocalDateTime updatedAt;
}
```

### Database Schema

```sql
CREATE TABLE accounts (
    id BIGSERIAL PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    account_holder VARCHAR(100) NOT NULL,
    balance DECIMAL(15,2) NOT NULL DEFAULT 0.00,
    account_type VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT chk_balance CHECK (balance >= 0),
    CONSTRAINT chk_account_type CHECK (account_type IN ('SAVINGS', 'CHECKING', 'BUSINESS'))
);

CREATE INDEX idx_account_number ON accounts(account_number);
CREATE INDEX idx_account_holder ON accounts(account_holder);
```

---

## 🔌 API Design

### RESTful Endpoints

#### 1. Get All Accounts
```http
GET /api/accounts
Response: 200 OK
[
  {
    "id": 1,
    "accountNumber": "ACC001",
    "accountHolder": "John Doe",
    "balance": 1000.00,
    "accountType": "SAVINGS",
    "createdAt": "2026-03-31T08:00:00Z",
    "updatedAt": "2026-03-31T08:00:00Z"
  }
]
```

#### 2. Get Account by ID
```http
GET /api/accounts/{id}
Response: 200 OK / 404 Not Found
{
  "id": 1,
  "accountNumber": "ACC001",
  "accountHolder": "John Doe",
  "balance": 1000.00,
  "accountType": "SAVINGS",
  "createdAt": "2026-03-31T08:00:00Z",
  "updatedAt": "2026-03-31T08:00:00Z"
}
```

#### 3. Create Account
```http
POST /api/accounts
Request Body:
{
  "accountNumber": "ACC002",
  "accountHolder": "Jane Smith",
  "balance": 500.00,
  "accountType": "CHECKING"
}
Response: 201 Created
```

#### 4. Update Account
```http
PUT /api/accounts/{id}
Request Body:
{
  "accountHolder": "Jane Smith Updated",
  "accountType": "SAVINGS"
}
Response: 200 OK / 404 Not Found
```

#### 5. Delete Account
```http
DELETE /api/accounts/{id}
Response: 204 No Content / 404 Not Found
```

#### 6. Deposit
```http
POST /api/accounts/{id}/deposit
Request Body:
{
  "amount": 100.00
}
Response: 200 OK
```

#### 7. Withdraw
```http
POST /api/accounts/{id}/withdraw
Request Body:
{
  "amount": 50.00
}
Response: 200 OK / 400 Bad Request (insufficient funds)
```

### Error Handling

```json
{
  "timestamp": "2026-03-31T08:00:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Insufficient funds for withdrawal",
  "path": "/api/accounts/1/withdraw"
}
```

---

## 🧪 Testing Strategy

### Test Pyramid

```
           ┌─────────────┐
           │   E2E Tests │  (10%)
           │   (Manual)  │
           └─────────────┘
         ┌─────────────────┐
         │ Integration Tests│ (20%)
         │   (MockMvc)     │
         └─────────────────┘
      ┌──────────────────────┐
      │    Unit Tests        │ (70%)
      │  (JUnit + Mockito)   │
      └──────────────────────┘
```

### Test Coverage Goals

| Layer | Target Coverage | Test Framework |
|-------|----------------|----------------|
| Controller | 90% | JUnit 5 + MockMvc |
| Service | 95% | JUnit 5 + Mockito |
| Model | 85% | JUnit 5 |
| Repository | 80% | Spring Data JPA Tests |
| **Overall** | **90%** | **JUnit 5 Suite** |

### Test Types

#### Unit Tests (61 tests)
- **AccountServiceTest** (24 tests)
  - CRUD operations
  - Business logic validation
  - Transaction handling
  - Error scenarios

- **AccountControllerTest** (17 tests)
  - HTTP endpoint testing
  - Request/response validation
  - Status code verification
  - Error handling

- **AccountTest** (20 tests)
  - Entity behavior
  - Data validation
  - Getter/setter verification

---

## 🔒 Security Considerations

### Authentication & Authorization
- Implement Spring Security
- JWT token-based authentication
- Role-based access control (RBAC)

### Data Protection
- Encrypt sensitive data at rest
- Use HTTPS for all communications
- Implement input validation
- Prevent SQL injection via JPA

### Audit Logging
- Log all account transactions
- Track user actions
- Maintain audit trail

---

## 📈 Performance Requirements

### Response Time Targets

| Operation | Target (95th percentile) |
|-----------|-------------------------|
| GET requests | < 100ms |
| POST/PUT requests | < 200ms |
| DELETE requests | < 150ms |
| Deposit/Withdraw | < 250ms |

### Scalability
- Support 1000 concurrent users
- Handle 10,000 requests per minute
- Database connection pooling (HikariCP)
- Horizontal scaling capability

---

## 🚀 Deployment Strategy

### Environments

1. **Development**
   - Local H2 database
   - Hot reload enabled
   - Debug logging

2. **Testing**
   - PostgreSQL database
   - Integration tests
   - Performance testing

3. **Staging**
   - Production-like environment
   - UAT testing
   - Load testing

4. **Production**
   - High availability setup
   - Monitoring and alerting
   - Backup and disaster recovery

### Containerization

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/account-api.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 📝 Migration Plan

### Phase 1: Preparation (Week 1-2)
- [ ] Set up development environment
- [ ] Create project structure
- [ ] Configure dependencies
- [ ] Set up CI/CD pipeline

### Phase 2: Core Implementation (Week 3-4)
- [ ] Implement data model
- [ ] Create repository layer
- [ ] Implement service layer
- [ ] Develop REST controllers

### Phase 3: Testing (Week 5)
- [ ] Write unit tests
- [ ] Implement integration tests
- [ ] Perform load testing
- [ ] Security testing

### Phase 4: Migration (Week 6)
- [ ] Data migration scripts
- [ ] Parallel run with legacy system
- [ ] Validation and verification
- [ ] Cutover to new system

### Phase 5: Post-Migration (Week 7-8)
- [ ] Monitor system performance
- [ ] Address issues
- [ ] Optimize performance
- [ ] Decommission legacy system

---

## 📚 Documentation

### Required Documentation
1. API Documentation (Swagger/OpenAPI)
2. Database Schema Documentation
3. Deployment Guide
4. Operations Manual
5. Troubleshooting Guide
6. User Guide

---

## 🔗 References

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Reference Implementation](https://github.com/kimikokao/AI-SDLC-lab)

---

## 📞 Contact & Support

- **Project Owner**: kimikokao
- **Repository**: [modernization-demo](https://github.com/kimikokao/modernization-demo)
- **Issue Tracker**: [GitHub Issues](https://github.com/kimikokao/modernization-demo/issues)

---

**Document Version**: 1.0.0  
**Last Updated**: 2026-03-31  
**Status**: ✅ Design Complete
