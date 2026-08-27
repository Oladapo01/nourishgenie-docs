# Auth Service

This repository contains the Authentication Service, a core microservice for the NourishGenie application backend. It is responsible for all aspects of user identity, authentication, authorization, and account management.

-----

## 1\. Overview

The Auth Service is a critical component in the NourishGenie microservices ecosystem. It provides secure and robust mechanisms for user registration, login, password management (including reset), and token issuance. It interacts with **PostgreSQL** for persistent user data storage and **Redis** for ephemeral data like session management (JWT blacklisting) and password reset codes, and also for inter-service communication (email triggers).

-----

## 2\. Features

  * **User Registration**: Allows new users to create accounts with strong password validation.
  * **User Login**: Authenticates users and issues secure **JWT access and refresh tokens**.
  * **OAuth Login (Google / Apple)**: Accepts a provider-issued ID token, cryptographically verifies it against the provider's published JWKS before trusting any claim in it, and issues the same access/refresh token pair as password login. New OAuth users are flagged so the client can route them through onboarding.
  * **Token Refreshment**: Enables clients to obtain new access tokens using refresh tokens without re-authenticating.
  * **Password Management**:
      * **Change Password**: Allows authenticated users to update their password.
      * **Forgot Password**: Initiates a secure password reset flow using time-bound verification codes sent via email.
      * **Reset Password**: Completes the password reset process after code verification.
  * **Account Deletion**: Provides a secure way for users to delete their accounts.
  * **Token Invalidation (Logout/Password Reset)**: Blacklists JWT tokens to immediately revoke access for security events like logout or password changes.
  * **Secure Password Storage**: Uses `bcrypt` for hashing and salting passwords.
  * **Rate Limiting (Internal)**: Implements internal rate limiting for failed login attempts to mitigate brute-force attacks.
  * **Database Schema Initialization**: Automated database setup upon container startup.
  * **Health Check**: Provides a `/health` endpoint for monitoring service availability.
  * **Logging**: Comprehensive logging for operational insights and debugging.
  * **Environment-based Configuration**: All sensitive data and configurations are managed via environment variables and Docker secrets.

-----

## 3\. Technology Stack

  * **Python 3.11**: Primary programming language.
  * **FastAPI**: Modern, fast (high-performance) web framework for building APIs with Python 3.7+ based on standard Python type hints.
  * **Uvicorn**: ASGI server for running FastAPI applications.
  * **PostgreSQL**: Relational database for persistent user data.
      * `asyncpg`: Asynchronous PostgreSQL driver for high-performance database interactions.
      * `psycopg2`: Synchronous PostgreSQL driver used in `init_db.py`.
  * **Redis**: In-memory data store for caching, rate limiting, token blacklisting, and message queuing (Pub/Sub).
      * `redis-py`: Python client for Redis.
  * **`python-jose[cryptography]` / `PyJWT`**: For JSON Web Token (JWT) creation and verification. `PyJWKClient` is used specifically to fetch and cache Google's and Apple's public JWKS for verifying OAuth ID token signatures.
  * **`passlib`**: For secure password hashing (bcrypt).
  * **`Pydantic`**: For data validation and settings management.
  * **Docker**: For containerization and deployment.

-----

## 4\. Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### 4.1. Prerequisites

  * Python 3.11+
  * `pip`
  * Docker and Docker Compose (recommended for local development)
  * PostgreSQL instance (local or remote)
  * Redis instance (local or remote)

