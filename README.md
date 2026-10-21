# 🚚 Système de Gestion Logistique — Full Stack

> **Migration complète de PHP procédural → Spring Boot 3 + React 18**  
> Application de gestion de bons de livraison avec authentification JWT, rôles utilisateurs, et interface moderne.

---

## 📋 Table des Matières

1. [État du Projet](#-état-du-projet)
2. [Prochaines Étapes](#-prochaines-étapes)
3. [Base de Données](#-base-de-données)
4. [Architecture](#-architecture)
5. [Stack Technologique](#-stack-technologique)
6. [Structure des Fichiers](#-structure-des-fichiers)
7. [Installation & Démarrage](#-installation--démarrage)
8. [Comptes par Défaut](#-comptes-par-défaut)
9. [API Reference](#-api-reference)
10. [Rôles et Permissions](#-rôles-et-permissions)
11. [Sécurité & JWT](#-sécurité--jwt)
12. [Schéma de la Base de Données](#-schéma-de-la-base-de-données)
13. [Variables de Configuration](#-variables-de-configuration)
14. [Passage de H2 à MySQL](#-passage-de-h2-à-mysql)

---

## ✅ État du Projet

### Ce qui est **terminé et fonctionnel**

| Composant | Statut | Détail |
|-----------|--------|--------|
| Backend Spring Boot | ✅ Complet | 33 fichiers Java, compilé, JAR prêt |
| Frontend React | ✅ Complet | 15 pages/composants, build Vite 247KB |
| Authentification JWT | ✅ Complet | Access token 15min + Refresh token 7 jours |
| Hash des mots de passe | ✅ Complet | BCrypt strength 10 |
| Base de données H2 | ✅ Active | Fonctionne sans MySQL |
| Base de données MySQL | ✅ Configurée | Prête dès que MySQL80 est démarré |
| Tableau de bord par rôle | ✅ Complet | 6 rôles avec champs spécifiques |
| CRUD Bons de Livraison | ✅ Complet | Créer, rechercher, modifier |
| Gestion Utilisateurs | ✅ Complet | Liste, recherche, suppression |
| Seed utilisateurs | ✅ Complet | 6 comptes créés au premier démarrage |
| Tests API E2E | ✅ Passés | Login, CRUD BL, refresh token |

### Ce qui **reste à faire** (améliorations futures)

| Fonctionnalité | Priorité | Description |
|----------------|----------|-------------|
| Pagination | Haute | Ajouter pagination côté serveur pour les BLs |
| Export PDF/Excel | Haute | Exporter la liste des BLs |
| Recherche avancée | Haute | Filtres par date, statut douane, livraison |
| Rôle admin — gestion users | Moyenne | Interface d'ajout d'utilisateurs côté admin |
| Notification toast | Moyenne | Feedback visuel sur actions (succès/erreur) |
| Tests unitaires | Moyenne | JUnit pour services, Jest pour composants |
| Docker Compose | Basse | Containerisation complète |
| HTTPS | Basse | Certificat SSL pour production |
| Logs d'audit | Basse | Traçabilité des modifications BL |

---

## 🚀 Prochaines Étapes

### Étape 1 — Démarrer MySQL (priorité immédiate)
```powershell
# Ouvrir PowerShell en tant qu'Administrateur
net start MySQL80
```
Puis relancer le backend **sans** le flag H2 :
```bash
cd c:\docs\stageproject.np\backend
java -jar target\stage-projet-0.0.1-SNAPSHOT.jar
```

### Étape 2 — Tester l'interface complète
1. Ouvrir **http://localhost:5173**
2. Se connecter avec `admin / admin123`
3. Créer des bons de livraison
4. Tester chaque rôle séparément

### Étape 3 — Déploiement (optionnel)
Pour un serveur de production :
```bash
# Backend — générer le JAR
mvn clean package -DskipTests

# Configurer application.properties avec vraie DB
# Lancer sur serveur
java -jar stage-projet-0.0.1-SNAPSHOT.jar

# Frontend — build statique
npm run build
# Servir le dossier dist/ via Nginx
```

---

## 🗄️ Base de Données

### Situation actuelle
Le projet utilise **deux profils de base de données** :

#### Profil H2 (actif en ce moment)
- **Type** : Base de données en mémoire (in-memory)
- **Fichier** : ❌ **Aucun fichier sur disque** — les données sont en RAM uniquement
- **Conséquence** : Toutes les données **disparaissent** au redémarrage du serveur
- **Avantage** : Fonctionne sans installer ni démarrer MySQL
- **Usage** : Développement et tests rapides

#### Profil MySQL (recommandé pour la persistance)
- **Type** : MySQL 8.0 installé sur votre machine
- **Nom de la base** : `stage_projet_v2`
- **Fichier physique** : `C:\ProgramData\MySQL\MySQL Server 8.0\Data\stage_projet_v2\`
- **Conséquence** : Les données **persistent** entre les redémarrages
- **Usage** : Production et développement normal

### Comment voir les données H2 maintenant
Pendant que le serveur tourne en mode H2 :
1. Ouvrir **http://localhost:8080/h2-console**
2. JDBC URL : `jdbc:h2:mem:stage_projet_v2`
3. Username : `sa` | Password : *(vide)*
4. Cliquer **Connect**

### Tables créées automatiquement
```sql
users           -- Utilisateurs avec mot de passe BCrypt
bonlivraison    -- Bons de livraison
refresh_tokens  -- Tokens JWT de rafraîchissement
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    NAVIGATEUR (port 5173)                    │
│                                                              │
│  React 18 + Vite                                            │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Login   │  │Dashboard  │  │  BLsPage │  │  Users   │  │
│  │ Signup   │  │(par rôle) │  │  KPIs    │  │  Page    │  │
│  └──────────┘  └───────────┘  └──────────┘  └──────────┘  │
│        │              │              │              │        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  AuthContext + Axios Interceptor (JWT auto-refresh)  │   │
│  └─────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTP / JSON
                            │ (proxy /api → :8080)
┌───────────────────────────▼─────────────────────────────────┐
│                  SPRING BOOT (port 8080)                     │
│                                                              │
│  ┌──────────────┐  ┌────────────────┐  ┌────────────────┐  │
│  │ AuthController│  │ BLController   │  │ UserController │  │
│  └──────┬───────┘  └───────┬────────┘  └───────┬────────┘  │
│         │                  │                    │            │
│  ┌──────▼───────┐  ┌───────▼────────┐  ┌───────▼────────┐  │
│  │  AuthService  │  │  BLService     │  │  UserService   │  │
│  │  RTService    │  │  (role-aware)  │  │                │  │
│  └──────┬───────┘  └───────┬────────┘  └───────┬────────┘  │
│         │                  │                    │            │
│  ┌──────▼──────────────────▼────────────────────▼────────┐  │
│  │   Spring Data JPA Repositories                         │  │
│  └──────────────────────────┬──────────────────────────┘  │
│                              │                              │
│  ┌───────────────────────────▼──────────────────────────┐  │
│  │  JwtAuthFilter → SecurityConfig → BCryptEncoder        │  │
│  └──────────────────────────────────────────────────────┘  │
└───────────────────────────┬─────────────────────────────────┘
                            │ JDBC
              ┌─────────────▼─────────────┐
              │   H2 (dev) / MySQL (prod)  │
              │   Database: stage_projet_v2│
              └───────────────────────────┘
```

---

## 🛠️ Stack Technologique

### Backend
| Technologie | Version | Usage |
|-------------|---------|-------|
| Java | 17 | Langage principal |
| Spring Boot | 3.2.5 | Framework principal |
| Spring Security | 6.x | Authentification + autorisation |
| Spring Data JPA | 3.x | ORM + repositories |
| Hibernate | 6.4 | JPA implementation |
| JJWT | 0.12.3 | Génération et validation des JWT |
| BCrypt | Built-in | Hash des mots de passe |
| MySQL Connector | 8.3.0 | Driver JDBC MySQL |
| H2 Database | Latest | Base in-memory pour dev |
| Maven | 3.8+ | Build et dépendances |

### Frontend
| Technologie | Version | Usage |
|-------------|---------|-------|
| React | 18.2 | Framework UI |
| Vite | 5.4 | Build tool et dev server |
| React Router | v6.23 | Navigation SPA |
| Axios | 1.6.8 | Client HTTP + intercepteurs JWT |
| Vanilla CSS | — | Styles glassmorphism dark mode |
| Google Fonts | Inter | Typographie |

---

## 📁 Structure des Fichiers

```
c:\docs\stageproject.np\
│
├── README.md                    ← Ce fichier
├── start.bat                    ← Lanceur double-clic (MySQL + Backend + Frontend)
│
├── backend/
│   ├── pom.xml                  ← Dépendances Maven
│   └── src/main/
│       ├── resources/
│       │   ├── application.properties       ← Config MySQL (prod)
│       │   └── application-h2.properties    ← Config H2 (dev, sans MySQL)
│       └── java/com/stageprojet/
│           ├── StageProjetApplication.java  ← Point d'entrée Spring Boot
│           ├── DataInitializer.java         ← Seed 6 utilisateurs au démarrage
│           │
│           ├── entity/
│           │   ├── User.java                ← Entité utilisateur
│           │   ├── BonLivraison.java        ← Entité bon de livraison
│           │   ├── RefreshToken.java        ← Entité token de rafraîchissement
│           │   ├── Role.java                ← Enum: admin, achat, magrec...
│           │   ├── DouaneStatus.java        ← Enum: EN_COURS, BLOQUE, DEADOUANE
│           │   ├── LivraisonStatus.java     ← Enum: OUI, NON, ENCOURS
│           │   └── OperationType.java       ← Enum: ACHAT, VENTE
│           │
│           ├── repository/
│           │   ├── UserRepository.java
│           │   ├── BonLivraisonRepository.java
│           │   └── RefreshTokenRepository.java
│           │
│           ├── dto/
│           │   ├── LoginRequest.java        ← { nom, motDePasse }
│           │   ├── LoginResponse.java       ← { accessToken, refreshToken, nom, role, userId }
│           │   ├── RegisterRequest.java     ← { nom, motDePasse, fonction }
│           │   ├── RefreshTokenRequest.java ← { refreshToken }
│           │   ├── BonLivraisonDto.java     ← Données BL complètes
│           │   ├── UserDto.java             ← Données utilisateur (sans password)
│           │   └── ApiResponse.java         ← Wrapper { success, message, data }
│           │
│           ├── security/
│           │   ├── JwtUtils.java            ← Génère/valide les tokens JWT (HS384)
│           │   ├── JwtAuthFilter.java       ← Filtre HTTP extrait Bearer token
│           │   ├── UserDetailsServiceImpl.java ← Charge user depuis DB
│           │   └── SecurityConfig.java      ← CORS, CSRF-off, routes publiques/privées
│           │
│           ├── service/
│           │   ├── AuthService.java         ← Register, login, refresh, logout
│           │   ├── RefreshTokenService.java ← Création, validation, révocation
│           │   ├── BonLivraisonService.java ← CRUD BL + contrôle par rôle
│           │   └── UserService.java         ← Liste et suppression users
│           │
│           ├── controller/
│           │   ├── AuthController.java      ← POST /api/auth/*
│           │   ├── BonLivraisonController.java ← /api/bls/*
│           │   └── UserController.java      ← /api/users/*
│           │
│           └── exception/
│               └── GlobalExceptionHandler.java ← Gestion erreurs 400/401/500
│
└── frontend/
    ├── package.json
    ├── vite.config.js               ← Proxy /api → http://localhost:8080
    ├── index.html
    └── src/
        ├── main.jsx                 ← Point d'entrée React
        ├── App.jsx                  ← Router + routes protégées
        ├── index.css                ← Design system (dark glassmorphism)
        │
        ├── api/
        │   └── axios.js             ← Client HTTP avec intercepteur JWT auto-refresh
        │
        ├── context/
        │   └── AuthContext.jsx      ← État global auth + hook useAuth()
        │
        ├── components/
        │   ├── ProtectedRoute.jsx   ← Guard route par auth + rôle
        │   ├── Navbar.jsx           ← Navigation avec avatar rôle coloré
        │   ├── BLForm.jsx           ← Formulaire BL sensible au rôle
        │   ├── BLTable.jsx          ← Table avec filtres et badges colorés
        │   └── *.css
        │
        └── pages/
            ├── Login.jsx            ← Page connexion (orbes animées)
            ├── Signup.jsx           ← Page inscription avec sélecteur rôle
            ├── Dashboard.jsx        ← Redirecteur → dashboard du rôle
            ├── RoleDashboard.jsx    ← Dashboard unifié (6 rôles)
            ├── BLsPage.jsx          ← KPIs + table BLs filtrée
            ├── UsersPage.jsx        ← Gestion utilisateurs (admin)
            ├── AchatDashboard.jsx   ← Wrapper rôle achat
            ├── MagrecDashboard.jsx
            ├── LogisticDashboard.jsx
            ├── MagexpDashboard.jsx
            ├── TransitDashboard.jsx
            └── *.css
```

---

## ⚙️ Installation & Démarrage

### Prérequis
- **Java 17+** — `java -version`
- **Maven 3.8+** — `mvn -version`
- **Node.js 18+** — `node -version`
- **MySQL 8.0** *(optionnel — l'app tourne sans avec H2)*

---

### Option A — Démarrage rapide (sans MySQL, H2 in-memory)

```bash
# Terminal 1 — Backend
cd c:\docs\stageproject.np\backend
java -jar target\stage-projet-0.0.1-SNAPSHOT.jar --spring.profiles.active=h2

# Terminal 2 — Frontend
cd c:\docs\stageproject.np\frontend
npm run dev
```

✅ Application disponible sur **http://localhost:5173**  
⚠️ Les données sont **perdues** à chaque redémarrage du backend

---

### Option B — Avec MySQL (données persistantes)

```powershell
# Étape 1 — Démarrer MySQL (PowerShell administrateur)
net start MySQL80

# Étape 2 — Backend (sans flag h2)
cd c:\docs\stageproject.np\backend
java -jar target\stage-projet-0.0.1-SNAPSHOT.jar

# Étape 3 — Frontend
cd c:\docs\stageproject.np\frontend
npm run dev
```

---

### Option C — Double-clic sur start.bat

```
c:\docs\stageproject.np\start.bat
```
Lance MySQL + Backend + Frontend automatiquement et ouvre le navigateur.

---

### Recompiler depuis les sources

```bash
# Backend
cd c:\docs\stageproject.np\backend
mvn clean package -DskipTests

# Frontend
cd c:\docs\stageproject.np\frontend
npm install
npm run build      # build production
npm run dev        # serveur développement
```

---

## 👤 Comptes par Défaut

Créés automatiquement au premier démarrage (si aucun utilisateur n'existe) :

| Utilisateur | Mot de passe | Rôle | Accès |
|-------------|-------------|------|-------|
| `admin` | `admin123` | admin | Tout + gestion users |
| `achat` | `achat123` | achat | BLs achat/vente |
| `magrec` | `magrec123` | magrec | BLs + date enlèvement |
| `logistic` | `logistic123` | logistic | BLs logistique |
| `magexp` | `magexp123` | magexp | BLs + date enlèvement |
| `transit` | `transit123` | transit | BLs + douane + livraison |

> Les mots de passe sont hashés avec **BCrypt** en base de données.

---

## 📡 API Reference

### Base URL
```
http://localhost:8080
```

### Format des réponses
Toutes les réponses suivent ce format :
```json
{
  "success": true,
  "message": "Description optionnelle",
  "data": { ... }
}
```

---

### 🔐 Authentification — `/api/auth`

#### POST `/api/auth/login`
Connexion utilisateur.

**Request Body:**
```json
{
  "nom": "admin",
  "motDePasse": "admin123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzM4NCJ9...",
    "refreshToken": "eyJhbGciOiJIUzM4NCJ9...",
    "tokenType": "Bearer",
    "nom": "admin",
    "role": "admin",
    "userId": 1
  }
}
```

---

#### POST `/api/auth/register`
Inscription d'un nouvel utilisateur.

**Request Body:**
```json
{
  "nom": "nouveau_user",
  "motDePasse": "motdepasse123",
  "fonction": "achat"
}
```
> Valeurs de `fonction` : `admin`, `achat`, `magrec`, `logistic`, `magexp`, `transit`

---

#### POST `/api/auth/refresh`
Renouveler le token d'accès (sans se reconnecter).

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzM4NCJ9..."
}
```

**Response:** Même format que `/login` avec nouveaux tokens.

---

#### POST `/api/auth/logout`
Invalider le refresh token.

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzM4NCJ9..."
}
```

---

### 📦 Bons de Livraison — `/api/bls`

> **Toutes ces routes nécessitent le header :** `Authorization: Bearer <accessToken>`

#### GET `/api/bls`
Récupérer tous les bons de livraison.

**Query Params (optionnel):**
- `?operation=ACHAT` — Filtrer par opération (ACHAT ou VENTE)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "numBl": "BL-2024-001",
      "clientFournisseur": "SONATRACH",
      "enlevement": "2024-10-15",
      "douane": "EN_COURS",
      "livraison": "NON",
      "operation": "ACHAT",
      "commentaire": "Livraison urgente",
      "createdByNom": "admin",
      "createdAt": "2024-10-21T16:12:08",
      "updatedAt": "2024-10-21T16:12:08"
    }
  ]
}
```

---

#### GET `/api/bls/{numBl}`
Récupérer un BL par son numéro.

**Exemple:** `GET /api/bls/BL-2024-001`

---

#### POST `/api/bls`
Créer un nouveau bon de livraison.

**Request Body:**
```json
{
  "numBl": "BL-2024-002",
  "clientFournisseur": "NAFTAL",
  "enlevement": "2024-10-20",
  "douane": "EN_COURS",
  "livraison": "NON",
  "operation": "VENTE",
  "commentaire": "Marchandises fragiles"
}
```

> Les champs disponibles dépendent du rôle de l'utilisateur connecté (voir section Permissions).

---

#### PUT `/api/bls/{numBl}`
Modifier un bon de livraison existant.

**Exemple:** `PUT /api/bls/BL-2024-001`

**Request Body:** Mêmes champs que POST (seuls les champs envoyés sont modifiés).

---

#### DELETE `/api/bls/{numBl}`
Supprimer un bon de livraison.

**Exemple:** `DELETE /api/bls/BL-2024-001`

---

### 👥 Utilisateurs — `/api/users`

> **Toutes ces routes nécessitent le header :** `Authorization: Bearer <accessToken>`

#### GET `/api/users`
Lister tous les utilisateurs.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "nom": "admin",
      "fonction": "admin",
      "createdAt": "2024-10-21T16:05:45"
    }
  ]
}
```

#### GET `/api/users/{id}`
Récupérer un utilisateur par son ID.

#### DELETE `/api/users/{id}`
Supprimer un utilisateur.

---

## 🔒 Rôles et Permissions

### Champs accessibles par rôle dans les BLs

| Champ | achat | magrec | logistic | magexp | transit | admin |
|-------|-------|--------|----------|--------|---------|-------|
| `numBl` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `clientFournisseur` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `operation` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `commentaire` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `enlevement` (date) | ❌ | ✅ | ❌ | ✅ | ❌ | ✅ |
| `douane` | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| `livraison` | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

### Accès aux routes frontend

| Route | Rôles autorisés |
|-------|----------------|
| `/login`, `/signup` | Public |
| `/achat` | achat, admin |
| `/magrec` | magrec, admin |
| `/logistic` | logistic, admin |
| `/magexp` | magexp, admin |
| `/transit` | transit, admin |
| `/admin` | admin uniquement |
| `/bls` | Tous les connectés |
| `/users` | Tous les connectés |

---

## 🛡️ Sécurité & JWT

### Fonctionnement des tokens

```
Connexion (login)
    │
    ▼
Access Token (15 minutes)    +    Refresh Token (7 jours)
    │                                    │
    │ Envoyé dans chaque requête         │ Stocké côté client
    │ Header: Authorization: Bearer ...  │
    ▼                                    │
Requête API (< 15min)                   │
    │                                    │
    ▼ (si 401 - token expiré)           │
Axios Interceptor appelle ──────────────┘
POST /api/auth/refresh
    │
    ▼
Nouveau Access Token + Nouveau Refresh Token
(rotation automatique — l'ancien est révoqué)
    │
    ▼
Requête originale retentée automatiquement
```

### Valeurs des tokens

| Token | Durée | Stockage |
|-------|-------|---------|
| Access Token | 15 minutes | `localStorage` |
| Refresh Token | 7 jours | `localStorage` |

> Les refresh tokens sont stockés en base de données et peuvent être révoqués individuellement ou tous à la fois (logout).

### Algorithme JWT
- Algorithme de signature : **HS384** (HMAC SHA-384)
- Clé secrète configurable dans `application.properties`

---

## 🗃️ Schéma de la Base de Données

### Table `users`
```sql
CREATE TABLE users (
    id          BIGINT AUTO_INCREMENT PRIMARY KEY,
    nom         VARCHAR(100) NOT NULL UNIQUE,     -- username
    password    VARCHAR(255) NOT NULL,             -- BCrypt hash
    fonction    ENUM('admin','achat','magrec',
                     'logistic','magexp','transit') NOT NULL,
    created_at  DATETIME                          -- auto-rempli
);
```

### Table `bonlivraison`
```sql
CREATE TABLE bonlivraison (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    num_bl              VARCHAR(100) NOT NULL UNIQUE,  -- numéro BL
    client_fournisseur  VARCHAR(200),
    enlevement          DATE,                          -- date enlèvement
    douane              ENUM('EN_COURS','BLOQUE','DEADOUANE'),
    livraison           ENUM('OUI','NON','ENCOURS'),
    operation           ENUM('ACHAT','VENTE'),
    commentaire         TEXT,
    created_by          BIGINT REFERENCES users(id),  -- qui a créé
    created_at          DATETIME,
    updated_at          DATETIME
);
```

### Table `refresh_tokens`
```sql
CREATE TABLE refresh_tokens (
    id          BIGINT AUTO_INCREMENT PRIMARY KEY,
    token       VARCHAR(512) NOT NULL UNIQUE,
    user_id     BIGINT NOT NULL REFERENCES users(id),
    expires_at  DATETIME NOT NULL,
    revoked     BOOLEAN NOT NULL DEFAULT FALSE,
    created_at  DATETIME
);
```

> Hibernate crée ces tables **automatiquement** au démarrage (`ddl-auto=update`).

---

## ⚙️ Variables de Configuration

### `application.properties` (MySQL)
```properties
# Serveur
server.port=8080

# MySQL
spring.datasource.url=jdbc:mysql://localhost:3306/stage_projet_v2?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root          # ← Changer si nécessaire

# JPA
spring.jpa.hibernate.ddl-auto=update     # auto-création des tables
spring.jpa.show-sql=false

# JWT
jwt.secret=404E635266556A586E3272357...  # ← Changer en production !
jwt.expiration=900000                    # 15 minutes (millisecondes)
jwt.refresh-expiration=604800000         # 7 jours (millisecondes)

# CORS — origine autorisée
spring.web.cors.allowed-origins=http://localhost:5173
```

### `application-h2.properties` (H2 - dev)
```properties
# H2 en mémoire
spring.datasource.url=jdbc:h2:mem:stage_projet_v2;DB_CLOSE_DELAY=-1;MODE=MySQL
spring.datasource.username=sa
spring.datasource.password=

# Console H2 (debug)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

---

## 🔄 Passage de H2 à MySQL

### Actuellement (H2)
```bash
java -jar target\stage-projet-0.0.1-SNAPSHOT.jar --spring.profiles.active=h2
```

### Passer à MySQL
```powershell
# 1. Démarrer MySQL (admin requis)
net start MySQL80

# 2. Vérifier la connexion MySQL
mysql -u root -proot -e "SELECT VERSION();"

# 3. Lancer le backend sans profil h2
cd c:\docs\stageproject.np\backend
java -jar target\stage-projet-0.0.1-SNAPSHOT.jar

# La base stage_projet_v2 est créée automatiquement
```

### Modifier les credentials MySQL
Éditer `c:\docs\stageproject.np\backend\src\main\resources\application.properties` :
```properties
spring.datasource.username=root
spring.datasource.password=VOTRE_MOT_DE_PASSE
```
Puis recompiler :
```bash
mvn package -DskipTests
```

---

## 🌐 Frontend — Pages de l'Interface

### `/login` — Connexion
- Formulaire glassmorphism avec orbes animées en arrière-plan
- Validation des champs
- Redirection automatique vers le dashboard du rôle connecté

### `/signup` — Inscription
- Sélecteur de département (rôle)
- Validation et feedback d'erreur

### `/[role]` — Dashboard par rôle
- En-tête avec icône et couleur spécifiques au rôle
- Cartes de navigation rapide (BLs, Nouveau BL, Utilisateurs)
- Onglets : Rechercher/Modifier | Nouveau BL

### `/bls` — Bons de Livraison
- **5 cartes KPI** : Total BLs, Achats, Ventes, Dédouanés, Livrés
- **Filtres par onglet** : Tous | Achats | Ventes
- **Recherche en temps réel** par N° BL ou client/fournisseur
- **Badges colorés** :
  - 🟠 `EN_COURS` / `ENCOURS` — orange
  - 🔴 `BLOQUE` / `NON` — rouge
  - 🟢 `DEADOUANE` / `OUI` — vert

### `/users` — Gestion Utilisateurs
- Liste avec badge de département coloré
- Recherche en temps réel
- Suppression avec confirmation
- Modal d'ajout de nouvel utilisateur

---

## 📞 Support & Maintenance

### Logs du backend
```bash
# Voir les logs en temps réel
java -jar target\stage-projet-0.0.1-SNAPSHOT.jar --logging.level.com.stageprojet=DEBUG
```

### Console H2 (mode dev)
- URL : **http://localhost:8080/h2-console**
- JDBC URL : `jdbc:h2:mem:stage_projet_v2`
- User : `sa` | Password : *(vide)*

### Ports utilisés
| Service | Port |
|---------|------|
| React dev server | 5173 |
| Spring Boot API | 8080 |
| MySQL | 3306 |
| H2 Console | 8080/h2-console |

---

*Projet migré de PHP procédural vers Spring Boot 3 + React 18 — Octobre 2024*
