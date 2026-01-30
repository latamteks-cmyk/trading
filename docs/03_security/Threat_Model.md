# Threat Modeling Document - Plataforma de Trading Algorítmico

**Versión:** 1.0  
**Fecha:** 30 de enero de 2026  
**Documento Base:** Documento Rector v2.1  
**Rol Responsable:** Security Advisor

---

## 1. Descripción General

Este documento presenta el modelo de amenazas para la plataforma de trading algorítmico, utilizando el modelo STRIDE para identificar, clasificar y mitigar amenazas potenciales al sistema. El modelo de amenazas se alinea con los principios de seguridad multicapa definidos en el Documento Rector v2.1 y cumple con las mejores prácticas de seguridad para sistemas financieros.

---

## 2. Metodología STRIDE

El modelo STRIDE categoriza las amenazas en seis tipos:

- **S**poofing (Suplantación)
- **T**ampering (Manipulación)
- **R**epudiation (Negación)
- **I**nformation Disclosure (Divulgación de información)
- **D**enial of Service (Denegación de servicio)
- **E**levation of Privilege (Elevación de privilegios)

---

## 3. Arquitectura del Sistema para Modelado de Amenazas

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        THREAT MODELING ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐    HTTP/HTTPS     ┌─────────────────────────────────────────┐ │
│  │   Trader    │ ────────────────→ │            API Gateway                │ │
│  │   Persona   │                   │                                       │ │
│  └─────────────┘                   │  - Autenticación                      │ │
│                                    │  - Autorización                       │ │
│                                    │  - Rate Limiting                      │ │
│                                    └─────────────────────────────────────────┘ │
│                                               │                                 │
│                                               ▼                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    PRESENTATION LAYER                                 │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │   │
│  │  │   Flet UI       │  │   REST API      │  │   WebSocket             │ │   │
│  │  │                 │  │                 │  │                         │ │   │
│  │  │  (Python/Flet)  │  │  (FastAPI)      │  │  (Real-time updates)    │ │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                           │
│                                    ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                 APPLICATION LAYER                                     │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │   │
│  │  │ Trading Use     │  │ Risk Use        │  │ Authentication Use      │ │   │
│  │  │ Cases           │  │ Cases           │  │ Cases                   │ │   │
│  │  │                 │  │                 │  │                         │ │   │
│  │  │ - Order Mgmt    │  │ - Validation    │  │ - Login/Register        │ │   │
│  │  │ - Position Mgmt │  │ - Limits        │  │ - Session Mgmt          │ │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                           │
│                                    ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                   DOMAIN LAYER                                        │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │   │
│  │  │ Trading Domain  │  │ Risk Domain     │  │ User/Account Domain     │ │   │
│  │  │                 │  │                 │  │                         │ │   │
│  │  │ - Orders        │  │ - Limits        │  │ - Authentication        │ │   │
│  │  │ - Positions     │  │ - Exposure      │  │ - Authorization         │ │   │
│  │  │ - Strategies    │  │ - Correlation   │  │ - Sessions              │ │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                           │
│                                    ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                 INFRASTRUCTURE LAYER                                  │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │   │
│  │  │ MT5 Adapter     │  │ Database        │  │ Cache                   │ │   │
│  │  │                 │  │                 │  │                         │ │   │
│  │  │ - Connection    │  │ - PostgreSQL    │  │ - Redis                 │ │   │
│  │  │ - Protocol      │  │ - Encryption    │  │ - Session Store         │ │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                           │
│                                    ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                      SERVIDOR MT5                                     │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐ │   │
│  │  │ Market Data     │  │ Order Execution │  │ Account Management      │ │   │
│  │  │                 │  │                 │  │                         │ │   │
│  │  │ - Quotes/Bars   │  │ - Orders        │  │ - Balance/Positions     │ │   │
│  │  │ - News Feed     │  │ - Positions     │  │ - History               │ │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Análisis de Amenazas por Componente

