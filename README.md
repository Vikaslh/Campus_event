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


### Events
- `GET /api/events` - List all events
- `GET /api/events/{event_id}` - Get event details
- `POST /api/events` - Create new event (Admin only)
- `PUT /api/events/{event_id}` - Update event (Admin only)
- `DELETE /api/events/{event_id}` - Delete event (Admin only)

### Registrations
- `POST /api/events/{event_id}/register` - Register for an event
- `GET /api/users/me/registrations` - Get user's event registrations
- `DELETE /api/registrations/{registration_id}` - Cancel registration

## Project Flow

### User Registration Flow
1. User submits registration form with email and password
2. Backend validates input and creates new user
3. Verification email sent to user's email
4. User clicks verification link to activate account

### Login Flow
1. User submits email and password
2. Backend verifies credentials
3. JWT token issued upon successful authentication
4. Token stored in client-side storage (secure HTTP-only cookie)

### Event Management Flow
1. Admin creates event with details (title, description, date, location, capacity)
2. Event is saved to database
3. Users can browse and register for events
4. System enforces registration limits and deadlines
5. Users receive confirmation and event reminders

### Registration Flow
1. Authenticated user selects an event
2. System checks event availability
3. If available, registration is created
4. Confirmation email sent to user
5. User can view/manage registrations in their dashboard

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
