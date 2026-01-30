# C4 Architecture Model - Plataforma de Trading Algorítmico

**Versión:** 1.0  
**Fecha:** 30 de enero de 2026  
**Documento Base:** Documento Rector v2.1  
**Rol Responsable:** Tech Lead

---

## 1. Introducción

Este documento presenta la arquitectura del sistema de trading algorítmico utilizando el modelo C4 (Contexto, Contenedores, Componentes y Código). El modelo C4 proporciona una forma escalonada de describir la arquitectura de software en diferentes niveles de detalle, ideal para sistemas complejos como plataformas de trading.

---

## 2. C4 Context Diagram

El diagrama de contexto muestra el sistema en su entorno, incluyendo los actores humanos y otros sistemas con los que interactúa.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        SYSTEM CONTEXT DIAGRAM                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐                                                              │
│  │   Trader    │                                                              │
│  │   Persona   │                                                              │
│  └─────────────┘                                                              │
│         │                                                                       │
│         │ (Web/WS)                                                              │
│         ▼                                                                       │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                PLATAFORMA DE TRADING ALGORÍTMICO                        │   │
│  │                                                                         │   │
│  │  [Sistema Interno]                                                      │   │
│  │  - Trading Engine                                                       │   │
│  │  - Risk Management                                                      │   │
│  │  - Strategy Framework                                                   │   │
│  │  - Backtesting Engine                                                   │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│         ▲                                                                       │
│         │ (MT5 API)                                                             │
│         │                                                                       │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                      SERVIDOR MT5                                       │   │
│  │                                                                         │   │
│  │  [Sistema Externo]                                                      │   │
│  │  - MetaTrader 5                                                         │   │
│  │  - Market Data                                                          │   │
│  │  - Order Execution                                                      │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌─────────────┐                                                              │
│  │   Admin     │                                                              │
│  │   Persona   │                                                              │
│  └─────────────┘                                                              │
│         │                                                                       │
│         │ (Admin Panel)                                                         │
│         ▼                                                                       │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Descripción:**
- **Plataforma de Trading Algorítmico**: Sistema principal que permite trading manual y automático
- **Trader**: Usuario que opera el sistema para realizar trading manual o configurar estrategias
- **Administrador**: Personal encargado de la gestión del sistema y supervisión de riesgos
- **Servidor MT5**: Sistema externo de MetaTrader 5 que ejecuta las órdenes y provee datos de mercado

---

## 3. C4 Container Diagram

El diagrama de contenedores muestra los principales contenedores de software que forman parte del sistema.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        CONTAINER DIAGRAM                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                PLATAFORMA DE TRADING ALGORÍTMICO                        │   │
│  │                                                                         │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐    │   │
│  │  │                    PRESENTATION LAYER                         │    │   │
│  │  │                                                                 │    │   │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │    │   │
│  │  │  │   Flet UI       │  │   REST API      │  │   WebSocket     │ │    │   │
│  │  │  │                 │  │                 │  │                 │ │    │   │
│  │  │  │  (Python/Flet)  │  │  (FastAPI)      │  │  (Real-time)    │ │    │   │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────┘ │    │   │
│  │  └─────────────────────────────────────────────────────────────────┘    │   │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                     APPLICATION LAYER                                 │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ Trading Use     │  │ Risk Use        │  │ Authentication Use      │ │    │
│  │  │  │ Cases           │  │ Cases           │  │ Cases                   │ │    │
│  │  │  │                 │  │                 │  │                         │ │    │
│  │  │  │ (Order, Position│  │ (Validation,    │  │ (Login, Register,       │ │    │
│  │  │  │ Management)     │  │ Limits)         │  │ Session Management)     │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                       DOMAIN LAYER                                    │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ Trading Domain  │  │ Risk Domain     │  │ User/Account Domain     │ │    │
│  │  │  │                 │  │                 │  │                         │ │    │
│  │  │  │ (Orders,        │  │ (Limits,        │  │ (Authentication,        │ │    │
│  │  │  │ Positions,      │  │ Exposure,       │  │ Authorization)          │ │    │
│  │  │  │ Strategies)     │  │ Correlation)    │  │                         │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                   INFRASTRUCTURE LAYER                                │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ MT5 Adapter     │  │ Database        │  │ Cache                   │ │    │
│  │  │  │                 │  │                 │  │                         │ │    │
│  │  │  │ (Connection     │  │ (PostgreSQL)    │  │ (Redis)                 │ │    │
│  │  │  │ Pool, Protocol) │  │                 │  │                         │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ External APIs   │  │ Message Broker  │  │ Notification Service    │ │    │
│  │  │  │                 │  │                 │  │                         │ │    │
│  │  │  │ (Market Data,   │  │ (Kafka/RabbitMQ │  │ (Email, SMS, Push)      │ │    │
│  │  │  │ News)           │  │ Future)         │  │                         │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  └─────────────────────────────────────────────────────────────────────────────────┘
│                                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────────┐       │
│  │                      SERVIDOR MT5                                       │       │
│  │                                                                         │       │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │       │
│  │  │ Market Data     │  │ Order Execution │  │ Account Management      │ │       │
│  │  │                 │  │                 │  │                         │ │       │
│  │  │ (Quotes, Bars)  │  │ (Orders,        │  │ (Balance, Positions,    │ │       │
│  │  │                 │  │ Positions)      │  │ History)                │ │       │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │       │
│  └─────────────────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. C4 Component Diagram - Trading Engine

