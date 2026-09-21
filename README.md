# CineData Analytics — Pipeline de Dados End-to-End

Pipeline em Arquitetura Medalhão no Databricks (PySpark + SQL + Delta Lake) sobre uma base combinada TMDB/IMDb,
com Star Schema para BI e uma tabela de contexto para o assistente de IA (RAG). Projeto do RocketLab 2026.2 (Visagio).

## Estrutura

```
├── notebooks/
│   ├── Landing_to_Bronze.ipynb   # CSVs + API PTAX -> bronze (Delta, append)
│   ├── Bronze_to_Silver.ipynb    # limpeza, tipagem, deduplicação -> silver
│   └── Silver_to_Gold.ipynb      # Star Schema, gold_genai_movies_context, desafio de analytics
├── job.yaml                      # exportação do Databricks Workflow
├── img/                          # prints da execução e do schedule
└── README.md
```

## Como executar

1. Suba os 5 CSVs em um Volume, ex.: `workspace.landing.inputs`.
2. Importe os 3 notebooks no Workspace.
3. Crie o Job com as tarefas `to_Bronze → to_Silver → to_Gold` (ou importe o `job.yaml`, ajustando caminhos).
4. Rode. Parâmetros: `catalogo` (padrão `workspace`), `caminho_volume`, `data_inicio`/`data_fim` (MM-DD-AAAA;
   vazios = últimos 7 dias).

## Decisões principais

- **Bronze** sem inferência de schema (tudo `STRING`), para não perder dado sujo na leitura.
- **Conversões seguras** (`try_cast`) em vez de `cast`: texto em coluna numérica vira `NULL` sem quebrar o job.
- **Lucro** fica `NULL` quando falta orçamento ou receita (tratar ausência como zero inventaria resultado).
- **Câmbio**: série contínua com forward fill; conversão usa a cotação mais recente.
- **Surrogate keys** por hash SHA-256 — determinísticas, não mudam a cada reprocessamento.
- **Fato** inclui só filmes com `status_filme = 'Lançado'` (flag `FATO_APENAS_LANCADOS`).
- **Tabela de contexto da IA**: `coalesce` com fallback em cada campo antes do `concat`, para nenhum filme
  sumir por causa de um campo nulo. O notebook valida isso ao final.
