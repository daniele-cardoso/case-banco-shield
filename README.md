# Case Prático — Banco Shield vs Hidra (Universo Marvel)  
**Engenharia de Analytics | Dados fictícios | Entrega via GitHub**

> No multiverso da Marvel, o **Banco Shield** precisa entender onde ganha, onde perde e onde o risco pode “explodir”, enquanto a **Hidra** cresce com agressividade comercial.  
> Este repositório entrega um pipeline de **tratamento + auditoria de qualidade + base GOLD** pronta para análises e consumo em **Power BI**.

---

## 👩‍💻 Autoria
**Daniele Cardoso**

---

## 🎯 Objetivo do projeto
Transformar dados brutos e ruidosos em **informação confiável e acionável** para apoiar a diretoria na tomada de decisão, por meio de:

- **Tratamento dos dados** (limpeza, padronização, consistência e integridade)
- **Avaliação de qualidade** (checagens, logs e evidências do que foi encontrado/corrigido)
- **Base final GOLD** para análises e dashboard
- **Base para storytelling** (insumos para PPT + recomendações)

---

## ❓ Perguntas de negócio (que o dashboard e a análise devem responder)
1. Quais produtos são mais vendidos (**unidades** e **valor**) no Shield e na Hidra?  
2. Em quais localidades/regiões cada banco é mais forte?  
3. Quais produtos/localidades concentram maior risco (ex.: **30+ DPD**) e qual o trade-off com volume?  
4. Qual a maior oportunidade para o Shield:  
   - (i) ganhar share onde já é forte  
   - (ii) atacar nichos onde a Hidra domina  
   - (iii) reduzir risco para liberar crescimento  
5. Quais sinais de qualidade devem virar **controles automáticos** (SLA de dados)?

---

## 🧾 Dados do case (tabelas)
O case possui **1 fato** e **2 dimensões**:

- `dim_produto.csv`  
  `product_id (PK)`, `product_name`, `category`, `tenor_months`, `base_rate_apr`

- `dim_localidade.csv`  
  `location_id (PK)`, `location_name`, `macro_region`, `risk_factor_region`

- `fato_contratos.csv`  
  `contract_id`, `ano_mes (YYYYMM)`, `bank`, `product_id (FK)`, `location_id (FK)`, `units`,  
  `financed_amount`, `outstanding_balance`, `dpd`, `delinquent_amount_30p`, `risk_score`

📌 **Importante:** os dados são **fictícios** e contêm **ruídos intencionais** para simular problemas reais de governança.

---

## ✅ Regras de qualidade esperadas (metadados)
Principais regras usadas como base de validação:

- **Integridade referencial:** `product_id` e `location_id` devem existir nas dimensões  
- **Domínios válidos:**
  - `bank` ∈ {`Banco Shield`, `Hidra`}
  - `ano_mes` no intervalo **202501 a 202512**
- **Regras de valores:**
  - `financed_amount` e `outstanding_balance` **não negativos**
  - `delinquent_amount_30p = 0` quando `dpd < 30`
  - `risk_score` ∈ **[0, 1]**
- **Unicidade:** `contract_id` deve ser único

---

## 🧠 Estratégia de solução (pipeline)
O notebook implementa um pipeline versionado, com **auditoria por etapa** e geração de **base GOLD**.

### 🔁 Etapas implementadas no notebook
**PASSO 1 — Configuração do projeto**  
Padroniza diretórios e garante portabilidade (raw → processed → logs).

**PASSO 2 — Auditoria pré-limpeza (baseline)**  
Gera um perfil inicial de qualidade para comparar “antes vs depois”.

**PASSO 3 — Padronização de `bank` e `ano_mes`**  
- Normaliza banco para `Banco Shield` ou `Hidra`  
- Trata formatação/consistência do `ano_mes`

**PASSO 4 — Remoção de `location_id` inválido (FK)**  
Remove registros com FK quebrada / ausente, garantindo integridade com `dim_localidade`.

**PASSO 5 — Tratamento de duplicidade de `contract_id`**  
Quando existem IDs duplicados, novos IDs são gerados para restabelecer unicidade (mantendo rastreabilidade).

**PASSO 6 — Validação final completa (regras do metadados)**  
Executa checagens do dicionário (domínios, ranges, sinais, integridade).

**PASSO 7 — Teste de consistência para `financed_amount` negativo**  
Evidencia a ocorrência e mede impacto.

**PASSO 8 — Correção de `financed_amount` negativo**  
Corrige inversão de sinal quando aplicável e registra auditoria da correção.

**PASSO 9 — Imputação de `financed_amount` (NaN) usando `outstanding_balance`**  
- Imputa somente quando `outstanding_balance` é válido (>= 0)  
- Cria flag `financed_amount_imputed`

**PASSO 10 — Remoção de `product_id` inválido (FK)**  
Remove registros com FK quebrada / ausente, garantindo integridade com `dim_produto`.

**PASSO 11 — Validação final de qualidade (conformidade estrutural + integridade)**  
Gera `summary` e `sample` de issues (para evidência e priorização de controles).

**PASSO 12 — Auditoria comparativa (Antes vs Depois) + base GOLD**  
- `before` é sempre o **raw `fato_contratos.csv`**  
- `after` é a **última versão** gerada em `processed/`  
- Emite:
  - `step12_comparison_summary.csv`
  - `step12_changes_sample.csv`
  - `fato_contratos_gold.csv` (**base final**)

---

## 📦 Outputs gerados (principais)
### Dados tratados (exemplos)
- `data/processed/fato_contratos_clean_v1.csv`
- `data/processed/fato_contratos_clean_v2.csv`
- `data/processed/fato_contratos_clean_v3.csv`
- `data/processed/fato_contratos_clean_v4.csv`
- `data/processed/fato_contratos_gold.csv` ✅ (**base final**)

### Logs e evidências (quality)
- `quality_logs/cleaning_audit_step3_v1.csv`
- `quality_logs/cleaning_audit_step4_v1.csv`
- `quality_logs/cleaning_audit_step5_v1.csv`
- `quality_logs/cleaning_audit_step8_v1.csv`
- `quality_logs/cleaning_audit_step9_v1.csv`
- `quality_logs/cleaning_audit_step10_v1.csv`
- `quality_logs/validation_step11_summary_v1.csv`
- `quality_logs/validation_step11_issues_sample_v1.csv`
- `quality_logs/step12_comparison_summary.csv`
- `quality_logs/step12_changes_sample.csv`

---

## 🗂️ Estrutura sugerida do repositório
```text
/
  data/
    raw/                      # arquivos recebidos no case (inalterados)
      fato_contratos.csv
      dim_produto.csv
      dim_localidade.csv

    processed/                # outputs do tratamento
      fato_contratos_gold.csv

  notebooks/
    Case_Banco_Shield_Tratamento_Dados.ipynb

  quality_logs/               # auditorias e validações por etapa
    cleaning_audit_*.csv
    validation_*.csv
    step12_*.csv

  dashboards/
    Case - Banco Shield.pbix  # arquivo do Power BI

  ppt/
    Apresentacao_Case_Shield_Hidra.pptx  # apresentação de slides
