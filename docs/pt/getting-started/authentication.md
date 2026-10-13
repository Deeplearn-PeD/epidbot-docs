# Autenticação

EpidBot utiliza autenticação por usuário/senha com gerenciamento seguro de sessões.

## Visão Geral

EpidBot requer autenticação para acessar a aplicação. Isso garante:
- Isolamento de usuários (cada usuário tem suas próprias sessões e dados)
- Controle de acesso seguro
- Persistência de sessões

## Sistema de Autenticação

### Recursos de Segurança

- **Hash de Senhas**: Todas as senhas são hasheadas usando bcrypt com geração automática de salt
- **Tokens JWT**: access tokens de curta duração (HS256, ~30 minutos) renovados automaticamente por refresh tokens de ~7 dias, revogáveis e armazenados no PostgreSQL
- **Chaves de API**: acesso programático opcional via `X-API-Key`
- **OAuth**: login social com provedores configuráveis

### Gerenciamento de Usuários

Usuários são gerenciados através da ferramenta de linha de comando `auth_cli.py`.

## Criando Usuários

### Configuração Inicial

Antes de iniciar EpidBot pela primeira vez, crie um usuário administrador:

```bash
uv run python auth_cli.py create-admin
```

Você será solicitado a fornecer:
- Nome de usuário (obrigatório)
- Senha (obrigatório, não será exibida)
- Confirmação de senha
- Email (opcional)

Exemplo:

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

### Criando Usuários Adicionais

Use o mesmo comando `create-admin` para criar usuários adicionais:

```bash
uv run python auth_cli.py create-admin
```

## Gerenciando Usuários

### Listar Todos os Usuários

```bash
uv run python auth_cli.py list
```

Saída:

```
=== User List ===

ID    Username             Email                          Created              Last Login          
-----------------------------------------------------------------------------------------------
1     admin                admin@example.com              2026-03-10           2026-03-10          
2     researcher           researcher@institute.org       2026-03-11           Never               

Total: 2 user(s)
```

### Alterar Senha

```bash
uv run python auth_cli.py passwd <username>
```

Exemplo:

```bash
$ uv run python auth_cli.py passwd admin
=== Change Password for 'admin' ===
New password: 
Confirm new password: 

✓ Password changed successfully for user 'admin'
```

### Excluir Usuário

```bash
uv run python auth_cli.py delete <username>
```

Exemplo:

```bash
$ uv run python auth_cli.py delete researcher
=== Delete User 'researcher' ===
⚠️  WARNING: This action cannot be undone!
Type 'yes' to confirm deletion of user 'researcher': yes

⚠️  Note: This will delete the user but preserve their chat history.
    To fully implement user deletion with cascade, database-level constraints need to be updated.

✓ User 'researcher' marked for deletion (manual database cleanup may be required)
```

## Fazendo Login

### Via Interface Web

1. Inicie EpidBot: `cd frontend && pnpm dev`
2. Abra o navegador em `http://localhost:3000`
3. Digite seu nome de usuário e senha
4. Clique em "Login"

### Gerenciamento de Sessão

- Sessões duram 30 dias por padrão
- Você permanecerá logado até a sessão expirar ou limpar os cookies do navegador
- Cada usuário tem sessões isoladas e histórico de chat próprio

## Variáveis de Ambiente

### SECRET_KEY

A variável de ambiente `SECRET_KEY` é usada para criptografar tokens de sessão:

```bash
export SECRET_KEY=sua-chave-secreta-aleatoria-aqui
```

**Importante**: 
- Gere uma chave aleatória segura para produção
- Mantenha esta chave em segredo
- Se você a alterar, todas as sessões existentes serão invalidadas

Para gerar uma chave segura:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Adicione ao seu arquivo `.env`:

```env
SECRET_KEY=chave-gerada-aqui
```

## Esquema do Banco de Dados

Usuários são armazenados na tabela `users` no **PostgreSQL** (`epidbot/db/schema.py`):

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | INTEGER | ID do usuário (auto-gerado) |
| `username` | VARCHAR | Nome de usuário único |
| `password_hash` | VARCHAR | Senha hasheada com bcrypt |
| `email` | VARCHAR | Endereço de email (opcional) |
| `avatar_url` | VARCHAR | URL do avatar (opcional) |
| `github_id` | INTEGER | ID do GitHub (para OAuth futuro, opcional) |
| `created_at` | TIMESTAMP | Data de criação da conta |
| `last_login` | TIMESTAMP | Data do último login |

## Melhores Práticas de Segurança

### Para Administradores

1. **Use Senhas Fortes**: Exija que usuários usem senhas fortes e únicas
2. **Limite o Acesso**: Crie contas apenas para usuários que precisam de acesso
3. **Monitore o Acesso**: Revise regularmente a lista de usuários com `auth_cli.py list`
4. **Proteja o SECRET_KEY**: Mantenha a chave secreta segura e nunca a commite no controle de versão
5. **HTTPS em Produção**: Sempre use HTTPS em ambientes de produção

### Para Usuários

1. **Senhas Fortes**: Use senhas fortes e únicas
2. **Logout**: Faça logout ao usar computadores compartilhados
3. **Proteja suas Credenciais**: Nunca compartilhe sua senha

## Solução de Problemas

### Não Consigo Fazer Login

**Problema**: Mensagem "Authentication failed"

**Soluções**:
1. Verifique a ortografia do nome de usuário (diferencia maiúsculas/minúsculas)
2. Verifique se a senha está correta
3. Tente redefinir a senha: `uv run python auth_cli.py passwd <username>`

### Esqueci a Senha

**Problema**: Usuário esqueceu a senha

**Solução**: O administrador pode redefini-la:

```bash
uv run python auth_cli.py passwd <username>
```

### Sessão Expirou

**Problema**: "Session expired" ou desconectado repentinamente

**Solução**: 
- Faça login novamente
- Refresh tokens expiram após ~7 dias
- Se todos foram desconectados de uma vez, `SECRET_KEY`/`JWT_SECRET` provavelmente foi alterada

## Referência de Comandos CLI

| Comando | Descrição | Exemplo |
|---------|-----------|---------|
| `create-admin` | Criar um novo usuário | `uv run python auth_cli.py create-admin` |
| `list` | Listar todos os usuários | `uv run python auth_cli.py list` |
| `passwd <username>` | Alterar senha | `uv run python auth_cli.py passwd admin` |
| `delete <username>` | Excluir usuário | `uv run python auth_cli.py delete user1` |

## Próximos Passos

- [Guia de Início Rápido](quickstart.md)
- [Configuração](configuration.md)
- [Interface de Chat](../user-guide/chat.md)
