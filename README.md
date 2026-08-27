# 🍽️Nourish Genie - Nutrition & Wellness Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![Kotlin](https://img.shields.io/badge/kotlin-%237F52FF.svg?style=flat&logo=kotlin&logoColor=white)](https://kotlinlang.org/)
[![Python](https://img.shields.io/badge/python-3670A0?style=flat&logo=python&logoColor=ffdd54)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Node.js](https://img.shields.io/badge/node.js-6DA55F?style=flat&logo=node.js&logoColor=white)](https://nodejs.org/)

## 📋 Table of Contents

- [🎯 Project Overview](#-project-overview)
- [🏗️ Architecture](#️-architecture)
- [🚀 Features](#-features)
- [🛠️ Technology Stack](#️-technology-stack)
- [📱 Frontend (Android)](#-frontend-android)
- [🖥️ Backend Services](#️-backend-services)
- [🗄️ Database Architecture](#️-database-architecture)
- [🔧 Setup & Installation](#-setup--installation)
- [📚 API Documentation](#-api-documentation)
- [🚢 Deployment](#-deployment)
- [🧪 Testing](#-testing)
- [📈 Monitoring & Analytics](#-monitoring--analytics)
- [🔒 Security](#-security)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

## 🎯 Project Overview

Nourish Genie is a comprehensive nutrition and wellness platform that empowers users to monitor their dietary habits, achieve health goals, and gain insights into their nutrition patterns. The application combines AI-powered food recognition, an achievement system, and subscription-based premium features.

### Key Objectives
- **Simplify Nutrition Tracking**: Use AI to identify food from photos and automatically log nutritional information
- **Personalized Health Goals**: Calculate custom nutrition targets based on user's profile and fitness objectives
- **Behavioral Insights**: Help users understand their eating patterns through logged food history (a dedicated analytics surface is not part of the current backend — see Roadmap)
- **Motivation Through Gamification**: Achievement system with badges (streaks and weekly challenges were part of an earlier design and are not in the current backend — see Roadmap)
- **Sustainable Business Model**: Freemium model with subscription-based premium features

### Target Audience
- Health-conscious individuals seeking to improve their nutrition
- People with specific dietary requirements or fitness goals
- Users who want detailed insights into their eating patterns
- Anyone looking to build healthy eating habits through gamification

## 🏗️ Architecture

### System Architecture Overview

```mermaid
graph TB
    subgraph "Client Layer"
        A[Android App - Kotlin/Compose]
        A2[iOS App - SwiftUI]
    end

    subgraph "Edge"
        CF[Cloudflare - DNS/CDN/WAF]
    end

    subgraph "API Gateway Layer"
        E[API Gateway - Node.js/Express]
    end

    subgraph "Microservices Layer"
        F[Auth Service - FastAPI/Python]
        G[User Service - Flask/Python]
        H[Image Service - Flask/Python]
        I[Subscription Service - Flask/Python]
        J[Email Service - Flask/Python]
    end

    subgraph "Data Layer"
        K[PostgreSQL]
        L[Redis - Caching/Sessions]
        M[MinIO - Object Storage]
    end

    subgraph "External Services"
        N[Vision AI Provider - Food Recognition]
        O[Stripe API - Payments]
        P[SMTP - Email Delivery]
    end

    A --> CF
    A2 --> CF
    CF --> E
    E --> F
    E --> G
    E --> H
    E --> I
    E --> J
    F --> K
    F --> L
    G --> K
    G --> M
    H --> L
    H --> N
    I --> K
    I --> O
    I -.->|webhook, bypasses gateway| O
    J --> L
    J --> P
```

**Notes on this diagram versus an idealized one**: `Subscription Service`'s `/webhook` route is called directly by Stripe against the service's own Railway URL rather than through the API Gateway (see the Subscription Service README for why), which is why that edge is drawn separately. Both the Android and iOS clients are shipped; there is no web client at the time of writing beyond a marketing/landing page served separately from the API surface.

### Design Principles

**Microservices Architecture**: Each service handles a specific domain with clear boundaries
- **Single Responsibility**: Each service has one clear purpose
- **Technology Diversity**: Services can use different tech stacks based on requirements
- **Independent Deployment**: Services can be deployed and scaled independently
- **Fault Isolation**: Failure in one service doesn't bring down the entire system

**Event-Driven Communication**: Services communicate through well-defined APIs and events
- **Asynchronous Processing**: Background tasks for heavy operations
- **Loose Coupling**: Services don't directly depend on each other's internal implementations
- **Scalability**: Individual services can be scaled based on demand

**Data Consistency**: Eventual consistency model with proper error handling
- **Database Per Service**: Each service owns its data
- **Saga Pattern**: For distributed transactions across services
- **CQRS**: Command Query Responsibility Segregation where appropriate

## 🚀 Features

### Core Features
- **📸 AI-Powered Food Recognition**: Take photos of meals for automatic nutritional analysis
- **📊 Personalized Nutrition Goals**: Custom calorie and macro targets based on user profile
- **🏆 Achievement System**: Badges awarded on food-logging activity, surfaced via the user's profile
- **📱 Intuitive Mobile Interface**: Modern, responsive design with smooth animations
- **🔄 Offline Support**: Local data storage with automatic sync when online

### Premium Features (Subscription)
- **🎯 Custom Goals**: Set specific targets beyond basic calorie counting
- **📞 Priority Support**: Direct access to nutrition experts and customer support

### Technical Features
- **🔐 OAuth Integration**: Google and Apple Sign-In support, with server-side ID token verification
- **💳 Subscription Management**: Stripe integration with trial periods and billing
- **📧 Email System**: Password reset codes today (see Email Service — other email types are designed for but not yet built)
- **🔄 Real-Time Sync**: Immediate data synchronization across devices
- **📱 Progressive Enhancement**: Graceful degradation for offline scenarios

## 🛠️ Technology Stack

### Frontend
| Technology | Purpose | Version |
|------------|---------|---------|
| **Kotlin** | Primary language | Latest |
| **Jetpack Compose** | Modern UI framework | Latest |
| **Android Architecture Components** | MVVM pattern, lifecycle management | Latest |
| **Dagger Hilt** | Dependency injection | 2.48+ |
| **Retrofit** | HTTP client | 2.9+ |
| **Room** | Local database | Latest |
| **Coroutines & Flow** | Asynchronous programming | Latest |
| **Material Design 3** | UI design system | Latest |

### Backend Services

#### Auth Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **FastAPI** | Web framework | 0.104+ |
| **Python** | Programming language | 3.11+ |
| **JWT** | Authentication tokens | Latest |
| **bcrypt** | Password hashing | Latest |
| **asyncpg** | PostgreSQL async driver | Latest |

#### User Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **Flask** | Web framework | 3.0+ |
| **Python** | Programming language | 3.11+ |
| **psycopg2** | PostgreSQL driver | Latest |
| **AI/LLM Provider** | Personalized meal suggestions (kept vendor-neutral) | — |
| **MinIO client (`minio`)** | Object storage for profile/food images | Latest |

#### Image Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **Flask** | Web framework | 3.0+ |
| **Python** | Programming language | 3.11+ |
| **Gunicorn** | WSGI server | Latest |
| **Vision AI Provider** | AI food recognition (kept vendor-neutral in this document) | — |
| **Redis** | Caching, rate/strike limiting | Latest |

#### Subscription Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **Flask** | Web framework | 3.0+ |
| **Python** | Programming language | 3.11+ |
| **Gunicorn** | WSGI server | Latest |
| **Stripe API** | Payment processing | Latest |
| **psycopg2** | PostgreSQL driver | Latest |

#### Email Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **Flask** | Web framework | 3.0+ |
| **Python** | Programming language | 3.11+ |
| **Gunicorn** | WSGI server | Latest |
| **SMTP (`smtplib`)** | Email delivery | - |
| **Redis** | Pub/Sub event delivery from other services | Latest |

### Infrastructure
| Technology | Purpose | Version |
|------------|---------|---------|
| **Docker** | Containerization (one image per service) | 24.0+ |
| **Docker Compose** | Local multi-container orchestration | 2.21+ |
| **Railway** | Container hosting and private networking (production) | — |
| **Cloudflare** | DNS, CDN, and WAF in front of the public domain | — |
| **PostgreSQL** | Primary database | 14+ |
| **Redis** | Caching, session/token state, and Pub/Sub | 6.2+ |
| **MinIO** | Object storage (S3-compatible) | Latest |

## 📱 Frontend (Android)

### Project Structure
```
android/
├── app/src/main/java/com/example/nourishgenie/
│   ├── core/                          # Core utilities and extensions
│   │   ├── logging/                   # Logging utilities
│   │   ├── network/                   # Network utilities
│   │   └── util/                      # General utilities
│   ├── data/                          # Data layer
│   │   ├── api/                       # API interfaces and implementations
│   │   ├── auth/                      # Authentication management
│   │   ├── database/                  # Local database (Room)
│   │   ├── model/                     # Data models and DTOs
│   │   ├── repository/                # Repository implementations
│   │   └── user/                      # User-specific data handling
│   ├── di/                            # Dependency injection modules
│   ├── domain/                        # Domain layer (business logic)
│   │   ├── model/                     # Domain models
│   │   ├── repository/                # Repository interfaces
│   │   └── usecase/                   # Use cases/interactors
│   ├── navigation/                    # Navigation setup and destinations
│   ├── ui/                            # UI layer
│   │   ├── components/                # Reusable UI components
│   │   ├── screens/                   # Screen implementations
│   │   │   ├── analytics/             # Analytics and reporting
│   │   │   ├── badges/                # Achievement system
│   │   │   ├── camera/                # Food photo capture
│   │   │   ├── dashboard/             # Main dashboard
│   │   │   ├── login/                 # Authentication screens
│   │   │   ├── meals/                 # Meal logging and history
│   │   │   ├── onboarding/            # User onboarding flow
│   │   │   ├── profile/               # User profile management
│   │   │   └── subscription/          # Subscription management
│   │   └── theme/                     # UI theming and styling
│   └── MainActivity.kt                # Main application entry point
├── build.gradle.kts                   # App-level build configuration
└── proguard-rules.pro                 # Code obfuscation rules
```

### Key Architecture Patterns

**MVVM (Model-View-ViewModel)**
- **Model**: Data layer with repositories and data sources
- **View**: Composable UI functions with state observation
- **ViewModel**: Business logic and state management

**Repository Pattern**
- Abstraction layer between ViewModels and data sources
- Handles data from multiple sources (API, local database, cache)
- Provides single source of truth for each data type

**Dependency Injection**
- Dagger Hilt for compile-time dependency injection
- Modular approach with separate modules for different concerns
- Testable architecture with easy mocking

### State Management
- **StateFlow/Flow**: Reactive data streams
- **Compose State**: UI state management
- **Room Database**: Local persistence with automatic sync

### Navigation
- **Jetpack Navigation Compose**: Type-safe navigation
- **Deep Linking**: Support for external links and notifications
- **Bottom Navigation**: Main app navigation pattern

> 📖 **Detailed Frontend Documentation**: [`/android/README.md`](./android/README.md)

## 🖥️ Backend Services

### Service Communication

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant Auth
    participant User
    participant Image
    participant Subscription
    participant AI as AI Provider(s)
    
    Client->>Gateway: POST /auth-service/login
    Gateway->>Auth: Forward request
    Auth-->>Gateway: JWT tokens
    Gateway-->>Client: Authentication response
    
    Client->>Gateway: GET /user-service/profile
    Gateway->>Gateway: Validate JWT
    Gateway->>User: Forward with user context
    User-->>Gateway: Profile data
    Gateway-->>Client: Profile response
    
    Client->>Gateway: POST /image-service/analyze-food
    Gateway->>Image: Forward with image
    Image->>AI: Vision AI request
    AI-->>Image: Food analysis
    Image-->>Gateway: Food analysis
    Gateway-->>Client: Analysis results

    Client->>Gateway: GET /user-service/food-suggestions
    Gateway->>User: Forward with user context
    User->>User: Infer cuisine from recent food_entries
    User->>AI: LLM request (goals, cuisine, recent meals)
    AI-->>User: Suggestions (or none, on failure)
    User-->>Gateway: Suggestions
    Gateway-->>Client: Suggestions
```

*Both the Image Service and the User Service call out to an AI provider independently — the Image Service for vision-based food recognition, the User Service for text-based meal suggestions. They are not routed through a shared internal AI gateway; each service makes its own call.*

### 1. Authentication Service

**Purpose**: Handles user authentication, authorization, and account management

**Key Features**:
- User registration and login
- OAuth integration (Google, Apple)
- JWT token management
- Password reset functionality
- Account security

**Technology**: FastAPI + Python + PostgreSQL + Redis

**Endpoints**:
```
POST   /login                 # User login
POST   /register              # User registration  
POST   /oauth/login           # OAuth authentication
POST   /refresh               # Token refresh
POST   /logout                # User logout
POST   /forgot-password       # Password reset request
POST   /reset-password        # Password reset confirmation
PUT    /change-password       # Password change
DELETE /delete-account        # Account deletion
GET    /profile               # User profile
```

**Database Schema**:
- `users`: User accounts and credentials
- `failed_login_attempts`: Security monitoring

> 📖 **Auth Service Documentation**: [`/auth_service/README.md`](auth_service.README.md)

### 2. User Service

**Purpose**: Manages user profiles, nutrition data, food logging, and AI-generated meal suggestions

**Key Features**:
- User profile management
- Food entry logging
- Nutrition goal calculation
- AI-powered meal suggestions, personalized by nutrition goals, dietary restrictions, and a cuisine preference inferred from the user's own food history
- Achievement system (badges awarded automatically on food-entry activity, surfaced via `GET /profile`)
- Onboarding flow

**Technology**: Flask + Python + PostgreSQL + MinIO + an AI/LLM provider (kept vendor-neutral, consistent with the Image Service)

**Endpoints**:
```
# Profile Management
GET    /profile               # Get user profile (includes stats and earned badges)
PUT    /profile               # Update profile
POST   /onboarding            # Save onboarding data

# Nutrition Management
POST   /food-entries          # Log food entry
GET    /food-entries          # Get food history
DELETE /food-entries/{id}     # Delete food entry
PUT    /nutrition-goals       # Update nutrition targets
GET    /nutrition-summary     # Nutrition summary for a date range
GET    /dashboard/summary     # Today's calorie/macro summary for a dashboard view

# AI Meal Suggestions
GET    /food-suggestions      # Personalized meal suggestions (?meal_type=breakfast|lunch|dinner|snack)

# Image Serving
GET    /images/{bucket}/{filename}  # Serve a stored image (profile picture, food photo)

# Health
GET    /health
```
*A standalone analytics/badges/streaks/challenges endpoint surface existed earlier in development and was removed during the backend hardening pass documented in this service's README; badge data is returned as part of `GET /profile` rather than through dedicated endpoints.*

**How meal suggestions are personalized**: recent food-entry history (up to 50 entries, last 30 days) is matched against a keyword set covering roughly 90 cuisines to infer a primary cuisine and a confidence score, which scales how strictly the AI provider is instructed to stick to that cuisine; the same recent-meal list is used to avoid suggesting repeats; and recipe instruction depth is scaled to real dish complexity (e.g. a multi-stage traditional dish isn't flattened into a generic 3-step recipe) rather than a fixed length. On an AI-provider failure, the endpoint returns an empty list with `200` rather than a `500` — full details in the User Service README.

**Database Schema**:
- `users`: User profile information (synced from Auth Service)
- `user_onboarding_data`: Onboarding questionnaire data
- `user_preferences`: User settings and nutrition goals
- `food_entries`: Food logging entries
- `user_badges`: Earned user badges

> 📖 **User Service Documentation**: [`/user_service/README.md`](user_service.README.md)

### 3. Image Service

**Purpose**: AI-powered food recognition and nutritional analysis from photos

**Key Features**:
- Food image analysis using a vision-language AI model
- Ingredient detection, recipe generation, and macro suggestions
- Server-side subscription enforcement on AI-spending endpoints
- Result caching, upload validation, and non-food rejection

**Technology**: Flask + Python + Vision AI Provider (kept vendor-neutral) + Redis

**Endpoints**:
```
POST   /analyze-food          # Analyze a food image
POST   /analyze-ingredients   # Detect ingredients in an image
POST   /generate-recipes      # Generate recipes (streamed, dual-accept auth)
POST   /macro-suggestions     # Suggest a macro breakdown
GET    /health                # Service health check
```

**Processing Flow**:
1. Image upload and validation (size limit, well-formed-image check)
2. Subscription/trial entitlement check
3. Vision AI provider processing
4. Nutritional data extraction
5. Result caching and return

> 📖 **Image Service Documentation**: [`/image_service/README.md`](image_service.README.md)

### 4. Subscription Service

**Purpose**: Manages user subscriptions, billing, and premium feature access

**Key Features**:
- Stripe integration for payment processing
- Subscription lifecycle management (trial, active, canceled)
- Promotional/coupon code redemption
- Idempotent Stripe webhook processing

**Technology**: Flask + Python + Stripe API + PostgreSQL

**Endpoints**:
```
GET    /plans                    # Available subscription plans
GET    /status                   # Current subscription status
POST   /start                    # Start a subscription or trial
POST   /cancel                   # Cancel subscription (active until period end)
POST   /payment-method           # Update default payment method
POST   /auto-renewal             # Enable/disable auto-renewal
POST   /sync-user                # Service-to-service: sync user from Auth Service
POST   /coupons/validate         # Validate a coupon code
POST   /coupons/redeem           # Redeem a coupon code
POST   /admin/coupons/generate   # Admin: generate coupon codes
POST   /webhook                  # Stripe webhooks (called directly by Stripe)
GET    /health                   # Service health check
```

**Subscription Tiers**:
- **Free**: Basic food logging
- **Premium (Monthly / Annual)**: Full feature access, gated server-side on `isSubscribed OR isInTrialPeriod`

> 📖 **Subscription Service Documentation**: [`/subscription_service/README.md`](subscription_service.README.md)

### 5. Email Service

**Purpose**: Sends transactional email, decoupled from the calling service via Redis Pub/Sub

**Key Features**:
- Password reset verification codes (currently the only implemented email type)
- Asynchronous dispatch via Redis Pub/Sub, so a slow or failing SMTP send doesn't block the caller
- A direct HTTP endpoint also exists for synchronous internal use, JWT-protected

**Technology**: Flask + Python + SMTP (`smtplib`)

**Endpoints**:
```
POST   /email-service/send-reset-code   # Internal, JWT-protected: send a password reset code
GET    /health                          # Service health check
```
**Primary path**: other services publish to the `password_reset_events` Redis channel rather than calling the HTTP endpoint directly; the Email Service consumes that channel in a background listener.

**Email Types**:
- Password reset (implemented)
- Welcome, subscription, and achievement notifications are not yet implemented — the service is designed to be extended for them (see its own README), but this document doesn't claim they exist yet.

> 📖 **Email Service Documentation**: [`/email_service/README.md`](email_service.README.md)

### 6. API Gateway

**Purpose**: Single entry point, request routing, authentication-aware rate limiting

**Key Features**:
- Request routing to appropriate services
- Per-user (JWT-keyed) rate limiting on authenticated routes, falling back to per-IP for unauthenticated ones
- Request/response logging
- CORS handling
- Shielding the Email Service from direct external access

**Technology**: Node.js + Express + http-proxy-middleware

**Configuration** (actual tiers, from `routes.js`):
- Auth (login/register): 20 req/15min
- Password reset (forgot/verify/reset): 5 req/hour
- Standard: 100 req/15min
- Email service: internal access only, not reachable externally

> 📖 **API Gateway Documentation**: [`/api-gateway/README.md`](api-gateway.README.md)

## 🗄️ Database Architecture

### Database Per Service Pattern

Each microservice owns a logically separate database (own tables, own migrations, no cross-service foreign keys), which is what the local development setup below reflects:

```mermaid
graph TB
    subgraph "Auth Service"
        A[authdb - PostgreSQL]
    end
    
    subgraph "User Service"  
        B[userdb - PostgreSQL]
    end
    
    subgraph "Subscription Service"
        C[subscriptiondb - PostgreSQL]
    end
    
    subgraph "Shared Infrastructure"
        D[Redis - Caching/Sessions/Pub-Sub]
        E[MinIO - Object Storage]
    end
```

**Production note**: On Railway, these logically-separate databases currently run as a single shared Postgres instance (Railway's default single-Postgres-per-project setup) rather than as physically separate database servers — each service still owns its own tables and doesn't read another service's tables directly, but the physical isolation implied by the diagram above is a local-development simplification, not the current production topology. Splitting into per-service Postgres instances is a reasonable follow-up rather than something already done.

### Data Consistency Strategy

**Eventual Consistency**: Services communicate through service-to-service API calls rather than a shared database
- User creation syncs from Auth Service to User Service and Subscription Service (`/sync-user`, service-authenticated — see the Auth and Subscription Service READMEs for a vulnerability found and fixed in this exact path)
- Food entries trigger badge-award checks within the same request in User Service

### Database Schemas

#### Auth Database (`authdb`)
```sql
-- User authentication and security
users (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255),              -- nullable: OAuth accounts have none
    name VARCHAR(255),
    auth_provider VARCHAR(50) DEFAULT 'email',
    oauth_provider_id VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE,
    updated_at TIMESTAMP WITH TIME ZONE,
    last_login TIMESTAMP WITH TIME ZONE
)
-- password is nullable via a migration-added conditional CHECK constraint:
-- required for auth_provider = 'email', optional otherwise.

-- Security monitoring
failed_login_attempts (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    ip_address VARCHAR(45),
    attempt_time TIMESTAMP WITH TIME ZONE
)
```

#### User Database (`userdb`)
```sql
-- See user_service/init_db.py for the complete schema.
-- Key tables: users, user_onboarding_data, user_preferences,
-- food_entries, user_badges.
```

#### Subscription Database (`subscriptiondb`)
```sql
-- Managed by init_db.py / run_migrations.py (Flask + psycopg2, not an ORM).
-- Key tables: users, user_payment_profiles, payment_methods, subscriptions,
-- subscription_payments, processed_webhook_events, promo_codes,
-- promo_code_redemptions.
```

### Data Migration Strategy

**Database Versioning**: Each service maintains its own migration scripts
- Auth Service and Subscription Service use a custom Python migration runner (`run_migrations.py`) with a Postgres advisory lock, so concurrent container starts on the same deploy don't race to apply the same migration twice
- User Service and Email Service currently rely on `init_db.py`'s `CREATE TABLE IF NOT EXISTS` baseline rather than a separate versioned migration runner
- Image Service has no persistent schema of its own (Redis-only)
- The container startup command chains schema setup and the app server with `&&`, not `;`, so a failed migration blocks the deploy rather than starting the app against a schema it doesn't match — this was a specific fix made during the hardening pass covered in the Auth and Subscription Service READMEs

## 🔧 Setup & Installation

> **Note**: The steps below set up a local, self-hosted stack via Docker Compose for development. The actual production deployment (Railway + Cloudflare) is described in the [Deployment](#-deployment) section and differs in some specifics — notably, production currently runs all services against a single shared Postgres instance rather than the per-service containers shown here (see [Database Architecture](#️-database-architecture)).

### Prerequisites

- **Docker**: 24.0+ and Docker Compose 2.21+
- **Node.js**: 18+ (for API Gateway)
- **Python**: 3.11+ (for the five Python backend services)
- **Android Studio**: Latest version (for Android development)
- **Xcode**: Latest version (for iOS development — not covered in detail in this document)

### Quick Start (Docker)

1. **Clone the Repository**
   ```bash
   git clone https://github.com/Oladapo01/FoodTrackerApp.git
   cd FoodTrackerApp
   ```

2. **Set Up Environment Variables**
   ```bash
   # Copy environment templates
   cp .env.example .env
   cp config/auth-service.env.example config/auth-service.env
   cp config/user-service.env.example config/user-service.env
   # ... repeat for all services
   
   # Edit configuration files with your values
   ```

3. **Set Up Secrets**
   ```bash
   # Create secrets directory
   mkdir -p secrets
   
   # Generate secure passwords and API keys
   echo "your-secure-db-password" > secrets/db_password.txt
   echo "your-jwt-secret-key" > secrets/jwt_secret.txt
   echo "your-redis-password" > secrets/redis_password.txt
   echo "your-vision-api-key" > secrets/vision_api_key.txt
   echo "your-stripe-api-key" > secrets/stripe_api_key.txt
   echo "your-minio-access-key" > secrets/minio_access_key.txt
   echo "your-minio-secret-key" > secrets/minio_secret_key.txt
   echo "your-smtp-username" > secrets/smtp_username.txt
   echo "your-smtp-password" > secrets/smtp_password.txt
   ```

4. **Start the Local Stack**
   ```bash
   # Start all services via Docker Compose
   docker-compose -f docker-compose.production.yml up -d
   
   # Check service status
   docker-compose -f docker-compose.production.yml ps
   
   # View logs
   docker-compose -f docker-compose.production.yml logs -f
   ```

5. **Initialize Databases**
   ```bash
   # Databases are automatically initialized on first run
   # Check initialization logs
   docker-compose logs postgres
   ```

6. **Verify Installation**
   ```bash
   # Test API Gateway health
   curl http://localhost:8080/health
   
   # Test individual services
   curl http://localhost:8080/auth-service/health
   curl http://localhost:8080/user-service/health
   curl http://localhost:8080/image-service/health
   curl http://localhost:8080/subscription-service/health
   ```

### Development Setup

#### Backend Development
```bash
# Set up Python virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies for the Python services (all five backend services)
pip install -r auth_service/requirements.txt
pip install -r user_service/requirements.txt
pip install -r image_service/requirements.txt
pip install -r subscription_service/requirements.txt
pip install -r email_service/requirements.txt

# Install Node.js dependencies for API Gateway
cd api-gateway
npm install
cd ..

# Start development services
docker-compose -f docker-compose.dev.yml up -d
```

#### Android Development
```bash
# Open Android Studio
# Import the android/ directory as a project
# Sync Gradle files
# Update local.properties with SDK path

# Build and run
./gradlew assembleDebug
./gradlew installDebug
```

### Environment Configuration

#### Required Environment Variables

**Database Configuration**:
```env
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD_FILE=/run/secrets/db_password
```

**Redis Configuration**:
```env
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD_FILE=/run/secrets/redis_password
```

**External API Keys**:
```env
VISION_API_KEY_FILE=/run/secrets/vision_api_key
STRIPE_API_KEY_FILE=/run/secrets/stripe_api_key
```

**Email Configuration**:
```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME_FILE=/run/secrets/smtp_username
SMTP_PASSWORD_FILE=/run/secrets/smtp_password
SENDER_EMAIL=noreply@nourishgenie.co.uk
```

## 📚 API Documentation

### Authentication

All API requests (except authentication and OAuth endpoints) require a valid JWT token:

```bash
# Include in Authorization header
Authorization: Bearer <jwt_token>

# Example authenticated request
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
     http://localhost:8080/user-service/profile
```

### API Response Format

There is no single uniform response envelope across services — each was documented independently rather than against a shared schema, so response shapes vary. Two patterns recur:

**A simple message body**, common for actions:
```json
{ "message": "Password reset successful" }
```

**A stable error slug plus a human-readable message**, used where the client needs to branch on the error type rather than just display it (see the Image and Subscription Service READMEs for the full list of these):
```json
{ "error": "subscription_required", "message": "Subscribe to continue analysing meals." }
```

### Rate Limiting

The table below reflects the API Gateway's actual configured tiers (see `routes.js`):

| Endpoint Type | Rate Limit | Window |
|---------------|------------|---------|
| Auth (login/register) | 20 requests | 15 minutes |
| Password Reset | 5 requests | 1 hour |
| Standard API | 100 requests | 15 minutes |
| Email Service | Not externally reachable | — |

Authenticated routes are rate-limited per user (keyed on the verified JWT's user ID), not per IP — see the API Gateway README for why that distinction mattered in practice.

### API Documentation Links

- **Auth Service**: FastAPI auto-generates interactive docs at `/docs` when run locally (`http://localhost:8081/docs`).
- The other four backend services are Flask-based and don't auto-generate OpenAPI docs; each service's own README under this repository is the authoritative endpoint reference.
- **Postman Collection**: [`/docs/api/nourishgenie.postman_collection.json`](./docs/api/nourishgenie.postman_collection.json)

## 🚢 Deployment

### Production Deployment

The system is deployed on **Railway**, with **Cloudflare** in front of the public domain and **Names.co.uk** as the domain registrar for `nourishgenie.co.uk`.

```mermaid
graph TB
    U[User] --> CF[Cloudflare - DNS / CDN / WAF]
    CF --> WEB[web - Railway service]
    CF --> GW[api-gateway - Railway service]

    GW --> AUTH[auth-service]
    GW --> USER[user-service]
    GW --> IMG[image-service]
    GW --> SUB[subscription-service]

    AUTH --> PG[(PostgreSQL - Railway)]
    USER --> PG
    SUB --> PG
    AUTH --> RD[(Redis - Railway)]
    IMG --> RD
    SUB --> RD
    EMAIL[email-service] --> RD

    STRIPE[Stripe] -.->|webhook, direct to service URL, bypasses gateway| SUB
```

**What this reflects, specifically**:

- Each service (`api-gateway`, `auth-service`, `user-service`, `image-service`, `subscription-service`, `email-service`, plus a `web` front end) runs as its own Railway service, built from its own Dockerfile, and communicates with the others over Railway's **private network — which is IPv6-only**. This is not incidental trivia: an IPv4-only DNS readiness check in the Auth Service's startup script caused startup crash loops on Railway specifically because of this, and had to be removed (see the Auth Service README's Production Hardening section).
- Postgres and Redis are shared Railway-managed instances rather than one per service — see the note in [Database Architecture](#️-database-architecture).
- Stripe's webhook calls the Subscription Service's Railway URL directly rather than going through the API Gateway, which meant that route didn't benefit from the Gateway's rate limiting and needed its own hardening (signature verification, idempotency) — covered in the Subscription Service README.
- Each service's container startup command chains schema setup and the app server with `&&` rather than `;`, so a failed migration or schema step blocks the deploy instead of starting the app against a database it doesn't match.
- Deployment happens via Railway's own build/deploy pipeline (build from each service's Dockerfile on push); there is no separate Kubernetes, Docker Swarm, or multi-cloud deployment path in use, and none is claimed here.

### Local / Self-Hosted Deployment

The `docker-compose.production.yml` setup described in [Setup & Installation](#-setup--installation) is for running the full stack locally or self-hosting outside Railway — it is not the configuration actually running in production, though it exercises the same Docker images.

### Monitoring and Logging

**Current state**: Each service logs structured messages to stdout/stderr (see each service's own README, "Monitoring and Troubleshooting" section, for what to look for) and exposes a `/health` endpoint reporting its own dependency connectivity (database, Redis, MinIO, Stripe, or the vision AI provider, as applicable). Log aggregation and metrics visualization tooling (e.g. Prometheus/Grafana or a hosted equivalent) is not yet in place — this is a genuine gap rather than an implemented feature, noted here rather than glossed over.

**Health Checks**:
- Per-service `/health` endpoints, used for container readiness
- Dependency connectivity checks embedded in `/health` (database, Redis, MinIO, Stripe, vision AI provider)

## 🧪 Testing

### Testing Strategy

#### Unit Tests
- **Backend**: pytest (all five Python services), Jest (API Gateway, Node.js)
- **Android**: JUnit, Mockk, Espresso
- **Coverage Target**: 80%+ code coverage

#### Integration Tests
- **API Testing**: Postman/Newman
- **Service Communication**: Contract testing

#### End-to-End Tests
- **Android UI**: Espresso, UI Automator
- **API Workflows**: Complete user journey testing
- **Performance Testing**: Load testing with Artillery/JMeter

### Running Tests

```bash
# Backend unit tests
cd auth_service && python -m pytest tests/
cd user_service && python -m pytest tests/
cd image_service && python -m pytest tests/
cd subscription_service && python -m pytest tests/
cd email_service && python -m pytest tests/

# API Gateway tests
cd api-gateway && npm test

# Android tests
cd android && ./gradlew test
cd android && ./gradlew connectedAndroidTest

# Integration tests
docker-compose -f docker-compose.test.yml up --abort-on-container-exit

# Performance tests
artillery run tests/performance/load-test.yml
```

## 📈 Monitoring & Analytics

### Application Metrics

**Key Performance Indicators**:
- **Response Time**: Service response latencies
- **Throughput**: Requests per second
- **Error Rate**: 4xx/5xx error percentages
- **Availability**: Uptime monitoring

**Business Metrics**:
- **User Engagement**: Daily/Monthly active users
- **Feature Usage**: Food logging frequency
- **Conversion**: Trial to subscription conversion rates
- **Retention**: User retention rates

### Alerting Rules (Illustrative)

The rules below are the kind of alerting this metric set would support with Prometheus in place; per the note in [Deployment](#-deployment), that tooling isn't deployed yet, so this is a target configuration rather than a running one.

```yaml
# prometheus/alerts.yml
groups:
  - name: nourish-genie-alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        annotations:
          summary: "High error rate detected"
      
      - alert: DatabaseConnectionFailure
        expr: up{job="postgres"} == 0
        for: 1m
        annotations:
          summary: "Database connection lost"
```

### Performance Optimization

**Database Optimization**:
- Query optimization and indexing
- Connection pooling
- Partitioning for large tables

**Caching Strategy**:
- Redis for session storage
- Application-level caching
- CDN for static assets
- API response caching

**Service Optimization**:
- Asynchronous processing for heavy operations
- Connection pooling for external APIs
- Resource-based auto-scaling
- Code profiling and optimization

## 🔒 Security

This section describes the security principles the system follows. For the specific vulnerabilities found and fixed to get here — with the reasoning behind each fix — see the **Production Hardening & Incident History** section in each service's own README (Auth, User, Image, Subscription, and API Gateway).

### Security Measures

#### Authentication & Authorization
- **JWT Tokens**: Secure, stateless authentication
- **OAuth2**: Google and Apple Sign-In integration, with the ID token verified server-side against the provider's JWKS before a session is issued
- **Subscription-Gated Features**: Server-side entitlement checks (`isSubscribed OR isInTrialPeriod`) on premium/AI-spending endpoints, not just client-side gating
- **Token Rotation**: Refresh-token-based session renewal

#### API Security
- **Rate Limiting**: Per-user (JWT-keyed) on authenticated routes, per-IP on unauthenticated ones, plus a stricter tier on auth and password-reset endpoints
- **Input Validation**: Type and range validation on client-supplied data, with a shared validation-error path returning `400` rather than an unhandled exception surfacing as `500`
- **CORS Configuration**: Restricted rather than wide-open (`CORS(app)` allowing any origin was found and removed in at least one service — see Subscription Service)
- **HTTPS Only**: TLS terminated at Cloudflare in front of the public domain

#### Data Security
- **Password Hashing**: bcrypt
- **Secrets Management**: Docker secrets, with several services refusing to start if a critical secret is missing, too short, or a known placeholder value
- **Database Security**: Parameterized queries throughout — verified by audit in at least one service (see User Service) rather than assumed

#### Infrastructure Security
- **Network Isolation**: Services communicate over Railway's private network rather than the public internet
- **Constant-Time Comparisons**: Admin and internal-service secrets are compared with `hmac.compare_digest` rather than `==`, to avoid a timing side-channel

### Security Best Practices

1. **Regular Security Updates**: Recommended practice; automated dependency scanning (e.g. Dependabot) is not yet configured for this project — noted here rather than claimed as done
2. **Incident Response**: Security incidents should be reported to `security@nourishgenie.co.uk`
3. **Data Privacy**: GDPR compliance and data protection measures (see below)

### Data Privacy & Compliance

**GDPR Compliance**:
- User consent management
- Right to data portability
- Right to be forgotten
- Data processing transparency

**Data Retention**:
- Automatic data cleanup policies
- User data export capabilities
- Secure data deletion procedures

## 🤝 Contributing

### Development Workflow

1. **Fork the Repository**
2. **Create Feature Branch**: `git checkout -b feature/your-feature-name`
3. **Make Changes**: Follow coding standards and write tests
4. **Run Tests**: Ensure all tests pass
5. **Submit Pull Request**: Include detailed description and test results

### Coding Standards

#### Backend (Python)
- **PEP 8**: Python style guide compliance
- **Type Hints**: Use type annotations
- **Docstrings**: Document all functions and classes
- **Error Handling**: Comprehensive exception handling

#### Android (Kotlin)
- **Kotlin Coding Conventions**: Official Kotlin style guide
- **Compose Guidelines**: Follow Jetpack Compose best practices
- **Architecture**: Maintain MVVM pattern
- **Testing**: Write unit and integration tests

### Code Review Guidelines

1. **Functionality**: Code works as intended
2. **Testing**: Adequate test coverage
3. **Performance**: No significant performance degradation
4. **Security**: No security vulnerabilities introduced
5. **Documentation**: Code is well-documented
6. **Style**: Follows project coding standards

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/Oladapo01/FoodTrackerApp/blob/main/LICENSE) file for details.

### Third-Party Licenses

- **Vision AI Provider**: Subject to the provider's Terms of Service (kept vendor-neutral in this document)
- **Stripe API**: Subject to Stripe Terms of Service
- **Android Components**: Apache License 2.0
- **FastAPI**: MIT License

---

## 📞 Support & Contact

- **Documentation Issues**: Create an issue in this repository
- **Bug Reports**: Use the bug report template
- **Feature Requests**: Use the feature request template
- **Security Issues**: security@nourishgenie.co.uk
- **Privacy**: privacy@nourishgenie.co.uk
- **General Contact**: contact@nourishgenie.co.uk
- **User Support**: support@nourishgenie.co.uk / help@nourishgenie.co.uk
- **Admin**: admin@nourishgenie.co.uk

## 🗺️ Roadmap

### Current Version (v1.0)
- ✅ Basic food logging
- ✅ AI-powered food recognition
- ✅ AI-powered, cuisine-aware, calorie-aware meal suggestions
- ✅ Achievement system
- ✅ Subscription management
- ✅ Android and iOS clients

### Upcoming Features (v1.1)
- 🔄 Weight tracking integration
- 🔄 Meal planning capabilities
- 🔄 Social features and sharing
- 🔄 Advanced export options

### Future Releases (v2.0+)
- 📅 Web app
- 📅 Analytics and reporting surface
- 📅 Integration with fitness trackers
- 📅 Nutritionist consultation features

---

*Created: July 4, 2025*
*Last Updated: August 27, 2026*
*Version: 1.0.0*
