# Audio Transcription App - Complete System Architecture

## System Overview

### Tech Stack
- **Frontend**: iOS Native App (Swift/SwiftUI)
- **Backend**: FastAPI (Python)
- **Database**: PostgreSQL
- **Authentication**: Firebase Auth (Apple & Google Login)
- **AI Services**: OpenAI Whisper API + GPT-4 API
- **Deployment**: Digital Ocean Droplet ($6/month)

## Complete System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          iOS APP                                │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐    ┌──────────────┐ │
│  │   Login Screen  │    │  Record Audio   │    │   Results    │ │
│  │                 │    │                 │    │   Screen     │ │
│  │  Apple/Google   │    │  - Record       │    │              │ │
│  │     Auth        │    │  - Upload       │    │ - Text       │ │
│  │                 │    │  - Process      │    │ - Summary    │ │
│  └─────────────────┘    └─────────────────┘    └──────────────┘ │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              Local Storage (Core Data/SQLite)              │ │
│  │  - Audio files saved locally                               │ │
│  │  - Transcription results cached                            │ │
│  │  - User preferences                                        │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                    │
                              HTTPS/REST API
                                    │
┌─────────────────────────────────────────────────────────────────┐
│                       BACKEND API                               │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                 FastAPI Server                              │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │ │
│  │  │    Auth     │  │   Audio     │  │    Transcription    │  │ │
│  │  │  Endpoints  │  │  Endpoints  │  │     Endpoints       │  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │               PostgreSQL Database                           │ │
│  │  - Users table                                              │ │
│  │  - Audio_files table                                        │ │
│  │  - Transcriptions table                                     │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                    │
                          External API Calls
                                    │
┌─────────────────────────────────────────────────────────────────┐
│                    External Services                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐ │
│  │   Firebase    │  │   OpenAI      │  │      OpenAI           │ │
│  │     Auth      │  │   Whisper     │  │      GPT-4            │ │
│  │               │  │     API       │  │       API             │ │
│  │ - Apple Login │  │ - Speech to   │  │ - Text               │ │
│  │ - Google Login│  │   Text        │  │   Summarization      │ │
│  └───────────────┘  └───────────────┘  └───────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Data Storage Strategy

### Frontend Storage (iOS App Responsibility)
The iOS app handles all file storage locally:

**What gets saved on device:**
- **Audio Files**: Original recordings stored in app's document directory
- **Transcription Results**: Cached locally for offline access
- **User Preferences**: Settings and app configuration
- **Authentication Tokens**: Firebase tokens for session management

**iOS Storage Implementation:**
- **Core Data** or **SQLite**: For structured data (transcriptions, metadata)
- **File Manager**: For audio files in document directory
- **UserDefaults**: For user preferences
- **Keychain**: For secure token storage

### Backend Storage
Backend only stores metadata and results:
- **User profiles** (no personal files)
- **Transcription text and summaries**
- **Processing status and metadata**
- **Usage analytics** (optional)

## API Endpoints Documentation

### 1. Authentication Endpoints

#### POST /auth/login
**Purpose**: Authenticate user with Firebase token

**Request:**
```json
{
  "firebase_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "provider": "apple"
}
```

**Response (Success - 200):**
```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "display_name": "John Doe",
  "access_token": "backend_jwt_token",
  "expires_in": 3600
}
```

**Response (Error - 401):**
```json
{
  "error": "Invalid firebase token",
  "message": "Authentication failed"
}
```

### 2. Audio Processing Endpoints

#### POST /audio/transcribe
**Purpose**: Upload audio file and start transcription process

**Request:**
- **Content-Type**: multipart/form-data
- **Headers**: Authorization: Bearer {access_token}
- **Body**: 
  - file: (audio file - max 25MB)
  - filename: string

**Response (Success - 202):**
```json
{
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "transcribing",
  "estimated_time": "30-90 seconds",
  "message": "Audio uploaded successfully, transcription started"
}
```

**Response (Error - 400):**
```json
{
  "error": "File too large",
  "message": "Maximum file size is 25MB",
  "max_size_mb": 25
}
```

#### GET /audio/transcription/{audio_id}
**Purpose**: Get transcription results (text only)

**Request:**
- **Headers**: Authorization: Bearer {access_token}
- **Path Parameter**: audio_id

**Response (Processing - 202):**
```json
{
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "transcribing",
  "progress": 45,
  "created_at": "2025-06-25T10:30:00Z"
}
```

**Response (Completed - 200):**
```json
{
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "transcribed",
  "transcription": "Hello, this is a test recording...",
  "word_count": 156,
  "processing_time_seconds": 45,
  "created_at": "2025-06-25T10:30:00Z",
  "completed_at": "2025-06-25T10:30:45Z"
}
```

**Response (Failed - 422):**
```json
{
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "failed",
  "error": "transcription_failed",
  "message": "Could not process audio file",
  "created_at": "2025-06-25T10:30:00Z"
}
```

#### POST /audio/summarize
**Purpose**: Generate summary from existing transcription

**Request:**
```json
{
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "summary_type": "brief"
}
```

**Response (Success - 202):**
```json
{
  "summary_id": "550e8400-e29b-41d4-a716-446655440002",
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "summarizing",
  "estimated_time": "10-30 seconds",
  "summary_type": "brief"
}
```

#### GET /audio/summary/{summary_id}
**Purpose**: Get summary results

**Request:**
- **Headers**: Authorization: Bearer {access_token}
- **Path Parameter**: summary_id

**Response (Processing - 202):**
```json
{
  "summary_id": "550e8400-e29b-41d4-a716-446655440002",
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "summarizing",
  "progress": 70,
  "created_at": "2025-06-25T10:31:00Z"
}
```

