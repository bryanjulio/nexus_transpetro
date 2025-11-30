# 📊 Diagramas do Sistema NEXUS

## Visualizações em Mermaid

Este documento contém diagramas que explicam visualmente a arquitetura e fluxo do sistema.

---

## 1. Fluxo Geral do Sistema

```mermaid
graph TB
    Start([Início]) --> LoadData[Carregar Dados]

    LoadData --> Data1[(Eventos<br/>50.904 registros)]
    LoadData --> Data2[(AIS<br/>415.724 registros)]
    LoadData --> Data3[(Consumo<br/>87.737 registros)]
    LoadData --> Data4[(IWS<br/>29 inspeções)]
    LoadData --> Data5[(Navios<br/>21 embarcações)]

    Data1 --> Preprocess[Pré-processamento]
    Data2 --> Preprocess
    Data3 --> Preprocess
    Data4 --> Preprocess
    Data5 --> Preprocess

    Preprocess --> AggAIS[Agregação AIS<br/>por Evento]
    AggAIS --> Features[Engenharia de Features<br/>26 features]

    Features --> IWS[Processar IWS<br/>Days Since Clean]
    IWS --> Target[Criar Target<br/>Fouling Rating 0-4]

    Target --> Merge[Merge Consumo<br/>e Navios]
    Merge --> PrepML[Preparar Dataset ML<br/>4.639 registros]

    PrepML --> Split[Split Temporal<br/>80% Treino / 20% Teste]

    Split --> Train[Treinamento Ensemble]
    Train --> XGB[XGBoost]
    Train --> LGB[LightGBM]
    Train --> RF[Random Forest]
    Train --> GB[Gradient Boosting]

    XGB --> Ensemble[Ensemble Ponderado<br/>R² 0.9996]
    LGB --> Ensemble
    RF --> Ensemble
    GB --> Ensemble

    Ensemble --> Predict[Predições]
    Predict --> Impact[Cálculo Impacto<br/>Econômico]
    Predict --> Scenarios[Simulação<br/>Cenários]

    Impact --> Report[Relatório da Frota<br/>fouling_por_navio.csv]
    Scenarios --> Report

    Report --> End([Fim])

    style Start fill:#90EE90
    style End fill:#90EE90
    style Ensemble fill:#FFD700
    style Report fill:#87CEEB
```

---

## 2. Pipeline de Dados Detalhado

```mermaid
flowchart LR
    subgraph Input["📥 ENTRADA DE DADOS"]
        E[Eventos<br/>Navigation]
        A[AIS<br/>Tracking]
        C[Consumo<br/>Fuel]
        I[IWS<br/>Inspections]
        N[Navios<br/>Ships]
    end

    subgraph Process["⚙️ PROCESSAMENTO"]
        P1[Parse Dates]
        P2[Clean Names]
        P3[Handle Missing]

        AG[Agregar AIS<br/>por Evento]

        F1[Idle Time<br/>Features]
        F2[Velocity<br/>Risk]
        F3[Region<br/>Risk]
        F4[Temp<br/>Proxy]
        F5[Days Since<br/>Clean]

        TG[Target<br/>Generation]
    end

    subgraph ML["🤖 MACHINE LEARNING"]
        TS[Time Series<br/>Split 80/20]

        M1[XGBoost]
        M2[LightGBM]
        M3[Random<br/>Forest]
        M4[Gradient<br/>Boosting]

        EN[Ensemble<br/>Weighted]
    end

    subgraph Output["📤 SAÍDA"]
        PR[Predições<br/>Fouling 0-4]
        EC[Impacto<br/>Econômico]
        SC[Cenários<br/>Futuros]
        RP[Relatório<br/>CSV]
    end

    E --> P1
    A --> P1
    C --> P1
    I --> P1
    N --> P2

    P1 --> P3
    P2 --> P3
    P3 --> AG

    AG --> F1
    AG --> F2
    AG --> F3
    AG --> F4
    P3 --> F5

    F1 --> TG
    F2 --> TG
    F3 --> TG
    F4 --> TG
    F5 --> TG

    TG --> TS

    TS --> M1
    TS --> M2
    TS --> M3
    TS --> M4

    M1 --> EN
    M2 --> EN
    M3 --> EN
    M4 --> EN

    EN --> PR
    PR --> EC
    PR --> SC
    EC --> RP
    SC --> RP

    style Input fill:#E8F5E9
    style Process fill:#FFF3E0
    style ML fill:#E3F2FD
    style Output fill:#F3E5F5
```

