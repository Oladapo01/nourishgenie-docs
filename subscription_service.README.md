# Subscription Service

This repository contains the Subscription Service, a dedicated microservice for the NourishGenie application backend. It is responsible for managing user subscriptions, payment profiles, handling billing logic, and integrating with external payment gateways like Stripe.

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
  * **Webhook Handling**: Processes real-time events from Stripe (e.g., `customer.subscription.created`, `invoice.payment_succeeded`, `customer.subscription.deleted`) to keep the internal database synchronized with Stripe's records. Idempotent against Stripe's retry behavior — a redelivered event is recognized and skipped rather than processed twice.
  * **Cross-Service User Sync**: A dedicated `/sync-user` endpoint lets the Auth Service create or update the minimal user record this service needs at registration time, authenticated via service-level claims rather than an ordinary user token.
  * **Promotional Codes**: Users can redeem coupon codes against their subscription; administrators can generate single or batch coupon codes. Both the redemption and validation endpoints are rate-limited per user.
  * **Admin Endpoints**: Coupon generation is gated behind a separate admin key, compared using a constant-time comparison rather than a direct string equality check.
  * **Database Persistence**: Stores subscription, payment profile, and payment history data in **PostgreSQL**.
  * **Authentication & Authorization**: Protects its API endpoints using **JWT verification**, ensuring only authenticated users can manage their subscriptions.
  * **Health Check**: Provides a `/health` endpoint to monitor the status of database, Redis, and Stripe connectivity.
  * **Secure Configuration**: Loads sensitive API keys and passwords from Docker secrets. Refuses to start if critical secrets (JWT secret, admin key, Stripe webhook secret) are missing or below a minimum length, and refuses to start if the Stripe price IDs are missing or still placeholder values, rather than failing later at charge time.
  * **Containerized Deployment**: Designed for easy deployment with Docker, including an `init_db.py` script for automated database schema setup and a versioned migration runner for schema changes after the initial release.

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

A separate `run_migrations.py`, using the same Postgres advisory-lock pattern as the Auth Service, applies versioned schema changes after the initial release (for example, the webhook-idempotency table and payment uniqueness constraint described below). The container's startup command runs both, in order, joined with `&&` rather than `;`, so a failed migration prevents the application from starting against a schema it doesn't match, instead of booting anyway and failing at request time.

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

