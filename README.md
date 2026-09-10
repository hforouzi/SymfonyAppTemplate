# Symfony App Template

Reusable Symfony 7.4 LTS browser-admin template with database-backed users, roles, permissions, dynamic menus, settings, Twig admin layouts, localization, Messenger, HttpClient, Mailer baseline, AssetMapper, Stimulus, PHPUnit, PHPStan, and coding-standard tooling.

## Requirements

- PHP 8.2+ (Composer is pinned to resolve dependencies against PHP 8.2)
- Composer
- MySQL or MariaDB
- Symfony CLI optional
- Node.js only if you choose to build npm-managed assets

## Installation

```powershell
composer install
```

## Environment

Copy `.env.example` to your local override if needed and set:

```dotenv
APP_SECRET=change-me
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/symfony_app?serverVersion=10.8.3-MariaDB&charset=utf8mb4"
MAILER_DSN=null://null
MESSENGER_TRANSPORT_DSN=doctrine://default?auto_setup=0
```

Use a project-specific database name. Do not point this template at another application's database.

## Database

```powershell
php bin/console doctrine:database:create --if-not-exists
php bin/console doctrine:migrations:migrate
php bin/console app:security:bootstrap-super-admin --email=admin@example.com
php bin/console app:menu:seed-baseline
```

The bootstrap command creates `ROLE_SUPER_ADMIN`, `ROLE_ADMIN`, `ROLE_USER`, generic baseline permissions, and the first Super Admin user when the email does not exist. When run interactively with `--email`, it prompts for the initial password if `--password` is not supplied. Use a unique local password and rotate it before deploying a derived application.

## Assets

This template uses Symfony AssetMapper/importmap by default.

```powershell
php bin/console importmap:install
```

If npm assets are added later:

```powershell
npm install
npm run build
```

## Run Locally

```powershell
symfony server:start
```

or use any local web server pointed at `public/`.

Key browser paths:

- `/login`
- `/dashboard`
- `/user/list`
- `/roles`
- `/permissions`
- `/settings`
- `/admin/menu/`
- `/admin/menu-categories/`

## Checks

```powershell
composer validate --strict
composer check-platform-reqs
composer audit
php bin/console cache:clear
php bin/console assets:install public
php bin/console importmap:install
php bin/console lint:yaml config
php bin/console lint:twig templates
php bin/console lint:container
php bin/console doctrine:migrations:migrate
php bin/console doctrine:schema:validate
php bin/phpunit
vendor\bin\phpstan analyse
```

## Symfony 7.4 LTS migration

This template targets Symfony 7.4 LTS. The completed upgrade notes and validation record are in [the upgrade checklist](docs/upgrades/symfony-7.4.md).
