# C4 Architecture Model - Plataforma de Trading Algorítmico

**Versión:** 2.0  
**Fecha:** 3 de febrero de 2026  
**Documento Base:** Documento Rector v2.1  
**Rol Responsable:** Tech Lead  
**Estado:** APROBADO CON OPTIMIZACIONES

---

## 1. Introducción

Este documento presenta la arquitectura del sistema de trading algorítmico utilizando el modelo C4 (Contexto, Contenedores, Componentes y Código). El modelo C4 proporciona una forma escalonada de describir la arquitectura de software en diferentes niveles de detalle, ideal para sistemas complejos como plataformas de trading.

**Cambios en v2.0:**
- Migración de diagramas ASCII a Mermaid.js para mejor visualización
- Incorporación de optimizaciones de resiliencia y seguridad
- Adición de consideraciones de observabilidad y monitoreo
- Actualización del roadmap de escalabilidad

---

## 2. C4 Context Diagram

El diagrama de contexto muestra el sistema en su entorno, incluyendo los actores humanos y otros sistemas con los que interactúa.

```mermaid
flowchart TD
    subgraph "External Systems"
        MT5[MetaTrader 5 Server]
        MarketData[Market Data Providers]
        NotificationServices[Notification Services<br/>Email/SMS/Push]
    end

    subgraph "Plataforma de Trading Algorítmico"
        PTA[Plataforma de Trading Algorítmico]
    end

    subgraph "Human Actors"
        Trader[Trader<br/>Persona]
        Admin[Administrator<br/>Persona]
        RiskManager[Risk Manager<br/>Persona]
    end

    Trader -->|Web/WebSocket<br/>Operaciones de Trading| PTA
    Admin -->|Admin Panel<br/>Gestión del Sistema| PTA
    RiskManager -->|Risk Dashboard<br/>Supervisión en Tiempo Real| PTA
    
    PTA -->|MT5 API<br/>Ejecución y Datos| MT5
    PTA -->|Market Data API<br/>Datos en Tiempo Real| MarketData
    PTA -->|Notification API<br/>Alertas y Notificaciones| NotificationServices

    style PTA fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    style MT5 fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style Trader fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
```

**Descripción:**
- **Plataforma de Trading Algorítmico**: Sistema principal que permite trading manual y automático
- **Trader**: Usuario que opera el sistema para realizar trading manual o configurar estrategias
- **Administrador**: Personal encargado de la gestión del sistema y supervisión de riesgos
- **Risk Manager**: Especialista en monitoreo y control de exposición al riesgo
- **Servidor MT5**: Sistema externo de MetaTrader 5 que ejecuta las órdenes y provee datos de mercado
- **Market Data Providers**: Fuentes adicionales de datos financieros
- **Notification Services**: Servicios externos para envío de alertas

---

## 3. C4 Container Diagram

El diagrama de contenedores muestra los principales contenedores de software que forman parte del sistema.