Diagrama detallado del componente principal de Trading Engine:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    TRADING ENGINE COMPONENT                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    TRADING ENGINE MODULE                                │   │
│  │                                                                         │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐    │   │
│  │  │                    CORE COMPONENTS                            │    │   │
│  │  │                                                                 │    │   │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │    │   │
│  │  │  │ OrderManager    │  │ PositionManager │  │ StrategyRunner  │ │    │   │
│  │  │  │                 │  │                 │  │                 │ │    │   │
│  │  │  │ - submit_order  │  │ - get_position  │  │ - run_strategy  │ │    │   │
│  │  │  │ - modify_order  │  │ - update_pos    │  │ - stop_strategy │ │    │   │
│  │  │  │ - cancel_order  │  │ - close_pos     │  │ - pause_resume  │ │    │   │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────┘ │    │   │
│  │  └─────────────────────────────────────────────────────────────────┘    │   │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                  INTEGRATION COMPONENTS                               │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ MT5Connector    │  │ RiskValidator   │  │ OrderExecutionEngine    │ │    │
│  │  │  │                 │  │                 │  │                         │ │    │
│  │  │  │ - connect       │  │ - validate_risk │  │ - execute_order         │ │    │
│  │  │  │ - disconnect    │  │ - check_limits  │  │ - handle_fill           │ │    │
│  │  │  │ - reconnect     │  │ - risk_alert    │  │ - handle_rejection      │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                   SUPPORT COMPONENTS                                  │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ OrderBook       │  │ MarketDataFeed  │  │ ExecutionReport         │ │    │
│  │  │  │                 │  │                 │  │                         │ │    │
│  │  │  │ - maintain      │  │ - subscribe     │  │ - generate_report       │ │    │
│  │  │  │ - sync_orders   │  │ - unsubscribe   │  │ - send_notifications    │ │    │
│  │  │  │ - match_orders  │  │ - process_feed  │  │ - store_history         │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  └─────────────────────────────────────────────────────────────────────────────────┘
│                                                                                   │
│  INTERACCIONES:                                                                   │
│  - OrderManager ↔ MT5Connector: Envío y recepción de órdenes                      │
│  - OrderManager ↔ RiskValidator: Validación de riesgo antes de enviar órdenes     │
│  - PositionManager ↔ OrderExecutionEngine: Actualización de posiciones            │
│  - MarketDataFeed ↔ OrderBook: Actualización de precios para matching              │
│  - ExecutionReport ↔ OrderExecutionEngine: Generación de reportes de ejecución     │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. C4 Component Diagram - Risk Management

