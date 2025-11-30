# NEXUS - Sistema de Predição de Bioincrustação

![Hackathon Transpetro](https://img.shields.io/badge/Hackathon-Transpetro-blue)
![Python](https://img.shields.io/badge/Python-3.11+-green)
![ML](https://img.shields.io/badge/ML-Ensemble-orange)

## 🎯 Sobre o Projeto

Sistema de predição de bioincrustação desenvolvido pela equipe **NEXUS** para o Hackathon Transpetro 2025. A solução utiliza Machine Learning para monitorar e prever níveis de bioincrustação em embarcações, permitindo otimização de manutenção, redução de consumo de combustível e apoio à descarbonização da frota.

## 🚢 Contexto

A Transpetro é a maior empresa de logística para os segmentos de óleo, gás e biocombustíveis da América Latina, operando:

- **48 navios** na frota
- **33 terminais**
- **8.500 km** de dutos
- **5.685 colaboradores**

A bioincrustação (acúmulo de organismos marinhos no casco) representa um desafio crítico, impactando:

- **5% a 25%** no consumo de combustível (segundo IMO)
- Emissões de CO₂
- Eficiência operacional
- Custos de manutenção

## 🎯 Desafio

> Como usar tecnologias inovadoras para monitorar e prever a bioincrustação, aumentando a eficiência operacional, reduzindo consumo de combustível e apoiando a descarbonização da frota da Transpetro?

## ✨ Solução Desenvolvida

### Funcionalidades Principais

1. **Predição de Níveis de Bioincrustação**

   - Modelo ensemble (XGBoost, LightGBM, Random Forest, Gradient Boosting)
   - Baseado em Fouling Rating IMO (escala 0-4)
   - Validação temporal (não aleatória)
   - R² de **0.9996** e MAE de **0.0048**

2. **Análise de Impacto Econômico**

   - Cálculo de penalidade de combustível (5-25%)
   - Estimativa de custos extras por dia/mês/ano
   - Projeção de emissões de CO₂

3. **Cenários Futuros**

   - Simulação "Não fazer nada" vs "Limpar agora"
   - Análise custo-benefício de limpezas
   - Recomendações de ação por navio

4. **Análise Individual da Frota**
   - Fouling Rating atual de cada navio
   - Priorização de ações (crítica, urgente, reativa, proativa)
   - Dashboard com classificação IMO

## 📊 Escala IMO MEPC.378(80)

| Rating | Tipo                             | Cobertura | Ação Recomendada    |
| ------ | -------------------------------- | --------- | ------------------- |
| **0**  | Sem bioincrustação               | -         | ✅ OK               |
| **1**  | Microincrustação (biofilme/limo) | -         | 🟡 Limpeza Proativa |
| **2**  | Macroincrustação leve            | 1-15%     | 🟠 Limpeza Reativa  |
| **3**  | Macroincrustação moderada        | 16-40%    | 🔴 Limpeza Urgente  |
| **4**  | Macroincrustação pesada          | 41-100%   | 🔴 Limpeza CRÍTICA  |

## 🏗️ Arquitetura da Solução

```
┌─────────────────────────────────────────────────────────────┐
│                     ENTRADA DE DADOS                         │
├─────────────────────────────────────────────────────────────┤
│ • Eventos de Navegação (50.904 registros)                   │
│ • Dados AIS (415.724 registros)                              │
│ • Consumo de Combustível (87.737 registros)                  │
│ • Relatórios IWS - Inspeções (29 registros)                  │
│ • Características dos Navios (21 embarcações)                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  PRÉ-PROCESSAMENTO                           │
├─────────────────────────────────────────────────────────────┤
│ • Agregação AIS por evento                                   │
│ • Cálculo de features avançadas                              │
│ • Processamento de dados IWS                                 │
│ • Merge de múltiplas fontes                                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                 ENGENHARIA DE FEATURES                       │
├─────────────────────────────────────────────────────────────┤
│ 1. Idle Time Features                                        │
│    • idle_time_ratio, idle_days, low_speed_days             │
│                                                              │
│ 2. Velocity Risk Score                                       │
│    • velocity_risk (0-3 baseado em velocidade)              │
│                                                              │
│ 3. Operational Profile                                       │
│    • operation_continuity, speed_variability                │
│                                                              │
│ 4. Low Shear Zones Exposure                                 │
│    • low_shear_exposure                                      │
│                                                              │
│ 5. Biogeographic Region Risk                                │
│    • bio_region, region_risk (Norte/Nordeste/Sudeste-Sul)   │
│                                                              │
│ 6. Temperature Proxy                                         │
│    • temp_proxy, temp_risk                                   │
│                                                              │
│ 7. Days Since Last Cleaning                                 │
│    • days_since_clean, median_interval                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    MODELO ENSEMBLE                           │
├─────────────────────────────────────────────────────────────┤
│ • XGBoost (n_estimators=300, lr=0.03)                       │
│ • LightGBM (n_estimators=300, lr=0.03)                      │
│ • Random Forest (n_estimators=200, max_depth=10)            │
│ • Gradient Boosting (n_estimators=200, lr=0.05)             │
│                                                              │
│ Pesos baseados em performance (1/MAE)                        │
│ Validação temporal (80/20 split)                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        OUTPUTS                               │
├─────────────────────────────────────────────────────────────┤
│ • Fouling Rating (0-4) por navio                            │
│ • Impacto econômico (custo extra/ano)                       │
│ • Emissões CO₂ extras                                        │
│ • Cenários futuros (limpar vs não limpar)                   │
│ • Recomendações de ação                                      │
│ • Relatório CSV da frota                                     │
└─────────────────────────────────────────────────────────────┘
```

## 🔬 Features Utilizadas (26 features)

### Operacionais

- `speed_mean`, `speed_std`, `speed_min`, `speed_max`
- `duration_h`, `distance`
- `displacement`

### Idle Time

- `frac_stop`, `frac_low_speed`
- `idle_days`, `low_speed_days`

### Risk Scores

- `velocity_risk` (0-3)
- `biofouling_risk_score` (0-1)
- `region_risk`, `temp_risk`

### Operacional Avançado

- `operation_continuity`
- `speed_variability`
- `low_shear_exposure`

### Ambientais

- `beaufort`, `seaCondition`
- `lat_mean`, `lon_mean`
- `temp_proxy`

### Histórico

- `days_since_clean`
- `fouling_stage` (0-3)

### Consumo

- `CONSUMED_QUANTITY` (quando disponível)

## 📈 Performance do Modelo

```
════════════════════════════════════════════════════════════
 PERFORMANCE ENSEMBLE
════════════════════════════════════════════════════════════
  MAE:  0.0048
  RMSE: 0.0170
  R²:   0.9996
════════════════════════════════════════════════════════════
```

### Performance Individual dos Modelos

| Modelo            | MAE    | RMSE   | R²     |
| ----------------- | ------ | ------ | ------ |
| XGBoost           | 0.0070 | 0.0251 | 0.9992 |
| LightGBM          | 0.0072 | 0.0222 | 0.9994 |
| Random Forest     | 0.0045 | 0.0194 | 0.9995 |
| Gradient Boosting | 0.0042 | 0.0152 | 0.9997 |

## 💰 Impacto Econômico

### Exemplo de Análise da Frota

```
Custo Extra Médio/Dia:  $5,145.33
Custo Extra Médio/Mês:  $154,359.77
Custo Extra Médio/Ano:  $1,878,043.86
CO2 Extra Médio/Ano:    8,997.27 toneladas
```

### Penalidades por Fouling Rating

| Rating | Penalidade | Impacto       |
| ------ | ---------- | ------------- |
| 0      | 0%         | Sem impacto   |
| 1      | 6.5%       | Leve          |
| 2      | 10%        | Moderado      |
| 3      | 15%        | Significativo |
| 4      | 21.5-25%   | Crítico       |

## 🚀 Como Usar

### 1. Instalação

```bash
# Clone o repositório
git clone <repository-url>
cd nexus_transpetro

# Instale as dependências
pip install -r requirements.txt
```

### 2. Preparação dos Dados

Coloque os seguintes arquivos na pasta `Hackathon Transpetro/`:

- `ResultadoQueryEventos.csv`
- `ResultadoQueryConsumo.csv`
- `Dados navios Hackathon.xlsx`
- `Relatorios IWS.xlsx`
- `Dados AIS frota TP.zip` (ou pasta descompactada)

### 3. Execução

```bash
# Abra o Jupyter Notebook
jupyter notebook Analise_Bioincrustacao_Frota_v2.ipynb

# Ou execute no Google Colab
# (o notebook fará download automático dos dados do Google Drive)
```

### 4. Outputs Gerados

- `model_xgboost_v2.pkl` - Modelo XGBoost treinado
- `model_lightgbm_v2.pkl` - Modelo LightGBM treinado
- `model_randomforest_v2.pkl` - Modelo Random Forest treinado
- `model_gradientboosting_v2.pkl` - Modelo Gradient Boosting treinado
- `model_metadata_v2.pkl` - Metadados (features, pesos, métricas)
- `fouling_por_navio.csv` - Relatório detalhado da frota

## 📋 Estrutura do Projeto

```
nexus_transpetro/
├── Analise_Bioincrustacao_Frota_v2.ipynb  # Notebook principal
├── README.md                               # Este arquivo
├── requirements.txt                        # Dependências Python
├── Recomendações.md                        # Documentação do desafio
├── Hackathon Transpetro/                   # Dados (não versionado)
│   ├── ResultadoQueryEventos.csv
│   ├── ResultadoQueryConsumo.csv
│   ├── Dados navios Hackathon.xlsx
│   ├── Relatorios IWS.xlsx
│   └── Dados AIS frota TP/
└── outputs/                                # Modelos e resultados
    ├── model_*.pkl
    └── fouling_por_navio.csv
```

## 🔍 Metodologia Científica

### Base Teórica

1. **Fouling Rating IMO MEPC.378(80)**

   - Escala internacional padronizada (0-4)
   - Adotada pela NORMAM 401 (Brasil) em junho/2025

2. **Fatores de Risco Considerados**

   - **Idle Time**: 0-14 dias (biofilme), 2-6 semanas (colonização), >6 semanas (comunidades complexas)
   - **Velocidade**: <5 nós (alto risco), 5-10 nós (moderado), >12 nós (baixo risco)
   - **Regiões Biogeográficas**: Norte (alto), Nordeste (moderado), Sudeste-Sul (baixo)
   - **Temperatura**: Águas quentes aceleram bioincrustação

3. **Validação Temporal**
   - Split 80/20 respeitando ordem cronológica
   - Evita data leakage
   - Simula cenário real de predição

### Inovações da Solução

1. **Features Avançadas**

   - `low_shear_exposure`: Combina idle time e velocity risk
   - `biofouling_risk_score`: Score composto ponderado
   - `operation_continuity`: Perfil operacional do navio

2. **Ensemble Inteligente**

   - Pesos baseados em performance (1/MAE)
   - Combina modelos complementares
   - Reduz overfitting

3. **Análise de Cenários**
   - Simulação de custos futuros
   - Comparação limpar vs não limpar
   - ROI de limpezas

## 📊 Exemplo de Resultados

### Análise da Frota (15 navios analisados)

```
Distribuição por Categoria (Escala IMO):
  🟢 0-1 (Sem/Micro):      1 navios (  6.7%)
  🟡 1-2 (Micro):          1 navios (  6.7%)
  🟠 2-3 (Leve):           2 navios ( 13.3%)
  🔴 3-4 (Moderada):       6 navios ( 40.0%)
  🔴 4   (Pesada):         5 navios ( 33.3%)

⚠️ AÇÕES REQUERIDAS:
  🔴 11 navios precisam LIMPEZA URGENTE (Fouling ≥ 3.0)
  🟠 2 navios precisam LIMPEZA REATIVA (Fouling 2.0-3.0)
  🟡 1 navios precisam LIMPEZA PROATIVA (Fouling 1.0-2.0)

IMPACTO ECONÔMICO TOTAL DA FROTA:
  Custo Extra Total/Ano: $26,280,162
  Custo Extra Médio/Navio: $1,752,011
  CO2 Extra Total/Ano: 125,901 toneladas
  CO2 Extra Médio/Navio: 8,393 toneladas
```

## 🎓 Referências

1. **IMO (International Maritime Organization)**

   - MEPC.378(80) - Fouling Rating Scale
   - Estratégia de Descarbonização do Transporte Marítimo (2023)
   - Meta: Net-Zero até 2050

2. **NORMAM-401 (Marinha do Brasil)**
   - Regulamentação de bioincrustação
   - Regiões biogeográficas brasileiras
   - Requisitos de limpeza

## 👥 Equipe NEXUS

Desenvolvido para o Hackathon Transpetro 2025

## 📄 Licença

Este projeto foi desenvolvido para fins educacionais e de competição no Hackathon Transpetro 2025.

**Equipe NEXUS** | Hackathon Transpetro 2025
