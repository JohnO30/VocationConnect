# VocationConnect Architecture Diagram

## System Overview

VocationConnect is an alumni connection & mock interview platform built with Node.js, Express, MySQL, and EJS templating.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          VocationConnect Platform                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│  │   Students   │    │   Alumni     │    │   Admin      │                  │
│  │              │    │              │    │              │                  │
│  │ - Browse     │    │ - Profile    │    │ - Monitor    │                  │
│  │ - Connect    │    │ - Interviews │    │ - Audit      │                  │
│  │ - Interview  │    │ - Mentor     │    │              │                  │
│  └──────┬───────┘    └──────┬───────┘    └──────────────┘                  │
│         │                    │                                             │
│         └────────────────────┼─────────────────────────────────────────────┤
│                              │                                              │
│                    ┌─────────▼─────────┐                                    │
│                    │   Web Browser     │                                    │
│                    │   (Client-side)   │                                    │
│                    │                   │                                    │
│                    │ - EJS Templates   │                                    │
│                    │ - CSS/JS          │                                    │
│                    │ - WebRTC Client   │                                    │
│                    └─────────┬─────────┘                                    │
│                              │                                              │
│                              │ HTTP/WebSocket                               │
│                              │                                              │
└──────────────────────────────┼──────────────────────────────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Express Server    │
                    │   (index.js)        │
                    │                     │
                    │ - Session Mgmt      │
                    │ - Route Handlers    │
                    │ - Socket.io Server  │
                    │ - Global Services   │
                    └──────────┬──────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
┌───────▼────────┐   ┌────────▼────────┐   ┌────────▼────────┐
│   Routes       │   │   Services      │   │   Database      │
│                │   │                 │   │                 │
│ • main.js      │   │ • Notification  │   │ • MySQL         │
│ • users.js     │   │   Service       │   │                 │
│ • alumni.js    │   │ • SearchEngine  │   │ Tables:         │
│ • interviews.js│   │ • SurveyService │   │ • users         │
│ • connections  │   │                 │   │ • alumni_profiles│
│ • messages.js  │   │                 │   │ • connections   │
│ • notifications│   │                 │   │ • direct_messages│
│ • survey.js    │   │                 │   │ • mock_interviews│
└────────────────┘   └─────────────────┘   │ • interview_questions│
                                            │ • chat_messages  │
                                            │ • interview_notes│
                                            │ • notifications  │
                                            │ • user_notification_settings│
                                            │ • login_audit    │
                                            └─────────────────┘
```

## Application Architecture Layers

### 1. Presentation Layer (Client)
- **EJS Templates** (26 views in `views/`)
  - Layout system with `layout.ejs`
  - Dynamic content rendering
  - Session-aware navigation
  
- **Static Assets** (`public/`)
  - CSS styling (`main.css`, `interview-room.css`)
  - Images
  - Client-side JavaScript

- **WebRTC Client**
  - Video/audio streaming
  - Real-time interview room
  - Socket.io client integration

### 2. Application Layer (Server)

#### Entry Point: `index.js`
```javascript
Express App (port 8000)
├── Middleware Stack
│   ├── Body parsing (JSON, URL-encoded)
│   ├── Session management (express-session)
│   ├── Input sanitization (express-sanitizer)
│   ├── Static file serving
│   └── Template engine (EJS + layouts)
│
├── Global Services
│   ├── MySQL connection pool (global.db)
│   └── NotificationService (global.notificationService)
│
├── Route Handlers (8 modules)
│   ├── / → main.js
│   ├── /users → users.js
│   ├── /alumni → alumni.js
│   ├── /interviews → interviews.js
│   ├── /connections → connections.js
│   ├── /messages → messages.js
│   ├── /notifications → notifications.js
│   └── /survey → survey.js
│
└── Real-time Communication
    └── Socket.io Server (WebRTC signaling)
        ├── joinInterview
        ├── webRTCOffer
        ├── webRTCAnswer
        ├── iceCandidate
        └── disconnect handling
