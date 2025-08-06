# API Gateway: Documentation for an Evolving Microservices Architecture

This document provides comprehensive and in-depth technical documentation for the Food Tracker application's API Gateway. It serves as the central entry point for all client interactions, orchestrating requests to various microservices, enforcing security policies, and providing a flexible foundation for future backend expansions.

-----

## 1\. Architecture Overview

The Food Tracker application is built on a robust, scalable microservices architecture. The API Gateway is the critical component that abstracts the complexity of the backend, offering a unified, secure, and performant interface to client applications.

\![Comprehensive Microservices Architecture Diagram - This diagram should clearly show:

  - Client (Android App) at the top.
  - API Gateway as the first point of contact.
  - API Gateway connecting to multiple microservices (Auth, User, Image, Subscription, Email, **and placeholders for future services like Analytics, Notifications, Payments, etc.**).
  - Redis as a central cache/message broker, connected to multiple services.
  - PostgreSQL as the primary data store, connected to relevant services.
  - External Integrations (OpenAI, SMTP Provider, Payment Gateway) clearly shown as outbound connections from relevant services.
  - Load Balancers/Ingress controllers in front of the API Gateway if applicable.
  - Monitoring/Logging infrastructure.
  - Arrows clearly depicting request flow and data flow.]

### 1.1. Core Components

  * **API Gateway (This Repository)**: The intelligent traffic cop. It handles:
      * **Routing**: Directing requests to the correct service.
      * **Rate Limiting**: Protecting against abuse.
      * **Authentication/Authorization Enforcement**: Initial validation of tokens and role-based access control (RBAC) at the edge.
      * **Request/Response Transformation**: Modifying headers, bodies, or query parameters.
      * **CORS Handling**: Managing Cross-Origin Resource Sharing.
      * **Health Checks**: Providing liveness and readiness probes.
      * **Internal Service Protection**: Shielding internal-only services from public access.
      * **Observability**: Centralizing logging, metrics, and tracing for incoming requests.
  * **Auth Service**: Core identity and access management. Responsible for user lifecycle (registration, login, profile management, password reset), token issuance (JWTs, refresh tokens), and session invalidation.
  * **User Service**: Manages detailed user profiles, preferences, and potentially user-generated content metadata.
  * **Image Service**: Dedicated to media processing. Handles image uploads, storage (e.g., S3, Google Cloud Storage), resizing, format conversion, and integration with AI/ML services (e.g., OpenAI for food recognition).
  * **Subscription Service**: Manages user subscriptions, tiers, entitlements, and integrates with payment gateways (e.g., Stripe, PayPal) for recurring billing.
  * **Email Service**: A dedicated messaging service for transactional emails (password resets, notifications, confirmations). Decoupled via message queues for asynchronous processing.
  * **Redis**: High-performance in-memory data store. Used for:
      * **Caching**: Frequently accessed data for improved response times.
      * **Rate Limiting Counters**: Efficiently tracking request counts.
      * **Ephemeral Data**: Session tokens, password reset codes with TTLs.
      * **Message Broker (Pub/Sub)**: Asynchronous communication between services (e.g., Auth Service publishing email events to Email Service).
  * **PostgreSQL**: Robust, ACID-compliant relational database for persistent storage of structured data (user records, subscription details, image metadata, food entries).
  * **Message Queue (e.g., RabbitMQ, Kafka, or extended Redis Pub/Sub)**: (Future consideration, currently Redis Pub/Sub) For more complex asynchronous communication, event streaming, and reliable message delivery between services.
  * **Observability Stack (e.g., Prometheus/Grafana, ELK/Loki, Jaeger)**: Tools for monitoring, logging, and tracing across all microservices, critical for debugging and operational insights.

-----

## 2\. API Gateway: Detailed Functionality and Extensibility

The API Gateway is designed for robustness and ease of extension, allowing new features and microservices to be integrated seamlessly.

### 2.1. Dynamic Request Routing and Proxying

The `routes.js` file is the central configuration for routing. Its modular design using `createServiceProxy` simplifies the addition of new microservices.

**Current Implementation (`routes.js`):**

```javascript
// Helper function to create proxy middleware
const createServiceProxy = (pathRewrite, target) => {
  return createProxyMiddleware({
    target,
    changeOrigin: true,
    pathRewrite,
    // ... onProxyReq, onError callbacks
  });
};

// Authentication Service Routes
router.use('/auth-service', createServiceProxy(
  {'^/auth-service': ''},
  AUTH_SERVICE_URL,
));

// ... other service routes
```

**Extensibility for New Services:**
Adding a new service, e.g., `Notification Service` on port `8086`, is straightforward:

