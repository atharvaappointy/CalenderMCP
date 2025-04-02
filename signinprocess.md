# Sign-In Process for MCP-Server

## Overview
The `mcp-server` application provides a productivity hub with calendar and email functionalities, supporting sign-in via Google and Microsoft accounts. This document details the authentication process, focusing on how users sign in using OAuth 2.0 with Google and Microsoft services, up to the point of successful authentication and session creation.

---

## Environment Requirements

The application relies on environment variables stored in a `.env` file for configuration. These variables are loaded using the `dotenv` package and are critical for secure authentication and API communication.

### `.env` Variables
| Variable                  | Description                                                                 | Why Needed                                                                                 |
|---------------------------|-----------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|
| `SERVER_PORT`             | Port number for the server (default: 3000)                                  | Defines where the server listens for requests.                                             |
| `GOOGLE_CLIENT_ID`        | Client ID from Google Cloud Console                                         | Identifies the app to Google's OAuth service for authentication.                           |
| `GOOGLE_CLIENT_SECRET`    | Client secret from Google Cloud Console                                     | Secures the OAuth token exchange with Google.                                              |
| `GOOGLE_REDIRECT_URI`     | Redirect URI (e.g., `http://localhost:3000/auth/google/callback`)          | Specifies where Google redirects users post-authentication; must match Google Console.     |
| `MICROSOFT_CLIENT_ID`     | Client ID from Azure Portal                                                 | Identifies the app to Microsoft's OAuth service for authentication.                        |
| `MICROSOFT_CLIENT_SECRET` | Client secret from Azure Portal                                             | Secures the OAuth token exchange with Microsoft.                                           |
| `MICROSOFT_REDIRECT_URI`  | Redirect URI (e.g., `http://localhost:3000/auth/microsoft/callback`)       | Specifies where Microsoft redirects users post-authentication; must match Azure settings.  |
| `JWT_SECRET`              | Secret key for signing JWTs (e.g., a random string)                         | Ensures the integrity and authenticity of session tokens issued by the app.                |
| `DB_HOST`                 | Database host (e.g., `localhost`)                                           | Connects to the PostgreSQL database for storing user and session data.                     |
| `DB_PORT`                 | Database port (e.g., `5432`)                                                | Specifies the port for database communication.                                             |
| `DB_NAME`                 | Database name (e.g., `mcp_db`)                                              | Identifies the database instance.                                                          |
| `DB_USER`                 | Database username                                                           | Authenticates the app with the database.                                                   |
| `DB_PASSWORD`             | Database password                                                           | Secures database access.                                                                   |

### Why These Are Needed
- **Security**: Client IDs, secrets, and JWT secrets are sensitive credentials that enable secure communication with external services and protect user sessions.
- **Configuration**: Redirect URIs ensure OAuth flows complete correctly, while database variables enable persistent storage of authentication data.
- **Flexibility**: The port allows customization of the server's network settings.

---

## Sign-In Process

The sign-in process uses OAuth 2.0 to authenticate users with Google or Microsoft, storing tokens and creating a session with a JSON Web Token (JWT). Below is a technical breakdown of the flow up to successful sign-in.

### Architecture
- **Frontend**: 
  - `src/client/index.html`: Displays sign-in buttons.
  - `src/client/script.js`: Initiates the sign-in flow and handles redirection.
- **Backend**: 
  - `src/core/auth.js`: Generates OAuth URLs and retrieves tokens.
  - `src/core/routes.js`: Defines authentication endpoints.
  - `src/core/database.js`: Manages user and session data in PostgreSQL.
  - `src/core/server.js`: Registers routes and starts the Hapi server.
- **Dependencies**: 
  - `googleapis`: Google API client.
  - `@microsoft/microsoft-graph-client`: Microsoft Graph API client.
  - `jsonwebtoken`: JWT generation.
  - `pg`: PostgreSQL client.
  - `dotenv`: Environment variable management.

### Steps