```

#### Route Handlers Breakdown

**main.js** - Core pages
- GET `/` - Home page
- GET `/about` - About page
- GET `/dashboard` - User dashboard (auth required)
- GET/POST `/directmessages` - Direct messaging

**users.js** - Authentication & profiles
- GET/POST `/register` - User registration
- GET/POST `/login` - User login
- GET/POST `/logout` - User logout
- GET `/profile` - View profile
- POST `/profile/update` - Update profile
- Middleware: `redirectLogin`, `redirectHome`

**alumni.js** - Alumni directory & search
- GET `/browse` - Browse all alumni
- GET/POST `/search` - Advanced search with algorithmic ranking
- GET `/api/search-suggestions` - Autocomplete API
- GET `/api/fuzzy-search` - Fuzzy search API
- GET `/api/industries` - Industry filter options
- GET `/:id` - Individual alumni profile
- Uses: `searchEngine` utility

**interviews.js** - Mock interview system
- GET `/my` - View my interviews
- GET `/request/:alumniId` - Request interview form
- POST `/request` - Submit interview request
- GET `/room/:id` - Live interview room (WebRTC)
- POST `/notes/:id` - Save interview notes
- POST `/chat/:id` - Send chat message
- GET `/chat/:id` - Get chat history
- POST `/:id/accept` - Accept interview request
- POST `/:id/decline` - Decline interview request
- POST `/:id/cancel` - Cancel interview request
- POST `/:id/complete` - Complete interview

**connections.js** - Connection management
- GET `/my` - View my connections
- POST `/request` - Send connection request
- POST `/:id/accept` - Accept connection
- POST `/:id/decline` - Decline connection
- POST `/:id/cancel` - Cancel connection request

**messages.js** - Direct messaging
- GET `/my` - View message conversations
- GET `/:id` - View conversation thread
- POST `/:id` - Send message

**notifications.js** - Notification management
- GET `/` - View notifications
- POST `/:id/read` - Mark as read
- GET `/settings` - Notification preferences
- POST `/settings` - Update preferences

**survey.js** - Survey system
- GET `/` - Survey list
- GET `/:id` - Take survey
- POST `/:id` - Submit survey
- GET `/history` - Survey history
- GET `/report/:id` - Survey report

### 3. Service Layer

#### NotificationService (`notificationService.js`)
```
NotificationService
├── Email Notifications (nodemailer)
│   ├── Connection requests
│   ├── Connection responses
│   ├── Interview requests
│   └── Interview responses
│
├── Push Notifications (web-push)
│   └── Browser notifications (placeholder)
│
└── Database Notifications
    ├── Create notification
    ├── Get unread notifications
    └── Mark as read
```

#### SearchEngine (`utils/searchEngine.js`)
```
SearchEngine
├── Algorithmic Search
│   ├── Keyword matching (multi-field)
│   ├── Fuzzy matching (typo tolerance)
│   ├── Relevance scoring
│   └── Filter application
│
├── Autocomplete
│   ├── Real-time suggestions
│   └── Fuzzy autocomplete
│
└── Analytics
    └── Search statistics
```

#### SurveyService (`utils/surveyService`)
```
SurveyService
├── Survey Management
│   ├── Create survey
│   ├── Get survey questions
│   └── Submit responses
│
└── Reporting
    ├── Generate reports
    └── Analytics
```

### 4. Data Layer (MySQL Database)

#### Database Schema
```
vocationconnect Database
│
├── users (Core user accounts)
│   ├── id, username, email, hashedPassword
│   ├── first_name, last_name
│   ├── user_type (student/alumni)
│   ├── graduation_year
│   └── created_at
│
├── alumni_profiles (Extended alumni info)
│   ├── id, user_id
│   ├── company, job_title, industry
│   ├── years_experience, skills, bio
│   └── available_for_mock
│
├── connections (Student-alumni relationships)
│   ├── id, student_id, alumni_id
│   ├── status (pending/accepted/declined)
│   ├── message
│   └── created_at, updated_at
│
├── direct_messages (Messaging for connections)
│   ├── id, connection_id, sender_id
│   ├── message_text
│   └── sent_at
│
├── mock_interviews (Interview scheduling)
│   ├── id, student_id, alumni_id
│   ├── scheduled_date, duration_minutes
│   ├── interview_type, status
│   ├── notes, feedback, rating
│   └── created_at
│
├── interview_questions (Questions per interview)
│   ├── id, interview_id
│   ├── question_text, time_allocated
│   └── answered_at
│
├── chat_messages (Real-time interview chat)
│   ├── id, interview_id, sender_id
│   ├── message_text
│   └── sent_at
│
├── interview_notes (Private interview notes)
│   ├── id, interview_id, user_id
│   ├── note_text
│   └── created_at, updated_at
│
├── notifications (Notification log)
│   ├── id, user_id, type
│   ├── title, message, related_id
│   ├── is_read, email_sent, push_sent
│   └── created_at
│
├── user_notification_settings (User preferences)
│   ├── id, user_id
│   ├── email_notifications, push_notifications
│   ├── connection_requests, messages, interviews
│   └── created_at, updated_at
│
└── login_audit (Security audit)
    ├── id, username, login_time
    ├── status, message
    └── login_time
