# Student Course Management System - Technical Report

**Course:** CSC 640  

**Project:** HW4 Part 3 - REST API Development with Database Integration & Laravel Migration

**Date:** December 2025  

**Author:** Kevin Deras

---

## Executive Summary

This report covers my implementation of a REST API for managing students, courses, and enrollments using the Laravel framework. The API handles all the basic CRUD operations and includes secure authentication for all endpoints using Laravel Sanctum. I migrated from my vanilla PHP + PDO implementation to Laravel 10 with Eloquent ORM, following RESTful design principles and implementing token-based authentication.

**What I Built:**

- 12 working REST API endpoints (including health check and authentication)

- All endpoints secured with Laravel Sanctum token authentication

- MySQL database integration with Laravel migrations and Eloquent ORM

- Clean MVC architecture following Laravel conventions

- Proper validation using Laravel's built-in validation system

- CORS support configured for cross-origin requests

- Docker/Sail setup for easy deployment

- Comprehensive model relationships with Eloquent

---

## Table of Contents

1. [Introduction](#introduction)

2. [System Architecture](#system-architecture)

3. [API Endpoints](#api-endpoints)

4. [Security Implementation](#security-implementation)

5. [Technical Implementation](#technical-implementation)

6. [Database Design](#database-design)

7. [Testing & Validation](#testing--validation)

8. [Laravel Migration from Vanilla PHP](#laravel-migration-from-vanilla-php)

9. [Challenges & Solutions](#challenges--solutions)

10. [Future Enhancements](#future-enhancements)

11. [Conclusion](#conclusion)

---

## 1. Introduction

### 1.1 Project Overview

The Student Course Management System is a REST API I built using Laravel to handle the basics of managing students and courses. The system lets you:

- **Student Management:** Full CRUD - create, read, update, and delete student records

- **Course Management:** Same deal for courses

- **Enrollment Management:** Let students enroll in courses (with proper authorization, of course)

- **Database Integration:** Full MySQL database with Laravel migrations, Eloquent ORM, and proper relationships

This is a complete migration from my vanilla PHP implementation. Moving to Laravel was a game-changer - the framework handles so much of the boilerplate that I was writing manually before. Eloquent ORM is way cleaner than raw PDO queries, and Laravel's validation system is incredibly powerful.

### 1.2 What I Set Out to Do

- Migrate from vanilla PHP to Laravel framework

- Replace PDO with Eloquent ORM for database operations

- Implement Laravel Sanctum for secure token-based authentication

- Use Laravel migrations instead of raw SQL schema

- Leverage Laravel's built-in validation system

- Maintain all the same functionality as the original PHP version

- Set up Docker/Sail for easy deployment

- Document everything clearly

### 1.3 Technology Stack

| Component | Technology | Why I Chose It |

|-----------|-----------|----------------|

| Framework | Laravel 10 | Industry-standard PHP framework with excellent tooling |

| Web Server | Laravel Sail (Docker) | Easy containerized deployment |

| Backend | PHP 8.1+ | Required for Laravel 10 |

| Database | MySQL 8.4 | Industry standard relational database |

| ORM | Eloquent (Laravel) | Clean, expressive database queries |

| Authentication | Laravel Sanctum | Built-in token authentication for APIs |

| Architecture | RESTful API | Industry standard |

| Data Format | JSON | Universal standard |

| Testing Tool | Postman | Industry standard API testing |

| Version Control | Git/GitHub | Good practice |

---

## 2. System Architecture

### 2.1 Project Structure

```
student-api-laravel/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── AuthController.php
│   │   │   ├── StudentController.php
│   │   │   ├── CourseController.php
│   │   │   └── EnrollmentController.php
│   │   └── Middleware/
│   └── Models/
│       ├── Student.php
│       ├── Course.php
│       ├── Enrollment.php
│       └── User.php
├── database/
│   └── migrations/
│       ├── create_students_table.php
│       ├── create_courses_table.php
│       └── create_enrollments_table.php
├── routes/
│   └── api.php          # All API routes defined here
├── config/
│   ├── auth.php
│   ├── sanctum.php
│   └── cors.php
├── compose.yaml         # Docker Compose configuration
└── artisan              # Laravel CLI tool
```

Much cleaner than the vanilla PHP version! Laravel's conventions make everything predictable and easy to find.

### 2.2 How I Organized Things

**MVC Architecture:**

Laravel follows the Model-View-Controller pattern:

- **Models** (`app/Models/`): Eloquent models representing database tables with relationships

- **Controllers** (`app/Http/Controllers/`): Handle HTTP requests and return responses

- **Routes** (`routes/api.php`): Define API endpoints and map them to controllers

**Separation of Concerns:**

- Controllers handle request/response logic

- Models handle data relationships and business logic

- Migrations define database schema

- Middleware handles authentication and other cross-cutting concerns

**RESTful Design:**

Following REST principles:

- Resources as URIs: `/api/students`, `/api/courses`, `/api/enrollments`

- HTTP methods define actions: GET (read), POST (create), PUT (update), DELETE (delete)

- Stateless - each request is independent

- JSON for everything

### 2.3 Request Flow

Here's what happens when someone hits my API:

```
Client sends request
     ↓
Laravel receives it (via Sail/Docker)
     ↓
Routes to api.php
     ↓
Middleware checks authentication (Sanctum)
     ↓
Routes to appropriate Controller
     ↓
Controller validates input (Laravel validation)
     ↓
Controller uses Eloquent Model for database operations
     ↓
Model returns data (automatic JSON serialization)
     ↓
Controller returns JSON response
     ↓
Client receives response
```

Laravel handles so much automatically - routing, validation, JSON serialization, error handling. It's beautiful.

---

## 3. API Endpoints

### 3.1 Overview

The API provides 12 endpoints across authentication and three resource types plus a health check:

| Resource | Total Endpoints | Secure Endpoints |

|----------|----------------|------------------|

| Health | 1 | 0 |

| Authentication | 2 | 0 (login), 1 (logout) |

| Students | 5 | 5 (all require auth) |

| Courses | 5 | 5 (all require auth) |

| Enrollments | 3 | 3 (all require auth) |

| **Total** | **12** | **11** |

**Note:** Unlike the vanilla PHP version where only some endpoints were protected, in this Laravel version ALL endpoints (except health check and login) require authentication. This is a more secure approach.

### 3.2 Health Check Endpoint

#### GET /api/status

**Purpose:** Check API health and version information  

**Authentication:** None  

**Response:**

```json
{
    "ok": true,
    "php": "8.1.0",
    "database": "MySQL (Laravel Eloquent)"
}
```

### 3.3 Authentication Endpoints

#### POST /api/login

**Purpose:** Authenticate and receive a Bearer token  

**Authentication:** None (this is how you get authenticated)  

**Request Body:**

```json
{
    "email": "user@example.com",
    "password": "password123"
}
```

**Response:** 200 OK with token

```json
{
    "token": "1|abcdef1234567890..."
}
```

**Usage:** Include this token in subsequent requests as `Authorization: Bearer {token}`

#### POST /api/logout

**Purpose:** Invalidate the current authentication token  

**Authentication:** **Bearer Token Required**  

**Response:** 200 OK

```json
{
    "message": "Logged out"
}
```

### 3.4 Student Endpoints

#### GET /api/students

**Purpose:** Get all students  

**Authentication:** **Bearer Token Required**  

**Response:** Array of student objects

```json
[
  {
    "id": 1,
    "name": "Kevin",
    "email": "kevin@example.com",
    "created_at": "2025-12-03T10:30:00.000000Z",
    "updated_at": "2025-12-03T10:30:00.000000Z"
  },
  {
    "id": 2,
    "name": "Anthony",
    "email": "anthony@example.com",
    "created_at": "2025-12-03T10:31:00.000000Z",
    "updated_at": "2025-12-03T10:31:00.000000Z"
  }
]
```

#### GET /api/students/{id}

**Purpose:** Get a specific student by ID  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Student ID  

**Response:** Student object or 404 error

```json
{
  "id": 1,
  "name": "Kevin",
  "email": "kevin@example.com",
  "created_at": "2025-12-03T10:30:00.000000Z",
  "updated_at": "2025-12-03T10:30:00.000000Z"
}
```

#### POST /api/students

**Purpose:** Create a new student  

**Authentication:** **Bearer Token Required**  

**Request Body:**

```json
{
  "name": "Jane Smith",
  "email": "jane@example.com"
}
```

**Validation:** 
- `name`: required, string
- `email`: required, valid email, unique in students table

**Response:** 201 Created with student object

#### PUT /api/students/{id}

**Purpose:** Update an existing student  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Student ID  

**Request Body:**

```json
{
  "name": "Jane Smith Updated",
  "email": "jane.new@example.com"
}
```

**Validation:** Same as POST, but fields are optional (partial updates allowed)

#### DELETE /api/students/{id}

**Purpose:** Delete a student  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Student ID  

**Response:** 200 OK with deletion confirmation

```json
{
  "message": "Deleted"
}
```

---

### 3.5 Course Endpoints

#### GET /api/courses

**Purpose:** Get all courses  

**Authentication:** **Bearer Token Required**  

**Response:** Array of course objects

```json
[
  {
    "id": 1,
    "code": "CSC640",
    "title": "Software Engineering",
    "created_at": "2025-12-04T10:30:00.000000Z",
    "updated_at": "2025-12-04T10:30:00.000000Z"
  },
  {
    "id": 2,
    "code": "CSC601",
    "title": "Algorithms",
    "created_at": "2025-12-04T10:31:00.000000Z",
    "updated_at": "2025-12-04T10:31:00.000000Z"
  }
]
```

#### GET /api/courses/{id}

**Purpose:** Retrieve a specific course  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Course ID

#### POST /api/courses

**Purpose:** Create a new course  

**Authentication:** **Bearer Token Required**  

**Request Body:**

```json
{
  "code": "CSC101",
  "title": "Introduction to Computer Science"
}
```

**Validation:**
- `code`: required, string, max 50 characters, unique in courses table
- `title`: required, string, max 255 characters

#### PUT /api/courses/{id}

**Purpose:** Update an existing course  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Course ID

#### DELETE /api/courses/{id}

**Purpose:** Delete a course  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Course ID

---

### 3.6 Enrollment Endpoints

#### GET /api/enrollments

**Purpose:** Retrieve all enrollments with student and course details  

**Authentication:** **Bearer Token Required**  

**Response:** Array of enrollment objects with relationships

```json
[
  {
    "id": 1,
    "student_id": 1,
    "course_id": 1,
    "created_at": "2025-12-04T10:35:00.000000Z",
    "updated_at": "2025-12-04T10:35:00.000000Z",
    "student": {
      "id": 1,
      "name": "Kevin",
      "email": "kevin@example.com"
    },
    "course": {
      "id": 1,
      "code": "CSC640",
      "title": "Software Engineering"
    }
  }
]
```

**Note:** Eloquent automatically includes related student and course data - no manual joins needed!

#### POST /api/enrollments

**Purpose:** Enroll a student in a course  

**Authentication:** **Bearer Token Required**  

**Request Body:**

```json
{
  "student_id": 1,
  "course_id": 1
}
```

**Validation:** 
- `student_id`: required, integer, must exist in students table
- `course_id`: required, integer, must exist in courses table
- Prevents duplicate enrollments (same student can't enroll twice in same course)

#### DELETE /api/enrollments/{id}

**Purpose:** Remove a student enrollment  

**Authentication:** **Bearer Token Required**  

**Parameters:** `id` (integer) - Enrollment ID

---

## 4. Security Implementation

### 4.1 Authentication Strategy

I implemented **Laravel Sanctum** for token-based authentication. This is a huge upgrade from the simple Bearer token in the vanilla PHP version.

**Key Improvements:**

- **Token Management:** Tokens are stored in the database and can be revoked

- **User-Based:** Each token is tied to a specific user account

- **Secure by Default:** Laravel handles token generation, validation, and expiration

- **Logout Support:** Tokens can be invalidated on logout

**All Endpoints Protected:**

Unlike the vanilla PHP version where only 5 endpoints were protected, ALL endpoints (except `/api/status` and `/api/login`) require authentication. This is a more secure approach - if you're using the API, you should be authenticated.

### 4.2 Laravel Sanctum Implementation

Laravel Sanctum is built into Laravel and handles everything automatically:

**Token Creation (Login):**

```php
// AuthController.php
$token = $user->createToken('api-token')->plainTextToken;
return response()->json(['token' => $token]);
```

**Token Validation (Automatic):**

Laravel's middleware handles this automatically:

```php
// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    // All protected routes here
});
```

**Token Revocation (Logout):**

```php
// AuthController.php
$request->user()->currentAccessToken()->delete();
```

**How it works:**

1. User logs in with email/password via `/api/login`

2. System validates credentials and creates a Sanctum token

3. Token is returned to client

4. Client includes token in `Authorization: Bearer {token}` header for all subsequent requests

5. Laravel middleware automatically validates the token on each request

6. If token is invalid or missing, request is rejected with 401 Unauthorized

Much cleaner than my manual Bearer token implementation!

### 4.3 SQL Injection Prevention

Laravel's Eloquent ORM uses **prepared statements automatically** for all database queries. This means SQL injection is impossible:

```php
// This is safe - Eloquent handles it
Student::where('email', $email)->first();

// Even raw queries are parameterized
DB::select('SELECT * FROM students WHERE id = ?', [$id]);
```

No need to manually write prepared statements like in the PDO version.

### 4.4 Input Validation

Laravel's validation system is incredibly powerful and prevents invalid data:

```php
$data = $request->validate([
    'name'  => 'required|string',
    'email' => 'required|email|unique:students,email',
]);
```

**Built-in Rules:**
- `required`: Field must be present
- `string`: Must be a string
- `email`: Must be valid email format
- `unique:table,column`: Must be unique in database
- `exists:table,column`: Must exist in database
- `integer`: Must be an integer
- And many more...

If validation fails, Laravel automatically returns a 422 Unprocessable Entity response with detailed error messages.

### 4.5 CORS Configuration

CORS is configured in `config/cors.php`:

```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_methods' => ['*'],
'allowed_origins' => ['*'],
'allowed_headers' => ['*'],
```

This allows cross-origin requests from any domain. For production, you'd want to restrict `allowed_origins` to your specific frontend domain.

---

## 5. Technical Implementation

### 5.1 Controllers

Controllers handle HTTP requests and return responses. Much cleaner than the routing logic in `index.php` from the vanilla PHP version.

**StudentController Example:**

```php
<?php

namespace App\Http\Controllers;

use App\Models\Student;
use Illuminate\Http\Request;

class StudentController extends Controller
{
    public function index()
    {
        return response()->json(Student::all());
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'name'  => 'required|string',
            'email' => 'required|email|unique:students,email',
        ]);

        $student = Student::create($data);
        return response()->json($student, 201);
    }

    public function show($id)
    {
        $student = Student::findOrFail($id);
        return response()->json($student);
    }

    public function update(Request $request, $id)
    {
        $student = Student::findOrFail($id);
        
        $data = $request->validate([
            'name'  => 'sometimes|required|string',
            'email' => 'sometimes|required|email|unique:students,email,' . $student->id,
        ]);

        $student->update($data);
        return response()->json($student);
    }

    public function destroy($id)
    {
        $student = Student::findOrFail($id);
        $student->delete();
        return response()->json(['message' => 'Deleted']);
    }
}
```

**Key Features:**

- **Automatic JSON:** Laravel automatically converts arrays/objects to JSON

- **Validation:** Built-in validation with clear error messages

- **findOrFail:** Automatically returns 404 if resource doesn't exist

- **Clean Code:** Much less boilerplate than the vanilla PHP version

### 5.2 Eloquent Models

Models define database relationships and business logic. This replaces the `Database.php` class from the vanilla PHP version.

**Student Model:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Student extends Model
{
    protected $fillable = ['name', 'email'];

    public function enrollments()
    {
        return $this->hasMany(Enrollment::class);
    }

    public function courses()
    {
        return $this->belongsToMany(Course::class, 'enrollments');
    }
}
```

**Key Features:**

- **Automatic Timestamps:** `created_at` and `updated_at` are handled automatically

- **Relationships:** Define relationships once, use them everywhere

- **Mass Assignment Protection:** Only `fillable` fields can be mass-assigned

- **Type Safety:** Eloquent handles type conversion automatically

**Using Relationships:**

```php
// Get student with all enrollments
$student = Student::with('enrollments')->find(1);

// Get enrollment with student and course
$enrollment = Enrollment::with(['student', 'course'])->find(1);

// Get all courses for a student
$student->courses; // Automatic relationship loading
```

No manual JOINs needed!

### 5.3 Database Migrations

Migrations define database schema in code. This is way better than raw SQL files.

**Students Migration:**

```php
Schema::create('students', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamps();
});
```

**Enrollments Migration (with Foreign Keys):**

```php
Schema::create('enrollments', function (Blueprint $table) {
    $table->id();
    $table->foreignId('student_id')->constrained()->cascadeOnDelete();
    $table->foreignId('course_id')->constrained()->cascadeOnDelete();
    $table->timestamps();
    
    $table->unique(['student_id', 'course_id']);
});
```

**Key Features:**

- **Version Control:** Schema changes are tracked in Git

- **Rollback Support:** Can undo migrations if needed

- **Database Agnostic:** Works with MySQL, PostgreSQL, SQLite, etc.

- **Foreign Keys:** `constrained()` automatically creates foreign key relationships

- **Cascade Deletes:** `cascadeOnDelete()` handles cleanup automatically

**Running Migrations:**

```bash
php artisan migrate        # Run migrations
php artisan migrate:rollback # Undo last migration
php artisan migrate:fresh   # Drop all tables and re-run migrations
```

### 5.4 Routing

Routes are defined in `routes/api.php` using Laravel's fluent routing API:

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/students', [StudentController::class, 'index']);
    Route::post('/students', [StudentController::class, 'store']);
    Route::get('/students/{id}', [StudentController::class, 'show']);
    Route::put('/students/{id}', [StudentController::class, 'update']);
    Route::delete('/students/{id}', [StudentController::class, 'destroy']);
});
```

**Key Features:**

- **Resource Routes:** Could use `Route::apiResource('students', StudentController::class)` for all CRUD routes

- **Middleware Groups:** Apply authentication to multiple routes at once

- **Type-Hinted Parameters:** Laravel automatically resolves controller dependencies

- **Clean URLs:** No need for manual regex pattern matching

### 5.5 Error Handling

Laravel provides excellent error handling out of the box:

**Automatic Error Responses:**

- **404 Not Found:** When `findOrFail()` doesn't find a resource

- **422 Unprocessable Entity:** When validation fails

- **401 Unauthorized:** When authentication fails

- **500 Server Error:** When exceptions occur (with detailed error messages in development)

**Custom Error Responses:**

```php
return response()->json(['error' => 'Course not found'], 404);
```

Laravel automatically formats these as JSON for API requests.

---

## 6. Database Design

### 6.1 Schema Overview

The database consists of three main tables with proper relationships, defined using Laravel migrations:

**Students Table:**

```php
Schema::create('students', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamps();
});
```

**Courses Table:**

```php
Schema::create('courses', function (Blueprint $table) {
    $table->id();
    $table->string('code')->unique();
    $table->string('title');
    $table->timestamps();
});
```

**Enrollments Table:**

```php
Schema::create('enrollments', function (Blueprint $table) {
    $table->id();
    $table->foreignId('student_id')->constrained()->cascadeOnDelete();
    $table->foreignId('course_id')->constrained()->cascadeOnDelete();
    $table->timestamps();
    
    $table->unique(['student_id', 'course_id']);
});
```

### 6.2 Database Relationships

**Eloquent Relationships:**

- `Student` has many `Enrollment` (one-to-many)
- `Course` has many `Enrollment` (one-to-many)
- `Student` belongs to many `Course` through `Enrollment` (many-to-many)
- `Course` belongs to many `Student` through `Enrollment` (many-to-many)
- `Enrollment` belongs to `Student` (many-to-one)
- `Enrollment` belongs to `Course` (many-to-one)

**Foreign Key Constraints:**

- `enrollments.student_id` → `students.id` (ON DELETE CASCADE)
- `enrollments.course_id` → `courses.id` (ON DELETE CASCADE)

**Benefits:**

- **Referential Integrity:** Can't enroll a student in a non-existent course

- **Cascade Deletes:** Deleting a student automatically removes their enrollments

- **Unique Constraints:** Prevents duplicate enrollments (same student can't enroll twice)

- **Automatic Joins:** Eloquent handles JOINs automatically when loading relationships

### 6.3 Data Validation at Database Level

- **NOT NULL constraints** ensure required fields are always present (handled by Laravel migrations)

- **UNIQUE constraints** prevent duplicate emails and course codes

- **Foreign key constraints** ensure referential integrity

- **AUTO_INCREMENT** ensures unique IDs (handled by `$table->id()`)

- **Timestamps** automatically track creation and update times (handled by `$table->timestamps()`)

---

## 7. Testing & Validation

### 7.1 Testing Strategy

**Primary Testing Tool: Postman**

I used Postman extensively to test all endpoints, just like in the vanilla PHP version. The workflow is similar, but now we need to authenticate first.

**Setting Up Postman:**

1. **Create a New Collection:**

   - Click "New" → "Collection"

   - Name it "Student API Laravel"

   - Click "Create"

2. **Set Up Environment Variables:**

   - Click the gear icon (⚙️) in top right

   - Click "Add"

   - Name it "Laravel API"

   - Add these variables:

     - `base_url` = `http://localhost/api`

     - `token` = (leave empty, will be set after login)

   - Click "Save"

   - Select "Laravel API" from the environment dropdown

### 7.2 Detailed Postman Testing Examples

#### Test 1: Health Check

**Request:**

- Method: `GET`

- URL: `{{base_url}}/status`

- Headers: None needed

**Expected Response (200 OK):**

```json
{
    "ok": true,
    "php": "8.1.0",
    "database": "MySQL (Laravel Eloquent)"
}
```

#### Test 2: Login (Get Token)

**Request:**

- Method: `POST`

- URL: `{{base_url}}/login`

- Headers:

  - `Content-Type: application/json`

- Body (raw JSON):

  ```json
  {
      "email": "user@example.com",
      "password": "password"
  }
  ```

**Steps:**

1. First, you need to create a user in the database. You can do this via Laravel Tinker:

   ```bash
   php artisan tinker
   >>> $user = new App\Models\User();
   >>> $user->name = "Test User";
   >>> $user->email = "user@example.com";
   >>> $user->password = Hash::make('password');
   >>> $user->save();
   ```

2. Create new request in Postman

3. Select `POST`

4. Enter URL: `{{base_url}}/login`

5. Add `Content-Type: application/json` header

6. Add JSON body with email and password

7. Click "Send"

**Expected Response (200 OK):**

```json
{
    "token": "1|abcdef1234567890..."
}
```

8. **Important:** Copy the token value and set it in your Postman environment variable `{{token}}`

#### Test 3: Get All Students (Requires Authentication)

**Request:**

- Method: `GET`

- URL: `{{base_url}}/students`

- Headers:

  - `Authorization: Bearer {{token}}`

**Steps:**

1. Create new request

2. Select `GET`

3. Enter URL: `{{base_url}}/students`

4. Go to "Authorization" tab

5. Select "Bearer Token" from Type dropdown

6. Enter `{{token}}` in the Token field (or paste the token directly)

7. Click "Send"

**Expected Response (200 OK):**

```json
[]
```

(Empty array if no students exist yet)

**Try without the Authorization header** - you should get a 401 Unauthorized error!

#### Test 4: Create a Student

**Request:**

- Method: `POST`

- URL: `{{base_url}}/students`

- Headers:

  - `Content-Type: application/json`

  - `Authorization: Bearer {{token}}`

- Body (raw JSON):

  ```json
  {
      "name": "John Doe",
      "email": "john@example.com"
  }
  ```

**Expected Response (201 Created):**

```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "created_at": "2025-12-04T10:30:00.000000Z",
    "updated_at": "2025-12-04T10:30:00.000000Z"
}
```

#### Test 5: Create a Course

**Request:**

- Method: `POST`

- URL: `{{base_url}}/courses`

- Headers:

  - `Content-Type: application/json`

  - `Authorization: Bearer {{token}}`

- Body (raw JSON):

  ```json
  {
      "code": "CSC640",
      "title": "Software Engineering"
  }
  ```

**Expected Response (201 Created):**

```json
{
    "id": 1,
    "code": "CSC640",
    "title": "Software Engineering",
    "created_at": "2025-12-04T10:30:00.000000Z",
    "updated_at": "2025-12-04T10:30:00.000000Z"
}
```

#### Test 6: Create an Enrollment

**Request:**

- Method: `POST`

- URL: `{{base_url}}/enrollments`

- Headers:

  - `Content-Type: application/json`

  - `Authorization: Bearer {{token}}`

- Body (raw JSON):

  ```json
  {
      "student_id": 1,
      "course_id": 1
  }
  ```

**Expected Response (201 Created):**

```json
{
    "id": 1,
    "student_id": 1,
    "course_id": 1,
    "created_at": "2025-12-04T10:35:00.000000Z",
    "updated_at": "2025-12-04T10:35:00.000000Z",
    "student": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com"
    },
    "course": {
        "id": 1,
        "code": "CSC640",
        "title": "Software Engineering"
    }
}
```

**Note:** Eloquent automatically includes the related student and course data!

#### Test 7: Get All Enrollments

**Request:**

- Method: `GET`

- URL: `{{base_url}}/enrollments`

- Headers:

  - `Authorization: Bearer {{token}}`

**Expected Response (200 OK):**

```json
[
  {
    "id": 1,
    "student_id": 1,
    "course_id": 1,
    "created_at": "2025-12-04T10:35:00.000000Z",
    "updated_at": "2025-12-04T10:35:00.000000Z",
    "student": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    },
    "course": {
      "id": 1,
      "code": "CSC640",
      "title": "Software Engineering"
    }
  }
]
```

### 7.3 Validation Testing

**Test Invalid Data:**

Try creating a student with invalid data:

```json
{
    "name": "",
    "email": "not-an-email"
}
```

**Expected Response (422 Unprocessable Entity):**

```json
{
    "message": "The name field is required. (and 1 more error)",
    "errors": {
        "name": ["The name field is required."],
        "email": ["The email must be a valid email address."]
    }
}
```

Laravel provides detailed validation error messages automatically!

### 7.4 Test Cases Executed

| Endpoint | Test Scenario | Expected Result | Status |

|----------|--------------|-----------------|--------|

| GET /api/status | Health check | 200 OK with version info | ✅ Pass |

| POST /api/login | Valid credentials | 200 OK with token | ✅ Pass |

| POST /api/login | Invalid credentials | 401 Unauthorized | ✅ Pass |

| GET /api/students | Without auth | 401 Unauthorized | ✅ Pass |

| GET /api/students | With valid token | 200 OK with array | ✅ Pass |

| POST /api/students | Create with valid data | 201 Created | ✅ Pass |

| POST /api/students | Create with invalid data | 422 Error | ✅ Pass |

| POST /api/students | Duplicate email | 422 Error | ✅ Pass |

| GET /api/students/1 | Get existing student | 200 OK | ✅ Pass |

| GET /api/students/999 | Get non-existent | 404 Not Found | ✅ Pass |

| PUT /api/students/1 | Update existing | 200 OK | ✅ Pass |

| DELETE /api/students/1 | Delete with auth | 200 OK | ✅ Pass |

| GET /api/courses | Retrieve all courses | 200 OK | ✅ Pass |

| POST /api/courses | Create with auth | 201 Created | ✅ Pass |

| POST /api/enrollments | Invalid student_id | 422 Error | ✅ Pass |

| POST /api/enrollments | Duplicate enrollment | 422 Error | ✅ Pass |

| POST /api/enrollments | Valid enrollment | 201 Created | ✅ Pass |

| GET /api/enrollments | With relationships | 200 OK with nested data | ✅ Pass |

### 7.5 Validation Results

**All 12 endpoints operational:**

- ✅ Health check returns proper status

- ✅ Authentication system works correctly

- ✅ All GET endpoints return proper data

- ✅ POST endpoints create resources correctly

- ✅ PUT endpoints update existing resources

- ✅ DELETE endpoints remove resources

- ✅ Authentication required on all protected endpoints

- ✅ Error handling returns appropriate status codes

- ✅ CORS headers present on all responses

- ✅ Database foreign key constraints work correctly

- ✅ Cascade deletes function as expected

- ✅ Eloquent relationships load correctly

---

## 8. Laravel Migration from Vanilla PHP

### 8.1 Why Migrate to Laravel?

The vanilla PHP implementation worked, but it had limitations:

**Issues with Vanilla PHP:**

- Manual routing with regex patterns

- Manual PDO connection management

- Manual validation logic

- Manual error handling

- Manual JSON serialization

- Hardcoded authentication token

- No built-in testing framework

- Lots of boilerplate code

**Benefits of Laravel:**

- Clean routing system

- Eloquent ORM (no raw SQL needed)

- Built-in validation system

- Automatic error handling

- Automatic JSON responses

- Laravel Sanctum for secure authentication

- PHPUnit testing framework included

- Less code, more functionality

### 8.2 Migration Process

**Step 1: Install Laravel**

```bash
composer create-project laravel/laravel student-api-laravel
cd student-api-laravel
```

**Step 2: Install Laravel Sanctum**

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

**Step 3: Create Migrations**

```bash
php artisan make:migration create_students_table
php artisan make:migration create_courses_table
php artisan make:migration create_enrollments_table
```

**Step 4: Create Models**

```bash
php artisan make:model Student
php artisan make:model Course
php artisan make:model Enrollment
```

**Step 5: Create Controllers**

```bash
php artisan make:controller StudentController
php artisan make:controller CourseController
php artisan make:controller EnrollmentController
php artisan make:controller AuthController
```

**Step 6: Define Routes**

Edit `routes/api.php` to define all API endpoints.

**Step 7: Implement Controllers**

Move business logic from `index.php` to appropriate controllers.

**Step 8: Test Everything**

Use Postman to verify all endpoints work correctly.

### 8.3 Code Comparison

**Before (Vanilla PHP):**

```php
// Manual routing
if ($method === 'GET' && preg_match('#^/students/(\d+)$#', $path, $m)) {
    $id = (int)$m[1];
    $student = Database::getStudentById($id);
    if ($student) Response::json($student);
    Response::json(['error' => 'Student not found'], 404);
}

// Manual PDO
$stmt = self::connect()->prepare('SELECT * FROM students WHERE id = ?');
$stmt->execute([$id]);
return $stmt->fetch();
```

**After (Laravel):**

```php
// Clean routing
Route::get('/students/{id}', [StudentController::class, 'show']);

// Eloquent ORM
$student = Student::findOrFail($id);
return response()->json($student);
```

**Much cleaner!**

### 8.4 What Stayed the Same

- Same database schema (students, courses, enrollments)

- Same REST API endpoints

- Same functionality (CRUD operations)

- Same security requirements (authentication for sensitive operations)

### 8.5 What Improved

- **Code Quality:** Much less boilerplate, more readable

- **Security:** Laravel Sanctum vs hardcoded token

- **Validation:** Built-in validation vs manual checks

- **Error Handling:** Automatic error responses

- **Database:** Eloquent ORM vs raw PDO

- **Testing:** PHPUnit included vs manual testing only

- **Deployment:** Docker/Sail setup included

---

## 9. Challenges & Solutions

### 9.1 Challenge: Understanding Laravel Sanctum

**Problem:** Initially, I was confused about how Sanctum tokens work. The vanilla PHP version used a simple hardcoded Bearer token, but Sanctum is more complex.

**Solution:** Read Laravel documentation and realized that:
- Tokens are stored in the database
- Each token is tied to a user
- Tokens can be revoked (logout)
- Middleware handles validation automatically

Much more secure than the hardcoded token approach!

### 9.2 Challenge: All Endpoints Requiring Auth

**Problem:** In the vanilla PHP version, only some endpoints required authentication. In Laravel, I initially made ALL endpoints require auth, which made testing harder.

**Solution:** Kept it that way! It's actually more secure. The health check endpoint doesn't require auth, and login doesn't require auth (obviously). Everything else does. This prevents unauthorized access to any data.

### 9.3 Challenge: Eloquent Relationships

**Problem:** Understanding how to define and use Eloquent relationships was initially confusing.

**Solution:** 
- `hasMany`: One-to-many (Student has many Enrollments)
- `belongsTo`: Many-to-one (Enrollment belongs to Student)
- `belongsToMany`: Many-to-many (Student belongs to many Courses through Enrollments)

Once I understood the relationship types, it became intuitive. The `with()` method for eager loading is incredibly powerful.

### 9.4 Challenge: Validation Rules

**Problem:** Learning all the Laravel validation rules and when to use them.

**Solution:** Laravel's validation is well-documented. Key rules I used:
- `required`: Field must be present
- `email`: Must be valid email
- `unique:table,column`: Must be unique in database
- `exists:table,column`: Must exist in database
- `sometimes`: Only validate if field is present (for partial updates)

The `sometimes` rule was particularly useful for PUT requests where you want to allow partial updates.

### 9.5 Challenge: Docker/Sail Setup

**Problem:** Setting up Docker and Laravel Sail was initially confusing, especially on Windows with WSL.

**Solution:** Laravel Sail makes it easy:
- `compose.yaml` is pre-configured
- `./vendor/bin/sail up` starts everything
- Database migrations run automatically
- No need to manually configure PHP, MySQL, etc.

The setup scripts (`setup.sh` and `run.sh`) make it even easier.

### 9.6 Challenge: Migration from PDO to Eloquent

**Problem:** Rewriting all the PDO queries to use Eloquent.

**Solution:** Eloquent is actually simpler:
- `Student::all()` instead of `SELECT * FROM students`
- `Student::find($id)` instead of `SELECT * FROM students WHERE id = ?`
- `$student->update($data)` instead of `UPDATE students SET ...`
- `$student->delete()` instead of `DELETE FROM students WHERE id = ?`

The relationship loading (`with()`) is a huge bonus - no manual JOINs needed!

---

## 10. Future Enhancements

### 10.1 Security Improvements

**Enhanced Authentication:**

- Implement refresh tokens for long-lived sessions

- Add rate limiting to prevent abuse

- Implement role-based access control (RBAC)

- Add IP whitelisting for sensitive operations

- Implement API key authentication for service-to-service communication

**Input Sanitization:**

- Add HTML entity encoding for XSS protection

- Implement file upload validation (if file uploads are added)

- Add request size limits

### 10.2 Advanced Features

**Pagination:**

- Add pagination for large result sets using Laravel's built-in pagination

- Implement `?page=1&per_page=10` query parameters

- Return metadata (total count, page info, links)

**Filtering and Search:**

- Add filtering capabilities (e.g., `?email=john@example.com`)

- Implement full-text search

- Add sorting options (e.g., `?sort=name&order=asc`)

**Field Selection:**

- Allow clients to request specific fields (e.g., `?fields=id,name`)

- Reduce payload size for mobile clients

**Caching:**

- Implement Redis caching for frequently accessed data

- Cache query results to reduce database load

### 10.3 Testing Improvements

- **Unit Tests:** Add PHPUnit tests for individual controller methods

- **Feature Tests:** Test full request/response cycles

- **Integration Tests:** Test database operations with test database

- **Automated Testing:** Set up CI/CD pipeline with GitHub Actions

- **API Documentation:** Generate OpenAPI/Swagger specification

- **Postman Collection:** Export and share Postman collection

### 10.4 Documentation

- **API Documentation:** Interactive API docs (Swagger/OpenAPI)

- **Code Comments:** Add PHPDoc comments to all methods

- **Architecture Diagrams:** Visual representation of system design

- **Deployment Guide:** Step-by-step production deployment instructions

- **Developer Guide:** Onboarding guide for new developers

### 10.5 Performance Optimization

- **Database Indexing:** Add indexes for frequently queried fields

- **Query Optimization:** Use eager loading to prevent N+1 queries

- **Response Compression:** Enable gzip compression for API responses

- **CDN Integration:** Serve static assets via CDN

- **Database Connection Pooling:** Optimize database connections

---
### 5.x Deployment Scripts (run.sh and setup.sh)

For Stage 2 I added two shell scripts to make deployment repeatable:

- `run.sh` – starts the Laravel Sail environment (or plain PHP server), runs migrations,
  and launches the API. This script is used for the “Deploy with a shell script” part
  of the assignment.

- `setup.sh` – builds and runs the Docker containers (Laravel app + MySQL) using
  `docker compose`. This script is used for the “Deploy with Docker” grading section.




---

## 11. Conclusion

### 11.1 Mission Accomplished

I successfully migrated my vanilla PHP REST API to Laravel with Eloquent ORM. All 12 endpoints work correctly, and authentication is handled securely with Laravel Sanctum.

**What I Got Done:**

- ✅ Migrated from vanilla PHP to Laravel 10

- ✅ Replaced PDO with Eloquent ORM

- ✅ Implemented Laravel Sanctum for secure authentication

- ✅ All 12 REST API endpoints working

- ✅ All endpoints secured with token authentication (except health check and login)

- ✅ MySQL database integration with Laravel migrations

- ✅ Foreign key relationships and cascade deletes

- ✅ Clean MVC architecture following Laravel conventions

- ✅ Built-in validation system

- ✅ CORS support configured

- ✅ Docker/Sail setup for easy deployment

- ✅ Everything tested and validated with Postman

- ✅ Comprehensive documentation (this report)

### 11.2 What I Learned

This migration taught me a lot about Laravel and modern PHP development:

- How Laravel's MVC architecture works

- Eloquent ORM and how it simplifies database operations

- Laravel Sanctum for API token authentication

- Laravel's validation system and how powerful it is

- Database migrations and why they're better than raw SQL

- Laravel's routing system and middleware

- Docker and Laravel Sail for containerized development

- How much boilerplate Laravel eliminates

- Best practices for REST API development in Laravel

- The importance of following framework conventions

Honestly, Laravel makes PHP development so much more enjoyable. The framework handles all the tedious stuff, so I can focus on building features.

### 11.3 Comparison: Vanilla PHP vs Laravel

| Aspect | Vanilla PHP | Laravel |

|--------|-------------|---------|

| Routing | Manual regex patterns | Clean route definitions |

| Database | Raw PDO queries | Eloquent ORM |

| Validation | Manual checks | Built-in validation |

| Authentication | Hardcoded token | Laravel Sanctum |

| Error Handling | Manual responses | Automatic error handling |

| Code Lines | ~500+ lines | ~200 lines (for same functionality) |

| Testing | Manual Postman only | PHPUnit included |

| Deployment | Manual setup | Docker/Sail included |

**Verdict:** Laravel is definitely the way to go for any serious PHP project. The productivity gains are massive.

### 11.4 Ready for Production?

The Laravel version is much closer to production-ready than the vanilla PHP version:

- ✅ Real database persistence with Eloquent ORM

- ✅ Secure database access (automatic prepared statements)

- ✅ Proper error handling (automatic)

- ✅ Input validation (built-in system)

- ✅ Secure authentication (Laravel Sanctum)

- ✅ Database migrations (version-controlled schema)

- ✅ Docker deployment (containerized)

**Still needs for production:**

- Environment variable configuration (`.env` file)

- Rate limiting implementation

- Comprehensive logging setup

- Unit/integration tests

- HTTPS enforcement

- Database backup strategy

- Monitoring and alerting

The Laravel foundation makes adding these features straightforward.

### 11.5 Final Thoughts

Migrating to Laravel was absolutely worth it. The code is cleaner, more maintainable, and follows industry best practices. Eloquent ORM is a game-changer - no more writing raw SQL queries. Laravel Sanctum provides proper token-based authentication instead of a hardcoded token.

The framework handles so much automatically that I can focus on building features instead of writing boilerplate. The MVC architecture makes the codebase easy to navigate and understand.

Overall, I'm very happy with how this migration turned out. The Laravel version is more secure, more maintainable, and more feature-rich than the vanilla PHP version, with significantly less code.

---

## Appendices

### Appendix A: Complete API Reference

**Base URL:** `http://localhost/api`

| Method | Endpoint | Auth | Description |

|--------|----------|------|-------------|

| GET | /api/status | - | Health check |

| POST | /api/login | - | Authenticate and get token |

| POST | /api/logout | 🔒 | Invalidate current token |

| GET | /api/students | 🔒 | List all students |

| GET | /api/students/{id} | 🔒 | Get student by ID |

| POST | /api/students | 🔒 | Create student |

| PUT | /api/students/{id} | 🔒 | Update student |

| DELETE | /api/students/{id} | 🔒 | Delete student |

| GET | /api/courses | 🔒 | List all courses |

| GET | /api/courses/{id} | 🔒 | Get course by ID |

| POST | /api/courses | 🔒 | Create course |

| PUT | /api/courses/{id} | 🔒 | Update course |

| DELETE | /api/courses/{id} | 🔒 | Delete course |

| GET | /api/enrollments | 🔒 | List enrollments |

| POST | /api/enrollments | 🔒 | Create enrollment |

| DELETE | /api/enrollments/{id} | 🔒 | Delete enrollment |

🔒 = Requires `Authorization: Bearer {token}` header (obtain token via `/api/login`)

### Appendix B: Environment Setup

**Requirements:**

- PHP 8.1 or higher

- Composer

- Docker and Docker Compose (for Laravel Sail)

- MySQL 8.4 (included in Docker setup)

- Postman for testing

**Installation Steps:**

1. Clone the repo

2. Install dependencies: `composer install`

3. Copy `.env.example` to `.env`: `cp .env.example .env`

4. Generate application key: `php artisan key:generate`

5. Start Docker containers: `./vendor/bin/sail up -d`

6. Run migrations: `./vendor/bin/sail artisan migrate`

7. Create a test user (via Tinker):

   ```bash
   ./vendor/bin/sail artisan tinker
   >>> $user = new App\Models\User();
   >>> $user->name = "Test User";
   >>> $user->email = "user@example.com";
   >>> $user->password = Hash::make('password');
   >>> $user->save();
   ```

8. Test: `curl http://localhost/api/status`

**Alternative Setup (without Docker):**

1. Install PHP 8.1+, Composer, MySQL

2. Configure `.env` with database credentials

3. Run `php artisan migrate`

4. Start server: `php artisan serve`

5. Test: `curl http://localhost:8000/api/status`

### Appendix C: Project Timeline

| Milestone | Planned Date | Actual Date | Status |

|-----------|--------------|-------------|--------|

| Laravel Installation | Nov 26, 2025 | Nov 26, 2025 | ✅ Complete |

| Database Migrations | Nov 27, 2025 | Nov 27, 2025 | ✅ Complete |

| Model Creation | Nov 27, 2025 | Nov 27, 2025 | ✅ Complete |

| Controller Implementation | Nov 28, 2025 | Nov 28, 2025 | ✅ Complete |

| Authentication Setup | Nov 29, 2025 | Nov 29, 2025 | ✅ Complete |

| Route Configuration | Nov 29, 2025 | Nov 29, 2025 | ✅ Complete |

| Testing with Postman | Nov 30, 2025 | Nov 30, 2025 | ✅ Complete |

| Docker/Sail Setup | Dec 1, 2025 | Dec 1, 2025 | ✅ Complete |

| Documentation | Dec 2, 2025 | Dec 2, 2025 | ✅ Complete |

| Migration Complete | Dec 3, 2025 | Dec 3, 2025 | ✅ Complete |

---

**End of Report**

*This document represents the completed migration of the Student Course Management System REST API from vanilla PHP to Laravel 10 with Eloquent ORM and Laravel Sanctum authentication for CSC 640.*

