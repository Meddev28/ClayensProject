<h1>PLATEFORME DE SUPERVISION SUPPLY CHAIN ET GESTION INTEGREE DES BONS DE LIVRAISON</h1>

<p>
Projet de Fin d Etudes presente pour l obtention du Diplome Universitaire de Technologie en Genie Informatique<br />
Etablissement : Ecole Superieure de Technologie de Sidi Bennour, Universite Chouaib Doukkali<br />
Entreprise d accueil : Societe Multinationale Clayens NP Morocco, Site Industriel de Mohammedia<br />
Auteur : Mohammed Hedda<br />
Encadrant pedagogique : Pr. Saidi Abdelali<br />
Annee universitaire : 2023 / 2024
</p>

<hr />

<h2>1. CONTEXTE INDUSTRIEL ET PRESENTATION DE L ENTREPRISE</h2>

<p>
Le groupe international Clayens est l un des leaders mondiaux dans la transformation des polymeres, des composites et des composants metalliques de haute precision. Sa filiale marocaine, Clayens NP Morocco, implantee au sein de la zone industrielle de Mohammedia, deploie son expertise dans l injection plastique technique et l assemblage de sous ensembles complexes destines principalement aux marches de pointe :
</p>

<ul>
  <li><strong>Secteur Automobile :</strong> pieces fonctionnelles sous capot, elements d habitacle, boitiers de connectique et systemes d eclairage destines aux constructeurs et equipementiers de rang 1.</li>
  <li><strong>Secteur Electrique et Electronique :</strong> coffrets industriels, pieces d isolation electrique et appareillages modulaires.</li>
  <li><strong>Secteur Medical et Aeronautique :</strong> dispositifs techniques repondant a des exigences de purete et de tolerance dimensionnelle tres strictes.</li>
</ul>

<p>
L usine de Mohammedia est certifiee selon les referentiels de qualite automobile IATF 16949 ainsi que les normes de management de la qualite ISO 9001 et ISO 14001. A ce titre, chaque etape de la chaine logistique doit garantir une traçabilite exemplaire et un respect imperatif des delais de livraison.
</p>

<hr />

<h2>2. PROBLEMATIQUE METIER ET OBJECTIFS DU PROJET</h2>

<p>
Au sein du site industriel de Clayens NP Morocco, les activites de la supply chain s articulent autour de cinq services operationnels complementaires : les Achats, la Logistique, le Magasin Reception, le Magasin Expedition et le Transit en charge des operations douanieres.
</p>

<p>
L audit des processus existants a mis en lumiere plusieurs contraintes et sources d inefficacite operationnelle :
</p>

<ol>
  <li><strong>Fragmentation des flux de communication :</strong> Le partage d informations sur les bons de livraison et les factures s effectuait principalement a travers des appels telephoniques incessants, des courriels repetitifs et des fichiers Excel disperses sur differents postes de travail.</li>
  <li><strong>Manque de visibilite en temps reel :</strong> Les equipes logistiques et les planificateurs ne disposaient d aucune vision consolidee sur l avancement des dossiers entre le moment de la commande fournisseur et la reception physique a quai.</li>
  <li><strong>Risques critiques de surestaries douanieres :</strong> Lorsqu une cargaison etait declaree bloquee en douane au port de Casablanca ou a l aeroport, l information parvenait souvent tardivement aux equipes d achat et de production. Ce retard engendrait d une part des couts financiers eleves de penalites douanieres et d immobilisation de conteneurs, et d autre part un risque imminent d arret de ligne d assemblage automobile.</li>
  <li><strong>Traitement manuel et fastidieux des commandes EDI :</strong> Les donnees de commandes transmises par echange de donnees informatise (EDI) arrivaient sous forme de fichiers volumineux regroupant indistinctement les references destinees a differents sites de fabrication. Le tri manuel ligne par ligne prenait plusieurs heures par semaine et constituait une source majeure d erreurs de saisie.</li>
</ol>

<p>
Le present projet apporte une reponse industrielle globale a travers deux solutions distinctes et parfaitement integrees :
</p>

