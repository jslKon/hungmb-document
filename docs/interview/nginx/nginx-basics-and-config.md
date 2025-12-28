# Nginx Basics and Configuration Guide

## Table of Contents

1. [What is Nginx?](#what-is-nginx)
2. [Key Features](#key-features)
3. [Installation](#installation)
4. [Basic Configuration Structure](#basic-configuration-structure)
5. [Main Configuration File](#main-configuration-file)
6. [Server Blocks (Virtual Hosts)](#server-blocks-virtual-hosts)
7. [Location Blocks](#location-blocks)
8. [Common Directives](#common-directives)
9. [Load Balancing](#load-balancing)
10. [SSL/TLS Configuration](#ssltls-configuration)
11. [Caching](#caching)
12. [Security Headers](#security-headers)
13. [Common Use Cases](#common-use-cases)
14. [Troubleshooting](#troubleshooting)

## What is Nginx?

Nginx (pronounced "engine-x") is a high-performance web server, reverse proxy, and load balancer.
It's designed to handle high concurrency with low memory usage, making it ideal for serving static
content and acting as a reverse proxy for dynamic applications.

## Key Features

- **High Performance**: Event-driven, asynchronous architecture
- **Low Memory Usage**: Efficient memory management
- **Reverse Proxy**: Forward requests to backend servers
- **Load Balancing**: Distribute traffic across multiple servers
- **SSL Termination**: Handle HTTPS encryption/decryption
- **Caching**: Store frequently accessed content
- **Compression**: Gzip compression for better performance
- **Static File Serving**: Efficiently serve static assets

## Installation

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install nginx
```

### CentOS/RHEL

```bash
sudo yum install nginx
# or for newer versions
sudo dnf install nginx
```

### Docker

```bash
docker run -d -p 80:80 nginx
```

## Basic Configuration Structure

Nginx configuration follows a hierarchical structure:

```
main context
├── events { }
├── http { }
│   ├── server { }
│   │   ├── location { }
│   │   └── location { }
│   └── server { }
└── stream { }
```

## Main Configuration File

The main configuration file is typically located at:

- **Linux**: `/etc/nginx/nginx.conf`
- **Windows**: `C:\nginx\conf\nginx.conf`

### Basic nginx.conf Structure

```nginx
# Main context - global settings
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Events context - connection processing
events {
    worker_connections 1024;
    use epoll;  # Linux only
    multi_accept on;
}

# HTTP context - web server configuration
http {
    # Basic settings
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    
    # Performance settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    
    # Include server configurations
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

## Server Blocks (Virtual Hosts)

Server blocks define how Nginx handles requests for specific domains or IP addresses.

### Basic Server Block

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html index.htm;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Multiple Server Blocks

```nginx
# HTTP server (redirects to HTTPS)
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html;
    
    # SSL configuration
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

## Location Blocks

Location blocks define how to handle requests for specific URL patterns.

### Location Block Types

```nginx
server {
    # Exact match (highest priority)
    location = /exact {
        return 200 "Exact match";
    }
    
    # Prefix match (longest match wins)
    location /api/ {
        proxy_pass http://backend;
    }
    
    # Regular expression match (case sensitive)
    location ~ \.(jpg|jpeg|png|gif)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Regular expression match (case insensitive)
    location ~* \.(css|js)$ {
        expires 1M;
    }
    
    # Regular expression match (highest priority)
    location ^~ /static/ {
        root /var/www;
    }
}
```

### Common Location Patterns

```nginx
server {
    # Serve static files
    location /static/ {
        alias /var/www/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # API proxy
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # PHP processing
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    # Deny access to sensitive files
    location ~ /\. {
        deny all;
    }
}
```

## Common Directives

### Basic Directives

```nginx
# Server identification
server_name example.com www.example.com *.example.com;

# Document root
root /var/www/html;

# Default files
index index.html index.htm index.php;

# Error pages
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;

# Client settings
client_max_body_size 10M;
client_body_timeout 60s;
client_header_timeout 60s;

# Connection settings
keepalive_timeout 65;
keepalive_requests 100;
```

### Proxy Directives

```nginx
# Upstream server
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

# Proxy settings
proxy_pass http://backend;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_connect_timeout 30s;
proxy_send_timeout 30s;
proxy_read_timeout 30s;
proxy_buffering on;
proxy_buffer_size 4k;
proxy_buffers 8 4k;
```

## Load Balancing

### Basic Load Balancing

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### Load Balancing Methods

```nginx
upstream backend {
    # Round Robin (default)
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Least Connections
    least_conn;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # IP Hash
    ip_hash;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Weighted
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=1;
    
    # Health checks
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 backup;
}
```

## SSL/TLS Configuration

### Basic SSL Setup

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # SSL certificates
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    
    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    
    location / {
        root /var/www/html;
        index index.html;
    }
}
```

### HTTP to HTTPS Redirect

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

## Caching

### Proxy Caching

```nginx
http {
    # Cache configuration
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;
    
    server {
        location / {
            proxy_cache my_cache;
            proxy_cache_valid 200 1h;
            proxy_cache_valid 404 1m;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
            proxy_cache_lock on;
            
            proxy_pass http://backend;
        }
    }
}
```

### Static File Caching

```nginx
server {
    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Vary Accept-Encoding;
    }
    
    # Cache HTML files for shorter time
    location ~* \.html$ {
        expires 1h;
        add_header Cache-Control "public";
    }
}
```

## Security Headers

```nginx
server {
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Hide Nginx version
    server_tokens off;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=login:10m rate=10r/m;
    
    location /login {
        limit_req zone=login burst=5 nodelay;
        proxy_pass http://backend;
    }
}
```

## Common Use Cases

### 1. Static File Server

```nginx
server {
    listen 80;
    server_name static.example.com;
    root /var/www/static;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 2. Reverse Proxy for API

```nginx
upstream api_backend {
    server 192.168.1.10:3000;
    server 192.168.1.11:3000;
}

server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://api_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3. Load Balancer for Web Applications

```nginx
upstream web_app {
    least_conn;
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:8080 backup;
}

server {
    listen 80;
    server_name app.example.com;
    
    location / {
        proxy_pass http://web_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 4. Microservices Gateway

```nginx
upstream user_service {
    server 192.168.1.10:3001;
}

upstream order_service {
    server 192.168.1.11:3002;
}

upstream payment_service {
    server 192.168.1.12:3003;
}

server {
    listen 80;
    server_name api.example.com;
    
    location /api/users/ {
        proxy_pass http://user_service/;
    }
    
    location /api/orders/ {
        proxy_pass http://order_service/;
    }
    
    location /api/payments/ {
        proxy_pass http://payment_service/;
    }
}
```

## Troubleshooting

### Common Commands

```bash
# Test configuration
sudo nginx -t

# Reload configuration
sudo nginx -s reload

# Restart Nginx
sudo systemctl restart nginx

# Check status
sudo systemctl status nginx

# View logs
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

### Common Issues

1. **502 Bad Gateway**: Backend server is down or unreachable
2. **403 Forbidden**: Permission issues or missing index file
3. **404 Not Found**: Incorrect root path or missing files
4. **SSL Certificate Errors**: Invalid or expired certificates
5. **Connection Refused**: Nginx not running or wrong port

### Debugging Tips

```nginx
# Enable debug logging
error_log /var/log/nginx/error.log debug;

# Add debug headers
add_header X-Debug-Backend $upstream_addr;
add_header X-Debug-Request-URI $request_uri;
```

### Performance Tuning

```nginx
# Worker processes
worker_processes auto;

# Worker connections
events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

# Buffer sizes
client_body_buffer_size 128k;
client_header_buffer_size 1k;
large_client_header_buffers 4 4k;

# Timeouts
client_body_timeout 12;
client_header_timeout 12;
keepalive_timeout 15;
send_timeout 10;
```

## Best Practices

1. **Security**: Always use HTTPS, implement security headers, and keep Nginx updated
2. **Performance**: Enable gzip compression, use appropriate caching, and optimize worker processes
3. **Monitoring**: Set up log monitoring and health checks
4. **Configuration**: Use include files for better organization
5. **Testing**: Always test configuration changes before applying to production

This guide covers the essential Nginx concepts and configuration patterns you'll need for most web
server and reverse proxy scenarios.
