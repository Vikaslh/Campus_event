# Campus Event Manager
Campus Event Manager is a user-friendly platform designed to simplify the process of organizing and managing college events. Administrators can quickly create, update, and track events through a straightforward web interface. Students can discover and register for events using either the website or the mobile app, making participation convenient. On event day, checking in is simple with QR codes, ensuring fast and accurate attendance. The platform is built with flexibility in mind, using React.js for the web frontend, React Native for mobile apps, and FastAPI for the backend, with data stored securely in SQLite or PostgreSQL. Role-based authentication with JWT ensures secure information storage and tailors access for both students and administrators.



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
