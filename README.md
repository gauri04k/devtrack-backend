# DevTrack

DevTrack is a Spring Boot REST API for tracking personal developer progress. It provides JWT-based registration and login, and API operations for a user's projects, milestones, skills, daily logs, notes, and dashboard summaries.

> **Repository scope:** This checkout contains the Java backend API. There is no React/Vite frontend source or `package.json` in this repository. A separately hosted or separately checked-out frontend can call the API at `http://localhost:8080`; the backend currently allows browser requests from `http://localhost:5173` through its CORS configuration.

## Features

- Register and authenticate users using JWT bearer tokens
- Create and manage projects and their milestones
- Track skills and their statuses and target dates
- Record daily logs, filter logs by date, and retrieve weekly summaries
- Create, search, paginate, update, and delete notes
- Retrieve dashboard aggregates, including skill counts, project counts, weekly hours, and recent activity
- Inspect the REST API using Swagger UI and OpenAPI
- Persist data using Spring Data JPA and MySQL

## Technology

### Backend

- Java 17
- Spring Boot 4.1.2
- Spring MVC, Spring Data JPA, and Hibernate
- Spring Security and JWT (JJWT)
- MySQL
- Maven Wrapper (Maven 3.9.16)
- Springdoc OpenAPI

## Repository Layout

```text
devtrack/
├── src/
│   ├── main/
│   │   ├── java/com/devtrack/
│   │   │   ├── controller/       # REST API endpoints
│   │   │   ├── dto/              # API request and response objects
│   │   │   ├── entity/           # JPA entities
│   │   │   ├── exception/        # API exception handling
│   │   │   ├── mapper/           # Entity/DTO mapping
│   │   │   ├── repository/       # Spring Data repositories
│   │   │   ├── security/         # JWT and Spring Security
│   │   │   └── service/          # Business logic
│   │   └── resources/
│   │       ├── application.properties
│   │       └── application-local.yml
│   └── test/java/                # Automated tests
├── .mvn/wrapper/                 # Maven Wrapper configuration
├── mvnw
├── mvnw.cmd
└── pom.xml
```

## Prerequisites

- JDK 17 or newer
- MySQL Server reachable from the application
- A terminal (the examples below use Windows PowerShell)

The Maven Wrapper downloads and runs the configured Maven distribution, so a separate Maven installation is not required.

## Database Configuration

The default configuration is in `src/main/resources/application.properties`. It sets:

| Setting | Default |
| --- | --- |
| JDBC URL | `jdbc:mysql://localhost:3307/devtrack_db?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=Asia/Kolkata` |
| Application port | `8080` |
| Hibernate schema mode | `update` |
| SQL logging | Enabled |

Make sure MySQL is running and listening on port `3307`, or override the JDBC URL to match your local MySQL installation. The URL requests that MySQL create `devtrack_db` if it does not exist; the configured database account must have permission to create it. Otherwise, create the database yourself before starting the API.

## Configure and Run with the `local` Profile

`src/main/resources/application-local.yml` is a Spring profile-specific configuration file. Spring Boot does **not** load it just because the file exists: you must explicitly activate the `local` profile.

The current local profile provides these defaults:

| Setting | Value |
| --- | --- |
| Database username | `local_username` |
| Database password | `local_password` |
| JWT expiration | `86400000` milliseconds (24 hours) |
| JWT signing secret | A development-only value in `application-local.yml` |

These are local-development defaults only. Do not reuse the checked-in database password or JWT secret in a shared, staging, or production environment. Use a private secret of at least 32 bytes for the HMAC JWT key, and do not commit real credentials.

### Option A: Use the profile defaults

From the repository root in PowerShell:

```powershell
.\mvnw.cmd spring-boot:run "-Dspring-boot.run.profiles=local"
```

This uses the local profile's `root` / `root` database credentials. It still uses the JDBC URL and application port configured in `application.properties`.

### Option B: Override credentials and JWT secret with environment variables

Environment variables take precedence over values in the configuration files. Set them in the same PowerShell window that will run the backend:

```powershell
$env:SPRING_DATASOURCE_URL = "jdbc:mysql://localhost:3307/devtrack_db?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=Asia/Kolkata"
$env:SPRING_DATASOURCE_USERNAME = "your_mysql_username"
$env:SPRING_DATASOURCE_PASSWORD = "your_mysql_password"
$env:JWT_SECRET = "replace-with-a-private-random-secret-at-least-32-bytes-long"
$env:JWT_EXPIRATION = "86400000"

.\mvnw.cmd spring-boot:run "-Dspring-boot.run.profiles=local"
```

Replace the example database credentials and JWT secret with values for your machine. These PowerShell variables apply only to the current terminal session; set them again in a new terminal. Do not paste production secrets into source files or commit them.

To clear the values from the current PowerShell session:

```powershell
Remove-Item Env:SPRING_DATASOURCE_URL, Env:SPRING_DATASOURCE_USERNAME, Env:SPRING_DATASOURCE_PASSWORD, Env:JWT_SECRET, Env:JWT_EXPIRATION -ErrorAction SilentlyContinue
```

