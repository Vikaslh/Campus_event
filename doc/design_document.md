# Campus Event Manager - System Design Document

## 1. Introduction

### 1.1 Purpose
This document outlines the system design for the Campus Event Manager, a comprehensive platform for managing college events, student registrations, attendance tracking, and analytics.

### 1.2 System Overview
The Campus Event Manager is a web-based application that enables:
- Event creation and management by administrators
- Student registration for events
- QR code-based attendance tracking
- Real-time analytics and reporting
- Feedback collection and analysis

## 2. System Architecture

### 2.1 High-Level Architecture
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Web Frontend   │◄───►│   Backend API   │◄───►│   Database      │
│  (React)        │     │   (FastAPI)     │     │   (SQLite)      │
│                 │     │                 │     │                 │
└────────┬────────┘     └────────┬────────┘     └─────────────────┘
         │                       │
         │                       │
         ▼                       ▼
┌─────────────────┐     ┌─────────────────┐
│  Mobile App     │     │  Admin Portal   │
│  (React Native) │     │  (Web)          │
└─────────────────┘     └─────────────────┘
```

### 2.2 Technology Stack
- **Frontend**: React.js with TypeScript
- **Mobile**: React Native
- **Backend**: Python FastAPI
- **Database**: SQLite (development), PostgreSQL (production)
- **Authentication**: JWT (JSON Web Tokens)
- **Real-time Updates**: WebSockets
- **Caching**: Redis
- **Containerization**: Docker
- **CI/CD**: GitHub Actions

## 3. Core Components

### 3.1 Authentication Service
- JWT-based authentication
- Role-based access control (Student, Admin, Super Admin)
- Session management
- Password reset functionality

### 3.2 Event Management
- Event CRUD operations
- Capacity management
- Event scheduling and conflicts resolution
- Recurring events support

### 3.3 Registration System
- Student registration for events
- Waitlist management
- Email notifications
- Registration validation

### 3.4 Attendance Tracking
- QR code generation and validation
- Real-time attendance monitoring
- Attendance reports
- Offline mode support

### 3.5 Feedback System
- Event feedback collection
- Rating system
- Sentiment analysis
- Report generation

## 4. API Design

### 4.1 Authentication
- `POST /auth/register` - User registration
- `POST /auth/login` - User login
- `POST /auth/refresh` - Refresh access token
- `POST /auth/password-reset` - Request password reset

### 4.2 Events
- `GET /events` - List all events
- `POST /events` - Create new event (Admin)
- `GET /events/{event_id}` - Get event details
- `PUT /events/{event_id}` - Update event (Admin)
- `DELETE /events/{event_id}` - Delete event (Admin)

### 4.3 Registrations
- `POST /events/{event_id}/register` - Register for event
- `GET /events/{event_id}/registrations` - List registrations (Admin)
- `DELETE /events/{event_id}/register` - Cancel registration

### 4.4 Attendance
- `POST /attendance/scan` - Record attendance via QR code
- `GET /events/{event_id}/attendance` - Get attendance report
- `POST /attendance/manual` - Manual attendance entry (Admin)

## 5. Database Schema

### 5.1 Users Table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    college_id INTEGER REFERENCES colleges(id),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 5.2 Events Table
```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    venue VARCHAR(255) NOT NULL,
    capacity INTEGER NOT NULL,
    college_id INTEGER REFERENCES colleges(id),
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 5.3 Registrations Table
```sql
CREATE TABLE registrations (
    id SERIAL PRIMARY KEY,
    event_id INTEGER REFERENCES events(id),
    user_id INTEGER REFERENCES users(id),
    registration_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(50) DEFAULT 'PENDING',
    UNIQUE(event_id, user_id)
);
```

### 5.4 Attendance Table
```sql
CREATE TABLE attendance (
    id SERIAL PRIMARY KEY,
    event_id INTEGER REFERENCES events(id),
    user_id INTEGER REFERENCES users(id),
    check_in_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    check_out_time TIMESTAMP,
    attendance_status VARCHAR(50) DEFAULT 'PRESENT',
    recorded_by INTEGER REFERENCES users(id)
);
```

## 6. Security Considerations

### 6.1 Authentication & Authorization
- JWT with short-lived access tokens and refresh tokens
- Role-based access control
- Secure password hashing (bcrypt)
- Rate limiting on authentication endpoints

### 6.2 Data Protection
- All sensitive data encrypted at rest
- HTTPS for all communications
- Input validation and sanitization
- Protection against SQL injection

### 6.3 API Security
- CORS configuration
- CSRF protection
- Request validation
- API versioning

## 7. Performance Considerations

### 7.1 Caching Strategy
- Redis for caching frequently accessed data
- Cache invalidation strategy
- Response compression

### 7.2 Database Optimization
- Proper indexing
- Query optimization
- Connection pooling

### 7.3 Frontend Performance
- Code splitting
- Lazy loading
- Asset optimization

## 8. Error Handling

### 8.1 Error Responses
- Standardized error format
- Appropriate HTTP status codes
- User-friendly error messages
- Detailed error logging

### 8.2 Monitoring & Logging
- Centralized logging
- Error tracking
- Performance monitoring
- Alerting system

## 9. Deployment

### 9.1 Development Environment
- Docker Compose for local development
- SQLite database
- Hot-reload for development

### 9.2 Production Environment
- Containerized deployment with Docker
- PostgreSQL database
- Load balancing
- Auto-scaling

## 10. Future Enhancements

### 10.1 Short-term
- Push notifications
- Calendar integration
- Bulk operations for admins

### 10.2 Long-term
- AI-powered event recommendations
- Advanced analytics dashboard
- Mobile app with offline support
- Integration with college systems

## 11. Conclusion
This design document provides a comprehensive overview of the Campus Event Manager system architecture, components, and technical specifications. The system is designed to be scalable, secure, and maintainable, with a focus on providing an excellent user experience for both students and administrators.
