# CLAUDE.md - Grade Management System (Team 56)

## Project Overview
UCL COMP0010 coursework: A student grade management system with Spring Boot backend and React frontend.

**Tech Stack:**
- Backend: Java 17, Spring Boot 3.4.10, Spring Data JPA, Spring Data REST, H2 Database
- Frontend: React (pre-built, open to extension)

## Quick Commands
```bash
# Full build, test, and verification (from /backend)
./mvnw compile test checkstyle:check spotbugs:check verify site

# Run tests only
./mvnw test

# Run backend server
./mvnw spring-boot:run

# Generate site report (includes coverage)
./mvnw site
```

## Running the Application

### Backend (Terminal 1)
```bash
cd backend
./mvnw spring-boot:run
# Server runs at http://localhost:2800
# Swagger UI: http://localhost:2800/swagger-ui/index.html
```

### Frontend (Terminal 2)
```bash
cd frontend
npm ci          # or npm install if npm ci fails
npm run dev
# Frontend runs at http://localhost:5173
```

## Project Structure
```
├── backend/
│   └── src/main/java/uk/ac/ucl/comp0010/
│       ├── config/        # SecurityConfig, RestConfiguration
│       ├── controller/    # GradeController
│       ├── model/         # Entity classes (Student, Module, Grade, Registration)
│       ├── repository/    # CrudRepository interfaces
│       └── exception/     # Custom exceptions
│   └── src/main/resources/
│       ├── application.properties
│       └── schema.sql     # Database schema (PostgreSQL syntax)
└── frontend/              # React app
```

## Critical Constraints
- **90% minimum test coverage** required (JaCoCo enforced)
- **Use TDD** - all code changes must include corresponding tests
- **Javadoc required** for every class and every public method/field

## Coding Standards (Google Java Style)

### Formatting
- **Indentation**: 2 spaces (no tabs)
- **Root package**: `uk.ac.ucl.comp0010`

### Naming Conventions
| Element | Style | Example |
|---------|-------|---------|
| Classes | UpperCamelCase | `HelloWorld` |
| Methods/Variables | lowerCamelCase | `helloWorld` |
| Constants | SCREAMING_SNAKE_CASE | `HELLO_WORLD` |
| Packages | lowercase only | `uk.ac.ucl.comp0010` |

## Commit Format
```
<type>(optional scope): <short summary>

<footer>
```

| Type | Meaning |
|------|---------|
| feat | Add a new feature |
| fix | Fix a bug |
| docs | Documentation changes |
| style | Code style (formatting, indentation) |
| refactor | Code changes that neither fix bugs nor add features |
| test | Add or update tests |
| chore | Maintenance tasks (build, CI, dependencies) |
| perf | Performance improvements |

## Spring Data REST Requirements

### Repositories
All repository classes must extend `org.springframework.data.repository.CrudRepository`:
```java
public interface StudentRepository extends CrudRepository<Student, Long> {}
```

### GradeController
Implement a custom controller for adding grades:
```java
@PostMapping(value = "/grades/addGrade")
public ResponseEntity<Grade> addGrade(@RequestBody AddGradeRequest request) {
  // Find the student by using student_id from request
  // Find the module by using module_code from request
  // Create a Grade object and set all values
  // Save the Grade object
  // Return the saved Grade object
}
```

**AddGradeRequest DTO:**
```java
public class AddGradeRequest {
  @JsonProperty("student_id")
  private Long studentId;

  @JsonProperty("module_code")
  private String moduleCode;

  @JsonProperty("score")
  private Integer score;
}
```

**Request Body Example:**
```json
{
  "student_id": 1,
  "module_code": "COMP0010",
  "score": 85
}
```

**Note:** The request body uses snake_case (student_id, module_code) due to @JsonProperty annotations.

### StudentController
Custom endpoints for student-specific operations:

#### Get Student Average
Calculates and returns the average grade across all modules for a student.

```java
@GetMapping("/students/{id}/average")
public ResponseEntity<StudentAverageResponse> getStudentAverage(@PathVariable Long id)
```

**Response Example:**
```json
{
  "average": 85.5,
  "student_id": 1
}
```

**Error Responses:**
- Returns 404 if student not found
- Returns 404 if student has no grades

#### Get Student Grade for Module
Returns the grade for a specific module for a student.

```java
@GetMapping("/students/{studentId}/grades/{moduleCode}")
public ResponseEntity<Grade> getGradeForModule(
    @PathVariable Long studentId,
    @PathVariable String moduleCode)
```

**Response Example:**
```json
{
  "id": 1,
  "score": 90,
  "student": { ... },
  "module": { ... }
}
```

**Error Responses:**
Returns 404 if:
- Student not found
- Module not found
- Student not registered for module
- No grade exists for module (even if registered)

### Configuration Classes
- `SecurityConfig` - CORS configuration for frontend access
- `RestConfiguration` - Exposes entity IDs in REST responses

## Domain Model (MVP)

### Entities (in `uk.ac.ucl.comp0010.model`)

**Student**
- Fields: `id` (INT), `firstName`, `lastName`, `username`, `email`
- Methods: `computeAverage()`, `addGrade(Grade)`, `getGrade(Module)`, `registerModule(Module)`

**Module**
- Fields: `code` (String, PRIMARY KEY), `name` (String), `mnc` (Boolean - mandatory non-condonable)

**Grade**
- Fields: `id` (SERIAL), `score` (Integer), `student_id` (FK), `module_code` (FK)

**Registration**
- Fields: `id` (SERIAL), `student_id` (FK), `module_code` (FK)

### Exceptions (in `uk.ac.ucl.comp0010.exception`)
- `NoGradeAvailableException` - thrown when no grade exists for a student/module
- `NoRegistrationException` - thrown when accessing grades for unregistered modules

### Relationships
| From | To | Type | Multiplicity |
|------|-----|------|--------------|
| Student | Grade | Composition | 1:many |
| Student | Registration | Composition | 1:many |
| Grade | Module | Aggregation | 1:1 |
| Registration | Module | Aggregation | 1:1 |

## Database
- **Type**: H2 in-memory database (PostgreSQL compatibility mode)
- **Schema**: Defined in `src/main/resources/schema.sql`
- **Tables**: student, module, grade, registration

## Static Analysis
All must pass before commit:
- **Checkstyle**: Google Java Style compliance
- **SpotBugs**: Bug detection (High severity threshold)
- **JaCoCo**: Coverage verification (90% minimum line coverage)

Validation command:
```bash
mvn compile test checkstyle:check spotbugs:check verify site
```

Source code private due to university licensing.