### 4.2. Local Setup (without Docker)

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/Oladapo01/FoodTrackerApp.git/auth-service
    cd auth-service
    ```

2.  **Create a virtual environment and activate it:**

    ```bash
    python3.11 -m venv venv
    source venv/bin/activate
    ```

3.  **Install dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

4.  **Set up environment variables and secrets:**

    Create a `.env` file in the root of the project (for local development convenience, **do not commit this file to Git**):

    ```
    POSTGRES_HOST=localhost
    POSTGRES_PORT=5432
    POSTGRES_DB=authdb
    POSTGRES_USER=postgres
    # POSTGRES_PASSWORD=your_db_password # Use a secrets file in production

    REDIS_HOST=localhost
    REDIS_PORT=6379
    # REDIS_PASSWORD=your_redis_password # Use a secrets file in production

    # TOKEN_EXPIRATION_MINUTES=30
    # REFRESH_TOKEN_EXPIRATION_DAYS=30
    # RESET_CODE_EXPIRATION_MINUTES=15

    # These would typically be managed via Docker secrets or Kubernetes secrets
    # For local testing without Docker, you can create them as local files:
    # echo "your_strong_db_password" > /run/secrets/db_password
    # echo "your_strong_redis_password" > /run/secrets/redis_password
    # echo "your_super_secret_jwt_key" > /run/secrets/jwt_secret
    ```

    **Important Note on Secrets:** In production, sensitive values like `DB_PASSWORD`, `REDIS_PASSWORD`, and `JWT_SECRET` **must** be managed using secure mechanisms like Docker Secrets (as implemented in the `Dockerfile` and `main.py`) or Kubernetes Secrets, and **never** hardcoded or exposed directly in `.env` files that are committed to version control. For purely local, non-Docker development, you'd need to manually create these `/run/secrets/` files or modify `main.py` to read from a different path/environment variable.

5.  **Initialize the database schema:**

    ```bash
    python init_db.py
    ```

    Ensure your PostgreSQL instance is running and accessible before running this.

6.  **Run the application:**

    ```bash
    uvicorn main:app --host 0.0.0.0 --port 8081 --workers 1
    ```

    For development, you can use `uvicorn main:app --reload` for automatic code reloading, but `workers` should be 1 with `reload`.

### 4.3. Docker Deployment

The `Dockerfile` provides a production-ready image for the Auth Service. It builds on `python:3.11-slim`, installs dependencies, sets up a non-root user, and defines the startup command.

1.  **Build the Docker image:**

    ```bash
    docker build -t your-repo/auth-service:latest .
    ```

    Replace `your-repo/auth-service` with your desired image name.

2.  **Run the Docker container (example with Docker Compose):**

    It's highly recommended to use **Docker Compose** for local development and orchestration with other services (API Gateway, PostgreSQL, Redis). A `docker-compose.yml` snippet for the Auth Service would look like this:

    ```yaml
    version: '3.8'
    services:
      auth-service:
        build:
          context: ./auth-service # Assuming auth-service is a sub-directory
          dockerfile: Dockerfile
        ports:
          - "8081:8081"
        environment:
          POSTGRES_HOST: postgres
          POSTGRES_PORT: 5432
          POSTGRES_DB: authdb
          POSTGRES_USER: postgres
          REDIS_HOST: redis
          REDIS_PORT: 6379
          TOKEN_EXPIRATION_MINUTES: 30
          REFRESH_TOKEN_EXPIRATION_DAYS: 30
          RESET_CODE_EXPIRATION_MINUTES: 15
        secrets:
          - db_password
          - redis_password
          - jwt_secret
        depends_on:
          postgres:
            condition: service_healthy # Or service_started if no healthcheck
          redis:
            condition: service_healthy
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:8081/health"]
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 20s # Give time for init_db.py to run
      postgres:
        image: postgres:13
        environment:
          POSTGRES_DB: authdb
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD_FILE: /run/secrets/db_password
        secrets:
          - db_password
        volumes:
          - postgres_data:/var/lib/postgresql/data
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres -d authdb"]
          interval: 10s
          timeout: 5s
          retries: 5
      redis:
        image: redis:6-alpine
        command: redis-server --requirepass $(cat /run/secrets/redis_password) # Use password from secret
        secrets:
          - redis_password
        healthcheck:
          test: ["CMD-SHELL", "redis-cli -a $(cat /run/secrets/redis_password) ping"]
          interval: 10s
          timeout: 5s
          retries: 5
    secrets:
      db_password:
        file: ./secrets/db_password.txt # Create this file with your DB password
      redis_password:
        file: ./secrets/redis_password.txt # Create this file with your Redis password
      jwt_secret:
        file: ./secrets/jwt_secret.txt # Create this file with a strong, random JWT secret
    volumes:
      postgres_data:
    ```

    To run this `docker-compose.yml`:

    1.  Create a `secrets` directory in your root project folder.
    2.  Inside `secrets`, create `db_password.txt`, `redis_password.txt`, and `jwt_secret.txt` and fill them with strong, random passwords/secrets.
    3.  From the directory containing your `docker-compose.yml`, run:
        ```bash
        docker compose up --build
        ```

-----

## 5\. API Endpoints

The Auth Service exposes the following RESTful API endpoints:

### 5.1. User Authentication and Management

#### `POST /login`

  * **Description**: Authenticates a user with email and password, issuing access and refresh tokens upon successful login. Implements internal rate limiting for failed attempts.
  * **Request Body**:
    ```json
    {
      "email": "user@example.com",
      "password": "SecurePassword123!"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "access_token": "eyJ...",
      "refresh_token": "eyJ...",
      "expires_in": 1800, // Access token expiration in seconds (30 minutes)
      "user_id": "uuid-of-user",
      "name": "User Name"
    }
    ```
  * **Errors**: `401 Unauthorized` (Invalid credentials), `429 Too Many Requests`, `500 Internal Server Error`.

#### `POST /register`

  * **Description**: Registers a new user account. Enforces strong password policies.
  * **Request Body**:
    ```json
    {
      "email": "newuser@example.com",
      "password": "StrongPassword123!",
      "name": "New User"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "id": "uuid-of-new-user",
      "email": "newuser@example.com",
      "name": "New User"
    }
    ```
  * **Errors**: `409 Conflict` (User already exists), `422 Unprocessable Entity` (Invalid password format), `500 Internal Server Error`.

#### `POST /oauth/login`

  * **Description**: Authenticates a user via a Google or Apple ID token. The token's signature and claims (issuer, audience, expiry) are verified server-side against the provider's published JWKS before the token is trusted — the service does not rely on the client's assertion that the token is valid. On success, issues the same access/refresh token pair as `/login`. The response indicates whether the account was just created (`is_new_user`) so the client can route first-time OAuth users through onboarding rather than assuming an existing profile.
  * **Response (200 OK)**: Same shape as `POST /login`, plus an `is_new_user` boolean.
  * **Errors**: `401 Unauthorized` (token fails provider signature/claim verification), `500 Internal Server Error`.
  * **Note**: This endpoint previously issued tokens without verifying the ID token against the provider — a critical account-takeover vulnerability, since any caller could submit an unverified token claiming to be any user. This was fixed by validating every OAuth token server-side via `PyJWKClient` before a session is issued.

#### `POST /refresh`

  * **Description**: Exchanges a valid refresh token for a new access token.
  * **Request Body**:
    ```json
    {
      "refresh_token": "eyJ..."
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "access_token": "eyJ...",
      "refresh_token": "eyJ...", // The same refresh token is returned
      "expires_in": 1800,
      "user_id": "uuid-of-user",
      "name": "User Name"
    }
    ```
  * **Errors**: `401 Unauthorized` (Invalid/expired/blacklisted refresh token), `404 Not Found` (User associated with token not found), `500 Internal Server Error`.
  * **Reliability Note**: The blacklist check against Redis is guarded — if Redis is unreachable, the endpoint logs the failure and fails open (treats the token as not blacklisted) rather than returning a 500 to every client. This was a deliberate availability trade-off: a Redis outage previously broke token refresh for every active session; the guard keeps the service usable during a Redis incident at the cost of not enforcing blacklist revocation until Redis recovers.

#### `PUT /change-password`

  * **Description**: Allows an authenticated user to change their password. Requires the current password for verification.
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Request Body**:
    ```json
    {
      "currentPassword": "OldSecurePassword123!",
      "newPassword": "NewStrongPassword123!"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "message": "Password changed successfully"
    }
    ```
  * **Errors**: `401 Unauthorized` (Invalid/missing token), `400 Bad Request` (Incorrect current password, invalid new password format), `404 Not Found` (User not found), `500 Internal Server Error`.

#### `DELETE /delete-account`

  * **Description**: Permanently deletes the authenticated user's account. Requires password and a confirmation phrase for security.
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Request Body**:
    ```json
    {
      "password": "UserCurrentPassword123!",
      "confirmationText": "delete my account"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "message": "Account deleted successfully"
    }
    ```
  * **Errors**: `401 Unauthorized` (Invalid/missing token), `400 Bad Request` (Incorrect password, invalid confirmation text), `404 Not Found` (User not found), `500 Internal Server Error`.

#### `POST /logout`

  * **Description**: Revokes the current access token by blacklisting it in Redis.
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Response (200 OK)**:
    ```json
    {
      "message": "Logout successful"
    }
    ```
  * **Errors**: `401 Unauthorized` (Invalid/missing token), `500 Internal Server Error`.

#### `GET /profile`

  * **Description**: Retrieves the profile information (ID, email, name) for the authenticated user.
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Response (200 OK)**:
    ```json
    {
      "id": "uuid-of-user",
      "email": "user@example.com",
      "name": "User Name"
    }
    ```
  * **Errors**: `401 Unauthorized` (Invalid/missing token), `404 Not Found` (User not found), `500 Internal Server Error`.

### 5.2. Password Reset Flow

#### `POST /forgot-password`

  * **Description**: Initiates the password reset process. If the email is registered, a 6-digit verification code is generated, stored in Redis, and a message is published to Redis for the Email Service to send the code.
  * **Request Body**:
    ```json
    {
      "email": "user@example.com"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "message": "If your email is registered, you will receive a reset code"
    }
    ```
    **Note**: This generic message is used to prevent email enumeration attacks (it doesn't reveal if an email exists).
  * **Errors**: `429 Too Many Requests` (internal rate limit exceeded), `500 Internal Server Error`.

#### `POST /verify-reset-code`

  * **Description**: Verifies the 6-digit code received by the user against the one stored in Redis.
  * **Request Body**:
    ```json
    {
      "email": "user@example.com",
      "code": "123456"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "message": "Code verified successfully"
    }
    ```
  * **Errors**: `400 Bad Request` (Invalid or expired code), `500 Internal Server Error`.

#### `POST /reset-password`

  * **Description**: Resets the user's password after successful code verification. Invalidates the reset code in Redis and updates the user's password in the database.
  * **Request Body**:
    ```json
    {
      "email": "user@example.com",
      "code": "123456",
      "new_password": "NewSecurePassword123!"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "message": "Password reset successful"
    }
    ```
  * **Errors**: `400 Bad Request` (Invalid or expired code, invalid new password format), `404 Not Found` (User not found), `500 Internal Server Error`.

### 5.3. Health Check

#### `GET /health`

  * **Description**: Endpoint for readiness and liveness probes.
  * **Response (200 OK)**:
    ```json
    {
      "status": "healthy",
      "service": "auth-service"
    }
    ```

-----

## 6\. Security Considerations

The Auth Service implements several security best practices:

  * **Secret Management**: Sensitive credentials (database password, Redis password, JWT secret) are read from Docker secrets (mounted from `/run/secrets/`). This is crucial for production deployments.
  * **Password Hashing**: User passwords are never stored in plaintext. They are hashed using `bcrypt` with a configured number of rounds (`bcrypt__rounds=4`) to make brute-force attacks computationally infeasible.
  * **Strong Password Validation**: New passwords (registration, change, reset) are enforced to meet complexity requirements (min length, uppercase, lowercase, digit, special character).
  * **JWT Security**:
      * **HS256 Algorithm**: JWTs are signed with a strong secret using HMAC-SHA256.
      * **Short-lived Access Tokens**: Access tokens have a short expiration (default 30 minutes) to minimize the impact of token compromise.
      * **Refresh Tokens**: Longer-lived refresh tokens are used for obtaining new access tokens, reducing the need for frequent re-login.
      * **Token Blacklisting**: Upon logout or password reset, access tokens are added to a Redis blacklist with a TTL matching their original expiration. This immediately revokes their validity.
  * **OAuth Token Verification**: Google and Apple ID tokens submitted to `/oauth/login` are verified server-side against the provider's published JWKS (via `PyJWKClient`) — signature, issuer, and audience are all checked before the token is trusted. This closed a previously-shipped account-takeover vulnerability where the endpoint issued session tokens for any submitted ID token without verifying it against the provider, meaning a forged or replayed token could be used to authenticate as any user.
  * **Auth-Only Password Constraint**: Because OAuth accounts have no password, `users.password` is nullable at the schema level, enforced by a conditional `CHECK` constraint that still requires a password for email/password accounts. This avoids the common anti-pattern of storing a dummy/placeholder password for OAuth users.
  * **Password Reset Flow Security**:
      * **One-Time Codes**: 6-digit verification codes are randomly generated and have a strict 15-minute expiration in Redis.
      * **No Email Enumeration**: The `forgot-password` endpoint returns a generic success message regardless of email existence to prevent attackers from discovering valid user emails.
      * **Code Invalidation on Use**: Once a password reset is successful, the verification code is immediately deleted from Redis.
  * **Rate Limiting on Login**: The `login` endpoint checks recent failed attempts for a given email *before* verifying the submitted credentials, and rejects the request once the threshold is exceeded regardless of whether the password is correct. This ordering matters: an earlier version checked the lockout after credential verification, which meant a correct password could succeed no matter how many prior failed attempts had occurred, defeating the purpose of the lockout. The check was moved ahead of verification to close that gap.
  * **Non-Root User**: The Dockerfile creates and runs the application as a non-root `appuser` for enhanced container security.
  * **CORS**: Configured to allow all origins in development (`allow_origins=["*"]`), but **must be restricted to specific trusted origins in production** to prevent cross-site request forgery (CSRF) and other attacks.
  * **Consistent Database Targeting**: `main.py` and `init_db.py` previously parsed `DATABASE_URL` differently, which allowed the running application to connect to a different database than the one the schema-initialization script had set up — a class of bug that is easy to introduce and easy to miss until a specific environment surfaces it. The parsing was unified so both always resolve to the same database.
  * **Error Message Hardening**: Endpoints were audited to stop returning raw exception text (`str(e)`) in API responses, replacing it with generic, safe error messages so internal implementation details (stack traces, query fragments, library errors) aren't leaked to clients.
  * **Secure Coding Practices**: FastAPI's Pydantic models enforce schema validation, reducing the risk of injection attacks and invalid data processing.

-----

## 7\. Project Structure

  * **`main.py`**: The primary FastAPI application file, containing all API endpoints, business logic, dependency injections, and core configurations (database, Redis, JWT).
  * **`init_db.py`**: A standalone script executed at container startup to ensure the PostgreSQL database schema (tables, indexes) is initialized before the main application starts. It originally included a DNS-based wait-for-database check (`socket.gethostbyname()`), which was removed after it was identified as the cause of startup crash loops on Railway: Railway's private networking is IPv6-only, and `socket.gethostbyname()` only resolves IPv4 addresses, so the readiness check failed even when the database was reachable.
  * **`run_migrations.py`**: Applies versioned schema migrations (e.g. making `users.password` nullable for OAuth accounts) using a Postgres advisory lock so that concurrent container starts don't race to apply the same migration twice.
  * **`Dockerfile`**: Defines the Docker image for the Auth Service, including dependencies, environment setup, non-root user creation, and the startup command.
  * **`requirements.txt`**: Lists all Python dependencies required by the service.
  * **`secrets/`**: (Conceptual directory for Docker Compose) Contains sensitive files like `db_password.txt`, `redis_password.txt`, `jwt_secret.txt` which are mounted as Docker secrets into the container.

-----

## 8\. Development and Contribution

### 8.1. Logging and Debugging

The service utilizes Python's `logging` module for structured logs. You'll see `logger.info`, `logger.warning`, and `logger.error` messages in the console/container logs. The FastAPI middleware also includes `print` statements for immediate console output during development.

### 8.2. Database Schema

The `init_db.py` script creates the following tables:

  * **`users`**:
      * `id` (`VARCHAR(36)` PK)
      * `email` (`VARCHAR(255)` UNIQUE NOT NULL)
      * `password` (`VARCHAR(255)`, nullable) - Hashed password for email/password accounts. Nullable to support OAuth accounts, which have no password; enforced by a migration-added conditional `CHECK` constraint requiring a password only for non-OAuth accounts.
      * `name` (`VARCHAR(255)`)
      * `is_active` (`BOOLEAN` DEFAULT TRUE)
      * `created_at` (`TIMESTAMP WITH TIME ZONE` NOT NULL)
      * `updated_at` (`TIMESTAMP WITH TIME ZONE`)
      * `last_login` (`TIMESTAMP WITH TIME ZONE`)
      * `idx_users_email` (Index on `email`)
  * **`failed_login_attempts`**:
      * `id` (`SERIAL` PK)
      * `email` (`VARCHAR(255)` NOT NULL)
      * `ip_address` (`VARCHAR(45)`) - Future enhancement to log IP from `X-Forwarded-For`
      * `attempt_time` (`TIMESTAMP WITH TIME ZONE` NOT NULL)
      * `idx_failed_logins` (Index on `email`, `attempt_time` for efficient rate limit checks)

### 8.3. Extending Functionality

  * **New Authentication Methods**: To add methods like social logins (Google, Facebook), you would extend `main.py` with new endpoints and integrate with external OAuth providers.
  * **Role-Based Access Control (RBAC)**: Currently, tokens only contain `user_id`. To implement RBAC, you would:
    1.  Add a `roles` column to the `users` table.
    2.  Include `roles` in the JWT `TokenPayload`.
    3.  Implement a dependency in FastAPI to check user roles for specific endpoints (e.g., `Depends(role_checker(['admin']))`).
  * **Multi-Factor Authentication (MFA)**: This would involve adding new endpoints for MFA setup, verification, and integrating with an MFA provider (e.g., Twilio for SMS codes, Google Authenticator for TOTP).

-----

## 9\. Monitoring and Health

  * **`/health` Endpoint**: Provides a basic health check for container orchestration systems.
  * **Structured Logging**: Configure log aggregation tools (e.g., Loki, Elasticsearch) to collect and analyze logs from the service. This is vital for debugging issues in production.
  * **Metrics (Future)**: Integrate a Prometheus client library to expose detailed metrics (request counts, latency, error rates) for scraping by Prometheus and visualization in Grafana.

-----

## 10\. Production Hardening & Incident History

The items below are resolved production issues, kept here as a record of the reasoning behind current behavior rather than as a changelog of code diffs.

  * **OAuth Account-Takeover Vulnerability (Critical)**: `/oauth/login` originally issued session tokens for any submitted ID token without verifying it against the issuing provider (Google/Apple). This meant the endpoint trusted claims it had no way to confirm — a forged or replayed token could authenticate as any user. Fixed by verifying every OAuth token server-side against the provider's JWKS (`PyJWKClient`) before a session is created.
  * **Database Connection Divergence**: `main.py` and `init_db.py` parsed `DATABASE_URL` differently, so the running service and the schema-initialization script could resolve to different databases in some environments. Unified the parsing so both always target the same database — a bug of this kind is silent until it isn't, and is disproportionately costly to diagnose in production.
  * **IPv6-Only Networking Incompatibility**: `init_db.py`'s startup readiness check used `socket.gethostbyname()`, which is IPv4-only. Railway's private network is IPv6-only, so the check failed unconditionally and caused startup crash loops even when the database was healthy and reachable. Removed the IPv4-only check.
  * **Login Lockout Ordering**: The failed-login-attempt check was being evaluated after credential verification rather than before it, so a correct password bypassed the lockout regardless of how many prior failed attempts existed. Reordered so the lockout check gates access before credentials are verified.
  * **Redis-Unavailable Fail-Open Decision**: The refresh-token blacklist check against Redis was unguarded, so a Redis outage returned `500` on every token refresh — effectively logging out every active user during an unrelated infrastructure incident. Guarded the check to log and fail open (treat as not blacklisted) when Redis is unreachable, trading strict blacklist enforcement during an outage for continued service availability, a deliberate and documented trade-off rather than an oversight.
  * **Error Message Leakage**: Raw exception text (`str(e)`) was being returned directly in some API error responses. Replaced with generic, safe messages so internal implementation details aren't exposed to clients.

-----

*Created: July 4, 2025*
*Last Updated: August 27, 2026*
