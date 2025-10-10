# drupal-geo-starter

Projet **Drupal 11** géré par **Composer**, docroot dans `web/`. Repo de départ propre avec `.gitignore`, `.gitattributes` et `.editorconfig`.

---

## Prérequis

- **PHP 8.2+** (extensions usuelles: intl, gd, mbstring, xml, pdo_pgsql ou pdo_mysql, zip)
- **Composer 2**
- **Base de données**: PostgreSQL 14+ (ou MySQL/MariaDB)
- (Optionnel) **Node.js** si build front dans un thème custom

---

## Installation locale (existante)

Le repo contient déjà `composer.json`/`lock`.

```bash
# 1) Installer les dépendances
composer install

# 2) Créer un vhost local OU lancer le serveur PHP intégré
php -S 127.0.0.1:8080 -t web web/.ht.router.php
# ➜ site accessible sur http://127.0.0.1:8080
```

---

## Installation du site Drupal

Deux façons : **via l’UI** (navigateur) ou **via Drush**.

### A) Via l’UI

1. Démarre le serveur (voir ci‑dessus)
2. Va sur `http://127.0.0.1:8080`
3. Suis l’installateur (choisis Standard) et fournis l’URL de ta DB

### B) Via Drush

Installe d’abord Drush si absent :

```bash
composer require drush/drush --dev
```

Installe le site :

```bash
# PostgreSQL\ nvendor/bin/drush site:install standard \
  --db-url="pgsql://USER:PASS@127.0.0.1:5432/DBNAME" \
  --site-name="Drupal Geo Starter" -y

# (MySQL)
# vendor/bin/drush site:install standard \
#   --db-url="mysql://USER:PASS@127.0.0.1:3306/DBNAME" \
#   --site-name="Drupal Geo Starter" -y
```

**Important**: `web/sites/default/settings.php` est ignoré par Git. Pour des overrides locaux, utilise plutôt `settings.local.php` (voir ci‑dessous).

---

## Configuration & secrets

### Fichiers recommandés

- `web/sites/default/settings.php` ➜ généré par l’installateur, **non versionné**
- `web/sites/default/settings.local.php` ➜ overrides locaux, **non versionné**
- `web/sites/development.services.yml` ➜ inclus par défaut pour le mode dev

Exemple minimal de `settings.local.php` :

```php
<?php
$databases['default']['default'] = [
  'driver' => 'pgsql', // ou 'mysql'
  'database' => 'appdb',
  'username' => 'jerome',
  'password' => 'changeme',
  'host' => '127.0.0.1',
  'port' => '5432',
  'prefix' => '',
];

$settings['config_sync_directory'] = DRUPAL_ROOT . '/../config/sync';
$settings['hash_salt'] = 'CHANGE_ME';
```

---

## Export / Import de configuration

Active **Drush** pour gérer la config.

```bash
# Définir le dossier de sync (déjà pointé dans l’exemple plus haut)
# $settings['config_sync_directory'] = DRUPAL_ROOT . '/../config/sync';

# Exporter
vendor/bin/drush cex -y

# Importer
vendor/bin/drush cim -y
```

Versionner le dossier `config/sync/` dans Git.

---

## Commandes utiles

```bash
# Statut
composer show drupal/core-recommended | grep versions
vendor/bin/drush status

# Cache
vendor/bin/drush cr

# Comptes administrateurs
vendor/bin/drush uli          # Lien de connexion unique
vendor/bin/drush user:password admin "nouveau_mdp"

# Modules
vendor/bin/drush en devel -y  # activer
vendor/bin/drush pmu devel -y # désinstaller
```

---

## Développement local

- Serveur PHP interne : `php -S 127.0.0.1:8080 -t web web/.ht.router.php`
- Logs : `web/*-debug.log` ignorés par Git
- Thèmes custom : placer dans `web/themes/custom/` (penser à ignorer les dossiers de build `dist/`, `.cache/`)

---

## Déploiement (générique)

1. `composer install --no-dev --optimize-autoloader`
2. Synchroniser `config/sync` ➜ `drush cim -y`
3. Lancer `drush updb -y` (mises à jour DB) et `drush cr`
4. S’assurer que `sites/*/files` est présent et writable (non versionné)

---

## Structure Git

- Versionné : `composer.*`, `web` (sans `core`/`contrib`/`settings.php`), `recipes/`, `config/sync/`, fichiers de projet
- Ignoré : `vendor/`, `web/core/`, `web/*/contrib/`, `web/sites/*/files`, `web/sites/*/settings*.php`

---

## Licence

licence MIT

---

## Notes

- Projet pensé pour Windows/Linux/macOS.
- Si tu utilises PostgreSQL localement, pense à adapter `driver`, `port` et identifiants.
- En cas d’erreur d’import de config « empty import »: vérifie que `config/sync` contient bien des fichiers exportés et que le chemin est bien défini dans `settings*.php`.
