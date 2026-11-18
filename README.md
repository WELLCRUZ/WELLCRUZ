# 🥇 Pipeline Medallion no Azure Databricks (Bronze → Silver → Gold)

Pipeline de dados ETL/ELT construído no **Azure Databricks**, seguindo a **Arquitetura Medallion**, com governança via **Unity Catalog** e armazenamento em **Delta Lake**. O projeto leva dados brutos até uma camada analítica pronta para consumo em BI.

![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=flat&logo=databricks&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta_Lake-00ADD4?style=flat&logo=delta&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-336791?style=flat&logo=postgresql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat&logo=powerbi&logoColor=black)

> ⚠️ Projeto de estudo e portfólio. Todos os dados são **fictícios ou públicos**, sem qualquer informação de empresas reais.

---

## 🎯 Problema

Dados operacionais chegam de forma bruta, com inconsistências (duplicidades, nulos, tipos incorretos e formatos diferentes), o que dificulta a criação de indicadores confiáveis. O objetivo é organizar esse fluxo em camadas, de modo que cada etapa aumente a qualidade e a confiabilidade dos dados.

## 🏗️ Arquitetura

```
 Fonte (CSV/JSON)
        │
        ▼
 ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
 │    BRONZE    │ ──▶ │    SILVER    │ ──▶ │     GOLD     │ ──▶  Power BI
 │ Dado bruto   │     │ Limpo e      │     │ Agregado e   │
 │ (como chegou)│     │ padronizado  │     │ regras de    │
 │              │     │              │     │ negócio      │
 └──────────────┘     └──────────────┘     └──────────────┘
        Delta Lake  ·  Unity Catalog  ·  Azure
```

| Camada | Objetivo | O que acontece |
|---|---|---|
| **Bronze** | Preservar o dado original | Ingestão sem alteração, com data de carga |
| **Silver** | Qualidade e padronização | Remoção de duplicados, tratamento de nulos, ajuste de tipos |
| **Gold** | Valor para o negócio | Agregações, CTEs e regras de negócio para indicadores |

## 🧰 Tecnologias e conceitos aplicados

- **Azure Databricks** como plataforma de processamento
- **Unity Catalog:** catálogos, schemas, **External Locations** e volumes (externos e gerenciados)
- **Delta Lake:** tabelas Delta criadas via SQL com External Location
- **Arquitetura Medallion** com validação e dependências entre tasks
- **Jobs e Pipelines:** Schedules & Triggers, notificações e gerenciamento de falhas
- **SQL:** CTEs, criação de tabelas, filtros e transformações

## 📁 Estrutura do repositório

```
pipeline-medallion-databricks/
├── README.md
├── notebooks/
│   ├── 01_bronze_ingestao.sql
│   ├── 02_silver_limpeza.sql
│   └── 03_gold_indicadores.sql
├── dados/
│   └── amostra.csv
├── docs/
│   ├── arquitetura.png
│   └── dicionario_de_dados.md
└── imagens/
    └── (prints do job e do dashboard)
```

## 🔄 Passo a passo

1. **Estruturação do ambiente:** criação de container, catálogo, schemas e volumes no Unity Catalog
2. **External Location:** configuração do acesso ao armazenamento no Azure
3. **Bronze:** ingestão dos arquivos de origem em tabelas Delta, sem transformação
4. **Silver:** limpeza, deduplicação, padronização de tipos e validações de qualidade
5. **Gold:** construção de tabelas analíticas com CTEs e regras de negócio
6. **Orquestração:** criação de Job com tasks dependentes (Bronze → Silver → Gold), agendamento (Schedule/Trigger) e notificação em caso de falha
7. **Consumo:** conexão da camada Gold ao Power BI para visualização

## 💻 Exemplo de código (camada Silver)

> Ajuste os nomes de catálogo, schema e colunas para os do seu projeto.

```sql
CREATE OR REPLACE TABLE meu_catalogo.silver.vendas AS
WITH base AS (
  SELECT
    CAST(id_pedido AS BIGINT)        AS id_pedido,
    CAST(data_pedido AS DATE)        AS data_pedido,
    UPPER(TRIM(categoria))           AS categoria,
    CAST(valor AS DECIMAL(10,2))     AS valor,
    ROW_NUMBER() OVER (
      PARTITION BY id_pedido
      ORDER BY data_carga DESC
    ) AS rn
  FROM meu_catalogo.bronze.vendas
  WHERE id_pedido IS NOT NULL
)
SELECT id_pedido, data_pedido, categoria, valor
FROM base
WHERE rn = 1;
```

## 📊 Resultado

- Camadas organizadas e rastreáveis, com separação clara entre dado bruto, tratado e analítico
- Execução automatizada por Job, com agendamento e notificação de falhas
- Camada Gold pronta para consumo em ferramentas de BI

> 📸 Adicione aqui os prints do Job executado com sucesso e do dashboard no Power BI.

## 📚 Aprendizados

- Separar o dado em camadas facilita auditoria, reprocessamento e correção de erros
- Definir dependências entre tasks evita que um dado inconsistente chegue ao consumo
- Governar acesso e armazenamento no Unity Catalog desde o início simplifica a manutenção

## 🔜 Próximos passos

- [ ] Incluir testes automatizados de qualidade de dados
- [ ] Adicionar carga incremental na camada Silver
- [ ] Documentar o dicionário de dados completo
- [ ] Publicar o dashboard final no Power BI

---

## 👤 Autor

**Wellington Cruz**
Analista de Dados | BI | Negócios | Requisitos

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/wellingtonsscruz)
[![E-mail](https://img.shields.io/badge/E--mail-EA4335?style=flat&logo=gmail&logoColor=white)](mailto:wellingtonsscruz@gmail.com)
