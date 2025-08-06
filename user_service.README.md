# User Service

This repository contains the User Service, a core microservice for the Food Tracker application backend. It is responsible for managing user-specific data, including profiles, preferences, food entries, health metrics, and achievements (badges). It integrates with PostgreSQL for persistent data, MinIO for object storage (images), and interacts with the Auth Service for user authentication.

-----

## 1\. Overview

The User Service acts as the central hub for all user-centric data within the Food Tracker application. It stores detailed user profiles, tracks their food consumption, calculates nutrition summaries, and manages their onboarding data and progress. By centralizing this data, the service provides a comprehensive view of a user's food tracking journey and health goals. It also manages storage for user-related images like profile pictures.

-----

## 2\. Features

  * **User Profile Management**: Stores and retrieves detailed user profiles, including name, email, dietary restrictions, and notification preferences.
  * **Onboarding Data**: Manages and processes initial user health and fitness data (age, gender, height, weight, goals, activity level) to calculate personalized nutrition goals.
  * **Food Entry Tracking**: Allows users to add, retrieve, and delete daily food entries with detailed nutritional information.
  * **Nutrition Summary & Goals**: Provides aggregated nutrition summaries for specific date ranges and allows users to manage their personalized daily nutrition goals.
  * **Achievement System (Badges)**: Awards users badges based on their activity (e.g., total entries, streaks), promoting engagement.
  * **MinIO Object Storage Integration**: Manages storage and retrieval of user-related images (e.g., profile pictures, food photos) in MinIO buckets.
  * **Inter-Service Communication**:
      * **Auth Service Integration**: Syncs basic user data from the Auth Service during initial login, and forwards password change and account deletion requests to the Auth Service.
      * **Image Service Integration**: Designed to receive food entry data from the Image Service after AI analysis.
  * **Authentication & Authorization**: Protects all endpoints using **JWT verification**, ensuring only authenticated users can access or modify their data.
  * **Database Persistence**: Stores all user-related structured data (profiles, entries, badges) in **PostgreSQL**.
  * **Health Check**: Provides a `/health` endpoint to monitor the status of database and MinIO connectivity.
  * **Secure Configuration**: Loads sensitive credentials (database password, MinIO keys, JWT secret) from Docker secrets.
  * **Containerized Deployment**: Designed for easy deployment with Docker, including an `init_db.py` script for automated database schema setup.

-----

## 3\. Technology Stack

  * **Python 3.11**: Primary programming language.
  * **Flask**: Lightweight web framework for building APIs.
  * **Gunicorn**: WSGI HTTP server for running the Flask application in production.
  * **PostgreSQL**: Relational database for persistent structured data.
      * `psycopg2`: PostgreSQL adapter for Python.
  * **MinIO**: High-performance, S3 compatible object storage for images.
      * `minio`: Python client for MinIO.
  * **`PyJWT`**: For JSON Web Token (JWT) verification.
  * **`requests`**: For making HTTP requests to other microservices (e.g., Auth Service).
  * **`Pillow (PIL)`**: (Though not directly used in `app.py` for image processing here, its presence suggests potential future use or a dependency of `minio` in some setups, but actual image processing is handled by Image Service).
  * **Docker**: For containerization and deployment.

-----

## 4\. Getting Started

These instructions will guide you through setting up and running the User Service.

### 4.1. Prerequisites

  * Python 3.11+
  * `pip`
  * Docker and Docker Compose (recommended for local development)
  * PostgreSQL instance (local or remote)
  * MinIO instance (local or remote)
  * Auth Service (running and accessible if you want to test full authentication and user syncing)

### 4.2. Local Setup (without Docker - for development purposes)

