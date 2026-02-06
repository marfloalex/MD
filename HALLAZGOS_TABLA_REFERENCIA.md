# HALLAZGOS DETALLADOS - TABLA DE REFERENCIA RÁPIDA

## 1. CREDENCIALES EXPUESTAS - TABLA DE ROTACIÓN

| ID | Tipo | Usuario | Contraseña | Ubicación | Servidor | BD | Ambiente | Estado |
|---|---|---|---|---|---|---|---|---|
| C001 | ACME API | XM_S_AcmeAplicacionPrd@xm.com.co | C&&8edRa7reN | web.config L18 | acmecalidadback.xm.com.co | ACME | DEV/MVM | 🔴 EXPUESTO |
| C002 | SQL Server | ADMMID | ADMMID | web.config L32 | 10.250.16.25 | BDMIDXM | DEV | 🔴 EXPUESTO |
| C003 | SQL Server/EF | ADMMID | ADMMID | web.config L33 | 10.250.16.25 | BDRAGXM | DEV | 🔴 EXPUESTO |
| C004 | Oracle | JOAQUINBERMUDEZ | mvm_joaquinbermudez | web.config L35 | ORCL-PDN1-AZURE | PDN | DEV | 🔴 EXPUESTO |
| C005 | SQL SSRS | ADMMID | ADMMID | web.config L36 | MVMSW523\SQLDEV2014 | BDRAGXM | DEV | 🔴 EXPUESTO |
| C006 | SQL Server | PASORAG | Hcwm5ZpKNOJEuW1L7UzfGh4+ | web.config L42 | COMEDXMAZ061.isamdnt.grupo-isa.com:3052 | BDMIDXM | TEST | 🔴 EXPUESTO |
| C007 | SQL Server PROD | PASORAG | eNluyLdJj!QhGV4%pRUrv1TqYm7A#st8zCkn$BD0WOa9wM6. | web.config L53 | COMEDXMV519.isamdnt.grupo-isa.com:3052 | BDRAGXM | PROD | 🔴 EXPUESTO |
| C008 | Oracle PROD | PASORAG | Q9a_BIg-EPxGMjVKtRmol4w | web.config L55 | XM_PDN1 | PDN | PROD | 🔴 EXPUESTO |
| C009 | SQL PROD | PASORAG | W4Yo#cDvC3xF.XNl9pmLEahJgI1GedO8B2PMbs!Zu70i5kr_ | web.config L51 | 172.16.1.163:3052 | BDMIDXM | PROD | 🔴 EXPUESTO |

---

## 2. VULNERABILIDADES SQL INJECTION

| Archivo | Línea | Método | Tipo | Parámetros Vulnerables | Severidad | Solución |
|---------|-------|--------|------|------------------------|-----------|----------|
| Transacciones.cs | 219 | ActualizarContactoPersona | Direct SQL Concat | CODIGONEG, CODIGOTCO, IDENTAGENT, CEDULA | 🔴 CRÍTICA | Usar parámetros @param |

---

## 3. WCF SECURITY MODE="None"

| Binding Name | Línea | Servicios Afectados | Security Level | Recomendación |
|--------------|-------|-------------------|-----------------|----------------|
| WSHttpBinding_IRealizacionSolicitudes | 98 | RealizacionSolicitudes | None | Transport (SSL) + Client Cert |
| WSHttpBinding_IRevisionSolicitudes | 105 | RevisionSolicitudes | None | Transport (SSL) + Windows Auth |
| WSHttpBinding_IGeneral | 112 | General | None | Transport (SSL) + Basic Auth |
| WSHttpBinding_IAdministracion | 119 | Administracion | None | Transport (SSL) + Windows Auth |
| WSHttpBinding_IIntegracionPDN | 126 | IntegracionPDN | None | Transport (SSL) + Certificate Auth |
| WSHttpBinding_IIntegracionMID | 133 | IntegracionMID | None | Transport (SSL) + Basic Auth |
| WSHttpBinding_INuevoRegfro | 140 | NuevoRegfro | None | Transport (SSL) + Client Cert |

---

## 4. SERVICE DEBUG EXPUESTO

| Servicio | Línea | Behavior | Estado | Acción |
|----------|-------|----------|--------|--------|
| RealizacionSolicitudes | 244 | RealizacionSolicitudesBehavior | EXPOSADO | Cambiar a false |
| General | 249 | GeneralBehavior | EXPOSADO | Cambiar a false |
| RevisionSolicitudes | 254 | RevisionSolicitudesBehavior | EXPOSADO | Cambiar a false |
| Administracion | 259 | AdministracionBehavior | EXPOSADO | Cambiar a false |
| IntegracionPDN | 264 | IntegracionPDNBehavior | EXPOSADO | Cambiar a false |
| IntegracionMID | 269 | IntegracionMIDBehavior | EXPOSADO | Cambiar a false |
| NuevoRegfro | 274 | NuevoRegfroBehavior | EXPOSADO | Cambiar a false |

---

## 5. PERSIST SECURITY INFO=TRUE

