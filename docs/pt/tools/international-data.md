# Ferramentas de Dados Internacionais

Ferramentas para baixar dados epidemiológicos e de saúde de fontes internacionais.

## Visão Geral

O EpidBot suporta múltiplas fontes de dados internacionais além do DATASUS brasileiro:

| Fonte | Descrição | Chave API Necessária |
|-------|-----------|---------------------|
| **Mosqlimate** | Dados climáticos e epidemiológicos brasileiros | Sim |
| **OMS (WHO GHO)** | Observatório Global de Saúde da OMS | Não |
| **ECDC** | Centro Europeu de Prevenção e Controle de Doenças | Não |
| **Banco Mundial** | Indicadores globais de saúde e desenvolvimento | Não |

---

## Mosqlimate

O [Mosqlimate](https://mosqlimate.org) fornece dados epidemiológicos e climáticos brasileiros complementares, incluindo nowcasting do Infodengue e variáveis climáticas.

### Pré-requisitos

Configure sua chave API:
```bash
export MOSQLIMATE_API_KEY=sua-chave-api
```

Obtenha sua chave em: https://mosqlimate.org/profile/auth

### list_mosqlimate_sources

Lista todas as fontes de dados disponíveis no Mosqlimate.

```python
list_mosqlimate_sources(ctx: RunContext[None]) -> str
```

**Exemplo:**
```
Liste as fontes de dados disponíveis no Mosqlimate
```

**Fontes disponíveis:**

| Fonte | Descrição | Parâmetros Obrigatórios |
|-------|-----------|------------------------|
| `infodengue` | Dados epidemiológicos semanais | disease, start, end |
| `climate` | Dados climáticos diários | start, end |
| `climate_weekly` | Clima agregado semanalmente | start, end |
| `mosquito` | Dados de monitoramento ovitrampas | date_start, date_end |
| `episcanner` | Parâmetros epidêmicos | disease, uf |

### download_mosqlimate_infodengue

Baixa dados epidemiológicos do Infodengue (dengue, zika, chikungunya).

```python
download_mosqlimate_infodengue(
    ctx: RunContext[None],
    disease: str,        # "dengue", "zika", ou "chik"
    start: str,          # YYYY-MM-DD
    end: str,            # YYYY-MM-DD
    uf: Optional[str],   # Sigla do estado (ex: "RJ")
    geocode: Optional[int]  # Código IBGE do município
) -> str
```

**Exemplo:**
```
Baixe dados do Infodengue para dengue no RJ de 2023-01-01 a 2023-12-31
```

### download_mosqlimate_climate

Baixa dados climáticos diários do Copernicus ERA5.

```python
download_mosqlimate_climate(
    ctx: RunContext[None],
    start: str,
    end: str,
    uf: Optional[str],
    geocode: Optional[int]
) -> str
```

**Exemplo:**
```
Baixe dados climáticos para o estado de São Paulo de 2023-01-01 a 2023-12-31
```

### download_mosqlimate_episcanner

Baixa parâmetros epidêmicos do EpiScanner (R0, beta, gamma, etc).

```python
download_mosqlimate_episcanner(
    ctx: RunContext[None],
    disease: str,  # "dengue", "zika", "chik"
    uf: str,       # Sigla do estado
    year: Optional[int]
) -> str
```

---

## OMS (WHO Global Health Observatory)

Acesso a milhares de indicadores de saúde da Organização Mundial da Saúde.

### list_who_health_indicators

Lista indicadores disponíveis da OMS.

```python
list_who_health_indicators(
    ctx: RunContext[None],
    limit: int = 50
) -> str
```

### search_who_health_indicators

Busca indicadores por palavra-chave.

```python
search_who_health_indicators(
    ctx: RunContext[None],
    query: str,
    limit: int = 20
) -> str
```

**Exemplo:**
```
Busque indicadores da OMS para tuberculose
```

### download_who_health_data

Baixa dados de indicadores da OMS.

```python
download_who_health_data(
    ctx: RunContext[None],
    indicator_code: str,
    countries_json: Optional[str]  # '["BRA", "USA", "IND"]'
) -> str
```

---

## ECDC (Centro Europeu de CDC)

Acesso a dados europeus de vigilância de doenças infecciosas.

### list_ecdc_datasets_tool

Lista datasets ECDC disponíveis.

```python
list_ecdc_datasets_tool(ctx: RunContext[None]) -> str
```

### download_ecdc_data

Baixa dados de vigilância do ECDC.

```python
download_ecdc_data(
    ctx: RunContext[None],
    dataset_name: str,
    countries_json: Optional[str],  # '["DE", "FR", "IT"]'
    year: Optional[int]
) -> str
```

---

## Banco Mundial

Acesso a indicadores globais de saúde, população e desenvolvimento.

### search_wb_indicators_tool

Busca indicadores do Banco Mundial.

```python
search_wb_indicators_tool(
    ctx: RunContext[None],
    query: str,
    limit: int = 20
) -> str
```

### download_wb_indicator_data

Baixa dados de indicadores do Banco Mundial.

```python
download_wb_indicator_data(
    ctx: RunContext[None],
    indicator_code: str,
    countries_json: Optional[str],
    start_year: Optional[int],
    end_year: Optional[int]
) -> str
```

---

## Armazenamento de Dados

Todos os dados baixados são armazenados em formato parquet:

| Fonte | Caminho de Armazenamento |
|--------|--------------|
| Mosqlimate | `~/pysus/mosqlimate/` |
| OMS | `~/pysus/who/` |
| ECDC | `~/pysus/ecdc/` |
| Banco Mundial | `~/pysus/worldbank/` |

Use `list_available_datasets` para ver todos os arquivos baixados.
