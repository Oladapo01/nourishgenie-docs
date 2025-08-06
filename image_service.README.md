# Image Service

This repository contains the Image Service, a vital microservice for the Food Tracker application backend. Its primary function is to leverage AI for analyzing food images, extracting nutritional information, and providing meal suggestions. It integrates with OpenAI's Vision API, uses Redis for caching analysis results, and interacts with the User Service to save food entries.

-----

## 1\. Overview

The Image Service acts as the intelligent processing unit for user-uploaded food photos. When a user uploads an image, this service takes it, processes it, sends it to an external AI model for analysis, caches the results, and can then record the analysis as a food entry linked to the user. This ensures efficient, scalable, and AI-powered food recognition within the application.

-----

## 2\. Features

  * **AI-Powered Food Analysis**: Integrates with **OpenAI's GPT-4 Vision API** to interpret food images and extract detailed nutritional information (dish name, calories, macros, portion size, ingredients).
  * **Image Pre-processing**: Resizes and converts images to optimize them for AI model input.
  * **Intelligent Caching**: Utilizes **Redis** to cache image analysis results based on image hashes, significantly reducing redundant AI API calls and improving response times for duplicate images.
  * **Food Entry Persistence**: Optionally saves the analyzed food data as a new food entry in the **User Service** for the authenticated user.
  * **Robust Error Handling**: Implements retries for OpenAI API calls using `tenacity` to handle transient network issues or API rate limits.
  * **Authentication & Authorization**: Protects its endpoints using **JWT verification**, ensuring only authenticated users can submit images for analysis.
  * **Health Check**: Provides a `/health` endpoint with checks for Redis and OpenAI API key configuration, indicating service readiness.
  * **Secure Configuration**: Loads sensitive API keys and passwords from Docker secrets.
  * **Containerized Deployment**: Designed for easy deployment with Docker, including a custom entrypoint script for robust secret loading and service readiness checks.

-----

## 3\. Technology Stack

  * **Python 3.11**: Primary programming language.
  * **Flask**: Lightweight web framework for handling HTTP requests.
  * **Gunicorn**: WSGI HTTP server for running the Flask application in production.
  * **Pillow (PIL)**: For image manipulation (resizing, format conversion).
  * **Requests**: For making HTTP requests to the OpenAI API and User Service.
  * **Redis**: In-memory data store for caching and potentially for session/token management.
      * `redis-py`: Python client for Redis.
  * **`PyJWT`**: For JSON Web Token (JWT) verification.
  * **`tenacity`**: For adding retry logic to external API calls.
  * **Docker**: For containerization and deployment.
  * **OpenAI GPT-4 Vision API**: External AI service for image analysis.

-----

## 4\. Getting Started

These instructions will guide you through setting up and running the Image Service.

### 4.1. Prerequisites

  * Python 3.11+
  * `pip`
  * Docker and Docker Compose (recommended for local development)
  * Redis instance (local or remote)
  * OpenAI API Key (requires access to GPT-4 Vision model)
  * User Service (running and accessible if you want to save food entries)

### 4.2. Local Setup (without Docker - for development purposes)

Running the Image Service directly without Docker is possible for development but requires manual setup of environment variables/secrets.

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/Oladapo01/FoodTrackerApp.git/image-service
    cd image-service
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

    For local development **without Docker**, you'll need to manually manage environment variables and potentially create dummy secret files to match how `app.py` expects to read them from `/run/secrets/`.

    Create a `.env` file in the root of the project (for local development convenience, **do not commit this file to Git**):

    ```
    REDIS_HOST=localhost
    REDIS_PORT=6379
    # REDIS_PASSWORD=your_redis_password # Use a secrets file in production

    # OPENAI_API_KEY=your_openai_api_key # Use a secrets file in production
    OPENAI_MODEL=gpt-4-vision-preview
    OPENAI_API_TIMEOUT=30
    CACHE_TTL_SECONDS=3600

    # USER_SERVICE_URL points to the User Service. 
    # If running User Service locally, use its exposed port.
    USER_SERVICE_URL=http://localhost:8082

    # These would typically be managed via Docker secrets or Kubernetes secrets.
    # For local testing without Docker, you'd need to create them as local files.
    # Example:
    # mkdir -p /run/secrets # Create this directory
    # echo "your_strong_redis_password" > /run/secrets/redis_password
    # echo "sk-your_openai_api_key" > /run/secrets/openai_api_key
    # echo "your_super_secret_jwt_key" > /run/secrets/jwt_secret
    ```

    **Important Note on Secrets:** In production, sensitive values like `REDIS_PASSWORD`, `OPENAI_API_KEY`, and `JWT_SECRET` **must** be managed using secure mechanisms like Docker Secrets (as implemented in the `Dockerfile` and `docker-entrypoint.sh`) or Kubernetes Secrets. **Never** hardcode secrets or expose them directly in `.env` files that are committed to version control.