| Ubicación | Archivo | Línea | Connection String | Acción |
|-----------|---------|-------|-------------------|--------|
| BDOracle | web.config | 35 | `PERSIST SECURITY INFO=True;USER ID=JOAQUINBERMUDEZ` | Cambiar a False |
| XM.RAG.Reportes | web.config | 36 | `Persist Security Info=True;User ID=ADMMID` | Cambiar a False |
| Settings Designer | Settings.Designer.cs | 29 | Hardcoded connection | Mover a configuración |

---

## 6. CLASES EXCESIVAMENTE GRANDES

| Archivo | Ruta | Líneas | Métodos (Est.) | Criticidad | Plan |
|---------|------|--------|-----------------|------------|------|
| ControladoraRealizacionSolicitudes.cs | Negocio/XM.RAG.RealizacionSolicitudes/Controladora | 3315 | 100+ | 🔴 CRÍTICA | Dividir en 4-5 controladores |
| ControladoraTransaccion.cs | Negocio/XM.RAG.IntegracionPDN/Controladora | 1950 | 80+ | 🔴 CRÍTICA | Dividir en 2-3 controladores |
| FachadaGeneral.cs | Negocio/XM.RAG.General/Fachada | 1325 | 60+ | 🟡 MEDIA | Dividir en 2-3 fachadas |

---

## 7. MÉTODOS CON PARÁMETROS EXCESIVOS (>4)

| Archivo | Método | Parámetros | Primera Línea |
|---------|--------|-----------|---------------|
| General.svc.cs | RealizarCargaGestor | 5 | string casoId, string rutaArchivoDestino, string nombreArchivo, string nombreConfiguracionCarga, string nombreTabla |
| Administracion.svc.cs | ObtenerSolicitudesConNotificacion | 5 | string usuario, DateTime fechaInicial, DateTime fechaFinal, List<string> roles, List<string> estados |
| BrokerTransaccion.cs | RemoverConceptoBasico | 5 | string codigoNeg, string codigoCTO, string idAplicacion, string idModulo, string codigoEntidad |
| BrokerConsulta.cs | ObtenerConceptoBasicoEmpresa | 5 | string codigoNeg, string codigoCTO, string idAplicacion, string idModulo, string nitEmpresa |
| BrokerConsulta.cs | ObtenerConceptoBasico | 5 | string codigoNeg, string codigoCTO, string idAplicacion, string idModulo, string codigoEntidad |
| ControladoraTransaccion.cs | IntegrarModificacionInformacionAgente | 4 | Entidades.Solicitudes.Agente agente, Entidades.Solicitudes.Empresa empresa, Entidades.General.Bitacora bitacora, bool FechaEstado |

---

## 8. EXCEPCIONES GENÉRICAS SIN LOGGING

| Archivo | Línea | Patrón | Severidad |
|---------|-------|--------|-----------|
| Comun.cs | 41, 58, 78, 121, 163, 204, 235, 265, 297, 339, 367 | catch(Exception ex) { } | 🔴 CRÍTICA |
| GeneralDAO.cs | 206-209 | catch (Exception ex) { return null; throw ex; } | 🔴 CRÍTICA (Unreachable) |
| LogTecnico.cs | 70, 120 | catch (Exception exc) { } | 🔴 CRÍTICA |
| Comun.cs (Multiple) | Various | catch (Exception exception) { } | 🔴 CRÍTICA |

**Total:** 50+ instancias encontradas

---

## 9. LIBRERÍAS ANTIGUAS

| Librería | Versión Usada | Versión Actual | Antigüedad | Vulnerabilidades Conocidas | Acción |
|----------|---------------|---------------|-----------|---------------------------|--------|
| Microsoft.Practices.EnterpriseLibrary | 5.0.414.0 | 10.x | 13 años | Sí, múltiples | 🔴 MIGRAR URGENTE |
| Newtonsoft.Json | 4.5.0.0 | 13.x | 10 años | Sí (CVE-2022-41603, etc) | 🔴 ACTUALIZAR |
| iTextSharp | 5.5.0.0 | DEPRECATED | 7 años | Deprecado | 🔴 REEMPLAZAR |
| Oracle.DataAccess | 2.112.3.0 | 21.x | 5+ años | Sí | 🔴 ACTUALIZAR |
| System.Data.Linq | 4.5 Framework | CONGELADO | 12 años | Sí | 🔴 MIGRAR a EF Core |
| LinqExtender | 2.5.0.0 | MUERTO | 10+ años | Sí, sin actualizaciones | 🔴 REEMPLAZAR |

---

## 10. VALORES MÁGICOS (MAGIC NUMBERS/STRINGS)

