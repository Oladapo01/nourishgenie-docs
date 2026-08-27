# Image Service

This repository contains the Image Service, a vital microservice for the NourishGenie application backend. Its primary function is to leverage AI for analyzing food images, extracting nutritional information, and providing meal suggestions. It integrates with a commercial vision-language AI API, uses Redis for caching analysis results, and interacts with the User Service to save food entries.

-----

## 1\. Overview

The Image Service acts as the intelligent processing unit for user-uploaded food photos. When a user uploads an image, this service takes it, processes it, sends it to an external AI model for analysis, caches the results, and can then record the analysis as a food entry linked to the user. This ensures efficient, scalable, and AI-powered food recognition within the application.

-----

## 2\. Features

  * **AI-Powered Food Analysis**: Integrates with **a state-of-the-art multimodal vision-language model** to interpret food images and extract detailed nutritional information (dish name, calories, macros, portion size, ingredients).
  * **Ingredient Detection, Recipe Generation, and Macro Suggestions**: Beyond single-image food analysis, the service also exposes endpoints for detecting ingredients in an image, generating recipes, and suggesting macro breakdowns — all backed by the same vision AI integration and subject to the same subscription gate described below.
  * **Server-Side Subscription Enforcement**: The four endpoints that spend AI-provider credit (`/analyze-food`, `/analyze-ingredients`, `/generate-recipes`, `/macro-suggestions`) are gated server-side on the caller's subscription/trial status, rather than relying on the client to hide the feature. The gate runs before the cache lookup, since the analysis cache is keyed on image hash rather than user — letting a cache hit through would give a lapsed subscriber free access to any previously-analyzed image.
  * **Non-Food Rejection**: Images classified as not food are rejected with a distinct response rather than a generic analysis, with a per-user strike counter to limit repeated non-food submissions (a non-food image still costs one vision AI call to classify, so unlimited free classification would be a cost hole).
  * **Image Pre-processing**: Resizes and converts images to optimize them for AI model input.
  * **Intelligent Caching**: Utilizes **Redis** to cache image analysis results based on image hashes, significantly reducing redundant AI API calls and improving response times for duplicate images.
  * **Food Entry Persistence**: Optionally saves the analyzed food data as a new food entry in the **User Service** for the authenticated user.
  * **Robust Error Handling**: Implements retries for vision AI API calls using `tenacity` to handle transient network issues or API rate limits.
  * **Authentication & Authorization**: Protects its endpoints using **JWT verification**, ensuring only authenticated users can submit images for analysis. The one endpoint intended for internal service-to-service use accepts either a valid user JWT or a shared internal secret, rather than being open to any caller.
  * **Upload Validation**: Enforces a maximum request size and validates that uploaded content is actually a well-formed image before it reaches the AI pipeline, rejecting oversized or malformed uploads early.
  * **Health Check**: Provides a `/health` endpoint with checks for Redis and vision AI API key configuration, indicating service readiness.
  * **Secure Configuration**: Loads sensitive API keys and passwords from Docker secrets.
  * **Containerized Deployment**: Designed for easy deployment with Docker, including a custom entrypoint script for robust secret loading and service readiness checks.

-----

## 3\. Technology Stack

  * **Python 3.11**: Primary programming language.
  * **Flask**: Lightweight web framework for handling HTTP requests.
  * **Gunicorn**: WSGI HTTP server for running the Flask application in production.
  * **Pillow (PIL)**: For image manipulation (resizing, format conversion).
  * **Requests**: For making HTTP requests to the vision AI provider and User Service.
  * **Redis**: In-memory data store for caching and potentially for session/token management.
      * `redis-py`: Python client for Redis.
  * **`PyJWT`**: For JSON Web Token (JWT) verification.
  * **`tenacity`**: For adding retry logic to external API calls.
  * **Docker**: For containerization and deployment.
  * **Vision-Language AI Provider**: A commercial multimodal vision-language model, used for interpreting food images (kept vendor-neutral in this document).

-----

## 4\. Getting Started

These instructions will guide you through setting up and running the Image Service.

