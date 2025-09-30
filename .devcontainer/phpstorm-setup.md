# PHPStorm Docker Setup

## Quick Start
1. Navigate to pizzahouse directory: `cd pizzahouse`
2. Start containers: `docker-compose up -d`
3. Open project in PHPStorm

## PHP Interpreter Setup
1. Go to **File** → **Settings** → **PHP**
2. Click **...** button next to CLI Interpreter
3. Click **+** → **From Docker, Vagrant, VM, WSL, Remote...**
4. Select **Docker Compose**
5. Configuration file: `pizzahouse/docker-compose.yaml`
6. Service: `app`
7. PHPStorm will auto-detect PHP 8.0

## Database Connection
1. Open **Database** tool window
2. Add **MySQL** datasource:
   - Host: `localhost`
   - Port: `3306`
   - Database: `laravel`
   - User: `root`
   - Password: `password`

## Path Mappings
Ensure path mapping is set to:
- **Local path**: `<project-root>/pizzahouse`
- **Remote path**: `/app`

## Useful Commands
- Start containers: `docker-compose up -d`
- Stop containers: `docker-compose down`
- View logs: `docker-compose logs -f`
- Access container: `docker-compose exec app bash`

## Laravel Commands in Container
```bash
# Artisan commands
docker-compose exec app php artisan migrate
docker-compose exec app php artisan make:controller ExampleController

# Composer commands
docker-compose exec app composer install
docker-compose exec app composer require package/name
```