---

## 3. Engenharia de Features

```mermaid
graph TD
    subgraph Raw["Dados Brutos"]
        AIS[Dados AIS<br/>speed, lat, lon]
        EVT[Eventos<br/>duration, distance]
        IWS_D[IWS<br/>cleaning dates]
    end

    subgraph Aggregated["Features Agregadas"]
        AIS --> SM[speed_mean]
        AIS --> SS[speed_std]
        AIS --> SMIN[speed_min]
        AIS --> SMAX[speed_max]
        AIS --> FS[frac_stop]
        AIS --> FLS[frac_low_speed]
        AIS --> LAT[lat_mean]
        AIS --> LON[lon_mean]
    end

    subgraph Advanced["Features Avançadas"]
        FS --> ITR[idle_time_ratio]
        ITR --> ID[idle_days]
        FLS --> LSD[low_speed_days]

        SM --> VR[velocity_risk<br/>0-3]

        ITR --> OC[operation_continuity]
        SS --> SV[speed_variability]

        ID --> LSE[low_shear_exposure]
        VR --> LSE

        LAT --> BR[bio_region<br/>Norte/Nordeste/Sul]
        BR --> RR[region_risk<br/>1-3]

        LAT --> TP[temp_proxy]
        TP --> TR[temp_risk<br/>0-1]

        IWS_D --> DSC[days_since_clean]
        DSC --> FS_ST[fouling_stage<br/>0-3]
    end

    subgraph Target["Target"]
        VR --> FR[fouling_rating<br/>0-4]
        ITR --> FR
        TR --> FR
        RR --> FR
        DSC --> FR

        FR --> BRS[biofouling_risk_score<br/>0-1]
    end

    style Raw fill:#FFEBEE
    style Aggregated fill:#FFF9C4
    style Advanced fill:#C8E6C9
    style Target fill:#BBDEFB
```

---

## 4. Modelo Ensemble

```mermaid
graph TB
    subgraph Data["Dataset ML"]
        X[Features X<br/>26 features]
        Y[Target y<br/>fouling_rating]
    end

    subgraph Split["Validação Temporal"]
        X --> TR[Training Set<br/>80% - 3.711]
        Y --> TR
        X --> TE[Test Set<br/>20% - 928]
        Y --> TE
    end

    subgraph Models["Modelos Base"]
        TR --> XGB[XGBoost<br/>n=300, lr=0.03]
        TR --> LGB[LightGBM<br/>n=300, lr=0.03]
        TR --> RF[Random Forest<br/>n=200, depth=10]
        TR --> GB[Gradient Boost<br/>n=200, lr=0.05]
    end

    subgraph Predictions["Predições"]
        XGB --> P1[y_pred_xgb<br/>MAE: 0.0070]
        LGB --> P2[y_pred_lgb<br/>MAE: 0.0072]
        RF --> P3[y_pred_rf<br/>MAE: 0.0045]
        GB --> P4[y_pred_gb<br/>MAE: 0.0042]
    end

    subgraph Weights["Cálculo de Pesos"]
        P1 --> W[Pesos = 1/MAE<br/>Normalizado]
        P2 --> W
        P3 --> W
        P4 --> W

        W --> W1[w_xgb: 0.23]
        W --> W2[w_lgb: 0.22]
        W --> W3[w_rf: 0.28]
        W --> W4[w_gb: 0.27]
    end

    subgraph Ensemble["Ensemble Final"]
        P1 --> ENS[y_pred = Σ wi × yi]
        P2 --> ENS
        P3 --> ENS
        P4 --> ENS
        W1 --> ENS
        W2 --> ENS
        W3 --> ENS
        W4 --> ENS

        ENS --> METRICS[MAE: 0.0048<br/>RMSE: 0.0170<br/>R²: 0.9996]
    end

    TE --> EVAL[Avaliação]
    METRICS --> EVAL

    style Data fill:#E8F5E9
    style Models fill:#FFF3E0
    style Ensemble fill:#FFD700
    style METRICS fill:#4CAF50,color:#FFF
```

