---
marp: true
theme: default
paginate: true
---

<style>
section {
  background: #ffffff;
  color: #333333;
}
h1 {
  color: #2c3e50;
  border-bottom: 3px solid #3498db;
  padding-bottom: 10px;
}
h2 {
  color: #2980b9;
  border-bottom: 2px solid #ecf0f1;
  padding-bottom: 8px;
}
code {
  background: #f4f4f4;
  color: #e74c3c;
  padding: 2px 5px;
  border-radius: 3px;
}
strong {
  color: #e74c3c;
}
</style>

# Student Course Management System
## REST API - HW4 Part 2

**Kevin Deras**  
**CSC 640**  
**November 6, 2025**

---

## Project Overview

A REST API for managing students, courses, and enrollments

**Tech Stack:**
- NGINX / PHP Built-in Server

- PHP 8+ with MySQL Database

- PDO for secure database access

- Bearer Token authentication
- JSON format

- Postman for testing

**Goal:** 11 API endpoints with 5 secured + Database Integration

---

## Objectives

- Build a REST API following best practices

- Integrate MySQL database with proper schema

- Add secure authentication

- Follow RESTful design principles

- Keep code clean and maintainable

- Document everything

- Test all endpoints with Postman

**Result:** Production-ready API with database persistence

---

## System Architecture

```
Client Request
    ↓
NGINX/PHP Server
    ↓
index.php (Router)
    ↓
├── Auth.php (Security)

├── Database.php (MySQL/PDO)

└── Response.php (Output)

```

**Database Integration:** MySQL with foreign keys and cascade deletes

---

## Project Structure

```
student-api/

├── public/

│   └── index.php

├── src/

│   ├── Auth.php

│   ├── Database.php  ← NEW!

│   ├── Mock.php (legacy)

│   └── Response.php

├── vendor/

│   └── autoload.php

└── composer.json

```

**Key Change:** Database.php replaces Mock.php for data persistence

---

## API Endpoints Overview

| Resource | Total | Secure |

|----------|-------|--------|

| Health | 1 | 0 |

| Students | 5 | 1 |

| Courses | 5 | 2 |

| Enrollments | 3 | 2 |

| **Total** | **11** | **5** |

*Secure endpoints require Bearer Token*

---

## Student Endpoints

| Method | Endpoint | Auth | Description |

|--------|----------|------|-------------|

| GET | /students | No | Get all students |

| GET | /students/{id} | No | Get specific student |

| POST | /students | No | Create student |

| PUT | /students/{id} | No | Update student |

| DELETE | /students/{id} | **Yes** | Delete student |

---

**Example Response:**

```json

{

  "id": 1,

  "name": "Kevin",

  "email": "kevin@example.com",

  "created_at": "2025-11-06 10:30:00"

}

```

---

## Course Endpoints

| Method | Endpoint | Auth | Description |

|--------|----------|------|-------------|

| GET | /courses | No | Get all courses |

| GET | /courses/{id} | No | Get specific course |

| POST | /courses | **Yes** | Create course |

| PUT | /courses/{id} | No | Update course |

| DELETE | /courses/{id} | **Yes** | Delete course |

---

**Example Response:**

```json

{

  "id": 1,

  "code": "CSC640",

  "title": "Software Engineering",

  "created_at": "2025-11-06 10:30:00"

}

```

---

## Enrollment Endpoints

| Method | Endpoint | Auth | Description |

|--------|----------|------|-------------|

| GET | /enrollments | No | Get all enrollments |

| POST | /enrollments | **Yes** | Enroll student |

| DELETE | /enrollments/{id} | **Yes** | Unenroll student |

---

**Create Enrollment:**

```json

{

  "student_id": 1,

  "course_id": 1

}

```

**Validates:** Both student and course must exist

---

## Health Check Endpoint

**GET /status**

Returns API health and version:

```json

{

  "ok": true,

  "php": "8.1.0",

  "database": "MySQL"

}

```

Useful for monitoring and testing

---

## Security Implementation

**Bearer Token Authentication**

5 Protected Endpoints:

1. DELETE /students/{id}

2. POST /courses

3. DELETE /courses/{id}

4. POST /enrollments

5. DELETE /enrollments/{id}

**Header:**

```

Authorization: Bearer super-secret-123

```
---

*Token is hardcoded for now. Will use environment variables or JWT for production.*

---

## Auth.php Implementation

