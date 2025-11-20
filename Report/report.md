---

marp: false

---

# Student Course Management System - Technical Report

**Course:** CSC 640  

**Project:** HW4 Part 2 - REST API Development with Database Integration & laravel migration

**Date:** November 20, 2025  

**Author:** Kevin Deras

---

## Executive Summary

This report covers my implementation of a REST API for managing students, courses, and enrollments. The API handles all the basic CRUD operations and includes secure authentication for sensitive endpoints. I built it using PHP 8+ with MySQL database integration, following RESTful design principles and implementing Bearer token auth for the protected routes.

**What I Built:**

- 11 working REST API endpoints (including health check)

- 5 secure endpoints with Bearer token authentication

- MySQL database integration with proper schema and foreign keys

- Clean code structure with separate classes for different concerns

- Proper error handling and input validation

- CORS support so I can test it from the browser

- Comprehensive testing with Postman

- Laravel ORM Migration(coming soon)

---

## Table of Contents

1. [Introduction](#introduction)

2. [System Architecture](#system-architecture)

3. [API Endpoints](#api-endpoints)

4. [Security Implementation](#security-implementation)

5. [Technical Implementation](#technical-implementation)

6. [Database Design](#database-design)

7. [Testing & Validation](#testing--validation)

8. [Optional: Setting Up Nginx](#optional-setting-up-nginx-beginner-friendly)

9. [Challenges & Solutions](#challenges--solutions)

10. [Future Enhancements](#future-enhancements)

11. [Work in Progress: Laravel ORM Migration](#work-in-progress-laravel-orm-migration-target-november-26)

12. [Conclusion](#conclusion)

---

## 1. Introduction

### 1.1 Project Overview

The Student Course Management System is a REST API I built to handle the basics of managing students and courses. The system lets you:

- **Student Management:** Full CRUD - create, read, update, and delete student records

- **Course Management:** Same deal for courses

- **Enrollment Management:** Let students enroll in courses (with proper authorization, of course)

- **Database Integration:** Full MySQL database with proper relationships and data persistence

Honestly, this was pretty fun to build once I got the routing figured out. The database integration in Part 2 was a big step up from the mock data in Part 1.

### 1.2 What I Set Out to Do

- Build a real REST API following best practices I learned in class

- Integrate MySQL database with proper schema design

- Add secure authentication to protect sensitive operations

- Make sure everything has proper error handling

- Keep the code clean and easy to maintain

- Document everything clearly

- Test thoroughly with Postman

- Laravel ORM Migration (coming soon)

### 1.3 Technology Stack

| Component | Technology | Why I Chose It |

|-----------|-----------|----------------|

| Web Server | NGINX / PHP Built-in | Flexible deployment options |

| Backend | PHP 8+ | Required for the assignment |

| Database | MySQL 5.7+ | Industry standard relational database |

| Database Access | PDO (PHP Data Objects) | Secure, prepared statements, prevents SQL injection |

| Architecture | RESTful API | Industry standard |

| Auth | Bearer Token | Simple but secure |

| Data Format | JSON | Universal standard |

| Testing Tool | Postman | Industry standard API testing |

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
│   ├── Database.php       # MySQL database operations
│   ├── Mock.php           # Legacy mock data (not used)
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

- `Database.php`: Handles all MySQL database operations using PDO

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
NGINX/PHP Server receives it
     ↓
Routes to index.php
     ↓
Check if auth needed (Auth::requireBearer())
     ↓
Process the request
     ↓
Database operations via Database.php (PDO)
     ↓
Send JSON response back
     ↓
Client receives response
```

Simple and effective!

---

## 3. API Endpoints

### 3.1 Overview

The API provides 11 endpoints across three resource types plus a health check:

| Resource | Total Endpoints | Secure Endpoints |
|----------|----------------|------------------|
| Health | 1 | 0 |
| Students | 5 | 1 (DELETE) |
| Courses | 5 | 2 (POST, DELETE) |
| Enrollments | 3 | 2 (POST, DELETE) |
| **Total** | **11** | **5** |

### 3.2 Health Check Endpoint

#### GET /status

**Purpose:** Check API health and version information  

**Authentication:** None  

**Response:**
```json
{
    "ok": true,
    "php": "8.1.0",
    "database": "MySQL"
}
```

### 3.3 Student Endpoints

#### GET /students

**Purpose:** Get all students  

**Authentication:** None  

**Response:** Array of student objects
```json
[
  {
    "id": 1,
    "name": "Kevin",
    "email": "kevin@example.com",
    "created_at": "2025-11-06 10:30:00"
  },
  {
    "id": 2,
    "name": "Anthony",
    "email": "anthony@example.com",
    "created_at": "2025-11-06 10:31:00"
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
  "email": "kevin@example.com",
  "created_at": "2025-11-06 10:30:00"
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

### 3.4 Course Endpoints

#### GET /courses

**Purpose:** Get all courses  

**Authentication:** None  

**Response:** Array of course objects
```json
[
  {
    "id": 1,
    "code": "CSC640",
    "title": "Software Engineering",
    "created_at": "2025-11-06 10:30:00"
  },
  {
    "id": 2,
    "code": "CSC601",
    "title": "Algorithms",
    "created_at": "2025-11-06 10:31:00"
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

### 3.5 Enrollment Endpoints

#### GET /enrollments

**Purpose:** Retrieve all enrollments  

**Authentication:** None  

**Response:** Array of enrollment objects
```json
[
  {
    "id": 1,
    "student_id": 1,
    "course_id": 1,
    "created_at": "2025-11-06 10:35:00"
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

**Validation:** Verifies both student and course exist before creating enrollment

#### DELETE /enrollments/{id} 🔒

**Purpose:** Remove a student enrollment  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Enrollment ID

---

## 4. Security Implementation

### 4.1 Authentication Strategy

I implemented **Bearer Token Authentication** for the sensitive operations. It follows the OAuth 2.0 standard but keeps it simple for this stage.

**The 5 Secure Endpoints:**

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

Yeah, the token is hardcoded for now. For production I'll move it to an environment variable or implement proper JWT tokens.

### 4.3 SQL Injection Prevention

All database queries use **PDO prepared statements**, which automatically prevent SQL injection attacks:

```php
$stmt = self::connect()->prepare('SELECT * FROM students WHERE id = ?');
$stmt->execute([$id]);
```

This ensures user input is properly escaped and parameterized, making SQL injection impossible.

### 4.4 CORS Configuration

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

**Key Features:**

- Consistent JSON formatting

- Proper HTTP status codes

- CORS headers for all responses

- Clean exit after response

### 5.2 Database Layer (Database.php)

This is the heart of the data persistence layer. It uses PDO for secure database operations:

```php
<?php
namespace App;

use PDO;
use PDOException;

class Database {
    private static ?PDO $pdo = null;

    public static function connect(): PDO {
        if (self::$pdo === null) {
            try {
                self::$pdo = new PDO(
                    'mysql:host=localhost;dbname=student_api;charset=utf8mb4',
                    'root',
                    '',
                    [
                        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                        PDO::ATTR_EMULATE_PREPARES => false,
                    ]
                );
            } catch (PDOException $e) {
                die(json_encode(['error' => 'Database connection failed: ' . $e->getMessage()]));
            }
        }
        return self::$pdo;
    }

    // Example: Get all students
    public static function getAllStudents(): array {
        $stmt = self::connect()->query('SELECT * FROM students ORDER BY id');
        return $stmt->fetchAll();
    }

    // Example: Create student with prepared statement
    public static function createStudent(string $name, string $email): array {
        $stmt = self::connect()->prepare('INSERT INTO students (name, email) VALUES (?, ?)');
        $stmt->execute([$name, $email]);
        $id = (int)self::connect()->lastInsertId();
        return ['id' => $id, 'name' => $name, 'email' => $email];
    }
}
```

**Key Features:**

- **Singleton Pattern:** Single database connection reused across requests

- **PDO Prepared Statements:** All queries use placeholders to prevent SQL injection

- **Error Handling:** Proper exception handling with meaningful error messages

- **Type Safety:** Returns typed arrays and handles null cases properly

- **Connection Pooling:** Reuses the same PDO instance for efficiency

**Why this works well:**

- Secure by default (prepared statements)

- Efficient (connection reuse)

- Clean API (static methods, easy to use)

- Maintainable (all database logic in one place)

### 5.3 Routing Logic

The routing system uses regex pattern matching for dynamic routes:

```php
// Pattern matching for ID-based routes
if ($method === 'GET' && preg_match('#^/students/(\d+)$#', $path, $m)) {
    $id = (int)$m[1];
    $student = Database::getStudentById($id);
    if ($student) Response::json($student);
    Response::json(['error' => 'Student not found'], 404);
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

- Type checking for numeric IDs

### 5.5 Error Handling

The API provides clear, actionable error messages:

| Status Code | Meaning | Example |
|-------------|---------|---------|
| 200 | Success | Resource retrieved/updated successfully |
| 201 | Created | New resource created |
| 401 | Unauthorized | Missing/invalid Bearer token |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable Entity | Invalid request data |
| 500 | Server Error | Database connection failed |

---

## 6. Database Design

### 6.1 Schema Overview

The database consists of three main tables with proper relationships:

```sql
CREATE DATABASE student_api;

USE student_api;

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

### 6.2 Database Relationships

**Foreign Key Constraints:**

- `enrollments.student_id` → `students.id` (ON DELETE CASCADE)
- `enrollments.course_id` → `courses.id` (ON DELETE CASCADE)

**Benefits:**

- **Referential Integrity:** Can't enroll a student in a non-existent course
- **Cascade Deletes:** Deleting a student automatically removes their enrollments
- **Unique Constraints:** Prevents duplicate enrollments (same student can't enroll twice)

### 6.3 Data Validation at Database Level

- **NOT NULL constraints** ensure required fields are always present
- **VARCHAR length limits** prevent excessively long strings
- **AUTO_INCREMENT** ensures unique IDs
- **TIMESTAMP defaults** automatically track creation times

---

## 7. Testing & Validation

### 7.1 Testing Strategy

**Primary Testing Tool: Postman**

I used Postman extensively to test all endpoints. It's perfect for:
- Testing different HTTP methods
- Adding authentication headers
- Sending JSON request bodies
- Viewing formatted responses
- Saving requests for repeated testing

**Setting Up Postman:**

1. **Create a New Collection:**
   - Click "New" → "Collection"
   - Name it "Student API"
   - Click "Create"

2. **Set Up Environment Variables (Optional but Helpful):**
   - Click the gear icon (⚙️) in top right
   - Click "Add"
   - Name it "Local Development"
   - Add these variables:
     - `base_url` = `http://localhost:8000`
     - `token` = `super-secret-123`
   - Click "Save"
   - Select "Local Development" from the environment dropdown

### 7.2 Detailed Postman Testing Examples

#### Test 1: Health Check

**Request:**
- Method: `GET`
- URL: `http://localhost:8000/status`
- Headers: None needed

**Steps:**
1. Click "New" → "HTTP Request"
2. Select `GET` from dropdown
3. Enter URL: `http://localhost:8000/status`
4. Click "Send"

**Expected Response (200 OK):**
```json
{
    "ok": true,
    "php": "8.1.0",
    "database": "MySQL"
}
```

#### Test 2: Get All Students

**Request:**
- Method: `GET`
- URL: `http://localhost:8000/students`
- Headers: None needed

**Steps:**
1. Create new request
2. Select `GET`
3. Enter URL: `http://localhost:8000/students`
4. Click "Send"

**Expected Response (200 OK):**
```json
[]
```
(Empty array if no students exist yet)

#### Test 3: Create a Student

**Request:**
- Method: `POST`
- URL: `http://localhost:8000/students`
- Headers:
  - `Content-Type: application/json`
- Body (raw JSON):
  ```json
  {
      "name": "John Doe",
      "email": "john@example.com"
  }
  ```

**Steps:**
1. Create new request
2. Select `POST`
3. Enter URL: `http://localhost:8000/students`
4. Go to "Headers" tab
5. Add header:
   - Key: `Content-Type`
   - Value: `application/json`
6. Go to "Body" tab
7. Select "raw"
8. Select "JSON" from dropdown (on the right)
9. Paste the JSON body above
10. Click "Send"

**Expected Response (201 Created):**
```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

#### Test 4: Get Student by ID

**Request:**
- Method: `GET`
- URL: `http://localhost:8000/students/1`
- Headers: None needed

**Steps:**
1. Create new request
2. Select `GET`
3. Enter URL: `http://localhost:8000/students/1` (use the ID from step 3)
4. Click "Send"

**Expected Response (200 OK):**
```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

#### Test 5: Update a Student

**Request:**
- Method: `PUT`
- URL: `http://localhost:8000/students/1`
- Headers:
  - `Content-Type: application/json`
- Body (raw JSON):
  ```json
  {
      "name": "Jane Doe",
      "email": "jane@example.com"
  }
  ```

**Steps:**
1. Create new request
2. Select `PUT`
3. Enter URL: `http://localhost:8000/students/1`
4. Add `Content-Type: application/json` header
5. Add JSON body (same as POST)
6. Click "Send"

**Expected Response (200 OK):**
```json
{
    "id": 1,
    "name": "Jane Doe",
    "email": "jane@example.com"
}
```

#### Test 6: Delete a Student (Requires Authentication)

**Request:**
- Method: `DELETE`
- URL: `http://localhost:8000/students/1`
- Headers:
  - `Authorization: Bearer super-secret-123`

**Steps:**
1. Create new request
2. Select `DELETE`
3. Enter URL: `http://localhost:8000/students/1`
4. Go to "Headers" tab
5. Add header:
   - Key: `Authorization`
   - Value: `Bearer super-secret-123`
   - **Important:** Include the word "Bearer" followed by a space, then the token!
6. Click "Send"

**Expected Response (200 OK):**
```json
{
    "deleted": true
}
```

**Try without the Authorization header** - you should get a 401 Unauthorized error!

#### Test 7: Create a Course (Requires Authentication)

**Request:**
- Method: `POST`
- URL: `http://localhost:8000/courses`
- Headers:
  - `Content-Type: application/json`
  - `Authorization: Bearer super-secret-123`
- Body (raw JSON):
  ```json
  {
      "code": "CSC640",
      "title": "Software Engineering"
  }
  ```

**Steps:**
1. Create new request
2. Select `POST`
3. Enter URL: `http://localhost:8000/courses`
4. Add both headers (`Content-Type` and `Authorization`)
5. Add JSON body
6. Click "Send"

**Expected Response (201 Created):**
```json
{
    "id": 1,
    "code": "CSC640",
    "title": "Software Engineering"
}
```

#### Test 8: Create an Enrollment (Requires Authentication)

**Request:**
- Method: `POST`
- URL: `http://localhost:8000/enrollments`
- Headers:
  - `Content-Type: application/json`
  - `Authorization: Bearer super-secret-123`
- Body (raw JSON):
  ```json
  {
      "student_id": 1,
      "course_id": 1
  }
  ```

**Steps:**
1. Make sure you have at least one student and one course created first!
2. Create new request
3. Select `POST`
4. Enter URL: `http://localhost:8000/enrollments`
5. Add both headers
6. Add JSON body (use IDs from your created student and course)
7. Click "Send"

**Expected Response (201 Created):**
```json
{
    "id": 1,
    "student_id": 1,
    "course_id": 1
}
```

### 7.3 Postman Tips

1. **Save Requests to Collection:**
   - After creating a request, click "Save"
   - Choose your "Student API" collection
   - Give it a descriptive name (e.g., "Create Student")

2. **Use Variables:**
   - Instead of typing `http://localhost:8000` every time, use `{{base_url}}` if you set up environment variables
   - Use `{{token}}` for the Bearer token

3. **Test Different Scenarios:**
   - Try creating a student without email (should get 422 error)
   - Try deleting a non-existent student (should get 404 error)
   - Try accessing protected endpoints without token (should get 401 error)

### 7.4 Manual Testing with cURL

For quick command-line tests:
```bash
# Test GET all students
curl http://localhost:8000/students

# Test POST with authentication
curl -X POST http://localhost:8000/enrollments \
  -H "Authorization: Bearer super-secret-123" \
  -H "Content-Type: application/json" \
  -d '{"student_id": 1, "course_id": 1}'

# Test DELETE without authentication (should fail)
curl -X DELETE http://localhost:8000/students/1
```

### 7.5 Test Cases Executed

| Endpoint | Test Scenario | Expected Result | Status |
|----------|--------------|-----------------|--------|
| GET /status | Health check | 200 OK with version info | ✅ Pass |
| GET /students | Retrieve all students | 200 OK with array | ✅ Pass |
| GET /students/1 | Get existing student | 200 OK with student object | ✅ Pass |
| GET /students/999 | Get non-existent student | 404 Not Found | ✅ Pass |
| POST /students | Create with valid data | 201 Created | ✅ Pass |
| POST /students | Create with missing data | 422 Error | ✅ Pass |
| PUT /students/1 | Update existing student | 200 OK | ✅ Pass |
| PUT /students/999 | Update non-existent | 404 Not Found | ✅ Pass |
| DELETE /students/1 | Delete without auth | 401 Unauthorized | ✅ Pass |
| DELETE /students/1 | Delete with valid token | 200 OK | ✅ Pass |
| GET /courses | Retrieve all courses | 200 OK | ✅ Pass |
| POST /courses | Create without auth | 401 Unauthorized | ✅ Pass |
| POST /courses | Create with auth | 201 Created | ✅ Pass |
| POST /enrollments | Invalid student_id | 422 Error | ✅ Pass |
| POST /enrollments | Valid enrollment | 201 Created | ✅ Pass |

### 7.6 Validation Results

**All 11 endpoints operational:**

- ✅ Health check returns proper status
- ✅ All GET endpoints return proper data
- ✅ POST endpoints create resources correctly
- ✅ PUT endpoints update existing resources
- ✅ DELETE endpoints remove resources
- ✅ Authentication works on 5 secure endpoints
- ✅ Error handling returns appropriate status codes
- ✅ CORS headers present on all responses
- ✅ Database foreign key constraints work correctly
- ✅ Cascade deletes function as expected

---

## 8. Optional: Setting Up Nginx (Beginner-Friendly)

Nginx is a web server that's great for production. Here's how to set it up for this project.

### 8.1 Installing Nginx

#### Windows:
1. Download from: http://nginx.org/en/download.html
2. Extract to `C:\nginx`
3. Open Command Prompt as Administrator
4. Navigate to Nginx: `cd C:\nginx`
5. Start Nginx: `nginx.exe`
6. Open browser: `http://localhost` (you should see "Welcome to nginx!")

#### macOS:
```bash
brew install nginx
brew services start nginx
```

#### Linux (Ubuntu/Debian):
```bash
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 8.2 Configuring Nginx for This Project

1. **Find your Nginx config file:**
   - Windows: `C:\nginx\conf\nginx.conf`
   - macOS: `/usr/local/etc/nginx/nginx.conf`
   - Linux: `/etc/nginx/nginx.conf`

2. **Edit the config file** (you may need admin/sudo privileges):
   
   Find the `server` block and replace it with:
   ```nginx
   server {
       listen 80;
       server_name localhost;
       root C:/Users/kevin/student-api/public;  # Windows path - UPDATE THIS!
       # root /path/to/student-api/public;       # macOS/Linux path - UPDATE THIS!
       index index.php;

       location / {
           try_files $uri $uri/ /index.php?$query_string;
       }

       location ~ \.php$ {
           fastcgi_pass 127.0.0.1:9000;  # PHP-FPM
           fastcgi_index index.php;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
       }
   }
   ```

   **Important:** 
   - Update the `root` path to match your project location
   - Windows paths use forward slashes: `C:/Users/kevin/student-api/public`
   - For macOS/Linux, use absolute path: `/home/username/student-api/public`

3. **Install PHP-FPM** (PHP FastCGI Process Manager):
   
   **Windows:**
   - XAMPP users: PHP-FPM is not typically included
   - Consider using Apache instead (included in XAMPP)
   
   **macOS:**
   ```bash
   brew install php
   brew services start php
   ```
   
   **Linux:**
   ```bash
   sudo apt install php-fpm
   sudo systemctl start php7.4-fpm  # or php8.0-fpm, php8.1-fpm, etc.
   ```

4. **Test Nginx Configuration:**
   ```bash
   # Windows
   C:\nginx\nginx.exe -t
   
   # macOS/Linux
   sudo nginx -t
   ```

5. **Reload Nginx:**
   ```bash
   # Windows
   C:\nginx\nginx.exe -s reload
   
   # macOS/Linux
   sudo nginx -s reload
   # or
   sudo systemctl reload nginx
   ```

6. **Access Your API:**
   - Open browser: `http://localhost/status`
   - You should see the same response as before!

### 8.3 Troubleshooting Nginx

- **"502 Bad Gateway"**: PHP-FPM is not running. Start it.
- **"404 Not Found"**: Check the `root` path in nginx.conf
- **"403 Forbidden"**: Check file permissions on your project folder
- **Can't edit config**: Make sure you're using admin/sudo privileges

---

## 9. Challenges & Solutions

### 9.1 Challenge: Database Connection Management

**Problem:** At first, I was creating a new database connection for every request, which was inefficient and could lead to connection exhaustion.

**Solution:** Implemented a singleton pattern in Database.php. Now the connection is created once and reused:

```php
private static ?PDO $pdo = null;

public static function connect(): PDO {
    if (self::$pdo === null) {
        // Create connection only once
        self::$pdo = new PDO(...);
    }
    return self::$pdo;
}
```

Much more efficient!

### 9.2 Challenge: Foreign Key Validation

**Problem:** When creating enrollments, I needed to verify both the student and course exist before allowing the enrollment.

**Solution:** Added validation in `createEnrollment()`:

```php
public static function createEnrollment(int $studentId, int $courseId): ?array {
    // Check if student and course exist
    if (!self::getStudentById($studentId) || !self::getCourseById($courseId)) {
        return null;
    }
    // ... create enrollment
}
```

This prevents invalid enrollments and provides clear error messages.

### 9.3 Challenge: CORS Headaches

**Problem:** When I tried testing from the browser, I kept getting CORS errors. Super annoying.

**Solution:** Added OPTIONS method handling and CORS headers to all responses. Now it works fine from anywhere.

### 9.4 Challenge: Error Handling for Database Operations

**Problem:** Database errors were showing raw PDO exceptions, which isn't user-friendly.

**Solution:** Wrapped database operations in try-catch blocks and returned meaningful error messages:

```php
try {
    $student = Database::createStudent($name, $email);
    Response::json($student, 201);
} catch (Exception $e) {
    Response::json(['error' => 'Could not create student: ' . $e->getMessage()], 500);
}
```

### 9.5 Challenge: PUT Request Partial Updates

**Problem:** PUT requests should allow partial updates (only update fields that are provided).

**Solution:** Used null coalescing to merge provided data with existing data:

```php
$name = trim($in['name'] ?? $student['name']);
$email = trim($in['email'] ?? $student['email']);
```

This allows partial updates while maintaining existing values for omitted fields.

The hardest part was honestly just getting the database schema right with foreign keys. Once that was working, the rest came together pretty quickly.

---

## 10. Future Enhancements

### 10.1 Security Improvements

**Enhanced Authentication:**

- Implement JWT (JSON Web Tokens) with expiration
- Add refresh token mechanism
- Move Bearer token to environment variables
- Implement rate limiting to prevent abuse
- Add IP whitelisting for sensitive operations

**Input Sanitization:**

- Add HTML entity encoding
- Implement XSS protection
- Add input length validation
- Sanitize file uploads (if added)

### 10.2 Advanced Features

**Pagination:**

- Add pagination for large result sets
- Implement `?page=1&limit=10` query parameters
- Return metadata (total count, page info)

**Filtering and Search:**

- Add filtering capabilities (e.g., `?email=john@example.com`)
- Implement search functionality
- Add sorting options (e.g., `?sort=name&order=asc`)

**Field Selection:**

- Allow clients to request specific fields (e.g., `?fields=id,name`)
- Reduce payload size for mobile clients

### 10.3 Scalability Considerations

- **Caching:** Implement Redis for frequently accessed data
- **Logging:** Add comprehensive logging for debugging and monitoring
- **Monitoring:** Integrate with APM tools for performance tracking
- **Load Balancing:** Prepare for horizontal scaling
- **Database Optimization:** Add indexes for frequently queried fields

### 10.4 Testing Improvements

- **Unit Tests:** Add PHPUnit tests for individual functions
- **Integration Tests:** Test full request/response cycles
- **Automated Testing:** Set up CI/CD pipeline
- **API Documentation:** Generate Postman/Swagger specification
- **Postman Collection:** Export and share collection

### 10.5 Documentation

- **API Documentation:** Interactive API docs (Postman/Swagger)
- **Code Comments:** Add PHPDoc comments to all methods
- **Architecture Diagrams:** Visual representation of system design
- **Deployment Guide:** Step-by-step production deployment instructions

---

## 11. Work in Progress: Laravel ORM Migration (Target: November 26)

I'm actively refactoring this project to run on the Laravel framework with Eloquent ORM. The migration plan includes:

- Moving routing and controllers into Laravel's structure
- Replacing the manual PDO layer with Eloquent models and relationships
- Using Laravel's validation, middleware, and auth scaffolding for cleaner security
- Managing configuration via `.env` files and Laravel's config system

**Timeline:** The Laravel/Eloquent migration is scheduled for completion by **November 26**. Until then, this report documents the vanilla PHP + PDO implementation that is currently deployed.

---

## 12. Conclusion

### 12.1 Mission Accomplished

I successfully built a working REST API with PHP and MySQL database integration. All 11 endpoints work correctly, and the 5 secured ones properly check for authentication.

**What I Got Done:**

- ✅ All 11 REST API endpoints working

- ✅ 5 secure endpoints with Bearer authentication  

- ✅ MySQL database integration with proper schema

- ✅ Foreign key relationships and cascade deletes

- ✅ Clean code that's easy to understand and modify

- ✅ Good error handling throughout

- ✅ CORS support for testing

- ✅ Everything tested and validated with Postman

- ✅ Comprehensive documentation (README + this report)

### 12.2 What I Learned

This project taught me a lot about building APIs:

- How REST actually works in practice

- When to use different HTTP methods and status codes

- How Bearer token auth works

- PHP namespaces and autoloading (Composer is pretty cool)

- PDO and prepared statements for secure database access

- Database schema design with foreign keys

- Setting up NGINX properly

- Working with JSON in PHP

- Security best practices (SQL injection prevention)

- How to test APIs properly with Postman

- Database connection management and efficiency

Honestly, it was more fun than I expected once I got into it. The database integration was particularly satisfying - seeing data persist and relationships work correctly.

### 12.3 Ready for Production?

The current version is much closer to production-ready than Part 1:

- ✅ Real database persistence (no mock data)

- ✅ Secure database access (PDO prepared statements)

- ✅ Proper error handling

- ✅ Input validation

- ✅ Authentication for sensitive operations

**Still needs for production:**

- Environment variable configuration
- JWT tokens with expiration
- Rate limiting
- Comprehensive logging
- Unit/integration tests
- HTTPS enforcement
- Database connection pooling optimization

The modular design means adding these features will be straightforward without major refactoring.

### 12.4 Final Thoughts

This REST API does exactly what it's supposed to do. Manage students, courses, and enrollments with proper security and database persistence.

The database integration was a significant step up from mock data, and I'm proud of how clean the Database.php class turned out. The singleton pattern for connection management and the use of prepared statements throughout make it both efficient and secure.

Overall, i am happy with how this is turning out. The combination of clean code, proper database design, and thorough testing makes this a secure project.

---

## Appendices

### Appendix A: Complete API Reference

**Base URL:** `http://localhost:8000`

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
| PUT | /courses/{id} | - | Update course |
| DELETE | /courses/{id} | 🔒 | Delete course |
| GET | /enrollments | - | List enrollments |
| POST | /enrollments | 🔒 | Create enrollment |
| DELETE | /enrollments/{id} | 🔒 | Delete enrollment |

🔒 = Requires `Authorization: Bearer super-secret-123` header

### Appendix B: Environment Setup

**Requirements:**

- PHP 8.0 or higher
- MySQL 5.7 or higher
- Composer for autoloading
- NGINX or PHP built-in server
- Postman for testing

**Installation Steps:**

1. Clone the repo
2. Run `composer install`
3. Create MySQL database and run schema SQL
4. Configure database credentials in `src/Database.php`
5. Start server: `php -S localhost:8000 -t public`
6. Test: `curl http://localhost:8000/status`

**Database Setup:**

```sql
CREATE DATABASE student_api;
USE student_api;
-- Run schema from Database Design section
```

### Appendix C: Project Timeline

| Milestone | Planned Date | Actual Date | Status |
|-----------|--------------|-------------|--------|
| Environment Setup | Nov 1, 2025 | Nov 1, 2025 | ✅ Complete |
| Database Schema Design | Nov 2, 2025 | Nov 2, 2025 | ✅ Complete |
| Database Integration | Nov 3, 2025 | Nov 4, 2025 | ✅ Complete |
| API Routes Implementation | Nov 4, 2025 | Nov 5, 2025 | ✅ Complete |
| Security Implementation | Nov 5, 2025 | Nov 5, 2025 | ✅ Complete |
| Testing with Postman | Nov 6, 2025 | Nov 6, 2025 | ✅ Complete |
| Documentation | Nov 6, 2025 | Nov 6, 2025 | ✅ Complete |
| HW4 Part 2 Submission | Nov 7, 2025 | - | 🟡 In Progress |

---

**End of Report**

*This document represents Part 2 completion of the Student Course Management System REST API project for CSC 640, featuring MySQL database integration and preparing for laravel migration.*
