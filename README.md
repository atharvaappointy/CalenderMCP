# MCP Productivity Suite

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Version](https://img.shields.io/badge/version-1.0.0-green.svg)

<div align="center">
  <img src="https://i.imgur.com/fXuGP7N.png" alt="MCP Productivity Suite Logo" width="120">
  <h3>Seamlessly manage meetings and communications with AI-powered tools</h3>
</div>

## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [User Flow](#user-flow)
- [Installation](#installation)
- [Configuration](#configuration)
- [API Documentation](#api-documentation)
- [Architecture](#architecture)
- [Authentication](#authentication)
- [Natural Language Processing](#natural-language-processing)
- [Tech Stack](#tech-stack)
- [Contributing](#contributing)
- [License](#license)

## 🚀 Overview

MCP Productivity Suite is a powerful web application that seamlessly integrates with Google and Microsoft services to help users manage their calendar events and emails efficiently. The application leverages AI-powered natural language processing to enable users to create meetings and send emails using plain language instructions.

## ✨ Features

- **Multi-Provider Integration**
  - Google Calendar & Gmail services
  - Microsoft Outlook Calendar & Email services

- **Calendar Management**
  - View upcoming calendar events
  - Schedule new meetings with online meeting links
  - Real-time timezone conversion
  - Automatic online meeting creation (Google Meet/Microsoft Teams)

- **Email Communication**
  - Send emails directly from the interface
  - Support for multiple recipients

- **AI-Powered Natural Language Processing**
  - Create calendar events using natural language
  - Compose emails using conversational instructions

- **Modern User Interface**
  - Responsive design for all devices
  - Dark/Light mode support
  - Real-time feedback and notifications
  - Interactive animations and transitions

## 👤 User Flow

1. **Authentication**
   - User arrives at the login screen
   - Selects either Google or Microsoft authentication
   - Authorizes the application to access their calendar and email
   - Returns to the application with proper authentication

2. **Dashboard Experience**
   - User views their upcoming calendar events
   - Can refresh calendar data at any time
   - Current timezone is displayed with ability to change

3. **Meeting Scheduling**
   - **Option 1:** Fill out the meeting form manually
     - Enter meeting title
     - Select start and end times
     - Add attendees (comma-separated emails)
     - Submit to create the meeting with online meeting link

   - **Option 2:** Use natural language
     - Enter a description like "Schedule a meeting with john@example.com tomorrow at 2 PM to 4 PM for Project Discussion"
     - AI processes the request and creates the meeting automatically

4. **Email Sending**
   - **Option 1:** Fill out the email form manually
     - Enter recipients
     - Enter subject
     - Compose email body
     - Send email

   - **Option 2:** Use natural language
     - Enter a description like "Send an email to sarah@example.com with subject 'Project Update' and tell her the project is on track"
     - AI processes the request and sends the email automatically

## 🔧 Installation

### Prerequisites
- Node.js (v14 or later)
- npm or yarn
- PostgreSQL database

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/mcp-server.git
   cd mcp-server
   ```

2. **Install dependencies**
   ```bash
   npm install
   # or
   yarn install
   ```

3. **Set up environment variables**
   Create a `.env` file in the root directory with the following variables:
   ```
   # Server Configuration
   PORT=3000
   HOST=localhost
   NODE_ENV=development

   # Database Configuration
   DB_HOST=localhost
   DB_PORT=5432
   DB_USER=postgres
   DB_PASSWORD=your_password
   DB_NAME=mcp_database

   # Google OAuth Configuration
   GOOGLE_CLIENT_ID=your_google_client_id
   GOOGLE_CLIENT_SECRET=your_google_client_secret
   GOOGLE_REDIRECT_URL=http://localhost:3000/auth/google/callback

   # Microsoft OAuth Configuration
   MICROSOFT_CLIENT_ID=your_microsoft_client_id
   MICROSOFT_CLIENT_SECRET=your_microsoft_client_secret
   MICROSOFT_REDIRECT_URL=http://localhost:3000/auth/microsoft/callback
   
   # Session Secret
   SESSION_SECRET=your_session_secret
   ```

4. **Initialize the database**
   ```bash
   npm run db:init
   # or
   yarn db:init
   ```

5. **Start the server**
   ```bash
   npm start
   # or
   yarn start
   ```

6. **Access the application**
   Open your browser and navigate to `http://localhost:3000`

## ⚙️ Configuration

### Database Schema

The application uses a PostgreSQL database with the following main table:

**users**
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255),
  picture TEXT,
  google_id VARCHAR(255),
  google_access_token TEXT,
  google_refresh_token TEXT,
  google_token_expiry TIMESTAMP,
  microsoft_id VARCHAR(255),
  microsoft_access_token TEXT,
  microsoft_refresh_token TEXT,
  microsoft_token_expiry TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### OAuth Configuration

#### Google OAuth
1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project
3. Navigate to "APIs & Services" > "Credentials"
4. Create an OAuth 2.0 Client ID
5. Add the following scopes:
   - `https://www.googleapis.com/auth/userinfo.email`
   - `https://www.googleapis.com/auth/userinfo.profile`
   - `https://www.googleapis.com/auth/calendar`
   - `https://www.googleapis.com/auth/gmail.send`
6. Add your redirect URL: `http://your-domain/auth/google/callback`

#### Microsoft OAuth
1. Go to the [Microsoft Azure Portal](https://portal.azure.com/)
2. Navigate to "App registrations"
3. Register a new application
4. Add the following Microsoft Graph API permissions:
   - `User.Read`
   - `Calendars.ReadWrite`
   - `Mail.Send`
5. Add your redirect URL: `http://your-domain/auth/microsoft/callback`

## 📡 API Documentation

### Authentication Endpoints

#### Google Authentication
- **GET** `/auth/google`
  - Initiates Google OAuth flow
  - No parameters required

- **GET** `/auth/google/callback`
  - Google OAuth callback endpoint
  - Query Parameters:
    - `code`: Authorization code from Google
  - Response: Redirects to dashboard with token

#### Microsoft Authentication
- **GET** `/auth/microsoft`
  - Initiates Microsoft OAuth flow
  - No parameters required

- **GET** `/auth/microsoft/callback`
  - Microsoft OAuth callback endpoint
  - Query Parameters:
    - `code`: Authorization code from Microsoft
  - Response: Redirects to dashboard with token

### Google Calendar Endpoints

- **GET** `/google/calendar/events`
  - Fetches user's Google Calendar events
  - Headers:
    - `Authorization`: Bearer token
  - Response: Array of calendar events

- **POST** `/google/calendar/events`
  - Creates a new Google Calendar event
  - Headers:
    - `Authorization`: Bearer token
  - Body:
    ```json
    {
      "summary": "Meeting Title",
      "start": {
        "dateTime": "2023-05-01T09:00:00",
        "timeZone": "America/New_York"
      },
      "end": {
        "dateTime": "2023-05-01T10:00:00",
        "timeZone": "America/New_York"
      },
      "attendees": [
        {"email": "attendee1@example.com"},
        {"email": "attendee2@example.com"}
      ]
    }
    ```
  - Response: Created event details with meeting link

- **POST** `/google/calendar/nl-create`
  - Creates a calendar event using natural language
  - Headers:
    - `Authorization`: Bearer token
  - Body:
    ```json
    {
      "text": "Schedule a meeting with john@example.com tomorrow at 2 PM for 1 hour",
      "timezone": "America/New_York"
    }
    ```
  - Response: Created event details with meeting link

### Microsoft Calendar Endpoints

- **GET** `/microsoft/calendar/events`
  - Fetches user's Microsoft Calendar events
  - Headers:
    - `Authorization`: Bearer token
  - Response: Array of calendar events

- **POST** `/microsoft/calendar/events`
  - Creates a new Microsoft Calendar event
  - Headers:
    - `Authorization`: Bearer token
  - Body:
    ```json
    {
      "summary": "Meeting Title",
      "start": {
        "dateTime": "2023-05-01T09:00:00",
        "timeZone": "America/New_York"
      },
      "end": {
        "dateTime": "2023-05-01T10:00:00",
        "timeZone": "America/New_York"
      },
      "attendees": [
        {"email": "attendee1@example.com"},
        {"email": "attendee2@example.com"}
      ]
    }
    ```
  - Response: Created event details with Teams meeting link

- **POST** `/microsoft/calendar/nl-create`
  - Creates a calendar event using natural language
  - Headers:
    - `Authorization`: Bearer token
  - Body:
    ```json
    {
      "text": "Schedule a meeting with john@example.com tomorrow at 2 PM for 1 hour",
      "timezone": "Europe/London"
    }
    ```
  - Response: Created event details with Teams meeting link

### Email Endpoints

- **POST** `/google/gmail/send`
  - Sends an email using Gmail
  - Headers:
    - `Authorization`: Bearer token
  - Body:
    ```json
    {
      "to": "recipient@example.com",
      "subject": "Email Subject",
      "body": "Email body content"
    }
    ```
  - Response: Success confirmation

- **POST** `/microsoft/outlook/send`
  - Sends an email using Microsoft Outlook
  - Headers:
    - `Authorization`: Bearer token
  - Body:
    ```json
    {
      "to": "recipient@example.com",
      "subject": "Email Subject",
      "body": "Email body content"
    }
    ```
  - Response: Success confirmation

## 🏗️ Architecture

The MCP Productivity Suite follows a modular architecture pattern:

```
mcp-server/
├── src/
│   ├── client/                 # Frontend code
│   │   ├── index.html          # Main HTML template
│   │   ├── styles.css          # CSS styles
│   │   └── script.js           # Client-side JavaScript
│   ├── core/                   # Core server functionality
│   │   ├── auth.js             # Authentication module
│   │   ├── database.js         # Database connection
│   │   ├── routes.js           # Route definitions
│   │   └── server.js           # Server initialization
│   ├── llm/                    # Language model service
│   │   └── service.js          # NLP processing service
│   ├── middleware/             # Custom middleware
│   │   └── auth.js             # Authentication middleware
│   ├── plugins/                # Service plugins
│   │   ├── google-services/    # Google API integration
│   │   │   ├── api.js          # Google API client
│   │   │   ├── index.js        # Plugin registration
│   │   │   └── tools.js        # Google API helpers
│   │   └── microsoft-services/ # Microsoft API integration
│   │       ├── api.js          # Microsoft Graph API client
│   │       ├── index.js        # Plugin registration
│   │       └── tools.js        # Microsoft API helpers
│   ├── utils/                  # Utility functions
│   │   └── logger.js           # Logging utility
│   └── index.js                # Application entry point
├── .env                        # Environment variables
├── package.json                # Dependencies and scripts
└── README.md                   # Project documentation
```

### Key Components

- **Server**: Built with [Hapi.js](https://hapi.dev/) for robust routing and plugin architecture
- **Authentication**: OAuth 2.0 flow for both Google and Microsoft
- **Database**: PostgreSQL for user data and token storage
- **API Clients**: Specialized modules for Google API and Microsoft Graph API
- **NLP Service**: Natural language processing for interpreting user requests
- **Frontend**: Vanilla JavaScript with modern CSS for a responsive UI

## 🔐 Authentication

The application implements OAuth 2.0 authentication flows for both Google and Microsoft:

1. **Authentication Flow**:
   - User initiates login
   - Redirected to provider's consent screen
   - Provider redirects back with authorization code
   - Server exchanges code for access and refresh tokens
   - Tokens stored in database associated with user
   - JWT created for client-side authentication

2. **Token Management**:
   - Access tokens typically expire after 1 hour
   - Application refreshes tokens automatically when needed
   - Refresh tokens stored securely in database
   - Session expiry information provided to client

## 🧠 Natural Language Processing

The application uses a sophisticated natural language processing service to interpret user requests:

1. **Meeting Creation Processing**:
   - Extracts meeting title/purpose
   - Identifies attendees
   - Parses date and time information
   - Determines meeting duration
   - Handles timezone specifications
   - Formats data for the appropriate provider's API

2. **Email Composition Processing**:
   - Identifies recipients
   - Extracts email subject
   - Interprets email body content
   - Formats data for the appropriate provider's API

## 💻 Tech Stack

- **Backend**:
  - Node.js
  - Hapi.js (Server framework)
  - PostgreSQL (Database)
  - JSON Web Tokens (Authentication)

- **Frontend**:
  - Vanilla JavaScript
  - Modern CSS with Flexbox/Grid
  - Font Awesome (Icons)

- **External APIs**:
  - Google Calendar API
  - Google Gmail API
  - Microsoft Graph API

- **Libraries**:
  - Luxon (Date/time handling)
  - Various OAuth libraries

## 🤝 Contributing

We welcome contributions to the MCP Productivity Suite! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">
  <p>Built with ❤️ by the MCP Team</p>
</div> 