<ul>
  <li>Une plateforme logicielle centralisee full web assurant le pilotage, la traçabilite et le controle d acces par profil metier pour l ensemble des bons de livraison Achat et Vente.</li>
  <li>Un script d automatisation PowerShell specialise dans le tri, la separation et l archivage automatique des commandes EDI selon les sites de production concernes.</li>
</ul>

<hr />

<h2>3. JUSTIFICATION DETAILLEE DES TECHNOLOGIES ET CHOIX ARCHITECTURAUX</h2>

<p>
Le choix de la stack technologique repond a des criteres d entreprise stricts : robustesse, maintenabilite a long terme, securite des donnees et performance de navigation sur les postes d atelier.
</p>

<h3>Couche Backend : Spring Boot 3 et Java 17</h3>
<ul>
  <li><strong>Stabilite et maturite en environnement de production :</strong> Spring Boot 3 fournit une architecture solide, normalisee et largement adoptee dans les grandes industries pour supporter des traitements transactionnels critiques.</li>
  <li><strong>Java 17 LTS :</strong> Version a support etendu offrant des gains significatifs de performances d execution, une gestion optimisee de la memoire avec les nouveaux ramasse miettes et une syntaxe moderne renforçant la fiabilite du code.</li>
  <li><strong>Spring Data JPA et Hibernate :</strong> Simplification de la couche de persistance sans ecriture manuelle de requetes SQL repetitives, elimination complete des failles par injection SQL grace au requetage parametre et gestion declarative des transactions via l annotation <code>@Transactional</code>.</li>
  <li><strong>Spring Security 6 et Architecture Stateless JWT :</strong> Elimination du stockage de session en memoire sur le serveur grace aux jetons web JSON signes cryptographiquement. Le protocole implemente une rotation avec un jeton d acces a courte duree (15 minutes) et un jeton de rafraichissement persistant en base (7 jours), garantissant a la fois haute securite et confort utilisateur.</li>
  <li><strong>Gestionnaire global des exceptions :</strong> La classe annotee <code>@RestControllerAdvice</code> intercepte toutes les anomalies de validation et d authentification pour renvoyer des reponses JSON homogenes de type <code>ApiResponse&lt;T&gt;</code>, facilitant l integration avec le client React.</li>
</ul>