### 4.1 API Gateway

| Amenaza | STRIDE | Impacto | Probabilidad | Mitigación |
|---------|--------|---------|--------------|------------|
| Suplantación de identidad | Spoofing | Alto | Media | MFA obligatorio, validación de tokens JWT |
| Manipulación de headers | Tampering | Medio | Baja | Validación de integridad, firmas digitales |
| Negación de envío de órdenes | Repudiation | Medio | Baja | Logging de todas las solicitudes con timestamps |
| Divulgación de tokens | Info Disclosure | Crítico | Baja | HTTPS obligatorio, almacenamiento seguro |
| Sobrecarga del gateway | DoS | Alto | Media | Rate limiting, circuit breaker, auto-scaling |
| Elevación de privilegios | EoP | Crítico | Baja | Validación estricta de roles y permisos |

### 4.2 Flet UI

| Amenaza | STRIDE | Impacto | Probabilidad | Mitigación |
|---------|--------|---------|--------------|------------|
| Inyección de scripts | Spoofing | Medio | Media | Sanitización de entradas, CSP |
| Manipulación de datos locales | Tampering | Bajo | Baja | Validación en backend de todas las operaciones |
| Captura de credenciales | Info Disclosure | Crítico | Media | HTTPS, no almacenar credenciales en cliente |
| Clickjacking | Info Disclosure | Medio | Baja | Headers de seguridad (X-Frame-Options) |
| Denegación de servicio local | DoS | Bajo | Baja | Validación de recursos consumidos |
| Acceso no autorizado | EoP | Crítico | Baja | Validación de sesión en cada operación |

### 4.3 REST API

| Amenaza | STRIDE | Impacto | Probabilidad | Mitigación |
|---------|--------|---------|--------------|------------|
| Inyección SQL | Spoofing | Crítico | Baja | Consultas parametrizadas, ORM |
| Manipulación de payloads | Tampering | Alto | Media | Validación de schemas, firmas |
| Negación de operaciones | Repudiation | Medio | Baja | Logging con identificadores únicos |
| Exposición de datos | Info Disclosure | Alto | Media | Control de acceso, enmascaramiento |
| Ataques de fuerza bruta | DoS | Medio | Media | Rate limiting, bloqueo temporal |
| Escalación de privilegios | EoP | Crítico | Baja | Validación de autorización en cada endpoint |

### 4.4 Motor de Trading

| Amenaza | STRIDE | Impacto | Probabilidad | Mitigación |
|---------|--------|---------|--------------|------------|
| Envío de órdenes fraudulentas | Spoofing | Crítico | Media | Validación de riesgo, doble firma |
| Manipulación de órdenes | Tampering | Crítico | Baja | Firmas digitales, checksums |
| Negación de operaciones | Repudiation | Medio | Baja | Logging detallado de todas las operaciones |
| Divulgación de estrategias | Info Disclosure | Alto | Media | Cifrado de estrategias, ACL |
| Paralización del motor | DoS | Alto | Baja | Kill switches, circuit breakers |
| Ejecución no autorizada | EoP | Crítico | Baja | Validación de autorización, controles de riesgo |

### 4.5 Adaptador MT5

| Amenaza | STRIDE | Impacto | Probabilidad | Mitigación |
|---------|--------|---------|--------------|------------|
| Suplantación de credenciales MT5 | Spoofing | Crítico | Baja | Almacenamiento seguro, rotación de credenciales |
| Manipulación de protocolo | Tampering | Crítico | Muy baja | Validación de protocolo, checksums |
| Negación de operaciones | Repudiation | Medio | Baja | Logging de todas las comunicaciones |
| Interceptación de credenciales | Info Disclosure | Crítico | Baja | Cifrado, aislamiento de red |
| Bloqueo de conexiones | DoS | Medio | Baja | Pool de conexiones, mecanismos de reconexión |
| Acceso no autorizado | EoP | Crítico | Muy baja | Aislamiento, controles de red |