### 4.1. Prerequisites

  * Python 3.11+
  * `pip`
  * Docker and Docker Compose (recommended for local development)
  * Redis instance (local or remote)
  * the vision AI provider API Key (requires access to the vision-language model)
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

    # VISION_API_KEY=your_vision_api_key # Use a secrets file in production
    VISION_MODEL=vision-model-v1
    VISION_API_TIMEOUT=30
    CACHE_TTL_SECONDS=3600

    # USER_SERVICE_URL points to the User Service. 
    # If running User Service locally, use its exposed port.
    USER_SERVICE_URL=http://localhost:8082

    # These would typically be managed via Docker secrets or Kubernetes secrets.
    # For local testing without Docker, you'd need to create them as local files.
    # Example:
    # mkdir -p /run/secrets # Create this directory
    # echo "your_strong_redis_password" > /run/secrets/redis_password
    # echo "your_vision_api_key" > /run/secrets/vision_api_key
    # echo "your_super_secret_jwt_key" > /run/secrets/jwt_secret
    ```

    **Important Note on Secrets:** In production, sensitive values like `REDIS_PASSWORD`, `VISION_API_KEY`, and `JWT_SECRET` **must** be managed using secure mechanisms like Docker Secrets (as implemented in the `Dockerfile` and `docker-entrypoint.sh`) or Kubernetes Secrets. **Never** hardcode secrets or expose them directly in `.env` files that are committed to version control.

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
          VISION_MODEL: vision-model-v1
          VISION_API_TIMEOUT: 60 # Increased timeout for potentially longer AI responses
          CACHE_TTL_SECONDS: 3600
        secrets:
          - redis_password
          - vision_api_key
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
      vision_api_key:
        file: ./secrets/vision_api_key.txt # Create this file with your vision AI API key
      jwt_secret:
        file: ./secrets/jwt_secret.txt # Create this file with a strong, random JWT secret
    ```

    To run this `docker-compose.yml`:

    1.  Create a `secrets` directory in your root project folder.
    2.  Inside `secrets`, create `redis_password.txt`, `vision_api_key.txt`, and `jwt_secret.txt` and fill them with your actual secrets.
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

  * **Description**: Analyzes an uploaded food image using a state-of-the-art multimodal vision-language model to extract nutritional information. It checks a Redis cache first and saves results to cache. Optionally, it can save the analysis as a food entry to the User Service.
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
  * **Errors**: `400 Bad Request` (No image file), `401 Unauthorized` (Invalid/missing token), `403 Forbidden` (`subscription_required` — caller is neither subscribed nor in trial), `413 Payload Too Large` (upload exceeds the configured size limit), `422 Unprocessable Entity` (`not_food` — image classified as non-food), `429 Too Many Requests` (`too_many_non_food_uploads` — non-food strike limit reached), `500 Internal Server Error` (vision AI API errors, image processing errors, User Service errors).
  * **Note**: The subscription check runs before the cache lookup and before the vision AI call, and fails open (allows the request) if the subscription service is unreachable, rather than blocking food logging during an unrelated outage.

#### `POST /analyze-ingredients`

  * **Description**: Detects the ingredients present in an uploaded food image via the vision AI provider, independent of full nutritional analysis. Subject to the same subscription gate, upload validation, and caching pattern as `/analyze-food`.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Content**: `multipart/form-data` with an `image` field.
  * **Errors**: Same error set as `/analyze-food`.

#### `POST /generate-recipes`

  * **Description**: Generates recipe suggestions via the vision AI provider, streamed as Server-Sent Events. Subject to the same subscription gate as `/analyze-food`.
  * **Authorization**: Accepts either a valid **Access Token** or an internal-service secret (`X-Internal-Secret` header, constant-time compared) — a dual-accept design rather than opening the endpoint to unauthenticated callers.
  * **Errors**: `401 Unauthorized` (neither a valid token nor a valid internal secret), `403 Forbidden` (`subscription_required`, for user-token callers).

