# Symfony 7.4 LTS upgrade validation

Status: dependency resolution and essential validation completed on the `chore/symfony-7.4-lts` branch.

## Scope

The reusable template now targets Symfony 7.4 LTS while retaining PHP 8.2 compatibility, Doctrine ORM/DBAL with MySQL/MariaDB, Twig/Vristo admin templates, database-backed users, roles, permissions, dynamic menus, authentication, and German/English localization.

PackPilot-specific code remains out of scope for this branch.

## Dependency resolution

Composer resolution was performed with Symfony constrained to `7.4.*`:

```sh
composer update "symfony/*" --with-all-dependencies --no-scripts
composer update "symfony/*" doctrine/instantiator --with-all-dependencies --no-scripts
```

The second command was required after adding a Composer platform pin for PHP 8.2. Without the platform pin, resolving on PHP 8.4 selected `doctrine/instantiator` 2.1.0, which requires PHP 8.4 and would break PHP 8.2 installs. The committed lockfile resolves `doctrine/instantiator` 2.0.0 and remains compatible with PHP 8.2.

Composer config now includes:

```json
"platform": {
  "php": "8.2.0"
}
```

Symfony Flex added the Symfony 7.3+ `symfony/property-info` recipe file at `config/packages/property_info.yaml`.

## Application changes

- Regenerated `composer.lock` for Symfony 7.4 LTS.
- Added `config/packages/property_info.yaml` from the Symfony Flex recipe.
- Added a Doctrine migration to update Messenger Doctrine transport indexes to the Symfony 7.4 schema.
- Set `framework.profiler.collect_serializer_data` explicitly in test config to avoid the Symfony 7.3+ profiler deprecation.
- Removed the obsolete DoctrineBundle `report_fields_where_declared` setting.
- Updated MariaDB example URLs to use the current `10.8.3-MariaDB` server version format.

## Validation results

Validated on PHP 8.4.20 unless noted:

```sh
composer validate --strict
composer check-platform-reqs
composer audit
php bin/console cache:clear
php bin/console assets:install public
php bin/console importmap:install
php bin/console lint:yaml config
php bin/console lint:twig templates
php bin/console lint:container
php bin/phpunit
php bin/console cache:clear --env=prod --no-debug
```

All commands above passed after the compatibility fixes.

Database validation used disposable local databases:

```sh
symfony_app_template_74_dev
symfony_app_template_74_test_test
```

The following passed against those disposable databases:

```sh
php bin/console doctrine:migrations:migrate --no-interaction
php bin/console doctrine:schema:validate
php bin/console doctrine:migrations:migrate --env=test --no-interaction
php bin/console doctrine:schema:validate --env=test
```

PHP 8.2 compatibility was checked with PHP 8.2.8 after pinning Composer's platform to PHP 8.2 and regenerating the lockfile. On PHP 8.2.8, `vendor/composer/platform_check.php`, `php bin/console about`, and `php bin/phpunit` passed.

## Known limitations

`vendor/bin/phpstan analyse` still reports 105 findings. The same 105 findings reproduce on `origin/main`, so they are pre-existing static-analysis debt rather than Symfony 7.4 upgrade regressions.

`composer run quality` still fails before PHPStan/tests because `php-cs-fixer` is not installed or available on PATH. This is also unrelated to the Symfony dependency upgrade.

Production cache clear on PHP 8.4 can still log Doctrine lazy-object deprecations because DoctrineBundle requires native lazy objects on PHP 8.4+, while enabling `enable_native_lazy_objects` unconditionally would break PHP 8.2. The template keeps the PHP 8.2-safe default.

Browser flows were not manually exercised in a real browser in this environment. Covered automated smoke checks include login page accessibility, anonymous dashboard redirect, and the dashboard route.

## Official references

- https://symfony.com/releases/7.4
- https://symfony.com/doc/7.4/setup/upgrade_minor.html