1.  Define its URL as an environment variable:
    `const NOTIFICATION_SERVICE_URL = process.env.NOTIFICATION_SERVICE_URL || 'http://notification-service:8086';`
2.  Add a new routing entry in `routes.js`:
    ```javascript
    // Notification Service Routes
    router.use('/notification-service', standardLimiter, createServiceProxy(
      {'^/notification-service': ''},
      NOTIFICATION_SERVICE_URL
    ));
    ```

This pattern ensures consistency and minimizes changes when scaling the architecture.

### 2.2. Advanced Rate Limiting Strategies

The API Gateway provides a multi-tiered rate-limiting approach using `express-rate-limit`.

  * **`standardLimiter`**: General purpose, protecting most endpoints from common abuse.
      * `windowMs: 15 * 60 * 1000` (15 minutes)
      * `max: 100` requests
  * **`authLimiter`**: Stricter for login/registration to prevent brute-forcing accounts.
      * `windowMs: 15 * 60 * 1000` (15 minutes)
      * `max: 20` requests
  * **`passwordResetLimiter`**: Most restrictive for password reset flow, critical for account security.
      * `windowMs: 60 * 60 * 1000` (1 hour)
      * `max: 5` requests

**Key Generation for IP Identification:**
The `keyGenerator` function ensures robust IP identification, crucial for accurate rate limiting:

```javascript
keyGenerator: (req) => {
    return req.headers['x-real-ip'] ||
           (req.headers['x-forwarded-for'] ? req.headers['x-forwarded-for'].split(',')[0].trim() : null) ||
           req.ip;
}
```

This prioritizes common proxy/load balancer headers (`x-real-ip`, `x-forwarded-for`) to get the true client IP, falling back to the direct connection IP.

**Future Enhancements for Rate Limiting:**

  * **Distributed Rate Limiting**: For multi-instance deployments, consider using a shared Redis store for rate limit counters (`express-rate-limit` can be configured for this) to ensure limits are consistent across all API Gateway instances.
  * **User-Specific Limits**: Implement rate limits per authenticated user (e.g., by extracting user ID from JWT) in addition to or instead of IP-based limits for specific sensitive operations.
  * **Burst Limits**: Add burst limiting to prevent sudden spikes of requests even within the rate limit window.

### 2.3. Request and Response Transformation

The `onProxyReq` and potentially `onProxyRes` callbacks are powerful hooks for modifying requests/responses at the edge.

  * **`onProxyReq` (Current Implementation)**:
      * **`X-Forwarded-For`**: Propagates the client's original IP to downstream services. This is vital for logging, geo-targeting, and service-level rate limiting.
      * **JSON Body Handling**: Correctly forwards JSON bodies, including setting `Content-Length`. This is a common pitfall with proxying POST requests.

**Future Enhancements for Transformation:**

  * **Authentication & Authorization Enforcement**:
      * Implement a middleware to **verify JWTs** for authenticated routes.
      * Extract user ID and roles from the token and add them as custom headers (e.g., `X-User-ID`, `X-User-Roles`) before forwarding to downstream services. This offloads authentication logic from individual microservices.
      * For basic authorization, the API Gateway could check if the user has the necessary roles for a given endpoint.
  * **Header Manipulation**: Add, remove, or modify other headers (e.g., adding a `Correlation-ID` for distributed tracing, removing sensitive headers before forwarding).
  * **Query Parameter Modification**: Transform or sanitize query parameters before forwarding.
  * **Response Compression**: Implement GZIP or Brotli compression on responses for faster delivery to clients.
  * **Response Caching**: For highly cacheable public data, the API Gateway could implement a simple edge cache.

### 2.4. Internal Service Protection

The current implementation restricts direct external access to the `Email Service` by checking the source IP address:

```javascript
const isInternalRequest = req.ip === '127.0.0.1' ||
                         req.ip.startsWith('10.') ||
                         // ... other private IP ranges
                         req.ip.startsWith('192.168.');

if (!isInternalRequest) {
  return res.status(403).json({ message: 'Access forbidden' });
}
next(); // Proceed to proxy if internal
```

**Future Enhancements for Internal Protection:**

  * **Service Mesh Integration**: In a Kubernetes environment, a service mesh (e.g., Istio, Linkerd) provides more robust internal access control, mutual TLS, and policy enforcement between services.
  * **Dedicated Internal API Gateway**: For larger setups, a separate "Internal API Gateway" could be deployed that is only accessible from within the private network, handling requests from other internal services.
  * **API Keys/Internal Tokens**: Internal services could use pre-shared API keys or short-lived internal tokens to authenticate with each other, adding another layer of security.