Diagrama detallado del componente de Gestión de Riesgo:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                 RISK MANAGEMENT COMPONENT                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                 RISK MANAGEMENT MODULE                                  │   │
│  │                                                                         │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐    │   │
│  │  │                    VALIDATION COMPONENTS                        │    │   │
│  │  │                                                                 │    │   │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │    │   │
│  │  │  │ AccountRisk     │  │ StrategyRisk    │  │ MarketRisk      │ │    │   │
│  │  │  │ Validator       │  │ Validator       │  │ Validator       │ │    │   │
│  │  │  │                 │  │                 │  │                 │ │    │   │
│  │  │  │ - daily_loss    │  │ - max_exposure  │  │ - volatility    │ │    │   │
│  │  │  │ - weekly_loss   │  │ - correlation   │  │ - market_halt   │ │    │   │
│  │  │  │ - equity_prot   │  │ - kill_switch   │  │ - news_impact   │ │    │   │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────┘ │    │   │
│  │  └─────────────────────────────────────────────────────────────────┘    │   │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                  MONITORING COMPONENTS                                │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ Position        │  │ Exposure        │  │ Drawdown                │ │    │
│  │  │  │ Monitor         │  │ Monitor         │  │ Monitor                 │ │    │
│  │  │  │                 │  │                 │  │                         │ │    │
│  │  │  │ - track_pos     │  │ - calc_exposure │  │ - calc_drawdown         │ │    │
│  │  │  │ - alert_changes │  │ - alert_limit   │  │ - trigger_protection    │ │    │
│  │  │  │ - enforce_stop  │  │ - risk_metrics  │  │ - risk_metrics          │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                   CONTROL COMPONENTS                                  │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ KillSwitch      │  │ EmergencyStop   │  │ OverrideControl         │ │    │
│  │  │  │ Controller      │  │ Controller      │  │                         │ │    │
│  │  │  │                 │  │                 │  │ - manual_override       │ │    │
│  │  │  │ - global_kill   │  │ - emergency     │  │ - permission_matrix     │ │    │
│  │  │  │ - strategy_kill │  │ - account_stop  │  │ - approval_workflow     │ │    │
│  │  │  │ - auto_reset    │  │ - position_close│  │                         │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  └─────────────────────────────────────────────────────────────────────────────────┘
│                                                                                   │
│  INTERACCIONES:                                                                   │
│  - AccountRiskValidator ↔ PositionMonitor: Validación contra posiciones actuales    │
│  - StrategyRiskValidator ↔ ExposureMonitor: Validación contra exposición            │
│  - MarketRiskValidator ↔ MarketDataFeed: Validación contra condiciones de mercado   │
│  - DrawdownMonitor ↔ KillSwitchController: Activación de protecciones automáticas   │
│  - OverrideControl ↔ Todos los componentes: Control de overrides manuales           │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. C4 Component Diagram - Strategy Framework

Diagrama detallado del componente de Framework de Estrategias:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                 STRATEGY FRAMEWORK COMPONENT                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                   STRATEGY FRAMEWORK MODULE                             │   │
│  │                                                                         │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐    │   │
│  │  │                    STRATEGY COMPONENTS                          │    │   │
│  │  │                                                                 │    │   │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │    │   │
│  │  │  │ StrategyBase    │  │ StrategyLoader  │  │ StrategyRunner  │ │    │   │
│  │  │  │                 │  │                 │  │                 │ │    │   │
│  │  │  │ - initialize    │  │ - load_strategy │  │ - start_strategy│ │    │   │
│  │  │  │ - on_tick       │  │ - validate_code │  │ - stop_strategy │ │    │   │
│  │  │  │ - on_bar        │  │ - compile_code  │  │ - pause_resume  │ │    │   │
│  │  │  │ - on_order      │  │ - cache_strategy│  │ - get_status    │ │    │   │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────┘ │    │   │
│  │  └─────────────────────────────────────────────────────────────────┘    │   │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                  ANALYTICS COMPONENTS                                 │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ Indicator       │  │ SignalGenerator │  │ RiskCalculator          │ │    │
│  │  │  │ Calculator      │  │                 │  │                         │ │    │
│  │  │  │                 │  │ - generate_sig  │  │ - position_sizing       │ │    │
│  │  │  │ - calc_tech_ind │  │ - validate_sig  │  │ - stop_loss_calc        │ │    │
│  │  │  │ - calc_fund_rat │  │ - signal_merge  │  │ - take_profit_calc      │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  │                                                                                   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  │                   BACKTEST COMPONENTS                                 │    │
│  │  │                                                                         │    │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │    │
│  │  │  │ BacktestEngine  │  │ Performance     │  │ Validation              │ │    │
│  │  │  │                 │  │ Analyzer        │  │ Framework               │ │    │
│  │  │  │ - run_backtest  │  │ - calc_metrics  │  │ - walk_forward          │ │    │
│  │  │  │ - replay_data   │  │ - draw_equity   │  │ - monte_carlo           │ │    │
│  │  │  │ - record_trades │  │ - risk_metrics  │  │ - robustness_test       │ │    │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │    │
│  │  └─────────────────────────────────────────────────────────────────────────┘    │
│  └─────────────────────────────────────────────────────────────────────────────────┘
│                                                                                   │
│  INTERACCIONES:                                                                   │
│  - StrategyLoader ↔ StrategyBase: Carga y validación de estrategias               │
│  - StrategyRunner ↔ IndicatorCalculator: Cálculo de indicadores para toma de       │
│    decisiones                                                                      │
│  - SignalGenerator ↔ RiskCalculator: Validación de señales contra controles de     │
│    riesgo                                                                          │
│  - BacktestEngine ↔ PerformanceAnalyzer: Análisis de resultados de backtest        │
│  - ValidationFramework ↔ BacktestEngine: Validación de robustez de estrategias     │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Tecnologías y Relaciones