```

## Key Data Flows

### 1. User Authentication Flow
```
User → Register Form
    ↓
POST /users/registered
    ↓
Validation (express-validator)
    ↓
Password Hashing (bcrypt)
    ↓
Insert into users table
    ↓
If alumni: Create alumni_profiles entry
    ↓
Render register_success
```

```
User → Login Form
    ↓
POST /users/login
    ↓
Query users table
    ↓
bcrypt.compare() password
    ↓
If match: Create session (req.session)
    ↓
Log to login_audit
    ↓
Render login_success → Redirect to dashboard
```

### 2. Alumni Search Flow
```
User → Alumni Search Form
    ↓
POST /alumni/search
    ↓
Fetch all alumni from database
    ↓
Apply searchEngine.performSearch()
    ├── Keyword matching (name, company, skills, bio)
    ├── Filter application (industry, experience, grad year)
    ├── Relevance scoring algorithm
    └── Secondary sorting (experience/name/company)
    ↓
Render alumni_search_results with ranked results
```

### 3. Connection Request Flow
```
Student → Alumni Profile
    ↓
POST /connections/request
    ↓
Validate: Student can only send to alumni
    ↓
Check if connection already exists
    ↓
Insert into connections table (status: pending)
    ↓
Trigger notificationService.notifyConnectionRequest()
    ├── Create notification in database
    ├── Check user preferences
    └── Send email if enabled
    ↓
Redirect to alumni profile with flash message
```

### 4. Mock Interview Flow
```
Student → Request Interview
    ↓
POST /interviews/request
    ↓
Verify: Alumni available_for_mock
    ↓
Insert into mock_interviews (status: pending)
    ↓
Seed default interview_questions
    ↓
Trigger notificationService.notifyInterviewRequest()
    ↓
Redirect to /interviews/my
```

```
Alumni → Accept Interview
    ↓
POST /interviews/:id/accept
    ↓
Update mock_interviews (status: scheduled)
    ↓
Trigger notificationService.notifyInterviewResponse()
    ↓
Redirect to /interviews/my
```

```
Both Users → Join Interview Room
    ↓
GET /interviews/room/:id
    ↓
Verify: User is participant
    ↓
Render interview_room.ejs (layout: false)
    ↓
Socket.io Connection
    ├── joinInterview event
    ├── WebRTC offer/answer exchange
    ├── ICE candidate exchange
    └── Chat messaging
```

### 5. Notification Flow
```
Event (connection/interview)
    ↓
Route handler calls notificationService
    ↓
NotificationService Method
    ├── Fetch user details
    ├── Create notification in database
    ├── Get user notification preferences
    └── If email enabled: sendEmailNotification()
        ├── Get user email
        ├── Compose HTML email
        ├── Send via nodemailer (SMTP)
        └── Mark email_sent in database
    ↓
Return notification ID
```

## Security Features

1. **Authentication**
   - bcrypt password hashing (10 salt rounds)
   - Session-based authentication
   - Login audit trail

2. **Authorization**
   - Route-level middleware (`redirectLogin`)
   - Role-based access control (student vs alumni)
   - Resource ownership verification

3. **Input Validation**
   - express-validator for form validation
   - express-sanitizer for XSS prevention
   - Parameterized SQL queries (SQL injection prevention)

4. **Session Security**
   - Server-side sessions
   - 1-hour expiration
   - httpOnly cookies

## Real-time Features

### WebRTC Video Interview (Socket.io)
```
Client A (Student)          Server          Client B (Alumni)
     │                        │                   │
     ├── joinInterview ──────>│                   │
     │<── joinedInterview ────│                   │
     │                        │                   │
     │                        │<── joinInterview──│
     │                        │── joinedInterview>│
     │                        │                   │
     │<── createOffer ────────│                   │
     │                        │                   │
     │── webRTCOffer ────────>│── webRTCOffer ──>│
     │                        │                   │
     │                        │<── webRTCAnswer ──│
     │<── webRTCAnswer ───────│                   │
     │                        │                   │
     │── iceCandidate ───────>│── iceCandidate ──>│
     │<── iceCandidate ───────│<── iceCandidate ──│
     │                        │                   │
     │   P2P Connection Established              │