```mermaid
flowchart TB
    subgraph "Frontend Layer"
        FletUI[Flet UI<br/>Python/Flet Application]
        REST_API[REST API Gateway<br/>FastAPI Application]
        WS_API[WebSocket Gateway<br/>Real-time Updates]
    end

    subgraph "Application Services"
        TradingService[Trading Service<br/>Order Management]
        RiskService[Risk Management Service<br/>Validation & Monitoring]
        AuthService[Authentication Service<br/>JWT/OAuth2]
        StrategyService[Strategy Service<br/>Execution Framework]
    end

    subgraph "Domain Core"
        TradingDomain[Trading Domain Model<br/>Orders, Positions]
        RiskDomain[Risk Domain Model<br/>Limits, Exposure]
        UserDomain[User Domain Model<br/>Accounts, Permissions]
    end

    subgraph "Infrastructure"
        DB[(PostgreSQL 14+<br/>Primary Database)]
        Cache[(Redis Cluster<br/>Caching & Session)]
        MT5Adapter[MT5 Adapter<br/>Connection Pool, Circuit Breaker]
        MessageQueue[Message Broker<br/>RabbitMQ/Kafka Future]
        ExternalAPIs[External APIs Gateway<br/>Market Data, News]
    end

    subgraph "External Systems"
        MT5_Server[MetaTrader 5 Server]
        Data_Providers[Market Data Providers]
    end

    %% Frontend connections
    FletUI -->|HTTP/HTTPS| REST_API
    FletUI -->|WebSocket| WS_API
    
    %% API to Services
    REST_API -->|gRPC/HTTP| TradingService
    REST_API -->|gRPC/HTTP| RiskService
    REST_API -->|gRPC/HTTP| AuthService
    WS_API -->|WebSocket| TradingService
    
    %% Services to Domain
    TradingService -->|Domain Events| TradingDomain
    RiskService -->|Domain Events| RiskDomain
    AuthService -->|Domain Events| UserDomain
    
    %% Domain to Infrastructure
    TradingDomain -->|Repository Pattern| DB
    RiskDomain -->|Repository Pattern| DB
    UserDomain -->|Repository Pattern| DB
    
    TradingDomain -->|Caching Strategy| Cache
    TradingService -->|Circuit Breaker Pattern| MT5Adapter
    
    %% Infrastructure to External
    MT5Adapter -->|MT5 Protocol| MT5_Server
    ExternalAPIs -->|API Gateway| Data_Providers
    
    %% Async Communication
    TradingService -->|Async Events| MessageQueue
    RiskService -->|Async Events| MessageQueue
    
    style FrontendLayer fill:#e8f5e8,stroke:#2e7d32
    style ApplicationServices fill:#fff3e0,stroke:#ef6c00
    style DomainCore fill:#f3e5f5,stroke:#7b1fa2
    style Infrastructure fill:#e3f2fd,stroke:#1565c0
    style ExternalSystems fill:#fce4ec,stroke:#c2185b
```

---

## 4. C4 Component Diagram - Trading Engine

Diagrama detallado del componente principal de Trading Engine:

```mermaid
flowchart TD
    subgraph "Trading Engine Core"
        OrderManager[Order Manager<br/>- submit_order()<br/>- modify_order()<br/>- cancel_order()]
        
        PositionManager[Position Manager<br/>- get_position()<br/>- update_position()<br/>- close_position()]
        
        StrategyRunner[Strategy Runner<br/>- run_strategy()<br/>- stop_strategy()<br/>- pause_resume()]
    end

    subgraph "Integration Components"
        MT5Connector[MT5 Connector<br/>- connect()<br/>- disconnect()<br/>- reconnect()<br/>Circuit Breaker Pattern]
        
        RiskValidator[Risk Validator<br/>- validate_risk()<br/>- check_limits()<br/>- risk_alert()]
        
        OrderExecution[Order Execution Engine<br/>- execute_order()<br/>- handle_fill()<br/>- handle_rejection()]
    end

    subgraph "Support Components"
        OrderBook[Order Book<br/>- maintain()<br/>- sync_orders()<br/>- match_orders()]
        
        MarketDataFeed[Market Data Feed<br/>- subscribe()<br/>- unsubscribe()<br/>- process_feed()<br/>Cache Strategy]
        
        ExecutionReport[Execution Reporter<br/>- generate_report()<br/>- send_notifications()<br/>- store_history()]
    end

    subgraph "Monitoring & Observability"
        MetricsCollector[Metrics Collector<br/>- track_latency()<br/>- count_orders()<br/>- monitor_fill_rate()]
        
        HealthChecker[Health Checker<br/>- service_health()<br/>- dependency_status()<br/>- circuit_breaker_state()]
    end

    %% Core interactions
    OrderManager -->|Send for validation| RiskValidator
    OrderManager -->|Execute validated order| OrderExecution
    OrderManager -->|Update state| OrderBook
    
    PositionManager -->|Synchronize| MT5Connector
    StrategyRunner -->|Generate signals| OrderManager
    
    %% Integration flows
    MT5Connector -->|Stream prices| MarketDataFeed
    MT5Connector -->|Execute orders| OrderExecution
    
    RiskValidator -->|Check market conditions| MarketDataFeed
    
    %% Support flows
    MarketDataFeed -->|Update prices| OrderBook
    OrderExecution -->|Report fills| ExecutionReport
    
    %% Monitoring flows
    OrderExecution -->|Emit metrics| MetricsCollector
    MT5Connector -->|Health status| HealthChecker
    RiskValidator -->|Risk metrics| MetricsCollector
    
    %% External dependencies
    OrderExecution -.->|Async notification| MessageQueue
    ExecutionReport -.->|Store in DB| DB[(Database)]
    
    style TradingEngineCore fill:#e1f5fe,stroke:#0277bd
    style IntegrationComponents fill:#f3e5f5,stroke:#7b1fa2
    style SupportComponents fill:#e8f5e8,stroke:#2e7d32
    style MonitoringObservability fill:#fff3e0,stroke:#f57c00
```

