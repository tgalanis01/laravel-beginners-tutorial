# PHPStorm Setup Guide for Laravel with Docker

## The Problem
PHPStorm's Dev Container feature (`/.jbdevcontainer`) is experimental and causes "IJent unexpectedly closed" errors.

## The Solution
Use PHPStorm's **native Docker Compose integration** instead. It's stable, mature, and works perfectly.

---

## Setup Instructions

### 1. Start the Docker Containers

```bash
cd C:\Users\tgalanis\PhpstormProjects\laravel-beginners-tutorial\pizzahouse
docker-compose up -d
```

Verify containers are running:
```bash
docker ps
```

You should see:
- `pizzahouse-app-1` (running)
- `pizzahouse-mysql-1` (running)

### 2. Configure PHP Interpreter in PHPStorm

1. Open PHPStorm and load the `pizzahouse` folder as your project
2. Go to **File** → **Settings** (or **Ctrl+Alt+S**)
3. Navigate to **PHP**
4. Next to "CLI Interpreter", click the **`...`** button
5. Click the **`+`** button → Select **"From Docker, Vagrant, VM, WSL, Remote..."**
6. Select **Docker Compose**
7. Configure:
   - **Configuration file(s)**: Browse to `C:\Users\tgalanis\PhpstormProjects\laravel-beginners-tutorial\pizzahouse\docker-compose.yaml`
   - **Service**: Select `app`
   - **Lifecycle**: Use **"Connect to existing container"** (important!)
8. Click **OK**

PHPStorm will now detect:
- PHP version: 7.4.33
- Composer (already installed in container)
- All PHP extensions

### 3. Configure Database Connection

1. Open the **Database** tool window (View → Tool Windows → Database)
2. Click **`+`** → **Data Source** → **MySQL**
3. Configure:
   - **Host**: `localhost`
   - **Port**: `3306`
   - **Database**: `laravel`
   - **User**: `root`
   - **Password**: `password`
4. Test Connection → Click **OK**

### 4. Configure Path Mappings (Usually Automatic)

PHPStorm should auto-detect, but verify:
1. Go to **Settings** → **PHP** → **Servers**
2. Check if path mapping exists:
   - **Local path**: `C:\Users\tgalanis\PhpstormProjects\laravel-beginners-tutorial\pizzahouse`
   - **Remote path**: `/app`

### 5. Run Composer & Artisan Commands

#### Option A: Through PHPStorm Terminal
PHPStorm's terminal will automatically use the container:
```bash
composer install
php artisan key:generate
php artisan migrate
```

#### Option B: Through Docker Exec
```bash
docker exec pizzahouse-app-1 sh -c "cd /app && composer install"
docker exec pizzahouse-app-1 sh -c "cd /app && php artisan key:generate"
```

---

## Testing

1. **Visit the application**: http://localhost:8080
2. **You should see**: Laravel welcome page
3. **PHPStorm features should work**:
   - Code completion ✅
   - Debugging ✅
   - Database queries ✅
   - Artisan commands ✅

---

## Debugging Configuration

### Enable Xdebug (Optional)

If you need debugging, add Xdebug to the Dockerfile:

```dockerfile
# In pizzahouse/Dockerfile, after installing PHP extensions, add:
RUN apk add --no-cache $PHPIZE_DEPS \
    && pecl install xdebug-3.1.6 \
    && docker-php-ext-enable xdebug \
    && apk del $PHPIZE_DEPS

# Configure Xdebug
RUN echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_host=host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_port=9003" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
```

Then rebuild:
```bash
docker-compose down
docker-compose up -d --build
```

In PHPStorm:
1. Settings → PHP → Debug
2. Set Xdebug port to `9003`
3. Enable "Start Listening for PHP Debug Connections"

---

## Common Commands

### Start containers:
```bash
docker-compose up -d
```

### Stop containers:
```bash
docker-compose down
```

### View logs:
```bash
docker-compose logs -f app
```

### Rebuild after Dockerfile changes:
```bash
docker-compose down
docker-compose up -d --build
```

### Access container shell:
```bash
docker exec -it pizzahouse-app-1 sh
```

### Run Artisan commands:
```bash
docker exec pizzahouse-app-1 sh -c "cd /app && php artisan [command]"
```

### Run Composer commands:
```bash
docker exec pizzahouse-app-1 sh -c "cd /app && composer [command]"
```

---

## Troubleshooting

### "Cannot connect to Docker daemon"
- Ensure Docker Desktop is running
- Restart Docker Desktop if needed

### "Port 8080 already in use"
- Check what's using the port: `netstat -ano | findstr :8080`
- Change the port in `docker-compose.yaml`: `"8081:8080"`

### "PHP interpreter not working"
- Make sure containers are running: `docker ps`
- Reconfigure interpreter in PHPStorm settings
- Use "Connect to existing container" lifecycle option

### Code changes not reflecting
- The container mounts your local code via volumes
- Changes should appear instantly
- If not, restart containers: `docker-compose restart app`

---

## Why Not DevContainers?

DevContainers (`.devcontainer`) work great in **VS Code** but PHPStorm's implementation is:
- Still experimental (as of 2025.2)
- Causes IJent communication errors
- Less stable than native Docker Compose support

**For PHPStorm users**: Use Docker Compose integration (this guide)
**For VS Code users**: DevContainers work perfectly - use the `.devcontainer` folder

---

## Summary

✅ **Works**: PHPStorm + Docker Compose (this method)
❌ **Unstable**: PHPStorm + DevContainers (causes IJent errors)
✅ **Works**: VS Code + DevContainers

Your development environment is now fully configured and portable!
