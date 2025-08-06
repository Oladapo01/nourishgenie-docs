# Subscription Service

This repository contains the Subscription Service, a dedicated microservice for the Food Tracker application backend. It is responsible for managing user subscriptions, payment profiles, handling billing logic, and integrating with external payment gateways like Stripe.

-----

## 1\. Overview

The Subscription Service provides the core business logic for offering premium features through a subscription model. It handles user enrollment in plans, manages their billing cycles, processes payments, and responds to events from the payment gateway. This service ensures a seamless and secure subscription experience for users, allowing the main application to focus on food tracking features.

-----

## 2\. Features

  * **Subscription Management**: Allows users to view available plans, start new subscriptions (including trials), and manage existing ones.
  * **Payment Gateway Integration**: Seamlessly integrates with **Stripe** for handling customer management, subscriptions, and payment processing.
  * **Trial Periods**: Supports trial periods for new subscriptions, allowing users to experience premium features before committing.
  * **Payment Method Management**: Enables users to add and update their primary payment methods securely through Stripe.
  * **Auto-Renewal Control**: Users can enable or disable auto-renewal for their subscriptions.
  * **Webhook Handling**: Processes real-time events from Stripe (e.g., `customer.subscription.created`, `invoice.payment_succeeded`, `customer.subscription.deleted`) to keep the internal database synchronized with Stripe's records.
  * **Database Persistence**: Stores subscription, payment profile, and payment history data in **PostgreSQL**.
  * **Promotional Codes (Future Ready)**: Includes database schema and basic structures for managing and applying promotional codes.
  * **Authentication & Authorization**: Protects its API endpoints using **JWT verification**, ensuring only authenticated users can manage their subscriptions.
  * **Health Check**: Provides a `/health` endpoint to monitor the status of database, Redis, and Stripe connectivity.
  * **Secure Configuration**: Loads sensitive API keys and passwords from Docker secrets.
  * **Containerized Deployment**: Designed for easy deployment with Docker, including an `init_db.py` script for automated database schema setup.

-----

## 3\. Technology Stack

  * **Python 3.11**: Primary programming language.
  * **Flask**: Lightweight web framework for building APIs.
  * **Gunicorn**: WSGI HTTP server for running the Flask application in production.
  * **PostgreSQL**: Relational database for persistent data storage.
      * `psycopg2`: PostgreSQL adapter for Python.
  * **Redis**: In-memory data store for caching or potential future uses.
      * `redis-py`: Python client for Redis.
  * **Stripe Python Library**: Official client library for interacting with the Stripe API.
  * **`PyJWT`**: For JSON Web Token (JWT) verification.
  * **Docker**: For containerization and deployment.

-----

## 4\. Getting Started

These instructions will guide you through setting up and running the Subscription Service.

### 4.1. Prerequisites

  * Python 3.11+
  * `pip`
  * Docker and Docker Compose (recommended for local development)
  * PostgreSQL instance (local or remote)
  * Redis instance (local or remote)
  * **Stripe Account and API Keys**: You'll need both your **Stripe Secret API Key** (`sk_test_...` for test mode) and a **Stripe Webhook Secret** for receiving events. You'll also need to configure **Product and Price IDs** in Stripe for your "MONTHLY" and "ANNUAL" plans.

### 4.2. Local Setup (without Docker - for development purposes)

