# Interface de Chat

A aba Chat é a interface principal para interagir com o EpidBot.

## Elementos da Interface

### Janela de Chat

Exibe o histórico da conversa com:

- **Mensagens do usuário** (alinhadas à direita)
- **Respostas do assistente** (alinhadas à esquerda)
- **Imagens** (plots e visualizações)

### Pensamento do Agente

Seção expansível mostrando:

- Chamadas de ferramentas sendo executadas
- Resultados intermediários
- Processo de raciocínio

!!! tip "Atualizações em Tempo Real"
    A caixa de pensamento atualiza em tempo real conforme o agente processa sua solicitação.

### Caixa de Entrada

Digite sua pergunta e pressione Enter ou clique em **Enviar**.

## Exemplos de Consultas

### Baixar Dados

```
Baixe os dados de Dengue para 2023
```

```
Download Zika data for 2022 and 2023
```

### Consultar Dados

```
Quantos casos de dengue foram reportados em São Paulo em 2023?
```

```
Show the distribution of dengue cases by age group
```

### Gerar Gráficos

```
Gere um gráfico de linha mostrando a tendência temporal
```

```
Create a bar chart of cases by state
```

### Mapas

```
Mostre um mapa dos casos de dengue por município
```

```
Generate a choropleth map of Brazil showing disease distribution
```

## Recursos de Resposta

### Renderização Markdown

As respostas suportam:

- Texto **negrito** e *itálico*
- Listas (numeradas e com marcadores)
- Blocos de código
- Tabelas

### Exibição de Imagens

Plots são exibidos automaticamente inline:

```
![Descrição](/file=/path/to/plot.png)
```

### Resultados de Ferramentas

Quando ferramentas são chamadas, você verá:

```
🔧 Calling: download_health_data
   Args: {"disease": "DENG", "year": 2023}

✓ Result: Downloaded 1,234,567 records...
```

## Atalhos de Teclado

| Tecla | Ação |
|-------|------|
| Enter | Enviar mensagem |
| Shift+Enter | Nova linha |

## Melhores Práticas

1. **Comece com descoberta de dados**: Verifique o que está disponível antes de consultas complexas
2. **Seja específico sobre anos**: Sempre especifique o(s) ano(s) de interesse
3. **Solicite visualizações**: O agente funciona melhor quando solicitado a criar gráficos
4. **Revise o pensamento**: Verifique a seção Pensamento do Agente para entender o que aconteceu
5. **Use português ou inglês**: Ambos os idiomas são suportados

## Tratamento de Erros

Se ocorrer um erro:

- Uma mensagem amigável é exibida
- Detalhes técnicos são registrados
- Problemas comuns (limites de API) incluem sugestões de nova tentativa

Exemplo de erro:

```
A cota da API do modelo de IA foi excedida. 
Por favor, tente novamente mais tarde ou contate o administrador.
```