### 2.5. Health Check and Observability

The API Gateway provides a `/health` endpoint for robust health checks in containerized environments.

**Healthcheck (`Dockerfile`):**

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=5 \
  CMD wget -q --spider http://localhost:8080/health || exit 1
```

This configuration tells Docker to regularly check the `/health` endpoint, contributing to container liveness and readiness.

**Future Enhancements for Observability:**

  * **Metrics Collection**: Integrate with a metrics library (e.g., Prometheus client library) to expose granular metrics:
      * Request counts per endpoint/service.
      * Latency (p90, p95, p99) for proxying requests.
      * Error rates.
      * Rate limiting hits.
  * **Structured Logging**: Implement structured logging (e.g., JSON logs) with relevant fields like `requestId`, `userId`, `sourceIp`, `userAgent`, `targetService`, `httpMethod`, `statusCode`, `responseTime`. This facilitates easier parsing and analysis in a centralized logging system.
  * **Distributed Tracing**: Generate and propagate tracing headers (e.g., `X-B3-TraceId`, `traceparent`) for every incoming request. This allows request flows to be visualized across multiple microservices, crucial for debugging complex interactions.

### 2.6. Robust Startup (`start.sh`)

The `start.sh` script is a critical operational component, ensuring the API Gateway only starts serving traffic after its dependent services are (at least partially) ready.

```bash
# ... (DELAY, ls, node --version, env)
echo "Waiting for ${DELAY} seconds to ensure all services are fully ready..."
sleep ${DELAY}

wait_for_service() {
  service_name=$1
  port=$2
  max_retries=30
  retry_count=0
  # ... nc -z -w2 check
}