---

## 5. Cálculo de Impacto Econômico

```mermaid
flowchart TD
    Start([Fouling Rating<br/>Predito]) --> Map{Mapear<br/>Penalidade}

    Map -->|0| P0[0%]
    Map -->|1| P1[6.5%]
    Map -->|2| P2[10%]
    Map -->|3| P3[15%]
    Map -->|4| P4[21.5-25%]

    P0 --> Calc[Calcular Extra Fuel<br/>= baseline × penalty]
    P1 --> Calc
    P2 --> Calc
    P3 --> Calc
    P4 --> Calc

    Calc --> Cost[Custo Extra<br/>= extra_fuel × $650/ton]

    Cost --> Day[Custo/Dia]
    Cost --> Month[Custo/Mês<br/>× 30]
    Cost --> Year[Custo/Ano<br/>× 365]

    Calc --> CO2[CO2 Extra<br/>= extra_fuel × 3.114]
    CO2 --> CO2Year[CO2/Ano<br/>× 365]

    Day --> Report[Relatório<br/>Impacto]
    Month --> Report
    Year --> Report
    CO2Year --> Report

    Report --> End([Fim])

    style Start fill:#90EE90
    style Calc fill:#FFD700
    style Report fill:#87CEEB
    style End fill:#90EE90
```

---

## 6. Simulação de Cenários

```mermaid
graph TB
    Start([Navio com<br/>Fouling Atual]) --> Input[Input:<br/>current_fouling<br/>days_since_clean<br/>baseline_consumption]

    Input --> Scenario1[Cenário 1:<br/>Não Fazer Nada]
    Input --> Scenario2[Cenário 2:<br/>Limpar Agora]

    subgraph S1["Cenário 1: Não Fazer Nada"]
        Scenario1 --> Evol1[Evolução Fouling<br/>+180 dias]
        Evol1 --> Future1[future_fouling<br/>= min current + 180/90, 4.0]
        Future1 --> Impact1[Impacto Atual]
        Future1 --> Impact2[Impacto Futuro]
        Impact1 --> Avg1[Custo Médio<br/>180 dias]
        Impact2 --> Avg1
        Avg1 --> Total1[Custo Total 1]
    end

    subgraph S2["Cenário 2: Limpar Agora"]
        Scenario2 --> Clean[Limpeza]
        Clean --> Cost2[Custo Limpeza<br/>$50k]
        Clean --> Down2[Downtime<br/>$120k]
        Clean --> Post[Post-Clean<br/>fouling = 0.5]
        Post --> Evol2[Evolução<br/>+180 dias]
        Evol2 --> Future2[future_fouling<br/>= min 0.5 + 180/120, 2.5]
        Future2 --> Impact3[Impacto Post]
        Future2 --> Impact4[Impacto Futuro]
        Impact3 --> Avg2[Custo Médio<br/>180 dias]
        Impact4 --> Avg2
        Cost2 --> Total2[Custo Total 2]
        Down2 --> Total2
        Avg2 --> Total2
    end

    Total1 --> Compare{Comparar<br/>Custos}
    Total2 --> Compare

    Compare --> Decision[Recomendação:<br/>Menor Custo]
    Compare --> Savings[Economia:<br/>Total1 - Total2]

    Decision --> Output[Output:<br/>Ação + Economia]
    Savings --> Output

    Output --> End([Fim])

    style Start fill:#90EE90
    style S1 fill:#FFCDD2
    style S2 fill:#C8E6C9
    style Decision fill:#FFD700
    style End fill:#90EE90
```