5.  **Run the application:**

    ```bash
    gunicorn --bind 0.0.0.0:8083 --workers 1 --threads 2 app:app
    ```

    (You might use `flask run` directly for simpler development, but Gunicorn is closer to the production setup).

### 4.3. Docker Deployment

The `Dockerfile` provides a production-ready image for the Image Service. It installs necessary system dependencies for image processing (like `ffmpeg`, `libgl1`), Python packages, sets up a non-root user, and uses a robust `docker-entrypoint.sh` script to manage secrets and service dependencies.

1.  **Build the Docker image:**

    ```bash
    docker build -t your-repo/image-service:latest .
    ```

    Replace `your-repo/image-service` with your desired image name.

2.  **Run the Docker container (example with Docker Compose):**

    It's highly recommended to use **Docker Compose** for local development and orchestration with other services (API Gateway, Redis, User Service). A `docker-compose.yml` snippet for the Image Service would look like this:

    ```yaml
    version: '3.8'
    services:
      image-service:
        build:
          context: ./image-service # Assuming image-service is a sub-directory
          dockerfile: Dockerfile
        ports:
          - "8083:8083" # Exposed internally, API Gateway handles external access
        environment:
          REDIS_HOST: redis
          REDIS_PORT: 6379
          USER_SERVICE_URL: http://user-service:8082 # Name of user service in compose network
          OPENAI_MODEL: gpt-4-vision-preview
          OPENAI_API_TIMEOUT: 60 # Increased timeout for potentially longer AI responses
          CACHE_TTL_SECONDS: 3600
        secrets:
          - redis_password
          - openai_api_key
          - jwt_secret
        depends_on:
          redis:
            condition: service_healthy
          user-service: # If you want to ensure user-service is up for saving entries
            condition: service_healthy
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:8083/health"]
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 45s # Allow time for entrypoint script and Redis connection
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
      user-service:
        # ... user-service configuration here
        # ensure it has a healthcheck and exposes its port
    secrets:
      redis_password:
        file: ./secrets/redis_password.txt # Create this file with your Redis password
      openai_api_key:
        file: ./secrets/openai_api_key.txt # Create this file with your OpenAI API key
      jwt_secret:
        file: ./secrets/jwt_secret.txt # Create this file with a strong, random JWT secret
    ```

    To run this `docker-compose.yml`:

    1.  Create a `secrets` directory in your root project folder.
    2.  Inside `secrets`, create `redis_password.txt`, `openai_api_key.txt`, and `jwt_secret.txt` and fill them with your actual secrets.
    3.  From the directory containing your `docker-compose.yml`, run:
        ```bash
        docker compose up --build
        ```

### 4.4. `docker-entrypoint.sh` Explained

This custom entrypoint script provides robust startup logic for the container:

  * **Secret Loading**: It waits for Docker secrets (mounted into `/run/secrets`) to become available before proceeding. This prevents the application from starting prematurely if secrets aren't ready. It then exports them as environment variables.
  * **Redis Readiness Check**: Uses `netcat` (`nc`) to verify that the Redis server is reachable and listening on its port before the main application starts. This ensures the service has its dependencies met.
  * **Execution**: Finally, it `exec`s the main `CMD` (Gunicorn server), ensuring proper signal handling by Docker.

-----

## 5\. API Endpoints

The Image Service exposes the following RESTful API endpoints:

### 5.1. Image Analysis

#### `POST /analyze-food`

  * **Description**: Analyzes an uploaded food image using OpenAI's GPT-4 Vision API to extract nutritional information. It checks a Redis cache first and saves results to cache. Optionally, it can save the analysis as a food entry to the User Service.
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Request Content**: `multipart/form-data` with an `image` field containing the image file.
  * **Query Parameters**:
      * `save` (optional, boolean): Set to `true` to save the analysis result as a food entry in the User Service. Default is `false`.
  * **Response (200 OK)**:
    ```json
    {
      "analysis": {
        "name": "Grilled Salmon with Asparagus",
        "calories": 450,
        "protein": 40,
        "carbs": 10,
        "fat": 25,
        "servingSize": "1 fillet, 6 spears",
        "ingredients": ["salmon", "asparagus", "olive oil", "lemon", "salt", "pepper"],
        "confidence": 0.92,
        "entryId": "uuid-of-new-food-entry" // Only if save=true
      },
      "source": "cache" // or "api"
    }
    ```
  * **Errors**: `400 Bad Request` (No image file), `401 Unauthorized` (Invalid/missing token), `500 Internal Server Error` (OpenAI API errors, image processing errors, User Service errors).