Running the Subscription Service directly without Docker is possible for development but requires manual setup of environment variables and potentially creating dummy secret files.

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/Oladapo01/FoodTrackerApp.git/subscription-service
    cd subscription-service
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
    POSTGRES_DB=subscriptiondb
    POSTGRES_USER=postgres
    # POSTGRES_PASSWORD=your_db_password # Use a secrets file in production

    REDIS_HOST=localhost
    REDIS_PORT=6379
    # REDIS_PASSWORD=your_redis_password # Use a secrets file in production

    # STRIPE_API_KEY=sk_test_YOUR_STRIPE_SECRET_KEY # Use a secrets file in production
    # STRIPE_WEBHOOK_SECRET=whsec_YOUR_STRIPE_WEBHOOK_SECRET # Use a secrets file in production

    # Define your Stripe Price IDs here for local testing.
    # Get these from your Stripe Dashboard (e.g., price_123abc)
    STRIPE_MONTHLY_PRICE_ID=price_monthly_test
    STRIPE_ANNUAL_PRICE_ID=price_annual_test

    SUBSCRIPTION_TRIAL_DAYS=7

    # This would typically be managed via Docker secrets or Kubernetes secrets.
    # For local testing without Docker, you'd need to create them as local files.
    # Example:
    # mkdir -p /run/secrets # Create this directory
    # echo "your_strong_db_password" > /run/secrets/db_password
    # echo "your_strong_redis_password" > /run/secrets/redis_password
    # echo "sk_test_YOUR_STRIPE_SECRET_KEY" > /run/secrets/stripe_api_key
    # echo "whsec_YOUR_STRIPE_WEBHOOK_SECRET" > /run/secrets/stripe_webhook_secret
    # echo "your_super_secret_jwt_key" > /run/secrets/jwt_secret
    ```

    **Important Note on Secrets:** In production, sensitive values like database passwords, Redis passwords, Stripe API keys, Stripe webhook secrets, and JWT secrets **must** be managed using secure mechanisms like Docker Secrets (as implemented in the `Dockerfile` and `app.py`) or Kubernetes Secrets. **Never** hardcode secrets or expose them directly in `.env` files that are committed to version control.

5.  **Initialize the database schema:**

    ```bash
    python init_db.py
    ```

    Ensure your PostgreSQL instance is running and accessible before running this.

6.  **Run the application:**

    ```bash
    gunicorn --bind 0.0.0.0:8084 --workers 1 --threads 2 app:app
    ```

    (You might use `flask run` directly for simpler development, but Gunicorn is closer to the production setup).

### 4.3. Docker Deployment

The `Dockerfile` provides a production-ready image for the Subscription Service. It installs necessary system dependencies, Python packages, sets up a non-root user, and defines the startup command, which first runs the database initialization script.

1.  **Build the Docker image:**

    ```bash
    docker build -t your-repo/subscription-service:latest .
    ```

    Replace `your-repo/subscription-service` with your desired image name.

2.  **Run the Docker container (example with Docker Compose):**

    It's highly recommended to use **Docker Compose** for local development and orchestration with other services (API Gateway, PostgreSQL, Redis). A `docker-compose.yml` snippet for the Subscription Service would look like this:

    ```yaml
    version: '3.8'
    services:
      subscription-service:
        build:
          context: ./subscription-service # Assuming subscription-service is a sub-directory
          dockerfile: Dockerfile
        ports:
          - "8084:8084" # Exposed internally, API Gateway handles external access
        environment:
          POSTGRES_HOST: postgres
          POSTGRES_PORT: 5432
          POSTGRES_DB: subscriptiondb
          POSTGRES_USER: postgres
          REDIS_HOST: redis
          REDIS_PORT: 6379
          SUBSCRIPTION_TRIAL_DAYS: 7
          STRIPE_MONTHLY_PRICE_ID: price_monthly_prod_id # Set actual production Price IDs
          STRIPE_ANNUAL_PRICE_ID: price_annual_prod_id
        secrets:
          - db_password
          - redis_password
          - jwt_secret
          - stripe_api_key
          - stripe_webhook_secret
        depends_on:
          postgres:
            condition: service_healthy # Or service_started
          redis:
            condition: service_healthy
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:8084/health"]
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 30s # Allow time for init_db.py to run and connect
      postgres:
        image: postgres:13
        environment:
          POSTGRES_DB: subscriptiondb
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD_FILE: /run/secrets/db_password
        secrets:
          - db_password
        volumes:
          - subscription_db_data:/var/lib/postgresql/data
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres -d subscriptiondb"]
          interval: 10s
          timeout: 5s
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
      db_password:
        file: ./secrets/db_password.txt # Create this file with your DB password
      redis_password:
        file: ./secrets/redis_password.txt # Create this file with your Redis password
      jwt_secret:
        file: ./secrets/jwt_secret.txt # Create this file with a strong, random JWT secret
      stripe_api_key:
        file: ./secrets/stripe_api_key.txt # Create this file with your Stripe Secret Key (sk_test_...)
      stripe_webhook_secret:
        file: ./secrets/stripe_webhook_secret.txt # Create this file with your Stripe Webhook Signing Secret (whsec_...)
    volumes:
      subscription_db_data:
    ```

    To run this `docker-compose.yml`:

    1.  Create a `secrets` directory in your root project folder.
    2.  Inside `secrets`, create `db_password.txt`, `redis_password.txt`, `jwt_secret.txt`, `stripe_api_key.txt`, and `stripe_webhook_secret.txt` and fill them with strong, random passwords/secrets and your Stripe keys.
    3.  From the directory containing your `docker-compose.yml`, run:
        ```bash
        docker compose up --build
        ```

### 4.4. `init_db.py` Explained

The `init_db.py` script is executed as part of the container's startup command (`CMD python init_db.py && ...`). Its responsibilities include:

  * **Waiting for PostgreSQL**: It includes a robust retry mechanism (`wait_for_postgres`) to ensure the PostgreSQL database is fully ready and accessible before attempting to connect and initialize the schema. This is crucial in containerized environments where services might start up at different rates.
  * **Database Schema Creation**: It checks for and creates all necessary tables and indexes for the Subscription Service if they don't already exist. This ensures that the database is correctly set up on the first run of the service.

-----

## 5\. API Endpoints

The Subscription Service exposes the following RESTful API endpoints:

### 5.1. Subscription Plans and Status

#### `GET /subscription-service/plans`

  * **Description**: Retrieves a list of all available subscription plans with their details (price, features, savings).
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Response (200 OK)**:
    ```json
    [
      {
        "type": "MONTHLY",
        "pricePerMonth": 5.99,
        "totalPrice": 5.99,
        "description": "Monthly Premium",
        "features": ["Advanced food recognition...", "..."]
      },
      {
        "type": "ANNUAL",
        "pricePerMonth": 4.99,
        "totalPrice": 59.99,
        "description": "Annual Premium (Best Value)",
        "features": ["Advanced food recognition...", "..."],
        "savingsPercent": 16,
        "savingsAmount": 11.89
      }
    ]
    ```
  * **Errors**: `401 Unauthorized`, `500 Internal Server Error`.

#### `GET /subscription-service/status`

  * **Description**: Retrieves the current subscription status for the authenticated user.
  * **Authorization**: Requires a valid **Access Token** in the `Authorization: Bearer <token>` header.
  * **Response (200 OK)**:
    ```json
    {
      "isSubscribed": true,
      "isInTrialPeriod": false,
      "currentPlan": "ANNUAL",
      "startDate": "2024-01-15T10:00:00Z",
      "trialEndDate": null,
      "nextBillingDate": "2025-01-15T10:00:00Z",
      "autoRenewEnabled": true,
      "canceledDate": null
    }
    ```
    (Or `isSubscribed: false` and other fields `null` if no active subscription).
  * **Errors**: `401 Unauthorized`, `500 Internal Server Error`.

### 5.2. Subscription Actions

#### `POST /subscription-service/start`

  * **Description**: Initiates a new subscription or a trial period for a user. Can either begin a free trial or immediately attempt to create a Stripe subscription if a `paymentMethodId` is provided.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**:
    ```json
    {
      "planType": "MONTHLY" | "ANNUAL",
      "paymentMethodId": "pm_card_visa" // Optional, for immediate subscription
      // "promoCode": "DISCOUNT10" // Optional, for future use
    }
    ```
  * **Response (200 OK)**: Returns the updated subscription status.
  * **Errors**: `400 Bad Request` (Invalid plan type, missing payment method ID if required), `401 Unauthorized`, `409 Conflict` (User already has an active subscription), `500 Internal Server Error` (Stripe errors, database errors).

#### `POST /subscription-service/cancel`

  * **Description**: Cancels the user's current active subscription. The subscription will remain active until the end of the current billing period or trial.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**: `{}` (empty JSON object)
  * **Response (200 OK)**: Returns the updated subscription status with `autoRenewEnabled: false` and `canceledDate` set.
  * **Errors**: `401 Unauthorized`, `404 Not Found` (No active subscription), `500 Internal Server Error`.

#### `POST /subscription-service/payment-method`

  * **Description**: Updates the default payment method for the user's Stripe customer profile and optionally for their active subscription.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**:
    ```json
    {
      "paymentMethodId": "pm_card_newcard_id"
    }
    ```
  * **Response (200 OK)**: Returns the updated subscription status.
  * **Errors**: `400 Bad Request` (Missing payment method ID), `401 Unauthorized`, `500 Internal Server Error` (Stripe errors).

#### `POST /subscription-service/auto-renewal`

  * **Description**: Enables or disables auto-renewal for the user's current subscription.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**:
    ```json
    {
      "enabled": true | false
    }
    ```
  * **Response (200 OK)**: Returns the updated subscription status.
  * **Errors**: `400 Bad Request` (Missing `enabled` flag), `401 Unauthorized`, `404 Not Found` (No active subscription), `500 Internal Server Error`.

### 5.3. Webhooks

#### `POST /subscription-service/webhook`

  * **Description**: Receives and processes events from the Stripe webhook system. This endpoint is called directly by Stripe, not by the client application. It verifies the Stripe signature for security.
  * **Request Content**: Stripe Event Payload.
  * **Response (200 OK)**: `{"status": "success"}`
  * **Errors**: `400 Bad Request` (Invalid payload, invalid signature, missing secret), `500 Internal Server Error`.
  * **Handled Events**:
      * `customer.subscription.created`: Records new subscriptions in the database.
      * `customer.subscription.updated`: Updates subscription status (e.g., trialing to active, changes in auto-renew).
      * `customer.subscription.deleted`: Marks subscriptions as canceled.
      * `invoice.payment_succeeded`: Records successful payments and activates subscriptions if coming from trial.
      * `invoice.payment_failed`: Records failed payments.

### 5.4. Health Check

#### `GET /health`

  * **Description**: Endpoint for readiness and liveness probes. Reports the connectivity status of PostgreSQL, Redis, and Stripe API configuration.
  * **Response (200 OK or 503 Service Unavailable)**:
    ```json
    {
      "status": "healthy", // or "unhealthy"
      "service": "subscription-service",
      "checks": {
        "database": "connected", // or "disconnected"
        "redis": "connected",    // or "disconnected"
        "stripe": "configured"   // or "not configured"
      }
    }
    ```

-----

## 6\. Database Schema

The `init_db.py` script ensures the following tables are created and managed by the Subscription Service in PostgreSQL:

  * **`users`**: (Simplified, minimal data needed for subscriptions; main user data is in Auth/User Service)
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `email` (`VARCHAR(255) UNIQUE NOT NULL`)
      * `name` (`VARCHAR(255)`)
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
  * **`user_payment_profiles`**: Stores a user's Stripe customer ID.
      * `user_id` (`VARCHAR(36) PRIMARY KEY REFERENCES users(id)`)
      * `stripe_customer_id` (`VARCHAR(255) UNIQUE`)
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `updated_at` (`TIMESTAMP WITH TIME ZONE`)
  * **`payment_methods`**: Stores details of payment methods linked to users.
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `user_id` (`VARCHAR(36) REFERENCES users(id)`)
      * `stripe_payment_method_id` (`VARCHAR(255) UNIQUE NOT NULL`)
      * `is_default` (`BOOLEAN DEFAULT FALSE`)
      * `card_brand` (`VARCHAR(50)`) - e.g., 'visa', 'mastercard' (from Stripe metadata)
      * `card_last4` (`VARCHAR(4)`)
      * `card_exp_month` (`INTEGER`)
      * `card_exp_year` (`INTEGER`)
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `updated_at` (`TIMESTAMP WITH TIME ZONE`)
  * **`subscriptions`**: Records user subscriptions.
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `user_id` (`VARCHAR(36) REFERENCES users(id)`)
      * `plan_type` (`VARCHAR(50) NOT NULL`) - e.g., 'MONTHLY', 'ANNUAL'
      * `status` (`VARCHAR(50) NOT NULL`) - e.g., 'trialing', 'active', 'canceled', 'past\_due'
      * `stripe_subscription_id` (`VARCHAR(255) UNIQUE`)
      * `start_date` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `trial_end_date` (`TIMESTAMP WITH TIME ZONE`)
      * `next_billing_date` (`TIMESTAMP WITH TIME ZONE`)
      * `canceled_date` (`TIMESTAMP WITH TIME ZONE`)
      * `auto_renew` (`BOOLEAN DEFAULT TRUE`)
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `updated_at` (`TIMESTAMP WITH TIME ZONE`)
  * **`subscription_payments`**: Logs successful and failed payment attempts.
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `user_id` (`VARCHAR(36) REFERENCES users(id)`)
      * `subscription_id` (`VARCHAR(36) REFERENCES subscriptions(id)`)
      * `stripe_invoice_id` (`VARCHAR(255) UNIQUE`)
      * `amount` (`DECIMAL(10, 2) NOT NULL`)
      * `status` (`VARCHAR(50) NOT NULL`) - e.g., 'succeeded', 'failed'
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
  * **`promo_codes`**: (Future use) Defines available promotional codes.
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `code` (`VARCHAR(50) UNIQUE NOT NULL`)
      * `description` (`VARCHAR(255)`)
      * `discount_percent` (`INTEGER`)
      * `discount_amount` (`DECIMAL(10, 2)`)
      * `valid_from` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
      * `valid_until` (`TIMESTAMP WITH TIME ZONE`)
      * `max_uses` (`INTEGER`)
      * `current_uses` (`INTEGER DEFAULT 0`)
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
  * **`promo_code_redemptions`**: (Future use) Records when a user redeems a promo code.
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `user_id` (`VARCHAR(36) REFERENCES users(id)`)
      * `promo_code_id` (`VARCHAR(36) REFERENCES promo_codes(id)`)
      * `subscription_id` (`VARCHAR(36) REFERENCES subscriptions(id)`)
      * `redeemed_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)