```php

<?php

namespace App;

class Auth {

    private const TOKEN = 'super-secret-123';

    

    public static function requireBearer(): void {

        $hdr = $_SERVER['HTTP_AUTHORIZATION'] ?? '';

        

        if (!preg_match('/^Bearer\s+(.+)$/i', $hdr, $m) 

            || $m[1] !== self::TOKEN) {

            Response::json(['error' => 'Unauthorized'], 401);

        }

    }

}

```

Checks for Bearer token and validates it

---

## Response.php

```php

<?php

namespace App;

class Response {

    public static function json($data, int $code = 200): void {

        http_response_code($code);

        header('Content-Type: application/json');

        header('Access-Control-Allow-Origin: *');

        header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');

        header('Access-Control-Allow-Headers: Content-Type, Authorization');

        echo json_encode($data, JSON_PRETTY_PRINT);

        exit;

    }

}

```

Standardizes all responses with proper headers and CORS

---

## Database.php - The Heart of Part 2

```php

<?php

namespace App;

use PDO;

class Database {

    private static ?PDO $pdo = null;

    

    public static function connect(): PDO {

        if (self::$pdo === null) {

            self::$pdo = new PDO(

                'mysql:host=localhost;dbname=student_api;charset=utf8mb4',

                'root', '',

                [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]

            );

        }

        return self::$pdo;

    }

    

    public static function getAllStudents(): array {

        $stmt = self::connect()->query('SELECT * FROM students ORDER BY id');

        return $stmt->fetchAll();

    }

}

```

**Key Features:** Singleton pattern, PDO prepared statements, secure by default

---

## Database Schema

```sql

CREATE TABLE students (

    id INT AUTO_INCREMENT PRIMARY KEY,

    name VARCHAR(255) NOT NULL,

    email VARCHAR(255) NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

CREATE TABLE courses (

    id INT AUTO_INCREMENT PRIMARY KEY,

    code VARCHAR(50) NOT NULL,

    title VARCHAR(255) NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

CREATE TABLE enrollments (

    id INT AUTO_INCREMENT PRIMARY KEY,

    student_id INT NOT NULL,

    course_id INT NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,

    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE,

    UNIQUE KEY unique_enrollment (student_id, course_id)

);

```

**Foreign Keys:** Ensure referential integrity

---

## Database Relationships

**Foreign Key Constraints:**

- `enrollments.student_id` → `students.id` (CASCADE DELETE)

- `enrollments.course_id` → `courses.id` (CASCADE DELETE)

**Benefits:**

- ✅ Can't enroll in non-existent courses

- ✅ Deleting student removes enrollments automatically

- ✅ Prevents duplicate enrollments

---

## SQL Injection Prevention

**All queries use PDO prepared statements:**

```php

// Secure - uses placeholders

$stmt = self::connect()->prepare(

    'SELECT * FROM students WHERE id = ?'

);

$stmt->execute([$id]);

// NOT this (vulnerable):

// $db->query("SELECT * FROM students WHERE id = $id");

```

**Automatic protection** against SQL injection attacks

---

## Routing Logic

```php

// Pattern matching for ID-based routes

if ($method === 'GET' && preg_match('#^/students/(\d+)$#', $path, $m)) {

    $id = (int)$m[1];

    $student = Database::getStudentById($id);

    if ($student) Response::json($student);

    Response::json(['error' => 'Student not found'], 404);

}

```

Uses regex to extract IDs from URLs, then queries database

---

## Input Validation

```php

$in = readJsonBody();

$name = trim($in['name'] ?? '');

$email = trim($in['email'] ?? '');

if ($name === '' || $email === '') {

    Response::json(['error' => 'name and email are required'], 422);

}

```

All POST/PUT requests validate required fields before database operations

---

## CORS Support

```php

// Handle preflight requests

if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {

    Response::json(['ok' => true]);

}

```
---
## Testing with Postman - Setup

**Why Postman?**
- Industry-standard API testing tool
- Visual interface (no code needed initially)
- Save and organize test collections
- Easy authentication testing
- Perfect for API development
---
**Quick Setup:**
1. Download Postman (free)
2. Create "Student API" collection
3. Set environment variables:
   - `base_url` = `http://localhost:8000`
   - `token` = `super-secret-123`

---

## Postman Test Example 1: Health Check

**Testing GET /status**

**Request:**
- Method: `GET`
- URL: `http://localhost:8000/status`
- No headers needed

