# Configuração do Nginx

Configure o Nginx como proxy reverso para o EpidBot.

## Configuração Básica

Criar `nginx/nginx.conf`:

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

    # Compressão Gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;

    # Limitação de taxa
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

    # Redirecionamento HTTP para HTTPS
    server {
        listen 80;
        server_name _;
        return 301 https://$host$request_uri;
    }

    # Servidor HTTPS
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

        # Cabeçalhos de segurança
        add_header Strict-Transport-Security "max-age=63072000" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;

        # Tamanho máximo de upload
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

            # Suporte a WebSocket
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

## Limitação de Taxa

Configure zonas de limitação de taxa:

```nginx
# No bloco http
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=api:10m rate=30r/s;
limit_req_zone $binary_remote_addr zone=upload:10m rate=5r/s;
```

Aplique às localizações:

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

## Suporte a WebSocket

Para atualizações em tempo real:

```nginx
location / {
    proxy_pass http://epidbot;
    proxy_http_version 1.1;
    
    # Cabeçalhos WebSocket
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    
    # Timeout longo para WebSocket
    proxy_read_timeout 86400;
}
```

## Cache de Arquivos Estáticos

```nginx
location /static/ {
    proxy_pass http://epidbot/static/;
    
    # Cachear assets estáticos
    expires 1d;
    add_header Cache-Control "public, immutable";
}

location /file= {
    proxy_pass http://epidbot;
    
    # Cachear arquivos de plot
    expires 1h;
    add_header Cache-Control "public";
}
```

## Cabeçalhos de Segurança

```nginx
server {
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
    
    # Prevenir clickjacking
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # Prevenir MIME sniffing
    add_header X-Content-Type-Options "nosniff" always;
    
    # Proteção XSS
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Política de referrer
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

## Tamanho de Upload de Arquivos

Para uploads de documentos:

```nginx
http {
    # Limite global
    client_max_body_size 50M;
}

server {
    # Limite do endpoint de upload
    location /api/kb/embed/upload {
        client_max_body_size 100M;
        # ...
    }
}
```

## Logging

Formato de log personalizado:

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
# Health check do EpidBot
location /health/epidbot {
    access_log off;
    proxy_pass http://epidbot/;
    proxy_http_version 1.1;
}

# Health check do Libby
location /health/libby {
    access_log off;
    proxy_pass http://libby/api/health;
    proxy_http_version 1.1;
}
```

Testar health checks:

```bash
curl https://your-domain.com/health/epidbot
curl https://your-domain.com/health/libby
```

## Solução de Problemas

### Verificar configuração

```bash
docker exec epidbot-nginx nginx -t
```

### Recarregar configuração

```bash
docker exec epidbot-nginx nginx -s reload
```

### Ver logs

```bash
docker exec epidbot-nginx tail -f /var/log/nginx/error.log
docker exec epidbot-nginx tail -f /var/log/nginx/access.log
```

### Problemas comuns

**502 Bad Gateway:**
- Serviço backend não está rodando
- Verifique `docker compose ps`
- Verifique logs do backend

**413 Request Entity Too Large:**
- Aumente `client_max_body_size`

**WebSocket desconecta:**
- Verifique `proxy_read_timeout`
- Verifique cabeçalhos `Upgrade`