-----

## 7\. Security Considerations

  * **Secrets Management**: All sensitive credentials (DB password, Redis password, Stripe API keys, JWT secret, Stripe webhook secret) are loaded from Docker secrets, preventing them from being hardcoded or exposed.
  * **Non-Root User**: The application runs as a non-root `appuser` within the container, adhering to container security best practices.
  * **JWT Protection**: All subscription management API endpoints require a valid JWT, ensuring only authenticated and authorized users can manage their subscriptions.
  * **Stripe Webhook Security**: The `/webhook` endpoint explicitly validates the `Stripe-Signature` header to ensure incoming events are genuinely from Stripe and have not been tampered with. This is crucial for preventing fraudulent updates to subscription statuses.
  * **PCI DSS Compliance**: As the service integrates with Stripe, it offloads direct handling of sensitive card data to Stripe. The service only works with Stripe-generated `paymentMethodId` tokens, reducing the scope of PCI DSS compliance for your backend.
  * **Database Transactionality**: While `autocommit` is used in `get_db_connection`, for multi-step operations that require atomic changes, explicit transactions (e.g., `conn.begin()` and `conn.commit()`) should be considered if not already implicitly handled by the ORM/driver for safety.

-----

## 8\. Project Structure

  * **`app.py`**: The main Flask application file, containing:
      * Flask app initialization and CORS.
      * Loading of environment variables and secrets.
      * Database connection (`get_db_connection`).
      * Redis client setup.
      * Stripe API initialization.
      * `token_required` decorator for JWT validation.
      * Helper functions for Stripe interaction (e.g., `get_or_create_stripe_customer`) and subscription status formatting.
      * API endpoints for subscription management.
      * Stripe webhook endpoint and event handlers.
      * `/health` endpoint.
  * **`init_db.py`**: A standalone script for:
      * Waiting for PostgreSQL to be available.
      * Creating all necessary database tables and indexes.
  * **`Dockerfile`**: Defines the Docker image for the Subscription Service.
  * **`requirements.txt`**: Lists all Python dependencies.
  * **`secrets/`**: (Conceptual directory for Docker Compose) Contains sensitive files mounted as Docker secrets.