#### `POST /macro-suggestions`

  * **Description**: Suggests a macro breakdown via the vision AI provider. Subject to the same subscription gate, upload validation, and caching pattern as `/analyze-food`.
  * **Authorization**: Requires a valid **Access Token**.
  * **Errors**: Same error set as `/analyze-food`.

### 5.2. Health Check

#### `GET /health`

  * **Description**: Endpoint for readiness and liveness probes. Reports the health status of Redis connection and vision AI API key configuration.
  * **Response (200 OK or 503 Service Unavailable)**:
    ```json
    {
      "status": "healthy", // or "unhealthy"
      "service": "image-service",
      "startup_mode": false, // or true during initial startup_period
      "checks": {
        "redis": "connected", // or "connection failed", "not configured", "error: ..."
        "vision_ai": "configured" // or "missing credentials"
      }
    }
    ```

-----

## 6\. Security Considerations

  * **Secrets Management**: `VISION_API_KEY`, `REDIS_PASSWORD`, and `JWT_SECRET` are securely loaded from Docker secrets at container startup using the `docker-entrypoint.sh` script. This prevents sensitive information from being exposed in environment variables or codebase.
  * **Non-Root User**: The application runs as a non-root `appuser` within the container, limiting potential damage in case of a security breach.
  * **JWT Protection**: Endpoints that serve individual users (`/analyze-food`, `/analyze-ingredients`, `/macro-suggestions`) are protected by JWT verification (`token_required` decorator), ensuring that only authenticated users can access the service.
  * **Resolved: Unauthenticated Streaming Endpoint**: `/generate-recipes` was previously reachable without any authentication — a publicly-accessible endpoint that streamed the model's output on request, at the service's AI-provider expense, despite its own docstring describing it as internal-only. It now requires either a valid user JWT or a shared internal secret, compared using a constant-time comparison to avoid a timing side-channel.
  * **Server-Side Subscription Enforcement**: Client-side paywalls can be bypassed by any client that skips the check, so entitlement (`isSubscribed OR isInTrialPeriod`) is verified server-side on every the vision AI provider-spending request, cached briefly (90s) with a longer stale-value fallback (24h) so a subscription-service blip doesn't lock out paying users. The gate deliberately runs before the cache lookup — the analysis cache is a global, image-hash-keyed cache with no per-user scoping, so a cache hit is still paid-tier output and must be gated the same as a fresh vision AI call.
  * **Rate Limiting (External)**: The API Gateway applies a standard rate limit to the `/image-service/*` route to protect against abuse and control costs for external API calls (e.g., the vision AI provider).
  * **Upload Size and Content Validation**: Requests are rejected with `413` before the body is buffered if the declared `Content-Length` exceeds the configured limit, and uploaded files are validated as well-formed images (not just checked by filename or declared content type) before being sent to the vision AI provider.
  * **Non-Food Abuse Control**: An image classified as non-food still costs one vision AI call to classify, so repeated non-food submissions from the same user are capped via a Redis-backed strike counter (failing open — i.e. not blocking the request — if Redis is unavailable, since availability of the core feature takes priority over this specific abuse control).
  * **Image Pre-processing**: Images are resized before sending to the vision AI provider, which can help mitigate potential resource exhaustion or excessively large API requests.
  * **API Key Protection**: vision AI API key is handled securely and not exposed to the client.
  * **Timeouts**: Requests to external APIs (the vision AI provider, User Service) include timeouts to prevent hanging connections and improve resilience.
  * **Retry Mechanism**: The `tenacity` library adds robustness to vision AI API calls, automatically retrying transient failures with exponential backoff.
  * **Error Message Hardening**: Endpoints were audited to stop returning raw exception text (`str(e)`) in API responses, logging the full exception server-side (`exc_info=True`) while returning a generic message to the client.

-----

## 7\. Project Structure

  * **`app.py`**: The main Flask application file, containing:
      * Flask app initialization and CORS.
      * Configuration loading (Redis, vision AI provider, JWT).
      * Redis client setup with robust connection error handling.
      * `token_required` decorator for JWT validation.
      * Image processing logic (`resize_image_for_vision_api`).
      * vision AI integration (`analyze_food_image` with retries).
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
  * **`secrets/`**: (Conceptual directory for Docker Compose) Contains sensitive files like `redis_password.txt`, `vision_api_key.txt`, `jwt_secret.txt` which are mounted as Docker secrets.

