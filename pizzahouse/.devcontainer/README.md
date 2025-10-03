## Dev Container: pizzahouse

Quick reference for opening, rebuilding and troubleshooting the devcontainer used by this project.

### Overview
- Devcontainer config: `.devcontainer/devcontainer.json`
- Uses Docker Compose (`../docker-compose.yaml`) and targets the `app` service.
- `postCreateCommand`: `composer install && npm install` (runs inside the container after build)
- Forwarded ports: `8080` (Laravel app), `3306` (MySQL)
- Remote user: `www-data`

### Prerequisites
- Docker Desktop (or Docker Engine) running
- VS Code with the "Dev Containers" (aka Remote - Containers / Dev Containers) extension installed if using the GUI
- (Optional) `@devcontainers/cli` if you prefer CLI commands

### Open in VS Code (recommended)
1. Open VS Code and open the `pizzahouse` folder.
2. Open the Command Palette (Ctrl+Shift+P) and choose:
   - `Dev Containers: Reopen in Container` or
   - `Dev Containers: Open Folder in Container...` and select the workspace folder.

VS Code will build the images defined by the compose file and attach to the `app` service workspace at `/app`. The `postCreateCommand` will run automatically after the container is created.

### CLI: devcontainer (optional)
Install the devcontainer CLI if you want a scripted/declarative flow:
```bash
npm install -g @devcontainers/cli
devcontainer up --workspace-folder .
# Rebuild
devcontainer rebuild --workspace-folder .
devcontainer down --workspace-folder .
```
Note: If `devcontainer` is not installed you may see `command not found`.

### Fallback: Docker Compose
You can run the compose services directly and then run the post-create steps inside the `app` container:
```bash
cd pizzahouse
docker compose up --build -d
docker compose exec app sh -lc "composer install --no-interaction --prefer-dist && npm install --no-audit --no-fund"
# Optional: build frontend assets
docker compose exec -T app sh -lc "npm run dev"
```

### Useful commands
```bash
# show running containers for this compose project
docker compose ps

# follow app logs
docker compose logs -f app

# open a shell in the app container
docker compose exec app sh

# stop and remove containers
docker compose down

# clean rebuild
docker compose down --volumes
docker compose up --build -d
```

### Troubleshooting
- If a build or `postCreateCommand` step fails, run the failing command interactively inside the container to see full error output:
  ```bash
  docker compose exec app sh
  # then inside container
  composer install
  npm install
  ```
- Permission issues: the container runs as `www-data`. If files created by the host have different owners, you may need to chown inside the container:
  ```bash
  docker compose exec app sh -c "chown -R www-data:www-data /app/storage /app/bootstrap/cache"
  ```
- If ports are not reachable, verify Docker Desktop networking and that no other local process is using the same ports.

### Notes
- The devcontainer forwards 8080 and 3306 from container -> host. Visit http://localhost:8080 to see the app.
- The Dockerfile uses PHP 7.4 FPM and a Composer image for build helper steps.

If you'd like, I can add a badge or a short status script to the repo to quickly check container health or automate common developer tasks.
# DevContainer Configuration for Laravel 6.2.x

This devcontainer configuration provides a fully portable development environment for the Laravel 6.2 Pizzahouse project.

## What's Included

- **PHP 7.4-FPM** on Alpine Linux
- **Nginx** web server
- **Supervisor** for process management
- **Composer 2.x**
- **Laravel Installer** (globally available)
- **Node.js** with npm
- **MySQL 8.0** database
- All required PHP extensions (pdo_mysql, mbstring, gd, zip, bcmath, exif, pcntl)

## Supported IDEs

### PHPStorm / IntelliJ IDEA
PHPStorm 2025.2+ has native DevContainer support. See `phpstorm-setup.md` for detailed instructions.

**Note**: PHPStorm's DevContainer support works best with simple, well-defined containers. This setup has been optimized for compatibility.

### Visual Studio Code
VS Code has excellent DevContainer support via the Remote-Containers extension.

1. Install the "Dev Containers" extension
2. Open the pizzahouse folder
3. Click "Reopen in Container" when prompted

## How It Works

The devcontainer configuration:
- Uses Docker Compose to orchestrate the app and database containers
- Mounts your local code into the container at `/app`
- Forwards ports 8080 (web) and 3306 (MySQL) to your host
- Automatically runs `composer install && npm install` after container creation
- Runs Nginx and PHP-FPM via Supervisor for reliable service management

## Benefits

- **Portable**: Share the project with anyone; they get the exact same environment
- **Isolated**: No need to install PHP, Composer, Node.js on your host machine
- **Consistent**: Everyone on the team uses the same PHP version and extensions
- **Fast**: Alpine Linux-based for quick startup and small image size
- **Reliable**: Supervisor ensures services restart automatically if they crash

## Getting Started

1. Ensure Docker Desktop is installed and running
2. Open the **pizzahouse** directory in your IDE
3. For PHPStorm: Use Settings → PHP → CLI Interpreter → From Docker Compose
4. For VSCode: Click "Reopen in Container" when prompted

Your Laravel development environment is ready at **http://localhost:8080**

## First Time Setup

After the containers start for the first time, Laravel needs initial setup:

```bash
# The devcontainer.json postCreateCommand handles this automatically:
composer install
npm install

# But you may need to manually:
# 1. Create .env file (if not exists)
cp .env.example .env

# 2. Generate application key
php artisan key:generate

# 3. Fix permissions
chmod -R 775 storage bootstrap/cache
```

## Sharing This Project

When sharing this project with others:
1. They clone the repository
2. Open the pizzahouse folder in PHPStorm/VSCode
3. The IDE automatically sets up the environment
4. They run the first-time setup commands (or use postCreateCommand)
5. No manual PHP/MySQL/Composer installation needed!

## Technical Details

- **Base Image**: `php:7.4-fpm-alpine`
- **Web Server**: Nginx (configured for Laravel)
- **Process Manager**: Supervisor
- **Dockerfile**: Located at `../Dockerfile`
- **Docker Compose**: Located at `../docker-compose.yaml`
- **Container User**: root (Supervisor manages www-data for PHP-FPM/Nginx)

## Troubleshooting

### Container starts but web server doesn't respond
- Check logs: `docker logs pizzahouse-app-1`
- Verify services: `docker exec pizzahouse-app-1 ps aux | grep -E "nginx|php-fpm"`
- Ensure permissions: `docker exec pizzahouse-app-1 chmod -R 775 /app/storage /app/bootstrap/cache`

### PHPStorm "IJent unexpectedly closed" error
This error was resolved by switching from a complex base image (serversideup/php) to a simpler Alpine-based setup with Supervisor. The current configuration should work reliably with PHPStorm's DevContainer support.

### Database connection issues
- Verify MySQL is running: `docker ps`
- Check .env file has correct database credentials:
  - `DB_HOST=mysql`
  - `DB_DATABASE=laravel`
  - `DB_USERNAME=root`
  - `DB_PASSWORD=password`