**Response (Completed - 200):**
```json
{
  "summary_id": "550e8400-e29b-41d4-a716-446655440002",
  "audio_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "completed",
  "summary": "The speaker discusses the evolution of audio transcription technology...",
  "summary_type": "brief",
  "processing_time_seconds": 15,
  "created_at": "2025-06-25T10:31:00Z",
  "completed_at": "2025-06-25T10:31:15Z"
}
```

#### GET /audio/history
**Purpose**: Get user's transcription history

**Request:**
- **Headers**: Authorization: Bearer {access_token}
- **Query Parameters**: 
  - limit: integer (default: 20)
  - offset: integer (default: 0)

**Response (200):**
```json
{
  "transcriptions": [
    {
      "audio_id": "550e8400-e29b-41d4-a716-446655440001",
      "filename": "meeting_recording.m4a",
      "status": "completed",
      "summary": "Meeting discussion about project timeline...",
      "created_at": "2025-06-25T10:30:00Z"
    },
    {
      "audio_id": "550e8400-e29b-41d4-a716-446655440002",
      "filename": "interview.wav",
      "status": "processing",
      "created_at": "2025-06-25T09:15:00Z"
    }
  ],
  "total": 15,
  "limit": 20,
  "offset": 0
}
```

### 3. User Management Endpoints

#### GET /user/profile
**Purpose**: Get user profile information

**Response (200):**
```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "display_name": "John Doe",
  "provider": "apple",
  "transcriptions_count": 25,
  "created_at": "2025-05-01T12:00:00Z"
}
```

#### DELETE /user/transcription/{audio_id}
**Purpose**: Delete specific transcription

**Response (204):** No content

## Application Workflow

### User Journey Flow

```
1. App Launch
   ↓
2. Firebase Authentication (Apple/Google)
   ↓
3. Main Dashboard
   ↓
4. Record Audio Button
   ↓
5. iOS Native Audio Recording
   ↓
6. Save Audio Locally (iOS handles this)
   ↓
7. Upload to Backend (/audio/transcribe)
   ↓
8. Show Transcription Status
   ↓
9. Poll for Transcription (/audio/transcription/{id})
   ↓
10. Display Transcription Text
    ↓
11. User Chooses to Summarize
    ↓
12. Request Summary (/audio/summarize)
    ↓
13. Poll for Summary (/audio/summary/{id})
    ↓
14. Display Summary + Save to Local Cache
    ↓
15. User can view/edit/share results
```

### Backend Processing Flow

```
Audio Upload Request
        ↓
    Validate File
        ↓
    Create Database Record
        ↓
    Send to Whisper API
        ↓
    Get Transcription
        ↓
    Save to Database
        ↓
    Status: Transcribed
        
        (Separate Flow)
        
Summary Request
        ↓
    Get Transcription from DB
        ↓
    Send to GPT-4 API
        ↓
    Get Summary
        ↓
    Save to Database
        ↓
    Status: Summarized
```

## Database Schema

### Users Table
```
users
├── id (UUID, Primary Key)
├── firebase_uid (String, Unique)
├── email (String)
├── display_name (String)
├── provider (String) // 'apple' or 'google'
├── created_at (Timestamp)
└── updated_at (Timestamp)
```

### Audio Files Table
```
audio_files
├── id (UUID, Primary Key)
├── user_id (UUID, Foreign Key)
├── filename (String)
├── file_size (Integer)
├── duration_seconds (Integer)
├── mime_type (String)
├── status (String) // 'processing', 'completed', 'failed'
├── created_at (Timestamp)
└── updated_at (Timestamp)
```

### Transcriptions Table
```
transcriptions
├── id (UUID, Primary Key)
├── audio_file_id (UUID, Foreign Key)
├── user_id (UUID, Foreign Key)
├── transcription_text (Text)
├── word_count (Integer)
├── processing_time_seconds (Integer)
├── whisper_response_data (JSON)
├── processing_status (String) // 'transcribing', 'transcribed', 'failed'
├── created_at (Timestamp)
└── updated_at (Timestamp)
```

### Summaries Table
```
summaries
├── id (UUID, Primary Key)
├── transcription_id (UUID, Foreign Key)
├── audio_file_id (UUID, Foreign Key)
├── user_id (UUID, Foreign Key)
├── summary_text (Text)
├── summary_type (String) // 'brief', 'detailed', 'bullet_points'
├── processing_time_seconds (Integer)
├── gpt_response_data (JSON)
├── processing_status (String) // 'summarizing', 'completed', 'failed'
├── created_at (Timestamp)
└── updated_at (Timestamp)
```

## Deployment Architecture

### Digital Ocean Setup
```
Digital Ocean Droplet ($6/month)
├── Ubuntu 22.04 LTS
├── Docker & Docker Compose
├── Nginx (Reverse Proxy)
├── SSL Certificate (Let's Encrypt)
└── PostgreSQL Database
```

### Server Configuration
- **CPU**: 1 vCPU
- **RAM**: 1 GB
- **Storage**: 25 GB SSD
- **Bandwidth**: 1 TB transfer

## Security Considerations

### Authentication Flow
1. User authenticates with Apple/Google on iOS
2. Firebase generates ID token
3. iOS app sends token to backend
4. Backend verifies token with Firebase Admin SDK
5. Backend generates JWT for API access

### API Security
- All endpoints require authentication
- Rate limiting implemented
- File upload size restrictions
- Input validation and sanitization

This architecture provides a scalable, cost-effective solution for your audio transcription MVP with clear separation of concerns between frontend and backend responsibilities.
