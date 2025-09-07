# Campus Event Management System - Database Schema

# Database Schema Diagram

## Entity Relationship Diagram

```mermaid
erDiagram
    COLLEGES ||--o{ USERS : has
    COLLEGES ||--o{ EVENTS : hosts
    USERS ||--o{ REGISTRATIONS : makes
    EVENTS ||--o{ REGISTRATIONS : has
    EVENTS ||--o{ ATTENDANCE : tracks
    USERS ||--o{ ATTENDANCE : has
    USERS ||--o{ FEEDBACK : provides
    EVENTS ||--o{ FEEDBACK : receives

    COLLEGES {
        int id PK
        string name
        datetime created_at
    }

    USERS {
        int id PK
        string email UK
        string hashed_password
        string full_name
        string role
        int college_id FK
        boolean is_active
        datetime created_at
    }

    EVENTS {
        int id PK
        string title
        text description
        string type
        datetime start_time
        datetime end_time
        string venue
        int capacity
        int college_id FK
        int created_by FK
        datetime created_at
        datetime updated_at
    }

    REGISTRATIONS {
        int id PK
        int event_id FK
        int user_id FK
        datetime registration_time
        string status
        UNIQUE(event_id, user_id)
    }

    ATTENDANCE {
        int id PK
        int event_id FK
        int user_id FK
        datetime check_in_time
        datetime check_out_time
        string status
        int recorded_by FK
    }

    FEEDBACK {
        int id PK
        int event_id FK
        int user_id FK
        int rating
        text comments
        datetime submitted_at
    }
```

## Table Descriptions

### COLLEGES
- `id` (Primary Key): Unique identifier for each college
- `name`: Name of the college
- `created_at`: Timestamp when the college was added

### USERS
- `id` (Primary Key): Unique user identifier
- `email`: User's email address (unique)
- `hashed_password`: Securely hashed password
- `full_name`: User's full name
- `role`: User role (student, admin, etc.)
- `college_id`: Reference to the user's college
- `is_active`: Whether the user account is active
- `created_at`: Account creation timestamp

### EVENTS
- `id` (Primary Key): Unique event identifier
- `title`: Event title
- `description`: Detailed event description
- `type`: Type/category of the event
- `start_time`: Event start time
- `end_time`: Event end time
- `venue`: Event location
- `capacity`: Maximum number of attendees
- `college_id`: Hosting college
- `created_by`: User who created the event
- `created_at`: Event creation timestamp
- `updated_at`: Last update timestamp

### REGISTRATIONS
- `id` (Primary Key): Unique registration ID
- `event_id`: Reference to the event
- `user_id`: Reference to the user
- `registration_time`: When the registration was made
- `status`: Registration status (pending, confirmed, cancelled)
- Unique constraint on (event_id, user_id)

### ATTENDANCE
- `id` (Primary Key): Unique attendance record ID
- `event_id`: Reference to the event
- `user_id`: Reference to the user
- `check_in_time`: When user checked in
- `check_out_time`: When user checked out (if applicable)
- `status`: Attendance status (present, late, absent)
- `recorded_by`: User who recorded the attendance

### FEEDBACK
- `id` (Primary Key): Unique feedback ID
- `event_id`: Reference to the event
- `user_id`: Reference to the user providing feedback
- `rating`: Numeric rating (e.g., 1-5)
- `comments`: Optional feedback text
- `submitted_at`: When feedback was submitted
    │ 📅 created_at       │ │ 🏫 college_id (FK)  │
    │ 📅 updated_at       │ │ 👤 created_by (FK)  │
    └─────────────────────┘ │ 📅 created_at       │
            │               │ 📅 updated_at       │
            │               └─────────────────────┘
            │ 1:N                    │
            │                        │ 1:N
            │                        │
            │               ┌─────────────────────┐
            │               │    REGISTRATIONS    │
            │               ├─────────────────────┤
            │               │ 🔑 id (PK)          │
            │               │ 👤 student_id (FK)  │◄──────┘
            │               │ 🎪 event_id (FK)    │
            │               │ 📅 created_at       │
            │               └─────────────────────┘
            │                        │
            │                        │ 1:1
            │                        │
            │               ┌─────────────────────┐
            │               │     ATTENDANCE      │
            │               ├─────────────────────┤
            │               │ 🔑 id (PK)          │
            │               │ 📋 registration_id  │◄──────┘
            │               │     (FK)            │
            │               │ 👤 student_id (FK)  │◄──────┘
            │               │ 🎪 event_id (FK)    │
            │               │ ⏰ check_in_time    │
            │               └─────────────────────┘
            │
            │               ┌─────────────────────┐
            │               │      FEEDBACK       │
            │               ├─────────────────────┤
            │               │ 🔑 id (PK)          │
            │               │ 📋 registration_id  │
            │               │     (FK)            │
            │               │ 👤 student_id (FK)  │◄──────┘
            │               │ 🎪 event_id (FK)    │
            │               │ ⭐ rating (1-5)     │
            │               │ 💬 comment          │
            │               │ 📅 created_at       │
            │               └─────────────────────┘

