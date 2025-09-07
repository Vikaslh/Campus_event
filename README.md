# Campus Event Manager


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