### 5.2. Health Check

#### `GET /health`

  * **Description**: Endpoint for readiness and liveness probes. Reports the health status of Redis connection and OpenAI API key configuration.
  * **Response (200 OK or 503 Service Unavailable)**:
    ```json
    {
      "status": "healthy", // or "unhealthy"
      "service": "image-service",
      "startup_mode": false, // or true during initial startup_period
      "checks": {
        "redis": "connected", // or "connection failed", "not configured", "error: ..."
        "openai": "configured" // or "missing credentials"
      }
    }
    ```

-----

## 6\. Security Considerations

  * **Secrets Management**: `OPENAI_API_KEY`, `REDIS_PASSWORD`, and `JWT_SECRET` are securely loaded from Docker secrets at container startup using the `docker-entrypoint.sh` script. This prevents sensitive information from being exposed in environment variables or codebase.
  * **Non-Root User**: The application runs as a non-root `appuser` within the container, limiting potential damage in case of a security breach.
  * **JWT Protection**: All critical endpoints (`/analyze-food`) are protected by JWT verification (`token_required` decorator), ensuring that only authenticated users can access the service.
  * **Rate Limiting (External)**: The API Gateway applies a standard rate limit to the `/image-service/*` route to protect against abuse and control costs for external API calls (e.g., OpenAI).
  * **Image Pre-processing**: Images are resized before sending to OpenAI, which can help mitigate potential resource exhaustion or excessively large API requests.
  * **Input Validation**: Flask's request handling and internal logic ensure that only valid image files are processed.
  * **API Key Protection**: OpenAI API key is handled securely and not exposed to the client.
  * **Timeouts**: Requests to external APIs (OpenAI, User Service) include timeouts to prevent hanging connections and improve resilience.
  * **Retry Mechanism**: The `tenacity` library adds robustness to OpenAI API calls, automatically retrying transient failures with exponential backoff.

-----

## 7\. Project Structure

  * **`app.py`**: The main Flask application file, containing:
      * Flask app initialization and CORS.
      * Configuration loading (Redis, OpenAI, JWT).
      * Redis client setup with robust connection error handling.
      * `token_required` decorator for JWT validation.
      * Image processing logic (`resize_image_for_openai`).
      * OpenAI integration (`analyze_food_image` with retries).
      * Redis caching functions (`save_to_cache`, `get_from_cache`).
      * User Service integration (`save_food_entry`).
      * `/analyze-food` and `/health` endpoints.
  * **`docker-entrypoint.sh`**: A critical startup script for:
      * Waiting for Docker secrets to be mounted.
      * Exporting secrets as environment variables.
      * Waiting for Redis to be available.
      * Executing the main `gunicorn` command.
  * **`Dockerfile`**: Defines the Docker image, including system dependencies for image processing, Python dependencies, user setup, and entrypoint configuration.
  * **`requirements.txt`**: Lists all Python dependencies.
  * **`secrets/`**: (Conceptual directory for Docker Compose) Contains sensitive files like `redis_password.txt`, `openai_api_key.txt`, `jwt_secret.txt` which are mounted as Docker secrets.

-----

## 8\. Monitoring and Troubleshooting

  * **Logs**: The service logs informational messages and errors to standard output/error. Integrate with a centralized logging solution to collect and analyze these logs.
      * Look for `Error analyzing food image` or `OpenAI API request failed` for issues with AI processing.
      * Check `Redis connection error` for caching problems.
      * Monitor `Cache hit` and `Cache miss` logs to understand caching efficiency.
      * Review `Error saving food entry to user service` for issues with saving data.
  * **Health Check (`/health`)**: Regularly poll this endpoint from your orchestrator to ensure the service is alive and its critical dependencies (Redis, OpenAI API key) are accessible. The `startup_grace_period` helps prevent premature restarts.
  * **OpenAI Dashboard**: Monitor your OpenAI API usage and error rates directly from your OpenAI dashboard to identify potential issues or quota limits.
  * **Redis Monitoring**: Monitor your Redis instance's memory usage and key activity for the `food_analysis:` prefix to ensure caching is working effectively.
  * **Network Connectivity**: Verify connectivity between the Image Service container and Redis, OpenAI API endpoints, and the User Service.
  * **Gunicorn Workers**: Monitor Gunicorn worker processes. If `workers` or `threads` are too low, requests might queue up; if too high, it might exhaust resources. Adjust based on load and available resources. The `timeout` setting prevents workers from hanging indefinitely.