<h3>Couche Frontend : React 18 et Vite</h3>
<ul>
  <li><strong>React 18 et paradigme declaratif par composants :</strong> Separation claire des responsabilites d interface (Navbar, formulaires, tables, modales de traçabilite, tableaux de bord). La reactivite du DOM virtuel garantit des mises a jour instantanees sans rechargement de page.</li>
  <li><strong>Outil de build Vite :</strong> Temps de demarrage quasi instantane grace au serveur de developpement utilisant les modules ES natifs, couplé a un build de production ultra optimise via Rollup produisant des paquets Javascript et CSS compacts.</li>
  <li><strong>Axios et Intercepteurs intelligents :</strong> Injection automatique de l en tete <code>Authorization: Bearer</code> sur toutes les requetes sortantes, couplée a un intercepteur de reponse qui detecte automatiquement les codes HTTP 401 pour renouveler le jeton de maniere transparente en arriere plan sans deconnecter l utilisateur.</li>
  <li><strong>Charte graphique Clayens :</strong> Interface developpee en coherence avec l identite visuelle corporate du groupe Clayens (accent orange <code>rgb(240, 128, 19)</code>, gris ardoise industriel <code>#1e293b</code>, contrastes clairs adaptes a l eclairage des bureaux d ordonnancement et postes d atelier).</li>
</ul>

<h3>Couche Donnees : MySQL 8.0</h3>
<ul>
  <li><strong>Moteur relationnel InnoDB :</strong> Support complet des transactions ACID assurant la coherence absolue des donnees d expedition et de reception.</li>
  <li><strong>Contraintes d integrite referentielle :</strong> Liaisons strictes par cle etrangere entre les utilisateurs createurs et les enregistrements de bons de livraison, interdisant toute donnee orpheline.</li>
  <li><strong>Indexation optimisee :</strong> Index d unicite sur le numero de bon de livraison (colonne <code>num_bl</code>) garantissant des recherches immediates meme sur des volumes de plusieurs dizaines de milliers d enregistrements.</li>
</ul>

<h3>Automatisation EDI : Script PowerShell</h3>
<ul>
  <li><strong>Disponibilite native Windows :</strong> Fonctionnement immediat sur l infrastructure informatique du groupe sans necessiter l installation d interpreteurs tiers comme Python sur les postes de travail d exploitation.</li>
  <li><strong>Interoperabilite Excel et CSV :</strong> Capacite a manipuler directement les flux textuels et les classeurs de reference pour classifier les lignes de commandes en quelques secondes.</li>
</ul>

<hr />

<h2>4. MATRICE DE DROITS D ACCES ET SEPARATION DES RESPONSABILITES (RBAC)</h2>

<p>
Conformement aux exigences de la norme automobile IATF 16949, chaque service ne doit intervenir que sur les champs relevant de son perimetre metier. L application applique cette politique de securite a deux niveaux : dans l interface React (champs grises et desactives) et de façon imperative au niveau du service backend Spring Boot :
</p>

<table border="1" cellpadding="8" cellspacing="0" width="100%">
  <thead>
    <tr style="background: rgb(241, 245, 249);">
      <th align="left">Profil / Role</th>
      <th align="left">Mission industrielle dans l usine</th>
      <th align="left">Habilitations en saisie et mise a jour</th>
      <th align="left">Champs strictement verrouilles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>achat</strong></td>
      <td>Service Achats et Approvisionnements : commande des matieres premieres polymeres, colorants et composants d assemblage aupres des fournisseurs internationaux.</td>
      <td>Numero BL ou Facture, Fournisseur, Date de reception prevue, Type d operation Achat, Commentaires logistiques.</td>
      <td>Etat douane et statut de livraison (reserves au transit et magasins).</td>
    </tr>
    <tr>
      <td><strong>logistic</strong></td>
      <td>Coordination Logistique et Flux : planification globale des flux physiques, gestion des transporteurs et des disponibilites.</td>
      <td>Numero BL ou Facture, Tiers, Type d operation (Achat ou Vente), Commentaires.</td>
      <td>Validation douaniere et pointage physique en magasin.</td>
    </tr>
    <tr>
      <td><strong>magrec</strong></td>
      <td>Magasin Reception : reception a quai, dechargement des camions, controle quantitatif et qualitatif, entree des lots en stock magasin.</td>
      <td>Date effective de reception a quai, validation de la reception physique (statut OUI), commentaires de reserve.</td>
      <td>Champs douaniers et financiers.</td>
    </tr>
    <tr>
      <td><strong>magexp</strong></td>
      <td>Magasin Expedition : conditionnement des pieces injectees, palettisation, chargement des vehicules et expedition aux clients finaux.</td>
      <td>Date d enlevement prevue, confirmation de depart de livraison (statut OUI), commentaires transport.</td>
      <td>Champs douaniers et achats fournisseurs.</td>
    </tr>
    <tr>
      <td><strong>transit</strong></td>
      <td>Service Transit et Douane : etablissement des declarations douanieres en douane marocaine, gestion des certificats d origine et suivi de dedouanement.</td>
      <td>Etat douanier (EN COURS, BLOQUE, DEADOUANE), attestation de mise a disposition pour livraison, commentaires douane.</td>
      <td>Creation de commandes ou modification des dates de planification de production.</td>
    </tr>
    <tr>
      <td><strong>admin</strong></td>
      <td>Direction Industrielle et Service Informatique : supervision transversale des flux, audit du systeme et gestion des comptes utilisateurs.</td>
      <td>Acces complet en lecture et ecriture sur l ensemble des tables, consultation de la liste des collaborateurs et gestion des habilitations.</td>
      <td>Aucune restriction.</td>
    </tr>
  </tbody>
</table>

<hr />

<h2>5. FONCTIONNALITES ET MODULES DE L APPLICATION</h2>

<p>
L application propose une suite complete de fonctionnalites conçues pour accelerer le travail des operateurs et des responsables logistiques :
</p>

<h3>Formulaire centralise a quatre actions normalisees</h3>
<p>
Le formulaire principal regroupe les quatre boutons d action definis dans le cahier des charges officiel :
</p>
<ol>
  <li><strong>Enregistrer :</strong> Enregistre un nouveau bon de livraison ou facture dans la base centrale. Valide l unicite du numero de document et applique les regles de securite selon le profil.</li>
  <li><strong>Rechercher :</strong> Interroge la base centrale par numero de document exact et charge instantanement toutes les informations correspondantes dans les champs autorises.</li>
  <li><strong>Modifier :</strong> Met a jour l enregistrement charge en n appliquant que les modifications autorisees pour le role de l utilisateur connecte.</li>
  <li><strong>Consulter les BLs :</strong> Redirige immediatement l operateur vers la page de synthese regroupant les tableaux Achat et Vente.</li>
</ol>

<h3>Fiche de Traçabilite et Suivi de Cargaison (Industrial Tracking Pipeline)</h3>
<p>
En cliquant sur n importe quelle ligne d un tableau ou sur le bouton dedie du formulaire, une fenetre modale grand format presente la fiche d identite complete du bon de livraison :
</p>
<ul>
  <li><strong>Pipeline visuel en 4 jalons :</strong> Creation initiale du dossier, Planification de la date d enlevement ou reception, Passage douanier, et Cloture par Reception ou Livraison effective.</li>
  <li><strong>Transitions rapides en un clic :</strong> Le responsable transit peut basculer le dossier a l etat <code>DEADOUANE</code> ou <code>BLOQUE</code> directement depuis la fiche sans ressaisie. De meme, le magasinier confirme l entree physique en un clic.</li>
  <li><strong>Mode impression normalise :</strong> Une feuille de route epuree et lisible peut etre imprimee a tout moment via la feuille de style <code>@media print</code> pour accompagner les colis en atelier.</li>
</ul>

<h3>Indicateurs KPI et Systeme d Alerte Douane Prioritaire</h3>
<p>
Le tableau de bord et la page de consultation calculent dynamiquement des indicateurs cles :
</p>
<ul>
  <li>Nombre total de dossiers geres (Achats et Ventes).</li>
  <li>Taux de dedouanement mesure en pourcentage.</li>
  <li>Taux de realisation des livraisons et receptions physiques.</li>
  <li><strong>Bandeau d alerte prioritaire :</strong> Des qu une marchandise est passee a l etat BLOQUE en douane, un bandeau d alerte s affiche immediatement au sommet de la page avec un bouton permettant de filtrer exclusivement les dossiers immobilises. Ce dispositif permet au service transit d intervenir sans delai pour eviter des frais de surestaries.</li>
</ul>

<h3>Double Table Synchronisee (Achats et Ventes)</h3>
<p>
La page des bons de livraison structure l information en deux espaces distincts conformement aux pratiques d atelier :
</p>
<ul>
  <li><strong>Tableau Achat :</strong> Oriente approvisionnement (Fournisseur, Numero de facture, Date prevue de reception, Etat douane, Statut de reception, Commentaire).</li>
  <li><strong>Tableau Vente :</strong> Oriente expedition clients (Client, Numero de bon de livraison, Date d enlevement, Etat douane, Statut de livraison, Commentaire).</li>
  <li>Chaque tableau integre deux champs de recherche instantanee independants pour filtrer par nom de tiers ou par numero de document.</li>
</ul>

<h3>Annuaire et Administration des Utilisateurs</h3>
<p>
Module reserve au profil administrateur permettant d afficher l ensemble des collaborateurs inscrits, de visualiser leur departement d affectation, de creer de nouveaux comptes et de revoquer des acces avec confirmation visuelle.
</p>

<hr />

<h2>6. FONCTIONNEMENT DU SCRIPT POWERSHELL POUR COMMANDES EDI</h2>

<p>
Le script situe dans <code>scripts/Separation_Commandes_EDI.ps1</code> resout le probleme de traitement des commandes EDI en automatisant le flux suivant :
</p>

<ol>
  <li><strong>Lecture du fichier source :</strong> Le script prend en entree le fichier CSV brut exporte depuis la plateforme EDI client.</li>
  <li><strong>Extraction et analyse reference :</strong> Pour chaque ligne de commande, le script extrait le code article present a la colonne numero 9.</li>
  <li><strong>Recherche croisee multi bases :</strong> Le script compare la reference avec les bases de donnees Excel du site 1 (bdd1.xlsx) et du site 2 (bdd2.xlsx).</li>
  <li><strong>Repartition automatique :</strong>
    <ul>
      <li>Si la reference appartient a la base 1, la ligne est inscrite dans le fichier des commandes du site A.</li>
      <li>Si la reference appartient a la base 2, la ligne est inscrite dans le fichier des commandes du site B.</li>
      <li>Si la reference n est pas repertoriee, elle est versee dans le fichier des rejets pour traitement manuel par le responsable ordonnancement.</li>
    </ul>
  </li>
  <li><strong>Archivage d audit :</strong> Le fichier original est deplace dans le dossier d archive avec horodatage pour satisfaire aux exigences de traçabilite IATF.</li>
</ol>

<hr />

<h2>7. GUIDE DE DEPLOIEMENT ET DEMARRAGE DU SYSTEME</h2>

<h3>Prerequis techniques</h3>
<ul>
  <li>Kit de developpement Java (JDK) version 17 ou superieure installe et configure dans la variable d environnement JAVA_HOME.</li>
  <li>Environnement Node.js version 18 ou superieure avec le gestionnaire de paquets npm.</li>
  <li>Serveur de base de donnees MySQL 8.0 actif et a l ecoute sur le port 3306.</li>
</ul>

<h3>Configuration de la base de donnees</h3>
<p>
Le backend Spring Boot cree automatiquement la base de donnees <code>stage_projet_v2</code> lors du premier lancement si elle n existe pas deja, grace aux parametres configures dans <code>backend/src/main/resources/application.properties</code> :
</p>

<pre><code>spring.datasource.url=jdbc:mysql://localhost:3306/stage_projet_v2?createDatabaseIfNotExist=true&amp;useSSL=false&amp;serverTimezone=UTC&amp;allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password=clayensnpProject
spring.jpa.hibernate.ddl-auto=update
</code></pre>

<h3>Demarrage rapide en un clic sous Windows</h3>
<p>
Un fichier de commande denomme <code>start.bat</code> est place a la racine du projet. En double cliquant sur ce fichier :
</p>
<ol>
  <li>Le serveur backend Spring Boot est lance sur le port 8080.</li>
  <li>Une tempo d initialisation verifie la disponibilite du service.</li>
  <li>Le serveur frontend React Vite est execute sur le port 5173.</li>
  <li>Le navigateur web par defaut s ouvre automatiquement a l adresse <code>http://localhost:5173</code>.</li>
</ol>

<h3>Comptes de test preconfigures par departement</h3>
<p>
Au premier demarrage, la classe <code>DataInitializer</code> alimente la table des utilisateurs avec un compte de demonstration pour chaque metier :
</p>

<table border="1" cellpadding="6" cellspacing="0" width="100%">
  <thead>
    <tr style="background: rgb(241, 245, 249);">
      <th align="left">Service</th>
      <th align="left">Identifiant</th>
      <th align="left">Mot de passe</th>
      <th align="left">Role associe</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Direction et SI</td>
      <td>admin</td>
      <td>admin123</td>
      <td>admin</td>
    </tr>
    <tr>
      <td>Achats et Approvisionnements</td>
      <td>achat</td>
      <td>achat123</td>
      <td>achat</td>
    </tr>
    <tr>
      <td>Magasin Reception</td>
      <td>magrec</td>
      <td>magrec123</td>
      <td>magrec</td>
    </tr>
    <tr>
      <td>Logistique et Ordonnancement</td>
      <td>logistic</td>
      <td>logistic123</td>
      <td>logistic</td>
    </tr>
    <tr>
      <td>Magasin Expedition</td>
      <td>magexp</td>
      <td>magexp123</td>
      <td>magexp</td>
    </tr>
    <tr>
      <td>Transit et Douane</td>
      <td>transit</td>
      <td>transit123</td>
      <td>transit</td>
    </tr>
  </tbody>
</table>

<hr />

<h2>8. STRUCTURE COMPLETE DU PROJET</h2>

<pre><code>stageproject.np/
|
+-- backend/
|   +-- pom.xml
|   +-- src/
|       +-- main/
|           +-- java/com/stageprojet/
|           |   +-- StageProjetApplication.java
|           |   +-- DataInitializer.java
|           |   +-- controller/
|           |   |   +-- AuthController.java
|           |   |   +-- BonLivraisonController.java
|           |   |   +-- UserController.java
|           |   +-- service/
|           |   |   +-- AuthService.java
|           |   |   +-- BonLivraisonService.java
|           |   |   +-- RefreshTokenService.java
|           |   |   +-- UserService.java
|           |   +-- repository/
|           |   |   +-- BonLivraisonRepository.java
|           |   |   +-- RefreshTokenRepository.java
|           |   |   +-- UserRepository.java
|           |   +-- entity/
|           |   |   +-- BonLivraison.java
|           |   |   +-- DouaneStatus.java
|           |   |   +-- LivraisonStatus.java
|           |   |   +-- OperationType.java
|           |   |   +-- RefreshToken.java
|           |   |   +-- Role.java
|           |   |   +-- User.java
|           |   +-- security/
|           |   |   +-- JwtAuthFilter.java
|           |   |   +-- JwtUtils.java
|           |   |   +-- SecurityConfig.java
|           |   |   +-- UserDetailsServiceImpl.java
|           |   +-- dto/
|           |   |   +-- ApiResponse.java
|           |   |   +-- BonLivraisonDto.java
|           |   |   +-- LoginRequest.java
|           |   |   +-- LoginResponse.java
|           |   |   +-- RefreshTokenRequest.java
|           |   |   +-- RegisterRequest.java
|           |   |   +-- UserDto.java
|           |   +-- exception/
|           |       +-- GlobalExceptionHandler.java
|           +-- resources/
|               +-- application.properties
|
+-- frontend/
|   +-- package.json
|   +-- vite.config.js
|   +-- index.html
|   +-- src/
|       +-- main.jsx
|       +-- App.jsx
|       +-- index.css
|       +-- api/
|       |   +-- axios.js
|       +-- context/
|       |   +-- AuthContext.jsx
|       +-- components/
|       |   +-- BLDetailModal.jsx
|       |   +-- BLDetailModal.css
|       |   +-- BLForm.jsx
|       |   +-- BLForm.css
|       |   +-- BLTable.jsx
|       |   +-- BLTable.css
|       |   +-- Footer.jsx
|       |   +-- Footer.css
|       |   +-- Navbar.jsx
|       |   +-- Navbar.css
|       |   +-- ProtectedRoute.jsx
|       +-- pages/
|           +-- BLsPage.jsx
|           +-- BLsPage.css
|           +-- Dashboard.jsx
|           +-- Login.jsx
|           +-- RoleDashboard.jsx
|           +-- RoleDashboard.css
|           +-- Signup.jsx
|           +-- UsersPage.jsx
|           +-- UsersPage.css
|
+-- scripts/
|   +-- Separation_Commandes_EDI.ps1
|
+-- README.md
+-- start.bat
</code></pre>

<hr />

<h2>9. CONCLUSION ET VALEUR AJOUTEE DU PROJET</h2>

<p>
La mise en œuvre de cette plateforme a permis a Clayens NP Morocco de moderniser en profondeur ses pratiques logistiques et de digitaliser un flux operationnel critique.
</p>

<p>
Les gains constates sont immediats et mesurables :
</p>

<ul>
  <li><strong>Suppression des echanges informels :</strong> Diminution drastique des appels telephoniques et eliminations des ressaisies manuelles redondantes.</li>
  <li><strong>Prevention active des surcouts douaniers :</strong> Traitement immediat des dossiers de douane bloques grace a l alerte visuelle et a la fiche de traçabilite interactive.</li>
  <li><strong>Gain de temps operationnel sur les commandes EDI :</strong> Separation instantanee et securisee des flux de commandes vers les sites de production respectifs.</li>
  <li><strong>Conformite rigoureuse aux audits de qualite :</strong> Respect des exigences de traçabilite et de cloisonnement des acces formulees par la norme automobile IATF 16949.</li>
</ul>
