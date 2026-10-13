# Nginx Configuration

Configure Nginx as a reverse proxy for EpidBot.

## Basic Configuration

Create `nginx/nginx.conf`:

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=api:10m rate=30r/s;

    # Upstreams
    upstream epidbot {
        server epidbot:3000;
        keepalive 32;
    }

    upstream libby {
        server libby:8000;
        keepalive 16;
    }

    # HTTP to HTTPS redirect
    server {
        listen 80;
        server_name _;
        return 301 https://$host$request_uri;
    }

    # HTTPS Server
    server {
        listen 443 ssl http2;
        server_name your-domain.com;

        # SSL
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers off;

        # Security headers
        add_header Strict-Transport-Security "max-age=63072000" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;

        # Max upload size
        client_max_body_size 50M;

        # EpidBot
        location / {
            limit_req zone=general burst=20 nodelay;
            
            proxy_pass http://epidbot;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Connection "";

            # WebSocket support
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }

        # Libby API
        location /api/kb/ {
            limit_req zone=api burst=50 nodelay;
            
            rewrite ^/api/kb/search$ /api/retrieve break;
            rewrite ^/api/kb/(.*)$ /api/$1 break;
            
            proxy_pass http://libby;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Health checks
        location /health/epidbot {
            access_log off;
            proxy_pass http://epidbot/;
        }

        location /health/libby {
            access_log off;
            proxy_pass http://libby/api/health;
        }
    }
}
```

## Rate Limiting

Configure rate limiting zones:

```nginx
# In http block
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=api:10m rate=30r/s;
limit_req_zone $binary_remote_addr zone=upload:10m rate=5r/s;
```

Apply to locations:

```nginx
location / {
    limit_req zone=general burst=20 nodelay;
    # ...
}

location /api/kb/embed/upload {
    limit_req zone=upload burst=5 nodelay;
    # ...
}
```

## WebSocket Support

For real-time updates:

```nginx
location / {
    proxy_pass http://epidbot;
    proxy_http_version 1.1;
    
    # WebSocket headers
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    
    # Long timeout for WebSocket
    proxy_read_timeout 86400;
}
```

## Static File Caching

```nginx
location /static/ {
    proxy_pass http://epidbot/static/;
    
    # Cache static assets
    expires 1d;
    add_header Cache-Control "public, immutable";
}

location /file= {
    proxy_pass http://epidbot;
    
    # Cache plot files
    expires 1h;
    add_header Cache-Control "public";
}
```

## Security Headers

```nginx
server {
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
    
    # Prevent clickjacking
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # Prevent MIME sniffing
    add_header X-Content-Type-Options "nosniff" always;
    
    # XSS protection
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

## File Upload Size

For document uploads:

```nginx
http {
    # Global limit
    client_max_body_size 50M;
}

server {
    # Upload endpoint limit
    location /api/kb/embed/upload {
        client_max_body_size 100M;
        # ...
    }
}
```

## Logging

Custom log format:

```nginx
log_format detailed '$remote_addr - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent" '
                     'rt=$request_time uct="$upstream_connect_time" '
                     'uht="$upstream_header_time" urt="$upstream_response_time"';

access_log /var/log/nginx/access.log detailed;
```

## Health Checks

```nginx
# EpidBot health
location /health/epidbot {
    access_log off;
    proxy_pass http://epidbot/;
    proxy_http_version 1.1;
}

# Libby health
location /health/libby {
    access_log off;
    proxy_pass http://libby/api/health;
    proxy_http_version 1.1;
}
```

Test health checks:

```bash
curl https://your-domain.com/health/epidbot
curl https://your-domain.com/health/libby
```

## Troubleshooting

### Check configuration

```bash
docker exec epidbot-nginx nginx -t
```

### Reload configuration

```bash
docker exec epidbot-nginx nginx -s reload
```

### View logs

```bash
docker exec epidbot-nginx tail -f /var/log/nginx/error.log
docker exec epidbot-nginx tail -f /var/log/nginx/access.log
```

### Common issues

**502 Bad Gateway:**
- Backend service not running
- Check `docker compose ps`
- Check backend logs

**413 Request Entity Too Large:**
- Increase `client_max_body_size`

**WebSocket disconnects:**
- Check `proxy_read_timeout`
- Verify `Upgrade` headers
