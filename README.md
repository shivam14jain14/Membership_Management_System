# Membership_Management_System
A RESTful backend for managing clubs, memberships, and users. Built with Spring Boot 3, JWT-based authentication, role-based access control, and MySQL persistence.

# Club Management System — Spring Boot REST API

A RESTful backend for managing clubs, memberships, and users. Built with Spring Boot 3, JWT-based authentication, role-based access control, and MySQL persistence.

---

## Features

- User registration and login with JWT authentication
- Role-based access control with dynamic role assignment
- Token blacklisting on role changes
- Club creation and management with category support
- Club membership with automatic expiry (Yearly / Monthly / Weekly)
- Paginated and sorted category listing
- Asynchronous membership processing with `CompletableFuture`
- Caffeine-based caching
- Comprehensive exception handling with custom error responses

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Spring Boot 3.5.7 |
| Language | Java 17 |
| Security | Spring Security + JWT (jjwt 0.11.5) |
| ORM | Spring Data JPA / Hibernate |
| Database | MySQL 8 |
| Caching | Caffeine 3.1.8 |
| Build | Maven |
| Utilities | Lombok |

---

## Project Structure

```
src/main/java/com/example/demo/
├── ShivamDemoApplication.java      # Entry point
├── controller/
│   ├── UserController.java         # /users endpoints
│   ├── ClubController.java         # /clubs endpoints
│   └── MasterController.java       # /master endpoints
├── service/
│   ├── UserService.java
│   ├── ClubServInterface.java
│   ├── ClubServiceImpl.java
│   ├── MasterService.java
│   ├── JWTService.java
│   └── MyUserDetailsService.java
├── repo/
│   ├── UserRepo.java
│   ├── ClubRepo.java
│   ├── ClubMemberRepo.java
│   ├── ClubMemberShipRepo.java
│   ├── CategoryRepo.java
│   └── RoleRepo.java
├── model/
│   ├── User.java
│   ├── Club.java
│   ├── ClubMember.java
│   ├── ClubMembership.java
│   ├── ClubCategory.java
│   ├── UserRole.java
│   ├── Address.java
│   ├── MembershipCategory.java     # Enum: YEARLY, MONTHLY, WEEKLY
│   └── UserPrinciple.java
├── dto/
│   ├── ClubRequestDto.java
│   ├── ClubResponseDto.java
│   ├── ClubMembershipResponseDTO.java
│   ├── PaginationObject.java
│   └── PaginationResponseObj.java
├── config/
│   ├── SecurityConfig.java
│   └── JwtFilter.java
└── exceptions/
    ├── GlobalExceptionHandler.java
    ├── NotFoundException.java
    ├── UnAuthorisedException.java
    ├── ClubExistsException.java
    ├── MembershipExistsException.java
    └── MembershipCategoryNotFound.java
```

---

## Prerequisites

- Java 17+
- Maven 3.6+
- MySQL 8.0+

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/SpringBootDemo.git
cd SpringBootDemo/demo
```

### 2. Set up the database

Create a MySQL database:

```sql
CREATE DATABASE demo_shivam;
```

### 3. Configure application properties

Edit `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/demo_shivam
spring.datasource.username=<your-mysql-username>
spring.datasource.password=<your-mysql-password>

jwt.secret=<your-base64-encoded-secret>
jwt.expiration=36000000
```

> **Note:** Do not commit real credentials to source control. Use environment variables or a secrets manager in production.

### 4. Build and run

```bash
./mvnw spring-boot:run
```

The API will start on **http://localhost:8082**.

---

## API Reference

### Authentication — `/users`

| Method | Endpoint | Auth Required | Description |
|---|---|---|---|
| POST | `/users/register` | No | Register a new user |
| POST | `/users/login` | No | Login and receive a JWT token |
| POST | `/users/logout` | Yes | Logout |
| POST | `/users/addUserRole` | Yes | Add a role to the current user |
| GET | `/users/getAllUsers` | Yes (ROLE_USER) | List all users |
| PATCH | `/users/{id}/address` | Yes | Update user address |

**Register request body:**
```json
{
  "userName": "john",
  "userEmail": "john@example.com",
  "userPassword": "secret",
  "phoneNo": "9876543210"
}
```

**Login response:**
```json
{
  "token": "<jwt-token>"
}
```

All protected endpoints require the header:
```
Authorization: Bearer <jwt-token>
```

---

### Clubs — `/clubs`

| Method | Endpoint | Auth Required | Description |
|---|---|---|---|
| POST | `/clubs/createClub` | Yes | Create a new club |
| POST | `/clubs/updateClub/{id}` | Yes | Update club details |
| GET | `/clubs/getClubById/{id}` | Yes | Get club by ID |
| GET | `/clubs/getAllClubs` | Yes | List all clubs |
| GET | `/clubs/getActiveMemberships` | Yes | Get active memberships for current user |
| POST | `/clubs/joinClub` | Yes | Join a club |

**Create club request body:**
```json
{
  "categoryId": 1,
  "clubName": "Photography Club",
  "clubDescription": "A club for photography enthusiasts"
}
```

**Join club request body:**
```json
{
  "clubId": 1,
  "category": "YEARLY"
}
```

Membership categories: `YEARLY`, `MONTHLY`, `WEEKLY`

---

### Master Data — `/master`

| Method | Endpoint | Auth Required | Description |
|---|---|---|---|
| POST | `/master/addCategory` | Yes | Add a club category |
| POST | `/master/addRole` | Yes | Add a user role |
| GET | `/master/getAllCategories` | Yes | Get paginated list of categories |

**Get all categories query params:**

| Param | Type | Description |
|---|---|---|
| `pageNo` | int | Page number (0-indexed) |
| `pageSize` | int | Number of items per page |
| `sortBy` | string | Field to sort by |

---

## Security Design

- **Stateless** sessions — no server-side session state
- **JWT tokens** expire after 10 hours by default
- **Token blacklisting** — tokens are invalidated when a user's roles change, forcing re-login
- **BCrypt** password hashing (strength 12)
- Public endpoints: `/users/register`, `/users/login`
- All other endpoints require a valid JWT

---

## Error Responses

All errors return a consistent JSON structure:

```json
{
  "time": "2026-05-28T10:30:00",
  "message": "Resource not found"
}
```

| Exception | HTTP Status |
|---|---|
| `NotFoundException` | 400 Bad Request |
| `UnAuthorisedException` | 404 Not Found |
| `ClubExistsException` | 409 Conflict |
| `MembershipExistsException` | 409 Conflict |
| `MembershipCategoryNotFound` | 404 Not Found |

---

## Running Tests

```bash
./mvnw test
```

---

## License

This project is open-source and available under the [MIT License](LICENSE).