You can also configure private local values directly in `src/main/resources/application-local.yml`, but keep that file limited to development-only values and never commit real/shared secrets. The environment-variable approach avoids editing configuration files for machine-specific credentials.

### Run without the `local` profile

The base `application.properties` does not define the database username, database password, or JWT properties. Supply them through environment variables (as shown above), or configure them in an appropriate profile, before starting without `local`.

## Start the API

From the repository root:

```powershell
.\mvnw.cmd spring-boot:run "-Dspring-boot.run.profiles=local"
```

The API listens on `http://localhost:8080` unless `server.port` is changed. Leave this terminal running while making API requests.

To build the application:

```powershell
.\mvnw.cmd clean package
```

After building, run the packaged application with the local profile:

```powershell
java -jar target\devtrack-0.0.1-SNAPSHOT.jar --spring.profiles.active=local
```

## Verify the Application

- Swagger UI: `http://localhost:8080/swagger-ui/index.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`
- Actuator health endpoint: `http://localhost:8080/actuator/health`

Swagger and OpenAPI documentation are configured as public endpoints. Other API operations require authentication.

## Authentication

Register or log in using the public endpoints:

- `POST /api/auth/register`
- `POST /api/auth/login`

Both return an authentication response containing a JWT. For protected requests, send the token in the `Authorization` HTTP header using the `Bearer` scheme.

The JWT expiration is configured in milliseconds by `jwt.expiration` (`JWT_EXPIRATION` as an environment variable). The default local value is one day.

## API Overview

Use Swagger UI for request schemas, parameters, and response details. The current controller route groups are:

| API group | Base routes |
| --- | --- |
| Authentication | `/api/auth/register`, `/api/auth/login` |
| Users | `/api/users` |
| Projects | `/api/users/{userId}/projects` |
| Project milestones | `/api/projects/{projectId}/milestones`, `/api/milestones/{milestoneId}` |
| Skills | `/api/users/{userId}/skills` |
| Daily logs | `/api/users/{userId}/logs` |
| Notes | `/api/users/{userId}/notes` |
| Dashboard | `/api/users/{userId}/dashboard` |

The user-scoped routes include operations for listing and filtering resources as well as reading, creating, updating, and deleting them. For example, notes support pagination and search, skills and projects support status filters, and daily logs support date filtering and weekly summaries.

User-scoped path parameters identify the requested account/resource context; they are not an authorization mechanism. Protected operations must still be authorized by the backend using the authenticated identity and resource ownership.

## CORS

The current security configuration allows browser requests from `http://localhost:5173`, with the configured API methods and headers. If you run a separate frontend on another origin, update the allowed origin in `SecurityConfig` to match that frontend. Do not use a broad wildcard origin together with credentials.

## Tests and Development Commands

Run the backend tests from the repository root:

```powershell
.\mvnw.cmd test
```

The current test sources include focused security regression tests for password hashing and rejecting an attempt to update another user's account.

## Configuration Reference

| Purpose | Configuration |
| --- | --- |
| Base application settings | `src/main/resources/application.properties` |
| Local development overrides | `src/main/resources/application-local.yml` |
| Activate local profile | `-Dspring-boot.run.profiles=local` when using `spring-boot:run`, or `--spring.profiles.active=local` when running the packaged JAR |
| Database connection | `spring.datasource.*` / `SPRING_DATASOURCE_*` |
| JWT secret and lifetime | `jwt.secret`, `jwt.expiration` / `JWT_SECRET`, `JWT_EXPIRATION` |
| Server port | `server.port` / `SERVER_PORT` |
| Security and allowed browser origin | `src/main/java/com/devtrack/security/SecurityConfig.java` |

## Local Development and Production Notes

- `spring.jpa.hibernate.ddl-auto=update` is convenient for local development. Use a deliberate schema-migration strategy before production rather than relying on Hibernate to update a production schema.
- SQL statement logging is enabled in the base configuration. Consider disabling it or reviewing its output policy outside local development.
- Replace development credentials and JWT signing secrets before deploying.
- Configure HTTPS and production-appropriate CORS origins when exposing the API beyond a local machine.
- The frontend is not included in this repository; its routes, build commands, and browser-side storage are therefore not documented here.

## Troubleshooting

| Symptom | Checks |
| --- | --- |
| Could not connect to MySQL | Confirm MySQL is running, the JDBC URL host/port is correct, and the configured credentials work. |
| Unknown database | Create `devtrack_db` manually or use an account with permission for `createDatabaseIfNotExist=true`. |
| Missing `jwt.secret` or `jwt.expiration` | Activate the `local` profile or provide `JWT_SECRET` and `JWT_EXPIRATION`. |
| JWT key is too short | Configure a private HMAC signing secret with at least 32 bytes. |
| Browser reports a CORS error | Confirm the frontend origin matches the allowlisted origin in `SecurityConfig`. |
| Port 8080 is already in use | Stop the conflicting process or set `SERVER_PORT` to another available port. |

## Author

Gauri Kapnadis
