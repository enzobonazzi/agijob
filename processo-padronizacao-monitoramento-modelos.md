# Processo de Padronização para Monitoramento de Modelos

**Documento técnico de referência** — Contrato de Dados, Engenharia de Dados e Business Intelligence

> Este documento consolida o novo processo padronizado para o monitoramento de modelos de crédito/risco, cobrindo desde a entrega da base analítica pelos Cientistas de Dados até a construção dos dashboards em Power BI. O objetivo é eliminar retrabalho, padronizar nomenclatura e tipos de dados, e permitir automação, escalabilidade e reaproveitamento de componentes entre modelos.

---

## Sumário

1. [Visão Geral do Processo](#1-visão-geral-do-processo)
2. [Contrato de Dados — Cientistas de Dados](#2-contrato-de-dados--cientistas-de-dados)
3. [Engenharia de Dados](#3-engenharia-de-dados)
4. [Bases Ouro — Schemas Finais](#4-bases-ouro--schemas-finais)
5. [Business Intelligence](#5-business-intelligence)
6. [Padronização Power BI](#6-padronização-power-bi)
7. [Resumo do Novo Processo e Ganhos](#7-resumo-do-novo-processo-e-ganhos)

---

## 1. Visão Geral do Processo

O processo padronizado é organizado em quatro etapas sequenciais, cada uma com responsabilidade e contrato bem definidos:

| Etapa | Responsável | Entrega |
|---|---|---|
| 1. Contrato de Dados | Cientistas de Dados | Base analítica padronizada (nomenclatura, tipos e colunas obrigatórias) |
| 2. Engenharia de Dados | Engenharia | Camadas **Bronze → Prata → Ouro** no DataLake |
| 3. Bases Ouro | Engenharia | Tabelas finais consumidas pelo BI (`performance`, `gr`, `variaveis`, `indicadores`) |
| 4. Business Intelligence | Analistas de BI | Dashboards padronizados via template e DAX reutilizável |

Essa separação de responsabilidades é o pilar do processo: o Power BI deixa de ser uma ferramenta de tratamento de dados e passa a atuar exclusivamente como camada de visualização, enquanto toda a lógica de transformação é centralizada na engenharia.

---

## 2. Contrato de Dados — Cientistas de Dados

### 2.1 Objetivo

Definir o padrão de entrega da base analítica pelos Cientistas de Dados para o processo de monitoramento de modelos. A base entregue deve:

- Estar limpa e padronizada
- Conter apenas as colunas necessárias
- Respeitar a nomenclatura obrigatória (baseada em prefixos)
- Possuir os tipos de dados corretos
- Estar pronta para processamento automatizado

### 2.2 Regras Gerais

- Apenas colunas efetivamente utilizadas no monitoramento devem existir na base
- A nomenclatura é obrigatória e baseada em prefixos padronizados
- Os tipos de dados devem seguir rigorosamente a especificação de cada prefixo
- Deve ser entregue uma única base, consolidando todas as colunas necessárias

### 2.3 Especificação de Colunas por Prefixo

A tabela abaixo resume o contrato completo. Os detalhes de cada prefixo são descritos nas subseções seguintes.

| Prefixo / Coluna | Tipo | Obrigatoriedade | Descrição resumida |
|---|---|---|---|
| `dt_*` | DATE | Obrigatório (≥ 1 coluna) | Datas utilizadas no monitoramento |
| `tipo_base` | STRING | Obrigatório | Origem da base (`DESENVOLVIMENTO`, `PRODUCAO`, `OOT`) |
| `id_*` | STRING | Obrigatório | Identificadores de entidades (não entram no monitoramento) |
| `f_*` | STRING | Opcional | Filtros e segmentações analíticas |
| `v_*` | STRING | Obrigatório (≥ 1 coluna) | Variáveis categorizadas/binadas (PSI, distribuição, variação de bad) |
| `score*` | FLOAT | Obrigatório (≥ 1 coluna) | Score do modelo |
| `gr_*` | STRING | Obrigatório (≥ 1 coluna) | Grupos de risco do modelo |
| `decil*` | STRING | Obrigatório (≥ 1 coluna) | Decis do score |
| `fl_bad*` | INT | Obrigatório (≥ 1 coluna) | Flags de inadimplência / evento ruim (define os MOBs) |

#### Prefixo `dt_*`

- **Tipo:** DATE
- **Obrigatoriedade:** obrigatório, pelo menos 1 coluna
- **Descrição:** datas utilizadas no monitoramento
- **Exemplos:** `dt_movimento DATE`, `dt_safra DATE`, `dt_referencia DATE`
- **Regra:** enviar apenas as datas efetivamente utilizadas

#### Coluna `tipo_base`

- **Tipo:** STRING
- **Obrigatoriedade:** obrigatório
- **Valores permitidos:** `'DESENVOLVIMENTO'`, `'PRODUCAO'`, `'OOT'`
- **Descrição:** identifica a origem da base
- **Exemplo:** `tipo_base STRING`

#### Prefixo `id_*`

- **Tipo:** STRING
- **Obrigatoriedade:** obrigatório
- **Descrição:** identificadores de entidades; não entram no monitoramento e permanecem apenas na base analítica
- **Exemplos:** `id_cpf_cliente STRING`, `id_nr_beneficio STRING`, `id_nr_contrato STRING`

#### Prefixo `f_*`

- **Tipo:** STRING
- **Obrigatoriedade:** opcional
- **Descrição:** filtros e segmentações analíticas
- **Exemplos:** `f_segmento STRING`, `f_canal STRING`, `f_regiao STRING`, `f_produto STRING`
- **Regra:** enviar apenas os filtros efetivamente utilizados no monitoramento

#### Prefixo `v_*`

- **Tipo:** STRING
- **Obrigatoriedade:** obrigatório, pelo menos 1 coluna
- **Descrição:** variáveis categorizadas/binadas, utilizadas para PSI e para distribuição/variação de bad
- **Exemplos:** `v_benef_sumarizado STRING`, `v_idade_woe_longo STRING`, `v_corte_rr_longo STRING`, `v_cd_sexo STRING`, `v_faixa_renda STRING`
- **Regras:** apenas variáveis categóricas; variáveis contínuas devem ser categorizadas previamente e não devem ser enviadas

#### Prefixo `score*`

- **Tipo:** FLOAT
- **Obrigatoriedade:** obrigatório, pelo menos 1 coluna
- **Descrição:** score do modelo
- **Exemplos:** `score FLOAT`, `score_v1 FLOAT`, `score_v2 FLOAT`
- **Regras:** valor entre 0 e 1, com 2 casas decimais; o primeiro score enviado é o utilizado para o cálculo de KS/GINI

#### Prefixo `gr_*`

- **Tipo:** STRING
- **Obrigatoriedade:** obrigatório, pelo menos 1 coluna
- **Descrição:** grupos de risco do modelo
- **Exemplos:** `gr_v1 STRING`, `gr_v2 STRING`, `gr_descritivo STRING`
- **Regra:** enviar todos os grupos monitorados

#### Prefixo `decil*`

- **Tipo:** STRING
- **Obrigatoriedade:** obrigatório, pelo menos 1 coluna
- **Descrição:** decis do score
- **Exemplos:** `decil STRING`, `decil_dev STRING`, `decil_prod STRING`
- **Regra:** valores entre 1 e 10

#### Prefixo `fl_bad*`

- **Tipo:** INT
- **Obrigatoriedade:** obrigatório, pelo menos 1 coluna
- **Descrição:** flags de inadimplência/evento ruim; definem os MOBs do monitoramento
- **Exemplos:** `fl_bad INT`, `fl_bad_over15m2 INT`, `fl_bad_over30m3 INT`, `fl_bad_over30m4 INT`, `fl_bad_over30m6 INT`, `fl_bad_over30m12 INT`
- **Regras:** valores permitidos são apenas 0 ou 1; múltiplas colunas geram automaticamente a dimensão MOB, enquanto uma única coluna gera um MOB fixo

### 2.4 Exemplo de Schema (DDL)

```sql
CREATE TABLE base_analitica_monitoramento_xpto (

    dt_movimento DATE,
    dt_atualizacao DATE,

    tipo_base STRING,

    id_nr_cpf_cliente STRING,
    id_nr_beneficio STRING,

    f_segmento STRING,
    f_canal STRING,

    v_benef_sumarizado STRING,
    v_idade_woe_longo STRING,
    v_corte_rr_longo STRING,
    v_cd_sexo STRING,

    score FLOAT,

    gr_v1 STRING,
    gr_v2 STRING,

    decil STRING,

    fl_bad_over15m2 INT,
    fl_bad_over30m3 INT,
    fl_bad_over30m6 INT,
    fl_bad_over30m12 INT
);
```

### 2.5 Checklists

#### Checklist — Itens Obrigatórios

- [ ] Pelo menos 1 coluna `dt_*` (DATE)
- [ ] Coluna `tipo_base` com valores `DESENVOLVIMENTO`, `PRODUCAO` ou `OOT`
- [ ] Pelo menos 1 coluna `v_*` (STRING, categorizada)
- [ ] Pelo menos 1 coluna `gr_*` (STRING)
- [ ] Pelo menos 1 coluna `score*` (FLOAT, entre 0 e 1, 2 casas decimais)
- [ ] Pelo menos 1 coluna `decil*` (STRING, entre 1 e 10)
- [ ] Pelo menos 1 coluna `fl_bad*` (INT, valores 0 ou 1)

#### Checklist — Qualidade

- [ ] Apenas as colunas necessárias estão presentes
- [ ] Variáveis `v_*` estão categorizadas
- [ ] Filtros `f_*` são efetivamente utilizados no monitoramento
- [ ] Tipos de dados estão corretos
- [ ] Prefixos seguem o padrão definido

#### Checklist — Validação Técnica

- [ ] `tipo_base`: apenas `DESENVOLVIMENTO`, `PRODUCAO` ou `OOT`
- [ ] `fl_bad*`: apenas 0 ou 1
- [ ] `decil*`: valores entre 1 e 10
- [ ] `score*`: entre 0 e 1, com 2 casas decimais
- [ ] `dt_*`: formato DATE

### 2.6 Exemplo de Metadata (JSON)

```json
{
  "metadata": {
    "nome_modelo": "Modelo Crédito Consignado INSS",
    "tipo_modelo": "Classificação Binária - Credit Scoring",
    "responsavel": "João Silva - Cientista de Dados",
    "data_criacao": "2025-01-20"
  },
  "schema": {
    "dt": {
      "obrigatorio": true,
      "tipo": "DATE",
      "colunas": ["dt_movimento", "dt_safra"]
    },
    "tipo_base": {
      "obrigatorio": true,
      "tipo": "STRING",
      "valores_permitidos": ["DESENVOLVIMENTO", "PRODUCAO", "OOT"]
    },
    "id": {
      "obrigatorio": false,
      "tipo": "STRING",
      "colunas": ["id_nr_cpf_cliente", "id_nr_beneficio"]
    },
    "f": {
      "obrigatorio": false,
      "tipo": "STRING",
      "colunas": ["f_segmento", "f_canal"]
    },
    "v": {
      "obrigatorio": true,
      "tipo": "STRING",
      "colunas": ["v_benef_sumarizado", "v_idade_woe_longo", "v_corte_rr_longo", "v_cd_sexo"]
    },
    "score": {
      "obrigatorio": true,
      "tipo": "FLOAT",
      "colunas": ["score"],
      "range": "0 a 1",
      "decimais": 2
    },
    "gr": {
      "obrigatorio": true,
      "tipo": "STRING",
      "colunas": ["gr_v1", "gr_v2"]
    },
    "decil": {
      "obrigatorio": true,
      "tipo": "STRING",
      "colunas": ["decil"],
      "range": "1 a 10"
    },
    "fl_bad": {
      "obrigatorio": true,
      "tipo": "INT",
      "colunas": ["fl_bad_over15m2", "fl_bad_over30m3", "fl_bad_over30m4", "fl_bad_over30m6", "fl_bad_over30m12"],
      "valores_permitidos": [0, 1]
    }
  }
}
```

### 2.7 Observação Final

A qualidade do monitoramento depende diretamente da qualidade da base analítica entregue. O cumprimento deste contrato garante:

- Processamento automatizado
- Padronização
- Redução de erros
- Compatibilidade com dashboards
- Geração correta de indicadores

---

## 3. Engenharia de Dados

A engenharia de dados processa a base analítica entregue pelos Cientistas de Dados através de três camadas — **Bronze**, **Prata** e **Ouro** — até chegar às tabelas finais consumidas pelo Power BI.

### 3.1 Camada Bronze — `df_analitica`

Padroniza tipos, nomenclatura e normaliza o score para a escala de 0 a 1, independentemente da escala original entregue pelo modelo.

```sql
df_analitica = spark.sql(
"""
SELECT
	"cpinssbehaviorchurnv4" AS monitoramento,
	
	CAST(
        CASE 
            WHEN periodo = 'PRD' THEN 'PRODUCAO'
            WHEN periodo = 'DEV' THEN 'DESENVOLVIMENTO'
            ELSE periodo 
        END AS STRING
    ) AS tipo_base,
	
	CAST(nr_cpf_cnpj AS STRING) AS id_nr_cpf_cnpj,
    CAST(dt_colunas_datas AS DATE) AS dt_colunas_datas,
	CAST(colunas_filtros AS STRING) AS f_colunas_filtros,
	CAST(coluna_variavel AS STRING) AS v_colunas_variaveis,
	
    CASE 
		WHEN MAX(score) OVER () <= 1 THEN score
		WHEN MAX(score) OVER () <= 10 THEN score / 10.0
		WHEN MAX(score) OVER () <= 100 THEN score / 100.0
		WHEN MAX(score) OVER () <= 1000 THEN score / 1000.0
		ELSE score / MAX(score) OVER ()
	END AS score,
	CAST(gr AS STRING) AS gr,
	CAST(decil AS STRING) AS decil,
	
    CAST(fl_bad AS INT) AS fl_bad,

FROM {tabela_output_modelo}
""")
```

### 3.2 Camada Prata — `df_cubo`

Empilha (unpivot) as colunas de `fl_bad*` para gerar a dimensão `mob`, agrega os dados e calcula o ranking padronizado de MOB.

```sql
WITH 
cte_colunas_selecionadas AS (
    SELECT 
    
        monitoramento,
        
        dt_colunas_data,

        tipo_base,
        score,     
        decil,
        gr,

        f_colunas_filtros,

        v_colunas_variaveis,

        fl_bad_colunas_bad,
		
        1 as n
        
    FROM tempview_{tabela_bronze}
),

cte_base_empilhada AS (
    SELECT 
    
        monitoramento,
        dt_colunas_data,
        score,
        tipo_base,
        decil,
        gr,
        f_colunas_filtros,
        v_colunas_variaveis,
        n,
        
        mob,
        qt_bad
        
    FROM cte_colunas_selecionadas
    LATERAL VIEW STACK(
        {qt_colunas_bads},
        'fl_bad_colunas_bad', fl_bad_colunas_bad
    ) AS mob, qt_bad
),

cte_agregado AS (
    SELECT
        
        monitoramento,
        dt_colunas_data,
        score,
        tipo_base,
        decil,
        gr,
        f_colunas_filtros,
        v_colunas_variaveis,
        mob,
        
        SUM(n) AS n,
        SUM(qt_bad) AS qt_bad
        
    FROM cte_base_empilhada
    GROUP BY ALL
),

cte_mob_rank AS (
    SELECT
        a.*,
        m.mob_rank
    FROM cte_agregado a
    LEFT JOIN (
        SELECT
            mob,
            ROW_NUMBER() OVER (ORDER BY mob) AS mob_rank
        FROM (
            SELECT DISTINCT mob
            FROM cte_agregado
        )
    ) m
        ON a.mob = m.mob
)

SELECT *
FROM cte_mob_rank
```

### 3.3 Camada Ouro — Tabelas Finais

A camada Ouro gera quatro tabelas finais, cada uma responsável por um domínio analítico do monitoramento: performance por decil, grupos de risco, variáveis, e indicadores estatísticos (KS, GINI, PSI).

#### `df_performance`

Realiza o unpivot das colunas `decil*`, gerando a estrutura vertical `decil_tipo` / `decil_valor` com ranking padronizado.

```sql
WITH 
cte_base_empilhada AS (
    SELECT 

        monitoramento,
        mob,
        dt_colunas_data,
        tipo_base,
        f_colunas_filtros,
        n,
        qt_bad,
        
        decil_tipo,
        decil_valor
        
    FROM tempview_cubo
    LATERAL VIEW STACK(
        {qt_colunas_decil},
        'decil_colunas_decil', decil_colunas_decil
    ) AS decil_tipo, decil_valor
),

cte_agregado AS (
    SELECT
    
        monitoramento,
        mob,
        dt_colunas_data,
        tipo_base,
        f_colunas_filtros,
        decil_tipo,
        decil_valor,
        
        SUM(n) AS n,
        SUM(qt_bad) AS qt_bad
        
    FROM cte_base_empilhada
    GROUP BY ALL
),

cte_decil_rank AS (
    SELECT
        a.*,
        d.decil_rank
    FROM cte_agregado a
    LEFT JOIN (
        SELECT
            decil_tipo,
            ROW_NUMBER() OVER (ORDER BY decil_tipo) AS decil_rank
        FROM (
            SELECT DISTINCT decil_tipo
            FROM cte_agregado
        )
    ) d
        ON a.decil_tipo = d.decil_tipo
)

SELECT *
FROM cte_decil_rank
```

#### `df_gr`

Realiza o unpivot das colunas `gr_*`, gerando a estrutura vertical `gr_tipo` / `gr_valor` com ranking padronizado.

```sql
WITH 
cte_base_empilhada AS (
    SELECT 

        monitoramento,
        mob,
        dt_colunas_data,
        tipo_base,
        f_colunas_filtros,
        n,
        qt_bad,
        
        gr_tipo,
        gr_valor
        
    FROM tempview_cubo
    LATERAL VIEW STACK(
        {qt_colunas_gr},
        'gr_colunas_gr', gr_colunas_gr
    ) AS gr_tipo, gr_valor
),

cte_agregado AS (
    SELECT
    
        monitoramento,
        mob,
        dt_colunas_data,
        tipo_base,
        f_colunas_filtros,
        gr_tipo,
        gr_valor,
        
        SUM(n) AS n,
        SUM(qt_bad) AS qt_bad
        
    FROM cte_base_empilhada
    GROUP BY ALL
),

cte_gr_rank AS (
    SELECT
        a.*,
        g.gr_rank
    FROM cte_agregado a
    LEFT JOIN (
        SELECT
            gr_tipo,
            ROW_NUMBER() OVER (ORDER BY gr_tipo) AS gr_rank
        FROM (
            SELECT DISTINCT gr_tipo
            FROM cte_agregado
        )
    ) g
        ON a.gr_tipo = g.gr_tipo
)

SELECT *
FROM cte_gr_rank
```

#### `df_variaveis`

Realiza o unpivot das colunas `v_*`, gerando a estrutura vertical `variavel_tipo` / `variavel_valor`, com ranking particionado por segmento (`f_segmento`).

```sql
WITH 
cte_base_empilhada AS (
    SELECT
    
        monitoramento,
        mob,
        dt_colunas_data,
        tipo_base,
        f_colunas_filtros,
        n,
        qt_bad,

        variavel_tipo,
        variavel_valor
        
    FROM tempview_cubo 
    LATERAL VIEW STACK(
        {qt_colunas_variaveis},
        'v_colunas_variaveis', v_colunas_variaveis
    ) AS variavel_tipo, variavel_valor
    WHERE variavel_valor IS NOT NULL
),

cte_agregado AS (
    SELECT
    
        monitoramento,
        mob,
        dt_colunas_data,
        tipo_base,
        f_colunas_filtros,
        variavel_tipo,
        variavel_valor,
        
        SUM(n) AS n,
        SUM(qt_bad) AS qt_bad
        
    FROM cte_base_empilhada
    GROUP BY ALL
),

cte_variavel_rank AS (
    SELECT
        a.*,
        v.variavel_rank
    FROM cte_agregado a
    LEFT JOIN (
        SELECT
            f_segmento,
            variavel_tipo,
            ROW_NUMBER() OVER (
                PARTITION BY f_segmento
                ORDER BY variavel_tipo
            ) AS variavel_rank
        FROM (
            SELECT DISTINCT
                f_segmento,
                variavel_tipo
            FROM cte_agregado
        )
    ) v
        ON a.f_segmento = v.f_segmento
       AND a.variavel_tipo = v.variavel_tipo
)

SELECT *
FROM cte_variavel_rank
```

#### `df_indicadores`

Calcula os três indicadores estatísticos centrais do monitoramento: **KS**, **GINI** e **PSI**, consolidados em uma única tabela via `UNION ALL`.

```sql
WITH 

cte_variaveis_unpivot AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        variavel_tipo,
        variavel_valor,
        n,
        qt_bad
    FROM tempview_variaveis
),

cte_discriminacao_base AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        score,
        SUM(n) AS n,
        SUM(qt_bad) AS qt_bad,
        SUM(n) - SUM(qt_bad) AS qt_good
    FROM tempview_cubo
    GROUP BY dt_movimento, tipo_base, f_colunas_filtros, monitoramento, mob, score
),

cte_discriminacao_totais AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        SUM(n) AS total_casos,
        SUM(qt_bad) AS total_bads,
        SUM(qt_good) AS total_goods
    FROM cte_discriminacao_base
    GROUP BY dt_movimento, tipo_base, f_colunas_filtros, monitoramento, mob
),

cte_discriminacao_acumulado AS (
    SELECT 
        b.dt_movimento,
        b.tipo_base,
        b.f_colunas_filtros,
        b.monitoramento,
        b.mob,
        b.score,
        b.n,
        b.qt_bad,
        b.qt_good,
        t.total_casos,
        t.total_bads,
        t.total_goods,
        SUM(b.n) OVER (
            PARTITION BY b.dt_movimento, b.tipo_base, b.f_colunas_filtros, b.monitoramento, b.mob
            ORDER BY b.score ASC
        ) AS acum_casos,
        SUM(b.qt_good) OVER (
            PARTITION BY b.dt_movimento, b.tipo_base, b.f_colunas_filtros, b.monitoramento, b.mob
            ORDER BY b.score ASC
        ) AS acum_goods,
        SUM(b.qt_bad) OVER (
            PARTITION BY b.dt_movimento, b.tipo_base, b.f_colunas_filtros, b.monitoramento, b.mob
            ORDER BY b.score ASC
        ) AS acum_bads
    FROM cte_discriminacao_base b
    INNER JOIN cte_discriminacao_totais t
        ON b.dt_movimento = t.dt_movimento
        AND b.tipo_base = t.tipo_base
        AND b.f_colunas_filtros = t.f_colunas_filtros
        AND b.monitoramento = t.monitoramento
        AND b.mob = t.mob
),

cte_discriminacao_com_lag AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        score,
        total_casos,
        total_bads,
        total_goods,
        acum_casos,
        acum_goods,
        acum_bads,
        LAG(acum_bads, 1, 0) OVER (
            PARTITION BY dt_movimento, tipo_base, f_colunas_filtros, monitoramento, mob
            ORDER BY score ASC
        ) AS acum_bads_anterior,
        LAG(acum_casos, 1, 0) OVER (
            PARTITION BY dt_movimento, tipo_base, f_colunas_filtros, monitoramento, mob
            ORDER BY score ASC
        ) AS acum_casos_anterior
    FROM cte_discriminacao_acumulado
),

cte_discriminacao_percentuais AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        total_casos,
        total_bads,
        total_goods,
        ROUND(acum_goods / NULLIF(total_goods, 0), 6) AS perc_acum_goods,
        ROUND(acum_bads / NULLIF(total_bads, 0), 6) AS perc_acum_bads,
        acum_bads,
        acum_bads_anterior,
        acum_casos,
        acum_casos_anterior
    FROM cte_discriminacao_com_lag
    WHERE total_goods > 0 
      AND total_bads > 0
),

cte_discriminacao_metricas AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        MAX(ABS(perc_acum_goods - perc_acum_bads)) AS ks,
        SUM(
            ((acum_bads / NULLIF(total_bads, 0)) + 
             (acum_bads_anterior / NULLIF(total_bads, 0))) / 2.0 *
            ((acum_casos / NULLIF(total_casos, 0)) - 
             (acum_casos_anterior / NULLIF(total_casos, 0)))
        ) AS area_sob_curva
    FROM cte_discriminacao_percentuais
    GROUP BY dt_movimento, tipo_base, f_colunas_filtros, monitoramento, mob, total_bads, total_casos
),

cte_ks_final AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        'KS' AS indicador_tipo,
        NULL AS variavel_tipo,
        ROUND(ks * 100, 2) AS indicador_valor
    FROM cte_discriminacao_metricas
),

cte_gini_final AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        'GINI' AS indicador_tipo,
        NULL AS variavel_tipo,
        ROUND(ABS((2 * area_sob_curva - 1)) * 100, 2) AS indicador_valor
    FROM cte_discriminacao_metricas
),

cte_psi_agregado AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        variavel_tipo,
        variavel_valor,
        SUM(n) AS n
    FROM cte_variaveis_unpivot
    GROUP BY dt_movimento, tipo_base, f_colunas_filtros, monitoramento, mob, variavel_tipo, variavel_valor
),

cte_psi_dev_baseline AS (
    SELECT 
        f_colunas_filtros,
        monitoramento,
        mob,
        variavel_tipo,
        variavel_valor,
        SUM(n) AS n_dev
    FROM cte_psi_agregado
    WHERE tipo_base = 'DESENVOLVIMENTO'
    GROUP BY f_colunas_filtros, monitoramento, mob, variavel_tipo, variavel_valor
),

cte_psi_proporcoes_dev AS (
    SELECT 
        f_colunas_filtros,
        monitoramento,
        mob,
        variavel_tipo,
        variavel_valor,
        n_dev,
        n_dev / SUM(n_dev) OVER (
            PARTITION BY f_colunas_filtros, monitoramento, mob, variavel_tipo
        ) AS proporcao_dev
    FROM cte_psi_dev_baseline
),

cte_psi_proporcoes_prod AS (
    SELECT 
        dt_movimento,
        tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        variavel_tipo,
        variavel_valor,
        n,
        n / SUM(n) OVER (
            PARTITION BY dt_movimento, f_colunas_filtros, monitoramento, mob, variavel_tipo
        ) AS proporcao_prod
    FROM cte_psi_agregado
    WHERE tipo_base = 'PRODUCAO'
),

cte_psi_safrado AS (
    SELECT 
        p_prod.dt_movimento,
        p_prod.f_colunas_filtros,
        p_prod.monitoramento,
        p_prod.mob,
        p_prod.variavel_tipo,
        p_prod.variavel_valor,
        p_dev.proporcao_dev,
        p_prod.proporcao_prod,
        (p_prod.proporcao_prod - p_dev.proporcao_dev) * 
        LN(p_prod.proporcao_prod / NULLIF(p_dev.proporcao_dev, 0)) AS psi_componente
    FROM cte_psi_proporcoes_prod p_prod
    LEFT JOIN cte_psi_proporcoes_dev p_dev
        ON p_prod.f_colunas_filtros = p_dev.f_colunas_filtros
        AND p_prod.monitoramento = p_dev.monitoramento
        AND p_prod.mob = p_dev.mob
        AND p_prod.variavel_tipo = p_dev.variavel_tipo
        AND p_prod.variavel_valor = p_dev.variavel_valor
    WHERE p_dev.proporcao_dev > 0 
      AND p_prod.proporcao_prod > 0
),

cte_psi_final AS (
    SELECT 
        dt_movimento,
        'PRODUCAO' AS tipo_base,
        f_colunas_filtros,
        monitoramento,
        mob,
        'PSI' AS indicador_tipo,
        variavel_tipo,
        ROUND(SUM(psi_componente), 4) AS indicador_valor
    FROM cte_psi_safrado
    GROUP BY dt_movimento, f_colunas_filtros, monitoramento, mob, variavel_tipo
)

SELECT * FROM cte_ks_final
UNION ALL
SELECT * FROM cte_gini_final
UNION ALL
SELECT * FROM cte_psi_final
```

---

## 4. Bases Ouro — Schemas Finais

As quatro tabelas Ouro abaixo (exemplo para o monitoramento `cpinssbehaviorchurnv4`) são as únicas fontes consumidas diretamente pelo Power BI.

### `monitoramento_cpinssbehaviorchurnv4_performance`

| Coluna | Tipo |
|---|---|
| monitoramento | string |
| mob | string |
| tipo_base | string |
| dt_movimento | date |
| dt_atualizacao | date |
| f_segmento | string |
| f_ds_tipo | string |
| f_fl_application | string |
| f_fl_ataque_banco | string |
| f_ds_produto_ativo | string |
| f_ds_banco_origem_proposta_in100 | string |
| f_ds_grupo_1 | string |
| f_fl_gc | string |
| f_fl_ultima_atualizacao | string |
| decil_tipo | string |
| decil_valor | string |
| decil_rank | int |
| n | bigint |
| qt_bad | bigint |

### `monitoramento_cpinssbehaviorchurnv4_indicadores`

| Coluna | Tipo |
|---|---|
| monitoramento | string |
| mob | string |
| tipo_base | string |
| dt_movimento | date |
| f_segmento | string |
| variavel_tipo | string |
| indicador_tipo | string |
| indicador_valor | double |

### `monitoramento_cpinssbehaviorchurnv4_variaveis`

| Coluna | Tipo |
|---|---|
| monitoramento | string |
| mob | string |
| tipo_base | string |
| dt_movimento | date |
| dt_atualizacao | date |
| f_segmento | string |
| f_ds_tipo | string |
| f_fl_application | string |
| f_fl_ataque_banco | string |
| f_ds_produto_ativo | string |
| f_ds_banco_origem_proposta_in100 | string |
| f_ds_grupo_1 | string |
| f_fl_gc | string |
| f_fl_ultima_atualizacao | string |
| variavel_tipo | string |
| variavel_valor | string |
| variavel_rank | int |
| n | bigint |
| qt_bad | bigint |

### `monitoramento_cpinssbehaviorchurnv4_gr`

| Coluna | Tipo |
|---|---|
| monitoramento | string |
| mob | string |
| tipo_base | string |
| dt_movimento | date |
| dt_atualizacao | date |
| f_segmento | string |
| f_ds_tipo | string |
| f_fl_application | string |
| f_fl_ataque_banco | string |
| f_ds_produto_ativo | string |
| f_ds_banco_origem_proposta_in100 | string |
| f_ds_grupo_1 | string |
| f_fl_gc | string |
| f_fl_ultima_atualizacao | string |
| gr_tipo | string |
| gr_valor | int |
| gr_rank | int |
| n | bigint |
| qt_bad | bigint |

---

## 5. Business Intelligence

### 5.1 Power Query

O Power Query deve conter **apenas** a conexão com a fonte e o `SELECT` da tabela final — nenhuma transformação adicional é permitida.

**GR**
```m
let
    Fonte = Odbc.Query("dsn=trino-reports", "SELECT * FROM datalake_creditdatascience_public.monitoramento_cpinssbehaviorchurnv4_gr")
in
    Fonte
```

**PERFORMANCE**
```m
let
    Fonte = Odbc.Query("dsn=trino-reports", "SELECT * FROM datalake_creditdatascience_public.monitoramento_cpinssbehaviorchurnv4_performance")
in
    Fonte
```

**VARIAVEIS**
```m
let
    Fonte = Odbc.Query("dsn=trino-reports", "SELECT * FROM datalake_creditdatascience_public.monitoramento_cpinssbehaviorchurnv4_variaveis")
in
    Fonte
```

**INDICADORES**
```m
let
    Fonte = Odbc.Query("dsn=trino-reports", "SELECT * FROM datalake_creditdatascience_public.monitoramento_cpinssbehaviorchurnv4_indicadores")
in
    Fonte
```

### 5.2 DAX — Medidas Padrão

Todas as métricas do dashboard devem ser criadas como *measures* explícitas na tabela virtual `_medidas`, nunca via soma automática de coluna ou agregação implícita do Power BI.

**Trigger de atualização**
```dax
MEASURE '_medidas'[trigger] = UTCNOW()*1
```

**Bad rate — Grupo de Risco**
```dax
MEASURE '_medidas'[m_gr_bad_rate] =
VAR QtBad =
    SUM(GR[qt_bad])

VAR QtTotal =
    SUM(GR[n])

VAR Resultado =
    DIVIDE(QtBad, QtTotal)

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

**Cor condicional do GR**
```dax
MEASURE '_medidas'[m_gr_cor] =
VAR ValorAtual =
    [m_gr_bad_rate]

VAR MaxNaColuna =
    MAXX(
        ALLSELECTED(GR[gr_valor]),
        [m_gr_bad_rate]
    )

VAR MinNaColuna =
    MINX(
        ALLSELECTED(GR[gr_valor]),
        [m_gr_bad_rate]
    )

RETURN
    DIVIDE(
        ValorAtual - MinNaColuna,
        MaxNaColuna - MinNaColuna
    )
```

**Quantidade total acumulada — GR**
```dax
MEASURE '_medidas'[m_gr_qtd_total_acumulado] =
CALCULATE(
    SUM(GR[n]),
    FILTER(
        ALL(GR[gr_valor]),
        GR[gr_valor] <= MAX(GR[gr_valor])
    )
)
```

**Quantidade bad acumulada — GR**
```dax
MEASURE '_medidas'[m_gr_qtd_bad_acumulado] =
CALCULATE(
    SUM(GR[qt_bad]),
    FILTER(
        ALLSELECTED(GR[gr_valor]),
        ISONORAFTER(
            GR[gr_valor],
            MAX(GR[gr_valor]),
            DESC
        )
    )
)
```

**Bad rate — Performance**
```dax
MEASURE '_medidas'[m_perfomance_bad_rate] =
VAR QtBad =
    SUM(PERFORMANCE[qt_bad])

VAR QtTotal =
    SUM(PERFORMANCE[n])

VAR Resultado =
    DIVIDE(QtBad, QtTotal)

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

**Bad rate — Variáveis**
```dax
MEASURE '_medidas'[m_variaveis_bad_rate] =
VAR QtBad =
    SUM(VARIAVEIS[qt_bad])

VAR QtTotal =
    SUM(VARIAVEIS[n])

VAR Resultado =
    DIVIDE(QtBad, QtTotal)

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

**Título dinâmico — Distribuição de variáveis**
```dax
MEASURE '_medidas'[m_variaveis_titulo_distribuicao] =
CONCATENATE(
    "DISTRIBUIÇÃO - ",
    UPPER(SELECTEDVALUE(VARIAVEIS[variavel_tipo]))
)
```

**Título dinâmico — Variação de bad**
```dax
MEASURE '_medidas'[m_variaveis_titulo_variacao] =
CONCATENATE(
    "VARIAÇÃO BAD - ",
    UPPER(SELECTEDVALUE(VARIAVEIS[variavel_tipo]))
)
```

**Label de última atualização**
```dax
MEASURE '_medidas'[m_refresh_timestamp_label] =
"Atualização: " &
FORMAT(
    MAX('_refresh_timestamp'[refresh_datetime]),
    "dd/MM/yyyy HH:mm:ss"
)
```

**Quantidades totais e de bad (KPIs base)**
```dax
MEASURE '_medidas'[m_perfomance_qtd_total] =
VAR Resultado =
    SUM(PERFORMANCE[n])

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

```dax
MEASURE '_medidas'[m_perfomance_qtd_bad] =
VAR Resultado =
    SUM(PERFORMANCE[qt_bad])

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

```dax
MEASURE '_medidas'[m_gr_qtd_total] =
VAR Resultado =
    SUM(GR[n])

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

```dax
MEASURE '_medidas'[m_gr_qtd_bad] =
VAR Resultado =
    SUM(GR[qt_bad])

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

```dax
MEASURE '_medidas'[m_variaveis_qtd_total] =
VAR Resultado =
    SUM(VARIAVEIS[n])

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

**Indicadores — KS, GINI e PSI**
```dax
MEASURE '_medidas'[m_indicadores_ks] =
VAR Resultado =
    CALCULATE(
        SUM(INDICADORES[valor]),
        INDICADORES[indicador] = "KS"
    )

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

```dax
MEASURE '_medidas'[m_indicadores_gini] =
VAR Resultado =
    CALCULATE(
        SUM(INDICADORES[valor]),
        INDICADORES[indicador] = "GINI"
    )

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

```dax
MEASURE '_medidas'[m_indicadores_psi] =
VAR Resultado =
    CALCULATE(
        SUM(INDICADORES[valor]),
        INDICADORES[indicador] = "PSI"
    )

RETURN
    IF(Resultado = 0, BLANK(), Resultado)
```

---

## 6. Padronização Power BI

### 6.1 Objetivo

Padronizar a arquitetura dos dashboards de monitoramento de modelos para garantir escalabilidade, sustentação simplificada, alta replicabilidade, melhor performance, padronização visual, facilidade de auditoria e de documentação, e redução do esforço operacional.

### 6.2 Princípio de Arquitetura

O Power BI deve atuar **apenas** como camada de visualização. Toda transformação, tratamento e modelagem de dados deve ocorrer no pipeline de engenharia — em SQL, Spark ou Python. O Power BI **não** deve ser utilizado como ferramenta de ETL.

### 6.3 Regras de Transformação

**Não é permitido no Power BI:**
- Criar colunas calculadas para tratamento
- Renomear colunas
- Alterar tipos
- Tratar labels
- Fazer ETL no Power Query
- Criar regras de negócio
- Aplicar transformações massivas

**Todo tratamento deve ocorrer:**
- Antes da carga
- Na camada de engenharia de dados
- Nas tabelas do DataLake

Essa disciplina garante melhor performance, menor consumo de memória, melhor tempo de atualização, melhor experiência de navegação e maior replicabilidade.

### 6.4 Power Query — Padrão

O Power Query deve conter apenas a conexão com a fonte e o `SELECT` da tabela final:

```sql
SELECT * FROM datalake_creditdatascience_public.monitoramento_{nome_monitoramento}_gr
```

Nenhuma transformação adicional deve existir, e o Editor Avançado deve permanecer mínimo.

### 6.5 Template Padrão

Foi definido um template padrão para novos projetos, composto por dois arquivos oficiais:

| Arquivo | Descrição | Link |
|---|---|---|
| `tema.json` | Padronização visual (cores, fontes, estilos) | [gitlab.com/.../powerbi_monitoramento/tema.json](https://gitlab.com/agi-repo/equipes/machine-learning/monitoramento-modelagem/powerbi_monitoramento/tema.json) |
| `template.pbit` | Estrutura base do dashboard | [gitlab.com/.../powerbi_monitoramento/template.pbit](https://gitlab.com/agi-repo/equipes/machine-learning/monitoramento-modelagem/powerbi_monitoramento/template.pbit) |

O uso do template garante padronização visual e estrutural, além de replicação rápida e redução do tempo de desenvolvimento.

#### `tema.json`

Arquivo responsável pela padronização visual: paleta de cores, fontes, estilos de gráficos, formatações e configurações visuais. Garante consistência visual, redução de ajustes manuais e padronização entre dashboards.

#### `template.pbit`

Arquivo base do Power BI, contendo estrutura do dashboard, layout, visuais padronizados, medidas DAX, configurações e navegação. Permite reutilização, escalabilidade, replicação rápida e sustentação simplificada — o objetivo é criar novos dashboards alterando apenas as fontes de dados.

### 6.6 Fontes Padrão

Todo dashboard deve possuir exatamente **5 fontes**:

1. `PERFORMANCE`
2. `GR`
3. `VARIAVEIS`
4. `INDICADORES`
5. `_medidas` *(tabela virtual usada apenas para armazenar as medidas DAX)*

### 6.7 DAX — Padrão Obrigatório

Todas as métricas devem ser criadas via DAX, nunca por soma automática de coluna, agregação implícita ou medida automática do Power BI.

| ✅ Utilizar | ❌ Não utilizar |
|---|---|
| `[m_gr_bad_rate]` (measure explícita) | `SUM(GR[qt_bad])` (agregação direta no visual) |

Objetivo: melhor governança, facilidade de manutenção, padronização, reaproveitamento e auditoria simplificada.

### 6.8 Filtros

As colunas `dt_*`, `f_*` e `mob` devem ficar prioritariamente nos filtros laterais, garantindo padronização de navegação, melhor experiência de uso, facilidade de replicação e redução de complexidade visual.

### 6.9 Campos dos Gráficos

- **Eixos:** colunas dimensionais padronizadas — `dt_*`, `gr_valor`, `decil_valor`, `variavel_valor`
- **Valores:** apenas medidas DAX
- **Não utilizar:** soma automática, média automática, count automático

Objetivo: comportamento consistente, replicabilidade e governança analítica.

### 6.10 Arquitetura Unpivot

As colunas `gr_*`, `decil*`, `v_*` e `fl_bad*` devem ser transformadas em estrutura vertical (unpivot), visando escalabilidade, estrutura padronizada, automatização de gráficos, facilidade de append e redução de manutenção.

#### Padrão GR (pós-unpivot)

| Coluna | Descrição |
|---|---|
| `gr_tipo` | Nome original da coluna |
| `gr_valor` | Valor da categoria |
| `gr_rank` | Ordenação padronizada para gráficos |

Objetivo: evitar dependência textual, automatizar visuais, permitir filtros genéricos e facilitar replicação.

#### Padrão Decil (pós-unpivot)

| Coluna |
|---|
| `decil_tipo` |
| `decil_valor` |
| `decil_rank` |

Objetivo: padronizar visualizações, automatizar ordenação e facilitar reaproveitamento.

#### Padrão Variáveis (pós-unpivot)

| Coluna |
|---|
| `variavel_tipo` |
| `variavel_valor` |
| `variavel_rank` |

Objetivo: automatizar páginas e gráficos, e permitir replicação entre modelos.

#### Padrão Bad / MOB

O unpivot de `fl_bad*` possui comportamento especial, gerando a estrutura `mob` (nome da visão MOB) e `qt_bad` (quantidade de eventos bad). Múltiplas colunas `fl_bad*` geram múltiplos MOBs; uma única coluna gera um MOB único padrão. Objetivo: permitir visão temporal de inadimplência e padronizar todos os dashboards.

### 6.11 Valores Default

Caso o modelo não possua GR ou DECIL, deve ser utilizado o valor padrão `"0"`, garantindo arquitetura única, evitando quebra de visual e assegurando compatibilidade estrutural.

### 6.12 Coluna `monitoramento`

Além das colunas pertencentes aos outputs dos modelos, todas as tabelas devem possuir a coluna `monitoramento`, que identifica o nome do processo monitorado. Isso permite append entre modelos, dashboards consolidados e garante rastreabilidade.

### 6.13 Append Futuro

A padronização estrutural permite `UNION`, `APPEND` e consolidação multi-modelos — por exemplo, um dashboard único de GR, um dashboard corporativo de monitoramento, ou comparativos entre modelos. Benefícios: escalabilidade corporativa, reaproveitamento e centralização analítica.

### 6.14 Automação de Visuais

Com as colunas padronizadas, os gráficos podem ser reutilizados, os mesmos DAX funcionam em múltiplos modelos e o mesmo layout funciona para diferentes projetos. Resultado: um processo baseado em copiar/colar, trocando apenas a fonte de dados, com criação rápida de novos dashboards.

### 6.15 Auditoria e Governança

A arquitetura padronizada permite melhor rastreabilidade, facilidade de auditoria, documentação automática, governança analítica, menor dependência operacional e maior previsibilidade técnica.

---

## 7. Resumo do Novo Processo e Ganhos

### 7.1 Pipeline Padronizado

1. O Cientista de Dados entrega a base analítica padronizada
2. A Engenharia processa as camadas **Bronze**, **Prata** e **Ouro**
3. O Power BI consome apenas as tabelas finais
4. O Dashboard reutiliza template, DAX e estrutura visual

### 7.2 Ganhos por Categoria

| Categoria | Ganhos |
|---|---|
| **Operacionais** | Menor tempo de desenvolvimento · Menor custo de sustentação · Menor retrabalho · Menor dependência manual · Replicação rápida · Facilidade de onboarding |
| **Performance** | Menor processamento no Power BI · Menor uso de memória · Melhor tempo de refresh · Melhor tempo de interação · Menor volume de cálculos locais |
| **Governança** | Estrutura única · Arquitetura previsível · Melhor auditabilidade · Versionamento simplificado · Documentação padronizada · Reaproveitamento de componentes |
| **Escalabilidade** | Dashboards reutilizáveis · DAX reutilizável · Visuais reutilizáveis · Possibilidade de append entre modelos · Consolidação corporativa futura |
| **Negócio** | Entrega mais rápida · Maior confiabilidade · Menor risco operacional · Maior consistência analítica · Melhor experiência do usuário final |

---

*Documento gerado a partir da especificação interna do processo de padronização de monitoramento de modelos.*
