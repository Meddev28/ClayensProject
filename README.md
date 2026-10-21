# 🚚 Logistics Management System

A full-stack logistics management web application built with **Spring Boot 3** on the backend and **React 18 + Vite** on the frontend. The system enables multi-department teams to manage delivery notes (Bons de Livraison) with role-based access control, JWT authentication, and a modern dark glassmorphism UI.

---

## 📋 Table of Contents

1. [Overview](#-overview)
2. [Architecture](#-architecture)
3. [Tech Stack](#-tech-stack)
4. [Project Structure](#-project-structure)
5. [Getting Started](#-getting-started)
6. [Default Accounts](#-default-accounts)
7. [API Reference](#-api-reference)
8. [Roles & Permissions](#-roles--permissions)
9. [Authentication Flow](#-authentication-flow)
10. [Database Schema](#-database-schema)
11. [Configuration](#-configuration)
12. [Running Modes](#-running-modes-h2-vs-mysql)

---

## 🔍 Overview

This application is designed for logistics companies that need to coordinate between multiple departments — purchasing, warehousing, shipping, transit, and administration — around a shared set of delivery documents.

### Key Features
- 🔐 **Secure JWT Authentication** — stateless access tokens (15 min) with rotating refresh tokens (7 days)
- 🔑 **BCrypt Password Hashing** — industry-standard password security
- 👥 **Role-Based Access Control** — 6 distinct department roles, each with specific field-level permissions
- 📦 **Delivery Note Management (BL)** — full CRUD with per-role field visibility
- 📊 **Live Dashboard** — KPI cards, color-coded status badges, real-time search
- 👤 **User Management** — admin can create, search, and delete accounts
- 🌙 **Modern UI** — dark glassmorphism design with smooth animations

---

## 🏗️ Architecture

The application follows a clean **3-tier architecture** with a fully decoupled frontend and backend communicating via a REST API.

```
┌─────────────────────────────────────────────────────────────┐
│                     BROWSER  (port 5173)                     │
│                                                              │
│   React 18 + Vite                                            │
│   ┌──────────┐  ┌────────────┐  ┌──────────┐  ┌─────────┐  │
│   │  Login   │  │  Dashboard │  │  BLsPage │  │  Users  │  │
│   │  Signup  │  │ (per role) │  │   KPIs   │  │  Page   │  │
│   └──────────┘  └────────────┘  └──────────┘  └─────────┘  │
│         │              │               │              │       │
│   ┌─────────────────────────────────────────────────────┐   │
│   │   AuthContext  +  Axios Interceptor (auto-refresh)   │   │
│   └─────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │  HTTP / JSON  (/api → :8080)
┌───────────────────────────▼─────────────────────────────────┐
│                  SPRING BOOT  (port 8080)                     │
│                                                              │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────────┐  │
│  │AuthController│   │ BLController │   │ UserController   │  │
│  └──────┬──────┘   └──────┬───────┘   └────────┬─────────┘  │
│         │                 │                     │             │
│  ┌──────▼──────┐   ┌──────▼───────┐   ┌────────▼─────────┐  │
│  │ AuthService  │   │  BLService   │   │   UserService    │  │
│  │  RTService   │   │ (role-aware) │   │                  │  │
│  └──────┬──────┘   └──────┬───────┘   └────────┬─────────┘  │
│         │                 │                     │             │
│  ┌──────▼─────────────────▼─────────────────────▼─────────┐  │
│  │              Spring Data JPA Repositories                │  │
│  └─────────────────────────┬───────────────────────────────┘  │
│                             │                                │
│  ┌──────────────────────────▼───────────────────────────┐   │
│  │   JwtAuthFilter  →  SecurityConfig  →  BCryptEncoder  │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │  JDBC
              ┌─────────────▼──────────────┐
              │  MySQL 8.0 (prod)           │
              │  H2 in-memory (dev)         │
              │  Database: stage_projet_v2  │
              └────────────────────────────┘
```

---

## 🛠️ Tech Stack

### Backend
| Technology | Version | Purpose |
|------------|---------|---------|
| Java | 17 | Core language |
| Spring Boot | 3.2.5 | Application framework |
| Spring Security | 6.x | Auth & authorization |
| Spring Data JPA | 3.x | Database abstraction |
| Hibernate | 6.4 | JPA implementation / ORM |
| JJWT | 0.12.3 | JWT generation & validation |
| BCrypt | Built-in | Password hashing |
| MySQL Connector/J | 8.3.0 | Production JDBC driver |
| H2 Database | Latest | In-memory DB for dev/test |
| Maven | 3.8+ | Build & dependency management |

### Frontend
| Technology | Version | Purpose |
|------------|---------|---------|
| React | 18.2 | UI framework |
| Vite | 5.4 | Build tool & dev server |
| React Router | v6.23 | Client-side routing |
| Axios | 1.6.8 | HTTP client + JWT interceptors |
| Vanilla CSS | — | Custom dark glassmorphism design system |

---

## 📁 Project Structure

```
logistics-management/
│
├── README.md
├── start.bat                          ← One-click launcher (Windows)
├── .gitignore
│
├── backend/                           ← Spring Boot REST API
│   ├── pom.xml
│   └── src/main/
│       ├── resources/
│       │   ├── application.properties          ← MySQL config (production)
│       │   └── application-h2.properties       ← H2 config (development)
│       └── java/com/stageprojet/
│           │
│           ├── StageProjetApplication.java     ← Entry point
│           ├── DataInitializer.java            ← Seeds default users on startup
│           │
│           ├── entity/                         ← JPA Entities
│           │   ├── User.java
│           │   ├── BonLivraison.java
│           │   ├── RefreshToken.java
│           │   ├── Role.java                   ← Enum: admin, achat, magrec...
│           │   ├── DouaneStatus.java            ← Enum: EN_COURS, BLOQUE, DEADOUANE
│           │   ├── LivraisonStatus.java         ← Enum: OUI, NON, ENCOURS
│           │   └── OperationType.java           ← Enum: ACHAT, VENTE
│           │
│           ├── repository/                     ← Spring Data JPA
│           │   ├── UserRepository.java
│           │   ├── BonLivraisonRepository.java
│           │   └── RefreshTokenRepository.java
│           │
│           ├── dto/                            ← Data Transfer Objects
│           │   ├── LoginRequest.java
│           │   ├── LoginResponse.java
│           │   ├── RegisterRequest.java
│           │   ├── RefreshTokenRequest.java
│           │   ├── BonLivraisonDto.java
│           │   ├── UserDto.java
│           │   └── ApiResponse.java            ← Generic response wrapper
│           │
│           ├── security/                       ← JWT & Spring Security
│           │   ├── JwtUtils.java               ← Token generation & validation
│           │   ├── JwtAuthFilter.java          ← Per-request auth filter
│           │   ├── UserDetailsServiceImpl.java
│           │   └── SecurityConfig.java         ← Security rules & CORS
│           │
│           ├── service/                        ← Business Logic
│           │   ├── AuthService.java
│           │   ├── RefreshTokenService.java
│           │   ├── BonLivraisonService.java
│           │   └── UserService.java
│           │
│           ├── controller/                     ← REST Controllers
│           │   ├── AuthController.java         ← /api/auth/**
│           │   ├── BonLivraisonController.java ← /api/bls/**
│           │   └── UserController.java         ← /api/users/**
│           │
│           └── exception/
│               └── GlobalExceptionHandler.java ← Centralized error handling
│
└── frontend/                          ← React + Vite SPA
    ├── package.json
    ├── vite.config.js                 ← Dev proxy: /api → :8080
    ├── index.html
    └── src/
        ├── main.jsx                   ← React entry point
        ├── App.jsx                    ← Router & protected routes
        ├── index.css                  ← Global design system
        │
        ├── api/
        │   └── axios.js               ← HTTP client with token refresh interceptor
        │
        ├── context/
        │   └── AuthContext.jsx        ← Global auth state + useAuth() hook
        │
        ├── components/
        │   ├── ProtectedRoute.jsx     ← Auth + role guard
        │   ├── Navbar.jsx             ← Top navigation bar
        │   ├── BLForm.jsx             ← Role-aware create/edit form
        │   └── BLTable.jsx            ← Searchable, color-coded BL table
        │
        └── pages/
            ├── Login.jsx              ← Animated login page
            ├── Signup.jsx             ← Registration with role selector
            ├── Dashboard.jsx          ← Role-based redirect
            ├── RoleDashboard.jsx      ← Unified dashboard (all 6 roles)
            ├── BLsPage.jsx            ← KPI cards + full BL listing
            └── UsersPage.jsx          ← User management
```

---

## 🚀 Getting Started

### Prerequisites
- **Java 17+**
- **Maven 3.8+**
- **Node.js 18+**
- **MySQL 8.0** *(optional — the app runs with H2 without it)*

---

### Quick Start — Development (H2, no MySQL needed)

```bash
# Terminal 1 — Start the backend
cd backend
java -jar target/stage-projet-0.0.1-SNAPSHOT.jar --spring.profiles.active=h2

# Terminal 2 — Start the frontend
cd frontend
npm install
npm run dev
```

> Open **http://localhost:5173**
> ⚠️ Data resets on every backend restart in H2 mode.

---

### Production Mode — With MySQL

```bash
# 1. Start MySQL service
net start MySQL80          # Windows (admin required)

# 2. Update credentials in backend/src/main/resources/application.properties
spring.datasource.password=YOUR_MYSQL_PASSWORD

# 3. Build & run the backend
cd backend
mvn clean package -DskipTests
java -jar target/stage-projet-0.0.1-SNAPSHOT.jar

# 4. Start the frontend
cd frontend
npm run dev
```

> The database `stage_projet_v2` is **created automatically** on first run.

---

### One-Click Start (Windows)
Double-click **`start.bat`** — starts MySQL, backend, and frontend automatically, then opens the browser.

---

## 👤 Default Accounts

The application seeds the following accounts on first startup (if no users exist):

| Username | Password | Role | Dashboard |
|----------|----------|------|-----------|
| `admin` | `admin123` | admin | Full access — all features |
| `achat` | `achat123` | achat | Purchase operations |
| `magrec` | `magrec123` | magrec | Receiving warehouse |
| `logistic` | `logistic123` | logistic | Logistics operations |
| `magexp` | `magexp123` | magexp | Shipping warehouse |
| `transit` | `transit123` | transit | Customs & transit |

> Passwords are stored as **BCrypt hashes** in the database — never in plain text.

---

## 📡 API Reference

### Base URL
```
http://localhost:8080
```

### Response Format
All responses follow this envelope:
```json
{
  "success": true,
  "message": "Optional description",
  "data": { }
}
```

---

### 🔐 Auth — `/api/auth`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/register` | ❌ Public | Register a new user |
| POST | `/api/auth/login` | ❌ Public | Login, receive token pair |
| POST | `/api/auth/refresh` | ❌ Public | Exchange refresh token for new pair |
| POST | `/api/auth/logout` | ✅ — | Revoke refresh token |

#### POST `/api/auth/login`
```json
// Request
{
  "nom": "admin",
  "motDePasse": "admin123"
}

// Response
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGci...",
    "refreshToken": "eyJhbGci...",
    "tokenType": "Bearer",
    "nom": "admin",
    "role": "admin",
    "userId": 1
  }
}
```

#### POST `/api/auth/register`
```json
{
  "nom": "new_user",
  "motDePasse": "securepass123",
  "fonction": "achat"
}
```
> `fonction` values: `admin` `achat` `magrec` `logistic` `magexp` `transit`

#### POST `/api/auth/refresh`
```json
{ "refreshToken": "eyJhbGci..." }
```
Returns a new `accessToken` + `refreshToken` pair. The old refresh token is **revoked immediately**.

---

### 📦 Bons de Livraison — `/api/bls`

> All routes require header: `Authorization: Bearer <accessToken>`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/bls` | Get all BLs |
| GET | `/api/bls?operation=ACHAT` | Filter by operation type |
| GET | `/api/bls/{numBl}` | Get a single BL by number |
| POST | `/api/bls` | Create a new BL |
| PUT | `/api/bls/{numBl}` | Update a BL (role-aware) |
| DELETE | `/api/bls/{numBl}` | Delete a BL |

#### BL Object
```json
{
  "id": 1,
  "numBl": "BL-2024-001",
  "clientFournisseur": "SONATRACH",
  "enlevement": "2024-10-20",
  "douane": "EN_COURS",
  "livraison": "NON",
  "operation": "ACHAT",
  "commentaire": "Handle with care",
  "createdByNom": "admin",
  "createdAt": "2024-10-21T17:30:49",
  "updatedAt": "2024-10-21T17:30:49"
}
```

| Field | Type | Values |
|-------|------|--------|
| `operation` | Enum | `ACHAT`, `VENTE` |
| `douane` | Enum | `EN_COURS`, `BLOQUE`, `DEADOUANE` |
| `livraison` | Enum | `OUI`, `NON`, `ENCOURS` |

---

### 👥 Users — `/api/users`

> All routes require header: `Authorization: Bearer <accessToken>`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/users` | List all users |
| GET | `/api/users/{id}` | Get user by ID |
| DELETE | `/api/users/{id}` | Delete a user |

---

## 🔒 Roles & Permissions

Each role has access to a specific set of BL fields. The backend enforces these restrictions server-side on every update request.

### Field-Level Access by Role

| Field | achat | magrec | logistic | magexp | transit | admin |
|-------|:-----:|:------:|:--------:|:------:|:-------:|:-----:|
| `numBl` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `clientFournisseur` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `operation` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `commentaire` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `enlevement` (date) | ❌ | ✅ | ❌ | ✅ | ❌ | ✅ |
| `douane` | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| `livraison` | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

### Frontend Route Protection

| Route | Allowed Roles |
|-------|--------------|
| `/login`, `/signup` | Public |
| `/achat` | achat, admin |
| `/magrec` | magrec, admin |
| `/logistic` | logistic, admin |
| `/magexp` | magexp, admin |
| `/transit` | transit, admin |
| `/admin` | admin only |
| `/bls` | All authenticated users |
| `/users` | All authenticated users |

---

## 🛡️ Authentication Flow

The application uses a **stateless dual-token strategy**:

```
User logs in
     │
     ▼
┌──────────────────────────────────────────────────┐
│  Access Token (15 min)  +  Refresh Token (7 days) │
└──────────────────────────────────────────────────┘
     │                            │
     │  Sent with every API call  │  Stored in localStorage
     │  Authorization: Bearer ... │
     ▼                            │
 API Request succeeds             │
     │                            │
     ▼ (if 401 — token expired)   │
 Axios Interceptor fires ─────────┘
     │
     ▼
 POST /api/auth/refresh
     │
     ▼
 New Access Token + New Refresh Token
 (old refresh token is revoked — rotation)
     │
     ▼
 Original request is retried automatically
```

### Token Details

| Token | Lifetime | Storage | Revocable |
|-------|----------|---------|-----------|
| Access Token | 15 minutes | `localStorage` | No (short-lived) |
| Refresh Token | 7 days | `localStorage` + DB | Yes — stored in `refresh_tokens` table |

- Signing algorithm: **HS384** (HMAC-SHA384)
- On logout, the refresh token is marked `revoked = true` in the database
- On next login, all previous refresh tokens for the user are revoked

---

## 🗃️ Database Schema

Tables are **auto-created by Hibernate** (`ddl-auto=update`) on first startup.

### `users`
```sql
id          BIGINT          PRIMARY KEY AUTO_INCREMENT
nom         VARCHAR(100)    NOT NULL UNIQUE          -- username
password    VARCHAR(255)    NOT NULL                 -- BCrypt hash
fonction    ENUM(...)       NOT NULL                 -- role
created_at  DATETIME(6)
```

### `bonlivraison`
```sql
id                  BIGINT          PRIMARY KEY AUTO_INCREMENT
num_bl              VARCHAR(100)    NOT NULL UNIQUE
client_fournisseur  VARCHAR(200)
enlevement          DATE
douane              ENUM('EN_COURS','BLOQUE','DEADOUANE')
livraison           ENUM('OUI','NON','ENCOURS')
operation           ENUM('ACHAT','VENTE')
commentaire         TEXT
created_by          BIGINT          FK → users(id)
created_at          DATETIME(6)
updated_at          DATETIME(6)
```

### `refresh_tokens`
```sql
id          BIGINT          PRIMARY KEY AUTO_INCREMENT
token       VARCHAR(512)    NOT NULL UNIQUE
user_id     BIGINT          NOT NULL FK → users(id)
expires_at  DATETIME(6)     NOT NULL
revoked     BIT(1)          NOT NULL DEFAULT 0
created_at  DATETIME(6)
```

---

## ⚙️ Configuration

### `application.properties` (MySQL — Production)
```properties
server.port=8080

spring.datasource.url=jdbc:mysql://localhost:3306/stage_projet_v2?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=YOUR_PASSWORD

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false

jwt.secret=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
jwt.expiration=900000          # 15 minutes
jwt.refresh-expiration=604800000  # 7 days

spring.web.cors.allowed-origins=http://localhost:5173
```

### `application-h2.properties` (H2 — Development)
```properties
spring.datasource.url=jdbc:h2:mem:stage_projet_v2;DB_CLOSE_DELAY=-1;MODE=MySQL
spring.datasource.username=sa
spring.datasource.password=

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console   # http://localhost:8080/h2-console

spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
```

---

## 🔄 Running Modes: H2 vs MySQL

| | H2 Mode | MySQL Mode |
|--|---------|------------|
| **Command** | `--spring.profiles.active=h2` | *(no flag)* |
| **Data persistence** | ❌ Lost on restart | ✅ Permanent |
| **MySQL required** | ❌ No | ✅ Yes |
| **DB console** | http://localhost:8080/h2-console | MySQL Workbench / CLI |
| **Use case** | Development & testing | Production |

### Switch to MySQL
```bash
# Make sure MySQL is running, then:
java -jar target/stage-projet-0.0.1-SNAPSHOT.jar
# (no profile flag = defaults to MySQL)
```

### View database via CLI
```bash
mysql -u root -p
USE stage_projet_v2;
SHOW TABLES;
SELECT id, nom, fonction FROM users;
SELECT id, num_bl, client_fournisseur, operation FROM bonlivraison;
```

---

## 🌐 UI Pages

| Route | Page | Description |
|-------|------|-------------|
| `/login` | Login | Glassmorphism card with animated gradient background |
| `/signup` | Register | New account form with department selector |
| `/achat` `/magrec` etc. | Role Dashboard | Quick-action cards + tabbed BL form (search / create) |
| `/bls` | BL Overview | 5 KPI stat cards, operation filter tabs, searchable color-coded table |
| `/users` | User Management | Live search, role badges, delete with confirmation, add user modal |

### BL Status Color Coding

| Status | Color | Meaning |
|--------|-------|---------|
| `EN_COURS` / `ENCOURS` | 🟠 Orange | In progress |
| `BLOQUE` / `NON` | 🔴 Red | Blocked / Not delivered |
| `DEADOUANE` / `OUI` | 🟢 Green | Cleared / Delivered |

---

## 📬 Ports Summary

| Service | Port | URL |
|---------|------|-----|
| React dev server | 5173 | http://localhost:5173 |
| Spring Boot API | 8080 | http://localhost:8080 |
| MySQL | 3306 | jdbc:mysql://localhost:3306 |
| H2 Console (dev) | 8080 | http://localhost:8080/h2-console |
