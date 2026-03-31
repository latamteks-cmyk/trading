# Product Vision Document: Sistema Integral de Trading Algorítmico Multiactivos

**Versión:** 1.3  
**Fecha:** 31 de marzo de 2026  
**Responsable:** Edgar Gomez

---

## 1. Visión General del Producto

El sistema es una plataforma integral de trading algorítmico y gestión de cuentas multiactivos, diseñada para:

1. Automatizar estrategias de trading cuantitativo en Forex, Commodities, Índices, Acciones y Criptomonedas.
2. Gestionar órdenes, posiciones y riesgos de manera profesional, incluyendo apalancamiento, stop loss dinámico y trailing stops.
3. Monitorear la cuenta y la cartera en tiempo real mediante dashboards interactivos.
4. Registrar predicciones y decisiones de trading en base de datos para auditoría y mejora continua.
5. Visualizar gráficamente cada símbolo y grupo de activos con velas, indicadores y overlays de señales.
6. Escalar a múltiples activos, asignando métodos predictivos especializados según la naturaleza de cada mercado.
7. Incorporar un Radar de Acciones para supervisar un universo amplio de símbolos y detectar oportunidades.
8. Incorporar un Análisis de Eventos Económicos que evalúe la viabilidad de ingreso al mercado y gestione riesgo frente a noticias.
9. Integrar fuentes fundamentales y alternativas (sentimiento, Google Trends) para enriquecer las señales.
10. Validar dinámicamente las señales del Ensemble mediante un motor de contexto basado en IA especializada por clase de activo.
11. Garantizar paridad entre backtesting y producción mediante arquitectura unificada y logging completo.

---

## 2. Alcance del Producto

### Activos cubiertos:
- **Forex**: EURUSD, GBPUSD, USDJPY...
- **Commodities**: XAUUSD (oro), WTIUSD (petróleo), SOYABEAN...
- **Índices**: SP500, DAX30...
- **Acciones**: AAPL, TSLA...
- **Criptomonedas**: BTCUSD, ETHUSD...

### Funciones principales:
- **Dual Data Pipeline**: Captura desde MT5 (producción) + fuentes alternativas (`yfinance`, `ccxt`, APIs públicas) para investigación.
- Generación de features técnicas, fundamentales y de sentimiento según activo.
- Modelos predictivos adaptados a cada clase de activo.
- **Motor de Ensemble**: Combina señales de ML, regímenes, eventos y sentimiento.
- **`ContextualSignalValidator`**: Valida coherencia de señal con entorno macro/fundamental usando IA especializada por grupo de activo.
- Motor de estrategias que traduce señales en órdenes.
- OMS con validación de riesgo, margen y exposición.
- Account Manager que supervisa balances, equity y drawdown.
- Registro histórico en PostgreSQL (research) + SQLite (producción/auditoría).
- Dashboard interactivo web para gráficos, cartera y alertas.
- Radar de Acciones para monitoreo proactivo.
- Análisis de Eventos Económicos con ajuste dinámico de exposición.
- Validación de datos en tiempo real: gaps, outliers, saltos extremos.
- Backtesting unificado: mismo código para simulación y producción.

---

## 3. Funciones Clave y Enfoques Detallados

### 3.1 Gestión de Datos y Feature Engine
- Extracción dual: MT5 (live) + `yfinance`/`ccxt`/`fredapi` (research).
- Validación en tiempo real: detección de anomalías de precios y volumen.
- Features:
  - Técnicas: RSI, EMA, ATR, etc.
  - Fundamentales: P/E, ROE, inventarios EIA, etc.
  - Sentimiento: Score FinBERT o modelo ligero (DistilBERT).

### 3.2 Motor Predictivo (Model Engine)
- Modelos especializados por activo (XGBoost+HMM para Forex, LSTM para cripto, etc.).
- Incorporación de factores fundamentales y de sentimiento como inputs.

### 3.3 Motor de Ensemble
- Fusión de señales mediante pesos dinámicos, SHAP y reglas de veto.
- Salida: señal + nivel de confianza.