#### 1. User Initiates Sign-In
- **Trigger**: User visits `http://localhost:3000/` and clicks either "Login with Google" or "Login with Microsoft" on the frontend (`index.html`).
- **Frontend Action**: 
  - `script.js` listens for button clicks:
    ```javascript
    document.getElementById('login-google-btn').addEventListener('click', () => {
        window.location.href = 'http://localhost:3000/auth/google';
    });
    document.getElementById('login-microsoft-btn').addEventListener('click', () => {
        window.location.href = 'http://localhost:3000/auth/microsoft';
    });
    ```
  - Redirects to the respective backend endpoint.

#### 2. Backend Generates OAuth URL
- **Endpoint**: 
  - Google: `GET /auth/google`
  - Microsoft: `GET /auth/microsoft`
- **Logic**: `src/core/auth.js`
  - Google:
    ```javascript
    function getGoogleAuthUrl() {
        const url = googleOauth2Client.generateAuthUrl({
            access_type: 'offline',
            scope: GOOGLE_SCOPES,
            prompt: 'consent',
        });
        logger.info('Generated Google OAuth URL');
        return url;
    }
    ```
  - Microsoft:
    ```javascript
    function getMicrosoftAuthUrl() {
        const url = `https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=${process.env.MICROSOFT_CLIENT_ID}&redirect_uri=${encodeURIComponent(process.env.MICROSOFT_REDIRECT_URI)}&response_type=code&scope=${MICROSOFT_SCOPES.join(' ')}`;
        logger.info('Generated Microsoft OAuth URL');
        return url;
    }
    ```
- **Route**: `src/core/routes.js`
  - Redirects the user to the OAuth provider's consent page:
    ```javascript
    {
        method: 'GET',
        path: '/auth/google',
        handler: (request, h) => {
            const url = auth.getGoogleAuthUrl();
            logger.info('Redirecting to Google OAuth');
            return h.redirect(url);
        },
    },
    {
        method: 'GET',
        path: '/auth/microsoft',
        handler: (request, h) => {
            const url = auth.getMicrosoftAuthUrl();
            logger.info('Redirecting to Microsoft OAuth');
            return h.redirect(url);
        },
    }
    ```

#### 3. User Grants Consent
- **Google**: User logs into their Google account and grants permissions for calendar, Gmail, and user info.
- **Microsoft**: User logs into their Microsoft account and grants permissions for email, calendar, and Teams meetings.
- **Result**: OAuth provider redirects back to the specified redirect URI with an authorization code (e.g., `http://localhost:3000/auth/google/callback?code=...` or `.../auth/microsoft/callback?code=...`).

#### 4. Backend Exchanges Code for Tokens
- **Endpoint**: 
  - Google: `GET /auth/google/callback`
  - Microsoft: `GET /auth/microsoft/callback`
- **Logic**: `src/core/auth.js`
  - Google:
    ```javascript
    async function getGoogleTokens(code) {
        const { tokens } = await googleOauth2Client.getToken(code);
        logger.info('Successfully retrieved Google tokens');
        return tokens;
    }
    ```
  - Microsoft:
    ```javascript
    async function getMicrosoftTokens(code) {
        const response = await fetch('https://login.microsoftonline.com/common/oauth2/v2.0/token', {
            method: 'POST',
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
            body: new URLSearchParams({
                client_id: process.env.MICROSOFT_CLIENT_ID,
                client_secret: process.env.MICROSOFT_CLIENT_SECRET,
                code,
                redirect_uri: process.env.MICROSOFT_REDIRECT_URI,
                grant_type: 'authorization_code',
            }),
        });
        const tokens = await response.json();
        logger.info('Successfully retrieved Microsoft tokens');
        return tokens;
    }
    ```
