# SSL Setup

Configure SSL/TLS certificates for secure HTTPS access.

## Let's Encrypt (Recommended)

### Install Certbot

```bash
sudo apt update
sudo apt install certbot -y
```

### Obtain Certificate

Stop any service on port 80:

```bash
# Stop nginx if running
docker compose stop nginx

# Obtain certificate
sudo certbot certonly --standalone -d your-domain.com

# Certificates will be at:
# /etc/letsencrypt/live/your-domain.com/fullchain.pem
# /etc/letsencrypt/live/your-domain.com/privkey.pem
```

### Copy to Project

```bash
# Create SSL directory
mkdir -p nginx/ssl

# Copy certificates
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem nginx/ssl/
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem nginx/ssl/

# Set permissions
sudo chown -R $USER:$USER nginx/ssl/
chmod 600 nginx/ssl/*.pem
```

### Auto-Renewal

Create renewal script:

```bash
# Create script
cat > renew-ssl.sh << 'EOF'
#!/bin/bash
# Renew Let's Encrypt certificate
certbot renew --quiet

# Copy new certificates
cp /etc/letsencrypt/live/your-domain.com/fullchain.pem /opt/epidbot/nginx/ssl/
cp /etc/letsencrypt/live/your-domain.com/privkey.pem /opt/epidbot/nginx/ssl/

# Reload nginx
docker compose -f /opt/epidbot/docker-compose.yml restart nginx
EOF

chmod +x renew-ssl.sh
```

Add to crontab:

```bash
sudo crontab -e

# Add this line (runs monthly at midnight on the 1st)
0 0 1 * * /opt/epidbot/renew-ssl.sh
```

## Self-Signed Certificate (Development)

For development/testing only:

```bash
# Create SSL directory
mkdir -p nginx/ssl

# Generate self-signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/privkey.pem \
  -out nginx/ssl/fullchain.pem \
  -subj "/CN=localhost"
```

!!! warning
    Self-signed certificates will show browser warnings. Only use for development.

## SSL Configuration

### Nginx SSL Settings

```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # Certificate paths
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # Session settings
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # Modern TLS
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
}
```

### HTTP to HTTPS Redirect

```nginx
server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

## Certificate Chain

For some providers, you need to include the chain:

```bash
# Concatenate certificates
cat your_certificate.pem intermediate.pem > fullchain.pem
```

## Verify SSL

### Check Certificate

```bash
# Check expiration
openssl x509 -in nginx/ssl/fullchain.pem -noout -dates

# Check details
openssl x509 -in nginx/ssl/fullchain.pem -noout -text
```

### Online Testing

Use [SSL Labs](https://www.ssllabs.com/ssltest/) to test your configuration.

### Command Line Testing

```bash
# Test connection
openssl s_client -connect your-domain.com:443 -servername your-domain.com

# Check certificate
echo | openssl s_client -connect your-domain.com:443 2>/dev/null | openssl x509 -noout -dates
```

## Troubleshooting

### Certificate not found

```bash
# Check files exist
ls -la nginx/ssl/

# Check permissions
chmod 644 nginx/ssl/fullchain.pem
chmod 600 nginx/ssl/privkey.pem
```

### Browser warnings

- Ensure certificate matches domain
- Check certificate chain is complete
- Verify system time is correct

### Certificate expired

```bash
# Check expiration
openssl x509 -in nginx/ssl/fullchain.pem -noout -dates

# Renew Let's Encrypt
sudo certbot renew
sudo cp /etc/letsencrypt/live/your-domain.com/*.pem nginx/ssl/
docker compose restart nginx
```

## Security Best Practices

1. **Use strong ciphers**: Only allow TLS 1.2+
2. **Enable HSTS**: Force HTTPS connections
3. **Regular renewal**: Auto-renew Let's Encrypt
4. **Protect keys**: Set restrictive permissions on privkey.pem
5. **Monitor expiration**: Set up alerts for certificate expiry
