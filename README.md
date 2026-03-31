# 📈 Sistema Integral de Trading Algorítmico Multiactivos

**Versión:** 1.3  
**Última actualización:** 31 de marzo de 2026  

> Plataforma cuantitativa open-source para trading algorítmico multiactivo con gestión institucional de riesgo, validación contextual mediante IA especializada y arquitectura limpia y escalable.

---

## 🔍 Visión General

Este sistema automatiza estrategias de trading en **Forex, Commodities, Índices, Acciones y Criptomonedas**, combinando:

- **Análisis técnico adaptativo** (basado en regímenes de mercado),
- **Factores fundamentales cuantitativos**,
- **Eventos económicos y sentimiento de noticias**,
- **Modelos de Machine Learning especializados por activo**,
- **Validación contextual mediante IA (LLM) especializada por clase de activo**,
- **Gestión de riesgo a nivel de portfolio** (volatility targeting, correlación, drawdown).

Diseñado con **Clean Architecture**, es modular, auditable, reproducible y listo para producción en entornos retail o institucionales pequeños.

---

## 🧩 Componentes Clave

| Módulo | Función |
|-------|--------|
| **Data Layer** | Captura dual: MT5 (producción) + `yfinance`/`ccxt` (research). Validación en tiempo real. |
| **Feature Engine** | Genera indicadores técnicos, fundamentales y de sentimiento según el activo. |
| **Model Engine** | Modelos especializados: XGBoost+HMM (Forex), LSTM (Cripto), LightGBM (Acciones), etc. |
| **Ensemble Engine** | Fusión robusta de señales con pesos dinámicos y reglas de veto. |
| **ContextualSignalValidator** ✨ | Valida coherencia de la señal usando LLMs con prompts especializados por grupo de activo (petróleo, oro, soya, etc.). |
| **OMS** | Gestión de órdenes con stop dinámico, trailing, apalancamiento y validación de margen. |
| **Account Manager** | Supervisa equity, drawdown, exposición y riesgo total. |
| **Radar de Acciones** | Monitoreo proactivo de oportunidades en universos amplios. |
| **Economic Event Analyzer** | Ajusta exposición o bloquea trading ante eventos de alto impacto. |
| **Backtesting Unificado** | Mismo código para simulación (`VectorBT`/`backtrader`) y producción. |
| **Web Dashboard** | Visualización en tiempo real: velas, cartera, PnL, alertas y radar. |

---

## 🛠️ Stack Tecnológico

- **Lenguaje principal**: Python 3.11+
- **Procesamiento**: Polars, NumPy, Numba
- **ML**: XGBoost, LightGBM, Scikit-learn, Optuna, SHAP
- **NLP**: FinBERT / DistilBERT (vía Hugging Face)
- **Backtesting**: VectorBT (rápido), backtrader (fidelidad)
- **Análisis**: TA-Lib, hmmlearn, statsmodels
- **Datos**: `yfinance`, `ccxt`, `fredapi`, scraping controlado
- **Broker**: MetaTrader 5 (MT5)
- **Bases de datos**: 
  - **Producción/Auditoría**: SQLite
  - **Research/Históricos**: PostgreSQL
- **IA Contextual**: API de LLM (GPT-4o, Claude 3.5) o modelo local (Phi-3-finance)
- **Frontend**: Dash / Streamlit (web interactiva)
- **Logging & Auditoría**: Registro completo en base de datos

---

## 📁 Estructura del Proyecto
trading-system/
├── core/ # Entities / Lógica de negocio
│ ├── data_layer/
│ ├── feature_engine/
│ └── model_engine/
├── interactors/ # Use Cases
│ ├── ensemble_engine.py
│ ├── contextual_signal_validator.py ← ¡Nuevo en v1.3!
│ └── strategy_engine.py
├── adapters/ # Interface Adapters
│ ├── mt5_adapter.py
│ ├── yfinance_adapter.py
│ ├── radar_de_acciones.py
│ └── economic_event_analyzer.py
├── frameworks/ # Drivers
│ ├── oms/
│ ├── account_manager/
│ ├── dashboard/
│ └── backtesting/
├── config/
│ └── prompt_templates.db ← Base de prompts por grupo de activo
├── logs/
│ └── sqlite/ # Logs de producción
├── research/ # Entorno de investigación
│ └── notebooks/
└── README.md

---

## 🚀 Inicio Rápido

### Requisitos
- Python 3.11+
- MetaTrader 5 (con API habilitada)
- Cuenta en broker compatible (opcional para live)

### Instalación
```bash
git clone https://github.com/tu-usuario/trading-system.git
cd trading-system
pip install -r requirements.txt

### Configuración
Copia .env.example a .env y completa tus credenciales.
Define tus grupos de activos y prompts en config/prompt_templates.db.
Ejecuta el pipeline de datos inicial:

python core/data_layer/initialize_data.py

Modos de Ejecución
Research:
jupyter lab research/notebooks/

📊 Validación y Calidad
Paridad backtest-producción: mismo código en ambos entornos.
Walk-forward analysis obligatoria antes de ir a producción.
Logging completo: cada decisión, predicción y orden se registra.
Tests unitarios: cobertura >85% en core/ y interactors/.
Fallback robusto: si falla la API de IA, se reduce tamaño de posición.
📄 Documentación
Product Vision Document v1.3.pdf
Software Architecture Document v1.0.pdf
Guía de Desarrollo
Catálogo de Prompts por Activo
🤝 Contribuciones
¡Las contribuciones son bienvenidas! Por favor, sigue:
Crea una rama desde develop.
Escribe tests para nuevas funcionalidades.
Actualiza la documentación.
Envía un Pull Request con descripción clara.
⚠️ Advertencia Legal
Este software es para uso educativo e investigación cuantitativa.
El trading algorítmico conlleva riesgo significativo de pérdida de capital.
El autor no asume responsabilidad por decisiones de inversión basadas en este sistema.
📬 Contacto
Responsable: Edgar Gomez
Email: edgar.gomez@example.com
Fecha del proyecto: Enero 2026 – Marzo 2026


