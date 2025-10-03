# PHPStorm Setup Guide - IMPORTANT

## ⚠️ PHPStorm Users: Do NOT Use DevContainers

**PHPStorm's DevContainer support is experimental and causes "IJent unexpectedly closed" errors.**

### ✅ Recommended Approach: Native Docker Compose Integration

See **`../PHPSTORM_SETUP.md`** for complete instructions.

**Quick Summary:**
1. Start containers: `docker-compose up -d`
2. Configure PHP Interpreter: Settings → PHP → CLI Interpreter → From Docker Compose
3. Point to `docker-compose.yaml` and select service `app`
4. Use lifecycle: **"Connect to existing container"**

---

## For VS Code Users

VS Code's DevContainer support is mature and stable. Follow the main setup:

1. Open the **pizzahouse** folder in VS Code
2. Install "Dev Containers" extension
3. Click "Reopen in Container" when prompted
4. Done!

---

## Why DevContainers Don't Work in PHPStorm

PHPStorm's DevContainer implementation (as of 2025.2.2):
- Creates `/.jbdevcontainer` directories in containers
- Uses experimental IJent protocol for communication
- Frequently fails with "IJent unexpectedly closed during initial protocol exchange"
- Is not production-ready for complex containers

**PHPStorm's native Docker Compose support is stable and recommended.**

---

## Container Stack (For Reference)

- **PHP**: 7.4-FPM on Alpine Linux
- **Nginx**: Web server (port 8080)
- **Supervisor**: Process manager
- **Composer**: 2.x
- **Node.js**: Latest with npm
- **MySQL**: 8.0 (port 3306)
- **Laravel Installer**: Globally installed

---

## Useful Commands

### Start/Stop Containers
```bash
# Start
docker-compose up -d

# Stop
docker-compose down

# Rebuild after Dockerfile changes
docker-compose up -d --build

# View logs
docker-compose logs -f app
```

### Laravel Commands in Container
```bash
# Artisan commands
docker exec pizzahouse-app-1 sh -c "cd /app && php artisan migrate"
docker exec pizzahouse-app-1 sh -c "cd /app && php artisan make:controller ExampleController"

# Composer commands
docker exec pizzahouse-app-1 sh -c "cd /app && composer install"
docker exec pizzahouse-app-1 sh -c "cd /app && composer require package/name"

# NPM commands
docker exec pizzahouse-app-1 sh -c "cd /app && npm install"
docker exec pizzahouse-app-1 sh -c "cd /app && npm run dev"
```

### Access Container Shell
```bash
docker exec -it pizzahouse-app-1 sh
```

---

## Troubleshooting

### "IJent unexpectedly closed" Error
**Solution**: Stop using PHPStorm DevContainers. Use Docker Compose integration instead (see `PHPSTORM_SETUP.md`).

### Container Won't Start
```bash
# Check container status
docker ps -a

# View container logs
docker logs pizzahouse-app-1

# Rebuild from scratch
docker-compose down
docker-compose up -d --build
```

### Permission Issues
```bash
# Fix Laravel storage permissions
docker exec pizzahouse-app-1 sh -c "chmod -R 775 /app/storage /app/bootstrap/cache"
```

### Web Server Not Responding
```bash
# Check if services are running
docker exec pizzahouse-app-1 ps aux | grep -E "nginx|php-fpm|supervisor"

# Should see:
# - supervisord (PID 1)
# - nginx master + workers
# - php-fpm master + workers
```

---

## Summary

| IDE | Method | Status |
|-----|--------|--------|
| **PHPStorm** | Docker Compose Integration | ✅ **Recommended** |
| **PHPStorm** | DevContainers | ❌ Causes IJent errors |
| **VS Code** | DevContainers | ✅ Works perfectly |

**PHPStorm users**: Follow `PHPSTORM_SETUP.md` for the working setup.
