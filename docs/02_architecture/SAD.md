# Software Architecture Document (SAD) - Plataforma de Trading Algorítmico

**Versión:** 1.0  
**Fecha:** 30 de enero de 2026  
**Documento Base:** Documento Rector v2.1  
**Rol Responsable:** Tech Lead

---

## 1. Descripción General

Este documento describe la arquitectura del sistema de trading algorítmico basado en el modelo de Arquitectura Evolutiva definido en el Documento Rector v2.1. El sistema sigue los principios de Clean Architecture, Domain Driven Design y está diseñado para evolucionar desde un Modular Monolith inicial hacia una arquitectura orientada a eventos con servicios desacoplados.

---

## 2. Objetivos Arquitectónicos

### 2.1 Objetivos Primarios
- **Seguridad Financiera**: Implementación de controles de riesgo multicapa
- **Escalabilidad**: Arquitectura evolutiva que permite crecimiento progresivo
- **Observabilidad**: Monitorización completa de métricas técnicas y de trading
- **Fiabilidad**: Alta disponibilidad y tolerancia a fallos
- **Mantenibilidad**: Código limpio y bien estructurado según principios de Clean Architecture

### 2.2 Requisitos No Funcionales
- Latencia de ejecución de órdenes < 100ms
- Disponibilidad del sistema > 99.9%
- Tasa de error < 0.1%
- Tiempo de respuesta de UI < 2 segundos

---

## 3. Estilo Arquitectónico

### 3.1 Clean Architecture
La arquitectura sigue el patrón de Clean Architecture de Robert Martin con las siguientes capas:

```
PRESENTATION LAYER
├── Flet UI
├── REST API
├── WebSockets
├── GraphQL (futuro)

APPLICATION LAYER
├── Use Cases
├── Commands
├── Queries
├── DTOs
├── Validators

DOMAIN LAYER
├── Entities
├── Value Objects
├── Aggregates
├── Domain Events
├── Domain Services
├── Repositories (Interfaces)

INFRASTRUCTURE LAYER
├── MT5 Adapter
├── Persistence (PostgreSQL)
├── Cache (Redis)
├── External APIs
├── Message Broker (futuro)
├── Email Service
├── Notification Service
```

### 3.2 Principios Aplicados
- **Dependency Inversion**: Las capas internas no dependen de las externas
- **Single Responsibility**: Cada componente tiene una única razón para cambiar
- **Separation of Concerns**: Lógica de negocio separada de detalles técnicos
- **Open/Closed Principle**: Abierto para extensión, cerrado para modificación

---

## 4. Diagrama de Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Flet UI   │  │   REST API  │  │ WebSockets  │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 APPLICATION LAYER                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ Use Cases   │  │ Commands    │  │ Queries     │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│  ┌─────────────┐  ┌─────────────┐                              │
│  │ DTOs        │  │ Validators  │                              │
│  └─────────────┘  └─────────────┘                              │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DOMAIN LAYER                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ Entities    │  │ Value Obj.  │  │ Aggregates  │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ Domain Ev.  │  │ Dom. Serv.  │  │ Repos. Int. │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 INFRASTRUCTURE LAYER                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ MT5 Adapter │  │ Persistence │  │ Cache       │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ Ext. APIs   │  │ Msg. Broker │  │ Notifications│            │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Componentes Principales

### 5.1 Motor de Trading
- **Responsabilidad**: Ejecución de órdenes, gestión de posiciones
- **Tecnología**: Python con integración MT5
- **Patrones**: State Machine, Command Pattern
- **Interacciones**: UI, Risk Engine, MT5

### 5.2 Motor de Riesgo
- **Responsabilidad**: Validación de órdenes, controles de exposición
- **Tecnología**: Python con reglas de negocio
- **Patrones**: Strategy Pattern, Chain of Responsibility
- **Interacciones**: Trading Engine, Database, Alerts

### 5.3 Adaptador MT5
- **Responsabilidad**: Conexión con servidor MT5, manejo de latencia
- **Tecnología**: Python-MetaTrader5, pool de conexiones
- **Patrones**: Adapter Pattern, Connection Pool
- **Interacciones**: Trading Engine, Market Data

### 5.4 API Gateway
- **Responsabilidad**: Autenticación, enrutamiento, rate limiting
- **Tecnología**: FastAPI
- **Patrones**: Facade Pattern, Circuit Breaker
- **Interacciones**: Todos los servicios

---

## 6. Flujo de Datos

