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
