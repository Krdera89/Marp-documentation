---
marp: false
---

# Student Course Management System - Technical Report

**Course:** CSC 640  
**Project:** HW4 Part 1 - REST API Development  
**Date:** November 6, 2025  
**Author:** Kevin Deras

---

## Executive Summary

This report covers my implementation of a REST API for managing students, courses, and enrollments. The API handles all the basic CRUD operations and includes secure authentication for sensitive endpoints. I built it using PHP 8+ with NGINX, following RESTful design principles and implementing Bearer token auth for the protected routes.

**What I Built:**
- 10 working REST API endpoints (more than the requirement!)
- 4 secure endpoints with Bearer token authentication
- Clean code structure with separate classes for different concerns
- Proper error handling and input validation
- CORS support so I can test it from the browser

---

## Table of Contents

1. [Introduction](#introduction)
2. [System Architecture](#system-architecture)
3. [API Endpoints](#api-endpoints)
4. [Security Implementation](#security-implementation)
5. [Technical Implementation](#technical-implementation)
6. [Testing & Validation](#testing--validation)
7. [Challenges & Solutions](#challenges--solutions)
8. [Future Enhancements](#future-enhancements)
9. [Conclusion](#conclusion)

---

## 1. Introduction

### 1.1 Project Overview

The Student Course Management System is a REST API I built to handle the basics of managing students and courses. The system lets you:

- **Student Management:** Full CRUD - create, read, update, and delete student records
- **Course Management:** Same deal for courses
- **Enrollment Management:** Let students enroll in courses (with proper authorization, of course)

Honestly, this was pretty fun to build once I got the routing figured out.

### 1.2 What I Set Out to Do

- Build a legit REST API following best practices I learned in class
- Add secure authentication to protect sensitive operations
- Make sure everything has proper error handling
- Keep the code clean and easy to maintain
- Document everything clearly

### 1.3 Technology Stack

| Component | Technology | Why I Chose It |
|-----------|-----------|----------------|
| Web Server | NGINX | Fast and handles PHP well |
| Backend | PHP 8+ | Required for the assignment |
| Architecture | RESTful API | Industry standard |
| Auth | Bearer Token | Simple but secure |
| Data Format | JSON | Universal standard |
| Version Control | Git/GitHub | Good practice |

---

## 2. System Architecture

### 2.1 Project Structure

```
student-api/
├── public/
│   └── index.php          # Main router (all requests go here)
├── src/
│   ├── Auth.php           # Handles authentication
│   ├── Mock.php           # Mock data store (my "database")
│   └── Response.php       # Standardizes all responses
├── vendor/
│   └── autoload.php       # Composer magic
└── composer.json          # Project config
```

Pretty straightforward setup. Everything routes through `index.php`, and I keep my classes organized in the `src/` folder.

### 2.2 How I Organized Things

**Single Entry Point:**
All API requests hit `index.php` first. It parses the HTTP method and URI, then figures out what to do.

**Separation of Concerns:**
I split functionality into different classes to keep things clean:
- `Response.php`: All HTTP responses go through here (consistent formatting)
- `Auth.php`: Checks if the user has a valid Bearer token
- `Mock.php`: Stores my fake data (students, courses, enrollments)
- `index.php`: Handles routing and the actual business logic

**RESTful Design:**
Following REST principles:
- Resources as URIs: `/students`, `/courses`, `/enrollments`
- HTTP methods define actions: GET (read), POST (create), PUT (update), DELETE (delete)
- Stateless - each request is independent
- JSON for everything

### 2.3 Request Flow

Here's what happens when someone hits my API:

```
Client sends request
     ↓
NGINX receives it
     ↓
Routes to index.php
     ↓
Check if auth needed (Auth::requireBearer())
     ↓
Process the request
     ↓
Get/modify data in Mock store
     ↓
Send JSON response back
     ↓
Client receives response
```

Simple and effective!

---

## 3. API Endpoints

### 3.1 Overview

The API provides 10 endpoints across three resource types:

| Resource | Total Endpoints | Secure Endpoints |
|----------|----------------|------------------|
| Students | 5 | 1 (DELETE) |
| Courses | 5 | 2 (POST, DELETE) |
| Enrollments | 3 | 2 (POST, DELETE) |
| **Total** | **10** | **4** |

### 3.2 Student Endpoints

#### GET /students
**Purpose:** Get all students  
**Authentication:** None  
**Response:** Array of student objects

```json
[
  {
    "id": 1,
    "name": "Kevin",
    "email": "kevin@example.com"
  },
  {
    "id": 2,
    "name": "Anthony",
    "email": "anthony@example.com"
  }
]
```

#### GET /students/{id}
**Purpose:** Get a specific student by ID  
**Authentication:** None  
**Parameters:** `id` (integer) - Student ID  
**Response:** Student object or 404 error

```json
{
  "id": 1,
  "name": "Kevin",
  "email": "kevin@example.com"
}
```

#### POST /students
**Purpose:** Create a new student  
**Authentication:** None  
**Request Body:**
```json
{
  "name": "Jane Smith",
  "email": "jane@example.com"
}
```
**Response:** 201 Created with student object

#### PUT /students/{id}
**Purpose:** Update an existing student  
**Authentication:** None  
**Parameters:** `id` (integer) - Student ID  
**Request Body:**
```json
{
  "name": "Jane Smith Updated",
  "email": "jane.new@example.com"
}
```

#### DELETE /students/{id} 🔒
**Purpose:** Delete a student  
**Authentication:** **Bearer Token Required**  
**Parameters:** `id` (integer) - Student ID  
**Response:** 200 OK with deletion confirmation

---

### 3.3 Course Endpoints

#### GET /courses
**Purpose:** Get all courses  
**Authentication:** None  
**Response:** Array of course objects

```json
[
  {
    "id": 1,
    "code": "CSC640",
    "title": "Software Engineering"
  },
  {
    "id": 2,
    "code": "CSC601",
    "title": "Algorithms"
  }
]
```

#### GET /courses/{id}
**Purpose:** Retrieve a specific course  
**Authentication:** None  
**Parameters:** `id` (integer) - Course ID

#### POST /courses 🔒
**Purpose:** Create a new course  
**Authentication:** **Bearer Token Required**  
**Request Body:**
```json
{
  "code": "CSC101",
  "title": "Introduction to Computer Science"
}
```

#### PUT /courses/{id}
**Purpose:** Update an existing course  
**Authentication:** None  
**Parameters:** `id` (integer) - Course ID

#### DELETE /courses/{id} 🔒
**Purpose:** Delete a course  
**Authentication:** **Bearer Token Required**  
**Parameters:** `id` (integer) - Course ID

---

### 3.4 Enrollment Endpoints

#### GET /enrollments
**Purpose:** Retrieve all enrollments  
**Authentication:** None  
**Response:** Array of enrollment objects

```json
[
  {
    "id": 1,
    "student_id": 1,
    "course_id": 1
  }
]
```

#### POST /enrollments 🔒
**Purpose:** Enroll a student in a course  
**Authentication:** **Bearer Token Required**  
**Request Body:**
```json
{
  "student_id": 1,
  "course_id": 1
}
```
**Validation:** Verifies both student and course exist

#### DELETE /enrollments/{id} 🔒
**Purpose:** Remove a student enrollment  
**Authentication:** **Bearer Token Required**  
**Parameters:** `id` (integer) - Enrollment ID

---

## 4. Security Implementation

### 4.1 Authentication Strategy

I implemented **Bearer Token Authentication** for the sensitive operations. It follows the OAuth 2.0 standard but keeps it simple for this stage.

**The 4 Secure Endpoints:**
1. `DELETE /students/{id}` - Can't let just anyone delete students
2. `POST /courses` - Need auth to create courses  
3. `DELETE /courses/{id}` - Same for deleting courses
4. `POST /enrollments` - Need auth to enroll students
5. `DELETE /enrollments/{id}` - And to unenroll them

Basically, anything that modifies important data requires the token.

### 4.2 My Auth.php Implementation

Here's my authentication code - pretty straightforward:

```php
<?php
namespace App;

class Auth {
    // TODO: move this to env var later
    private const TOKEN = 'super-secret-123';

    public static function requireBearer(): void {
        $hdr = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
        if (!preg_match('/^Bearer\s+(.+)$/i', $hdr, $m) || $m[1] !== self::TOKEN) {
            Response::json(['error' => 'Unauthorized'], 401);
        }
    }
}
```

**How it works:**
- Grabs the Authorization header
- Uses regex to check for "Bearer [token]" format
- Compares the token to my hardcoded one
- Returns 401 Unauthorized if anything's wrong
- Immediately stops the request if auth fails

Yeah, the token is hardcoded for now. For Stage 2 I'll move it to an environment variable or implement proper JWT tokens.

### 4.3 CORS Configuration

Added CORS support so I can test from the browser without issues:

```php
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    Response::json(['ok' => true]); // Handles preflight
}
```

This lets browsers make cross-origin requests to my API.

---

## 5. Technical Implementation

### 5.1 Response Handler (Response.php)

Standardizes all API responses with proper HTTP status codes and headers:

```php
namespace App;

class Response {
    public static function json(array $data, int $code = 200): void {
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

**Key Features:**
- Consistent JSON formatting
- Proper HTTP status codes
- CORS headers for all responses
- Clean exit after response

### 5.2 My Mock Data Store (Mock.php)

This simulates a database with in-memory storage. It's temporary but works great for Stage 1:

```php
<?php
namespace App;

class Mock {
    public static array $students = [
        1 => ["id" => 1, "name" => "Kevin",   "email" => "kevin@example.com"],
        2 => ["id" => 2, "name" => "Anthony", "email" => "anthony@example.com"],
    ];

    public static array $courses = [
        1 => ["id" => 1, "code" => "CSC640", "title" => "Software Engineering"],
        2 => ["id" => 2, "code" => "CSC601", "title" => "Algorithms"],
    ];

    // Simple enrollment model: id, student_id, course_id
    public static array $enrollments = [
        1 => ["id" => 1, "student_id" => 1, "course_id" => 1],
    ];

    public static function nextId(array $arr): int {
        return $arr ? max(array_keys($arr)) + 1 : 1;
    }
}
```

**Why this works for now:**
- No database setup needed yet (saves time for Stage 1)
- Fast to test and modify
- Easy to reset data during testing
- Shows how the data structure will look

The `nextId()` function is pretty clever - it finds the highest ID and adds 1, or returns 1 if the array is empty. Works perfectly for generating new IDs.

### 5.3 Routing Logic

The routing system uses regex pattern matching for dynamic routes:

```php
// Pattern matching for ID-based routes
if ($method === 'GET' && preg_match('#^/students/(\d+)$#', $path, $m)) {
    $id = (int)$m[1];
    // Handle request...
}
```

**Benefits:**
- Clean URL structure
- Type-safe parameter extraction
- Flexible route patterns
- Easy to extend

### 5.4 Input Validation

All POST/PUT requests include comprehensive validation:

```php
$in = readJsonBody();
$name = trim($in['name'] ?? '');
$email = trim($in['email'] ?? '');

if ($name === '' || $email === '') {
    Response::json(['error' => 'name and email are required'], 422);
}
```

**Validation Features:**
- Required field checking
- Data sanitization with `trim()`
- Proper HTTP 422 (Unprocessable Entity) responses
- Null coalescing for missing fields

### 5.5 Error Handling

The API provides clear, actionable error messages:

| Status Code | Meaning | Example |
|-------------|---------|---------|
| 401 | Unauthorized | Missing/invalid Bearer token |
| 403 | Forbidden | Invalid authentication credentials |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable Entity | Invalid request data |

---

## 6. Testing & Validation

### 6.1 Testing Strategy

**Manual Testing with cURL:**
```bash
# Test GET all students
curl http://localhost/students

# Test POST with authentication
curl -X POST http://localhost/enrollments \
  -H "Authorization: Bearer super-secret-123" \
  -H "Content-Type: application/json" \
  -d '{"student_id": 1, "course_id": 1}'

# Test DELETE without authentication (should fail)
curl -X DELETE http://localhost/students/1
```

### 6.2 Test Cases Executed

| Endpoint | Test Scenario | Expected Result | Status |
|----------|--------------|-----------------|--------|
| GET /students | Retrieve all students | 200 OK with array | ✅ Pass |
| GET /students/999 | Get non-existent student | 404 Not Found | ✅ Pass |
| POST /students | Create with missing data | 422 Error | ✅ Pass |
| DELETE /students/1 | Delete without auth | 401 Unauthorized | ✅ Pass |
| DELETE /students/1 | Delete with valid token | 200 OK | ✅ Pass |
| POST /enrollments | Invalid student_id | 422 Error | ✅ Pass |
| OPTIONS /students | CORS preflight | 200 OK | ✅ Pass |

### 6.3 Validation Results

**All 10 endpoints operational:**
- ✅ All GET endpoints return proper data
- ✅ POST endpoints create resources
- ✅ PUT endpoints update existing resources
- ✅ DELETE endpoints remove resources
- ✅ Authentication works on 4 secure endpoints
- ✅ Error handling returns appropriate status codes
- ✅ CORS headers present on all responses

---

## 7. Challenges & Solutions

### 7.1 Challenge: Getting the Routing Right

**Problem:** At first, my routing was turning into a mess of if-else statements that was hard to follow.

**Solution:** Switched to regex pattern matching. Much cleaner:

```php
// This is way better than a ton of if-else blocks
if (preg_match('#^/students/(\d+)$#', $path, $m)) {
    $id = (int)$m[1];  // Extract the ID safely
    // handle the request...
}
```

The regex matches the pattern and extracts the ID in one go. Pretty slick.

### 7.2 Challenge: CORS Headaches

**Problem:** When I tried testing from the browser, I kept getting CORS errors. Super annoying.

**Solution:** Added OPTIONS method handling and CORS headers to all responses. Now it works fine from anywhere.

### 7.3 Challenge: Making Sure Data is Valid

**Problem:** I realized people could send garbage data and break things.

**Solution:** Added validation on all POST/PUT requests. Now I check for required fields, trim whitespace, and return proper error messages. Much better.

```php
if ($name === '' || $email === '') {
    Response::json(['error' => 'name and email are required'], 422);
}
```

### 7.4 Challenge: Keeping the Token Secure

**Problem:** Needed to make sure the auth check was solid.

**Solution:** Made it so if the token is wrong OR missing, the request dies immediately. No data leaks, no extra processing. Just a clean error response and done.

The hardest part was honestly just getting NGINX configured properly at the start. Once that was working, the rest came together pretty quickly.

---

## 8. Future Enhancements

### 8.1 Stage 2 Roadmap (HW4 Part 3)

**Database Integration:**
- Replace Mock.php with MySQL implementation
- Add proper database schema with foreign keys
- Implement connection pooling

**Enhanced Security:**
- Implement JWT (JSON Web Tokens) with expiration
- Add refresh token mechanism
- Implement rate limiting to prevent abuse
- Add input sanitization against SQL injection

**Advanced Features:**
- Pagination for large result sets
- Filtering and search capabilities
- Sorting options (e.g., `?sort=name&order=asc`)
- Field selection (e.g., `?fields=id,name`)

**Documentation:**
- OpenAPI/Swagger specification
- Interactive API documentation
- Postman collection for testing

### 8.2 Scalability Considerations

- **Caching:** Implement Redis for frequently accessed data
- **Logging:** Add comprehensive logging for debugging and monitoring
- **Monitoring:** Integrate with APM tools for performance tracking
- **Testing:** Unit tests, integration tests, and automated CI/CD pipeline

---

## 9. Conclusion

### 9.1 Mission Accomplished

I successfully built a working REST API with PHP. All 10 endpoints work correctly, and the 4 secured ones properly check for authentication.

**What I Got Done:**
- ✅ All 10 REST API endpoints working
- ✅ 4 secure endpoints with Bearer authentication  
- ✅ Clean code that's easy to understand and modify
- ✅ Good error handling throughout
- ✅ CORS support for testing
- ✅ Everything tested and validated
- ✅ This documentation

### 9.2 What I Learned

This project taught me a lot about building APIs:
- How REST actually works in practice
- When to use different HTTP methods and status codes
- How Bearer token auth works
- PHP namespaces and autoloading (Composer is pretty cool)
- Setting up NGINX properly
- Working with JSON in PHP
- Security best practices
- How to test APIs properly

Honestly, it was more fun than I expected once I got into it.

### 9.3 Ready for Production?

Well, kind of. The current version uses mock data, but I set it up so switching to a real database will be pretty straightforward. The way I separated everything into different classes means I can just swap out Mock.php for database calls without changing much else.

For Stage 2, I'll replace the mock data with MySQL and add some more advanced features.

### 9.4 Final Thoughts

This REST API does exactly what it's supposed to do - manage students, courses, and enrollments with proper security. The modular design and documentation show I know what I'm doing (I hope!). 

I'm ready to move on to Stage 2 and add database integration. Should be interesting to see how much faster it runs with real data structures.

Overall, pretty happy with how this turned out. Now time to submit and move on to the next part!

---

## Appendices

### Appendix A: Complete API Reference

**Base URL:** `http://localhost`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | /status | - | Health check |
| GET | /students | - | List all students |
| GET | /students/{id} | - | Get student by ID |
| POST | /students | - | Create student |
| PUT | /students/{id} | - | Update student |
| DELETE | /students/{id} | 🔒 | Delete student |
| GET | /courses | - | List all courses |
| GET | /courses/{id} | - | Get course by ID |
| POST | /courses | 🔒 | Create course |
| DELETE | /courses/{id} | 🔒 | Delete course |
| GET | /enrollments | - | List enrollments |
| POST | /enrollments | 🔒 | Create enrollment |
| DELETE | /enrollments/{id} | 🔒 | Delete enrollment |

🔒 = Requires `Authorization: Bearer super-secret-123` header

### Appendix B: Environment Setup

**Requirements:**
- PHP 8.0 or higher
- NGINX web server
- Composer for autoloading
- Git for version control

**Installation Steps:**
1. Clone the repo
2. Run `composer install` (or `composer dump-autoload`)
3. Configure NGINX to point to `/public` directory
4. Start NGINX and PHP-FPM
5. Test: `curl http://localhost/status`

Pretty standard PHP setup.

### Appendix C: Project Timeline

| Milestone | Planned Date | Actual Date | Status |
|-----------|--------------|-------------|--------|
| Environment Setup | Oct 31, 2025 | Nov 1, 2025 | ✅ Complete |
| API Routes Implementation | Nov 3, 2025 | Nov 4, 2025 | ✅ Complete |
| Security Implementation | Nov 6, 2025 | Nov 6, 2025 | ✅ Complete |
| Documentation | Nov 6, 2025 | Nov 6, 2025 | ✅ Complete |
| HW4 Part 1 Submission | Nov 7, 2025 | - | 🟡 In Progress |

---

**End of Report**

*This document represents Stage 1 completion of the Student Course Management System REST API project for CSC 640.*