# Campus Event Management System - Workflows

## Overview
This document outlines the key workflows in the Campus Event Management System using sequence diagrams. The workflows cover the complete lifecycle from event registration through attendance tracking to reporting and analytics.

## Table of Contents
- [Student Registration Workflow](#student-registration-workflow)
- [Attendance Tracking Workflow](#attendance-tracking-workflow)
- [Event Management Workflow](#event-management-workflow)
- [Feedback Collection Workflow](#feedback-collection-workflow)
- [Reporting Workflow](#reporting-workflow)
- [Complete End-to-End Workflow](#complete-end-to-end-workflow)
- [Error Handling Workflows](#error-handling-workflows)

---

## Student Registration Workflow

### 📝 Event Registration Process

```mermaid
sequenceDiagram
    participant S as Student
    participant FE as Frontend App
    participant API as FastAPI Backend
    participant DB as Database
    participant Auth as Auth Service
    participant Email as Email Service

    %% Student Browsing Events
    S->>FE: Browse available events
    FE->>API: GET /api/events?status=upcoming
    API->>DB: Query events with capacity
    DB-->>API: Return events list
    API-->>FE: Return events data
    FE-->>S: Display events with details

    %% Registration Process
    S->>FE: Select event to register
    FE->>Auth: Verify authentication
    Auth-->>FE: Return auth status
    
    alt Not authenticated
        FE-->>S: Show login/register prompt
    else Authenticated
        FE->>API: POST /api/events/{id}/register
        API->>DB: Check event capacity
        API->>DB: Check existing registration
        
        alt Already registered
            API-->>FE: Return 409 Conflict
            FE-->>S: Show already registered message
        else Capacity available
            API->>DB: Create registration
            API->>Email: Send confirmation email
            API-->>FE: Return success response
            FE-->>S: Show success message
        else Event full
            API->>DB: Add to waitlist
            API-->>FE: Return waitlist status
            FE-->>S: Show waitlist confirmation
        end
    end
```

## Attendance Tracking Workflow

### 🎯 QR Code Attendance Process

```mermaid
sequenceDiagram
    participant A as Admin
    participant S as Student
    participant FE as Frontend App
    participant API as FastAPI Backend
    participant DB as Database
    participant Notif as Notification Service

    %% Admin initiates attendance
    A->>FE: Open attendance for event
    FE->>API: GET /api/events/{id}/attendance/qr
    API->>DB: Generate unique QR code
    DB-->>API: Return QR data
    API-->>FE: Return QR code
    FE-->>A: Display QR code

    %% Student scans QR code
    S->>S: Scan QR code with mobile app
    S->>FE: Send QR data
    FE->>API: POST /api/attendance/scan
    API->>DB: Validate QR code
    
    alt Valid QR code
        API->>DB: Record attendance
        API->>Notif: Send confirmation
        API-->>FE: Success response
        FE-->>S: Show attendance confirmed
    else Invalid/Expired QR
        API-->>FE: Error response
        FE-->>S: Show error message
    end
```

## Event Management Workflow

### 🎪 Event Creation Process

```mermaid
sequenceDiagram
    participant A as Admin
    participant FE as Frontend App
    participant API as FastAPI Backend
    participant DB as Database
    participant Notif as Notification Service

    A->>FE: Fill event creation form
    FE->>A: Show form validation
    
    A->>FE: Submit event details
    FE->>API: POST /api/events
    API->>DB: Validate event data
    
    alt Validation passed
        API->>DB: Create event
        API->>Notif: Notify subscribers
        API-->>FE: Success response
        FE-->>A: Show success message
    else Validation failed
        API-->>FE: Error details
        FE-->>A: Show error messages
    end
```

## Feedback Collection Workflow

### 💬 Feedback Submission Process

```mermaid
sequenceDiagram
    participant S as Student
    participant FE as Frontend App
    participant API as FastAPI Backend
    participant DB as Database
    participant Analytics as Analytics Service

    S->>FE: Open attended events
    FE->>API: GET /api/events?attended=true
    API-->>FE: Return attended events
    FE-->>S: Show feedback form
    
    S->>FE: Submit feedback (rating + comments)
    FE->>API: POST /api/events/{id}/feedback
    API->>DB: Save feedback
    API->>Analytics: Update event metrics
    API-->>FE: Success response
    FE-->>S: Show thank you message
```

## Reporting Workflow

### 📊 Generate Event Report

```mermaid
sequenceDiagram
    participant A as Admin
    participant FE as Frontend App
    participant API as FastAPI Backend
    participant DB as Database
    participant Cache as Cache Service

    A->>FE: Request event report
    FE->>API: GET /api/reports/events/{id}
    API->>Cache: Check cached report
    
    alt Cached report exists
        Cache-->>API: Return cached data
    else No cache
        API->>DB: Query event data
        API->>DB: Query attendance data
        API->>DB: Query feedback data
        API->>Cache: Cache results (5 min TTL)
    end
    
    API-->>FE: Return report data
    FE-->>A: Display report with charts
```

## Complete End-to-End Workflow

### 🔄 Event Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Event_Creation
    Event_Creation --> Registration_Open: Admin publishes
    Registration_Open --> Registration_Closed: Registration period ends
    Registration_Open --> Event_Cancelled: Admin cancels
    Registration_Closed --> Event_Ongoing: Event starts
    Event_Ongoing --> Attendance_Tracking: Event in progress
    Attendance_Tracking --> Feedback_Collection: Event ends
    Feedback_Collection --> Reporting: All feedback collected
    Reporting --> [*]
    
    state Event_Creation {
        [*] --> Draft
        Draft --> Published: Admin approves
    }
    
    state Attendance_Tracking {
        [*] --> Check_In
        Check_In --> Check_Out: Student attends
        Check_Out --> [*]
    }
```

## Error Handling Workflows

### ⚠️ Registration Error Handling

```mermaid
sequenceDiagram
    participant S as Student
    participant FE as Frontend App
    participant API as FastAPI Backend
    participant DB as Database
    
    S->>FE: Attempt to register
    FE->>API: POST /api/events/{id}/register
    
    alt Event full
        API-->>FE: 409 Conflict
        FE-->>S: Show "Event Full" with waitlist option
    else Already registered
        API-->>FE: 400 Bad Request
        FE-->>S: Show "Already Registered"
    else Server error
        API-->>FE: 500 Internal Server Error
        FE-->>S: Show error with retry option
    else Network failure
        FE-->>S: Show offline message
        FE->>FE: Queue request for retry
    end
        FE->>S: Redirect to login
        S->>FE: Enter credentials
        FE->>API: POST /auth/login
        API->>Auth: Verify credentials
        Auth->>DB: Check user credentials
        DB-->>Auth: User data
        Auth-->>API: JWT token
        API-->>FE: Access token
        FE->>FE: Store token
    end

    S->>FE: Click "Register for Event"
    FE->>API: POST /registrations
    Note right of API: Authorization: Bearer {token}
    
    API->>Auth: Verify JWT token
    Auth-->>API: User validated
    
    API->>DB: Check if already registered
    DB-->>API: Registration status
    
    alt Already registered
        API-->>FE: 400 Bad Request
        FE-->>S: "Already registered" message
    else Registration allowed
        API->>DB: Check event capacity
        DB-->>API: Current registrations count
        
        alt Event full
            API-->>FE: 400 Bad Request
            FE-->>S: "Event is full" message
        else Space available
            API->>DB: INSERT into registrations
            DB-->>API: Registration created
            API-->>FE: 201 Created - Registration success
            FE-->>S: "Successfully registered!" message
            FE->>FE: Update UI - show registered status
        end
    end
```

### 📋 Registration Validation Flow

```mermaid
sequenceDiagram
    participant API as FastAPI Backend
    participant DB as Database
    participant Valid as Validation Service

    Note over API, Valid: Registration Validation Process

    API->>Valid: Validate registration request
    Valid->>DB: Check event exists
    DB-->>Valid: Event details
    
    Valid->>DB: Check user role (student only)
    DB-->>Valid: User role confirmed
    
    Valid->>DB: Check duplicate registration
    DB-->>Valid: Registration status
    
    Valid->>DB: Check event capacity
    DB-->>Valid: Current vs max attendees
    
    Valid->>DB: Check event date (future)
    DB-->>Valid: Event date validation
    
    Valid-->>API: Validation result
    
    alt Validation passed
        API->>DB: Create registration record
        DB-->>API: Registration ID
    else Validation failed
        API-->>API: Return appropriate error
    end
```

---

## Attendance Tracking Workflow

### ✅ QR Code Attendance Flow (Admin Scanning)

```mermaid
sequenceDiagram
    participant A as Admin
    participant MA as Mobile App
    participant S as Student
    participant QR as QR Generator
    participant API as FastAPI Backend
    participant DB as Database

    Note over A, DB: QR Code Attendance Tracking (Admin Mode)

    S->>QR: Request attendance QR code
    QR->>DB: Get student & event details
    DB-->>QR: Student and event data
    QR->>QR: Generate QR with student ID & event ID
    QR-->>S: Display QR code on screen

    A->>MA: Open QR scanner
    MA->>MA: Activate camera
    A->>MA: Scan student's QR code
    MA->>MA: Decode QR data
    
    MA->>API: POST /attendance/qr
    Note right of API: QR data: {"type":"attendance","eventId":1,"studentId":5}
    
    API->>API: Parse QR code data
    API->>DB: Validate event exists
    DB-->>API: Event details
    
    API->>DB: Check student registration
    DB-->>API: Registration record
    
    alt Student not registered
        API-->>MA: 400 Bad Request
        MA-->>A: "Student not registered" error
    else Student registered
        API->>DB: Check existing attendance
        DB-->>API: Attendance status
        
        alt Already marked
            API-->>MA: Success: false, "Already marked"
            MA-->>A: "Attendance already recorded"
        else Not marked
            API->>DB: INSERT attendance record
            DB-->>API: Attendance ID
            API-->>MA: Success: true, attendance_id
            MA-->>A: "Attendance marked successfully"
            MA->>MA: Show success animation
        end
    end
```

### 📱 Student Self-Attendance Flow

```mermaid
sequenceDiagram
    participant S as Student
    participant SA as Student App
    participant QR as Event QR Code
    participant API as FastAPI Backend
    participant DB as Database

    Note over S, DB: Student Self-Attendance Flow

    S->>SA: Open event details
    SA->>API: GET /events/{event_id}
    API->>DB: Get event details
    DB-->>API: Event data
    API-->>SA: Event information
    SA-->>S: Show event details

    S->>QR: Scan event QR code at venue
    QR-->>S: Event QR data
    S->>SA: QR code scanned
    SA->>SA: Extract event ID from QR

    SA->>API: POST /attendance/qr/student
    Note right of API: Body: {"event_id": 1}
    
    API->>API: Verify student JWT token
    API->>DB: Check student registration
    DB-->>API: Registration record
    
    alt Not registered
        API-->>SA: 400 Bad Request
        SA-->>S: "You're not registered for this event"
    else Registered
        API->>DB: Check existing attendance
        DB-->>API: Attendance status
        
        alt Already marked
            API-->>SA: Success: false, "Already marked"
            SA-->>S: "You've already checked in"
        else Not marked
            API->>DB: INSERT attendance record
            DB-->>API: Attendance created
            API-->>SA: Success: true, attendance details
            SA-->>S: "Check-in successful!"
            SA->>SA: Update UI - show attended status
        end
    end
```

### 📊 Attendance Analytics Flow

```mermaid
sequenceDiagram
    participant A as Admin
    participant FE as Frontend
    participant API as FastAPI Backend
    participant DB as Database
    participant Analytics as Analytics Service

    Note over A, Analytics: Real-time Attendance Analytics

    A->>FE: View event dashboard
    FE->>API: GET /events/{event_id}
    API->>DB: Get event details
    DB-->>API: Event data with registrations
    API-->>FE: Event information

    FE->>API: GET /reports/events/{event_id}/attendance
    API->>Analytics: Calculate attendance metrics
    Analytics->>DB: Query attendance records
    DB-->>Analytics: Attendance data
    Analytics->>DB: Query registration records  
    DB-->>Analytics: Registration data
    Analytics->>Analytics: Calculate attendance rate
    Analytics->>Analytics: Generate hourly check-in stats
    Analytics-->>API: Attendance analytics
    API-->>FE: Attendance report
    FE-->>A: Display attendance dashboard

    loop Real-time updates (every 30 seconds)
        FE->>API: GET /reports/events/{event_id}/attendance
        API->>Analytics: Get latest metrics
        Analytics->>DB: Query recent attendance
        DB-->>Analytics: Updated data
        Analytics-->>API: Updated metrics
        API-->>FE: Refreshed data
        FE->>FE: Update dashboard charts
    end
```

---

## Reporting Workflow

### 📈 Event Performance Report Generation

```mermaid
sequenceDiagram
    participant A as Admin
    participant FE as Frontend
    participant API as FastAPI Backend
    participant DB as Database
    participant Report as Report Generator
    participant Cache as Redis Cache

    Note over A, Cache: Event Performance Report Workflow

    A->>FE: Request event report
    FE->>API: GET /reports/events/{event_id}/performance
    
    API->>Cache: Check cached report
    Cache-->>API: Cache miss
    
    API->>Report: Generate performance report
    Report->>DB: Query event details
    DB-->>Report: Event data
    
    Report->>DB: Query registrations by date
    DB-->>Report: Registration timeline
    
    Report->>DB: Query attendance records
    DB-->>Report: Attendance data
    
    Report->>DB: Query feedback & ratings
    DB-->>Report: Feedback data
    
    Report->>Report: Calculate KPIs
    Note right of Report: - Registration rate<br/>- Attendance rate<br/>- Average rating<br/>- Capacity utilization
    
    Report->>Report: Generate charts data
    Note right of Report: - Registration trends<br/>- Check-in patterns<br/>- Rating distribution
    
    Report-->>API: Complete report data
    API->>Cache: Cache report (5 min TTL)
    API-->>FE: Report JSON
    FE->>FE: Render charts & metrics
    FE-->>A: Display performance dashboard
```

### 📊 College Analytics Workflow

```mermaid
sequenceDiagram
    participant A as Admin
    participant FE as Frontend
    participant API as FastAPI Backend
    participant DB as Database
    participant Analytics as Analytics Engine
    participant Export as Export Service

    Note over A, Export: College-wide Analytics Workflow

    A->>FE: Access college analytics
    FE->>API: GET /reports/colleges/{college_id}/analytics
    
    API->>Analytics: Generate college report
    Analytics->>DB: Query all college events
    DB-->>Analytics: Events data
    
    Analytics->>DB: Query student engagement
    DB-->>Analytics: Registration & attendance data
    
    Analytics->>DB: Query feedback trends
    DB-->>Analytics: Feedback & ratings
    
    Analytics->>Analytics: Calculate college KPIs
    Note right of Analytics: - Total events hosted<br/>- Student participation rate<br/>- Average event rating<br/>- Most popular event types
    
    Analytics->>Analytics: Generate comparative metrics
    Note right of Analytics: - Month-over-month growth<br/>- Event type performance<br/>- Student retention rates
    
    Analytics-->>API: College analytics data
    API-->>FE: Analytics JSON
    FE->>FE: Render analytics dashboard
    FE-->>A: Display college performance

    alt Export report requested
        A->>FE: Click "Export Report"
        FE->>API: GET /reports/colleges/{college_id}/export
        API->>Export: Generate PDF/Excel report
        Export->>Analytics: Get formatted data
        Analytics-->>Export: Report data
        Export->>Export: Create formatted document
        Export-->>API: Generated file
        API-->>FE: File download link
        FE-->>A: Download report file
    end
```

### 📋 Student Performance Tracking

```mermaid
sequenceDiagram
    participant S as Student
    participant A as Admin
    participant FE as Frontend
    participant API as FastAPI Backend
    participant DB as Database
    participant Profile as Profile Service

    Note over S, Profile: Student Performance Tracking

    alt Student viewing own profile
        S->>FE: Access my profile
        FE->>API: GET /reports/students/me/analytics
    else Admin viewing student profile
        A->>FE: Search student
        FE->>API: GET /users (admin only)
        API->>DB: Query users
        DB-->>API: Users list
        API-->>FE: Students data
        A->>FE: Select student
        FE->>API: GET /reports/students/{student_id}/analytics
    end
    
    API->>Profile: Generate student analytics
    Profile->>DB: Query student registrations
    DB-->>Profile: Registration history
    
    Profile->>DB: Query attendance records
    DB-->>Profile: Attendance data
    
    Profile->>DB: Query feedback submissions
    DB-->>Profile: Feedback history
    
    Profile->>Profile: Calculate student metrics
    Note right of Profile: - Events registered<br/>- Attendance rate<br/>- Feedback participation<br/>- Favorite event types
    
    Profile->>Profile: Generate engagement timeline
    Profile->>Profile: Calculate performance trends
    
    Profile-->>API: Student analytics
    API-->>FE: Student performance data
    FE->>FE: Render student dashboard
    FE-->>S: Display personal analytics
```

---

## Complete End-to-End Workflow

### 🔄 Full Event Lifecycle

```mermaid
sequenceDiagram
    participant A as Admin
    participant S as Student
    participant API as FastAPI Backend
    participant DB as Database
    participant Email as Email Service
    participant Analytics as Analytics

    Note over A, Analytics: Complete Event Lifecycle Workflow

    %% Event Creation
    A->>API: POST /events (Create event)
    API->>DB: INSERT event record
    DB-->>API: Event created
    API-->>A: Event details

    %% Student Registration Phase
    S->>API: GET /events (Browse events)
    API->>DB: Query available events
    DB-->>API: Events list
    API-->>S: Available events

    S->>API: POST /registrations (Register)
    API->>DB: INSERT registration
    DB-->>API: Registration created
    API->>Email: Send confirmation email
    API-->>S: Registration confirmed

    %% Event Day - Attendance
    S->>API: POST /attendance/qr/student (Check-in)
    API->>DB: INSERT attendance record
    DB-->>API: Attendance marked
    API-->>S: Check-in successful

    %% Post-Event - Feedback
    S->>API: POST /feedback (Submit feedback)
    API->>DB: INSERT feedback record
    DB-->>API: Feedback saved
    API-->>S: Feedback submitted

    %% Analytics & Reporting
    A->>API: GET /reports/events/{id}/performance
    API->>Analytics: Generate event report
    Analytics->>DB: Query all event data
    DB-->>Analytics: Complete event metrics
    Analytics-->>API: Performance report
    API-->>A: Event analytics dashboard
```

---

## Error Handling Workflows

### ⚠️ Registration Error Scenarios

```mermaid
sequenceDiagram
    participant S as Student
    participant FE as Frontend
    participant API as FastAPI Backend
    participant DB as Database

    Note over S, DB: Registration Error Handling

    S->>FE: Attempt event registration
    FE->>API: POST /registrations

    alt Invalid/Expired Token
        API-->>FE: 401 Unauthorized
        FE->>FE: Clear stored token
        FE-->>S: Redirect to login
    else Event Not Found
        API->>DB: Query event
        DB-->>API: No event found
        API-->>FE: 404 Not Found
        FE-->>S: "Event no longer available"
    else Already Registered
        API->>DB: Check existing registration
        DB-->>API: Registration exists
        API-->>FE: 400 Bad Request
        FE-->>S: "You're already registered"
    else Event Full
        API->>DB: Check capacity
        DB-->>API: Max capacity reached
        API-->>FE: 400 Bad Request
        FE-->>S: "Event is full - join waitlist?"
    else Registration Deadline Passed
        API->>API: Check event date
        API-->>FE: 400 Bad Request
        FE-->>S: "Registration deadline passed"
    else Network Error
        FE->>FE: Retry mechanism (3 attempts)
        FE-->>S: "Connection error - please try again"
    end
```

### 🚨 Attendance Error Scenarios

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile App
    participant API as FastAPI Backend
    participant DB as Database

    Note over U, DB: Attendance Error Handling

    U->>App: Scan QR code for attendance
    App->>API: POST /attendance/qr

    alt Invalid QR Code Format
        API->>API: Parse QR data
        API-->>App: 400 Bad Request
        App-->>U: "Invalid QR code"
    else QR Code Expired
        API->>API: Check timestamp
        API-->>App: 400 Bad Request
        App-->>U: "QR code expired - get new one"
    else Event Not Started
        API->>DB: Check event time
        DB-->>API: Event start time
        API-->>App: 400 Bad Request
        App-->>U: "Event hasn't started yet"
    else Not Registered
        API->>DB: Check registration
        DB-->>API: No registration found
        API-->>App: 400 Bad Request
        App-->>U: "Please register first"
    else Already Checked In
        API->>DB: Check attendance
        DB-->>API: Attendance exists
        API-->>App: 200 OK (success: false)
        App-->>U: "Already checked in"
    else Database Error
        API->>DB: INSERT attendance
        DB-->>API: Database error
        API-->>App: 500 Internal Server Error
        App->>App: Queue for retry
        App-->>U: "Saved - will sync when online"
    end
```

---

## Performance Considerations

### 🚀 Optimization Strategies

1. **Caching Layer**
   - Redis cache for frequent queries
   - Event details cached for 5 minutes
   - Analytics reports cached for 15 minutes

2. **Database Optimization**
   - Indexed foreign keys for fast joins
   - Pagination for large result sets
   - Connection pooling for concurrent requests

3. **Real-time Updates**
   - WebSocket connections for live attendance
   - Server-sent events for dashboard updates
   - Background jobs for report generation

4. **Mobile Optimization**
   - Offline QR code generation
   - Local storage for attendance queue
   - Progressive sync when online

---

## Security Workflows

### 🔒 Authentication & Authorization Flow

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant API as FastAPI Backend
    participant Auth as Auth Service
    participant DB as Database

    Note over U, DB: Security & Authorization Workflow

    U->>FE: Access protected resource
    FE->>FE: Check stored JWT token
    
    alt No token or expired
        FE->>U: Redirect to login
        U->>FE: Enter credentials
        FE->>API: POST /auth/login
        API->>Auth: Verify credentials
        Auth->>DB: Check user & password
        DB-->>Auth: User validation
        Auth-->>API: JWT token
        API-->>FE: Access token
        FE->>FE: Store token securely
    end
    
    FE->>API: Request with Bearer token
    API->>Auth: Validate JWT token
    Auth->>Auth: Check token signature
    Auth->>Auth: Verify expiration
    Auth->>DB: Get user permissions
    DB-->>Auth: User role & status
    Auth-->>API: Authorization result
    
    alt Authorized
        API->>API: Process request
        API-->>FE: Response data
    else Unauthorized
        API-->>FE: 403 Forbidden
        FE-->>U: "Access denied"
    end
```
