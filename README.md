# 🍽️Nourish Genie - Nutrition & Wellness Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![Kotlin](https://img.shields.io/badge/kotlin-%237F52FF.svg?style=flat&logo=kotlin&logoColor=white)](https://kotlinlang.org/)
[![Python](https://img.shields.io/badge/python-3670A0?style=flat&logo=python&logoColor=ffdd54)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Spring Boot](https://img.shields.io/badge/spring-%236DB33F.svg?style=flat&logo=spring&logoColor=white)](https://spring.io/projects/spring-boot)
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

Nourish Genie is a comprehensive nutrition and wellness platform that empowers users to monitor their dietary habits, achieve health goals, and gain insights into their nutrition patterns. The application combines AI-powered food recognition, real-time analytics, gamification through achievements, and subscription-based premium features.

### Key Objectives
- **Simplify Nutrition Tracking**: Use AI to identify food from photos and automatically log nutritional information
- **Personalized Health Goals**: Calculate custom nutrition targets based on user's profile and fitness objectives
- **Behavioral Insights**: Provide detailed analytics and trends to help users understand their eating patterns
- **Motivation Through Gamification**: Achievement system with badges, streaks, and weekly challenges
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
        B[Future iOS App]
        C[Future Web App]
    end
    
    subgraph "API Gateway Layer"
        D[Nginx Load Balancer]
        E[API Gateway - Node.js/Express]
    end
    
    subgraph "Microservices Layer"
        F[Auth Service - FastAPI/Python]
        G[User Service - Flask/Python]
        H[Image Service - Spring Boot/Java]
        I[Subscription Service - Spring Boot/Java]
        J[Email Service - FastAPI/Python]
    end
    
    subgraph "Data Layer"
        K[PostgreSQL - Multiple DBs]
        L[Redis - Caching/Sessions]
        M[MinIO - Object Storage]
    end
    
    subgraph "External Services"
        N[OpenAI API - Food Recognition]
        O[Stripe API - Payments]
        P[SMTP - Email Delivery]
    end
    
    A --> D
    B --> D
    C --> D
    D --> E
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
    J --> L
    J --> P
```

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
- **📈 Real-Time Analytics**: Detailed insights into nutrition patterns and trends
- **🏆 Achievement System**: Badges, streaks, and challenges to motivate healthy habits
- **📱 Intuitive Mobile Interface**: Modern, responsive design with smooth animations
- **🔄 Offline Support**: Local data storage with automatic sync when online

### Premium Features (Subscription)
- **📋 Advanced Analytics**: Export data, detailed reports, and historical trends
- **🎯 Custom Goals**: Set specific targets beyond basic calorie counting
- **💪 Integration Support**: Connect with fitness trackers and health apps
- **👥 Multi-User Profiles**: Family accounts and sharing capabilities
- **📞 Priority Support**: Direct access to nutrition experts and customer support

### Technical Features
- **🔐 OAuth Integration**: Google and Apple Sign-In support
- **💳 Subscription Management**: Stripe integration with trial periods and billing
- **📧 Email System**: Welcome emails, password resets, and notifications
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
| **Pillow** | Image processing | Latest |

#### Image Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **Spring Boot** | Web framework | 3.1+ |
| **Java** | Programming language | 17+ |
| **OpenAI API** | AI food recognition | v1 |
| **Redis** | Caching | Latest |

#### Subscription Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **Spring Boot** | Web framework | 3.1+ |
| **Java** | Programming language | 17+ |
| **Stripe API** | Payment processing | Latest |
| **JPA/Hibernate** | ORM | Latest |

#### Email Service
| Technology | Purpose | Version |
|------------|---------|---------|
| **FastAPI** | Web framework | 0.104+ |
| **Python** | Programming language | 3.11+ |
| **SMTP** | Email delivery | - |
| **Jinja2** | Email templating | Latest |

### Infrastructure
| Technology | Purpose | Version |
|------------|---------|---------|
| **Docker** | Containerization | 24.0+ |
| **Docker Compose** | Multi-container orchestration | 2.21+ |
| **Nginx** | Load balancer & reverse proxy | 1.25+ |
| **PostgreSQL** | Primary database | 14+ |
| **Redis** | Caching & session storage | 6.2+ |
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
    Image->>Image: Process with OpenAI
    Image-->>Gateway: Food analysis
    Gateway-->>Client: Analysis results
```

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

**Purpose**: Manages user profiles, nutrition data, food logging, and analytics

**Key Features**:
- User profile management
- Food entry logging
- Nutrition goal calculation
- Analytics and reporting
- Achievement system
- Onboarding flow

**Technology**: Flask + Python + PostgreSQL + MinIO

**Endpoints**:
```
# Profile Management
GET    /profile               # Get user profile
PUT    /profile               # Update profile
POST   /onboarding           # Save onboarding data

# Nutrition Management  
POST   /food-entries         # Log food entry
GET    /food-entries         # Get food history
DELETE /food-entries/{id}    # Delete food entry
PUT    /nutrition-goals      # Update nutrition targets

# Analytics
GET    /analytics/overview        # Overview statistics
GET    /analytics/calorie-trend   # Calorie trend data
GET    /analytics/macro-distribution # Macro breakdown
GET    /analytics/detailed        # Detailed analytics
POST   /analytics/export          # Export data

# Achievements
GET    /badges                    # All badges with progress
GET    /achievements/recent       # Recent achievements
GET    /dashboard/goals          # Today's goals
GET    /challenges/weekly        # Weekly challenges
GET    /statistics/badges        # Badge statistics
GET    /streaks                  # User streak data
POST   /goals/complete           # Mark goal complete
POST   /challenges/claim         # Claim challenge reward

# Dashboard
GET    /dashboard/summary        # Dashboard overview
GET    /nutrition-summary        # Nutrition summary
```

**Database Schema**:
- `users`: User profile information
- `user_onboarding_data`: Onboarding questionnaire data
- `user_preferences`: User settings and nutrition goals
- `food_entries`: Food logging entries
- `daily_nutrition_summaries`: Daily nutrition aggregates
- `badge_definitions`: Available badges and requirements
- `user_badges`: Earned user badges
- `user_streaks`: Logging streak tracking
- `daily_goals`: Daily goal tracking
- `weekly_challenges`: Challenge definitions
- `user_weekly_challenges`: User challenge participation
- `user_statistics`: User analytics data

> 📖 **User Service Documentation**: [`/user_service/README.md`](user_service.README.md)

### 3. Image Service

**Purpose**: AI-powered food recognition and nutritional analysis from photos

**Key Features**:
- Food image analysis using OpenAI Vision API
- Nutritional information extraction
- Confidence scoring
- Result caching
- Batch processing capabilities

**Technology**: Spring Boot + Java + OpenAI API + Redis

**Endpoints**:
```
POST   /analyze-food          # Analyze food image
GET    /analysis/{id}         # Get analysis results
POST   /batch-analyze         # Batch image processing
GET    /health                # Service health check
```

**Processing Flow**:
1. Image upload and validation
2. OpenAI Vision API processing
3. Nutritional data extraction
4. Confidence scoring
5. Result caching and return

> 📖 **Image Service Documentation**: [`/image_service/README.md`](image_service.README.md)

### 4. Subscription Service

**Purpose**: Manages user subscriptions, billing, and premium feature access

**Key Features**:
- Stripe integration for payment processing
- Subscription lifecycle management
- Trial period handling
- Premium feature gating
- Billing history and invoices

**Technology**: Spring Boot + Java + Stripe API + PostgreSQL

**Endpoints**:
```
GET    /status                # Subscription status
POST   /start                 # Start subscription
POST   /cancel                # Cancel subscription
POST   /update                # Update subscription
POST   /sync-user             # Sync user data
GET    /billing-history       # Billing history
POST   /webhook               # Stripe webhooks
```

**Subscription Tiers**:
- **Free**: Basic food logging, limited analytics
- **Premium Monthly**: Full analytics, export, advanced features
- **Premium Yearly**: All premium features with discount

> 📖 **Subscription Service Documentation**: [`/subscription_service/README.md`](subscription_service.README.md)

### 5. Email Service

**Purpose**: Handles all email communications and notifications

**Key Features**:
- Password reset emails
- Welcome emails
- Subscription notifications
- Achievement notifications
- Template-based email system

**Technology**: FastAPI + Python + SMTP + Jinja2

**Endpoints**:
```
POST   /send-email            # Send email
POST   /send-template         # Send templated email
GET    /templates             # Available templates
GET    /health                # Service health check
```

**Email Templates**:
- Welcome email
- Password reset
- Subscription confirmation
- Achievement notifications
- Weekly summaries

> 📖 **Email Service Documentation**: [`/email_service/README.md`](email_service.README.md)

### 6. API Gateway

**Purpose**: Single entry point, request routing, authentication, and rate limiting

**Key Features**:
- Request routing to appropriate services
- JWT token validation
- Rate limiting and DDoS protection
- Request/response logging
- CORS handling
- Load balancing

**Technology**: Node.js + Express + http-proxy-middleware

**Configuration**:
- Authentication endpoints: Stricter rate limiting (20 req/15min)
- OAuth endpoints: Medium rate limiting (30 req/15min)
- Standard endpoints: Normal rate limiting (100 req/15min)
- Email service: Internal access only

> 📖 **API Gateway Documentation**: [`/api-gateway/README.md`](api-gateway.README.md)

## 🗄️ Database Architecture

### Database Per Service Pattern

Each microservice maintains its own database to ensure loose coupling and independent deployments:

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
        D[Redis - Caching/Sessions]
        E[MinIO - Object Storage]
    end
```

### Data Consistency Strategy

**Eventual Consistency**: Services communicate through events and APIs to maintain data consistency
- User creation triggers sync across services
- Subscription changes update user permissions
- Food entries trigger achievement calculations

**Compensating Transactions**: Handle failures in distributed operations
- User registration rollback if subscription sync fails
- Achievement revocation if data is corrected

### Database Schemas

#### Auth Database (`authdb`)
```sql
-- User authentication and security
users (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255),
    name VARCHAR(255),
    auth_provider VARCHAR(50) DEFAULT 'email',
    oauth_provider_id VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE,
    updated_at TIMESTAMP WITH TIME ZONE,
    last_login TIMESTAMP WITH TIME ZONE
)

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
-- Core user data and nutrition tracking
-- See user_service/init_db.py for complete schema
-- Key tables: users, food_entries, user_preferences, 
-- badge_definitions, user_badges, daily_goals, etc.
```

#### Subscription Database (`subscriptiondb`)
```sql
-- Subscription and billing management
-- Managed by Spring Boot JPA entities
-- Key entities: User, Subscription, PaymentMethod, Invoice
```

### Data Migration Strategy

**Database Versioning**: Each service maintains its own migration scripts
- Flyway for Java services (Image, Subscription)
- Custom Python scripts for Python services (Auth, User, Email)
- Version-controlled schema changes

**Backward Compatibility**: All schema changes maintain backward compatibility
- New columns are nullable or have defaults
- Deprecated columns are marked but not immediately dropped
- API versioning supports multiple schema versions

## 🔧 Setup & Installation

### Prerequisites

- **Docker**: 24.0+ and Docker Compose 2.21+
- **Node.js**: 18+ (for API Gateway)
- **Python**: 3.11+ (for Python services)
- **Java**: 17+ (for Java services)
- **Android Studio**: Latest version (for Android development)

### Quick Start (Docker)

1. **Clone the Repository**
   ```bash
   git clone https://github.com/your-org/nourishgenie.git
   cd nourish-genie
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
   echo "your-openai-api-key" > secrets/openai_api_key.txt
   echo "your-stripe-api-key" > secrets/stripe_api_key.txt
   echo "your-minio-access-key" > secrets/minio_access_key.txt
   echo "your-minio-secret-key" > secrets/minio_secret_key.txt
   echo "your-smtp-username" > secrets/smtp_username.txt
   echo "your-smtp-password" > secrets/smtp_password.txt
   ```

4. **Start the Infrastructure**
   ```bash
   # Start all services in production mode
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
   curl http://localhost:81/health
   
   # Test individual services
   curl http://localhost:81/auth-service/health
   curl http://localhost:81/user-service/health
   curl http://localhost:81/image-service/health
   curl http://localhost:81/subscription-service/health
   curl http://localhost:81/email-service/health
   ```

### Development Setup

#### Backend Development
```bash
# Set up Python virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies for Python services
pip install -r auth_service/requirements.txt
pip install -r user_service/requirements.txt
pip install -r email_service/requirements.txt

# Set up Java development environment
# Ensure Java 17+ and Maven are installed

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
OPENAI_API_KEY_FILE=/run/secrets/openai_api_key
STRIPE_API_KEY_FILE=/run/secrets/stripe_api_key
```

**Email Configuration**:
```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME_FILE=/run/secrets/smtp_username
SMTP_PASSWORD_FILE=/run/secrets/smtp_password
SENDER_EMAIL=noreply@foodtracker.example.com
```

## 📚 API Documentation

### Authentication

All API requests (except authentication endpoints) require a valid JWT token:

```bash
# Include in Authorization header
Authorization: Bearer <jwt_token>

# Example authenticated request
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
     http://localhost:81/user-service/profile
```

### API Response Format

**Success Response**:
```json
{
  "data": { /* response data */ },
  "message": "Success",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Error Response**:
```json
{
  "error": "Error description",
  "code": "ERROR_CODE",
  "timestamp": "2024-01-15T10:30:00Z",
  "details": { /* additional error details */ }
}
```

### Rate Limiting

| Endpoint Type | Rate Limit | Window |
|---------------|------------|---------|
| Authentication | 20 requests | 15 minutes |
| OAuth | 30 requests | 15 minutes |
| Password Reset | 5 requests | 1 hour |
| Standard API | 100 requests | 15 minutes |

### API Documentation Links

- **OpenAPI/Swagger Docs**: Available at service endpoints
  - Auth Service: `http://localhost:8081/docs`
  - User Service: `http://localhost:8082/docs` (if FastAPI)
  - Image Service: `http://localhost:8083/swagger-ui.html`
  - Subscription Service: `http://localhost:8084/swagger-ui.html`
  - Email Service: `http://localhost:8085/docs`

- **Postman Collection**: [`/docs/api/food-tracker.postman_collection.json`](./docs/api/food-tracker.postman_collection.json)

## 🚢 Deployment

### Production Deployment Architecture

```mermaid
graph TB
    subgraph "Load Balancer"
        LB[Nginx/HAProxy]
    end
    
    subgraph "Application Tier"
        API1[API Gateway 1]
        API2[API Gateway 2]
        AUTH1[Auth Service 1]
        AUTH2[Auth Service 2]
        USER1[User Service 1]
        USER2[User Service 2]
    end
    
    subgraph "Data Tier"
        DB[(PostgreSQL Cluster)]
        REDIS[(Redis Cluster)]
        MINIO[(MinIO Cluster)]
    end
    
    LB --> API1
    LB --> API2
    API1 --> AUTH1
    API1 --> USER1
    API2 --> AUTH2
    API2 --> USER2
    AUTH1 --> DB
    AUTH2 --> DB
    USER1 --> DB
    USER2 --> DB
```

### Deployment Options

#### 1. Docker Swarm Deployment
```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.production.yml nourish-genie

# Scale services
docker service scale nourish-genie_auth-service=3
docker service scale nourish-genie_user-service=3
```

#### 2. Kubernetes Deployment
```bash
# Apply Kubernetes manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmaps/
kubectl apply -f k8s/secrets/
kubectl apply -f k8s/deployments/
kubectl apply -f k8s/services/
kubectl apply -f k8s/ingress.yaml

# Monitor deployment
kubectl get pods -n nourish-genie
kubectl logs -f deployment/auth-service -n nourish-genie
```

#### 3. Cloud Deployment (AWS/GCP/Azure)
- **Container Services**: ECS, GKE, AKS
- **Managed Databases**: RDS, Cloud SQL, Azure Database
- **Object Storage**: S3, Cloud Storage, Blob Storage
- **Load Balancers**: ALB, Cloud Load Balancer, Azure Load Balancer

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy Nourish Genie
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          # Run backend tests
          # Run Android tests
          # Run integration tests

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker images
        run: |
          docker build -t nourish-genie/auth-service ./auth_service
          docker build -t nourish-genie/user-service ./user_service
          # Build all services
      
      - name: Deploy to production
        run: |
          # Deploy to production environment
```

### Monitoring and Logging

**Application Monitoring**:
- **Prometheus**: Metrics collection
- **Grafana**: Metrics visualization
- **AlertManager**: Alert handling

**Logging**:
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Structured Logging**: JSON format across all services
- **Log Aggregation**: Centralized log collection

**Health Checks**:
- Service health endpoints
- Database connectivity checks
- External API availability
- Resource utilization monitoring

## 🧪 Testing

### Testing Strategy

#### Unit Tests
- **Backend**: pytest (Python), JUnit (Java), Jest (Node.js)
- **Android**: JUnit, Mockk, Espresso
- **Coverage Target**: 80%+ code coverage

#### Integration Tests
- **API Testing**: Postman/Newman, REST Assured
- **Database Testing**: Testcontainers
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
cd image_service && mvn test
cd subscription_service && mvn test

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
- **Feature Usage**: Food logging frequency, analytics views
- **Conversion**: Trial to subscription conversion rates
- **Retention**: User retention rates

### Alerting Rules

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
- Read replicas for analytics
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

### Security Measures

#### Authentication & Authorization
- **JWT Tokens**: Secure, stateless authentication
- **OAuth2**: Google and Apple Sign-In integration
- **Role-Based Access**: User permissions and premium features
- **Token Rotation**: Automatic token refresh

#### API Security
- **Rate Limiting**: DDoS protection and abuse prevention
- **Input Validation**: Comprehensive input sanitization
- **CORS Configuration**: Proper cross-origin policies
- **HTTPS Only**: TLS encryption for all communications

#### Data Security
- **Password Hashing**: bcrypt with appropriate rounds
- **Data Encryption**: Sensitive data encryption at rest
- **Secrets Management**: Docker secrets and environment isolation
- **Database Security**: Connection encryption and access controls

#### Infrastructure Security
- **Container Security**: Regular image updates and vulnerability scanning
- **Network Isolation**: Service-to-service communication restrictions
- **Access Controls**: Principle of least privilege
- **Audit Logging**: Comprehensive security event logging

### Security Best Practices

1. **Regular Security Updates**: Automated dependency updates
2. **Vulnerability Scanning**: Container and dependency scanning
3. **Penetration Testing**: Regular security assessments
4. **Incident Response**: Security incident response procedures
5. **Data Privacy**: GDPR compliance and data protection measures

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

#### Backend (Java)
- **Google Java Style**: Follow Google Java style guide
- **JavaDoc**: Document public APIs
- **Null Safety**: Use Optional and null checks
- **Exception Handling**: Proper exception management

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

- **OpenAI API**: Subject to OpenAI Terms of Service
- **Stripe API**: Subject to Stripe Terms of Service
- **Android Components**: Apache License 2.0
- **Spring Boot**: Apache License 2.0
- **FastAPI**: MIT License

---

## 📞 Support & Contact

- **Documentation Issues**: Create an issue in this repository
- **Bug Reports**: Use the bug report template
- **Feature Requests**: Use the feature request template
- **Security Issues**: Email security@foodtracker.example.com

## 🗺️ Roadmap

### Current Version (v1.0)
- ✅ Basic food logging and analytics
- ✅ AI-powered food recognition
- ✅ Achievement system
- ✅ Subscription management

### Upcoming Features (v1.1)
- 🔄 Weight tracking integration
- 🔄 Meal planning capabilities
- 🔄 Social features and sharing
- 🔄 Advanced export options

### Future Releases (v2.0+)
- 📅 Multi-platform support (iOS, Web)
- 📅 Integration with fitness trackers
- 📅 Nutritionist consultation features
- 📅 Machine learning meal recommendations

---

*Last Updated: July 2025*
*Version: 1.0.0*