```

## Relationship Details

### 1. COLLEGES → USERS (1:N)
- **Relationship**: One college can have many users
- **Foreign Key**: `users.college_id` → `colleges.id`
- **Description**: Students and admins belong to a specific college

### 2. COLLEGES → EVENTS (1:N)
- **Relationship**: One college can host many events
- **Foreign Key**: `events.college_id` → `colleges.id`
- **Description**: Events are organized by colleges

### 3. USERS → EVENTS (1:N) - Creator Relationship
- **Relationship**: One user (admin) can create many events
- **Foreign Key**: `events.created_by` → `users.id`
- **Description**: Admin users create and manage events

### 4. USERS → REGISTRATIONS (1:N)
- **Relationship**: One user can register for many events
- **Foreign Key**: `registrations.student_id` → `users.id`
- **Description**: Students register for events they want to attend

### 5. EVENTS → REGISTRATIONS (1:N)
- **Relationship**: One event can have many registrations
- **Foreign Key**: `registrations.event_id` → `events.id`
- **Description**: Multiple students can register for the same event

### 6. REGISTRATIONS → ATTENDANCE (1:1)
- **Relationship**: One registration can have one attendance record
- **Foreign Key**: `attendance.registration_id` → `registrations.id`
- **Description**: Attendance is tracked per registration

### 7. REGISTRATIONS → FEEDBACK (1:1)
- **Relationship**: One registration can have one feedback
- **Foreign Key**: `feedback.registration_id` → `registrations.id`
- **Description**: Students can provide feedback for events they attended

## Table Structures

### COLLEGES
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | Unique college identifier |
| name | VARCHAR | NOT NULL | College name |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Record creation time |

### USERS
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | Unique user identifier |
| email | VARCHAR | UNIQUE, NOT NULL, INDEXED | User email address |
| hashed_password | VARCHAR | NOT NULL | Encrypted password |
| full_name | VARCHAR | NOT NULL | User's full name |
| role | VARCHAR | DEFAULT 'student' | User role (student/admin) |
| college_id | INTEGER | FOREIGN KEY → colleges.id | Associated college |
| is_active | BOOLEAN | DEFAULT TRUE | Account status |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Account creation time |
| updated_at | DATETIME | DEFAULT CURRENT_TIMESTAMP, ON UPDATE | Last modification time |

### EVENTS
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | Unique event identifier |
| title | VARCHAR | NOT NULL | Event title |
| description | TEXT | NULLABLE | Event description |
| type | VARCHAR | NOT NULL | Event type (Workshop, Fest, Seminar, etc.) |
| date | DATETIME | NOT NULL | Event date and time |
| location | VARCHAR | NULLABLE | Event venue |
| max_attendees | INTEGER | NULLABLE | Maximum capacity |
| college_id | INTEGER | FOREIGN KEY → colleges.id, NOT NULL | Hosting college |
| created_by | INTEGER | FOREIGN KEY → users.id, NOT NULL | Event creator (admin) |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Record creation time |
| updated_at | DATETIME | DEFAULT CURRENT_TIMESTAMP, ON UPDATE | Last modification time |

### REGISTRATIONS
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | Unique registration identifier |
| student_id | INTEGER | FOREIGN KEY → users.id, NOT NULL | Registered student |
| event_id | INTEGER | FOREIGN KEY → events.id, NOT NULL | Target event |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Registration time |

### ATTENDANCE
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | Unique attendance identifier |
| registration_id | INTEGER | FOREIGN KEY → registrations.id, NOT NULL | Associated registration |
| student_id | INTEGER | FOREIGN KEY → users.id, NOT NULL | Student who attended |
| event_id | INTEGER | FOREIGN KEY → events.id, NOT NULL | Event attended |
| check_in_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | Attendance timestamp |

### FEEDBACK
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | Unique feedback identifier |
| registration_id | INTEGER | FOREIGN KEY → registrations.id, NOT NULL | Associated registration |
| student_id | INTEGER | FOREIGN KEY → users.id, NOT NULL | Student providing feedback |
| event_id | INTEGER | FOREIGN KEY → events.id, NOT NULL | Event being rated |
| rating | INTEGER | NOT NULL, CHECK (rating BETWEEN 1 AND 5) | Rating score (1-5) |
| comment | TEXT | NULLABLE | Optional feedback comment |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Feedback submission time |

## Key Features

### 🔐 Authentication & Authorization
- JWT-based authentication system
- Role-based access control (admin/student)
- Password hashing with bcrypt

### 🎯 Core Functionality
- **Event Management**: CRUD operations for events
- **Registration System**: Students can register for events
- **Attendance Tracking**: QR code-based check-in system
- **Feedback System**: Post-event ratings and comments
- **College Management**: Multi-college support

### 📊 Data Integrity
- Foreign key constraints ensure referential integrity
- Unique constraints prevent duplicate registrations
- Check constraints validate rating values (1-5)
- Timestamps track all data modifications

### 🔍 Indexing Strategy
- Primary keys are automatically indexed
- Email field is indexed for fast user lookups
- Foreign keys are indexed for efficient joins

## Database Technology
- **Engine**: SQLite (Development) / PostgreSQL (Production recommended)
- **ORM**: SQLAlchemy with declarative base
- **Migration**: Alembic (recommended for production)
