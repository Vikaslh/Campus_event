# Campus Event Manager
Campus Event Manager is a full-stack platform designed to streamline the organization and management of college events. Administrators can effortlessly create, manage, and monitor events via a user-friendly web interface, while students can browse and register for upcoming activities through web or mobile apps. The system features QR code-based attendance tracking for accurate and efficient check-ins on event day. Built with a modular architecture, the platform leverages React.js for the web frontend, React Native for mobile applications, and FastAPI for the backend API, supported by a SQLite/PostgreSQL database. It also implements role-based authentication with JWT, ensuring secure access and a personalized experience for both administrators and students.


## Features
- Event creation and management
- User registration and authentication
- Event registration and ticketing
- Attendee management
- Event calendar and scheduling

## Project Structure
```
|
├── backend/  
       |--Database         # FastAPI backend
├──  src/         # React frontend
├── mobile/            # React Native mobile app
└── src            # Project documentation
```

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


## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
