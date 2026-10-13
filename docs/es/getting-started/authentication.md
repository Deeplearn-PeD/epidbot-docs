# Autenticación

EpidBot utiliza autenticación por usuario/contraseña con gestión segura de sesiones.

## Descripción General

EpidBot requiere autenticación para acceder a la aplicación. Esto garantiza:
- Aislamiento de usuarios (cada usuario tiene sus propias sesiones y datos)
- Control de acceso seguro
- Persistencia de sesiones

## Sistema de Autenticación

### Características de Seguridad

- **Hash de Contraseñas**: Todas las contraseñas se hashean usando bcrypt con generación automática de salt
- **Tokens JWT**: access tokens de corta duración (HS256, ~30 minutos) renovados automáticamente por refresh tokens de ~7 días, revocables y almacenados en PostgreSQL
- **Claves de API**: acceso programático opcional vía `X-API-Key`
- **OAuth**: inicio de sesión social con proveedores configurables

### Gestión de Usuarios

Los usuarios se gestionan a través de la herramienta de línea de comandos `auth_cli.py`.

## Creando Usuarios

### Configuración Inicial

Antes de iniciar EpidBot por primera vez, cree un usuario administrador:

```bash
uv run python auth_cli.py create-admin
```

Se le solicitará:
- Nombre de usuario (obligatorio)
- Contraseña (obligatorio, no se mostrará)
- Confirmación de contraseña
- Email (opcional)

Ejemplo:

```bash
$ uv run python auth_cli.py create-admin
=== Create Admin User ===
Username: admin
Password: 
Confirm password: 
Email (optional, press Enter to skip): admin@example.com

✓ Admin user created successfully!
  Username: admin
  User ID: 1
  Email: admin@example.com
```

### Creando Usuarios Adicionales

Use el mismo comando `create-admin` para crear usuarios adicionales:

```bash
uv run python auth_cli.py create-admin
```

## Gestionando Usuarios

### Listar Todos los Usuarios

```bash
uv run python auth_cli.py list
```

Salida:

```
=== User List ===

ID    Username             Email                          Created              Last Login          
-----------------------------------------------------------------------------------------------
1     admin                admin@example.com              2026-03-10           2026-03-10          
2     researcher           researcher@institute.org       2026-03-11           Never               

Total: 2 user(s)
```

### Cambiar Contraseña

```bash
uv run python auth_cli.py passwd <username>
```

Ejemplo:

```bash
$ uv run python auth_cli.py passwd admin
=== Change Password for 'admin' ===
New password: 
Confirm new password: 

✓ Password changed successfully for user 'admin'
```

### Eliminar Usuario

```bash
uv run python auth_cli.py delete <username>
```

Ejemplo:

```bash
$ uv run python auth_cli.py delete researcher
=== Delete User 'researcher' ===
⚠️  WARNING: This action cannot be undone!
Type 'yes' to confirm deletion of user 'researcher': yes

⚠️  Note: This will delete the user but preserve their chat history.
    To fully implement user deletion with cascade, database-level constraints need to be updated.

✓ User 'researcher' marked for deletion (manual database cleanup may be required)
```

## Iniciando Sesión

### Via Interfaz Web

1. Inicie EpidBot: `cd frontend && pnpm dev`
2. Abra el navegador en `http://localhost:3000`
3. Ingrese su nombre de usuario y contraseña
4. Haga clic en "Login"

### Gestión de Sesión

- Las sesiones duran 30 días por defecto
- Permanecerá conectado hasta que la sesión expire o borre las cookies del navegador
- Cada usuario tiene sesiones aisladas e historial de chat propio

## Variables de Entorno

### SECRET_KEY

La variable de entorno `SECRET_KEY` se usa para encriptar tokens de sesión:

```bash
export SECRET_KEY=su-clave-secreta-aleatoria-aqui
```

**Importante**: 
- Genere una clave aleatoria segura para producción
- Mantenga esta clave en secreto
- Si la cambia, todas las sesiones existentes serán invalidadas

Para generar una clave segura:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Agregue a su archivo `.env`:

```env
SECRET_KEY=clave-generada-aqui
```

## Esquema de Base de Datos

Los usuarios se almacenan en la tabla `users` en **PostgreSQL** (`epidbot/db/schema.py`):

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | INTEGER | ID de usuario (auto-generado) |
| `username` | VARCHAR | Nombre de usuario único |
| `password_hash` | VARCHAR | Contraseña hasheada con bcrypt |
| `email` | VARCHAR | Dirección de email (opcional) |
| `avatar_url` | VARCHAR | URL del avatar (opcional) |
| `github_id` | INTEGER | ID de GitHub (para OAuth futuro, opcional) |
| `created_at` | TIMESTAMP | Fecha de creación de cuenta |
| `last_login` | TIMESTAMP | Fecha del último login |

## Mejores Prácticas de Seguridad

### Para Administradores

1. **Use Contraseñas Fuertes**: Exija que los usuarios usen contraseñas fuertes y únicas
2. **Limite el Acceso**: Cree cuentas solo para usuarios que necesitan acceso
3. **Monitoree el Acceso**: Revise regularmente la lista de usuarios con `auth_cli.py list`
4. **Proteja el SECRET_KEY**: Mantenga la clave secreta segura y nunca la confíe al control de versiones
5. **HTTPS en Producción**: Siempre use HTTPS en entornos de producción

### Para Usuarios

1. **Contraseñas Fuertes**: Use contraseñas fuertes y únicas
2. **Cerrar Sesión**: Cierre sesión al usar computadoras compartidas
3. **Proteja sus Credenciales**: Nunca comparta su contraseña

## Solución de Problemas

### No Puedo Iniciar Sesión

**Problema**: Mensaje "Authentication failed"

**Soluciones**:
1. Verifique la ortografía del nombre de usuario (distingue mayúsculas/minúsculas)
2. Verifique que la contraseña sea correcta
3. Intente restablecer la contraseña: `uv run python auth_cli.py passwd <username>`

### Olvidé la Contraseña

**Problema**: Usuario olvidó la contraseña

**Solución**: El administrador puede restablecerla:

```bash
uv run python auth_cli.py passwd <username>
```

### Sesión Expirada

**Problema**: "Session expired" o desconectado repentinamente

**Solución**: 
- Inicie sesión nuevamente
- Las sesiones expiran después de 30 días por defecto
- Verifique si `SECRET_KEY` fue cambiado

## Referencia de Comandos CLI

| Comando | Descripción | Ejemplo |
|---------|-------------|---------|
| `create-admin` | Crear un nuevo usuario | `uv run python auth_cli.py create-admin` |
| `list` | Listar todos los usuarios | `uv run python auth_cli.py list` |
| `passwd <username>` | Cambiar contraseña | `uv run python auth_cli.py passwd admin` |
| `delete <username>` | Eliminar usuario | `uv run python auth_cli.py delete user1` |

## Próximos Pasos

- [Guía de Inicio Rápido](quickstart.md)
- [Configuración](configuration.md)
- [Interfaz de Chat](../user-guide/chat.md)
