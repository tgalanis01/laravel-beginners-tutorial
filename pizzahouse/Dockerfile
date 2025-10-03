FROM php:7.4-fpm-alpine
LABEL authors="tgalanis"

# Install system dependencies and PHP extensions
RUN apk add --no-cache \
    nginx \
    supervisor \
    git \
    curl \
    libzip-dev \
    libpng-dev \
    libjpeg-turbo-dev \
    freetype-dev \
    oniguruma-dev \
    nodejs \
    npm \
    bash \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Install Laravel installer globally
RUN composer global require laravel/installer

# Add Composer global bin to PATH
ENV PATH="${PATH}:/root/.composer/vendor/bin"

# Configure Nginx
RUN mkdir -p /run/nginx

# Copy Nginx configuration
COPY <<EOF /etc/nginx/http.d/default.conf
server {
    listen 8080;
    server_name localhost;
    root /app/public;
    index index.php index.html;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF

# Configure Supervisor
COPY <<EOF /etc/supervisord.conf
[supervisord]
nodaemon=true
user=root
logfile=/dev/stdout
logfile_maxbytes=0
pidfile=/var/run/supervisord.pid

[program:php-fpm]
command=php-fpm -F
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
autorestart=true

[program:nginx]
command=nginx -g 'daemon off;'
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
autorestart=true
EOF

# Ensure php-fpm listens on TCP 127.0.0.1:9000 so nginx's fastcgi_pass can connect
# (the default php-fpm pool may use a unix socket depending on the base image)
RUN if [ -f /usr/local/etc/php-fpm.d/www.conf ]; then \
    sed -i "s/^listen = .*/listen = 127.0.0.1:9000/" /usr/local/etc/php-fpm.d/www.conf || true; \
fi

# Set working directory
WORKDIR /app

# Set proper permissions
RUN chown -R www-data:www-data /app /var/lib/nginx /var/log/nginx

# Expose port
EXPOSE 8080

# Start supervisor
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]

# Ensure container runs as root by default so shells/terminals can start
# (some environments set a user with /sbin/nologin which prevents interactive shells)
USER root
