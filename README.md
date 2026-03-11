# Redirects Checker

> **EN** -- Detect redirect chains, loops, self-redirects and HTTP errors (404, 5xx) in a bulk list of 301 redirections, with automatic chain flattening and CSV export.

---

## Description

**Redirects Checker** est un outil d'audit SEO qui analyse une liste de redirections 301 pour detecter automatiquement les problemes courants : chaines de redirections (A -> B -> C), boucles infinies (A -> B -> A), auto-redirections (A -> A) et erreurs HTTP (404, 5xx, timeouts). L'outil propose des corrections automatiques en aplatissant les chaines pour pointer directement vers la destination finale.

L'utilisateur soumet ses redirections via un textarea (copier-coller) ou un fichier CSV/TSV. L'outil effectue d'abord une analyse statique du graphe de redirections (detection des chaines, boucles et auto-redirections), puis lance optionnellement un worker en arriere-plan qui crawle les URLs sources en HTTP HEAD concurrent pour verifier les codes de reponse reels. La progression du crawl est affichee en temps reel via un systeme de polling AJAX avec logs en streaming.

Les resultats sont presentes dans une interface a onglets avec KPI synthetiques, tableau des problemes filtrable/triable, liste complete des redirections et export CSV. L'outil supporte l'internationalisation (francais/anglais) et s'integre a la plateforme SEO en mode embedded.

---

## Fonctionnalites

