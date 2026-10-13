# Início Rápido

Comece a usar o EpidBot em 5 minutos.

## Pré-requisitos

Antes de iniciar, certifique-se de:
- Ter instalado as dependências: `uv sync`
- Ter criado pelo menos uma conta de usuário (veja [Autenticação](authentication.md))

## Iniciando o EpidBot

### 1. Criar Primeiro Usuário (Obrigatório)

Se você ainda não criou um usuário:

```bash
uv run python auth_cli.py create-admin
```

Siga as instruções para criar um nome de usuário e senha.

### 2. Iniciar a Aplicação

```bash
cd epidbot
cd frontend && pnpm dev
```

### 3. Fazer Login

Abra seu navegador em `http://localhost:3000`.

Você verá uma tela de login. Digite seu nome de usuário e senha para acessar o EpidBot.

## Uso Básico


![Tela Principal do EpidBot](img/tela_inicial.png)

### 1. Baixar Dados

Peça ao EpidBot para baixar dados de saúde:

```
Baixe os dados de Dengue para 2023
```

Ou em inglês:

```
Download Dengue data for 2023
```

### 2. Explorar Dados Disponíveis

```
Liste todos os dados disponíveis
```

```
What data is available in the catalog?
```

### 3. Consultar Dados

```
Quantos casos de dengue foram reportados em 2023?
```

```
Show me the temporal trend of dengue cases in 2023
```

### 4. Gerar Visualizações

```
Gere um gráfico da distribuição de casos de dengue por estado
```

```
Create a map showing dengue cases by municipality
```

### 5. Exportar Resultados

```
Exporte os resultados para CSV
```

## Exemplos de Consultas

### Descoberta de Dados

| Consulta | Descrição |
|----------|-----------|
| "Quais bancos de dados estão disponíveis?" | Listar bancos de dados PySUS |
| "Liste os datasets baixados" | Mostrar dados em cache |
| "Descreva o schema do dataset de dengue" | Mostrar colunas e tipos |

### Análise

| Consulta | Descrição |
|----------|-----------|
| "Calcule a idade média dos pacientes" | Análise estatística |
| "Compare casos de dengue 2022 vs 2023" | Comparação temporal |
| "Top 10 municípios por casos" | Consultas de ranking |

### Visualização

| Consulta | Descrição |
|----------|-----------|
| "Plote série temporal de casos" | Gráfico de linha |
| "Gráfico de barras de casos por estado" | Gráfico de barras |
| "Mapa de casos por município" | Mapa coroplético |

## Base de Conhecimento

### Pesquisar Documentação

```
Pesquise na base de conhecimento por metadados SINAN
```

### Adicionar Fontes

```
Adicione esta documentação à base de conhecimento: https://portalsinan.saude.gov.br/
```

## Dicas

1. **Seja específico**: Inclua anos, doenças e localizações
2. **Solicite visualizações**: Peça gráficos ao explorar dados
3. **Verifique o raciocínio**: Expanda "Pensamento do Agente" para ver chamadas de ferramentas
4. **Use sessões**: Crie sessões separadas para análises diferentes

## Próximos Passos

- [Interface de Chat](../user-guide/chat.md)
- [Análise de Dados](../user-guide/analysis.md)
- [Base de Conhecimento](../user-guide/knowledge-base.md)
