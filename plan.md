El plan incluye:
- **Fases de desarrollo secuenciales y realistas**.
- **Especificaciones técnicas detalladas por componente**.
- **Dependencias entre módulos**.
- **Criterios de aceptación**.

---

# 📅 Plan de Desarrollo y Especificaciones Técnicas  
**Sistema:** Sistema Integral de Trading Algorítmico Multiactivos  
**Versión del Plan:** 1.3  
**Fecha:** 31 de marzo de 2026  

---

## 🧱 Principios del Plan

- **Incremental**: Cada fase entrega valor funcional.
- **Validable**: Cada componente tiene criterios de aceptación claros.
- **Backtest-first**: Todo modelo o lógica debe validarse antes de producción.
- **Modular**: Los componentes son independientes y reemplazables.
- **Documentado**: Cada módulo incluye logging, tests y documentación.

---

## 🗂️ Fase 1: Infraestructura Base y Dual Data Pipeline  
**Duración estimada:** 2 semanas  
**Objetivo:** Establecer la base de datos, captura de datos dual y validación.

### Componentes a desarrollar:

| Componente | Especificaciones Técnicas | Dependencias | Criterios de Aceptación |
|-----------|--------------------------|--------------|--------------------------|
| **`MarketDataAdapter`** | - Interfaz abstracta `IMarketData`. <br> - Implementaciones: `MT5Adapter` (producción), `YFinanceAdapter`, `CCXTAdapter` (research). <br> - Soporta OHLCV + volumen + ticks (MT5). | Ninguna | - Puede obtener 1 año de datos históricos sin fallos. <br> - Streaming en tiempo real estable >99%. |
| **`EventFeedAdapter`** | - Scraping controlado de ForexFactory (Playwright). <br> - API de Google Calendar Económico (alternativa). <br> - Clasificación automática: alto/medio/bajo impacto. | `MarketDataAdapter` | - Eventos llegan con ≥15 min de anticipación. <br> - Impacto clasificado correctamente en 90%+ de casos. |
| **`DataValidator`** | - Detecta: gaps >5σ, precios negativos, volumen cero en activos líquidos, saltos sin causa. <br> - Emite alertas y bloquea señales si hay anomalía crítica. | `MarketDataAdapter` | - Anomalías detectadas en <1 segundo. <br> - Falsos positivos <2%. |
| **`Repository`** | - SQLite para producción (tablas: `market_data`, `events`, `logs`). <br> - PostgreSQL opcional para research (misma estructura). <br> - SQLAlchemy como ORM. | Todos los anteriores | - Escritura/lectura concurrente estable. <br> - Backups automáticos diarios. |

---

## ⚙️ Fase 2: Feature Engine y Model Engine  
**Duración estimada:** 3 semanas  
**Objetivo:** Generar features y entrenar modelos especializados por clase de activo.

### Componentes:

| Componente | Especificaciones Técnicas | Dependencias | Criterios de Aceptación |
|-----------|--------------------------|--------------|--------------------------|
| **`FeatureEngine`** | - Features técnicas: RSI, EMA, ATR, MACD, volatilidad. <br> - Features fundamentales: P/E, ROE, inventarios EIA (acciones/commodities). <br> - Sentimiento: FinBERT/DistilBERT sobre titulares. <br> - Normalización Z-score rolling. | `MarketDataAdapter`, `EventFeedAdapter` | - Features generadas en <500ms por activo. <br> - Reproducibles (mismo input → mismo output). |
| **`ModelEngine`** | - **Forex**: XGBoost + HMM. <br> - **Commodities energéticos**: LightGBM + Prophet. <br> - **Metales preciosos**: Random Forest + VIX. <br> - **Acciones**: LightGBM + sentimiento + fundamentales. <br> - **Cripto**: LSTM. <br> - Entrenamiento automático con Optuna. | `FeatureEngine` | - Walk-forward Sharpe >1.0 en out-of-sample. <br> - Modelo versionado y registrado en DB. |