### 4.6 Base de Datos

| Amenaza | STRIDE | Impacto | Probabilidad | Mitigación |
|---------|--------|---------|--------------|------------|
| Acceso no autorizado | Spoofing | Crítico | Baja | Control de acceso, autenticación |
| Manipulación de datos | Tampering | Crítico | Media | Auditoría, integridad referencial |
| Negación de acceso | Repudiation | Alto | Baja | Copias de seguridad, alta disponibilidad |
| Fuga de datos | Info Disclosure | Crítico | Media | Cifrado en reposo, máscaras de datos |
| Sobrecarga del sistema | DoS | Medio | Baja | Limitación de consultas, particionamiento |
| Escalación de privilegios | EoP | Crítico | Baja | Principio de menor privilegio, auditoría |

---

## 5. Análisis de Amenazas por Flujo de Datos

### 5.1 Flujo de Ejecución de Órdenes

```
Usuario → UI → API → Trading Engine → Risk Engine → MT5 Adapter → MT5 Server
```

**Amenazas críticas:**
- Intercepción de órdenes (Info Disclosure)
- Manipulación de parámetros (Tampering)
- Suplantación de identidad (Spoofing)
- Negación de servicio en puntos críticos (DoS)

**Mitigaciones:**
- Validación en cada capa
- Logging de todas las operaciones
- Controles de riesgo multicapa
- Mecanismos de auditoría

### 5.2 Flujo de Autenticación

```
Usuario → UI → API Gateway → Auth Service → Database
```

**Amenazas críticas:**
- Robo de credenciales (Info Disclosure)
- Ataques de fuerza bruta (DoS)
- Suplantación de sesión (Spoofing)
- Elevación de privilegios (EoP)

**Mitigaciones:**
- MFA obligatorio
- Rate limiting
- Tokens cortos con refresh
- Validación de dispositivos

### 5.3 Flujo de Datos de Mercado

```
MT5 Server → MT5 Adapter → Cache → UI
```

**Amenazas críticas:**
- Manipulación de precios (Tampering)
- Retraso intencionado (DoS)
- Acceso no autorizado a datos (Info Disclosure)

**Mitigaciones:**
- Validación de integridad
- Monitoreo de latencia
- Control de acceso a datos sensibles

---

## 6. Matriz de Riesgo

### 6.1 Riesgos de Alto Impacto

| Amenaza | Componente | Impacto | Probabilidad | Riesgo | Estado |
|---------|------------|---------|--------------|--------|--------|
| Pérdida de fondos por órdenes fraudulentas | Trading Engine | Crítico | Media | Alto | Mitigado |
| Compromiso de credenciales MT5 | MT5 Adapter | Crítico | Baja | Medio | Mitigado |
| Divulgación de estrategias de trading | Storage | Alto | Media | Alto | Mitigado |
| Paralización del sistema | Infraestructura | Alto | Baja | Medio | Mitigado |

### 6.2 Riesgos de Mediano Impacto

| Amenaza | Componente | Impacto | Probabilidad | Riesgo | Estado |
|---------|------------|---------|--------------|--------|--------|
| Ataques de fuerza bruta | API Gateway | Medio | Alta | Alto | Mitigado |
| Fugas de información de usuarios | Database | Medio | Media | Medio | Mitigado |
| Manipulación de datos históricos | Storage | Medio | Baja | Bajo | Mitigado |
| Denegación de servicio | UI | Medio | Media | Medio | Mitigado |

---

## 7. Estrategias de Mitigación

### 7.1 Prevención

**Controles Técnicos:**
- Validación de entrada en todas las capas
- Autenticación multifactor
- Cifrado de extremo a extremo
- Principio de menor privilegio
- Aislamiento de redes

**Controles Procesales:**
- Revisión de código
- Pruebas de seguridad
- Capacitación de personal
- Políticas de seguridad

### 7.2 Detección