| Archivo | Línea | Magic String/Number | Contexto | Recomendación |
|---------|-------|-------------------|----------|----------------|
| FachadaGeneral.cs | 327 | IdTipoSolicitud == 1, 3 | Tipos específicos de solicitud | Crear enum TipoSolicitud { ModificacionAgente=1, ModificacionContacto=3 } |
| FachadaGeneral.cs | 327 | "CREA", "NAIA" | Tipos de documento | Crear enum DocumentoCorto { Creacion="CREA", Adicion="NAIA" } |
| FachadaGeneral.cs | 1220 | IdTipoSolicitud == 7 | Inactivar contacto | Enum value: { InactivarContacto=7 } |
| FachadaGeneral.cs | 1223 | "SCUC" | Documento específico | Enum value: { ScuDocument="SCUC" } |

---

## 11. MATRIZ DE IMPACTO Y URGENCIA

```
┌─────────────────────────────────────────────────────────┐
│                 MATRIZ DE RIESGO                        │
│                                                         │
│ URGENCIA                                                │
│    ▲                                                    │
│    │   ┌──────────────┐        ┌──────────────┐        │
│  A │   │SQL Injection │        │Creds Exposed │        │
│  L │   │    (L:219)   │        │  (L:32-55)   │        │
│  T │   │   🔴 CRÍTICA │        │  🔴 CRÍTICA  │        │
│  O │   └──────────────┘        └──────────────┘        │
│    │                                                   │
│    │   ┌──────────────┐        ┌──────────────┐        │
│    │   │WCF sin Auth  │        │ Class Sizes  │        │
│    │   │   (7 casos)  │        │  >1000 lines │        │
│    │   │  🔴 CRÍTICA  │        │  🔴 CRÍTICA  │        │
│    │   └──────────────┘        └──────────────┘        │
│    │                                                   │
│    │   ┌──────────────┐        ┌──────────────┐        │
│    │   │ Debug Info   │        │Lib Versions  │        │
│    │   │ (7 services) │        │   10+ years  │        │
│    │   │  🔴 CRÍTICA  │        │  🔴 CRÍTICA  │        │
│    │   └──────────────┘        └──────────────┘        │
│    │                                                   │
│    └────────────────────────────────────────────────┬──
│                    COMPLEJIDAD/VOLUMEN →            │
│                                                     │
│    Timeline to Fix (Estimated):                    │
│    - Phase 1: 1 week (Immediate vulnerabilities)   │
│    - Phase 2: 1 month (Architectural issues)       │
│    - Phase 3: 1 quarter (Platform upgrade)         │
│    - Phase 4: 1 year (Full modernization)          │
└─────────────────────────────────────────────────────┘
```

---

## 12. CHECKLIST DE REMEDIACIÓN INMEDIATA

### Semana 1 - CRÍTICO
- [ ] Rotar credencial C001 (ACME)
- [ ] Rotar credencial C002 (SQL ADMMID)
- [ ] Rotar credencial C004 (Oracle JOAQUINBERMUDEZ)
- [ ] Rotar todas las credenciales PROD
- [ ] Migrar secretos a Azure Key Vault / HashiCorp Vault
- [ ] Deshabilitar `serviceDebug includeExceptionDetailInFaults` en PROD
- [ ] Cambiar todas las `PERSIST SECURITY INFO` a False
- [ ] Crear ticket de urgencia para SQL Injection en Transacciones.cs:219

### Mes 1 - ALTO
- [ ] Implementar autenticación en todos los 7 WCF bindings
- [ ] Implementar logging centralizado (CloudWatch / ELK / Splunk)
- [ ] Crear soporte de encriptación de connection strings
- [ ] Parchar SQL Injection + crear test unitarios
- [ ] Crear guía de seguridad de conexión a BD

### Trimestre 1 - MEDIO
- [ ] Refactorizar ControladoraRealizacionSolicitudes.cs (dividir en 4-5 clases)
- [ ] Refactorizar ControladoraTransaccion.cs (dividir en 2-3 clases)
- [ ] Crear abstracción DAO (reducer duplicación)
- [ ] Reemplazar magic numbers por enums/constantes
- [ ] Crear unit tests para métodos críticos

### Año 1 - LARGO PLAZO
- [ ] Migrar a .NET Core / .NET 6+
- [ ] Reemplazar Enterprise Library 5.0 con Microsoft.Extensions.*
- [ ] Migrar LINQ to SQL a Entity Framework Core
- [ ] Reemplazar iTextSharp 5.5 con alternativa (QuestPDF / SelectPdf)
- [ ] Evaluar migración de WCF a ASP.NET Core / gRPC

---

## 13. REFERENCIAS DE ESTÁNDARES

**Estándares de Seguridad Aplicables:**
- OWASP Top 10 2021
  - A03: Injection (SQL Injection risk)
  - A01: Broken Access Control (No WCF auth)
  - A02: Cryptographic Failures (Hardcoded credentials)

- CWE (Common Weakness Enumeration)
  - CWE-89: SQL Injection
  - CWE-798: Use of Hard-coded Credentials
  - CWE-434: Unrestricted Upload of File

- NIST Cybersecurity Framework
  - Identify: Asset management
  - Protect: Access control, encryption
  - Detect: Monitoring, logging

---

**Documento:** Tabla de Referencia Rápida - Análisis RAG  
**Generado:** 6 de febrero de 2026  
**Clasificación:** CONFIDENCIAL
