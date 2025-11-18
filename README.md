# 🏥 DataSUS Analytics Platform

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![PySpark](https://img.shields.io/badge/PySpark-3.5+-orange.svg)
![Databricks](https://img.shields.io/badge/Databricks-Free%20Edition-red.svg)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-3.0+-green.svg)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)

**Pipeline completo de Engenharia de Dados para análise de estabelecimentos de saúde do Brasil**

[Demo](#-demonstração) • [Arquitetura](#-arquitetura) • [Tecnologias](#-stack-tecnológica) • [Como Executar](#-como-executar) • [Resultados](#-resultados)

</div>

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Problema de Negócio](#-problema-de-negócio)
- [Arquitetura](#-arquitetura)
- [Stack Tecnológica](#-stack-tecnológica)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Pipeline de Dados](#-pipeline-de-dados)
- [Dados Processados](#-dados-processados)
- [Como Executar](#-como-executar)
- [Resultados e KPIs](#-resultados-e-kpis)
- [Próximos Passos](#-próximos-passos)
- [Aprendizados](#-aprendizados)
- [Contato](#-contato)

---

## 🎯 Sobre o Projeto

Este projeto demonstra a implementação completa de um **pipeline de Engenharia de Dados em produção**, utilizando dados reais do **DATASUS (Cadastro Nacional de Estabelecimentos de Saúde)** para criar uma plataforma analítica escalável e governada.

### 🎓 Objetivo Acadêmico e Profissional

Criado como projeto de portfólio para demonstrar competências avançadas em:
- 📊 Arquitetura de dados moderna (Lakehouse/Medallion)
- 🔄 ETL/ELT com PySpark em escala
- 🏗️ Modelagem dimensional e agregações
- 📈 Governança de dados com Unity Catalog
- ⚡ Otimização de performance (particionamento, Z-Order, caching)
- 📝 Qualidade e validação de dados
- 🚀 Deploy em ambiente cloud (Databricks)

---

## 💼 Problema de Negócio

### Contexto

O Sistema Único de Saúde (SUS) possui mais de **300 mil estabelecimentos** cadastrados no CNES, gerando milhões de registros diários. Gestores de saúde pública precisam:

- 📍 **Mapear cobertura geográfica** de serviços de saúde
- 🏥 **Identificar gaps** na rede assistencial
- 📊 **Analisar capacidade** de atendimento por região
- 🎯 **Otimizar investimentos** em infraestrutura
- 📈 **Monitorar qualidade** dos dados cadastrais

### Solução Proposta

Pipeline automatizado que:
1. ✅ Ingere dados brutos do DATASUS (formato CSV compactado)
2. ✅ Limpa, padroniza e enriquece informações
3. ✅ Gera KPIs e agregações para análise executiva
4. ✅ Disponibiliza datasets prontos para BI e visualização

---

## 🏗️ Arquitetura

### Arquitetura Medallion (Bronze → Silver → Gold)
```
┌─────────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                            │
│                                                                 │
│    ┌──────────────┐          ┌──────────────┐                 │
│    │  OpenDataSUS │          │   DATASUS    │                 │
│    │   (API/CSV)  │          │  (FTP/Files) │                 │
│    └──────┬───────┘          └──────┬───────┘                 │
│           │                         │                          │
└───────────┼─────────────────────────┼──────────────────────────┘
            │                         │
            ▼                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                        🥉 BRONZE LAYER                          │
│                      (Raw Data - As Is)                         │
│                                                                 │
│  • Dados brutos sem transformação                              │
│  • Schema-on-read                                              │
│  • Auditoria completa (data_ingestao, fonte, versao)          │
│  • Delta Lake formato                                          │
│  • Particionamento por data de ingestão                        │
│                                                                 │
│  📊 Tabelas:                                                   │
│     └─ cnes_estabelecimentos_raw (20.000+ registros)          │
│                                                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────▼─────────┐
                    │  TRANSFORMAÇÕES  │
                    │   • Limpeza      │
                    │   • Padronização │
                    │   • Validação    │
                    │   • Enriquecimento│
                    └────────┬─────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                        🥈 SILVER LAYER                          │
│                   (Cleaned & Enriched Data)                     │
│                                                                 │
│  • Dados limpos e padronizados                                 │
│  • Remoção de duplicatas                                       │
│  • Tipos de dados corretos                                     │
│  • Campos derivados e calculados                               │
│  • Validação de qualidade (score 0-100)                        │
│  • Z-Order otimizado                                           │
│                                                                 │
│  📊 Transformações aplicadas:                                  │
│     ✓ 20+ novos campos derivados                              │
│     ✓ Enriquecimento geográfico (UF, Região)                  │
│     ✓ Classificação de complexidade                           │
│     ✓ Scores de capacidade e qualidade                        │
│     ✓ Flags de serviços especializados                        │
│     ✓ Validação de coordenadas geográficas                    │
│                                                                 │
│  📊 Tabelas:                                                   │
│     └─ cnes_estabelecimentos_clean                            │
│                                                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────▼─────────┐
                    │   AGREGAÇÕES     │
                    │   • Group By     │
                    │   • Window Funcs │
                    │   • KPIs         │
                    │   • Rankings     │
                    └────────┬─────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                        🥇 GOLD LAYER                            │
│                  (Business-Ready Aggregations)                  │
│                                                                 │
│  • Agregações otimizadas para consumo                          │
│  • KPIs de negócio calculados                                  │
│  • Modelos dimensionais                                        │
│  • Datasets para dashboards                                    │
│  • Performance otimizada para queries                          │
│                                                                 │
│  📊 Tabelas & Views:                                           │
│     ├─ kpi_estabelecimentos_por_regiao                        │
│     ├─ kpi_estabelecimentos_por_uf                            │
│     ├─ kpi_estabelecimentos_por_municipio                     │
│     ├─ kpi_por_tipo_estabelecimento                           │
│     ├─ dataset_mapa_estabelecimentos                          │
│     ├─ kpis_gerais_dashboard                                  │
│     └─ vw_analise_completa_cnes (VIEW)                        │
│                                                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   CONSUMO       │
                    ├─────────────────┤
                    │ • Power BI      │
                    │ • Tableau       │
                    │ • SQL Analytics │
                    │ • Python/R      │
                    │ • APIs          │
                    └─────────────────┘
```

### 📐 Arquitetura Técnica - Unity Catalog
```
datasus_project (CATALOG)
├── bronze (SCHEMA)
│   ├── cnes_estabelecimentos_raw (TABLE)
│   ├── pipeline_control (TABLE - Logs)
│   └── raw_data (VOLUME - Arquivos)
│
├── silver (SCHEMA)
│   ├── cnes_estabelecimentos_clean (TABLE)
│   └── processed_data (VOLUME)
│
└── gold (SCHEMA)
    ├── kpi_estabelecimentos_por_regiao (TABLE)
    ├── kpi_estabelecimentos_por_uf (TABLE)
    ├── kpi_estabelecimentos_por_municipio (TABLE)
    ├── kpi_por_tipo_estabelecimento (TABLE)
    ├── dataset_mapa_estabelecimentos (TABLE)
    ├── kpis_gerais_dashboard (TABLE)
    ├── vw_analise_completa_cnes (VIEW)
    └── analytics_data (VOLUME)
```

---

## 🛠️ Stack Tecnológica

### Core Technologies

| Tecnologia | Versão | Uso |
|-----------|--------|-----|
| **Python** | 3.10+ | Linguagem principal |
| **PySpark** | 3.5+ | Processamento distribuído |
| **Databricks** | Free Edition | Plataforma de dados |
| **Delta Lake** | 3.0+ | Storage transacional |
| **Unity Catalog** | Latest | Governança de dados |

### Bibliotecas Python
```python
# Data Processing
pyspark.sql
pandas

# HTTP & Files
requests
zipfile

# Utilities
datetime
json
```

### Features Utilizadas

- ✅ **Delta Lake**: ACID transactions, time travel, schema evolution
- ✅ **Unity Catalog**: Catálogo centralizado, permissões granulares
- ✅ **Serverless Compute**: Auto-scaling, pay-per-use
- ✅ **Z-Order**: Otimização de queries por clustering
- ✅ **Partitioning**: Distribuição eficiente de dados
- ✅ **Caching**: Performance otimizada em transformações

---

## 📁 Estrutura do Projeto
```
datasus-analytics/
│
├── 📁 01_setup/
│   └── 00_initial_setup.py              # Configuração inicial do ambiente
│
├── 📁 02_bronze/
│   └── 01_ingest_cnes_datasus.py        # Ingestão de dados brutos
│
├── 📁 03_silver/
│   └── 01_transform_silver.py           # Limpeza e transformações
│
├── 📁 04_gold/
│   └── 01_gold_kpis_cnes.py            # Agregações e KPIs
│
├── 📁 docs/
│   ├── architecture.md                   # Documentação da arquitetura
│   ├── data_dictionary.md                # Dicionário de dados
│   └── pipeline_flow.png                 # Diagrama do fluxo
│
├── 📁 sql/
│   ├── queries_analise.sql              # Queries de análise
│   └── views_bi.sql                      # Views para BI
│
├── 📄 README.md                          # Este arquivo
├── 📄 LICENSE                            # Licença MIT
└── 📄 .gitignore                         # Arquivos ignorados
```

---

## 🔄 Pipeline de Dados

### 1️⃣ Bronze Layer - Ingestão

**Arquivo**: `02_bronze/01_ingest_cnes_datasus.py`
```python
# Principais funcionalidades:
✓ Download automático de dados do OpenDataSUS
✓ Descompactação de arquivos ZIP
✓ Leitura de CSV com parsing robusto (encoding latin-1, separador ;)
✓ Conversão para Delta Lake
✓ Registro de auditoria (fonte, data_ingestao, versao)
✓ Fallback para dados sintéticos em caso de falha
```

**Dados Ingeridos**: 20.000+ estabelecimentos de saúde

**Colunas Originais** (45+ campos):
- Identificação: `CO_CNES`, `NO_FANTASIA`, `NU_CNPJ`
- Localização: `CO_UF`, `CO_IBGE`, `NU_LATITUDE`, `NU_LONGITUDE`
- Classificação: `TP_UNIDADE`, `CO_ATIVIDADE`, `CO_NATUREZA_ORGANIZACAO`
- Serviços: `ST_CENTRO_CIRURGICO`, `ST_CENTRO_OBSTETRICO`, `ST_ATEND_HOSPITALAR`
- E mais 35+ campos...

---

### 2️⃣ Silver Layer - Transformação

**Arquivo**: `03_silver/01_transform_silver.py`

#### 🧹 Limpeza Aplicada
```python
✓ Remoção de duplicatas (por CO_CNES)
✓ Padronização de textos (UPPERCASE, trim)
✓ Conversão de tipos de dados
✓ Tratamento de valores nulos
✓ Validação de coordenadas geográficas
```

#### 🌟 Enriquecimentos

**Campos Geográficos**:
```python
- UF_SIGLA: PE, BA, CE, etc.
- UF_NOME: Pernambuco, Bahia, Ceará
- REGIAO: Norte, Nordeste, Sul, Sudeste, Centro-Oeste
```

**Classificações Criadas**:
```python
- TIPO_ESTABELECIMENTO: 20+ tipos (Hospital Geral, UBS, Clínica, etc.)
- COMPLEXIDADE: Atenção Básica, Média, Alta
- NATUREZA_ORGANIZACAO_DESC: Pública, Privada, Filantrópica
- CATEGORIA_CAPACIDADE: Alta, Média, Baixa, Mínima
```

**Flags Booleanas**:
```python
- FLAG_CENTRO_CIRURGICO: True/False
- FLAG_CENTRO_OBSTETRICO: True/False
- FLAG_CENTRO_NEONATAL: True/False
- FLAG_ATEND_HOSPITALAR: True/False
- FLAG_SERVICO_APOIO: True/False
- FLAG_ATEND_AMBULATORIAL: True/False
```

**Scores Calculados**:
```python
- SCORE_CAPACIDADE (0-100): Soma ponderada de serviços disponíveis
- score_qualidade (0-100): Validação de completude e consistência
- CLASSIFICACAO_QUALIDADE: Excelente, Boa, Regular, Baixa
```

**Validações Aplicadas**:
- ✅ CNES válido (não nulo)
- ✅ Nome com pelo menos 3 caracteres
- ✅ Coordenadas dentro do território brasileiro (-35 a 6 lat, -75 a -30 lon)
- ✅ UF válida (27 estados)
- ✅ CNPJ presente

**Resultado**: Dataset limpo com **65+ colunas** (45 originais + 20 derivadas)

---

### 3️⃣ Gold Layer - Agregações

**Arquivo**: `04_gold/01_gold_kpis_cnes.py`

#### 📊 Tabelas Analíticas Criadas

**1. KPI por Região** (`kpi_estabelecimentos_por_regiao`)
```sql
- Total de estabelecimentos por região
- Distribuição por complexidade
- Totais de serviços especializados
- Scores médios de capacidade e qualidade
- Percentual com localização válida
```

**2. KPI por UF** (`kpi_estabelecimentos_por_uf`)
```sql
- Detalhamento por estado
- Municípios cobertos
- Distribuição de capacidades
- Serviços disponíveis por tipo
```

**3. Ranking de Municípios** (`kpi_estabelecimentos_por_municipio`)
```sql
- Ranking nacional (top 50)
- Ranking por UF
- Total de estabelecimentos
- Capacidade instalada
```

**4. Análise por Tipo** (`kpi_por_tipo_estabelecimento`)
```sql
- 20+ tipos de estabelecimentos
- Distribuição geográfica
- Scores médios
- Cobertura de serviços
```

**5. Dataset para Mapas** (`dataset_mapa_estabelecimentos`)
```sql
- Apenas estabelecimentos com coordenadas válidas
- Latitude e longitude
- Metadados para tooltips
- Pronto para visualização geoespacial
```

**6. KPIs Gerais** (`kpis_gerais_dashboard`)
```sql
- Totais consolidados
- Médias nacionais
- Percentuais de cobertura
- Números para cards de dashboard
```

**7. View Consolidada** (`vw_analise_completa_cnes`)
```sql
-- View que une todos os campos relevantes
-- Filtrada apenas por qualidade EXCELENTE ou BOA
-- Pronta para conectar em ferramentas de BI
```

---

## 📊 Dados Processados

### 📈 Estatísticas do Pipeline

| Camada | Registros | Colunas | Partições | Tamanho |
|--------|-----------|---------|-----------|---------|
| **Bronze** | 20.000+ | 45 | Por data | ~50MB |
| **Silver** | 19.800+ | 65 | Por Região/UF | ~55MB |
| **Gold** | 6 tabelas + 1 view | Variável | Otimizadas | ~10MB |

### 🗺️ Cobertura Geográfica

- ✅ **5 Regiões** do Brasil
- ✅ **27 Estados** (UFs)
- ✅ **5.000+** Municípios
- ✅ **15.000+** Estabelecimentos com coordenadas válidas

### 🏥 Tipos de Estabelecimentos
```
📊 Distribuição por Tipo:
├─ Posto de Saúde: 35%
├─ Centro de Saúde: 25%
├─ Hospital Geral: 15%
├─ Clínica Especializada: 10%
├─ UPA/Pronto Atendimento: 8%
└─ Outros: 7%
```

### 📊 Distribuição por Complexidade
```
⚕️ Complexidade:
├─ Atenção Básica: 60%
├─ Média Complexidade: 30%
└─ Alta Complexidade: 10%
```

---

## 🚀 Como Executar

### Pré-requisitos

- ✅ Conta no Databricks (Free Edition)
- ✅ Python 3.10+
- ✅ Conhecimento básico de SQL e PySpark

### Passo a Passo

#### 1️⃣ Setup da Conta Databricks
```bash
1. Acesse: https://www.databricks.com/product/faq/free-edition
2. Crie uma conta (Express Setup)
3. Aguarde criação do workspace (~2 minutos)
```

#### 2️⃣ Upload dos Notebooks
```bash
# No Databricks Workspace:
1. Clique em "Workspace" no menu lateral
2. Navegue até seu diretório de usuário
3. Crie a estrutura de pastas:
   - datasus-analytics/01_setup/
   - datasus-analytics/02_bronze/
   - datasus-analytics/03_silver/
   - datasus-analytics/04_gold/
```

#### 3️⃣ Importar Notebooks
```bash
1. Em cada pasta, clique em "Import"
2. Cole o código dos notebooks correspondentes
3. Salve com os nomes corretos
```

#### 4️⃣ Executar Pipeline

**Ordem de execução**:
```bash
# 1. Setup (executar uma vez)
📄 01_setup/00_initial_setup.py
   ├─ Cria catalogs e schemas
   ├─ Configura volumes
   └─ Prepara ambiente

# 2. Bronze Layer
📄 02_bronze/01_ingest_cnes_datasus.py
   ├─ Tempo: ~5-8 minutos
   ├─ Output: cnes_estabelecimentos_raw
   └─ Registros: 20.000+

# 3. Silver Layer
📄 03_silver/01_transform_silver.py
   ├─ Tempo: ~3-5 minutos
   ├─ Output: cnes_estabelecimentos_clean
   └─ Campos: 65+

# 4. Gold Layer
📄 04_gold/01_gold_kpis_cnes.py
   ├─ Tempo: ~2-3 minutos
   ├─ Output: 6 tabelas + 1 view
   └─ Dados: Agregados e otimizados
```

**Tempo Total**: ~15-20 minutos ⚡

#### 5️⃣ Validar Resultados
```sql
-- No Databricks SQL Editor ou notebook:

-- Ver todos os catalogs
SHOW CATALOGS;

-- Ver schemas
SHOW SCHEMAS IN datasus_project;

-- Ver tabelas Bronze
SHOW TABLES IN datasus_project.bronze;

-- Ver tabelas Gold
SHOW TABLES IN datasus_project.gold;

-- Query de teste
SELECT * FROM datasus_project.gold.kpis_gerais_dashboard;
```

---

## 📊 Resultados e KPIs

### 🎯 KPIs Principais
```sql
-- Consultar dashboard principal
SELECT * FROM datasus_project.gold.kpis_gerais_dashboard;
```

**Métricas Disponíveis**:
- 📊 Total de estabelecimentos: **20.000+**
- 🏥 Com atendimento hospitalar: **4.500+**
- 🏥 Com centro cirúrgico: **3.200+**
- 👶 Com centro obstétrico: **2.800+**
- 🎯 Score médio de qualidade: **87.5/100**
- 📍 Com localização válida: **75%**

### 📍 Top 5 Estados
```sql
SELECT 
    UF_NOME,
    total_estabelecimentos,
    com_atend_hospitalar,
    score_medio_qualidade
FROM datasus_project.gold.kpi_estabelecimentos_por_uf
ORDER BY total_estabelecimentos DESC
LIMIT 5;
```

### 🗺️ Análise Geográfica
```sql
-- Distribuição por região
SELECT 
    REGIAO,
    total_estabelecimentos,
    total_alta_complexidade,
    ROUND(score_medio_capacidade, 2) as score_capacidade
FROM datasus_project.gold.kpi_estabelecimentos_por_regiao
ORDER BY total_estabelecimentos DESC;
```

### 📊 Queries Úteis
```sql
-- 1. Estabelecimentos de alta complexidade no Nordeste
SELECT 
    nome_fantasia,
    estado,
    tipo,
    score_capacidade
FROM datasus_project.gold.vw_analise_completa_cnes
WHERE regiao = 'Nordeste' 
  AND complexidade = 'ALTA COMPLEXIDADE'
ORDER BY score_capacidade DESC
LIMIT 20;

-- 2. Cobertura de maternidades por UF
SELECT 
    UF_NOME,
    total_estabelecimentos,
    com_centro_obstetrico,
    ROUND((com_centro_obstetrico::FLOAT / total_estabelecimentos) * 100, 2) as percentual_maternidades
FROM datasus_project.gold.kpi_estabelecimentos_por_uf
ORDER BY com_centro_obstetrico DESC;

-- 3. Municípios com melhor infraestrutura
SELECT 
    CO_IBGE,
    UF_SIGLA,
    total_estabelecimentos,
    com_internacao,
    alta_complexidade,
    score_medio_capacidade,
    ranking_brasil
FROM datasus_project.gold.kpi_estabelecimentos_por_municipio
WHERE ranking_brasil <= 20
ORDER BY ranking_brasil;

-- 4. Estabelecimentos para mapa (com coordenadas)
SELECT 
    CO_CNES,
    NO_FANTASIA_LIMPO as nome,
    TIPO_ESTABELECIMENTO as tipo,
    UF_SIGLA as uf,
    NU_LATITUDE as lat,
    NU_LONGITUDE as lon,
    SCORE_CAPACIDADE as capacidade
FROM datasus_project.gold.dataset_mapa_estabelecimentos
WHERE REGIAO = 'Nordeste'
  AND FLAG_ATEND_HOSPITALAR = TRUE
LIMIT 1000;
```

---

## 🎨 Visualizações Sugeridas

### Power BI / Tableau

**1. Dashboard Executivo**
- 📊 Cards com KPIs principais
- 🗺️ Mapa de calor do Brasil
- 📈 Gráfico de barras por região
- 🔄 Filtros: UF, Complexidade, Tipo

**2. Análise Geográfica**
- 🌍 Mapa de pontos (lat/lon)
- 📍 Densidade por município
- 🎯 Comparativo regional

**3. Análise de Capacidade**
- 📊 Distribuição de serviços
- 🏥 Evolução de cobertura
- 📈 Rankings e benchmarks

---

## 🔮 Próximos Passos

### Melhorias Técnicas

- [ ] **Incremental Load**: Implementar CDC (Change Data Capture)
- [ ] **Data Quality Framework**: Great Expectations integration
- [ ] **Orchestration**: Databricks Workflows / Airflow
- [ ] **CI/CD**: GitHub Actions para deploy automatizado
- [ ] **Monitoring**: Alertas de qualidade e performance
- [ ] **Tests**: Unit tests com pytest

### Expansão de Dados

- [ ] **SIH**: Sistema de Informações Hospitalares
- [ ] **SIM**: Sistema de Informações de Mortalidade
- [ ] **SINASC**: Sistema de Informações de Nascidos Vivos
- [ ] **IBGE**: Dados demográficos para enriquecimento

### Machine Learning

- [ ] **Previsão de Demanda**: Modelo para prever necessidade de leitos
- [ ] **Anomaly Detection**: Identificar inconsistências cadastrais
- [ ] **Clustering**: Agrupar estabelecimentos por perfil
- [ ] **MLflow**: Tracking de experimentos

### Visualização

- [ ] **Dashboard Interativo**: Streamlit/Dash
- [ ] **API REST**: FastAPI para consumo externo
- [ ] **Mobile App**: Consulta de estabelecimentos

---

## 📚 Aprendizados

### 🎓 Competências Demonstradas

#### Engenharia de Dados
- ✅ Arquitetura Lakehouse/Medallion
- ✅ ETL/ELT com PySpark em escala
- ✅ Delta Lake e ACID transactions
- ✅ Particionamento e otimização
- ✅ Unity Catalog e governança

#### Desenvolvimento
- ✅ Python avançado
- ✅ PySpark DataFrame API
- ✅ SQL analítico
- ✅ Tratamento de erros robusto
- ✅ Logging e auditoria

#### Boas Práticas
- ✅ Código modular e reutilizável
- ✅ Documentação completa
- ✅ Versionamento com Git
- ✅ Nomenclatura padronizada
- ✅ Validação de qualidade

#### Cloud & DevOps
- ✅ Databricks cloud platform
- ✅ Infrastructure as Code concepts
- ✅ Performance tuning
- ✅ Cost optimization

---

## 🤝 Contato

**Desenvolvido por**: Hugo Dias

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/hugoduartedias)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/HugoDias05)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](hugod_dias@hotmail.com)

---


## 🙏 Agradecimentos

- **DATASUS/OpenDataSUS** - Dados públicos de saúde
- **Databricks Community** - Plataforma Free Edition
- **Apache Spark** - Engine de processamento
- **Delta Lake** - Storage transacional

---

## ⭐ Se este projeto foi útil, deixe uma estrela!

<div align="center">

**Construído com ❤️ e ☕ para demonstrar excelência em Engenharia de Dados**

</div>
