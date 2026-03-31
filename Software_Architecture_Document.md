# Software Architecture Document (SAD)

**Sistema:** Sistema Integral de Trading Algorítmico Multiactivos  
**Versión:** 1.3  
**Fecha:** 31 de marzo de 2026  
**Responsable:** Edgar Gomez  

---

## 1. Introducción

### 1.1 Propósito
Este documento describe la arquitectura de software del sistema de trading algorítmico multiactivos, proporcionando una guía técnica detallada para su desarrollo, mantenimiento, escalabilidad, auditoría y evolución. El diseño se basa en los principios de **Clean Architecture** (Robert C. Martin), priorizando:

- **Separación estricta de responsabilidades** entre lógica de negocio, adaptadores y frameworks.
- **Independencia de proveedores externos** (brokers, APIs, bases de datos).
- **Reproducibilidad y paridad entre backtesting y producción**.
- **Validación contextual robusta** mediante inteligencia artificial especializada por clase de activo.
- **Escalabilidad horizontal** para nuevos activos, estrategias y fuentes de datos.

### 1.2 Alcance
El sistema cubre:

- Captura dual de datos: MT5 (producción) + fuentes alternativas (`yfinance`, `ccxt`, etc.) para investigación.
- Generación de features técnicas, fundamentales y de sentimiento.
- Modelos predictivos especializados por clase de activo.
- Motor de ensemble para fusión de señales.
- **`ContextualSignalValidator`**: validación de coherencia de señal mediante LLM con prompts por grupo de activo.
- Gestión de órdenes (OMS) y riesgo a nivel de portfolio.
- Radar de Acciones y Análisis de Eventos Económicos.
- Dashboard web interactivo.
- Logging centralizado en SQLite (producción) y PostgreSQL (research).

---

## 2. Filosofía de Arquitectura

### 2.1 Clean Architecture
La arquitectura sigue las cuatro capas clásicas, con dependencias que apuntan **hacia adentro**:

| Capa | Componentes | Dependencia |
|------|-------------|-------------|
| **Entities / Core** | Data Layer, Feature Engine, Model Engine | Independiente |
| **Use Cases / Interactors** | Ensemble Engine, **ContextualSignalValidator**, Strategy Engine | Depende solo de Entities |
| **Interface Adapters** | Radar, Account Manager, Economic Event Analyzer, Dual Data Pipeline Adapter | Depende de Use Cases y Entities |
| **Frameworks & Drivers** | MT5, `yfinance`, LLM API, PostgreSQL, SQLite, Web Dashboard | Depende de todo lo anterior |

### 2.2 Principios Clave
- **Dependency Inversion**: Todos los módulos externos se conectan mediante interfaces definidas en capas internas.
- **Single Responsibility**: Cada componente tiene una única razón para cambiar.
- **Open/Closed**: Nuevo activo = nueva implementación, sin modificar código existente.
- **Backtest-Production Parity**: El mismo código se ejecuta en ambos entornos.
- **Event-Driven Communication**: Los módulos se comunican mediante eventos o callbacks, no llamadas directas.

---

## 3. Componentes Principales (Actualizados)

### 3.1 Data Layer
**Responsabilidad**: Captura, validación, normalización y almacenamiento de datos.  
**Módulos**:
- `MarketDataAdapter`: interfaz abstracta con implementaciones para MT5 (live) y `yfinance`/`ccxt` (research).
- `EventFeedAdapter`: consume calendarios económicos y noticias.
- `DataValidator`: detecta gaps, outliers, saltos extremos sin volumen correlacionado.
- `Repository`: 
  - **Producción**: SQLite (ligero, local, confiable).
  - **Research**: PostgreSQL (escalable, soporta consultas complejas).

### 3.2 Feature & Transformation Layer
**Responsabilidad**: Generación de features estructuradas para modelos y validación.  
**Funciones**:
- Técnicas: RSI, EMA, ATR, MACD, volatilidad histórica.
- Fundamentales: P/E, ROE, inventarios EIA, tasas reales.
- Sentimiento: score FinBERT o DistilBERT.
- Normalización y rolling windows.
- Registro de features en base de datos para reproducibilidad.

### 3.3 Model Engine Layer
**Responsabilidad**: Predicción de dirección de precio por activo.  
**Modelos especializados**:
- **Forex**: XGBoost + HMM (regímenes de mercado).
- **Commodities energéticos**: LightGBM + Prophet (estacionalidad).
- **Metales preciosos**: Random Forest + VIX como input.
- **Acciones**: LightGBM con features fundamentales + sentimiento.
- **Cripto**: LSTM con volatilidad y volumen.
- **Índices**: Gradient Boosting + ARIMA.
- **Registro**: Todas las predicciones se loguean con timestamp, versión de modelo y contexto.

### 3.4 Ensemble Engine
**Responsabilidad**: Fusión robusta de señales múltiples.  
**Entradas**:
- Señal del Model Engine.
- Estado del régimen (HMM).
- Impacto de evento económico.
- Score de sentimiento.
**Salida**: Señal final + nivel de confianza (0–1).  
**Métodos**: Pesos dinámicos, SHAP-based blending, reglas de veto.