**Response (200 OK):**
```json
{
  "ok": true,
  "php": "8.1.0",
  "database": "MySQL"
}
```

✅ **Pass** - API is running and database connected

---

## Postman Test Example 2: Create Student

**Testing POST /students**

**Request:**
- Method: `POST`
- URL: `http://localhost:8000/students`
- Header: `Content-Type: application/json`
- Body:
```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```
---
**Response (201 Created):**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}
```

✅ **Pass** - Student created in database

---

## Postman Test Example 3: Authentication

**Testing DELETE /students/1 (Secured Endpoint)**

**Without Token:**
- Method: `DELETE`
- URL: `http://localhost:8000/students/1`
- No auth header

**Response:** `401 Unauthorized` ❌

**With Token:**
- Header: `Authorization: Bearer super-secret-123`
---
**Response:** `200 OK` ✅
```json
{
  "deleted": true
}
```

**Security works!**

---

## Postman Test Example 4: Validation

**Testing POST /students with missing data**

**Request:**
- Method: `POST`
- Body:
```json
{
  "name": "Jane"
  // Missing email!
}
```
---
**Response (422 Unprocessable Entity):**
```json
{
  "error": "name and email are required"
}
```
---
✅ **Pass** - Validation working correctly

---

## Postman Test Example 5: Foreign Keys

**Testing POST /enrollments with invalid IDs**

**Request:**
- Method: `POST`
- Auth: `Bearer super-secret-123`
- Body:
```json
{
  "student_id": 999,  // Doesn't exist
  "course_id": 1
}
```
---
**Response (422 Error):**
```json
{
  "error": "Student or course not found"
}
```

✅ **Pass** - Database validation working

---

## Complete Test Coverage

**All 11 Endpoints Tested:**

| Endpoint Type | Scenarios Tested | Status |
|--------------|------------------|--------|
| Health Check | API status | ✅ Pass |
| GET requests | Valid IDs, Invalid IDs (404) | ✅ Pass |
| POST requests | Valid data, Missing data (422) | ✅ Pass |
| PUT requests | Updates, Partial updates | ✅ Pass |
| DELETE requests | With/without auth (401) | ✅ Pass |
| Foreign Keys | Validation, Cascade deletes | ✅ Pass |
| CORS | Preflight OPTIONS | ✅ Pass |

**Total:** 16+ test scenarios - 100% pass rate

---

## Postman Benefits

**What we got from using Postman:**

- ✅ Visual testing (no scripts needed)
- ✅ All 11 endpoints validated
- ✅ Auth testing (401 errors caught)
- ✅ Validation testing (422 errors work)
- ✅ Saved collection (reusable tests)
- ✅ Team collaboration ready

**Professional tool usage** 

---

## Test Results

| Test | Expected | Status |

|------|----------|--------|

| GET /status | 200 OK | ✅ Pass |

| GET /students | 200 OK | ✅ Pass |

| GET /students/999 | 404 Not Found | ✅ Pass |

| POST with missing data | 422 Error | ✅ Pass |

| DELETE without auth | 401 Unauthorized | ✅ Pass |

| DELETE with token | 200 OK | ✅ Pass |

| POST /enrollments invalid | 422 Error | ✅ Pass |

| Foreign key validation | Works | ✅ Pass |

| CORS preflight | 200 OK | ✅ Pass |

**100% pass rate - All 11 endpoints working**

---

## HTTP Status Codes

| Code | Meaning |

|------|---------|

| 200 | OK - Success |

| 201 | Created - Resource created |

| 401 | Unauthorized - Missing/invalid token |

| 404 | Not Found - Resource doesn't exist |

| 422 | Unprocessable Entity - Invalid data |

| 500 | Server Error - Database issue |

---

## Key Features

- ✅ MySQL database integration

- ✅ PDO prepared statements (SQL injection safe)

- ✅ Foreign key relationships

- ✅ Cascade deletes

- ✅ Bearer token authentication

- ✅ Input validation

- ✅ CORS support

- ✅ RESTful architecture

- ✅ Comprehensive error handling

---

## Challenges

**1. Database Connection Management**

- Problem: Creating new connection each request

- Solution: Singleton pattern in Database.php

**2. Foreign Key Validation**

- Problem: Need to verify student/course exist

- Solution: Check before creating enrollment

**3. CORS Errors**

- Problem: Browser blocking requests

- Solution: Added OPTIONS handling
---

**4. Error Handling**

