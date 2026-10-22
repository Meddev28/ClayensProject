# Portail logistique Clayens NP Morocco

Application web de suivi des bons de livraison et des factures pour les équipes Achats, Logistique, Magasin Réception, Magasin Expédition et Transit. Elle centralise les informations d’achat et de vente, les dates prévues, les statuts douaniers et les livraisons. Le dépôt contient également un script PowerShell indépendant pour trier des fichiers de commandes EDI.

## Architecture

| Composant | Technologie | Responsabilité |
| --- | --- | --- |
| Frontend | React 18, Vite, Axios | Connexion, tableaux de bord par rôle, formulaires, listes filtrables et fiches de suivi |
| Backend | Java 17, Spring Boot, Spring Security, JPA | API REST, authentification JWT, règles métier et accès aux données |
| Base de données | MySQL 8 | Comptes, bons de livraison et jetons de renouvellement |
| Serveur web | Nginx | Sert le frontend et relaie `/api` vers le backend |

Docker Compose démarre le frontend et le backend. MySQL reste un service externe : en configuration Windows par défaut, le backend le rejoint via `host.docker.internal:3306`. Les données sont donc conservées par votre serveur MySQL existant et non dans un nouveau conteneur de base de données.

Dans le backend, les contrôleurs reçoivent les requêtes, les services appliquent les règles métier, les dépôts Spring Data JPA accèdent aux entités MySQL, et Spring Security valide les jetons JWT. Le frontend appelle l’API via Axios et utilise le proxy Nginx en déploiement Docker.

## Fonctionnalités et habilitations

Les utilisateurs peuvent consulter les bons de livraison, rechercher par numéro, filtrer les listes et suivre les étapes de transport, douane et livraison. Les modifications sont contrôlées par le backend, et pas uniquement par les champs affichés dans l’interface.

| Profil | Modifications autorisées sur un bon |
| --- | --- |
| Achats | Tiers, date prévue, commentaire |
| Logistique | Tiers, type d’opération, commentaire |
| Magasin Réception et Expédition | Tiers, date prévue, livraison, commentaire |
| Transit | Tiers, douane, livraison, commentaire |
| Administrateur | Tous les champs et gestion des comptes |

La consultation et la création des bons nécessitent une connexion. La suppression des bons et les opérations de gestion des utilisateurs sont réservées à l’administrateur. L’inscription publique permet de créer un compte opérationnel, mais pas un compte administrateur.

## API

Les réponses suivent le format JSON `{ "success": true, "message": "...", "data": ... }`. En cas d’erreur, l’API renvoie un statut HTTP adapté et un message sans exposer les détails internes.

Un bon contient un numéro, un client ou fournisseur, une date prévue, un statut douanier (`EN_COURS`, `BLOQUE`, `DEADOUANE`), un statut de livraison (`OUI`, `NON`, `ENCOURS`), un type (`ACHAT`, `VENTE`) et un commentaire. Les champs de statut et la date sont facultatifs à la création.

| Méthode et chemin | Accès | Usage |
| --- | --- | --- |
| `POST /api/auth/login` | Public | Ouvrir une session et obtenir les jetons |
| `POST /api/auth/register` | Public | Créer un compte opérationnel |
| `POST /api/auth/refresh` | Jeton de renouvellement | Renouveler les jetons |
| `POST /api/auth/logout` | Public avec jeton | Révoquer un jeton de renouvellement |
| `GET /api/bls` | Connecté | Lister les bons, option `?operation=ACHAT` ou `VENTE` |
| `GET /api/bls/{numBl}` | Connecté | Consulter un bon |
| `POST /api/bls` | Connecté | Créer un bon |
| `PUT /api/bls/{numBl}` | Connecté | Modifier les champs permis par le profil |
| `DELETE /api/bls/{numBl}` | Administrateur | Supprimer un bon |
| `/api/users` | Administrateur | Lister, consulter, créer et supprimer les comptes |

Les endpoints de connexion, d’inscription et de santé ne demandent pas de jeton. Les autres endpoints exigent `Authorization: Bearer <accessToken>`. Les jetons d’accès expirent après 15 minutes et les jetons de renouvellement après 7 jours.

## Démarrage avec Docker

Prérequis : Docker Desktop avec Compose, un serveur MySQL accessible et une base de données déjà créée. Le compte MySQL doit pouvoir lire et modifier les tables de cette base. Sous Windows, autorisez les connexions du réseau Docker au serveur et au port MySQL utilisé, sans exposer ce port à des réseaux non fiables.

Créez `.env` à partir du modèle si nécessaire, puis renseignez les valeurs correspondant à votre serveur et votre base :

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

Paramètres essentiels :

```env
MYSQL_HOST=host.docker.internal
MYSQL_SERVER_PORT=3306
MYSQL_DATABASE=stage_projet_v2
MYSQL_USER=votre_utilisateur_mysql
MYSQL_PASSWORD=votre_mot_de_passe_mysql
JWT_SECRET=
```

Générez une clé Base64 aléatoire de 32 octets et reportez le résultat dans `JWT_SECRET` :

```powershell
$bytes = New-Object byte[] 32
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rng.GetBytes($bytes)
$rng.Dispose()
[Convert]::ToBase64String($bytes)
```

La valeur publique précédemment utilisée a été supprimée ; le fichier `.env` local a reçu une nouvelle clé, ce qui invalide les sessions déjà ouvertes. Ne partagez jamais `.env`.

Si la table des utilisateurs est vide, définissez également `INITIAL_ADMIN_USERNAME` et `INITIAL_ADMIN_PASSWORD` dans `.env`. Le mot de passe initial doit contenir au moins 12 caractères. Le backend crée uniquement ce premier compte administrateur ; il ne crée aucun compte avec un mot de passe par défaut. Si la base contient déjà des utilisateurs, ces paramètres de démarrage ne les modifient pas.

Démarrez l’application en mode Watch :

```powershell
docker compose up --build --watch
```

Laissez cette commande active pendant le développement. Une modification de `backend/` reconstruit le backend ; une modification de `frontend/` reconstruit le frontend. L’application est disponible sur `http://localhost`, l’API sur `http://localhost:8080` et son état sur `http://localhost/actuator/health`.

Pour arrêter les conteneurs, utilisez `Ctrl+C`, puis `docker compose down`. Cette configuration ne gère pas le service MySQL et ne supprime pas ses données. Ne supprimez pas l’ancien conteneur MySQL tant que vous n’avez pas vérifié où se trouvent ses données.

## Développement et vérification

Les tests du backend et le build de production du frontend se lancent séparément :

```powershell
Set-Location backend
mvn test
Set-Location ..\frontend
npm ci
npm run build
```

Le backend peut aussi être lancé localement avec `mvn spring-boot:run`, en définissant les variables de connexion MySQL et `JWT_SECRET` dans son environnement. Pour le frontend local, utilisez `npm run dev` depuis `frontend`; Vite sert l’interface sur le port 5173 et relaie `/api` vers `localhost:8080`.

Les migrations de schéma sont gérées actuellement par Hibernate avec `spring.jpa.hibernate.ddl-auto=update`. Pour une exploitation durable, sauvegardez régulièrement la base et envisagez un outil de migration versionnée avant de faire évoluer le schéma en production.
