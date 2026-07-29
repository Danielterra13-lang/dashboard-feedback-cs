# Dashboard Standalone com Google Sheets como Fonte de Dados
850a10b9-48b5-4e60-9a32-bcf1c716b170.png

Template de dashboard operacional em HTML puro (sem backend, sem build step) que lê dados
diretamente de uma planilha do Google Sheets e se atualiza sozinho. Construído originalmente
para um painel de Customer Success (fila de atendimento por urgência, sentimento, risco de
churn), mas a arquitetura é genérica — qualquer planilha com dados tabulares pode alimentar
esse layout trocando duas variáveis.

Contexto completo e decisões de arquitetura: [link do case study no Notion — adicionar depois de publicar]

Fluxo de automação que categoriza os feedbacks (Make.com), alimentando a planilha:
https://us2.make.com/public/shared-scenario/tnGDXksyFPG/fluxo-de-categorizacao-feedback

## Como funciona

- Um único arquivo `dashboard.html` — abre direto no navegador, sem servidor.
- Busca os dados via [Google Visualization API (`gviz/tq`)](https://developers.google.com/chart/interactive/docs/querylanguage),
  usando JSONP — não precisa de chave de API nem de backend intermediário.
- Atualiza sozinho a cada 30 segundos (`setInterval`).
- Nomes de coluna são normalizados automaticamente (acentos, maiúsculas, espaços) e a coluna
  de data é detectada dinamicamente entre variações comuns de nome ("Timestamp", "Carimbo de
  data/hora") — não depende de a planilha usar um nome de coluna específico.

## Uso

Abra `dashboard.html` no navegador. Por padrão, ele aponta para uma planilha de demonstração
com dados fictícios. Para usar sua própria planilha:

```
dashboard.html?sheetId=SEU_ID_DA_PLANILHA&gid=SEU_GID_DA_ABA
```

- `sheetId`: parte da URL da planilha entre `/d/` e `/edit`
- `gid`: identificador da aba específica (aparece na URL após `gid=`)

A planilha precisa estar compartilhada como "Qualquer pessoa com o link pode visualizar".

### Colunas esperadas

O dashboard foi construído para uma estrutura de feedback de Customer Success, mas a lógica de
leitura é genérica. Colunas usadas neste caso: `Timestamp`, `Nome`, `Empresa`, `Tipo de
Solicitação`, `Sentimento`, `Urgência`, `Risco de Churn`, `Score de Saúde`, `Status`.

## Limitações conhecidas

- Sem paginação — funciona bem até a ordem de milhares de linhas; acima disso, o `gviz/tq`
  fica lento e vale migrar pra uma fonte com paginação real (ex: API própria ou BigQuery).
- Classificação de sentimento/urgência/risco acontece fora deste dashboard (no fluxo do Make,
  usando um LLM) — o dashboard só exibe e agrega o que já chega categorizado.
- Não há autenticação — qualquer pessoa com o link do `dashboard.html` e os parâmetros da
  planilha consegue visualizar os dados, então isso só é adequado para planilhas que já podem
  ser públicas ou para uso atrás de uma rede interna.