-----

## 8\. Monitoring and Troubleshooting

  * **Logs**: The service logs informational messages and errors to standard output/error. Integrate with a centralized logging solution to collect and analyze these logs.
      * Look for `Error analyzing food image` or `Vision API request failed` for issues with AI processing.
      * Check `Redis connection error` for caching problems.
      * Monitor `Cache hit` and `Cache miss` logs to understand caching efficiency.
      * Review `Error saving food entry to user service` for issues with saving data.
  * **Health Check (`/health`)**: Regularly poll this endpoint from your orchestrator to ensure the service is alive and its critical dependencies (Redis, vision AI API key) are accessible. The `startup_grace_period` helps prevent premature restarts.
  * **AI Provider Dashboard**: Monitor your AI provider API usage and error rates directly from your AI provider's dashboard to identify potential issues or quota limits.
  * **Redis Monitoring**: Monitor your Redis instance's memory usage and key activity for the `food_analysis:` prefix to ensure caching is working effectively.
  * **Network Connectivity**: Verify connectivity between the Image Service container and Redis, vision AI API endpoints, and the User Service.
  * **Gunicorn Workers**: Monitor Gunicorn worker processes. If `workers` or `threads` are too low, requests might queue up; if too high, it might exhaust resources. Adjust based on load and available resources. The `timeout` setting prevents workers from hanging indefinitely.

-----

## 9\. Production Hardening & Incident History

The items below are resolved production issues, kept here as a record of the reasoning behind current behavior rather than as a changelog of code diffs.

  * **Nutrition Data Corruption (Q&A Follow-Up Path)**: A conditional guard intended to trigger a text-based nutrition retry only when an item's calorie value was genuinely zero was mis-indented, so the retry ran unconditionally for every item in the Q&A follow-up path. The retry's result then overwrote the item unconditionally — meaning vision-derived calorie and macro values (the core output of the AI analysis) were being silently replaced with a text-only estimate that never saw the image, for every item, on every Q&A-path request. The equivalent code in the non-Q&A path had the same guard correctly nested, which is what surfaced the discrepancy. Fixed by aligning the Q&A path with the already-correct non-Q&A path.
  * **Unauthenticated Streaming Endpoint (Critical)**: `/generate-recipes` had no authentication despite its own docstring describing it as internal-only, making it a publicly reachable endpoint that would stream the model's output at the service's expense for any caller. Closed with dual-accept authentication (user JWT or internal secret).
  * **Duplicated Vision AI Call**: `/analyze-ingredients` called the vision AI provider's ingredient-detection function twice per request, discarding the first result — doubling both latency and AI-provider spend on every request to that endpoint. Removed the redundant call.
  * **Cache Key Mismatch**: The same endpoint's cache write used a different key than its cache read, so every request was a cache miss regardless of whether the image had been analyzed before — silently defeating the caching layer's purpose. Fixed so the write uses the same key the read checks.
  * **Cache-Hit NameError**: On the cache-hit path, a variable used in the response was only ever assigned on the cache-miss path, so a cache hit raised an unbound-variable error and returned a 500 instead of the cached result. Fixed.
  * **SSE Headers Set After Return**: The code setting Server-Sent Event headers (including `Connection: keep-alive`) sat after a `return` statement and was therefore unreachable, so the streaming response was missing headers that affect client-side buffering behavior. Restructured so the headers are actually applied.
  * **Dead Duplicate Route**: `/submit-correction` was registered twice under different endpoint names — one forwarding corrections to the User Service, the other storing them locally for a planned custom-model training pipeline. Flask/Werkzeug silently resolves duplicate routes to whichever was registered first, so the local-storage handler never ran, and its backing table had not been receiving data despite the endpoint appearing to work end-to-end from the client's perspective.

-----

*Created: July 4, 2025*
*Last Updated: August 27, 2026*