-----

## 9\. Monitoring and Troubleshooting

  * **Logs**: The service logs informational messages and errors to standard output/error. Integrate with a centralized logging solution for analysis.
      * Look for `Database connection error` if the service cannot connect to PostgreSQL.
      * Check for `Redis connection error` or `Stripe error` messages for issues with external services.
      * Monitor logs from `/webhook` for `Invalid payload` or `Invalid signature` if Stripe webhooks are not being processed correctly.
  * **Health Check (`/health`)**: Use this endpoint from your orchestrator (e.g., Docker, Kubernetes) to monitor the liveness and readiness of the Subscription Service instances. It explicitly checks PostgreSQL, Redis, and Stripe connectivity.
  * **Stripe Dashboard**:
      * Monitor your Stripe dashboard for payment failures, subscription events, and webhook delivery status.
      * Ensure your webhook endpoint is correctly configured in Stripe and is receiving events.
      * Check for any `failed` or `errored` webhook attempts in Stripe's webhook logs.
  * **PostgreSQL Monitoring**: Monitor your database's connections, query performance, and disk usage.
  * **Network Connectivity**: Verify network connectivity between the Subscription Service container and PostgreSQL, Redis, and Stripe API endpoints.
  * **Gunicorn Workers**: Monitor Gunicorn worker processes. Adjust `workers` and `threads` based on load and available resources to prevent performance bottlenecks.

