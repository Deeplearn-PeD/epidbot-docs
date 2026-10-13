# Sessões

EpidBot suporta múltiplas sessões de chat com histórico persistente.

## Aba Sessões

### Sessão Ativa

- **Dropdown de Sessão**: Selecione entre sessões existentes
- **Nome da Sessão**: Visualize ou renomeie a sessão atual

### Ações de Sessão

| Botão | Ação |
|-------|------|
| Nova Sessão | Criar uma nova conversação |
| Renomear | Renomear a sessão atual |
| Excluir | Excluir a sessão atual |

### Confirmação de Exclusão

Excluir uma sessão requer confirmação:

1. Clique em **Excluir**
2. Confirme clicando em **Sim, Excluir**
3. Ou cancele com **Cancelar**

## Exportar Sessões

### Formatos de Exportação

| Formato | Extensão | Descrição |
|---------|----------|-----------|
| Markdown | `.md` | Legível por humanos, inclui imagens |
| JSON | `.json` | Legível por máquina, dados completos |
| HTML | `.html` | Página web autônoma |
| TXT | `.txt` | Texto simples |

### Passos para Exportar

1. Selecione o formato de exportação
2. Clique em **Exportar Sessão Atual**
3. Baixe o arquivo

## Buscar Sessões

Busque em todas as sessões:

1. Digite a consulta de busca
2. Opcionalmente filtre por:
   - Sessão
   - Intervalo de datas
3. Clique em **Buscar**
4. Clique em um resultado para ver a mensagem completa

## Gerenciamento de Sessões

### Comportamento Automático

- Uma sessão padrão é criada automaticamente
- Sessões são salvas no DuckDB (`chat_history.duckdb`)
- Mensagens incluem timestamps
- Imagens são armazenadas como BLOBs

### Melhores Práticas

1. **Crie sessões específicas por tópico**: Sessões separadas para análises diferentes
2. **Use nomes descritivos**: Nomeie sessões de acordo com o tópico de análise
3. **Exporte sessões importantes**: Salve conversas valiosas
4. **Limpe periodicamente**: Exclua sessões antigas e desnecessárias

## Armazenamento de Dados

Os dados da sessão são armazenados em:

```
chat_history.duckdb
├── sessions
├── messages
└── message_files
```

### Backup

Para fazer backup das sessões:

```bash
cp chat_history.duckdb chat_history_backup.duckdb
```

### Localização

Por padrão, o banco de dados está no diretório do projeto. Configure com:

```bash
export DB_PATH=/path/to/chat_history.duckdb
```