#### `POST /subscription-service/sync-user`

  * **Description**: Creates or updates the minimal user record this service needs (id, email, name), called by the Auth Service at registration time. This is a service-to-service endpoint, not a user-facing one.
  * **Authorization**: Requires a JWT carrying service-level claims (`service: "auth"`, `purpose: "sync"`) rather than an ordinary user access token.
  * **Response (200 OK)**: Confirmation of the synced user record.
  * **Errors**: `401 Unauthorized` (missing or non-service token).
  * **Note**: This endpoint previously granted privileged access to *any* valid user access token, based only on which endpoint was being called rather than what the token actually claimed. Because the request body supplies `user_id`, `email`, and `name` directly, any logged-in user could have overwritten another user's subscription record and altered the email associated with their Stripe customer. Fixed by requiring the actual service claims rather than inferring trust from the endpoint name.

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
  * **Errors**: `400 Bad Request` (Invalid payload, invalid signature, **or missing signature header**), `500 Internal Server Error`.
  * **Handled Events**:
      * `customer.subscription.created`: Records new subscriptions in the database.
      * `customer.subscription.updated`: Updates subscription status (e.g., trialing to active, changes in auto-renew).
      * `customer.subscription.deleted`: Marks subscriptions as canceled.
      * `invoice.payment_succeeded`: Records successful payments and activates subscriptions if coming from trial.
      * `invoice.payment_failed`: Records failed payments.
  * **Resolved: Signature Bypass (Critical)**: A request with no `Stripe-Signature` header at all previously fell through to parsing the payload as trusted JSON rather than being rejected — so an attacker could POST a fabricated event (e.g. `customer.subscription.created` naming an arbitrary `user_id` in its metadata) and grant that account an active subscription, without needing to forge a valid signature. A forged signature was already correctly rejected; the gap was specifically the *absence* of one. Fixed so a missing header is rejected with the same `400` as an invalid one.
  * **Idempotency**: Stripe retries webhook delivery for up to three days on any non-2xx response. The payment-recording handlers previously inserted a new row on every delivery, so a single retried event produced a duplicate payment record — and because those handlers also caught and swallowed their own exceptions before returning `200`, a genuine processing failure was silently acknowledged rather than retried, permanently losing that payment record. Both issues are addressed together: incoming events are recorded against a processed-events table keyed on the Stripe event ID before being handled, so a redelivery is recognized and skipped, which in turn makes it safe for a genuine failure to return a `500` and let Stripe retry rather than swallowing the error to avoid a duplicate.
  * **Architecture Note**: This is the only route under the `/subscription-service` prefix that Stripe calls directly at the underlying service URL rather than through the API Gateway (the gateway's path-rewrite for this prefix does not resolve to this route, so a gateway-routed webhook would 404). One consequence worth being explicit about: this endpoint does not currently receive the Gateway's rate limiting, which is part of why the signature-bypass fix above and the idempotency table matter more here than they would on a route that does sit behind the Gateway.

### 5.4. Promotional Codes

#### `POST /subscription-service/coupons/validate`

  * **Description**: Checks whether a coupon code is valid without redeeming it.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**: `{"code": "DISCOUNT10"}`
  * **Errors**: `400 Bad Request` (missing or non-string code), `429 Too Many Requests` (rate limit exceeded — see note below).
  * **Note**: Validation and redemption share the same per-user rate-limit counter, because validation is the cheaper of the two to abuse: it confirms whether a code exists without consuming a use, making it the more attractive target for brute-forcing an 8-character code. The limiter fails open (allows the request) if Redis is unavailable, consistent with this service's general preference for availability over strict enforcement during an infrastructure blip.

#### `POST /subscription-service/coupons/redeem`

  * **Description**: Redeems a coupon code against the user's subscription.
  * **Authorization**: Requires a valid **Access Token**.
  * **Request Body**: `{"code": "DISCOUNT10"}`
  * **Errors**: `400 Bad Request` (missing/non-string code), `429 Too Many Requests` (shared rate limit with `/coupons/validate`, see above).

#### `POST /subscription-service/admin/coupons/generate`

  * **Description**: Generates one or more coupon codes. Administrative endpoint, not intended for client use.
  * **Authorization**: Requires the admin API key via a request header, compared using a constant-time comparison rather than `==`, to avoid leaking key length or prefix through response-timing differences.
  * **Errors**: `400 Bad Request` (invalid input — see validation note below), `403 Forbidden` (missing/invalid admin key).
  * **Note**: Batch generation with an 8-character-or-longer prefix previously produced a zero-length random suffix, so every generated code was identical to the prefix and the uniqueness-check loop spun indefinitely, hanging the worker process handling the request. The endpoint now caps the prefix at 4 characters. Input fields (coupon code, coupon type, counts) are also now validated for type before use — a non-string value previously reached string methods unguarded and returned a generic `500` rather than a clear `400`.

### 5.5. Health Check

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
      * `stripe_invoice_id` (`VARCHAR(255)`) — unique together with `status`, not on its own: a card that fails and is then retried successfully produces `invoice.payment_failed` followed by `invoice.payment_succeeded` for the *same* invoice ID, and both rows are legitimate. A uniqueness constraint on `stripe_invoice_id` alone would have rejected the second, correct row.
      * `amount` (`DECIMAL(10, 2) NOT NULL`)
      * `status` (`VARCHAR(50) NOT NULL`) - e.g., 'succeeded', 'failed'
      * `created_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)
  * **`processed_webhook_events`**: Records every Stripe webhook event ID this service has seen, so a redelivered event is recognized and skipped rather than reprocessed (see the webhook idempotency note in Section 5.3).
      * `event_id` (Stripe's event ID, unique)
      * `event_type`
      * `status` — allows a failed processing attempt to be retried on redelivery without permanently marking the event as handled
      * `attempts`
      * `first_seen_at`
  * **`promo_codes`**: Defines available promotional/coupon codes, used by the coupon validate/redeem/admin-generate endpoints in Section 5.4.
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
  * **`promo_code_redemptions`**: Records when a user redeems a promo code.
      * `id` (`VARCHAR(36) PRIMARY KEY`)
      * `user_id` (`VARCHAR(36) REFERENCES users(id)`)
      * `promo_code_id` (`VARCHAR(36) REFERENCES promo_codes(id)`)
      * `subscription_id` (`VARCHAR(36) REFERENCES subscriptions(id)`)
      * `redeemed_at` (`TIMESTAMP WITH TIME ZONE NOT NULL`)

-----

## 7\. Security Considerations

  * **Secrets Management**: All sensitive credentials (DB password, Redis password, Stripe API keys, JWT secret, Stripe webhook secret) are loaded from Docker secrets, preventing them from being hardcoded or exposed.
  * **Startup Validation**: The service refuses to start if `JWT_SECRET` (minimum 32 characters), `ADMIN_API_KEY` (minimum 16 characters), or `STRIPE_WEBHOOK_SECRET` is missing, and refuses to start if the Stripe monthly/annual price IDs are missing or still hold placeholder values. This was a deliberate change from failing later: the price ID case previously failed only at charge time, after a user had already entered card details.
  * **Resolved: Hardcoded Admin Key Default**: `ADMIN_API_KEY` previously had a hardcoded fallback value committed in source, which would have granted coupon-generation access to anyone who read the repository if the environment variable was ever left unset. The default was removed in favor of the startup validation above, and the comparison itself now uses `hmac.compare_digest` over hashed values rather than direct string equality, since `!=` can leak information about key length and matching prefix through response timing.
  * **Non-Root User**: The application runs as a non-root `appuser` within the container, adhering to container security best practices.
  * **JWT Protection**: All user-facing subscription management API endpoints require a valid JWT, ensuring only authenticated and authorized users can manage their subscriptions. The one non-user-facing endpoint (`/sync-user`) requires actual service-level claims rather than any valid user token — see Section 5.2 for the vulnerability this closed.
  * **Stripe Webhook Security**: The `/webhook` endpoint validates the `Stripe-Signature` header to ensure incoming events are genuinely from Stripe and have not been tampered with — including rejecting requests where the header is absent entirely, which previously bypassed verification rather than failing it. See Section 5.3 for the full account of this fix and why it mattered more than usual given this route isn't behind the API Gateway.
  * **Webhook Idempotency**: Every processed Stripe event is recorded against its event ID before being handled, so Stripe's automatic retries (which continue for up to three days on any non-2xx response) don't produce duplicate payment records, and a genuine processing failure can safely return an error and be retried rather than being silently swallowed to avoid that duplication.
  * **Coupon Brute-Force Limiting**: Coupon codes are short (8 characters) and previously had no attempt limit on validation or redemption. Both endpoints now share a per-user Redis-backed rate limit (10 attempts/hour), since validation is the cheaper of the two to abuse — it reveals whether a code exists without consuming a use.
  * **Input Type Validation**: Request fields (coupon code, coupon type, batch counts) are validated for type before use, so a malformed value (e.g. a number where a string was expected) returns a clear `400` instead of an unhandled exception surfacing as a generic `500`.
  * **PCI DSS Compliance**: As the service integrates with Stripe, it offloads direct handling of sensitive card data to Stripe. The service only works with Stripe-generated `paymentMethodId` tokens, reducing the scope of PCI DSS compliance for your backend.
  * **Error Message Hardening**: Stripe-related errors are handled through a helper that logs the full exception server-side and returns a generic message to the client — except `CardError`, where Stripe's own user-facing message (e.g. "Your card was declined") is deliberately preserved, since suppressing it would break the decline-handling UX rather than improve security.
  * **Wide-Open CORS Removed**: An unconditional `CORS(app)` (allowing any origin) was removed.
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
  * **`run_migrations.py`** / **`migrations/`**: Applies versioned schema changes after the initial release (e.g. the webhook-idempotency table and payment uniqueness constraint), using the same Postgres advisory-lock pattern as the Auth Service so concurrent container starts don't race to apply the same migration twice.
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
  * **`processed_webhook_events` Growth**: This table grows without bound by design (see Section 6). A periodic job removing `completed` rows older than 30 days — well past Stripe's 3-day retry window — was identified as a reasonable follow-up but is not itself part of the request-handling code. A useful monitoring query in the meantime: rows that are non-`completed` and older than an hour indicate webhook state has drifted from Stripe and warrants investigation.

-----

## 10\. Production Hardening & Incident History

The items below are resolved production issues, kept here as a record of the reasoning behind current behavior rather than as a changelog of code diffs.

  * **Stripe Webhook Signature Bypass (Critical)**: A request to `/webhook` with no `Stripe-Signature` header at all previously fell through to parsing the payload as trusted JSON, rather than being rejected the way a request with an *invalid* signature already was. This meant an attacker could POST a fabricated event directly — no signature forgery required, just omitting the header — and grant an arbitrary account an active subscription. Fixed so a missing header is rejected with the same `400` response as a forged one.
  * **`/sync-user` Privilege Escalation**: This endpoint granted service-level trust to any valid user access token based on which endpoint was being called, rather than verifying what the token actually claimed. Because the request body supplies the user ID, email, and name to sync, any authenticated user could have overwritten another user's subscription record. Fixed by requiring the actual service claims the Auth Service issues, and separately removing an unrelated token purpose (`email`) that had also been accepted here without justification.
  * **Duplicate Payment Records on Webhook Retry**: Stripe retries webhook delivery for up to three days on a non-2xx response. The payment-recording handlers inserted a new row on every delivery rather than checking whether the event had already been processed, so a single retried event produced duplicate payment records — and because those handlers also caught and swallowed their own exceptions before returning `200`, a genuine failure was acknowledged to Stripe and never retried, silently losing the payment record entirely. Both problems were fixed together via an events-processed table keyed on Stripe's event ID, checked before any handler runs.
  * **Hardcoded Admin Key Default**: `ADMIN_API_KEY` had a literal fallback value in source, which would have granted admin (coupon-generation) access to anyone who read the codebase if the real key was ever left unset in the environment. Removed, replaced with a startup check that refuses to boot without a real key of sufficient length.
  * **Silent Price ID Misconfiguration**: Stripe price ID lookups fell back to placeholder values that don't exist in Stripe, so a misconfiguration only surfaced as a failure at charge time — after a user had entered card details. Startup validation now refuses to boot on a missing or placeholder price ID, moving the failure from a customer-facing charge error to a deploy-time check.
  * **Coupon Generation Worker Hang**: Batch coupon generation with a prefix of 8 or more characters left zero characters for the randomized suffix, so every generated code was identical to the prefix and the uniqueness-check loop never terminated, hanging the Gunicorn worker handling the (admin-only) request. Fixed by capping the prefix length.

-----

*Created: July 4, 2025*
*Last Updated: August 27, 2026*