**Interacciones Optimizadas:**
- **Circuit Breaker**: MT5Connector incluye patrón Circuit Breaker para manejo de fallos
- **Caching Estratégico**: MarketDataFeed implementa cache L1/L2 para datos frecuentes
- **Métricas en Tiempo Real**: Todos los componentes emiten métricas para observabilidad
- **Comunicación Asíncrona**: Notificaciones se envían a Message Queue para desacoplamiento

---

## 5. C4 Component Diagram - Risk Management

Diagrama detallado del componente de Gestión de Riesgo:

```mermaid
flowchart LR
    subgraph "Validation Layer"
        AccountRisk[Account Risk Validator<br/>- daily_loss_limit()<br/>- weekly_loss_limit()<br/>- equity_protection()]
        
        StrategyRisk[Strategy Risk Validator<br/>- max_exposure()<br/>- correlation_check()<br/>- kill_switch()]
        
        MarketRisk[Market Risk Validator<br/>- volatility_check()<br/>- market_halt_detection()<br/>- news_impact_analysis()]
    end

    subgraph "Monitoring Layer"
        PositionMonitor[Position Monitor<br/>- track_positions()<br/>- alert_changes()<br/>- enforce_stop_loss()]
        
        ExposureMonitor[Exposure Monitor<br/>- calculate_exposure()<br/>- alert_limit_breaches()<br/>- risk_metrics_collection()]
        
        DrawdownMonitor[Drawdown Monitor<br/>- calculate_drawdown()<br/>- trigger_protection()<br/>- risk_metrics_aggregation()]
    end

    subgraph "Control Layer"
        KillSwitch[Kill Switch Controller<br/>- global_kill()<br/>- strategy_kill()<br/>- auto_reset()]
        
        EmergencyStop[Emergency Stop Controller<br/>- emergency_trigger()<br/>- account_stop()<br/>- position_close_all()]
        
        OverrideControl[Override Control<br/>- manual_override()<br/>- permission_matrix()<br/>- approval_workflow()]
    end

    subgraph "Data Sources"
        MarketData[Market Data Stream]
        PositionData[Position Database]
        StrategyData[Strategy Configuration]
        UserData[User/Account Data]
    end

    subgraph "Output Actions"
        Alerts[Alert System<br/>Real-time Notifications]
        TradingControls[Trading Controls<br/>Order Block/Allow]
        Reports[Risk Reports<br/>Dashboards & Analytics]
    end

    %% Data flow from sources
    MarketData --> MarketRisk
    PositionData --> PositionMonitor
    StrategyData --> StrategyRisk
    UserData --> AccountRisk
    
    %% Validation to Monitoring
    AccountRisk -->|Validation rules| PositionMonitor
    StrategyRisk -->|Strategy limits| ExposureMonitor
    MarketRisk -->|Market conditions| DrawdownMonitor
    
    %% Monitoring to Control
    PositionMonitor -->|Position alerts| KillSwitch
    ExposureMonitor -->|Exposure alerts| EmergencyStop
    DrawdownMonitor -->|Drawdown alerts| OverrideControl
    
    %% Control to Actions
    KillSwitch -->|Stop commands| TradingControls
    EmergencyStop -->|Emergency actions| TradingControls
    OverrideControl -->|Override decisions| TradingControls
    
    KillSwitch -->|Alert notifications| Alerts
    EmergencyStop -->|Emergency notifications| Alerts
    OverrideControl -->|Override notifications| Alerts
    
    %% Reporting
    PositionMonitor -->|Position metrics| Reports
    ExposureMonitor -->|Exposure metrics| Reports
    DrawdownMonitor -->|Drawdown metrics| Reports
    
    %% Async communication
    KillSwitch -.->|Event publishing| MessageQueue[Message Queue]
    EmergencyStop -.->|Event publishing| MessageQueue
    
    style ValidationLayer fill:#fff3e0,stroke:#ef6c00
    style MonitoringLayer fill:#e8f5e8,stroke:#2e7d32
    style ControlLayer fill:#fce4ec,stroke:#c2185b
    style DataSources fill:#e3f2fd,stroke:#1565c0
    style OutputActions fill:#f3e5f5,stroke:#7b1fa2
```

