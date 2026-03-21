# AGENTS.md

## Cursor Cloud specific instructions

### Overview

baserCMS is a Japanese CMS built on PHP 8.1 + CakePHP 5. All development runs inside Docker containers (MySQL 8.0 + PHP 8.1/Apache). The source is mounted from the host into the `bc-php` container at `/var/www/html`.

### Starting the development environment

```bash
cd /workspace/docker
cp docker-compose.yml.default docker-compose.yml
cp .env.example .env
# Disable xdebug for performance in CI/dev
sed -i 's/XDEBUG_MODE: "debug"/XDEBUG_MODE: "off"/g' docker-compose.yml
docker compose up -d bc-db bc-php
```

Wait for MySQL to be ready (~10s), then install:

```bash
docker exec bc-php composer install --no-plugins
docker exec bc-php bin/cake setup install
docker exec bc-php bin/cake install https://localhost admin@example.com baserCMS1234 basercms --host bc-db --username root --password root
```

### Running tests

Tests must run inside the `bc-php` container. Always run `bin/cake setup test` first to configure `.env` for testing:

```bash
docker exec bc-php bin/cake setup test
docker exec bc-php vendor/bin/phpunit --testsuite BcFavorite --colors=always
```

To run all tests: `docker exec bc-php vendor/bin/phpunit --colors=always`

Do NOT use `-v` or `--verbose` flags with PHPUnit (they are not supported in this setup).

### Lint

```bash
docker exec bc-php vendor/bin/phpcs --colors -p src/ tests/
```

### Key gotchas

- Use `docker exec` (not `docker compose exec`) for running commands in containers.
- The `config/install.php` file is created during installation; it does not exist in the repo.
- The `composer install --no-plugins` flag is required for the first install to avoid autoload errors from post-install hooks before dependencies exist.
- npm workspaces (`plugins/bc-admin-third`, `plugins/bc-front`) are for frontend assets only; run `npm install` from the workspace root.
- The app is accessible at `https://localhost` (self-signed SSL) and the admin panel at `https://localhost/baser/admin`.
- Admin credentials for development: `admin@example.com` / `baserCMS1234`.
- For Docker-in-Docker in Cloud VMs: fuse-overlayfs storage driver and iptables-legacy are required.