---

## 🔀 Fase 3: Ensemble, Validación Contextual y Estrategia  
**Duración estimada:** 3 semanas  
**Objetivo:** Fusionar señales y validar con IA especializada.

### Componentes:

| Componente | Especificaciones Técnicas | Dependencias | Criterios de Aceptación |
|-----------|--------------------------|--------------|--------------------------|
| **`EnsembleEngine`** | - Entradas: señal ML, régimen HMM, evento, sentimiento. <br> - Pesos dinámicos basados en volatilidad y régimen. <br> - Reglas de veto (ej. no operar en eventos altos). <br> - Salida: señal + confianza [0–1]. | `ModelEngine`, `EventFeedAdapter` | - Reduce drawdown vs modelo individual en backtest. <br> - Confianza calibrada (reliability diagram). |
| **`ContextualSignalValidator`** | - Base de prompts: `prompt_templates.db` (SQL schema definido). <br> - Renderizado Jinja2 con contexto actual. <br> - Llamada condicional a LLM (solo si confianza ∈ [0.4, 0.7]). <br> - Fallback: reducir tamaño si API falla. | `EnsembleEngine`, `FeatureEngine` | - Concordancia LLM ↔ Ensemble >75% en mercado estable. <br> - Costo de API < $5/día en promedio. |
| **`StrategyEngine`** | - Recibe señal validada. <br> - Consulta `AccountManager` para margen/exposición. <br> - Aplica trailing stop, scale-out parcial. <br> - Emite órdenes al OMS. | `ContextualSignalValidator`, `AccountManager` | - Órdenes generadas en <1s tras señal. <br> - 100% de decisiones logueadas. |

---

## 📊 Fase 4: Gestión de Órdenes, Cuenta y Logging  
**Duración estimada:** 2 semanas  
**Objetivo:** Ejecución segura, trazable y con gestión de riesgo.

### Componentes:

| Componente | Especificaciones Técnicas | Dependencias | Criterios de Aceptación |
|-----------|--------------------------|--------------|--------------------------|
| **`OMS`** | - Tipos: Market, Limit, Stop, Trailing. <br> - Validación en tiempo real: margen, apalancamiento, drawdown. <br> - Magic Numbers únicos por estrategia. <br> - Reintentos automáticos (3 intentos). | `StrategyEngine`, `AccountManager` | - Órdenes ejecutadas con slippage <0.1%. <br> - 0 órdenes perdidas por fallo de conexión. |
| **`AccountManager`** | - Métricas: equity, balance, margen libre, drawdown. <br> - Volatility targeting: ajusta apalancamiento para mantener σ_target. <br> - Kill switch si drawdown >10%. | `OMS`, `MarketDataAdapter` | - Actualización de métricas en <500ms. <br> - Kill switch activado en <10s tras breach. |
| **`Logging & Audit`** | - Tablas: `predictions`, `signals`, `orders`, `events`, `prompts_llm`. <br> - Timestamp UTC estricto. <br> - Snapshots diarios de estado de cuenta. | Todos los módulos | - Capacidad de reproducir cualquier día de trading. <br> - Queries de auditoría en <2s. |

---

## 👁️ Fase 5: Radar, Eventos y Dashboard  
**Duración estimada:** 2 semanas  
**Objetivo:** Visualización, monitoreo proactivo y alertas.

### Componentes:

| Componente | Especificaciones Técnicas | Dependencias | Criterios de Aceptación |
|-----------|--------------------------|--------------|--------------------------|
| **`RadarDeAcciones`** | - Universo: S&P 500 + top 100 cripto/commodities. <br> - Heatmap con RSI, momentum, volumen relativo. <br> - Alertas para `StrategyEngine`. | `FeatureEngine`, `ModelEngine` | - Actualización cada 5 min. <br> - Latencia <1s en selección de activo. |
| **`EconomicEventAnalyzer`** | - Bloqueo automático de trading 15 min antes/después de evento alto. <br> - Ajuste de exposición según sensibilidad del activo. | `EventFeedAdapter`, `StrategyEngine` | - 100% de eventos altos gestionados. <br> - Falsos bloqueos <5%. |
| **`WebDashboard`** | - Tecnología: Dash con WebSocket. <br> - Vistas: velas, cartera agrupada, radar, eventos, PnL. <br> - Filtros dinámicos por activo/clase. | Todos los módulos | - Actualización en tiempo real (<1s). <br> - Funciona en móvil y desktop. |