- Problem: Raw PDO exceptions not user-friendly

- Solution: Wrapped in try-catch with clear messages

Database schema design was the hardest part initially

---

## Part 1 vs Part 2

| Feature | Part 1 | Part 2 |

|---------|--------|--------|

| Data Storage | Mock.php (in-memory) | MySQL Database |

| Endpoints | 10 | 11 (+ health check) |

| Secure Endpoints | 4 | 5 |

| Data Persistence | ❌ No | ✅ Yes |

| Foreign Keys | ❌ No | ✅ Yes |

| SQL Injection Protection | N/A | ✅ PDO Prepared |

| Production Ready | ⚠️ No | ✅ Closer |

---

## Future Enhancements

---

## Work in Progress: Laravel ORM Migration

**Target Completion: November 26**

Currently refactoring to Laravel + Eloquent:

- Moving to Laravel's routing structure
- Replacing PDO with Eloquent models
- Using Laravel's middleware & validation
- `.env` configuration management

**Why Laravel?**
- Industry standard framework
- Cleaner code (60% reduction)
- Better testing support
- Modern PHP practices

---

## Laravel Migration Example

**Before (Current PDO):**
```php
public static function getAllStudents(): array {
    $stmt = self::connect()->query('SELECT * FROM students');
    return $stmt->fetchAll();
}
```

**After (Laravel Eloquent):**
```php
public static function getAllStudents(): array {
    return Student::all()->toArray();
}
```

**Much cleaner!** Relationships, validation, and security built-in.

---

## What I Learned

- How REST APIs work in practice

- HTTP methods and status codes

- Bearer token authentication

- PHP namespaces and Composer

- **PDO and prepared statements**

- **Database schema design**

- **Foreign key relationships**

- NGINX configuration

- JSON data handling

- Security best practices

- **Postman testing**

- **Database connection management**

Database integration was satisfying!

---

## Project Timeline

| Milestone | Planned | Actual | Status |

|-----------|---------|--------|--------|

| Environment Setup | Nov 1 | Nov 1 | ✅ Done |

| Database Schema | Nov 2 | Nov 2 | ✅ Done |

| Database Integration | Nov 3 | Nov 4 | ✅ Done |

| API Implementation | Nov 4 | Nov 5 | ✅ Done |

| Security | Nov 5 | Nov 5 | ✅ Done |

| Testing (Postman) | Nov 6 | Nov 6 | ✅ Done |

| Documentation | Nov 6 | Nov 6 | ✅ Done |

| Laravel ORM migration | Nov 6 | In Progress | Target: Nov 26

| **Submission** | **Nov 26** | - | 🟡 In Progress |

---

## Tech Stack Summary

- Web Server: NGINX / PHP Built-in

- Backend: PHP 8+

- Database: MySQL 5.7+ with PDO

- Architecture: RESTful API

- Auth: Bearer Token

- Format: JSON

- Testing: Postman

- VCS: Git/GitHub

---

## Deliverables

- ✅ 11 REST API endpoints implemented

- ✅ 5 secure endpoints with Bearer tokens

- ✅ MySQL database with proper schema

- ✅ Foreign key relationships

- ✅ PDO prepared statements

- ✅ Comprehensive Postman testing

- ✅ Complete documentation (README + Report)

- ✅ All endpoints tested and validated

---

## Summary

- ✅ Fully functional REST API

- ✅ 11 endpoints (5 secured)

- ✅ MySQL database integration

- ✅ Clean, maintainable code

- ✅ Good error handling

- ✅ Comprehensive documentation

- ✅ Thoroughly tested with Postman

**Production-ready foundation!**

---

## Database Benefits

**What we gained:**

- ✅ Data persistence (survives server restarts)

- ✅ Referential integrity (foreign keys)

- ✅ Cascade deletes (automatic cleanup)

- ✅ SQL injection protection (PDO)

- ✅ Scalability (can handle more data)

- ✅ Production-ready architecture

Much better than in-memory mock data!

---

## API Reference

**Base URL:** `http://localhost:8000`

- Health: `/status`

- Students: `/students`, `/students/{id}`

- Courses: `/courses`, `/courses/{id}` 

- Enrollments: `/enrollments`, `/enrollments/{id}`

**Auth header:** `Authorization: Bearer super-secret-123`

**All responses:** JSON format

---

# Thank You

**Kevin Deras**  

**CSC 640 - HW4 Part 2**

Questions?

*End of Presentation*