### 3.5 ContextualSignalValidator (Nuevo en v1.3)
**Responsabilidad**: Validar la coherencia lógica de la señal del Ensemble con el entorno macro/fundamental actual.  
**Flujo**:
1. Identifica el **grupo de activo** (ej. `oil_group`, `precious_metals`).
2. Consulta `prompt_templates.db` para obtener el prompt especializado.
3. Renderiza el prompt con datos actuales (precio, eventos, fundamentales, sentimiento).
4. Llama a una **API de LLM especializado** (GPT-4o, Claude 3.5, o modelo local).
5. Compara respuesta (ALCISTA/BAJISTA/NEUTRAL) con la señal del Ensemble.
6. Decide: emitir orden completa, reducir tamaño o cancelar.
**Buenas prácticas**:
- Cache inteligente (mismo contexto → misma respuesta).
- Fallback robusto si la API falla.
- Logging completo del prompt, respuesta y decisión.

### 3.6 Strategy Engine (Use Cases Layer)
**Responsabilidad**: Coordinar señales, riesgo y ejecución.  
**Funciones**:
- Recibe señal validada del `ContextualSignalValidator`.
- Consulta `Account Manager` para validar margen y exposición.
- Aplica reglas de negocio (ej. no operar en eventos de alto impacto).
- Envía órdenes al OMS.
- Registra decisiones en base de datos.

### 3.7 Order Management System (OMS)
**Responsabilidad**: Ejecución segura y trazable de órdenes.  
**Funciones**:
- Tipos: Market, Limit, Stop, Stop-Limit, Trailing.
- Validación en tiempo real: margen, apalancamiento, drawdown.
- Reintentos automáticos en fallos de conexión.
- Magic Numbers para trazabilidad.
- Registro en SQLite/PostgreSQL.

### 3.8 Account Manager
**Responsabilidad**: Supervisión del estado de la cuenta y gestión de riesgo.  
**Métricas**:
- Equity, balance, margen libre.
- Drawdown máximo, racha de pérdidas.
- Exposición por activo y grupo.
- Volatility targeting (ajuste dinámico de apalancamiento).
- Kill switch automático si drawdown > 10%.

### 3.9 Radar de Acciones
**Responsabilidad**: Monitoreo proactivo de universos amplios.  
**Funciones**:
- Heatmap de oportunidades.
- Indicadores técnicos y fundamentales.
- Alertas para el `Strategy Engine`.
- Visualización en dashboard.

### 3.10 Economic Event Analyzer
**Responsabilidad**: Mitigación de riesgo exógeno.  
**Funciones**:
- Clasificación de eventos por impacto.
- Bloqueo temporal de trading en activos sensibles.
- Ajuste dinámico de exposición.
- Registro histórico para backtesting.

### 3.11 Web Dashboard
**Responsabilidad**: Visualización centralizada y control.  
**Tecnología**: Dash / Streamlit con WebSocket para tiempo real.  
**Vistas**:
- Gráficos de velas con overlays de señales.
- Cartera agrupada por clase de activo.
- Radar de acciones.
- Calendario de eventos económicos.
- Métricas de riesgo y PnL.

---

## 4. Flujo de Comunicación

- **Internamente**: Patrón **Observer** y **Event-Driven**. Ejemplo:  
  `ModelEngine` emite evento → `EnsembleEngine` lo procesa → `ContextualSignalValidator` lo valida → `StrategyEngine` decide.
- **Externamente**: 
  - **REST API** para comandos manuales (start/stop/pause).
  - **WebSocket** para streaming de precios, órdenes y alertas al dashboard.
- **Logging**: Todo evento relevante (predicción, decisión, orden, error) se registra en base de datos con timestamp UTC.

---

## 5. Buenas Prácticas y Convenciones

| Área | Práctica |
|------|--------|
| **Arquitectura** | Clean Architecture estricta; dependencias invertidas. |
| **Testing** | Unit tests en `core/` (>85% cobertura); integration tests en `interactors/`. |
| **Versionado** | Modelos, features, prompts y pipelines versionados. |
| **Seguridad** | Conexión cifrada con MT5; autenticación en dashboard. |
| **Escalabilidad** | Módulos críticos (Radar, Event Analyzer) pueden correr en hilos separados o microservicios. |
| **Costos** | Llamadas a LLM condicionales (solo en incertidumbre); cache inteligente. |
| **Auditoría** | Paridad backtest-producción; logging completo; snapshots periódicos. |

---

## 6. Diagrama de Arquitectura (Resumen Textual)
[Web Dashboard] ←(REST/WebSocket)→ [Interface Adapters]
↑
[Radar] [Account Manager] [Event Analyzer] [Dual Data Pipeline]
↑
[Use Cases: Strategy Engine ← ContextualSignalValidator ← Ensemble Engine]
↑
[Core: Model Engine ← Feature Engine ← Data Layer]
↓
[Frameworks: MT5, yfinance, ccxt, LLM API, PostgreSQL, SQLite]


> **Nota**: Las flechas indican flujo de control y dependencia. Todas las dependencias apuntan hacia el núcleo (`Core`).

---