---

## 🔁 Fase 6: Backtesting Unificado y Optimización  
**Duración estimada:** 2 semanas  
**Objetivo:** Validar todo el sistema en simulación antes de producción.

### Componentes:

| Componente | Especificaciones Técnicas | Dependencias | Criterios de Aceptación |
|-----------|--------------------------|--------------|--------------------------|
| **`BacktestingFramework`** | - Stack: `VectorBT` (rápido) + `backtrader` (fidelidad). <br> - Mismo código que producción (`StrategyEngine`, `OMS`). <br> - Simulación de slippage, latencia, costos. | Todos los módulos | - Paridad backtest-producción >95%. <br> - Walk-forward analysis obligatoria. |
| **`PerformanceAnalyzer`** | - Reportes con `quantstats`: Sharpe, Sortino, Calmar, drawdown. <br> - Comparación contra benchmark (SP500, BTC). | `BacktestingFramework` | - Reporte HTML interactivo generado automáticamente. <br> - Métricas auditables y reproducibles. |

---

## ✅ Fase 7: Pruebas Finales, Documentación y Lanzamiento  
**Duración estimada:** 1 semana  
**Objetivo:** Preparar el sistema para producción estable.

### Actividades:
- Pruebas de estrés (100 activos simultáneos).
- Auditoría de seguridad (MT5, dashboard).
- Documentación final: `README.md`, `SOFTWARE_ARCHITECTURE.md`, guía de usuario.
- Script de despliegue automatizado (Docker opcional).
- Checklist de lanzamiento: logging activo, fallbacks probados, kill switch funcional.

---

## 📌 Notas Finales

- **Total estimado:** 15 semanas (~4 meses).
- **Equipo recomendado:** 1–2 desarrolladores full-stack cuantitativos.
- **Priorización:** Si el tiempo es limitado, prioriza Fases 1–4 + backtesting (Fase 6).
- **Escalabilidad futura**: Los módulos críticos (Radar, Event Analyzer) pueden convertirse en microservicios.

```mermaid
gantt
    title Plan de Desarrollo: Sistema de Trading Cuantitativo Híbrido v1.3
    dateFormat  YYYY-MM-DD
    section Infraestructura Base
    Dual Data Pipeline           :a1, 2026-04-01, 14d
    Validación de Datos         :a2, after a1, 7d

    section Feature & Model Engine
    Feature Engine              :b1, 2026-04-15, 10d
    Model Engine (por activo)   :b2, after b1, 11d

    section Ensemble & Validación
    Ensemble Engine             :c1, 2026-05-06, 10d
    ContextualSignalValidator   :c2, after c1, 11d
    Strategy Engine             :c3, after c2, 5d

    section Gestión de Órdenes y Riesgo
    OMS                         :d1, 2026-05-27, 7d
    Account Manager             :d2, after d1, 7d
    Logging & Auditoría         :d3, after d2, 7d

    section Visualización y Monitoreo
    Radar de Acciones           :e1, 2026-06-17, 7d
    Economic Event Analyzer     :e2, after e1, 7d
    Web Dashboard               :e3, after e2, 7d

    section Backtesting y Optimización
    Backtesting Unificado       :f1, 2026-07-08, 10d
    Performance Analyzer        :f2, after f1, 4d

    section Lanzamiento Final
    Pruebas Finales             :g1, 2026-07-22, 3d
    Documentación y Checklist   :g2, after g1, 2d
```
