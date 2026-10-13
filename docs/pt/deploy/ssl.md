# Configuração SSL

Configure certificados SSL/TLS para acesso HTTPS seguro.

## Let's Encrypt (Recomendado)

### Instalar Certbot

```bash
sudo apt update
sudo apt install certbot -y
```

### Obter Certificado

Pare qualquer serviço na porta 80:

```bash
# Parar nginx se estiver rodando
docker compose stop nginx

# Obter certificado
sudo certbot certonly --standalone -d your-domain.com

# Certificados estarão em:
# /etc/letsencrypt/live/your-domain.com/fullchain.pem
# /etc/letsencrypt/live/your-domain.com/privkey.pem
```

### Copiar para o Projeto

```bash
# Criar diretório SSL
mkdir -p nginx/ssl

# Copiar certificados
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem nginx/ssl/
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem nginx/ssl/

# Definir permissões
sudo chown -R $USER:$USER nginx/ssl/
chmod 600 nginx/ssl/*.pem
```

### Renovação Automática

Criar script de renovação:

```bash
# Criar script
cat > renew-ssl.sh << 'EOF'
#!/bin/bash
# Renovar certificado Let's Encrypt
certbot renew --quiet

# Copiar novos certificados
cp /etc/letsencrypt/live/your-domain.com/fullchain.pem /opt/epidbot/nginx/ssl/
cp /etc/letsencrypt/live/your-domain.com/privkey.pem /opt/epidbot/nginx/ssl/

# Recarregar nginx
docker compose -f /opt/epidbot/docker-compose.yml restart nginx
EOF

chmod +x renew-ssl.sh
```

Adicionar ao crontab:

```bash
sudo crontab -e

# Adicionar esta linha (executa mensalmente à meia-noite no dia 1)
0 0 1 * * /opt/epidbot/renew-ssl.sh
```

## Certificado Auto-assinado (Desenvolvimento)

Apenas para desenvolvimento/testes:

```bash
# Criar diretório SSL
mkdir -p nginx/ssl

# Gerar certificado auto-assinado
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/privkey.pem \
  -out nginx/ssl/fullchain.pem \
  -subj "/CN=localhost"
```

!!! warning
    Certificados auto-assinados mostrarão avisos no navegador. Use apenas para desenvolvimento.

## Configuração SSL

### Configurações SSL do Nginx

```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # Caminhos dos certificados
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # Configurações de sessão
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # TLS moderno
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
}
```

### Redirecionamento HTTP para HTTPS

```nginx
server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

## Cadeia de Certificados

Para alguns provedores, você precisa incluir a cadeia:

```bash
# Concatenar certificados
cat your_certificate.pem intermediate.pem > fullchain.pem
```

## Verificar SSL

### Verificar Certificado

```bash
# Verificar expiração
openssl x509 -in nginx/ssl/fullchain.pem -noout -dates

# Verificar detalhes
openssl x509 -in nginx/ssl/fullchain.pem -noout -text
```

### Teste Online

Use [SSL Labs](https://www.ssllabs.com/ssltest/) para testar sua configuração.

### Teste via Linha de Comando

```bash
# Testar conexão
openssl s_client -connect your-domain.com:443 -servername your-domain.com

# Verificar certificado
echo | openssl s_client -connect your-domain.com:443 2>/dev/null | openssl x509 -noout -dates
```

## Solução de Problemas

### Certificado não encontrado

```bash
# Verificar se arquivos existem
ls -la nginx/ssl/

# Verificar permissões
chmod 644 nginx/ssl/fullchain.pem
chmod 600 nginx/ssl/privkey.pem
```

### Avisos do navegador

- Garanta que o certificado corresponde ao domínio
- Verifique se a cadeia de certificados está completa
- Verifique se a hora do sistema está correta

### Certificado expirado

```bash
# Verificar expiração
openssl x509 -in nginx/ssl/fullchain.pem -noout -dates

# Renovar Let's Encrypt
sudo certbot renew
sudo cp /etc/letsencrypt/live/your-domain.com/*.pem nginx/ssl/
docker compose restart nginx
```

## Melhores Práticas de Segurança

1. **Use cifras fortes**: Permita apenas TLS 1.2+
2. **Habilite HSTS**: Force conexões HTTPS
3. **Renovação regular**: Renove automaticamente Let's Encrypt
4. **Proteja chaves**: Defina permissões restritivas em privkey.pem
5. **Monitore expiração**: Configure alertas para expiração de certificados