echo "Verifying all services are ready..."
wait_for_service "auth-service" "8081"
wait_for_service "user-service" "8082"
# ... other services
```

This uses `netcat` (`nc`) to check if the target service's port is open, providing a basic but effective readiness probe for the entire system at startup.

**Future Enhancements for Startup:**

  * **Service Readiness Probes (Advanced)**: For more sophisticated checks, services could expose dedicated "readiness" endpoints that verify database connections, external service availability, etc. The `start.sh` script could then hit these specific endpoints.
  * **Configuration Management Tools**: For complex deployments, configuration management tools (e.g., Ansible, Terraform) or Kubernetes Init Containers can orchestrate startup order and dependencies more elegantly.

-----

## 3\. Operational Excellence

Maintaining a production-grade API Gateway requires continuous attention to operational practices.

### 3.1. Scalability Considerations

  * **Horizontal Scaling**: The API Gateway is designed to be stateless (aside from in-memory rate limiting, which can be moved to Redis). This allows for easy horizontal scaling by running multiple instances behind a load balancer.
  * **Resource Allocation**: Monitor CPU, memory, and network I/O usage to correctly size instances.
  * **Connection Pooling**: Ensure underlying `http-proxy-middleware` or Node.js HTTP agents are configured with appropriate connection pooling to downstream services to prevent resource exhaustion.

### 3.2. Performance Tuning

  * **Keep-Alive Connections**: Enable HTTP keep-alive connections to and from the API Gateway to reduce overhead for repeated requests.
  * **Node.js Event Loop Monitoring**: Monitor the Node.js event loop latency to identify blocking operations.
  * **Profiling**: Use Node.js profiling tools to identify performance bottlenecks in custom middleware or `onProxyReq`/`onProxyRes` logic.

### 3.3. Security Audits and Best Practices

  * **Regular Dependency Updates**: Keep `npm` packages up-to-date to patch known vulnerabilities. Use tools like `npm audit` or Snyk.
  * **Security Scanning**: Integrate container image scanning (e.g., Clair, Trivy) into your CI/CD pipeline.
  * **Principle of Least Privilege**: Ensure the `appuser` in the Dockerfile has only the necessary permissions.
  * **Environment Hardening**: Configure firewalls, network ACLs, and security groups to restrict traffic flow only to necessary ports.
  * **DDoS Protection**: Consider external DDoS protection services (e.g., Cloudflare, AWS Shield) for public-facing endpoints.

### 3.4. Deployment Strategy

  * **CI/CD Pipelines**: Automate building, testing, and deploying the API Gateway through a CI/CD pipeline.
  * **Blue/Green or Canary Deployments**: Implement deployment strategies that minimize downtime and risk during updates.
  * **Rollback Procedures**: Define clear rollback procedures in case of deployment failures.

-----

## 4\. API Endpoints and Future Considerations

This section lists the current API Gateway endpoints and outlines how new features or services might integrate.

| Endpoint Prefix         | Service Routed To         | Rate Limit                   | Notes                                                     |
| :---------------------- | :------------------------ | :--------------------------- | :-------------------------------------------------------- |
| `/auth-service/login`   | Auth Service              | 20 reqs/15 min                 | Stricter for auth attempts.                               |
| `/auth-service/register`| Auth Service              | 20 reqs/15 min                 | Stricter for auth attempts.                               |
| `/auth-service/forgot-password`| Auth Service              | 5 reqs/hour                  | Very strict for password reset.                           |
| `/auth-service/verify-reset-code`| Auth Service              | 5 reqs/hour                  | Very strict for password reset.                           |
| `/auth-service/reset-password`| Auth Service              | 5 reqs/hour                  | Very strict for password reset.                           |
| `/auth-service/*`       | Auth Service              | 100 reqs/15 min                | General auth-related endpoints.                           |
| `/user-service/*`       | User Service              | 100 reqs/15 min                | Standard rate limit.                                      |
| `/image-service/*`      | Image Service             | 100 reqs/15 min                | Standard rate limit.                                      |
| `/subscription-service/*`| Subscription Service      | 100 reqs/15 min                | Standard rate limit.                                      |
| `/email-service/*`      | Email Service             | Internal access only         | Shielded from direct external access.                     |
| `/health`               | API Gateway (Self)        | No limit                     | For liveness/readiness probes.                            |

### 4.1. Adding New API Endpoints

When adding new functionalities or services, the process will typically involve:

1.  **Develop New Microservice**: Create the new microservice (e.g., `Analytics Service`).
2.  **Define Environment Variable**: Add `ANALYTICS_SERVICE_URL` to the API Gateway's environment configuration.
3.  **Add Routing Rule**: Add a new `router.use()` entry in `routes.js` for the new service prefix (e.g., `/analytics-service`).
4.  **Apply Rate Limiting**: Decide on and apply an appropriate rate limit to the new route.
5.  **Update Documentation**: Crucially, update this documentation to reflect the new endpoint and its characteristics.

### 4.2. Example: Integrating a New "Notification" Service

If a new `Notification Service` is introduced (e.g., for in-app notifications, push notifications), the integration would be seamless:

  * **Service Name**: `notification-service`
  * **Default URL**: `http://notification-service:8086`
  * **API Gateway Route**: `/notification-service/*`
  * **Rate Limit**: Likely `standardLimiter` or a custom one if notifications are very frequent.

The API Gateway would then enable clients to interact with this new service through a consistent `https://api.foodtracker.example.com/notification-service/...` endpoint.

-----

## 5\. Troubleshooting Common Issues

  * **`502 Bad Gateway` / `500 Proxy Error`**:
      * **Cause**: Downstream service is unreachable, crashing, or returning errors.
      * **Troubleshooting**:
          * Check `api-gateway` logs for `[Proxy Error]` messages.
          * Verify the target microservice is running and healthy (check its logs).
          * Confirm environment variables (`*_SERVICE_URL`) are correctly set in the API Gateway container and point to the right service names/IPs and ports.
          * Use `docker-compose logs <service_name>` to inspect individual service logs.
          * From the API Gateway container, `ping <service_name>` or `nc -zv <service_name> <port>` to check network connectivity.
  * **`429 Too Many Requests`**:
      * **Cause**: Client has exceeded the configured rate limits.
      * **Troubleshooting**:
          * Check `api-gateway` logs for rate limit messages.
          * Verify the `keyGenerator` correctly identifies the client's IP.
          * Adjust `max` or `windowMs` settings in `routes.js` if the limits are too restrictive for legitimate use cases.
  * **`403 Forbidden` for `email-service`**:
      * **Cause**: External request attempted to access the internal-only `email-service` endpoint.
      * **Troubleshooting**: This is expected behavior. If an internal service is experiencing this, check its source IP to ensure it's within the allowed ranges.
  * **JSON Body Not Received by Downstream Service**:
      * **Cause**: `Content-Type` header mismatch or `onProxyReq` not correctly handling the body.
      * **Troubleshooting**: Ensure `Content-Type: application/json` is correctly set by the client. Double-check the `onProxyReq` logic for handling POST requests.
  * **Startup Delays/Service Unreachable (during `start.sh`)**:
      * **Cause**: Dependent services are not starting quickly enough, or `STARTUP_DELAY` is too short.
      * **Troubleshooting**:
          * Increase `STARTUP_DELAY` in the environment.
          * Check the logs of the dependent services for startup errors.
          * Verify network configuration in Docker Compose or Kubernetes.