### 6.1 Flujo de Ejecución de Órdenes
```
1. UI envía orden → API Gateway
2. API Gateway autentica → Trading Engine
3. Trading Engine → Risk Engine (validación)
4. Risk Engine aprueba → MT5 Adapter
5. MT5 Adapter → MT5 Server
6. Resultado ← MT5 Server
7. Actualización estado → Database
8. Notificación → WebSocket → UI
```

### 6.2 Flujo de Gestión de Riesgo
```
1. Market Data → Risk Engine
2. Risk Engine monitorea posiciones
3. Si límite excedido → Alerta
4. Si riesgo crítico → Kill Switch
5. Notificación → Admin UI
```

---

## 7. Tecnologías y Stack

### 7.1 Backend
- **Lenguaje**: Python 3.11+
- **Framework Web**: FastAPI
- **Base de Datos**: PostgreSQL 14+
- **Cache**: Redis
- **Conexión MT5**: python-mt5
- **ORM**: SQLAlchemy
- **Autenticación**: JWT con refresh tokens

### 7.2 Frontend
- **Framework**: Flet
- **Lenguaje**: Python
- **Estilos**: Material Design
- **WebSocket**: Real-time updates

### 7.3 Infraestructura
- **Contenedores**: Docker
- **Orquestación**: Docker Compose (MVP), Kubernetes (futuro)
- **Monitorización**: Prometheus + Grafana
- **Logging**: ELK Stack o similar
- **CI/CD**: GitHub Actions

---

## 8. Patrones de Diseño

### 8.1 Patrones de Creación
- **Factory Pattern**: Creación de objetos de dominio
- **Builder Pattern**: Construcción de órdenes complejas
- **Singleton Pattern**: Configuración global

### 8.2 Patrones de Estructura
- **Adapter Pattern**: Integración MT5
- **Facade Pattern**: API Gateway
- **Proxy Pattern**: Control de acceso a recursos

### 8.3 Patrones de Comportamiento
- **Observer Pattern**: Notificaciones de eventos
- **State Pattern**: Estados de órdenes
- **Strategy Pattern**: Diferentes tipos de órdenes
- **Command Pattern**: Ejecución de órdenes

---

## 9. Consideraciones de Seguridad

### 9.1 Autenticación
- JWT tokens con expiración corta
- Refresh tokens con rotación
- MFA obligatorio para cuentas reales

### 9.2 Autorización
- Control basado en roles (RBAC)
- Permisos granulares por funcionalidad
- Auditoría de todas las acciones

### 9.3 Datos
- Encriptación AES-256 en reposo
- TLS 1.3 para comunicación
- Validación de entrada en todos los niveles

---

## 10. Consideraciones de Riesgo

### 10.1 Controles de Riesgo
- Límites diarios/semanales de pérdida
- Control de exposición por estrategia
- Detección de volatilidad extrema
- Kill Switch global

### 10.2 Validación
- Simulación antes de ejecución real
- Validación de parámetros
- Control de correlación entre estrategias

---

## 11. Escalabilidad

### 11.1 Fase MVP (Sprints 0-3)
- Modular Monolith
- Único nodo
- PostgreSQL + Redis

### 11.2 Fase Escalamiento (Sprint 4+)
- Microservicios desacoplados
- Message Broker (Kafka/RabbitMQ)
- Auto-scaling
- Load balancing

---

## 12. Observabilidad

### 12.1 Métricas Técnicas
- Latencia MT5
- Tiempo de respuesta API
- Tiempo de ejecución de órdenes
- Uso de recursos

### 12.2 Métricas de Trading
- Slippage
- Fill ratio
- Calidad de ejecución
- Drawdown en tiempo real

---

## 13. Despliegue

### 13.1 MVP
- Docker Compose
- Single node
- PostgreSQL local
- Redis local

### 13.2 Producción
- Kubernetes
- Horizontal scaling
- PostgreSQL cluster
- Redis cluster
- Backup automático

---

## 14. Referencias
- Documento Rector v2.1
- Clean Architecture - Robert Martin
- Domain Driven Design - Eric Evans
- Building Evolutionary Architectures - Ford et al.
- Designing Data-Intensive Applications - Kleppmann

---

## 15. Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| CTO | [Nombre CTO] | _________________ | ___________ |
| Tech Lead | [Nombre TL] | _________________ | ___________ |

---

**Estado:** APROBADO PARA IMPLEMENTACIÓN  
**Versión:** 1.0