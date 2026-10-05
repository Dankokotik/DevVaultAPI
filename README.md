# DevVault API

A RESTful web service built with **Spring Boot 3** for storing, managing, and categorizing developer code snippets and associated configuration files.

Designed following multi-tier architecture principles, domain model isolation through immutable **Java Records**, compile-time object mapping with **MapStruct**, and standardized error reporting compliant with **RFC 7807**.

---

## Tech Stack

* **Language:** Java 17+
* **Framework:** Spring Boot 3.3+ (Spring Web MVC, Jakarta Validation)
* **Object Mapping:** MapStruct
* **Serialization:** Jackson (configured with UTC ISO-8601 timestamp formatting)
* **Build Tool:** Apache Maven
* **Error Specification:** RFC 7807 (Problem Details for HTTP APIs)

---

## Features

* **CRUD Lifecycle:** Complete snippet management with automated `UUID` identifiers and audit timestamps (`createdAt`, `updatedAt`).
* **Custom Bean Validation:** Declarative parameter checking via `jakarta.validation`, including a custom `@SupportedLanguage` constraint supporting `JAVA`, `PYTHON`, `CPP`, `SQL`, `KOTLIN`, and `JAVASCRIPT`.
* **In-Memory Querying:** Filtering by programming language and tags using Java Stream API predicates.
* **File Uploads:** Support for attaching supplementary source or configuration files via `MultipartFile`.
* **Centralized Exception Handling:** Standardized error contracts using `@RestControllerAdvice` and Spring's `ProblemDetail` with granular `invalidParams` field diagnostics.

---

## REST API Endpoints

Base URL: `/api/v1/snippets`

* **POST** `/api/v1/snippets`
* **Description:** Create a new code snippet
* **Success Code:** `201 Created`


* **GET** `/api/v1/snippets/{id}`
* **Description:** Retrieve a specific snippet by UUID
* **Success Code:** `200 OK`


* **GET** `/api/v1/snippets`
* **Description:** Fetch all snippets with optional query filters (`language`, `tag`)
* **Success Code:** `200 OK`


* **PUT** `/api/v1/snippets/{id}`
* **Description:** Fully update an existing snippet
* **Success Code:** `200 OK`


* **DELETE** `/api/v1/snippets/{id}`
* **Description:** Delete a snippet by UUID
* **Success Code:** `204 No Content`


* **POST** `/api/v1/snippets/{id}/attachment`
* **Description:** Upload and associate a configuration file (`multipart/form-data`)
* **Success Code:** `200 OK`



---

## Examples

### 1. Create a Snippet

**Request:**

```bash
curl -X POST http://localhost:8080/api/v1/snippets \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Quick Sort Algorithm",
    "content": "public void quickSort(int[] arr, int low, int high) { ... }",
    "language": "JAVA",
    "tags": ["algorithms", "sorting"]
  }'

```

**Response (`201 Created`):**

```json
{
  "id": "9caebb9a-5be7-48c2-a4f3-74047d4b0dd0",
  "title": "Quick Sort Algorithm",
  "content": "public void quickSort(int[] arr, int low, int high) { ... }",
  "language": "JAVA",
  "tags": [
    "algorithms",
    "sorting"
  ],
  "attachmentName": null,
  "createdAt": "2026-10-03T18:29:17.922213Z",
  "updatedAt": null
}

```

---

### 2. Validation Error (RFC 7807)

**Request with invalid payload:**

```json
{
  "title": "",
  "content": "puts 'hello world'",
  "language": "RUBY",
  "tags": []
}

```

**Response (`400 Bad Request`):**

```json
{
  "type": "https://api.devvault.com/errors/bad-request",
  "title": "Некоректні параметри запиту",
  "status": 400,
  "detail": "Помилка валідації вхідних даних",
  "instance": "/api/v1/snippets",
  "timestamp": "2026-10-03T18:29:17.922213700Z",
  "invalidParams": {
    "title": "must not be blank",
    "language": "Непідтримувана мова програмування. Дозволені: JAVA, PYTHON, CPP, SQL, KOTLIN, JAVASCRIPT"
  }
}

```

---

### 3. Upload File Attachment

**Request:**

```bash
curl -X POST http://localhost:8080/api/v1/snippets/9caebb9a-5be7-48c2-a4f3-74047d4b0dd0/attachment \
  -F "file=@config.yml"

```

---

## Project Structure

```
src/main/java/org/example/devvaultapi/
├── controller/        # REST endpoints and GlobalExceptionHandler
├── domain/            # Internal domain models
├── dto/               # Immutable Java records for request and response contracts
├── exception/         # Custom business exceptions (ResourceNotFoundException)
├── mapper/            # MapStruct interfaces for DTO-entity conversion
├── repository/        # In-memory storage backed by ConcurrentHashMap
├── service/           # Application business logic layer
└── validation/        # Custom constraint annotations and validator implementations

```

---

## Local Setup

### Prerequisites

* **Java Development Kit (JDK) 17** or newer
* **Apache Maven 3.8+** (or use the included Maven wrapper)

### Installation Steps

1. Clone the repository:
```bash
git clone https://github.com/your-username/devvault-api.git
cd devvault-api

```


2. Compile dependencies and generate MapStruct sources:
```bash
mvn clean compile

```


3. Launch the application:
```bash
mvn spring-boot:run

```



The service will start locally on port `8080` at `http://localhost:8080`.

---

## Roadmap

* [ ] Integrate **PostgreSQL** and automate schema migrations with **Flyway**.
* [ ] Migrate repository layer to **Spring Data JPA** (`JpaRepository`).
* [ ] Implement query optimization via `JOIN FETCH` and prevent **N+1** query anomalies.
* [ ] Introduce catalog pagination using `Pageable` and **Java Record Projections**.