**Monitoreo Continuo:**
- SIEM para correlación de eventos
- Detección de anomalías
- Supervisión de métricas de seguridad
- Alertas en tiempo real

**Auditoría:**
- Logging estructurado
- Revisión periódica de accesos
- Análisis forense
- Informes de cumplimiento

### 7.3 Respuesta

**Procedimientos:**
- Plan de respuesta a incidentes
- Equipos de respuesta
- Comunicación de incidentes
- Recuperación de servicios

**Herramientas:**
- SOAR para automatización
- Forensics para investigación
- Kill switches para contención

---

## 8. Evaluación de Efectividad

### 8.1 Pruebas de Penetración
- **Frecuencia**: Trimestral
- **Alcance**: Todas las capas del sistema
- **Metodología**: OWASP Testing Guide
- **Remediación**: Priorización por riesgo

### 8.2 Análisis de Amenazas
- **Frecuencia**: Semestral o tras cambios significativos
- **Metodología**: STRIDE, PASTA
- **Participantes**: Equipo de seguridad, arquitectos
- **Documentación**: Actualización del modelo de amenazas

### 8.3 Evaluación de Controles
- **Frecuencia**: Mensual
- **Indicadores**: KPIs de seguridad
- **Informe**: Tablero de mandos de seguridad
- **Acciones**: Mejora continua

---

## 9. Consideraciones Específicas para Trading Algorítmico

### 9.1 Riesgos de Trading
- **Latencia**: Manipulación de tiempos para ventaja injusta
- **Slippage**: Aprovechamiento de diferencias de precio
- **Front-running**: Acceso anticipado a órdenes
- **Correlación**: Estrategias que se afectan mutuamente

### 9.2 Controles de Riesgo
- **Kill switches**: Detención automática de operaciones
- **Límites de exposición**: Control de posición y riesgo
- **Validación de estrategias**: Pruebas de robustez
- **Supervisión humana**: Intervención manual cuando sea necesario

### 9.3 Seguridad de Algoritmos
- **Protección de IP**: Cifrado y control de acceso a estrategias
- **Integridad**: Prevención de manipulación de algoritmos
- **Aislamiento**: Separación de estrategias competitivas
- **Auditoría**: Supervisión de comportamiento de algoritmos

---

## 10. Herramientas de Modelado de Amenazas

### 10.1 Microsoft SDL Threat Modeling Tool
- **Uso**: Modelado visual de arquitectura
- **Beneficios**: Identificación sistemática de amenazas
- **Integración**: Parte del proceso de desarrollo seguro

### 10.2 OWASP Threat Dragon
- **Uso**: Modelado colaborativo de amenazas
- **Beneficios**: Acceso web, integración con CI/CD
- **Características**: Plantillas para diferentes arquitecturas

### 10.3 PASTA (Process for Attack Simulation and Threat Analysis)
- **Uso**: Análisis de amenazas basado en escenarios
- **Beneficios**: Enfoque centrado en el negocio
- **Aplicación**: Evaluación de impacto real de amenazas

---

## 11. Referencias

### 11.1 Estándares de Seguridad
- STRIDE - Microsoft Security Development Lifecycle
- OWASP Top 10 - Open Web Application Security Project
- ISO 27005 - Gestión de Riesgos de Seguridad de la Información
- NIST SP 800-30 - Guía para la Evaluación de Riesgos

### 11.2 Publicaciones Técnicas
- "The CERT Guide to Insider Threats" - Dawn Cappelli
- "Applied Cryptography" - Bruce Schneier
- "Security Engineering" - Ross Anderson
- "The Art of Software Security Assessment" - Mark Dowd

---

## 12. Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| CTO | [Nombre CTO] | _________________ | ___________ |
| Security Advisor | [Nombre SA] | _________________ | ___________ |
| Tech Lead | [Nombre TL] | _________________ | ___________ |

---

**Estado:** APROBADO PARA IMPLEMENTACIÓN  
**Versión:** 1.0