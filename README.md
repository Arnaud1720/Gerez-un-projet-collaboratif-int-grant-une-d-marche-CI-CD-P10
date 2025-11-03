#  BobApp - Application de Blagues

[![CI/CD Pipeline](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/actions)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10&metric=alert_status)](https://sonarcloud.io/project/overview?id=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10&metric=coverage)](https://sonarcloud.io/project/overview?id=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10)
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10&metric=bugs)](https://sonarcloud.io/project/overview?id=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10)

> Application web permettant de découvrir des blagues aléatoires avec une pipeline CI/CD complète et automatisée.

---

## 📋 Table des matières

- [À propos](#-à-propos)
- [Architecture CI/CD](#-architecture-cicd)
- [KPIs et Quality Gates](#-kpis-et-quality-gates)
- [Stack technique](#-stack-technique)
- [Développement local](#-développement-local)
- [Contribuer](#-contribuer)
- [Déploiement Docker](#-déploiement-docker)
- [Métriques](#-métriques)

---

##  À propos

BobApp est une application web open source composée de :
- **Backend** : API REST en Java 11 / Spring Boot 2.6.1
- **Frontend** : Application Angular avec Node 16
- **CI/CD** : Pipeline GitHub Actions complète (5-6 min)
- **Qualité** : Analyse automatique avec SonarCloud
- **Déploiement** : Conteneurs Docker sur Docker Hub

###  Pipeline automatisée

Chaque push et pull request déclenche automatiquement :
1. ✅ Tests backend et frontend
2. ✅ Analyse qualité SonarCloud
3. ✅ Build de l'application
4. ✅ Création et déploiement des images Docker (sur `main` uniquement)

**Temps total** : ~5-6 minutes

### Étape 1 : Test Backend (27s)

**Objectif** : Valider le code Java et générer le rapport de couverture

**Quand** : À chaque push et pull request

**Actions** :
- Setup Java 11
- Cache Maven pour optimisation
- Exécution des tests : `mvn clean test`
- Génération rapport JaCoCo
- Upload artifact `backend-coverage`

**Succès si** : Tous les tests JUnit passent ✅

---

### Étape 2 : Test Frontend (44s)

**Objectif** : Valider le code Angular et générer le rapport de couverture

**Quand** : À chaque push et pull request

**Actions** :
- Setup Node.js 16
- Cache npm pour optimisation
- Installation dépendances : `npm ci`
- Exécution des tests : `npm run test -- --watch=false --code-coverage`
- Upload artifact `frontend-coverage`

**Succès si** : Tous les tests Jasmine/Karma passent ✅

---

### Étape 3 : SonarCloud Analysis (46s)

**Objectif** : Analyser la qualité du code et vérifier le Quality Gate

**Quand** : Après succès des tests backend ET frontend

**Actions** :
- Récupération des rapports de couverture
- Analyse statique du code (Java + TypeScript)
- Calcul des métriques : bugs, vulnérabilités, code smells, coverage
- Vérification du Quality Gate
- Publication sur le dashboard SonarCloud

**Succès si** : Quality Gate PASSED ✅

🔗 **Voir l'analyse** : [Dashboard SonarCloud](https://sonarcloud.io/project/overview?id=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10)

---

### Étape 4 : Build Application (1m 30s)

**Objectif** : Compiler le backend et builder le frontend pour la production

**Quand** : Après succès de l'analyse SonarCloud

**Actions** :
- **Backend** : `mvn clean package -DskipTests` → JAR exécutable (17 MB)
- **Frontend** : `npm run build --prod` → Fichiers optimisés (99 KB)
- Upload des artifacts

**Succès si** : Build sans erreur ✅

**Artifacts produits** :
- `backend-jar` : Fichier JAR Spring Boot
- `frontend-dist` : Fichiers de production Angular

---

### Étape 5 : Docker Build & Push (2m 22s)

**Objectif** : Créer et déployer les images Docker sur Docker Hub

**Quand** : ⚠️ **Uniquement sur la branche `main`** après un push

**Actions** :
- Connexion à Docker Hub
- Build des images backend + frontend
- Tag avec `latest` et `<git-sha>` pour traçabilité
- Push vers Docker Hub

**Succès si** : Images publiées avec succès ✅

**Images disponibles** :
- `username/bobapp-backend:latest`
- `username/bobapp-frontend:latest`

> 💡 **Note** : Cette étape ne s'exécute pas sur les PR pour éviter les déploiements non désirés.

---

## 📊 KPIs et Quality Gates

### Seuils configurés

Pour valider une pull request, le code doit respecter les critères suivants :

| KPI | Seuil | Actuel | Bloquant ? |
|-----|-------|--------|------------|
| **Coverage (nouveau code)** | ≥ 80% | N/A* | ✅ Oui |
| **Bugs** | 0 | 0 | ✅ Oui |
| **Vulnerabilities** | 0 | 0 | ✅ Oui |
| **Reliability Rating** | A | B | ⚠️ Recommandé |
| **Duplications** | ≤ 3% | 0.0% | ❌ Non |

\* *N/A : Aucun nouveau code depuis la configuration. Le seuil s'appliquera au prochain commit.*

---

### KPI #1 : Coverage ≥ 80% (OBLIGATOIRE)

**Pourquoi ?**
- Le projet actuel a une couverture de 38.8% (insuffisant)
- Tout nouveau code doit être correctement testé
- Empêche l'accumulation de dette technique

**Comment ?**
```
Si j'ajoute 10 lignes de code :
- 8+ lignes testées (80%) → ✅ Merge autorisé
- 7 lignes testées (70%)  → ❌ Merge bloqué
```

**Impact** : Réduit drastiquement les bugs en production

---

### KPI #2 : 0 Bug & 0 Vulnérabilité (CRITIQUE)

**Pourquoi ?**
- Aucun bug ne doit être déployé en production
- La sécurité des utilisateurs est prioritaire

**Détection automatique** :
- NullPointerException
- Failles de sécurité (XSS, SQL injection)
- Memory leaks
- Dépendances vulnérables

**Impact** : Application stable et sécurisée

---

### KPI #3 : Reliability Rating A (RECOMMANDÉ)

**État actuel** : Rating B (dû à un problème de `Random` non réutilisé)

**Objectif** : Passer en Rating A pour améliorer la fiabilité

**Effort** : 35 minutes de corrections

---

### Configuration Quality Gate

**Quality Gate actif** : "Sonar way" (personnalisé)

```yaml
Conditions sur le nouveau code :
  - Coverage ≥ 80%              → BLOCKING ❌
  - Bugs = 0                    → BLOCKING ❌
  - Vulnerabilities = 0         → BLOCKING ❌
  - Duplications ≤ 3%           → WARNING ⚠️
  - Maintainability Rating ≥ A  → WARNING ⚠️
```

---

## ️ Stack technique

### Backend
```
Langage      : Java 11
Framework    : Spring Boot 2.6.1
Build        : Maven 3.x
Tests        : JUnit 5
Coverage     : JaCoCo
Architecture : API REST (Controller/Service/Repository)
```

### Frontend
```
Framework    : Angular 13+
Langage      : TypeScript
Build        : Angular CLI / npm
Tests        : Jasmine + Karma
Coverage     : Istanbul
```

### CI/CD & Qualité
```
CI/CD        : GitHub Actions
Qualité      : SonarCloud
Containers   : Docker + Docker Hub
Quality Gate : Sonar way
```

---

##  Développement local

### Prérequis

- **Java 11** (JDK)
- **Node.js 16+** et npm
- **Maven 3.x**
- **Git**
- **Docker** (optionnel)

### Installation

#### 1. Cloner le repository

```bash
git clone https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10.git
cd Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10
```

#### 2. Backend (Spring Boot)

```bash
cd back

# Installer les dépendances et lancer les tests
mvn clean test

# Générer le rapport de couverture
mvn test jacoco:report
# Rapport disponible dans : target/site/jacoco/index.html

# Builder l'application
mvn clean package

# Lancer l'application
mvn spring-boot:run
# API disponible sur http://localhost:8080
```

#### 3. Frontend (Angular)

```bash
cd front

# Installer les dépendances
npm install

# Lancer les tests
npm test

# Générer le rapport de couverture
npm run test -- --code-coverage
# Rapport disponible dans : coverage/bobapp/index.html

# Lancer en mode développement
npm start
# Application disponible sur http://localhost:4200
```

---

## 🤝 Contribuer

### Workflow de contribution

#### 1. Forker et cloner

```bash
# Fork le projet sur GitHub, puis :
git clone https://github.com/VOTRE-USERNAME/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10.git
cd Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10
```

#### 2. Créer une branche

```bash
git checkout -b feature/ma-nouvelle-fonctionnalite
```

**Convention de nommage** :
- `feature/` : Nouvelle fonctionnalité
- `fix/` : Correction de bug
- `refactor/` : Refactoring
- `test/` : Ajout de tests
- `docs/` : Documentation

#### 3. Développer et tester

**Backend** :
```bash
cd back
mvn clean test        # Tests doivent passer ✅
mvn test jacoco:report # Vérifier la couverture
```

**Frontend** :
```bash
cd front
npm test              # Tests doivent passer ✅
npm run test -- --code-coverage # Vérifier la couverture
```

#### 4. Commiter

```bash
git add .
git commit -m "feat: description de la fonctionnalité"
```

**Convention de commit** :
- `feat:` Nouvelle fonctionnalité
- `fix:` Correction de bug
- `test:` Ajout de tests
- `refactor:` Refactoring
- `docs:` Documentation
- `style:` Formatage
- `chore:` Tâches de maintenance

#### 5. Pusher et créer une PR

```bash
git push origin feature/ma-nouvelle-fonctionnalite
```

Puis sur GitHub :
1. Cliquer sur "New Pull Request"
2. Remplir la description
3. Attendre la validation automatique de la CI/CD

---

### ✅ Validation automatique

Votre Pull Request sera **automatiquement testée** :

```
✅ Tests Backend (27s)
   → Tous les tests JUnit doivent passer

✅ Tests Frontend (44s)
   → Tous les tests Jasmine/Karma doivent passer

✅ SonarCloud Analysis (46s)
   → Quality Gate doit être PASSED
   → Coverage ≥ 80% sur votre nouveau code
   → 0 bug
   → 0 vulnérabilité

✅ Build Application (1m 30s)
   → Build doit réussir sans erreur
```

**Si tous les checks passent** ✅ → Votre PR peut être mergée !  
**Si un check échoue** ❌ → Corrigez et poussez à nouveau

---

###  Bonnes pratiques

#### Tests (OBLIGATOIRE)

**Toute nouvelle fonctionnalité doit avoir des tests.**

```java
// Backend - Exemple de test JUnit
@Test
public void testGetRandomJoke() {
    Joke joke = jokeService.getRandomJoke();
    assertNotNull(joke);
    assertNotNull(joke.getJoke());
}
```

```typescript
// Frontend - Exemple de test Jasmine
it('should return a random joke', () => {
  jokesService.getRandomJoke().subscribe(joke => {
    expect(joke).toBeDefined();
    expect(joke.joke).toBeDefined();
  });
});
```

#### Coverage ≥ 80%

Vérifiez votre coverage avant de pusher :

```bash
# Backend
mvn test jacoco:report
# Ouvrir : target/site/jacoco/index.html

# Frontend
npm run test -- --code-coverage
# Ouvrir : coverage/bobapp/index.html
```

#### Code propre

- ✅ Respectez les conventions Java / TypeScript
- ✅ Commentez le code complexe
- ✅ Évitez les duplications
- ✅ Gérez les erreurs proprement

#### Commits atomiques

- ✅ Un commit = une fonctionnalité / un fix
- ✅ Messages clairs et descriptifs
- ✅ Référencez les issues si applicable

---

##  Déploiement Docker

### Lancer avec Docker

#### Option 1 : Depuis Docker Hub (recommandé)

```bash
# Backend
docker pull username/bobapp-backend:latest
docker run -p 8080:8080 username/bobapp-backend:latest

# Frontend
docker pull username/bobapp-frontend:latest
docker run -p 80:80 username/bobapp-frontend:latest
```

#### Option 2 : Build local

```bash
# Backend
cd back
docker build -t bobapp-backend .
docker run -p 8080:8080 bobapp-backend

# Frontend
cd front
docker build -t bobapp-frontend .
docker run -p 80:80 bobapp-frontend
```

### Accéder à l'application

- **Frontend** : http://localhost
- **Backend API** : http://localhost:8080
- **Endpoint test** : http://localhost:8080/api/random

---

##  Métriques

### Rapports de couverture

Les rapports sont générés automatiquement à chaque build et disponibles dans les artifacts GitHub Actions :

1. Aller dans l'onglet **[Actions](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/actions)**
2. Sélectionner un workflow run
3. Télécharger les artifacts :
    - `backend-coverage` : Rapport JaCoCo (HTML)
    - `frontend-coverage` : Rapport Karma (HTML)

### Dashboard SonarCloud

Consultez les métriques en temps réel :

 **[Dashboard complet](https://sonarcloud.io/project/overview?id=Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10)**

**Métriques disponibles** :
- Coverage global et par fichier
- Bugs et vulnérabilités avec localisation
- Code smells et dette technique
- Historique des analyses
- Tendances qualité

### Métriques actuelles

| Métrique | Valeur | Objectif | Statut |
|----------|---------|----------|--------|
| Coverage | 38.8% | 80% | ⚠️ En amélioration |
| Bugs | 0 | 0 | ✅ Excellent |
| Vulnerabilities | 0 | 0 | ✅ Sécurisé |
| Code Smells | 9 | < 10 | ⚠️ Acceptable |
| Duplications | 0.0% | < 3% | ✅ Parfait |
| Maintainability | A | A/B | ✅ Excellent |
| Reliability | B | A | ⚠️ À améliorer |
| Security | A | A | ✅ Sécurisé |

---

## 📚 Documentation

-  **[Analyse CI/CD complète](./docs/Analyse_CICD_BobApp.md)** : Document détaillé sur la pipeline et la qualité
-  **[Issues GitHub](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/issues)** : Bugs et améliorations
-  **[Pull Requests](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/pulls)** : Contributions en cours

---

##  Support

### Problème avec la CI/CD ?

1. Vérifiez les logs dans l'onglet [Actions](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/actions)
2. Consultez les commentaires SonarCloud sur votre PR
3. Vérifiez que votre coverage ≥ 80%
4. Si le problème persiste, ouvrez une [issue](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/issues)

### Questions sur le projet ?

-  Ouvrir une [issue GitHub](https://github.com/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/issues)
- Consulter la [documentation complète](./docs/Analyse_CICD_BobApp.md)

---

## 📜 Licence

Projet open source sous licence à définir.

---

## 👥 Contributeurs

- **Bob** - Créateur original de BobApp
- **Arnaud DERISBOURG** - CI/CD et qualité du code

---

##  Roadmap

### Court terme (1 mois)
- [ ] Corriger les 3 issues High (30 min)
- [ ] Atteindre 50% de coverage global
- [ ] Passer en Reliability Rating A

### Moyen terme (3 mois)
- [ ] Atteindre 70% de coverage global
- [ ] Tests E2E complets
- [ ] Documentation technique complète

### Long terme (6 mois)
- [ ] Atteindre 80% de coverage global
- [ ] Monitoring et alerting
- [ ] Déploiement multi-environnements

---

##  Statistiques

![GitHub Actions](https://img.shields.io/github/actions/workflow/status/Arnaud1720/Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10/ci-cd.yml?label=CI%2FCD)
![SonarCloud Coverage](https://img.shields.io/sonar/coverage/Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10?server=https%3A%2F%2Fsonarcloud.io)
![SonarCloud Quality Gate](https://img.shields.io/sonar/quality_gate/Arnaud1720_Gerez-un-projet-collaboratif-int-grant-une-d-marche-CI-CD-P10?server=https%3A%2F%2Fsonarcloud.io)

---

**Dernière mise à jour** : Octobre 2025  
**Version** : 1.0

---

⭐ **Si ce projet vous plaît, n'hésitez pas à lui donner une étoile !** ⭐