### 7.1 Tecnologías Utilizadas
- **Backend**: Python 3.11+, FastAPI, SQLAlchemy
- **Frontend**: Flet (Python-based UI)
- **Base de Datos**: PostgreSQL 14+
- **Cache**: Redis
- **Conexión MT5**: python-mt5
- **Mensajeo**: WebSocket para real-time, futuro: Kafka/RabbitMQ

### 7.2 Relaciones entre Contenedores
- **Flet UI** → **REST API**: Comunicación síncrona para operaciones CRUD
- **REST API** → **Application Layer**: Invocación de casos de uso
- **Application Layer** → **Domain Layer**: Ejecución de lógica de negocio
- **Domain Layer** → **Infrastructure Layer**: Acceso a recursos externos
- **Infrastructure Layer** → **MT5 Server**: Comunicación con servidor externo

---

## 8. Consideraciones de Seguridad

### 8.1 En el Contexto
- Autenticación y autorización entre todos los actores y el sistema
- Cifrado de comunicaciones entre componentes
- Auditoría de todas las operaciones críticas

### 8.2 En los Contenedores
- Validación de entrada en la capa de presentación y aplicación
- Control de acceso basado en roles
- Segregación de privilegios entre componentes

### 8.3 En los Componentes
- Validación de riesgo antes de ejecutar órdenes
- Controles de acceso a datos sensibles
- Registro de todas las operaciones de trading

---

## 9. Consideraciones de Riesgo

### 9.1 Riesgos Técnicos
- Conectividad con MT5: Implementar mecanismos de reconexión y buffering
- Latencia: Optimizar rutas críticas y usar cache apropiadamente
- Escalabilidad: Diseñar componentes para eventual desacoplamiento

### 9.2 Riesgos de Negocio
- Pérdidas financieras: Implementar controles de riesgo multicapa
- Fallo del sistema: Alta disponibilidad y mecanismos de recuperación
- Seguridad: Protección contra accesos no autorizados y fraudes

---

## 10. Escalabilidad Futura

### 10.1 Fase MVP
- Arquitectura monolítica modular
- Único nodo de ejecución
- Componentes acoplados pero bien definidos

### 10.2 Fase Escalamiento
- Desacoplamiento en microservicios
- Message broker para comunicación asíncrona
- Auto-scaling basado en carga

---

## 11. Referencias
- Documento Rector v2.1
- C4 Model - Simon Brown
- Clean Architecture - Robert Martin
- Domain Driven Design - Eric Evans

---

## 12. Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| CTO | [Nombre CTO] | _________________ | ___________ |
| Tech Lead | [Nombre TL] | _________________ | ___________ |

---

**Estado:** APROBADO PARA IMPLEMENTACIÓN  
**Versión:** 1.0