# Symfony 7.4 LTS upgrade — completion checklist

Status: dependency constraints prepared; dependency resolution and runtime validation pending.
Do not merge or use this branch to generate a new project until all gates below pass.
The existing composer.lock still describes the previous dependency set.

## Scope

Target Symfony 7.4 LTS while retaining PHP 8.2+ and the template's existing Doctrine,
Twig, database-backed authorization, dynamic menus, and DE/EN localization.
PackPilot-specific code belongs in a separate derived repository.

## Resolve dependencies on this branch

Use PHP 8.2+ with the application's required extensions, Composer 2, network access,
and a dedicated disposable MariaDB/MySQL database. Never use another application's database.

```sh
composer update "symfony/*" --with-all-dependencies --no-scripts
composer validate --strict
composer check-platform-reqs
composer audit
php bin/console cache:clear
php bin/console assets:install public
php bin/console importmap:install
```

If resolution fails, inspect the reported package constraints and make the smallest
compatible change. Do not ignore platform requirements or security advisories.
Commit the Composer-generated composer.lock. Review any symfony.lock or recipe changes;
do not overwrite application-specific configuration with recipe defaults.

## Validate

Configure local environment overrides and a separate test DATABASE_URL.
Run migrations against disposable development and test databases before schema checks
and database-dependent tests.

```sh
php bin/console doctrine:migrations:migrate --no-interaction
php bin/console doctrine:schema:validate
php bin/console doctrine:migrations:migrate --env=test --no-interaction
php bin/console lint:yaml config
php bin/console lint:twig templates
php bin/console lint:container
php bin/phpunit
vendor/bin/phpstan analyse
php bin/console cache:clear --env=prod --no-debug
```

Test PHP 8.2 and the intended production PHP version.
Review Symfony 7.3 and 7.4 upgrade/deprecation guidance as this upgrade crosses both minors.
Investigate failures against the main-branch baseline to distinguish existing issues
from upgrade regressions.

Browser acceptance: login/logout; anonymous dashboard redirect; active/inactive users;
database roles and denied permissions (including JSON 403 responses); dynamic menus;
settings; German/English switching; and the existing admin layout/assets.

## Reproducibility gate

In a fresh checkout with no vendor directory, run composer install from the committed
lockfile and repeat the checks. Record actual commands and outcomes in the PR before
marking it ready for review. A changed composer.json alone is not a completed upgrade.

Official references:
- https://symfony.com/releases/7.4
- https://symfony.com/doc/7.4/setup/upgrade_minor.html