```

## File Structure
```
vocationconnect/
├── index.js                    # Main application entry point
├── package.json                # Dependencies
├── .env                        # Environment variables
├── init.sql                    # Database schema
├── seed.sql                    # Sample data
├── notificationService.js      # Notification service
├── routes/                     # Route handlers
│   ├── main.js                # Home, about, dashboard
│   ├── users.js               # Authentication, profiles
│   ├── alumni.js              # Alumni browsing, search
│   ├── interviews.js          # Mock interviews
│   ├── connections.js         # Connection requests
│   ├── messages.js            # Direct messaging
│   ├── notifications.js       # Notification management
│   └── survey.js              # Survey system
├── views/                      # EJS templates (26 files)
│   ├── layout.ejs             # Main layout
│   ├── index.ejs              # Home page
│   ├── dashboard.ejs          # User dashboard
│   ├── register.ejs           # Registration form
│   ├── login.ejs              # Login form
│   ├── profile.ejs            # User profile
│   ├── alumni_browse.ejs      # Alumni directory
│   ├── alumni_search.ejs      # Search form
│   ├── alumni_search_results.ejs  # Search results
│   ├── alumni_profile.ejs     # Individual profile
│   ├── connections_my.ejs     # My connections
│   ├── interviews_my.ejs      # My interviews
│   ├── interview_schedule.ejs # Schedule interview
│   ├── interview_room.ejs     # Live interview room
│   ├── messages_my.ejs        # Message list
│   ├── message_conversation.ejs  # Conversation view
│   ├── notifications.ejs      # Notification list
│   ├── notification_settings.ejs # Preferences
│   ├── survey_form.ejs        # Survey form
│   ├── survey_history.ejs     # Survey history
│   └── survey_report.ejs      # Survey report
├── public/                     # Static assets
│   ├── main.css               # Main stylesheet
│   ├── interview-room.css     # Interview room styles
│   └── images/                # Images
├── utils/                      # Utility modules
│   ├── searchEngine.js        # Advanced search algorithm
│   ├── searchOptimization.js  # Search optimization
│   └── surveyService.js       # Survey service
└── tests/                      # Test files
```

## Technology Stack

### Backend
- **Node.js** - JavaScript runtime
- **Express.js** - Web framework
- **MySQL2** - Database driver
- **Socket.io** - Real-time communication
- **bcrypt** - Password hashing
- **express-session** - Session management
- **express-validator** - Form validation
- **express-sanitizer** - Input sanitization
- **nodemailer** - Email notifications
- **web-push** - Push notifications
- **dotenv** - Environment configuration

### Frontend
- **EJS** - Templating engine
- **express-ejs-layouts** - Layout support
- **Custom CSS** - Responsive styling
- **Vanilla JavaScript** - Client-side logic
- **WebRTC** - Video/audio streaming

### Database
- **MySQL 8.0+** - Relational database
- **Connection pooling** - Performance optimization
- **Foreign keys** - Data integrity
- **Indexes** - Query optimization

## Deployment Architecture

```
Development (Local)
├── Node.js server on port 8000
├── MySQL on localhost
└── Access: http://localhost:8000

Production (VM)
├── Node.js server on port 8000
├── MySQL on localhost
├── VOCATION_BASE_PATH=/usr/260
└── Access: https://www.doc.gold.ac.uk/usr/260/
```

## Key Features Summary

1. **User Management**
   - Separate student/alumni registration
   - Secure authentication
   - Profile management

2. **Alumni Directory**
   - Browse all alumni
   - Advanced algorithmic search
   - Fuzzy matching & autocomplete
   - Detailed profiles

3. **Connection System**
   - Send connection requests
   - Accept/decline requests
   - Track connection status
   - Direct messaging

4. **Mock Interviews**
   - Request interviews
   - Schedule with alumni
   - Live video room (WebRTC)
   - Real-time chat
   - Interview notes
   - Feedback & ratings

5. **Notifications**
   - In-app notifications
   - Email notifications
   - User preferences
   - Notification history

6. **Survey System**
   - Create surveys
   - Take surveys
   - View history
   - Generate reports

7. **Security**
   - Password hashing
   - Session management
   - Input validation
   - SQL injection prevention
   - Login audit trail