**Mejoras de Seguridad:**
- **Validación en Múltiples Capas**: Tres niveles de validación independientes
- **Circuitos de Corte**: Múltiples mecanismos de emergencia redundantes
- **Auditoría Completa**: Todos los eventos de riesgo son registrados y reportados
- **Control de Sobreescritura**: Workflow de aprobación para operaciones críticas

---

## 6. C4 Component Diagram - Strategy Framework

Diagrama detallado del componente de Framework de Estrategias:

```mermaid
flowchart TB
    subgraph "Strategy Core"
        StrategyBase[Strategy Base Class<br/>- initialize()<br/>- on_tick()<br/>- on_bar()<br/>- on_order()]
        
        StrategyLoader[Strategy Loader<br/>- load_strategy()<br/>- validate_code()<br/>- compile_code()<br/>- cache_strategy()]
        
        StrategyExecutor[Strategy Executor<br/>- start_strategy()<br/>- stop_strategy()<br/>- pause_resume()<br/>- get_status()]
    end

    subgraph "Analytics Engine"
        IndicatorCalc[Indicator Calculator<br/>- technical_indicators()<br/>- fundamental_ratios()<br/>- custom_indicators()<br/>Cached Results]
        
        SignalGenerator[Signal Generator<br/>- generate_signals()<br/>- validate_signals()<br/>- merge_signals()<br/>Confidence Scoring]
        
        RiskCalculator[Risk Calculator<br/>- position_sizing()<br/>- stop_loss_calculation()<br/>- take_profit_calculation()]
    end

    subgraph "Backtesting System"
        BacktestEngine[Backtest Engine<br/>- run_backtest()<br/>- replay_data()<br/>- record_trades()<br/>Monte Carlo Simulation]
        
        PerformanceAnalyzer[Performance Analyzer<br/>- calculate_metrics()<br/>- draw_equity_curve()<br/>- risk_metrics()]
        
        ValidationFramework[Validation Framework<br/>- walk_forward_test()<br/>- robustness_test()<br/>- stress_test()]
    end

    subgraph "Data Management"
        MarketData[(Market Data Store)]
        HistoricalData[(Historical Data)]
        StrategyConfig[(Strategy Configuration)]
        ResultsDB[(Results Database)]
    end

    subgraph "Execution Pipeline"
        SignalQueue[Signal Queue<br/>Priority-based Processing]
        RiskCheck[Risk Checkpoint<br/>Pre-execution Validation]
        OrderDispatcher[Order Dispatcher<br/>Optimized Routing]
    end

    %% Strategy lifecycle
    StrategyLoader -->|Load & compile| StrategyBase
    StrategyBase -->|Extend & customize| StrategyExecutor
    
    %% Analytics flow
    MarketData -->|Real-time feed| IndicatorCalc
    IndicatorCalc -->|Calculated values| SignalGenerator
    SignalGenerator -->|Trading signals| RiskCalculator
    
    %% Backtesting flow
    HistoricalData -->|Historical prices| BacktestEngine
    StrategyConfig -->|Strategy parameters| BacktestEngine
    BacktestEngine -->|Backtest results| PerformanceAnalyzer
    PerformanceAnalyzer -->|Performance metrics| ValidationFramework
    
    %% Execution pipeline
    RiskCalculator -->|Risk-adjusted signals| SignalQueue
    SignalQueue -->|Prioritized signals| RiskCheck
    RiskCheck -->|Validated signals| OrderDispatcher
    
    %% Results storage
    BacktestEngine -->|Store results| ResultsDB
    PerformanceAnalyzer -->|Store analytics| ResultsDB
    
    %% Monitoring
    StrategyExecutor -.->|Execution metrics| MetricsCollector[Metrics Collector]
    BacktestEngine -.->|Backtest metrics| MetricsCollector
    
    style StrategyCore fill:#e1f5fe,stroke:#0277bd
    style AnalyticsEngine fill:#f3e5f5,stroke:#7b1fa2
    style BacktestingSystem fill:#e8f5e8,stroke:#2e7d32
    style DataManagement fill:#fff3e0,stroke:#ef6c00
    style ExecutionPipeline fill:#fce4ec,stroke:#c2185b
```

