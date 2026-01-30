# RACI Matrix - Plataforma de Trading Algorítmico

**Versión:** 1.0  
**Fecha:** 30 de enero de 2026  
**Documento Maestro:** Documento Rector v2.1

---

## Descripción

La matriz RACI (Responsible, Accountable, Consulted, Informed) define claramente quién es responsable de cada actividad crítica en el proyecto de la plataforma de trading algorítmico. Esta matriz asegura la transparencia en la asignación de responsabilidades y facilita la toma de decisiones.

**Leyenda:**
- **R (Responsible)**: Quién realiza la tarea o proceso
- **A (Accountable)**: Quién aprueba o firma la tarea (uno por fila)
- **C (Consulted)**: Quién debe ser consultado antes de tomar decisiones
- **I (Informed)**: Quién debe ser informado después de la decisión

---

## Matriz de Responsabilidades

| Actividad | CTO | Tech Lead | Backend Dev | Quant Dev | DevOps Eng | Security Adv | Project Manager |
|-----------|-----|-----------|-------------|-----------|------------|--------------|-----------------|
| Aprobación de Arquitectura | A | R | C | C | C | C | I |
| Desarrollo de Dominio | I | A | R | C | I | C | I |
| Implementación de Seguridad | C | A | R | I | C | A | I |
| Integración MT5 | I | A | R | I | I | C | I |
| Desarrollo de UI | I | A | R | I | I | C | I |
| Validación Cuantitativa | I | C | I | A | I | I | I |
| Deployment | I | C | I | I | A | C | I |
| Seguimiento de Riesgos | A | C | I | I | I | A | R |
| Gestión de Incidentes | A | C | R | I | A | A | R |
| Auditoría de Seguridad | A | I | I | I | I | A | I |
| Documentación Técnica | I | A | R | I | I | C | R |
| Pruebas de Integración | I | A | R | I | I | C | R |
| Validación de Modelos | I | C | I | A | I | I | I |
| Cumplimiento Normativo | A | C | I | I | I | A | I |

---

## Módulos del Sistema

| Módulo | CTO | Tech Lead | Backend Dev | Quant Dev | DevOps Eng | Security Adv | Project Manager |
|--------|-----|-----------|-------------|-----------|------------|--------------|-----------------|
| Motor de Trading | A | A | R | I | I | C | I |
| Gestión de Riesgos | A | A | R | C | I | A | I |
| Autenticación | I | A | R | I | I | A | I |
| Persistencia | I | A | R | I | A | A | I |
| Cache | I | A | R | I | A | C | I |
| Logging | I | A | R | I | A | C | I |
| Monitoreo | I | A | R | I | A | C | I |
| API Gateway | I | A | R | I | A | C | I |
| WebSocket | I | A | R | I | A | C | I |

---

## Documentos Clave

| Documento | CTO | Tech Lead | Backend Dev | Quant Dev | DevOps Eng | Security Adv | Project Manager |
|-----------|-----|-----------|-------------|-----------|------------|--------------|-----------------|
| Documento Rector | A | R | I | I | I | C | I |
| Project Charter | A | C | I | I | I | C | R |
| SAD | A | A | C | I | C | C | I |
| Security Arch | A | A | C | I | C | A | I |
| Threat Model | I | C | I | I | C | A | I |
| MT5 Integration | I | A | R | I | I | C | I |
| Risk Management | A | A | R | C | I | A | I |
| API Spec | I | A | R | I | I | C | I |
| Test Plan | I | A | R | C | A | C | R |

---

## Procesos de Negocio

| Proceso | CTO | Tech Lead | Backend Dev | Quant Dev | DevOps Eng | Security Adv | Project Manager |
|---------|-----|-----------|-------------|-----------|------------|--------------|-----------------|
| Onboarding Usuario | I | A | R | I | I | A | I |
| Validación Trading | A | A | R | I | I | A | I |
| Control de Riesgo | A | A | R | C | I | A | I |
| Ejecución Órdenes | I | A | R | I | I | A | I |
| Alertas y Notificaciones | I | A | R | I | A | C | I |
| Backup y Recuperación | I | C | R | I | A | A | I |
| Auditoría Financiera | A | I | I | I | I | A | I |

---

## Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| CTO | [Nombre CTO] | _________________ | ___________ |
| Project Manager | [Nombre PM] | _________________ | ___________ |

---

**Estado:** APROBADO PARA IMPLEMENTACIÓN  
**Versión:** 1.0