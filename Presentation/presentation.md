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
## REST API - HW4 Part 1

**Kevin Deras**  
**CSC 640**  
**November 6, 2025**

---

## Project Overview

A REST API for managing students, courses, and enrollments

**Tech Stack:**
- NGINX web server
- PHP 8+ with MySQL
- Bearer Token authentication
- JSON format

**Goal:** 10 API endpoints with 4 secured

---

## Objectives

- Build a REST API following best practices
- Add secure authentication
- Follow RESTful design principles
- Keep code clean and maintainable
- Document everything
- Test all endpoints

**Result:** Clean API ready for Stage 2

---

## System Architecture

```
Client Request
    ↓
NGINX
    ↓
index.php (Router)
    ↓
├── Auth.php (Security)
├── Mock.php (Data)
└── Response.php (Output)
```

---

## Project Structure

```
student-api/
├── public/
│   └── index.php
├── src/
│   ├── Auth.php
│   ├── Mock.php
│   └── Response.php
├── vendor/
│   └── autoload.php
└── composer.json
```

Single Entry Point + Separation of Concerns

---

## API Endpoints Overview

| Resource | Total | Secure |
|----------|-------|--------|
| Students | 5 | 1 |
| Courses | 5 | 2 |
| Enrollments | 3 | 2 |
| **Total** | **10** | **4** |

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

**Example:**
```json
{
  "id": 1,
  "name": "Kevin",
  "email": "kevin@example.com"
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

**Example:**
```json
{
  "id": 1,
  "code": "CSC640",
  "title": "Software Engineering"
}
```

---

## Enrollment Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | /enrollments | No | Get all enrollments |
| POST | /enrollments | **Yes** | Enroll student |
| DELETE | /enrollments/{id} | **Yes** | Unenroll student |

**Create Enrollment:**
```json
{
  "student_id": 1,
  "course_id": 1
}
```

Validates that both student and course exist

---

## Security Implementation

**Bearer Token Authentication**

4 Protected Endpoints:
1. DELETE /students/{id}
2. POST /courses
3. DELETE /courses/{id}
4. POST /enrollments
5. DELETE /enrollments/{id}

**Header:**
```
Authorization: Bearer super-secret-123
```

*Token is hardcoded for Stage 1. Will use environment variables or JWT for Stage 2.*

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

## Mock.php

```php
<?php
namespace App;

class Mock {
    public static array $students = [
        1 => ["id" => 1, "name" => "Kevin", "email" => "kevin@example.com"],
        2 => ["id" => 2, "name" => "Anthony", "email" => "anthony@example.com"],
    ];

    public static array $courses = [
        1 => ["id" => 1, "code" => "CSC640", "title" => "Software Engineering"],
        2 => ["id" => 2, "code" => "CSC601", "title" => "Algorithms"],
    ];

    public static array $enrollments = [
        1 => ["id" => 1, "student_id" => 1, "course_id" => 1],
    ];

    public static function nextId(array $arr): int {
        return $arr ? max(array_keys($arr)) + 1 : 1;
    }
}
```

Temporary in-memory data storage

---

## Routing Logic

```php
// Pattern matching for ID-based routes
if ($method === 'GET' && preg_match('#^/students/(\d+)$#', $path, $m)) {
    $id = (int)$m[1];
    
    if (isset(Mock::$students[$id])) {
        Response::json(Mock::$students[$id]);
    }
    
    Response::json(['error' => 'Student not found'], 404);
}
```

Uses regex to extract IDs from URLs

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

All POST/PUT requests validate required fields

---

## CORS Support

```php
// Handle preflight requests
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    Response::json(['ok' => true]);
}
```

Enables browser testing and frontend integration

---

## Testing

**Manual testing with cURL:**

```bash
# Test GET
curl http://localhost/students

# Test POST with auth
curl -X POST http://localhost/enrollments \
  -H "Authorization: Bearer super-secret-123" \
  -H "Content-Type: application/json" \
  -d '{"student_id": 1, "course_id": 1}'
```

All endpoints tested and working

---

## Test Results

| Test | Expected | Status |
|------|----------|--------|
| GET /students | 200 OK | Pass |
| GET /students/999 | 404 Not Found | Pass |
| POST with missing data | 422 Error | Pass |
| DELETE without auth | 401 Unauthorized | Pass |
| DELETE with token | 200 OK | Pass |
| Invalid enrollment | 422 Error | Pass |
| CORS preflight | 200 OK | Pass |

100% pass rate

---

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK - Success |
| 201 | Created |
| 401 | Unauthorized - Missing/invalid token |
| 404 | Not Found |
| 422 | Invalid data |

---

## Key Features

- Clean separation of concerns
- Modular, maintainable code
- Bearer token authentication
- Input validation
- CORS support
- RESTful architecture

---

## Challenges

**1. Routing**
- Problem: Too many if-else statements
- Solution: Used regex pattern matching

**2. CORS Errors**
- Problem: Browser blocking requests
- Solution: Added OPTIONS handling

**3. Data Validation**
- Problem: Bad data breaking the API
- Solution: Added validation on all inputs

NGINX setup was the hardest part initially

---

## Future Enhancements (Stage 2)

**Database Integration:**
- Replace Mock.php with MySQL/PDO
- Laravel
- Connection pooling

**Enhanced Security:**
- JWT with expiration
- Refresh tokens
- Rate limiting
- SQL injection prevention

**Advanced Features:**
- Pagination
- Filtering and search
- Sorting options

---

## What I Learned

- How REST APIs work in practice
- HTTP methods and status codes
- Bearer token authentication
- PHP namespaces and Composer
- NGINX configuration
- JSON data handling
- Security best practices
- API testing

More fun than expected once I got into it

---

## Project Timeline

| Milestone | Planned | Actual | Status |
|-----------|---------|--------|--------|
| Environment Setup | Oct 31 | Nov 1 | Done |
| API Implementation | Nov 3 | Nov 4 | Done |
| Security | Nov 6 | Nov 6 | Done |
| Documentation | Nov 6 | Nov 6 | Done |
| **Submission** | **Nov 7** | - | Done |

---

## Tech Stack Summary

- Web Server: NGINX
- Backend: PHP 8+ | MySQL
- Frontend: Vanilla JS
- Architecture: RESTful API
- Auth: Bearer Token
- Format: JSON
- VCS: Git/GitHub

---

## Deliverables

- 10+ REST API endpoints implemented
- 4 secure endpoints with Bearer tokens
- NGINX + PHP setup working
- Mock data store for Stage 1
- Plan and rubric documents
- Software quality documentation
- All endpoints tested

---

## Summary

- Fully functional REST API
- 10 endpoints (4 secured)
- Clean, maintainable code
- Good error handling
- Comprehensive documentation

Ready for Stage 2 database integration

---

## Stage 2 Readiness

Easy transition to database:

```php
// Replace this:
$students = Mock::$students;

// With this:
$students = $db->query("SELECT * FROM students")->fetchAll();
```

Minimal code changes needed

---

## API Reference

**Base URL:** `http://localhost`

- Students: `/students`, `/students/{id}`
- Courses: `/courses`, `/courses/{id}` 
- Enrollments: `/enrollments`, `/enrollments/{id}`
- Health: `/status`

Auth header: `Authorization: Bearer super-secret-123`

---

# Thank You

**Kevin Deras**  
**CSC 640 - HW4 Part 1**

Questions?
*End of Presentation*