- **Route**: `src/core/routes.js`
  - Exchanges the code for access and refresh tokens, retrieves user email, and stores tokens in the database:
    ```javascript
    {
        method: 'GET',
        path: '/auth/google/callback',
        handler: async (request, h) => {
            const code = request.query.code;
            const tokens = await auth.getGoogleTokens(code);
            auth.googleOauth2Client.setCredentials({ access_token: tokens.access_token });
            const userInfo = await google.oauth2({ version: 'v2' }).userinfo.get();
            const email = userInfo.data.email;
            await pool.query(
                `INSERT INTO users (email, google_access_token, google_refresh_token, google_token_expiry)
                VALUES ($1, $2, $3, $4)
                ON CONFLICT (email) DO UPDATE SET ...`,
                [email, tokens.access_token, tokens.refresh_token, new Date(tokens.expiry_date)]
            );
            // ... (JWT and session creation below)
        },
    },
    {
        method: 'GET',
        path: '/auth/microsoft/callback',
        handler: async (request, h) => {
            const code = request.query.code;
            const tokens = await auth.getMicrosoftTokens(code);
            const graphClient = require('@microsoft/microsoft-graph-client').Client.init({
                authProvider: (done) => done(null, tokens.access_token),
            });
            const userInfo = await graphClient.api('/me').get();
            const email = userInfo.mail || userInfo.userPrincipalName;
            await pool.query(
                `INSERT INTO users (email, microsoft_access_token, microsoft_refresh_token, microsoft_token_expiry)
                VALUES ($1, $2, $3, $4)
                ON CONFLICT (email) DO UPDATE SET ...`,
                [email, tokens.access_token, tokens.refresh_token, new Date(Date.now() + tokens.expires_in * 1000)]
            );
            // ... (JWT and session creation below)
        },
    }
    ```

#### 5. Create Session and Issue JWT
- **Logic**: `src/core/routes.js` (within callback handlers)
  - Generates a JWT and stores it in the sessions table:
    ```javascript
    const jwtToken = jwt.sign({ email }, process.env.JWT_SECRET, { expiresIn: '1h' });
    const expiresAt = new Date(Date.now() + 60 * 60 * 1000);
    await pool.query(
        `INSERT INTO sessions (user_email, jwt_token, expires_at)
        VALUES ($1, $2, $3)
        ON CONFLICT (user_email) DO UPDATE SET ...`,
        [email, jwtToken, expiresAt]
    );
    logger.info(`Session created for user: ${email}`);
    ```
  - Redirect: Sends the user to the frontend with the JWT:
    ```return h.redirect(`/?token=${jwtToken}`);```

#### 6. Frontend Handles Sign-In Completion
- **Logic**: `src/client/script.js`
  - Extracts the JWT from the URL and stores it, setting the provider:
    ```javascript
    const token = new URLSearchParams(window.location.search).get('token');
    if (token) {
        localStorage.setItem('jwt', token);
        const provider = window.location.href.includes('microsoft') ? 'microsoft' : 'google';
        localStorage.setItem('provider', provider);
        window.history.replaceState({}, document.title, '/');
        showDashboard();
    }
    ```
  - Displays the dashboard, completing the sign-in process.

### Database Schema
#### Users Table:
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    google_access_token TEXT,
    google_refresh_token TEXT,
    google_token_expiry TIMESTAMP,
    microsoft_access_token TEXT,
    microsoft_refresh_token TEXT,
    microsoft_token_expiry TIMESTAMP
);
```

#### Sessions Table:
```sql
CREATE TABLE sessions (
    id SERIAL PRIMARY KEY,
    user_email VARCHAR(255) REFERENCES users(email),
    jwt_token TEXT NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_email UNIQUE (user_email)
);
```

### Flow Diagram
```
User -> Frontend -> Backend (/auth/[google|microsoft]) -> OAuth Provider
   <- Redirect w/ Code <- Callback (/auth/[google|microsoft]/callback) <- Tokens
Frontend <- Redirect w/ JWT <- Backend (Stores Tokens & JWT in DB)
```

### Notes
- **Error Handling**: Logs errors (e.g., missing code, token retrieval failures) to `logs/error.log`.
- **Security**: Tokens are stored securely in the database; JWTs are short-lived (1 hour).
- **Scalability**: Supports both Google and Microsoft with separate flows, extensible for additional providers.

This concludes the sign-in process documentation up to successful authentication.

---

### Instructions
1. **Save the File**: Copy the above content into a file named `signin.md` in your project root or a `docs` folder.
2. **Review**: Ensure the `.env` variables match your setup, and update placeholders (e.g., database name) if needed.
3. **Enhance**: Add diagrams (e.g., using Mermaid syntax) or more details if required for your team. 