---

## 7. Fluxo de Decisão por Urgência

```mermaid
graph TD
    Start([Fouling Rating<br/>do Navio]) --> Check{Avaliar<br/>Rating}

    Check -->|< 1.0| OK[🟢 OK<br/>Sem ação necessária]
    Check -->|1.0-2.0| Proactive[🟡 Limpeza Proativa<br/>Monitorar<br/>Planejar em 90 dias]
    Check -->|2.0-3.0| Reactive[🟠 Limpeza Reativa<br/>Agendar em 60 dias<br/>Compliance em risco]
    Check -->|3.0-4.0| Urgent[🔴 Limpeza Urgente<br/>Agendar em 30 dias<br/>Alto impacto econômico]
    Check -->|>= 4.0| Critical[🔴 Limpeza CRÍTICA<br/>IMEDIATO<br/>Não conformidade]

    OK --> Monitor[Continuar<br/>Monitoramento]

    Proactive --> Plan1[Planejar Limpeza<br/>Proativa]
    Plan1 --> ROV[ROV com<br/>Jato d'água]

    Reactive --> Plan2[Planejar Limpeza<br/>Reativa com Captura]
    Plan2 --> Check2{AFS<br/>Deteriorado?}
    Check2 -->|Sim| Dock[Docagem<br/>+ Reaplicação AFS]
    Check2 -->|Não| InWater[Limpeza<br/>In-Water]

    Urgent --> Plan3[Agendar<br/>Urgente]
    Plan3 --> Simulate[Simular<br/>Cenários]
    Simulate --> Decision1[Decisão:<br/>Limpar ou Não]

    Critical --> Immediate[Ação<br/>Imediata]
    Immediate --> Compliance[Verificar<br/>Compliance]
    Compliance --> Block{Bloquear<br/>Travessia?}
    Block -->|Sim| NoTravel[❌ Não pode<br/>atravessar regiões]
    Block -->|Não| Emergency[Limpeza<br/>Emergencial]

    Monitor --> End([Fim])
    ROV --> End
    Dock --> End
    InWater --> End
    Decision1 --> End
    NoTravel --> End
    Emergency --> End

    style Start fill:#90EE90
    style OK fill:#C8E6C9
    style Proactive fill:#FFF59D
    style Reactive fill:#FFCC80
    style Urgent fill:#EF9A9A
    style Critical fill:#E57373
    style End fill:#90EE90
```

---

## 8. Arquitetura de Sistema (Deployment Futuro)

```mermaid
graph TB
    subgraph Sources["📊 Fontes de Dados"]
        DB1[(Database<br/>Eventos)]
        DB2[(Database<br/>AIS)]
        DB3[(Database<br/>Consumo)]
        API1[API IWS]
    end

    subgraph ETL["⚙️ ETL Pipeline"]
        Extract[Extract]
        Transform[Transform<br/>+ Features]
        Load[Load to<br/>Data Lake]
    end

    subgraph ML["🤖 ML Service"]
        Models[Modelos<br/>Treinados]
        Predict[Prediction<br/>Service]
        Retrain[Retraining<br/>Scheduler]
    end

    subgraph API["🔌 API Layer"]
        REST[REST API<br/>FastAPI]
        Auth[Authentication]
        Cache[Redis Cache]
    end

    subgraph App["📱 Applications"]
        Dashboard[Dashboard<br/>Streamlit]
        Mobile[Mobile App]
        Alerts[Alert System]
    end

    subgraph Storage["💾 Storage"]
        S3[Object Storage<br/>S3/Blob]
        Postgres[(PostgreSQL<br/>Results)]
        Mongo[(MongoDB<br/>Logs)]
    end

    DB1 --> Extract
    DB2 --> Extract
    DB3 --> Extract
    API1 --> Extract

    Extract --> Transform
    Transform --> Load
    Load --> S3

    S3 --> Models
    Models --> Predict
    Predict --> REST

    S3 --> Retrain
    Retrain --> Models

    REST --> Auth
    Auth --> Cache
    Cache --> Dashboard
    Cache --> Mobile
    Cache --> Alerts

    Predict --> Postgres
    Dashboard --> Postgres

    Alerts --> Email[📧 Email]
    Alerts --> SMS[📱 SMS]
    Alerts --> Teams[💬 Teams]

    Dashboard --> Mongo
    Mobile --> Mongo

    style Sources fill:#E8F5E9
    style ETL fill:#FFF3E0
    style ML fill:#E3F2FD
    style API fill:#F3E5F5
    style App fill:#FFE0B2
    style Storage fill:#CFD8DC
```