- **Detection des chaines de redirections** : identifie les chaines de 2+ sauts et propose l'aplatissement automatique vers la destination finale
- **Detection des boucles** : algorithme de coloration du graphe (blanc/gris/noir) pour reperer les cycles
- **Detection des auto-redirections** : signale les URLs qui redirigent vers elles-memes
- **Verification HTTP en arriere-plan** : crawl concurrent des URLs sources via Guzzle Pool (HEAD avec fallback GET pour les 403/405)
- **Protection SSRF** : validation des URLs avec blocage des IP privees et des ports non standards
- **Progression en temps reel** : polling AJAX avec logs JSON Lines (JSONL) affiches dans un terminal de streaming
- **Options avancees du crawler** : concurrence, delai entre requetes, timeout, max redirections suivies, User-Agent configurable (Googlebot, Bingbot, Chrome, custom), header HTTP custom
- **Sauvegarde des configurations** : persistance des presets de crawler (jusqu'a 50 par utilisateur), avec chargement et suppression
- **Historique des analyses** : conservation des jobs pendant 72h avec pagination, nettoyage automatique
- **Edition inline des corrections** : clic sur la destination corrigee pour ajuster manuellement
- **Application en masse** : bouton "Appliquer toutes les corrections" pour accepter les aplatissements proposes
- **Export CSV** : deux modes d'export (corrections avec destinations aplaties, ou liste des problemes)
- **Export JSON** : telechargement complet des donnees brutes via `download.php`
- **Filtres et recherche** : filtrage par type de probleme (chaines, boucles, 404, erreurs HTTP, auto-redirections, redirections inattendues) et recherche textuelle
- **Tri des colonnes** : tri ascendant/descendant sur les colonnes source
- **Pagination cote client** : pagination des tableaux pour les listes volumineuses
- **Internationalisation** : interface disponible en francais et anglais (systeme `data-i18n` + `translations.js`)
- **Copie en un clic** : clic sur une URL pour la copier dans le presse-papiers avec toast de confirmation
- **Detection des redirections inattendues** : signale quand la destination reelle ne correspond pas a la destination declaree

---

## Prerequis

- **PHP >= 8.3** (typed class constants, `str_starts_with`, `str_contains`)
- **Composer** pour l'installation des dependances
- Extension PHP `curl` (requise par Guzzle)
- Permissions d'ecriture sur le repertoire `data/` (creation automatique)

---

## Installation

```bash
cd /home/aymeric/projects/redirects-checker/
composer install
```

Aucune variable d'environnement requise (`env_keys` est vide dans `module.json`).

---

## Utilisation

### Developpement local (standalone)

```bash
cd /home/aymeric/projects/redirects-checker/
php -S localhost:8080
```

Ouvrir `http://localhost:8080` dans le navigateur.

### Workflow

1. **Importer les redirections** : coller la liste dans le textarea ou importer un fichier CSV/TSV (max 5 Mo). Formats acceptes : tabulation, point-virgule ou virgule comme separateur.
2. **Configurer les options** (optionnel) : choisir le separateur, activer/desactiver la verification HTTP, ajuster les parametres du crawler (concurrence, delai, timeout, User-Agent, headers custom).
3. **Lancer l'analyse** : clic sur "Analyser les redirections". Si des URLs relatives sont detectees avec la verification HTTP activee, une modale demande le domaine a prefixer.
4. **Consulter les resultats** : KPI synthetiques en haut, onglet "Problemes" avec filtres, onglet "Liste complete" pour toutes les redirections.
5. **Corriger** : editer les destinations en cliquant dessus, ou appliquer toutes les corrections automatiques en un clic.
6. **Exporter** : telecharger le CSV des corrections ou le CSV des problemes.

### Lancer les tests

```bash
composer test
# ou directement :
vendor/bin/pest
```

---

## Stack technique

| Composant | Technologie |
|-----------|-------------|
| Backend | PHP 8.3 |
| Client HTTP | Guzzle 7 (Pool concurrent, HEAD + fallback GET) |
| Frontend | Bootstrap 5.3.3, Bootstrap Icons 1.11.3, Poppins |
| JavaScript | Vanilla JS (IIFE), aucun framework |
| Tests | Pest 3 |
| Stockage | Fichiers JSON sur disque (`data/jobs/`) |
| i18n | `translations.js` avec systeme `data-i18n` |

---

## Structure du projet

```
redirects-checker/
|-- module.json              # Manifeste du plugin (slug, routes, quota, etc.)
|-- boot.php                 # Autoloader Composer + autoload PSR-0 pour src/
|-- index.php                # Page d'accueil (formulaire + historique)
|-- process.php              # Endpoint POST : parsing, analyse du graphe, lancement du worker
|-- worker.php               # Worker CLI en arriere-plan : crawl HTTP concurrent
|-- progress.php             # Endpoint GET : progression du worker + logs JSONL
|-- results.php              # Page de resultats (KPI, tableaux, exports)
|-- download.php             # Export CSV/JSON des resultats
|-- configs.php              # CRUD des configurations de crawler sauvegardees
|-- historique.php           # Endpoint GET : liste des jobs existants
|-- app.js                   # Logique JS complete (formulaire, polling, tableaux, export, i18n)
|-- translations.js          # Traductions FR/EN
|-- styles.css               # Charte graphique (variables CSS, composants)
|-- composer.json            # Dependances PHP (Guzzle, Pest)
|-- phpunit.xml              # Configuration PHPUnit/Pest
|-- .gitignore               # Exclusions (vendor, data, .env, logs)
|-- src/
|   |-- AnalyseurRedirections.php  # Parsing du texte/CSV, detection du separateur, prefixage domaine
|   |-- GrapheRedirections.php     # Analyse du graphe : chaines, boucles, auto-redirections, aplatissement
|   |-- GestionnaireJobs.php       # Gestion des jobs sur disque (CRUD, progression, nettoyage 72h)
|   |-- VerificateurHttp.php       # Crawl HTTP concurrent via Guzzle Pool (HEAD/GET, SSRF protection)
|-- tests/
|   |-- Pest.php                   # Bootstrap Pest
|   |-- Unit/
|   |   |-- AnalyseurRedirectionsTest.php
|   |   |-- GrapheRedirectionsTest.php
|   |   |-- GestionnaireJobsTest.php
|   |-- Feature/                   # (repertoire present, tests a venir)
|-- data/
|   |-- jobs/                      # Stockage des jobs (analyse.json, config.json, progress.json, resultats.json, logs.jsonl)
|   |-- configs/                   # Configurations de crawler sauvegardees par utilisateur
```

---

## Routes (module.json)

| Methode | Chemin | Type | Description |
|---------|--------|------|-------------|
| `POST` | `process.php` | `ajax` | Soumission du formulaire : parsing, analyse du graphe, creation du job, lancement du worker |
| `GET` | `results.php` | `page` | Page de resultats (rendue dans le layout plateforme) |
| `GET` | `progress.php` | `ajax` | Progression du worker + logs temps reel (polling) |
| `GET` | `download.php` | `ajax` | Export CSV ou JSON des resultats (`?type=corrige\|problemes&format=csv\|json`) |
| `GET` | `configs.php` | `ajax` | Lister les configurations sauvegardees |
| `POST` | `configs.php` | `ajax` | Sauvegarder une configuration de crawler |
| `DELETE` | `configs.php` | `ajax` | Supprimer une configuration sauvegardee |
| `GET` | `historique.php` | `ajax` | Lister tous les jobs existants |

---

## Integration plateforme

### Mode d'affichage

Le plugin fonctionne en mode **embedded** (`display_mode: "embedded"`). La plateforme extrait le contenu du `<body>`, recrit les chemins CSS/JS, supprime la navbar et injecte le CSRF automatiquement. Le plugin reste fonctionnel en standalone sans modification.

### Quota

- **Mode** : `form_submit` (incremente sur POST du formulaire)
- **Quota par defaut** : 100 analyses/mois
- Le worker verifie egalement le quota via `Platform\Module\Quota::trackerSiDisponible()` dans `process.php`
- Gestion du code 429 cote client avec message d'erreur

### Internationalisation

- Langues supportees : `fr`, `en`
- La langue est determinee par `window.PLATFORM_LANG` (plateforme) ou par le parametre `?lg=` / `localStorage` (standalone)
- Selecteur de langue affiche uniquement en mode standalone (masque quand `PLATFORM_EMBEDDED` est defini)

### Routage AJAX

Tous les appels `fetch()` utilisent `window.MODULE_BASE_URL || '.'` comme prefixe pour garantir la resolution correcte des URLs en mode embedded.

### Constantes plateforme utilisees

- `PLATFORM_EMBEDDED` : ajuste les redirections PHP (erreur inline vs `header('Location')`) et masque le selecteur de langue
- `$_SESSION['user_id']` : isole les configurations sauvegardees par utilisateur en mode plateforme

### Variables d'environnement

Aucune (`env_keys: []`). Le plugin ne necessite aucune cle API externe.
