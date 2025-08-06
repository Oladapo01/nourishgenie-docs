# Email Service

This repository contains the Email Service, a dedicated microservice for the Food Tracker application backend. Its sole responsibility is to send various types of transactional emails, such as password reset verification codes, to users. It integrates with Redis for receiving email sending requests asynchronously and uses an SMTP server for delivery.

-----

## 1\. Overview

The Email Service is designed to decouple email sending logic from other microservices, improving system resilience and scalability. Instead of directly sending emails, other services (like the Auth Service) publish messages to a Redis Pub/Sub channel, and the Email Service consumes these messages and dispatches the emails. This asynchronous pattern prevents email sending failures from impacting core business logic.

-----

## 2\. Features

  * **Asynchronous Email Sending**: Listens for email events via Redis Pub/Sub, ensuring that email sending does not block the calling service's operations.
  * **Transactional Email Support**: Currently configured for password reset emails. Easily extensible for other transactional emails (e.g., welcome emails, order confirmations).
  * **SMTP Integration**: Uses standard SMTP for email delivery, supporting TLS encryption.
  * **HTML Email Templates**: Utilizes a pre-defined HTML template for password reset emails, ensuring consistent branding and readability.
  * **Internal Access Only**: Designed to be invoked by other internal services (e.g., via the API Gateway's internal routing) and not directly exposed to external clients for security.
  * **JWT Protected API Endpoint**: The `/send-reset-code` endpoint is protected by JWT verification, ensuring only authorized internal services can trigger email sends via direct HTTP calls.
  * **Health Check**: Provides a `/health` endpoint for monitoring service availability.
  * **Environment-based Configuration**: All sensitive data and configurations are managed via environment variables and Docker secrets.
  * **Robust Logging**: Provides detailed logs for troubleshooting email delivery issues.

-----

## 3\. Technology Stack

  * **Python 3.11**: Primary programming language.
  * **Flask**: Lightweight web framework for handling HTTP requests (though its primary role is asynchronous processing).
  * **Gunicorn**: WSGI HTTP server for running the Flask application in production.
  * **Redis**: In-memory data store used as a message broker (Pub/Sub) for email sending events.
      * `redis-py`: Python client for Redis.
  * **`PyJWT`**: For JSON Web Token (JWT) verification when direct HTTP requests are made.
  * **`email` package**: Python's standard library for constructing email messages.
  * **`smtplib`**: Python's standard library for sending emails using the SMTP protocol.
  * **Docker**: For containerization and deployment.

-----

## 4\. Getting Started

These instructions will guide you through setting up and running the Email Service.

### 4.1. Prerequisites

  * Python 3.11+
  * `pip`
  * Docker and Docker Compose (recommended for local development)
  * Redis instance (local or remote)
  * SMTP server credentials (e.g., Outlook, Gmail, SendGrid, Mailgun)

### 4.2. Local Setup (without Docker)

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/Oladapo01/FoodTrackerApp.git/email-service
    cd email-service
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
    REDIS_HOST=localhost
    REDIS_PORT=6379
    # REDIS_PASSWORD=your_redis_password # Use a secrets file in production

    SMTP_HOST=smtp-mail.outlook.com
    SMTP_PORT=587
    SMTP_USERNAME=your_smtp_username@example.com
    SMTP_PASSWORD=your_smtp_password
    SENDER_EMAIL=noreply@foodtracker.example.com
    SENDER_NAME=Food Tracker App

    # This would typically be managed via Docker secrets or Kubernetes secrets
    # For local testing without Docker, you'd need to create this file:
    # echo "your_super_secret_jwt_key" > /run/secrets/jwt_secret
    ```

    **Important Note on Secrets:** In production, `REDIS_PASSWORD`, `SMTP_PASSWORD`, and `JWT_SECRET` **must** be managed using secure mechanisms like Docker Secrets (as implemented in the `Dockerfile` and `app.py`) or Kubernetes Secrets. **Never** hardcode secrets or commit them to version control. For purely local, non-Docker development, you'd need to manually create the `/run/secrets/jwt_secret` file or modify `app.py` to read the JWT secret from an environment variable directly if not using Docker secrets.

5.  **Run the application:**

    ```bash
    python app.py
    ```

    This will start the Flask application and the Redis listener in a background thread.

### 4.3. Docker Deployment

The `Dockerfile` provides a production-ready image for the Email Service. It installs necessary system dependencies, Python packages, sets up a non-root user, and defines the Gunicorn startup command.

1.  **Build the Docker image:**

    ```bash
    docker build -t your-repo/email-service:latest .
    ```

    Replace `your-repo/email-service` with your desired image name.

2.  **Run the Docker container (example with Docker Compose):**

    It's highly recommended to use **Docker Compose** for local development and orchestration with other services (API Gateway, Redis). A `docker-compose.yml` snippet for the Email Service would look like this:

    ```yaml
    version: '3.8'
    services:
      email-service:
        build:
          context: ./email-service # Assuming email-service is a sub-directory
          dockerfile: Dockerfile
        ports:
          - "8085:8085" # Exposed internally, API Gateway handles external access
        environment:
          REDIS_HOST: redis
          REDIS_PORT: 6379
          SMTP_HOST: smtp-mail.outlook.com
          SMTP_PORT: 587
          SMTP_USERNAME: your_smtp_username@example.com
          SENDER_EMAIL: noreply@foodtracker.example.com
          SENDER_NAME: Food Tracker App
        secrets:
          - redis_password
          - smtp_password # Create this secret file
          - jwt_secret
        depends_on:
          redis:
            condition: service_healthy
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:8085/health"]
          interval: 30s
          timeout: 10s
          retries: 5
      redis:
        image: redis:6-alpine
        command: redis-server --requirepass $(cat /run/secrets/redis_password)
        secrets:
          - redis_password
        healthcheck:
          test: ["CMD-SHELL", "redis-cli -a $(cat /run/secrets/redis_password) ping"]
          interval: 10s
          timeout: 5s
          retries: 5
    secrets:
      redis_password:
        file: ./secrets/redis_password.txt # Create this file with your Redis password
      smtp_password:
        file: ./secrets/smtp_password.txt # Create this file with your SMTP password
      jwt_secret:
        file: ./secrets/jwt_secret.txt # Create this file with a strong, random JWT secret
    ```

    To run this `docker-compose.yml`:

    1.  Create a `secrets` directory in your root project folder.
    2.  Inside `secrets`, create `redis_password.txt`, `smtp_password.txt`, and `jwt_secret.txt` and fill them with strong, random passwords/secrets.
    3.  From the directory containing your `docker-compose.yml`, run:
        ```bash
        docker compose up --build
        ```

-----

## 5\. Functionality and Usage

The Email Service primarily operates in two modes:

### 5.1. Asynchronous Event Listener (Recommended)

This is the primary and preferred method for other services to request emails. Services (like the Auth Service) publish messages to a Redis Pub/Sub channel.

  * **Redis Channel**: `password_reset_events`
  * **Message Format (JSON)**:
    ```json
    {
      "email": "recipient@example.com",
      "code": "123456",
      "timestamp": "2025-07-11T16:30:00.000Z"
      // Add other fields as needed for different email types
    }
    ```
  * **Workflow**:
    1.  Another microservice (e.g., Auth Service) generates an email event (e.g., password reset code).
    2.  It publishes a JSON message to the `password_reset_events` Redis channel.
    3.  The Email Service, running in a background thread, listens to this channel.
    4.  Upon receiving a message, it parses the JSON data.
    5.  It constructs the email (e.g., using `PASSWORD_RESET_TEMPLATE`).
    6.  It sends the email via the configured SMTP server.
    7.  Logs success or failure.

### 5.2. HTTP API Endpoint (Internal Use Only)

While asynchronous messaging is preferred, a direct HTTP endpoint is provided for scenarios where a synchronous request might be necessary or for internal service-to-service communication that doesn't utilize Redis Pub/Sub.

#### `POST /email-service/send-reset-code`

  * **Description**: Sends a password reset verification code email. This endpoint is intended for **internal service consumption only** and is protected by JWT verification. The API Gateway ensures this endpoint is not exposed directly to external clients.
  * **Authorization**: Requires a valid **JWT Access Token** in the `Authorization: Bearer <token>` header. This token should be issued by the Auth Service to an internal calling service (e.g., if another service needs to trigger an email).
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
      "message": "Password reset code sent successfully"
    }
    ```
  * **Errors**: `400 Bad Request` (missing email/code), `401 Unauthorized` (missing/invalid token), `500 Internal Server Error` (failed to send email).

### 5.3. Health Check

#### `GET /health`

  * **Description**: Endpoint for readiness and liveness probes.
  * **Response (200 OK)**:
    ```json
    {
      "status": "healthy",
      "service": "email-service"
    }
    ```

-----

## 6\. Configuration

The Email Service is configured via environment variables.

| Environment Variable            | Default Value               | Description                                                                 |
| :------------------------------ | :-------------------------- | :-------------------------------------------------------------------------- |
| `REDIS_HOST`                    | `redis`                     | Hostname or IP of the Redis server.                                         |
| `REDIS_PORT`                    | `6379`                      | Port of the Redis server.                                                   |
| `REDIS_PASSWORD`                | `(empty string)`            | Password for Redis authentication (read from `/run/secrets/redis_password`). **Mandatory in production.** |
| `SMTP_HOST`                     | `smtp-mail.outlook.com`     | SMTP server host for sending emails.                                        |
| `SMTP_PORT`                     | `587`                       | SMTP server port (e.g., 587 for TLS, 465 for SSL).                          |
| `SMTP_USERNAME`                 | `(empty string)`            | Username for SMTP authentication. **Mandatory in production.** |
| `SMTP_PASSWORD`                 | `(empty string)`            | Password for SMTP authentication (read from `/run/secrets/smtp_password`). **Mandatory in production.** |
| `SENDER_EMAIL`                  | `noreply@foodtracker.example.com` | The email address that appears as the sender.                           |
| `SENDER_NAME`                   | `Food Tracker App`          | The display name for the sender.                                            |
| `JWT_SECRET`                    | `development_placeholder_do_not_use_in_production` | Secret key for JWT verification (read from `/run/secrets/jwt_secret`). **Mandatory in production.** |
| `PYTHONDONTWRITEBYTECODE`       | `1`                         | Prevents Python from writing `.pyc` files.                                  |
| `PYTHONUNBUFFERED`              | `1`                         | Forces Python streams to be unbuffered.                                     |

**Note on `REDIS_PASSWORD` and `JWT_SECRET`**:
The `app.py` has fallback logic to attempt Redis connection without a password and to use a placeholder JWT secret if the secret file is not found. **This is strictly for development convenience.** In any production deployment, these secrets **must** be securely provided via Docker/Kubernetes secrets to ensure the service operates securely.

-----

## 7\. Security Considerations

  * **Secrets Management**: Critical credentials (`REDIS_PASSWORD`, `SMTP_PASSWORD`, `JWT_SECRET`) are loaded from Docker secrets, which is a secure method for injecting sensitive data into containers.
  * **Non-Root User**: The Dockerfile runs the application as a non-root `appuser`, limiting potential damage in case of a container compromise.
  * **Internal Service Only**: The API Gateway strictly controls access to the Email Service, preventing direct external access.
  * **JWT Protection for HTTP Endpoint**: The `/send-reset-code` endpoint requires a valid JWT, ensuring that only trusted internal services can trigger email sends via HTTP.
  * **TLS for SMTP**: The `smtplib` connection uses `starttls()` to encrypt communication with the SMTP server, protecting credentials and email content in transit.
  * **Rate Limiting (External)**: While the Email Service itself doesn't implement rate limiting for outgoing emails, the **API Gateway** imposes strict rate limits on the `/forgot-password` endpoint to prevent abuse of the password reset flow.

-----

## 8\. Project Structure

  * **`app.py`**: The main Flask application, containing:
      * Flask app initialization and CORS configuration.
      * Loading of environment variables and secrets.
      * Redis client initialization and background listener for Pub/Sub events.
      * SMTP email sending logic.
      * `token_required` decorator for JWT validation.
      * Email template definition.
      * `/email-service/send-reset-code` HTTP endpoint.
      * `/health` endpoint.
      * Background thread management for the Redis listener.
  * **`Dockerfile`**: Defines the Docker image for the Email Service.
  * **`requirements.txt`**: Lists all Python dependencies.
  * **`secrets/`**: (Conceptual directory for Docker Compose) Contains sensitive files like `redis_password.txt`, `smtp_password.txt`, `jwt_secret.txt` which are mounted as Docker secrets.

-----

## 9\. Monitoring and Troubleshooting

  * **Logs**: The service is configured to log informational messages and errors to standard output/error. These logs should be collected by your centralized logging solution (e.g., ELK Stack, Grafana Loki).
      * Look for `Error sending email` messages to diagnose SMTP issues.
      * Check for `Redis connection error` if the service cannot connect to Redis.
      * Monitor `Received password reset event` to confirm Pub/Sub messages are being processed.
  * **Health Check (`/health`)**: Use this endpoint in your container orchestration system to monitor the liveness and readiness of the Email Service instances.
  * **SMTP Server Logs**: If emails are not being received, check the logs of your configured SMTP server for delivery issues or authentication failures.
  * **Redis Monitoring**: Monitor your Redis instance's health, memory usage, and Pub/Sub activity. Ensure the `password_reset_events` channel has messages being published and consumed.
  * **Network Connectivity**: Verify network connectivity between the Email Service container and the Redis server, as well as between the Email Service container and the external SMTP server.
