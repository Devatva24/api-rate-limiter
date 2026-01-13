# 🚦 API Rate Limiter

A robust, production-ready API rate limiting solution built with Spring Boot and Redis. Implements the Token Bucket algorithm to control API request rates and protect your backend services from abuse and DDoS attacks.

[![Java](https://img.shields.io/badge/Java-17+-orange?style=flat&logo=java)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen?style=flat&logo=spring)](https://spring.io/projects/spring-boot)
[![Redis](https://img.shields.io/badge/Redis-Latest-red?style=flat&logo=redis)](https://redis.io/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## 📌 Overview

This project provides a flexible rate limiting middleware that restricts the number of API requests per client IP address. Built using Spring Boot and Bucket4j with Redis as the distributed storage backend, it's designed to handle high-traffic scenarios while maintaining low latency.

### Key Features

- ✅ **IP-based Rate Limiting**: Tracks requests per client IP
- ⚡ **High Performance**: Redis-backed for distributed environments
- 🔄 **Token Bucket Algorithm**: Industry-standard rate limiting approach
- 🛡️ **DDoS Protection**: Prevents API abuse and overload
- 📊 **Customizable Rules**: Easy configuration for different rate limits
- 🚀 **Production Ready**: Battle-tested in real-world scenarios

## 🏗️ Architecture

```
┌─────────┐      ┌────────────────┐      ┌──────────────┐      ┌────────────┐
│ Client  │─────▶│ Rate Limiter   │─────▶│ Controller   │─────▶│ Service    │
│ Request │      │ Filter (IP)    │      │ Endpoint     │      │ Logic      │
└─────────┘      └────────────────┘      └──────────────┘      └────────────┘
                        │
                        ▼
                  ┌──────────┐
                  │  Redis   │
                  │  Cache   │
                  └──────────┘
```

### How It Works

1. **Request Interception**: Every incoming request passes through the rate limiter filter
2. **IP Extraction**: Client IP address is extracted from the request
3. **Token Check**: System checks if tokens are available in the bucket for that IP
4. **Decision**: 
   - If tokens available → Request proceeds, token consumed
   - If no tokens → Returns `429 Too Many Requests`
5. **Token Refill**: Tokens automatically refill based on configured rate

## ⚙️ Tech Stack

| Technology | Purpose |
|------------|---------|
| **Java 17+** | Programming language |
| **Spring Boot 3.x** | Application framework |
| **Redis** | Distributed cache for rate limit storage |
| **Bucket4j** | Rate limiting library (Token Bucket algorithm) |
| **Maven** | Dependency management |

## 🚦 Default Rate Limiting Rules

- **Limit**: 5 requests per minute per IP address
- **Algorithm**: Token Bucket
- **Bucket Capacity**: 5 tokens
- **Refill Rate**: 5 tokens per minute

## 📋 Prerequisites

Before running this project, ensure you have:

- Java 17 or higher
- Maven 3.6+
- Redis server (local or remote)
- Your favorite IDE (IntelliJ IDEA, Eclipse, VS Code)

## 🚀 Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/Devatva24/api-rate-limiter.git
cd api-rate-limiter
```

### 2. Install Redis

**On macOS:**
```bash
brew install redis
brew services start redis
```

**On Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install redis-server
sudo systemctl start redis
```

**On Windows:**
Download and install from [Redis Windows](https://github.com/microsoftarchive/redis/releases)

**Using Docker:**
```bash
docker run -d -p 6379:6379 --name redis redis:latest
```

### 3. Configure Application Properties

Update `src/main/resources/application.properties`:

```properties
# Redis Configuration
spring.data.redis.host=localhost
spring.data.redis.port=6379

# Rate Limiter Configuration
rate.limit.requests=5
rate.limit.duration=1m

# Server Configuration
server.port=8080
```

### 4. Build the Project

```bash
mvn clean install
```

### 5. Run the Application

```bash
mvn spring-boot:run
```

Or run the JAR directly:
```bash
java -jar target/api-rate-limiter-0.0.1-SNAPSHOT.jar
```

The application will start on `http://localhost:8080`

## 🧪 Testing the Rate Limiter

### Basic Test

```bash
# Send a request
curl http://localhost:8080/api/test

# Expected Response (first 5 requests):
# Status: 200 OK
# Body: "Request successful!"
```

### Testing Rate Limit

Run this script to test the rate limit:

```bash
# Send 10 requests quickly
for i in {1..10}; do
  curl -w "\nStatus: %{http_code}\n" http://localhost:8080/api/test
  echo "Request $i completed"
done
```

**Expected Results:**
- First 5 requests: `200 OK`
- Requests 6-10: `429 Too Many Requests`

### Using Postman

1. Create a new GET request to `http://localhost:8080/api/test`
2. Click "Send" multiple times rapidly
3. After 5 requests, you should receive a 429 error

### Response Examples

**Success Response (200 OK):**
```json
{
  "message": "Request successful!",
  "timestamp": "2026-01-13T10:30:00"
}
```

**Rate Limit Exceeded (429 Too Many Requests):**
```json
{
  "error": "Too Many Requests",
  "message": "Rate limit exceeded. Try again in 45 seconds.",
  "timestamp": "2026-01-13T10:30:45"
}
```

## ⚡ Customization

### Changing Rate Limit Rules

Modify the rate limiter configuration in your filter or configuration class:

```java
// Example: 10 requests per minute
Bandwidth limit = Bandwidth.simple(10, Duration.ofMinutes(1));

// Example: 100 requests per hour
Bandwidth limit = Bandwidth.simple(100, Duration.ofHours(1));

// Example: Multiple rate limits (burst + sustained)
Bandwidth[] limits = {
    Bandwidth.simple(10, Duration.ofSeconds(1)),  // Burst: 10/sec
    Bandwidth.simple(100, Duration.ofMinutes(1))  // Sustained: 100/min
};
```

### Per-Endpoint Rate Limiting

You can implement different limits for different endpoints:

```java
if (request.getRequestURI().startsWith("/api/heavy")) {
    // Stricter limit for resource-intensive endpoints
    return Bandwidth.simple(2, Duration.ofMinutes(1));
} else {
    // Default limit
    return Bandwidth.simple(5, Duration.ofMinutes(1));
}
```

### User-based Rate Limiting

Instead of IP-based limiting, use authentication tokens:

```java
String userId = extractUserIdFromToken(request);
String key = "rate_limit:user:" + userId;
```

## 📁 Project Structure

```
api-rate-limiter/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/devatva/ratelimiter/
│   │   │       ├── config/          # Configuration classes
│   │   │       ├── filter/          # Rate limiter filter
│   │   │       ├── controller/      # REST controllers
│   │   │       ├── service/         # Business logic
│   │   │       └── RateLimiterApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/                        # Unit and integration tests
├── target/                          # Compiled classes
├── pom.xml                          # Maven dependencies
└── README.md
```

## 🛡️ Production Considerations

### 1. Distributed Systems
When running multiple instances, Redis ensures consistent rate limiting across all nodes.

### 2. Redis Clustering
For high availability, configure Redis in cluster mode:
```properties
spring.data.redis.cluster.nodes=node1:6379,node2:6379,node3:6379
```

### 3. Monitoring
Add metrics to track:
- Rate limit hits
- Average response time
- Redis connection health

### 4. Graceful Degradation
Implement fallback behavior if Redis is unavailable:
```java
try {
    checkRateLimit(ip);
} catch (RedisConnectionException e) {
    // Log error and allow request (or use in-memory fallback)
    logger.error("Redis unavailable, bypassing rate limit");
}
```

## 🔍 Algorithm Deep Dive

### Token Bucket Algorithm

The Token Bucket algorithm works like this:

1. **Bucket**: Each client has a bucket that holds tokens
2. **Capacity**: Maximum tokens the bucket can hold (e.g., 5)
3. **Refill**: Tokens are added at a constant rate (e.g., 5 per minute)
4. **Consumption**: Each request consumes one token
5. **Rejection**: If no tokens available, request is rejected

**Visual Example:**
```
Time: 0s    [🪙🪙🪙🪙🪙] 5 tokens available
Request 1   [🪙🪙🪙🪙-] 4 tokens (✅ Allowed)
Request 2   [🪙🪙🪙--] 3 tokens (✅ Allowed)
Request 3   [🪙🪙---] 2 tokens (✅ Allowed)
Request 4   [🪙----] 1 token  (✅ Allowed)
Request 5   [-----] 0 tokens (✅ Allowed)
Request 6   [-----] 0 tokens (❌ Rejected - 429)
Time: 60s   [🪙🪙🪙🪙🪙] Refilled!
```

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Areas for Contribution

- Add user authentication-based rate limiting
- Implement different algorithms (Leaky Bucket, Sliding Window)
- Add comprehensive test coverage
- Create admin dashboard for monitoring
- Add API documentation with Swagger

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🙏 Acknowledgments

- [Bucket4j](https://github.com/bucket4j/bucket4j) - Excellent rate limiting library
- [Spring Boot](https://spring.io/projects/spring-boot) - Amazing framework
- [Redis](https://redis.io/) - Blazing fast in-memory store

## 👤 Author

**Devatva Rachit**

- GitHub: [@Devatva24](https://github.com/Devatva24)
- LinkedIn: [DevatvaRachit](https://www.linkedin.com/in/devatva-rachit-317a11229)

## 📚 Further Reading

- [Token Bucket Algorithm Explained](https://en.wikipedia.org/wiki/Token_bucket)
- [Rate Limiting Strategies](https://cloud.google.com/architecture/rate-limiting-strategies-techniques)
- [Redis Best Practices](https://redis.io/docs/manual/patterns/)

## ⭐ Show Your Support

Give a ⭐️ if this project helped you learn about rate limiting!

---

**Built with ☕ by Devatva Rachit**