### 3.4 ContextualSignalValidator (Nuevo Componente Clave)
- **Responsabilidad**: Validar si la señal del Ensemble es coherente con el contexto de mercado actual.
- **Mecanismo**:
  - Identifica el **grupo de activo** (ej. `oil_group`, `precious_metals`).
  - Consulta una **base de datos de prompts especializados** (`prompt_templates.db`).
  - Renderiza un prompt estructurado con datos actuales (precio, eventos, sentimiento, fundamentales).
  - Llama a una **API de LLM especializado** (GPT-4o, Claude 3.5, o modelo local).
  - Compara la respuesta (ALCISTA/BAJISTA/NEUTRAL) con la señal del Ensemble.
- **Acción**:
  - Concordancia → emitir orden completa.
  - Discordancia → reducir tamaño o cancelar.
  - Neutral → mantener posición actual.
- **Buenas prácticas**:
  - Cache inteligente para controlar costos.
  - Fallback robusto si la API falla.
  - Logging completo del prompt, respuesta y decisión.

### 3.5 Gestión de Órdenes (OMS)
- Recibe la señal validada. Sin cambios sustanciales.

### 3.6 Gestión de Cuenta (Account Manager)
- Supervisa riesgo total, drawdown y exposición por grupo.

### 3.7 Radar de Acciones
- Usa datos fundamentales y técnicos para rankear oportunidades.

### 3.8 Análisis de Eventos Económicos
- Integra impacto de eventos como input al Ensemble y al `ContextualSignalValidator`.

### 3.9 Backtesting Unificado
- Mismo código para backtest y live.
- Stack: `VectorBT` (rápido) + `backtrader` (fidelidad).

### 3.10 Logging y Auditoría
- **Producción**: SQLite.
- **Research**: PostgreSQL.

---

## 4. Arquitectura Escalada del Sistema

La arquitectura sigue **Clean Architecture**, con los siguientes niveles:

- **Entities / Core**: Data Layer, Feature Engine, Model Engine.
- **Use Cases / Interactors**: Ensemble Engine, **ContextualSignalValidator**, Strategy Engine.
- **Interface Adapters**: Radar, Account Manager, Economic Event Analyzer, Dual Data Pipeline Adapter.
- **Frameworks & Drivers**: MT5, `yfinance`, `ccxt`, PostgreSQL, SQLite, Web Dashboard, LLM API.

La comunicación es **event-driven** y desacoplada mediante interfaces definidas.

---

## 5. Reglas de Riesgo y Buenas Prácticas

- Riesgo máximo por trade: 1–2% del equity.
- Exposición máxima total: 5–10%.
- Cierre automático si drawdown > 10%.
- **Paridad backtest-producción obligatoria.**
- **Validación de datos en tiempo real obligatoria.**
- **Uso condicional del `ContextualSignalValidator` solo en zonas de incertidumbre.**
- Logs en SQLite (producción) y PostgreSQL (research).
- Separación estricta de entornos Research vs Producción.
- Versionado de modelos, features y prompts.

---

## 6. Roadmap de Desarrollo

1. **Fase 1**: Conexión MT5, Data Pipeline dual, SQLite/PostgreSQL.
2. **Fase 2**: Modelos predictivos + Feature Engine (técnicos y fundamentales).
3. **Fase 3**: Ensemble Engine + **ContextualSignalValidator** + Validación de datos.
4. **Fase 4**: OMS + Account Manager + logging.
5. **Fase 5**: Dashboard web con gráficos de velas y cartera agrupada.
6. **Fase 6**: Radar de acciones y alertas inteligentes.
7. **Fase 7**: Análisis de eventos económicos.
8. **Fase 8**: Integración completa de backtesting unificado y pruebas walk-forward.

---

## 7. Valor Agregado

- Monitoreo proactivo de oportunidades en un universo amplio de activos.
- Gestión de riesgo profesional y dinámica, considerando eventos, fundamentales y contexto global.
- Fusión robusta de señales mediante ensemble learning validado por IA especializada.
- Visualización centralizada: radar, gráficos de velas, PnL y alertas.
- Paridad garantizada entre investigación y producción.
- Escalabilidad a nuevos activos, estrategias y métodos predictivos.
- Base sólida para automatización, auditoría y mejora continua del trading cuantitativo.
