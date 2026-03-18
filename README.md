# Config Server Service

A **centralized configuration management server** built with **Spring Cloud Config**. It serves configuration properties to all microservices from a single source, enabling environment-specific settings without code changes or redeployment.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Dependencies](#dependencies)
- [Configuration Sources](#configuration-sources)
- [Configuration Properties](#configuration-properties)
- [Setup & Installation](#setup--installation)
- [Building & Running](#building--running)
- [Accessing Configuration](#accessing-configuration)
- [Configuration Endpoints](#configuration-endpoints)
- [Managing Configurations](#managing-configurations)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

The **Config Server** is a critical infrastructure component in the NexaShopping microservices architecture. It:

- ✅ Provides **centralized configuration management** for all microservices
- ✅ Supports **environment-specific configurations** (dev, staging, prod)
- ✅ Offers **dynamic configuration updates** without service redeployment
- ✅ Enables **configuration versioning** via Git integration
- ✅ Provides **fallback configurations** using native filesystem backend
- ✅ Reduces **configuration drift** across distributed systems
- ✅ Implements **single source of truth** for all service properties

**Application Name:** `Config-Server`  
**Default Port:** `9000`  
**Artifact ID:** `Config-Server`  
**Group ID:** `lk.ijse.eca`

---

## 🏗️ Architecture

The Config Server sits at the core of the microservices configuration ecosystem:

```
┌──────────────────────────────────────────────┐
│      Spring Cloud Config Server              │
│      (Port 9000)                             │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │ Configuration Sources:                  │ │
│  │ 1. Git Repository (Primary)             │ │
│  │ 2. Classpath/Filesystem (Fallback)      │ │
│  └─────────────────────────────────────────┘ │
└──────┬───────────────────────────────────────┘
       │
   ┌───┴─────┬──────────┬──────────┐
   │         │          │          │
   ▼         ▼          ▼          ▼
┌─────┐ ┌──────┐ ┌──────┐ ┌──────┐
│API  │ │User  │ │Item  │ │Order │
│Gate │ │Svc   │ │Svc   │ │Svc   │
│way  │ └──────┘ └──────┘ └──────┘
└─────┘
 (All fetch their configs on startup)

┌──────────────────────────────────┐
│  GitHub Repository (Optional)    │
│ (Capstone-Project-Configurations)│
└──────────────────────────────────┘
   (Primary config source)
```

---

## 🛠️ Tech Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| **Java** | JDK 21 | Language runtime |
| **Spring Boot** | 4.0.3 | Application framework |
| **Spring Cloud** | 2025.1.0 | Microservices framework |
| **Spring Cloud Config Server** | Latest (2025.1.0) | Configuration management engine |
| **Git Integration** | Latest | Version control for configurations |
| **Spring Boot Actuator** | Latest | Health checks & monitoring endpoints |
| **Spring Boot DevTools** | Latest (runtime) | Hot reload during development |

---

## 📁 Project Structure

```
config-server/
├── src/
│   ├── main/
│   │   ├── java/lk/ijse/eca/configserver/
│   │   │   └── ConfigServerApplication.java   # Main Spring Boot entry point
│   │   └── resources/
│   │       ├── application.yaml               # Config Server config
│   │       └── configurations/                # Fallback configs (classpath)
│   │           ├── application.yaml           # Shared defaults
│   │           ├── platform/
│   │           │   ├── api-gateway.yaml
│   │           │   ├── config-server.yaml
│   │           │   └── service-registry.yaml
│   │           └── services/
│   │               ├── user-service.yaml
│   │               ├── item-service.yaml
│   │               └── order-service.yaml
│   └── test/
├── pom.xml                                     # Maven configuration
├── mvnw / mvnw.cmd                            # Maven wrapper scripts
└── README.md                                   # This file
```

### Key Java Classes

- **ConfigServerApplication.java**: Spring Boot main application class
  - Annotated with `@EnableConfigServer` - activates configuration server functionality
  - Bootstraps on port 9000

---

## 📦 Dependencies

### Core Dependencies

1. **spring-cloud-config-server**
   - Core Spring Cloud Config Server functionality
   - Handles configuration fetching from Git/filesystem
   - Provides REST API for accessing configurations
   - Supports property source ordering and resolution

2. **spring-boot-starter-actuator**
   - Health and monitoring endpoints
   - `/actuator/health` - service health status
   - `/actuator/env` - environment properties
   - `/actuator/configprops` - configuration properties

3. **spring-boot-devtools** (optional, runtime only)
   - Enables hot reload during development
   - Useful for rapid testing of configuration changes

---

## ⚙️ Configuration Sources

The Config Server supports **two configuration sources** in order of priority:

### 1. Git Repository (Primary)

**Configuration:**
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/YOUR-ORG/Capstone-Project-Configurations.git
          search-paths: platform,services
```

**Benefits:**
- ✅ Version control and history tracking
- ✅ Audit trail of configuration changes
- ✅ Easy rollback to previous versions
- ✅ Supports multiple branches (dev, staging, prod)
- ✅ Collaborative configuration management

**Current Repository:**
```
https://github.com/ITS-2130-ECA-HDSE-69-70/Capstone-Project-Configurations.git
```

**Expected Repository Structure:**
```
Capstone-Project-Configurations/
├── platform/
│   ├── api-gateway.yaml
│   ├── api-gateway-dev.yaml
│   ├── config-server.yaml
│   ├── config-server-dev.yaml
│   ├── service-registry.yaml
│   └── service-registry-dev.yaml
├── services/
│   ├── user-service.yaml
│   ├── user-service-dev.yaml
│   ├── item-service.yaml
│   ├── item-service-dev.yaml
│   ├── order-service.yaml
│   └── order-service-dev.yaml
└── README.md
```

### 2. Classpath/Filesystem Backend (Fallback)

**Configuration:**
```yaml
spring:
  cloud:
    config:
      server:
        native:
          search-locations:
            - classpath:/configurations
            - classpath:/configurations/platform
            - classpath:/configurations/services
```

**Benefits:**
- ✅ Works without external Git repository
- ✅ Useful for development and testing
- ✅ No internet connectivity required
- ✅ Fast startup time

**Local Fallback Structure:**
```
src/main/resources/configurations/
├── application.yaml
├── platform/
│   ├── api-gateway.yaml
│   ├── config-server.yaml
│   └── service-registry.yaml
└── services/
    ├── user-service.yaml
    ├── item-service.yaml
    └── order-service.yaml
```

---

## 🔧 Configuration Properties

### application.yaml

```yaml
spring:
  application:
    name: Config-Server                # Service name

  cloud:
    config:
      server:
        git:
          uri: https://github.com/ITS-2130-ECA-HDSE-69-70/Capstone-Project-Configurations.git
          search-paths: platform,services  # Directories to search for configs
        native:
          search-locations:
            - classpath:/configurations
            - classpath:/configurations/platform
            - classpath:/configurations/services

server:
  port: 9000                          # Config Server runs on port 9000
```

### Environment-Specific Profiles

Services can request profile-specific configurations:

```
/v2/keys/{application}/{profile}
```

**Example:**
- `user-service.yaml` - production config
- `user-service-dev.yaml` - development config
- `user-service-staging.yaml` - staging config

---

## 🚀 Setup & Installation

### Prerequisites

- **Java 21** (or compatible version)
- **Maven 3.6+** (or use the included `mvnw` wrapper)
- **Git** (optional but recommended for Git-backed configurations)
- **GitHub Account** (optional, for remote config repository)

### Installation Steps

1. **Clone the repository** (if not already done)
   ```bash
   git clone <repository-url>
   cd NexaShopping-Project/platform/config-server
   ```

2. **Verify Java version**
   ```bash
   java -version
   # Should output Java 21 or compatible
   ```

3. **(Optional) Update Git repository URL**
   
   Edit `src/main/resources/application.yaml` and update the Git URI:
   ```yaml
   spring:
     cloud:
       config:
         server:
           git:
             uri: https://github.com/YOUR-ORG/YOUR-CONFIG-REPO.git
   ```

4. **Build the project**
   ```bash
   ./mvnw clean install
   ```

5. **Verify dependencies**
   ```bash
   ./mvnw dependency:tree
   ```

---

## ▶️ Building & Running

### Option 1: Using Maven (Recommended)

**Build:**
```bash
./mvnw clean package
```

**Run:**
```bash
./mvnw spring-boot:run
```

### Option 2: Run JAR Directly

**After building:**
```bash
./mvnw clean package
java -jar target/Config-Server-1.0.0.jar
```

### Option 3: IDE Run (IntelliJ/Eclipse)

1. Right-click `ConfigServerApplication.java`
2. Select **Run** or **Debug**

### Startup Verification

**Check if Config Server is running:**
```bash
curl http://localhost:9000/health
```

**Expected Response:**
```json
{
  "status": "UP"
}
```

---

## 📥 Accessing Configuration

Once the Config Server is running, microservices can fetch their configuration via REST endpoints.

### Configuration Endpoint Format

```
/v2/keys/{application}/{profile}/{label}
```

**Components:**
- `{application}` - Service name (e.g., `api-gateway`, `user-service`)
- `{profile}` - Environment profile (e.g., `dev`, `prod`) - optional
- `{label}` - Git branch name (e.g., `main`, `develop`) - optional

### Examples

**1. Get User Service Production Config**
```bash
curl http://localhost:9000/user-service/prod
```

**2. Get API Gateway Development Config**
```bash
curl http://localhost:9000/api-gateway/dev
```

**3. Get Item Service Default Config**
```bash
curl http://localhost:9000/item-service
```

**4. Get Configuration from Specific Git Branch**
```bash
curl http://localhost:9000/order-service/prod/develop
```

### Response Format

The server returns configurations in JSON format:

```json
{
  "name": "user-service",
  "profiles": ["dev"],
  "label": "main",
  "version": "abc123def456",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/.../user-service-dev.yaml",
      "source": {
        "spring.datasource.url": "jdbc:mysql://localhost:3306/nexashopping_dev",
        "spring.jpa.hibernate.ddl-auto": "create-drop",
        "logging.level.root": "DEBUG"
      }
    }
  ]
}
```

---

## 🔌 Configuration Endpoints

### Health Endpoints

| Endpoint | Purpose | URL |
|----------|---------|-----|
| **health** | Service health | `http://localhost:9000/actuator/health` |
| **info** | Service info | `http://localhost:9000/actuator/info` |
| **env** | Environment properties | `http://localhost:9000/actuator/env` |

### Configuration Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/{app}/{profile}/{label}` | Get configuration |
| `/{app}/{profile}` | Get config (default label) |
| `/{app}` | Get config (no profile) |

---

## 📝 Managing Configurations

### Adding New Service Configuration

1. **Create configuration file** in Git repo or classpath:
   ```
   services/new-service.yaml
   services/new-service-dev.yaml
   ```

2. **Add service-specific properties:**
   ```yaml
   spring:
     application:
       name: new-service
     datasource:
       url: jdbc:mysql://localhost:3306/new_service
     jpa:
       hibernate:
         ddl-auto: validate

   server:
     port: 8004
   ```

3. **Commit and push** to Git repository (if using Git)

4. **Service will fetch on next startup**

### Updating Configuration

**For Git-backed configs:**
1. Update `.yaml` file in Git repository
2. Commit and push changes
3. Services will fetch on next:
   - Bootstrap (if using `spring.config.import`)
   - Manual refresh request

**For classpath-based configs:**
1. Update file in `src/main/resources/configurations/`
2. Rebuild and restart Config Server

### Environment-Specific Overrides

Create profile-specific files:

```yaml
# Default (all environments)
user-service.yaml
  spring.jpa.show-sql: false

# Development
user-service-dev.yaml
  spring.jpa.show-sql: true
  logging.level.root: DEBUG

# Production
user-service-prod.yaml
  logging.level.root: WARN
  spring.jpa.show-sql: false
```

Services can request specific profiles via `spring.profiles.active`:
```yaml
spring:
  profiles:
    active: dev
```

---

## 🐛 Troubleshooting

### Issue 1: Port Already in Use

**Error:** `Address already in use: 9000`

**Solution:**
```bash
# Check what's using port 9000
netstat -ano | findstr :9000

# Kill the process
taskkill /PID <PID> /F

# Or change the port in application.yaml
server:
  port: 9010
```

### Issue 2: Cannot Access Git Repository

**Error:** `org.eclipse.jgit.api.errors.TransportException: https://github.com/...`

**Solutions:**
1. Verify Git repository URL is correct and accessible
2. Check internet connectivity
3. Ensure GitHub credentials are configured (if private repo)
4. Use fallback native filesystem configuration

**Check Git connectivity:**
```bash
git clone https://github.com/ITS-2130-ECA-HDSE-69-70/Capstone-Project-Configurations.git
```

### Issue 3: Configuration Not Found

**Error:** `Cannot find property source`

**Solutions:**
1. Verify configuration file exists in Git repo or classpath
2. Check file naming convention matches service name
3. Ensure `search-paths` in application.yaml includes the right directories
4. Check for typos in application name or profile

### Issue 4: Stale Configuration Cache

**Problem:** Services still using old configuration

**Solution:**
```bash
# Refresh configuration on running service
curl -X POST http://localhost:<SERVICE_PORT>/actuator/refresh
```

(Requires `spring-boot-starter-actuator` and `spring-cloud-starter-config`)

### Common Log Messages

| Log | Meaning | Action |
|-----|---------|--------|
| `Fetching config from git repo` | ✅ Healthy | Config loaded from Git |
| `Using native filesystem config` | ⚠️ Warning | Git fallback activated |
| `No remote tracking branch` | ✅ Normal | Git is configured correctly |
| `Connection timeout to Git` | ❌ Error | Check internet/Git URL |

---

## 🔐 Security Considerations

1. **Git Repository Access**
   - For private repositories: Configure SSH keys or HTTPS credentials
   - Consider using GitHub tokens for CI/CD pipelines

2. **Sensitive Configuration**
   - Avoid storing passwords in configuration files
   - Use environment variables for sensitive values
   - Consider Spring Cloud Config Encryption/Decryption

3. **Network Security**
   - Config Server should only be accessible from internal network
   - Restrict access via firewall rules
   - Consider API Gateway authentication

---

## 📊 Performance Characteristics

- **Startup Time**: Few seconds (fast Git clone on first run)
- **Configuration Caching**: Cached by clients until refresh
- **Git Repository Size**: Scales well with typical microservices configs
- **Throughput**: Can handle hundreds of config requests per second

---

## 🔄 Typical Configuration Flow

1. **Service Startup**
   ```
   Service Instance Starting
   ↓
   Reads spring.config.import: "configserver:"
   ↓
   Sends request to Config Server (port 9000)
   ↓
   Config Server fetches from Git or classpath
   ↓
   Returns configuration properties
   ↓
   Service completes initialization with config
   ```

2. **Dynamic Refresh** (if enabled)
   ```
   Configuration updated in Git
   ↓
   Developer/CI commits and pushes
   ↓
   Service calls /actuator/refresh endpoint
   ↓
   Config Server fetches latest from Git
   ↓
   Service applies new configuration (without restart)
   ```

---

## 📚 References

- [Spring Cloud Config Documentation](https://spring.io/projects/spring-cloud-config)
- [Spring Cloud Config Server Guide](https://cloud.spring.io/spring-cloud-config/reference/html/)
- [Git Configuration Management](https://cloud.spring.io/spring-cloud-config/reference/html/#_version_control_backend)
- [Spring Boot Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

---

## 📝 Notes

- **Starting Order**: Config Server should start **first** - before all other services
- **Fallback Mechanism**: If Git unavailable, uses classpath configurations automatically
- **Stateless Design**: Config Server is stateless - can be easily scaled
- **Configuration Versioning**: Git integration provides full version history

---

## 🤝 Support

For issues or questions:
1. Check the logs for detailed error messages
2. Verify Git repository URL and accessibility
3. Ensure all dependencies are correctly installed
4. Test configuration endpoint manually
5. Check Configuration file naming and location

```bash
./mvnw spring-boot:run
```

The server will be available at: `http://localhost:9000`

You can verify a service's configuration is being served correctly by visiting:
```
http://localhost:9000/{service-name}/default
```

## Need Help?

If you encounter any issues, feel free to reach out and start a discussion via the Slack workspace.
