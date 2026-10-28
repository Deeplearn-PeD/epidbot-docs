# Plugin QGIS

O **plugin QGIS do EpidBot** traz o assistente para dentro do [QGIS](https://qgis.org/), permitindo fazer perguntas epidemiológicas, gerar gráficos e carregar resultados geoespaciais no seu mapa sem sair do fluxo de trabalho em SIG.

O plugin conversa com a sua conta EpidBot pela mesma API REST da interface web, então sessões, gráficos, cotas e base de conhecimento permanecem sincronizados. É open source em [github.com/Deeplearn-PeD/epidbot-qgis-plugin](https://github.com/Deeplearn-PeD/epidbot-qgis-plugin) sob licença GPL-2.0-or-later.

## Requisitos

- **QGIS 3.28** ou mais recente (Windows, macOS ou Linux)
- Uma conta EpidBot nos planos **Max ou Equipe** — as chaves de API fazem parte dos dois planos
- Uma **chave de API** pessoal do EpidBot — crie em **Configurações > Chaves de API** na interface web (veja [REST API](../api/rest.md#authentication))
- Acesso de rede ao servidor do EpidBot (`https://api.epidbot.kwar-ai.com.br` por padrão)

## Instalação

### Pelo repositório oficial de plugins do QGIS (recomendado)

1. No QGIS, abra **Complementos > Gerenciar e Instalar Complementos**
2. Busque por **EpidBot** (ID `6764`) ou abra a [página do plugin](https://plugins.qgis.org/plugins/epidbot/)
3. Clique em **Instalar complemento**
4. Ative o painel **EpiDBot Chat** em **Ver > Painéis** ou clique no botão da barra de ferramentas

Instalado assim, o plugin se atualiza pelo próprio gerenciador de complementos do QGIS.

### A partir de um arquivo ZIP

Use apenas quando precisar de uma versão mais nova que a publicada no repositório:

1. Baixe o [epidbot.zip](https://github.com/Deeplearn-PeD/epidbot-qgis-plugin/releases/latest/download/epidbot.zip) mais recente
2. No QGIS, abra **Complementos > Gerenciar e Instalar Complementos > Instalar a partir de ZIP**
3. Selecione o arquivo e clique em **Instalar complemento**

## Configuração

Clique no botão **⚙ (engrenagem)** do painel de chat:

| Configuração | Significado |
|---|---|
| **URL do servidor** | URL base da API do EpidBot — preenchida com `https://api.epidbot.kwar-ai.com.br` |
| **Chave de API** | Sua chave pessoal, enviada no cabeçalho `X-API-Key` em cada requisição. Guardada no keyring do sistema quando o pacote opcional `keyring` está disponível |
| **Idioma das respostas** | Idioma das respostas do assistente: `en` ou `pt-BR` |
| **Intervalo de consulta** | Segundos entre consultas de status do job (padrão 1,5 s) |

## Conversando pelo QGIS

O painel **EpiDBot Chat**, ancorável, funciona como o chat web:

- As respostas trazem Markdown, tabelas e miniaturas de gráficos
- Escolha uma **sessão** existente, inicie outra ou deixe o servidor criar uma; ao trocar de sessão o histórico é recarregado, incluindo gráficos
- Exporte a conversa para um arquivo Markdown
- A interface do painel existe em **inglês e português do Brasil**, e as requisições rodam em thread de segundo plano para o QGIS continuar responsivo

## Integração com o mapa

- **Injeção de contexto do mapa** — cada prompt carrega automaticamente as camadas ativas (com nomes de campos, valores de amostra e seleções), a extensão do canvas em EPSG:4326 e o CRS do projeto, para o assistente saber o que está na sua tela
- **Prompts espaciais** — ações de um clique perguntam sobre a **extensão atual** ou as **feições selecionadas**
- **Adicionar ao mapa** — blocos GeoJSON e URLs de conjuntos de dados nas respostas viram camadas estilizadas no canvas com um clique

Exemplos de prompts:

> Mapeie a incidência de dengue dos municípios carregados no meu projeto

> Qual é a população total dos municípios selecionados?

## Trabalhando com gráficos

Todo gráfico gerado no plugin pode ser:

- **Salvo em disco** como imagem
- **Aberto** no visualizador de imagens do sistema
- **Inserido em um layout de impressão** — adicionado ao layout *"EpidBot Plots"* para composições de mapa

## Solução de problemas

| Sintoma | Solução |
|---|---|
| `Unauthorized` no painel | A chave de API está ausente ou inválida. Clique em **⚙**, cole uma chave criada em **Configurações > Chaves de API** |
| Não consegue acessar o servidor | Verifique a **URL do servidor** nas configurações do plugin e a configuração de proxy. Falhas de rede transitórias são repetidas automaticamente |
| O painel não aparece | Ative em **Ver > Painéis > EpiDBot Chat**, ou confirme em **Complementos > Gerenciar e Instalar Complementos** que o plugin está habilitado |