Running the User Service directly without Docker is possible for development but requires manual setup of environment variables and creating dummy secret files.

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/Oladapo01/FoodTrackerApp.git/user-service
    cd user-service
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
    POSTGRES_HOST=localhost
    POSTGRES_PORT=5432
    POSTGRES_DB=userdb
    POSTGRES_USER=postgres
    # POSTGRES_PASSWORD=your_db_password # Use a secrets file in production

    MINIO_HOST=localhost
    MINIO_PORT=9000
    # MINIO_ACCESS_KEY=your_minio_access_key # Use a secrets file in production
    # MINIO_SECRET_KEY=your_minio_secret_key # Use a secrets file in production

    # AUTH_SERVICE_URL=http://localhost:8081 # Points to your local Auth Service

    # These would typically be managed via Docker secrets or Kubernetes secrets.
    # For local testing without Docker, you'd need to create them as local files.
    # Example:
    # mkdir -p /run/secrets # Create this directory
    # echo "your_strong_db_password" > /run/secrets/db_password
    # echo "your_minio_access_key" > /run/secrets/minio_access_key
    # echo "your_minio_secret_key" > /run/secrets/minio_secret_key
    # echo "your_super_secret_jwt_key" > /run/secrets/jwt_secret
    ```

    **Important Note on Secrets:** In production, sensitive values like database passwords, MinIO keys, and JWT secrets **must** be managed using secure mechanisms like Docker Secrets (as implemented in the `Dockerfile` and `app.py`) or Kubernetes Secrets. **Never** hardcode secrets or expose them directly in `.env` files that are committed to version control.

5.  **Initialize the database schema:**

    ```bash
    python init_db.py
    ```

    Ensure your PostgreSQL instance is running and accessible before running this.

6.  **Run the application:**

    ```bash
    gunicorn --bind 0.0.0.0:8082 --workers 1 --threads 2 app:app
    ```

    (You might use `flask run` directly for simpler development, but Gunicorn is closer to the production setup).

### 4.4. Docker Deployment

The `Dockerfile` provides a production-ready image for the User Service. It installs necessary system dependencies (`libpq-dev`), Python packages, sets up a non-root user, and defines the startup command, which first runs the database initialization script.

1.  **Build the Docker image:**

    ```bash
    docker build -t your-repo/user-service:latest .
    ```

    Replace `your-repo/user-service` with your desired image name.

2.  **Run the Docker container (example with Docker Compose):**

    It's highly recommended to use **Docker Compose** for local development and orchestration with other services (API Gateway, PostgreSQL, MinIO, Auth Service). A `docker-compose.yml` snippet for the User Service would look like this:

    ```yaml
    version: '3.8'
    services:
      user-service:
        build:
          context: ./user-service # Assuming user-service is a sub-directory
          dockerfile: Dockerfile
        ports:
          - "8082:8082" # Exposed internally, API Gateway handles external access
        environment:
          POSTGRES_HOST: postgres
          POSTGRES_PORT: 5432
          POSTGRES_DB: userdb
          POSTGRES_USER: postgres
          MINIO_HOST: minio
          MINIO_PORT: 9000
          AUTH_SERVICE_URL: http://auth-service:8081 # Name of Auth service in compose network
        secrets:
          - db_password
          - minio_access_key
          - minio_secret_key
          - jwt_secret
        depends_on:
          postgres:
            condition: service_healthy
          minio:
            condition: service_healthy
          auth-service: # For user syncing and password/account deletion forwarding
            condition: service_healthy
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:8082/health"]
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 30s # Allow time for init_db.py to run and MinIO bucket creation
      postgres:
        image: postgres:13
        environment:
          POSTGRES_DB: userdb
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD_FILE: /run/secrets/db_password
        secrets:
          - db_password
        volumes:
          - user_db_data:/var/lib/postgresql/data
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres -d userdb"]
          interval: 10s
          timeout: 5s
          retries: 5
      minio:
        image: minio/minio
        command: server /data --console-address ":9001"
        environment:
          MINIO_ROOT_USER_FILE: /run/secrets/minio_access_key
          MINIO_ROOT_PASSWORD_FILE: /run/secrets/minio_secret_key
        secrets:
          - minio_access_key
          - minio_secret_key
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"] # Or custom MinIO API check
          interval: 10s
          timeout: 5s
          retries: 5
      auth-service:
        # ... auth-service configuration here
    secrets:
      db_password:
        file: ./secrets/db_password.txt
      minio_access_key:
        file: ./secrets/minio_access_key.txt
      minio_secret_key:
        file: ./secrets/minio_secret_key.txt
      jwt_secret:
        file: ./secrets/jwt_secret.txt
    volumes:
      user_db_data:
    ```

    To run this `docker-compose.yml`:

    1.  Create a `secrets` directory in your root project folder.
    2.  Inside `secrets`, create `db_password.txt`, `minio_access_key.txt`, `minio_secret_key.txt`, and `jwt_secret.txt` and fill them with strong, random passwords/secrets.
    3.  From the directory containing your `docker-compose.yml`, run:
        ```bash
        docker compose up --build
        ```

### 4.5. `init_db.py` Explained

The `init_db.py` script is executed as part of the container's startup command (`CMD python init_db.py && ...`). Its responsibilities include:

  * **Waiting for PostgreSQL**: It includes a robust retry mechanism (`wait_for_postgres`) to ensure the PostgreSQL database is fully ready and accessible before attempting to connect and initialize the schema. This is crucial in containerized environments.
  * **Database Schema Creation**: It checks for and creates all necessary tables and indexes for the User Service if they don't already exist. This ensures that the database is correctly set up on the first run of the service.
  * **MinIO Bucket Initialization (from `app.py`)**: The `app.py`'s `init_app` function (called at global scope) ensures that the necessary MinIO buckets (`profile-pictures`, `food-images`) are created if they don't already exist when the application starts.

-----

## 5\. API Endpoints

The User Service exposes the following RESTful API endpoints:

### 5.1. User Profile and Onboarding

#### `GET /profile`

  * **Description**: Retrieves the comprehensive profile of the authenticated user, including personal data, preferences, onboarding data, general stats (total entries, calories, tracking days), and earned badges.
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Response (200 OK)**:
    ```json
    {
      "userId": "uuid-of-user",
      "name": "User Name",
      "email": "user@example.com",
      "preferences": {
        "dietaryRestrictions": ["Vegetarian", "Gluten-Free"],
        "nutritionGoals": { "calories": 2000, "protein": 100, "carbs": 250, "fat": 70 },
        "notifications": true
      },
      "onboardingData": {
        "age": 30,
        "gender": "female",
        "height": 165,
        "currentWeight": 60,
        "targetWeight": 58,
        "goal": "lose_weight",
        "activityLevel": "moderately_active"
      },
      "stats": {
        "totalEntries": 150,
        "totalCalories": 250000,
        "trackingDays": 60
      },
      "badges": [
        { "id": "week_streak", "name": "Week Streak Champion", "description": "Logged food entries for 7 consecutive days", "earnedAt": "2024-06-01T10:00:00Z" }
      ]
    }
    ```
  * **Errors**: `401 Unauthorized`, `404 Not Found` (User not found after sync attempt), `500 Internal Server Error`.

#### `PUT /profile`

  * **Description**: Updates the authenticated user's name and/or preferences (dietary restrictions, nutrition goals, notification settings).
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**:
    ```json
    {
      "name": "Updated User Name",
      "preferences": {
        "dietaryRestrictions": ["Vegan"],
        "nutritionGoals": { "calories": 1800, "protein": 90, "carbs": 200, "fat": 60 },
        "notifications": false
      }
    }
    ```
  * **Response (200 OK)**: Returns the updated basic user profile and preferences.
  * **Errors**: `400 Bad Request` (No data provided), `401 Unauthorized`, `500 Internal Server Error`.

#### `POST /onboarding`

  * **Description**: Saves initial onboarding data for a user (age, gender, height, weight, fitness goal, activity level). Based on this data, it calculates initial daily nutrition goals (calories, protein, carbs, fat) and stores them in user preferences.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**:
    ```json
    {
      "age": 30,
      "gender": "male",
      "height": 175,
      "currentWeight": 70,
      "targetWeight": 72,
      "goal": "gain_weight",
      "activityLevel": "moderately_active"
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "message": "Onboarding data saved successfully",
      "nutritionGoals": { "calories": 2500, "protein": 126, "carbs": 300, "fat": 70 }
    }
    ```
  * **Errors**: `400 Bad Request` (Missing fields, invalid data for calculation), `401 Unauthorized`, `500 Internal Server Error`.

### 5.2. Food Tracking

#### `POST /food-entries`

  * **Description**: Adds a new food entry for the authenticated user. Automatically checks and awards relevant badges (e.g., total entries, streaks).
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**:
    ```json
    {
      "name": "Grilled Chicken Salad",
      "calories": 350,
      "mealType": "lunch",
      "servingSize": "1 bowl",
      "protein": 30,
      "carbs": 20,
      "fat": 15,
      "notes": "With vinaigrette dressing"
    }
    ```
  * **Response (201 Created)**:
    ```json
    {
      "id": "uuid-of-entry",
      "userId": "uuid-of-user",
      "name": "Grilled Chicken Salad",
      "calories": 350,
      "mealType": "lunch",
      "servingSize": "1 bowl",
      "protein": 30,
      "carbs": 20,
      "fat": 15,
      "notes": "With vinaigrette dressing",
      "createdAt": "2024-07-11T10:30:00.000Z"
    }
    ```
  * **Errors**: `400 Bad Request` (Missing required fields), `401 Unauthorized`, `500 Internal Server Error`.

#### `GET /food-entries`

  * **Description**: Retrieves food entries for the authenticated user. Supports filtering by specific date, date range, and meal type.
  * **Authorization**: Requires a valid **Access Token**.
  * **Query Parameters**:
      * `date`: (Optional) `YYYY-MM-DD` or ISO format (`2025-05-28T23:00:00Z`) to get entries for a specific day.
      * `startDate`: (Optional) `YYYY-MM-DD` or ISO format for the beginning of a date range.
      * `endDate`: (Optional) `YYYY-MM-DD` or ISO format for the end of a date range.
      * `mealType`: (Optional) e.g., `breakfast`, `lunch`, `dinner`, `snack`.
  * **Response (200 OK)**:
    ```json
    [
      {
        "id": "uuid-of-entry-1",
        "userId": "uuid-of-user",
        "name": "Oatmeal",
        "calories": 200,
        "mealType": "breakfast",
        "timestamp": "2024-07-10T08:00:00Z"
      },
      {
        "id": "uuid-of-entry-2",
        "userId": "uuid-of-user",
        "name": "Chicken Sandwich",
        "calories": 450,
        "mealType": "lunch",
        "timestamp": "2024-07-10T13:00:00Z"
      }
    ]
    ```
  * **Errors**: `400 Bad Request` (Invalid date format), `401 Unauthorized`, `500 Internal Server Error`.

#### `DELETE /food-entries/<entry_id>`

  * **Description**: Deletes a specific food entry for the authenticated user.
  * **Authorization**: Requires a valid **Access Token**.
  * **URL Parameters**: `entry_id` (UUID of the food entry to delete).
  * **Response (200 OK)**:
    ```json
    {
      "message": "Food entry deleted successfully"
    }
    ```
  * **Errors**: `401 Unauthorized`, `404 Not Found` (Entry not found or does not belong to user), `500 Internal Server Error`.

### 5.3. Nutrition Summary and Goals

#### `PUT /nutrition-goals`

  * **Description**: Directly updates a user's daily nutrition goals (calories, protein, carbs, fat). This overrides any goals derived from onboarding data.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**:
    ```json
    {
      "calories": 2200,
      "protein": 110,
      "carbs": 275,
      "fat": 75
    }
    ```
  * **Response (200 OK)**:
    ```json
    {
      "message": "Nutrition goals updated successfully"
    }
    ```
  * **Errors**: `400 Bad Request` (Missing fields, out-of-range values), `401 Unauthorized`, `500 Internal Server Error`.

#### `GET /nutrition-summary`

  * **Description**: Provides a daily nutrition summary for a specified date range, including consumed and target macros/calories.
  * **Authorization**: Requires a valid **Access Token**.
  * **Query Parameters**:
      * `startDate`: `YYYY-MM-DD` or ISO format for the start of the summary period.
      * `endDate`: `YYYY-MM-DD` or ISO format for the end of the summary period.
  * **Response (200 OK)**: (Currently returns a single daily summary for the `startDate` provided, if `startDate` and `endDate` are the same day. For a range, it would typically be an array of daily summaries.)
    ```json
    [
      {
        "caloriesConsumed": 1800,
        "caloriesBurned": 0,
        "caloriesGoal": 2000,
        "proteinConsumed": 85,
        "proteinGoal": 100,
        "carbsConsumed": 220,
        "carbsGoal": 250,
        "fatConsumed": 50,
        "fatGoal": 70,
        "date": "2024-07-10"
      }
    ]
    ```
  * **Errors**: `400 Bad Request` (Missing or invalid date parameters), `401 Unauthorized`, `500 Internal Server Error`.

#### `GET /dashboard/summary`

  * **Description**: Provides a real-time summary for the *current day*, including calories consumed, remaining, burned (mocked to 0), and consumed/target macros. This is tailored for a dashboard view.
  * **Authorization**: Requires a valid **Access Token**.
  * **Response (200 OK)**:
    ```json
    {
      "caloriesConsumed": 1200,
      "caloriesRemaining": 800,
      "caloriesBurned": 0,
      "targetCalories": 2000,
      "proteinConsumed": 60,
      "targetProtein": 100,
      "carbsConsumed": 150,
      "targetCarbs": 250,
      "fatConsumed": 35,
      "targetFat": 70,
      "foodEntriesCount": 3
    }
    ```
  * **Errors**: `401 Unauthorized`, `500 Internal Server Error`.

### 5.4. Account Actions (Forwarded to Auth Service)

These endpoints primarily act as proxies, forwarding requests to the Auth Service to centralize user authentication and account deletion logic.

#### `PUT /change-password`

  * **Description**: Forwards the password change request to the Auth Service.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**: `{"currentPassword": "...", "newPassword": "..."}`
  * **Response**: Proxies response from Auth Service.
  * **Errors**: `401 Unauthorized`, `400 Bad Request` (Auth Service errors), `503 Service Unavailable` (Auth Service unreachable), `500 Internal Server Error`.

#### `DELETE /account`

  * **Description**: Forwards the account deletion request to the Auth Service. If Auth Service confirms deletion, it also deletes the user's data from the User Service's database.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**: `{"password": "...", "confirmationText": "delete my account"}`
  * **Response**: Proxies response from Auth Service if deletion fails there, otherwise `200 OK` from User Service.
  * **Errors**: `401 Unauthorized`, `400 Bad Request` (Auth Service errors), `503 Service Unavailable` (Auth Service unreachable), `500 Internal Server Error`.

#### `POST /logout`

  * **Description**: Forwards the logout request to the Auth Service to blacklist the token.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**: `{}`
  * **Response (200 OK)**: Always returns success to the client, even if Auth Service call fails, to ensure client-side logout.
  * **Errors**: `401 Unauthorized`, `500 Internal Server Error` (backend error, but client still gets success).

### 5.5. MinIO Image Serving

#### `GET /images/<bucket>/<filename>`

  * **Description**: Serves image files directly from specified MinIO buckets. **Note**: In a production environment, it's often more efficient to serve static files directly from a dedicated CDN or object storage proxy (like Nginx) rather than routing through the application. This endpoint is primarily for simple development or scenarios where application-level access control for images is strictly required.
  * **URL Parameters**:
      * `bucket`: The MinIO bucket name (e.g., `profile-pictures`, `food-images`).
      * `filename`: The name of the image file.
  * **Response (200 OK)**: Image file with appropriate `Content-Type`.
  * **Errors**: `404 Not Found` (Image or bucket not found), `500 Internal Server Error`.

### 5.6. Health Check

#### `GET /health`

  * **Description**: Endpoint for readiness and liveness probes. Reports the connectivity status of PostgreSQL and MinIO.
  * **Response (200 OK or 503 Service Unavailable)**:
    ```json
    {
      "status": "healthy", // or "unhealthy"
      "service": "user-service",
      "checks": {
        "database": "connected", // or "disconnected"
        "minio": "connected"     // or "disconnected", "error: ..."
      }
    }
    ```

-----

## 6\. Database Schema

The `init_db.py` script ensures the following tables are created and managed by the User Service in PostgreSQL:

  * **`users`**: Core user data.
      * `id` (`VARCHAR(36) PRIMARY KEY`) - Synced from Auth Service.
      * `email` (`VARCHAR(255) UNIQUE NOT NULL`) - Synced from Auth Service.
      * `name` (`VARCHAR(255)`) - Synced from Auth Service.
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
  * **`user_preferences`**: User-specific settings and calculated goals.
      * `user_id` (`VARCHAR(36) PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE`)
      * `dietary_restrictions` (`TEXT[]`) - Array of strings (e.g., 'Vegetarian', 'Vegan').
      * `nutrition_goals` (`JSONB`) - JSON object (e.g., `{"calories": 2000, "protein": 100}`).
      * `notifications_enabled` (`BOOLEAN DEFAULT TRUE`)
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `updated_at` (`TIMESTAMP WITH TIME ZONE`)
  * **`user_onboarding_data`**: Initial health and fitness data for goal calculation.
      * `user_id` (`VARCHAR(36) PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE`)
      * `age` (`INTEGER`)
      * `gender` (`VARCHAR(20)`)
      * `height_cm` (`DECIMAL(5, 2)`)
      * `current_weight_kg` (`DECIMAL(5, 2)`)
      * `target_weight_kg` (`DECIMAL(5, 2)`)
      * `fitness_goal` (`VARCHAR(50)`) - e.g., 'lose\_weight', 'maintain\_weight', 'gain\_weight'
      * `activity_level` (`VARCHAR(50)`) - e.g., 'sedentary', 'lightly\_active'
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `updated_at` (`TIMESTAMP WITH TIME ZONE`)
  * **`food_entries`**: Records of individual food items consumed.
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `user_id` (`VARCHAR(36) REFERENCES users(id) ON DELETE CASCADE`)
      * `name` (`VARCHAR(255) NOT NULL`)
      * `calories` (`DECIMAL(10, 2) NOT NULL`)
      * `meal_type` (`VARCHAR(50) NOT NULL`) - e.g., 'breakfast', 'lunch', 'dinner', 'snack'
      * `serving_size` (`VARCHAR(255)`)
      * `protein` (`DECIMAL(10, 2)`)
      * `carbs` (`DECIMAL(10, 2)`)
      * `fat` (`DECIMAL(10, 2)`)
      * `notes` (`TEXT`)
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
  * **`user_badges`**: Records earned achievements.
      * `id` (`SERIAL PRIMARY KEY`)
      * `user_id` (`VARCHAR(36) REFERENCES users(id) ON DELETE CASCADE`)
      * `badge_id` (`VARCHAR(255) NOT NULL`) - Unique identifier for the badge (e.g., 'week\_streak', 'first\_10\_entries')
      * `badge_name` (`VARCHAR(255) NOT NULL`)
      * `badge_description` (`TEXT`)
      * `earned_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `UNIQUE(user_id, badge_id)` - Ensures a user can earn each badge only once.