---

## 9. Timeline de Desenvolvimento

```mermaid
gantt
    title Roadmap NEXUS - Sistema de Predição de Bioincrustação
    dateFormat YYYY-MM-DD
    section Fase 1: MVP
    Modelo de Predição           :done, mvp1, 2025-11-01, 2025-11-30
    Análise Impacto Econômico    :done, mvp2, 2025-11-15, 2025-11-30
    Relatório da Frota           :done, mvp3, 2025-11-25, 2025-11-30
    Dashboard Streamlit          :active, mvp4, 2025-12-01, 2025-12-15

    section Fase 2: Produção
    API REST                     :prod1, 2025-12-15, 2026-01-15
    Sistema de Alertas           :prod2, 2026-01-01, 2026-02-01
    Dashboard Tempo Real         :prod3, 2026-01-15, 2026-02-15
    Integração Sistemas          :prod4, 2026-02-01, 2026-03-01

    section Fase 3: Otimização
    Deep Learning LSTM           :opt1, 2026-03-01, 2026-04-15
    Dados Meteorológicos         :opt2, 2026-03-15, 2026-05-01
    Otimização Cronograma        :opt3, 2026-04-01, 2026-05-15
    Módulo BI                    :opt4, 2026-05-01, 2026-06-01

    section Fase 4: Expansão
    Expansão Frota Completa      :exp1, 2026-06-01, 2026-07-15
    Predição Trajetória          :exp2, 2026-07-01, 2026-08-15
    Roteamento Otimizado         :exp3, 2026-08-01, 2026-09-15
    App Mobile                   :exp4, 2026-09-01, 2026-10-15
```

---

## 10. Mapa de Stakeholders

```mermaid
mindmap
  root((Sistema NEXUS))
    Operacional
      Gestores de Frota
        Planejamento
        Priorização
        Compliance
      Capitães
        Monitoramento
        Relatórios
      Manutenção
        Cronograma
        Recursos
    Financeiro
      CFO
        ROI
        Orçamento
      Controladoria
        Custos
        Savings
      Procurement
        Limpezas
        Contratos
    Ambiental
      Sustentabilidade
        CO2
        ESG
        Metas IMO
      Compliance
        NORMAM-401
        Regulatório
    Técnico
      TI
        Infraestrutura
        Integração
      Data Science
        Modelos
        Features
      DevOps
        Deploy
        Monitoring
```

---

## Como Usar os Diagramas

### No GitHub/GitLab

Os diagramas Mermaid são renderizados automaticamente em arquivos Markdown.

### Em Apresentações

1. Use ferramentas como [Mermaid Live Editor](https://mermaid.live/)
2. Exporte como PNG/SVG
3. Insira nas apresentações

### Em Documentação

Cole o código Mermaid diretamente em:

- GitHub/GitLab README
- Confluence
- Notion
- Obsidian

---

**Equipe NEXUS** | Hackathon Transpetro 2025