**Optimizaciones de Rendimiento:**
- **Caching de Indicadores**: Resultados de cálculos frecuentes almacenados en Redis
- **Procesamiento por Lotes**: Operaciones de backtest optimizadas para paralelización
- **Validación de Código**: Análisis estático y sandboxing para estrategias personalizadas
- **Pipeline de Ejecución**: Procesamiento en cola con prioridades para señales

---

## 7. Tecnologías y Relaciones Optimizadas

### 7.1 Stack Tecnológico Actualizado
```mermaid
quadrantChart
    title "Stack Tecnológico - Evaluación por Madurez y Adecuación"
    x-axis "Bajo" --> "Alto"
    y-axis "Evitar" --> "Adoptar"
    "Python 3.11+": [0.9, 0.9]
    "FastAPI": [0.85, 0.8]
    "PostgreSQL 14+": [0.95, 0.85]
    "Redis Cluster": [0.8, 0.75]
    "Docker": [0.9, 0.7]
    "Kubernetes (Futuro)": [0.6, 0.6]
    "Apache Kafka": [0.5, 0.7]
    "Prometheus/Grafana": [0.7, 0.8]
    "Flet UI": [0.4, 0.6]
    "JWT/OAuth2": [0.85, 0.9]
```

### 7.2 Relaciones entre Componentes Optimizadas

```mermaid
graph LR
    A[Frontend Layer] -->|HTTPS + JWT| B[REST API Gateway]
    B -->|gRPC con Load Balancing| C[Application Services]
    C -->|Domain Events| D[Domain Layer]
    D -->|Repository Pattern + Cache| E[Infrastructure Layer]
    E -->|Circuit Breaker + Retry| F[External Systems]
    
    G[WebSocket Gateway] -->|Compressed WebSocket| H[Real-time Services]
    H -->|Event Sourcing| I[Message Broker]
    I -->|Async Processing| J[Background Workers]
    
    K[Metrics Collector] -->|Push/Pull| L[Monitoring Stack]
    L -->|Alert Rules| M[Alert Manager]
    
    style A fill:#e8f5e8
    style B fill:#fff3e0
    style C fill:#e1f5fe
    style D fill:#f3e5f5
    style E fill:#fce4ec
    style F fill:#ffecb3
```

---

## 8. Consideraciones de Seguridad Mejoradas

### 8.1 Arquitectura de Seguridad por Capas
```mermaid
flowchart TD
    subgraph "Perimeter Security"
        WAF[Web Application Firewall]
        DDoS[DDoS Protection]
        APIGW[API Gateway Security]
    end
    
    subgraph "Application Security"
        AuthN[Authentication<br/>OAuth2/JWT/MFA]
        AuthZ[Authorization<br/>RBAC/ABAC]
        InputVal[Input Validation<br/>Schema-based]
        OutputEnc[Output Encoding]
    end
    
    subgraph "Data Security"
        Encryption[Encryption at Rest<br/>AES-256]
        TransEnc[Encryption in Transit<br/>TLS 1.3]
        KeyMgmt[Key Management<br/>HSM/KMS]
        DataMasking[Data Masking]
    end
    
    subgraph "Monitoring & Audit"
        SIEM[SIEM Integration]
        AuditLogs[Audit Logs<br/>Immutable Storage]
        AnomalyDetect[Anomaly Detection]
        ThreatIntel[Threat Intelligence]
    end
    
    WAF --> AuthN
    APIGW --> AuthZ
    AuthZ --> InputVal
    InputVal --> OutputEnc
    OutputEnc --> Encryption
    Encryption --> TransEnc
    TransEnc --> SIEM
    SIEM --> AuditLogs
    AuditLogs --> AnomalyDetect
    AnomalyDetect --> ThreatIntel
    
    style PerimeterSecurity fill:#ffebee,stroke:#c62828
    style ApplicationSecurity fill:#f3e5f5,stroke:#7b1fa2
    style DataSecurity fill:#e3f2fd,stroke:#1565c0
    style MonitoringAudit fill:#e8f5e8,stroke:#2e7d32
```