-----

## 7\. Security Considerations

  * **Secrets Management**: `DB_PASSWORD`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY`, and `JWT_SECRET` are securely loaded from Docker secrets, preventing hardcoding.
  * **Non-Root User**: The application runs as a non-root `appuser` within the container, adhering to container security best practices.
  * **JWT Protection**: All core API endpoints are protected by JWT verification (`token_required` decorator), ensuring that only authenticated users can access and modify their data.
      * The `token_required` decorator also handles **user synchronization**, querying the Auth Service if a user's basic profile isn't found in the User Service's local `users` table. This provides resilience and ensures eventual consistency for basic user data.
  * **Input Validation**: Data received from the client (e.g., food entry details, onboarding data, nutrition goals) is validated for presence and basic data types. More extensive validation (e.g., numerical ranges) is applied where appropriate (e.g., for `nutrition-goals` and `calculate_calories`).
  * **Inter-Service Authentication**: When forwarding requests to the Auth Service (`/change-password`, `/account`, `/logout`), the User Service re-uses the original client's JWT token in the `Authorization` header. This ensures the Auth Service still authenticates the *client*, not just the User Service itself.
  * **Object Storage Security**: MinIO access keys are used to control access to buckets. In production, MinIO should be configured with secure connections (HTTPS/TLS, `secure=True` in `minio_client`).
  * **Data Isolation**: Data is partitioned by `user_id` in all relevant tables, ensuring that users can only access their own data. The `ON DELETE CASCADE` on foreign keys (e.g., from `users` to `user_preferences`, `food_entries`) ensures that when a user account is deleted, all associated data in the User Service is also removed.

-----

## 8\. Project Structure

  * **`app.py`**: The main Flask application file, containing:
      * Flask app initialization and CORS.
      * Loading of environment variables and secrets.
      * Database connection (`get_db_connection`).
      * MinIO client setup and bucket initialization.
      * `token_required` decorator for JWT validation and user synchronization.
      * Helper functions for parsing dates, getting user info from Auth Service.
      * Core API endpoints for user profile, food entries, nutrition goals, dashboard summary, and onboarding.
      * Forwarding logic for password change, account deletion, and logout to Auth Service.
      * Badge awarding logic within `add_food_entry`.
      * Nutrition calculation logic (`calculate_calories`, `calculate_macros`).
      * MinIO image serving endpoint.
      * `/health` endpoint.
  * **`init_db.py`**: A standalone script for:
      * Waiting for PostgreSQL to be available.
      * Creating all necessary database tables and indexes for the User Service.
  * **`Dockerfile`**: Defines the Docker image for the User Service.
  * **`requirements.txt`**: Lists all Python dependencies.
  * **`secrets/`**: (Conceptual directory for Docker Compose) Contains sensitive files mounted as Docker secrets.

-----

## 9\. Monitoring and Troubleshooting

  * **Logs**: The service logs informational messages and errors to standard output/error. Integrate with a centralized logging solution for analysis.
      * Look for `Database connection error` if PostgreSQL is unreachable.
      * Check for `MinIO connection error` or `Error ensuring bucket exists` for object storage issues.
      * Monitor `User not found in user-service DB, attempting to sync from auth-service` and related messages to track user synchronization.
      * Look for `Error calling auth service` messages for issues with inter-service communication.
      * Examine `Error adding food entry`, `Error retrieving food entries`, etc., for API-specific problems.
      * `Error saving onboarding data` or `Invalid data for calorie calculation` point to client input issues.
  * **Health Check (`/health`)**: Use this endpoint from your orchestrator to monitor the liveness and readiness of the User Service instances. It checks PostgreSQL and MinIO connectivity.
  * **PostgreSQL Monitoring**: Monitor your database's connections, query performance, and disk usage. Pay attention to queries on `food_entries` for potential indexing needs if performance degrades over time with many entries.
  * **MinIO Monitoring**: Monitor MinIO's health, disk usage, and access logs. Ensure buckets are created and accessible.
  * **Network Connectivity**: Verify network connectivity between the User Service container and PostgreSQL, MinIO, and the Auth Service.
  * **Gunicorn Workers**: Monitor Gunicorn worker processes. Adjust `workers` and `threads` based on load and available resources to prevent performance bottlenecks.
