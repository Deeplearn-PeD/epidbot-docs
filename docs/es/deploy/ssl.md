# Configuración SSL

Configure certificados SSL/TLS para acceso HTTPS seguro.

## Let's Encrypt (Recomendado)

### Instalar Certbot

```bash
sudo apt update
sudo apt install certbot -y
```

### Obtener Certificado

Detenga cualquier servicio en el puerto 80:

```bash
# Detener nginx si está ejecutándose
docker compose stop nginx

# Obtener certificado
sudo certbot certonly --standalone -d your-domain.com

# Los certificados estarán en:
# /etc/letsencrypt/live/your-domain.com/fullchain.pem
# /etc/letsencrypt/live/your-domain.com/privkey.pem
```

### Copiar al Proyecto

```bash
# Crear directorio SSL
mkdir -p nginx/ssl

# Copiar certificados
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem nginx/ssl/
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem nginx/ssl/

# Establecer permisos
sudo chown -R $USER:$USER nginx/ssl/
chmod 600 nginx/ssl/*.pem
```

### Renovación Automática

Cree script de renovación:

```bash
# Crear script
cat > renew-ssl.sh << 'EOF'
#!/bin/bash
# Renovar certificado Let's Encrypt
certbot renew --quiet

# Copiar nuevos certificados
cp /etc/letsencrypt/live/your-domain.com/fullchain.pem /opt/epidbot/nginx/ssl/
cp /etc/letsencrypt/live/your-domain.com/privkey.pem /opt/epidbot/nginx/ssl/

# Recargar nginx
docker compose -f /opt/epidbot/docker-compose.yml restart nginx
EOF

chmod +x renew-ssl.sh
```

Añada a crontab:

```bash
sudo crontab -e

# Añada esta línea (ejecuta mensualmente a medianoche el día 1)
0 0 1 * * /opt/epidbot/renew-ssl.sh
```

## Certificado Autofirmado (Desarrollo)

Solo para desarrollo/pruebas:

```bash
# Crear directorio SSL
mkdir -p nginx/ssl

# Generar certificado autofirmado
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/privkey.pem \
  -out nginx/ssl/fullchain.pem \
  -subj "/CN=localhost"
```

!!! warning
    Los certificados autofirmados mostrarán advertencias del navegador. Solo use para desarrollo.

## Configuración SSL

### Configuración SSL de Nginx

```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # Rutas de certificados
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # Configuración de sesión
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # TLS Moderno
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
}
```

### Redirección HTTP a HTTPS

```nginx
server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

## Cadena de Certificados

Para algunos proveedores, necesita incluir la cadena:

```bash
# Concatenar certificados
cat your_certificate.pem intermediate.pem > fullchain.pem
```

## Verificar SSL

### Verificar Certificado

```bash
# Verificar expiración
openssl x509 -in nginx/ssl/fullchain.pem -noout -dates

# Verificar detalles
openssl x509 -in nginx/ssl/fullchain.pem -noout -text
```

### Prueba en Línea

Use [SSL Labs](https://www.ssllabs.com/ssltest/) para probar su configuración.

### Prueba desde Línea de Comandos

```bash
# Probar conexión
openssl s_client -connect your-domain.com:443 -servername your-domain.com

# Verificar certificado
echo | openssl s_client -connect your-domain.com:443 2>/dev/null | openssl x509 -noout -dates
```

## Solución de Problemas

### Certificado no encontrado

```bash
# Verificar que los archivos existen
ls -la nginx/ssl/

# Verificar permisos
chmod 644 nginx/ssl/fullchain.pem
chmod 600 nginx/ssl/privkey.pem
```

### Advertencias del navegador

- Asegúrese de que el certificado coincide con el dominio
- Verifique que la cadena del certificado esté completa
- Verifique que la hora del sistema sea correcta

### Certificado expirado

```bash
# Verificar expiración
openssl x509 -in nginx/ssl/fullchain.pem -noout -dates

# Renovar Let's Encrypt
sudo certbot renew
sudo cp /etc/letsencrypt/live/your-domain.com/*.pem nginx/ssl/
docker compose restart nginx
```

## Mejores Prácticas de Seguridad

1. **Use cifrados fuertes**: Solo permita TLS 1.2+
2. **Habilite HSTS**: Fuerce conexiones HTTPS
3. **Renovación regular**: Auto-renueve Let's Encrypt
4. **Proteja claves**: Establezca permisos restrictivos en privkey.pem
5. **Monitoree expiración**: Configure alertas para vencimiento de certificados