---

## 9. Roadmap de Escalabilidad

### 9.1 Evolución Arquitectónica
```mermaid
gantt
    title Roadmap de Evolución Arquitectónica
    dateFormat  YYYY-MM
    axisFormat %Y-%m
    
    section Fase MVP (Monolítico Modular)
    Diseño Arquitectura           :2026-02, 2M
    Core Trading Engine           :2026-02, 3M
    Risk Management Básico        :2026-03, 2M
    UI/UX Principal               :2026-04, 2M
    Integración MT5               :2026-05, 2M
    
    section Fase 1 (Resiliencia)
    Circuit Breakers              :2026-06, 1M
    Observabilidad                :2026-06, 2M
    Caching Estratégico           :2026-07, 1M
    Async Processing              :2026-07, 2M
    Security Hardening            :2026-08, 2M
    
    section Fase 2 (Microservicios)
    Desacoplamiento Dominios      :2026-10, 2M
    Message Broker                :2026-11, 2M
    API Gateway                   :2026-12, 1M
    Service Mesh                  :2027-01, 2M
    Database Sharding             :2027-02, 2M
    
    section Fase 3 (Escala Enterprise)
    Multi-región                  :2027-04, 3M
    Disaster Recovery             :2027-05, 2M
    Auto-scaling                  :2027-06, 2M
    Machine Learning Integration  :2027-07, 3M
```

---

## 10. Métricas Clave de Observabilidad

### 10.1 Dashboard de Monitoreo
```mermaid
graph TD
    subgraph "Business Metrics"
        BM1[Trades por Segundo]
        BM2[Fill Rate %]
        BM3[Latencia Promedio Ejecución]
        BM4[Sharpe Ratio]
        BM5[Maximum Drawdown]
    end
    
    subgraph "Technical Metrics"
        TM1[CPU/Memory Usage]
        TM2[Request Rate/Latency]
        TM3[Error Rate]
        TM4[Circuit Breaker State]
        TM5[Queue Depth]
    end
    
    subgraph "Risk Metrics"
        RM1[Exposición Total]
        RM2[Value at Risk]
        RM3[Stop Loss Triggers]
        RM4[Risk Limit Utilization]
        RM5[Correlation Matrix]
    end
    
    subgraph "Alerting Rules"
        AR1[Latencia > 100ms]
        AR2[Error Rate > 1%]
        AR3[Drawdown > 5%]
        AR4[Circuit Breaker Open]
        AR5[MT5 Disconnection]
    end
    
    BM1 --> Dashboard[Grafana Dashboard]
    BM2 --> Dashboard
    BM3 --> Dashboard
    TM1 --> Dashboard
    TM2 --> Dashboard
    RM1 --> Dashboard
    RM2 --> Dashboard
    AR1 --> AlertManager[Alert Manager]
    AR2 --> AlertManager
    AR3 --> AlertManager
    
    style BusinessMetrics fill:#e8f5e8
    style TechnicalMetrics fill:#e1f5fe
    style RiskMetrics fill:#fce4ec
    style AlertingRules fill:#ffebee
```

---

## 11. Referencias Actualizadas
- Documento Rector v2.1
- C4 Model - Simon Brown
- Clean Architecture - Robert Martin
- Domain Driven Design - Eric Evans
- Site Reliability Engineering - Google
- Microservices Patterns - Chris Richardson
- The Twelve-Factor App Methodology

---

## 12. Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| CTO | [Nombre CTO] | _________________ | 2026-02-03 |
| Tech Lead | [Nombre TL] | _________________ | 2026-02-03 |
| Security Architect | [Nombre SA] | _________________ | 2026-02-03 |
| DevOps Lead | [Nombre DL] | _________________ | 2026-02-03 |

---
