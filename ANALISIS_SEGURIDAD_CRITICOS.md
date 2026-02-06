# 🔒 ANÁLISIS DE SEGURIDAD Y RIESGOS TÉCNICOS

**Sistema**: XM.RAG  
**Fecha**: 6 de Febrero de 2026  
**Clasificación**: Confidencial - Riesgos de Seguridad  
**Audiencia**: CISO, Arquitectura de Seguridad, Team Líderes

---

## 📋 TABLA DE CONTENIDOS

1. [Hallazgos de Seguridad Críticos](#hallazgos-críticos)
2. [Matriz de Riesgos](#matriz-de-riesgos)
3. [Vulnerabilidades Detalladas](#vulnerabilidades-detalladas)
4. [Plan de Remediación](#plan-de-remediación)

---

## 🔴 HALLAZGOS CRÍTICOS

### 1. CREDENCIALES HARDCODEADAS Y EXPUESTAS

**Severidad**: 🔴 CRÍTICA  
**CVSS**: 9.8 (Información Disclosure)

#### Ubicación 1: web.config (SQL Server - BDRAGXM)

```xml
<!-- Línea 32-33 -->
<add name="BDRAG" 
     connectionString="...;uid=ADMMID;password=ADMMID;..."/>
```

| Campo | Valor | Exposición |
|-------|-------|-----------|
| **Usuario** | ADMMID | En archivo public (control de versiones) |
| **Contraseña** | ADMMID | En archivo public (control de versiones) |
| **Riesgo** | Acceso total a BDRAGXM | Datos sensibles expuestos |
| **Impacto** | Lectura/Escritura de todas solicitudes | Medium |

#### Ubicación 2: web.config (SQL Server - BDMIDXM)

```xml
<!-- Línea 32 -->
<add name="LINQ2MIDConnectionString" 
     connectionString="server=10.250.16.25;uid=ADMMID;password=ADMMID;..."/>
```

**Mismas credenciales que ubicación 1**

#### Ubicación 3: web.config (Oracle - PDN)

```xml
<!-- Línea 35 -->
<add name="BDOracle" 
     connectionString="...PASSWORD=mvm_joaquinbermudez;USER ID=JOAQUINBERMUDEZ..."/>
```

| Campo | Valor | Exposición |
|-------|-------|-----------|
| **Usuario** | JOAQUINBERMUDEZ | En archivo public |
| **Contraseña** | mvm_joaquinbermudez | En archivo public |
| **Riesgo** | Acceso a base de permisos críticos | Datos de autorización expuestos |
| **Impacto** | Podría escalar permisos ilegalmente | Critical |

#### Ubicación 4: web.config (ACME - Sistema Externo)

```xml
<!-- Línea 23 -->
<add key="ACME_Password" value="C&&8edRa7reN"/>
```

| Campo | Valor | Exposición |
|-------|-------|-----------|
| **Contraseña** | C&&8edRa7reN | En archivo public |
| **Riesgo** | Acceso a sistema integrado ACME | Integración externa comprometida |
| **Impacto** | Podría hacer operaciones bancarias | Critical |

#### Ubicación 5-11: Scripts de Despliegue PowerShell

En `FUENTES/Commands/*.ps1`:
- Contraseñas de SharePoint en variables comentadas
- Estructura hardcodeada para token replacement
- Almacenamiento de contraseñas en scripts versionados

#### Remediación Inmediata (Semana 1)

```powershell
# 1. Mover a Azure Key Vault
$keyVault = Get-AzKeyVault -VaultName "RAG-Secrets"
$secret = Get-AzKeyVaultSecret -VaultName "RAG-Secrets" -Name "sql-admmid-password"

# 2. Rotación de credenciales
# - Cambiar contraseña ADMMID en SQL Server
# - Cambiar contraseña JOAQUINBERMUDEZ en Oracle
# - Cambiar credencial ACME_Password
# - Revocar acceso anterior

# 3. Verificar logs de acceso para compromiso potencial
# SELECT * FROM sys.dm_exec_sessions WHERE login_name = 'ADMMID'
```

---

### 2. SERVICIOS WCF SIN AUTENTICACIÓN

**Severidad**: 🔴 CRÍTICA  
**CVSS**: 9.0 (Unauthorized Access)

**7 servicios configurados con `security mode="None"`**:

```xml
<wsHttpBinding>
  <binding name="WSHttpBinding_IRealizacionSolicitudes" 
           maxReceivedMessageSize="2147483647">
    <security mode="None">
      <transport clientCredentialType="None"/>
      <message clientCredentialType="UserName"/>
    </security>
  </binding>
  <!-- Similar para: RevisionSolicitudes, General, Administracion, IntegracionMID, IntegracionPDN, NuevoRegfro -->
</wsHttpBinding>
```

**Impacto**:
- ❌ **Cualquier cliente HTTP puede llamar servicios** sin validar identidad
- ❌ **No hay autorización a nivel WCF** (confía en ID del cliente)
- ❌ **Puerta abierta a ataques de negación de servicio**
- ❌ **Datos sensibles transmitidos sin cifrado TLS**

**Ataque de ejemplo**:
```python
# Cliente malicioso puede hacer:
import requests

# Crear solicitud sin autenticación
solicitud = {
    "IdEmpresa": 1,
    "NombreEmpresa": "Empresa Maliciosa",
    "NIT": "1234567890"
}

response = requests.post(
    "http://serv idor:8732/Services/RealizacionSolicitudes",
    json=solicitud
)
# ✅ Éxito - Solicitud creada sin autorización
```

**Remediación** (Inmediata):

```xml
<!-- Cambiar a WSHttpBinding con seguridad -->
<wsHttpBinding>
  <binding name="WSHttpBinding_IRealizacionSolicitudes">
    <security mode="TransportWithMessageCredential">
      <transport clientCredentialType="Certificate"/>
      <message clientCredentialType="UserName"/>
    </security>
  </binding>
</wsHttpBinding>

<!-- Implementar validación de cliente -->
[OperationContract]
[FaultContract(typeof(ServiceFault))]
public void CrearSolicitud(SolicitudDTO solicitud)
{
    var userName = OperationContext.Current.ServiceSecurityContext.PrimaryIdentity.Name;
    if (string.IsNullOrEmpty(userName))
        throw new FaultException("Autenticación requerida");
    
    // Validar autorización
    if (!ValidarPermiso(userName, "CrearSolicitud"))
        throw new FaultException("No autorizado");
}
```

---

### 3. DEBUG INFORMATION EXPUESTO

**Severidad**: 🔴 CRÍTICA  
**CVSS**: 7.5 (Information Disclosure)

```xml
<!-- web.config línea ~175 -->
<serviceDebug includeExceptionDetailInFaults="true"/>
<serviceMetadata httpGetEnabled="true"/>
```

**Impacto**:
- ⚠️ Excepciones retornan stack traces completos a clientes
- ⚠️ Metadata WSDL pública expone estructura de servicios
- ⚠️ Nombres de tablas, stored procs, rutas de archivo revelados
- ⚠️ Información de versiones de .NET/SQL Server expuesta

**Ejemplo de exposición**:
```
Exception expuesta al cliente:
"Error en BrokerRevisionSolicitudes.ProcesarSolicitud() 
 línea 234 en c:\Proyectos\RAG\Fuentes\...
 at XM.RAG.Negocio.Administracion.BrokerAdministracion
 SelectMultiple FROM Solicitudes WHERE IdEmpresa = '1' 
 AND Estado LIKE '%Rechazada%' 
 Line 219: unrecognized escape sequence'\R'"
```

**Atacante aprende**:
- Estructura de código fuente
- Nombres exactos de tablas/campos
- Patrones de SQL usado
- Rutas del servidor

**Remediación** (Inmediata - Producción):

```xml
<!-- Producción -->
<serviceDebug includeExceptionDetailInFaults="false"/>

<!-- Desarrollo/Testing solamente -->
<system.diagnostics>
  <trace autoflush="true" useGlobalLock="false">
    <listeners>
      <add name="textWriterTraceListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="C:\Logs\WCFTrace.log"/>
    </listeners>
  </trace>
  <sources>
    <source name="System.ServiceModel" switchValue="Verbose" propagateActivity="true">
      <listeners>
        <add name="textWriterTraceListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="C:\Logs\WCFTrace.log"/>
      </listeners>
    </source>
  </sources>
</system.diagnostics>
```

---

### 4. SQL INJECTION VULNERABILI DADES

**Severidad**: 🔴 CRÍTICA  
**CVSS**: 9.8 (Remote Code Execution vía SQL)

#### Ubicación: BrokerReportes.cs (línea ~219)

```csharp
// VULNERABLE - Construcción manual de SQL
public DataSet GenerarReporteSolicitudes(
    string estado,
    string empresa,
    DateTime fecha)
{
    string sql = "SELECT * FROM Solicitudes WHERE 1=1";
    
    if (!string.IsNullOrEmpty(estado))
        sql += " AND Estado = '" + estado + "'";  // 🔴 SQL INJECTION
    
    if (!string.IsNullOrEmpty(empresa))
        sql += " AND NombreEmpresa LIKE '%" + empresa + "%'";  // 🔴 SQL INJECTION
    
    sql += " AND FechaCreacion >= '" + fecha.ToShortDateString() + "'";  // 🔴 SQL INJECTION
    
    var db = new BDRAG();
    return db.ExecuteDataSet(sql);
}
```

#### Ejemplos de Ataque

**Ataque 1: Inyección de lógica booleana**
```
Entrada: estado = "' OR '1'='1"
SQL resultante: SELECT * FROM Solicitudes WHERE Estado = '' OR '1'='1'
Resultado: Retorna TODAS las solicitudes sin validación
```

**Ataque 2: UNION-based SQL Injection**
```
Entrada: empresa = "' UNION SELECT user_name, password FROM usuarios --"
SQL resultante: 
  SELECT * FROM Solicitudes 
  UNION SELECT user_name, password FROM usuarios --
Resultado: Extrae credenciales de usuarios
```

**Ataque 3: Stacked Queries (si permitido)**
```
Entrada: estado = "'; DROP TABLE Solicitudes; --"
SQL resultante: 
  SELECT * FROM Solicitudes WHERE Estado = ''; 
  DROP TABLE Solicitudes; --'
Resultado: Destrucción de datos (negación de servicio)
```

**Remediación** (Inmediata):

```csharp
// ✅ SEGURO - Parametrización
public DataSet GenerarReporteSolicitudes(
    string estado,
    string empresa,
    DateTime fecha)
{
    string sql = @"
        SELECT * FROM Solicitudes 
        WHERE 1=1
          AND (@Estado IS NULL OR Estado = @Estado)
          AND (@Empresa IS NULL OR NombreEmpresa LIKE @Empresa)
          AND FechaCreacion >= @Fecha";
    
    var parameters = new List<SqlParameter>
    {
        new SqlParameter("@Estado", string.IsNullOrEmpty(estado) ? DBNull.Value : (object)estado),
        new SqlParameter("@Empresa", 
            string.IsNullOrEmpty(empresa) ? DBNull.Value : (object)("%" + empresa + "%")),
        new SqlParameter("@Fecha", fecha)
    };
    
    using (var connection = new SqlConnection(connectionString))
    using (var command = new SqlCommand(sql, connection))
    {
        command.Parameters.AddRange(parameters.ToArray());
        using (var adapter = new SqlDataAdapter(command))
        {
            var result = new DataSet();
            adapter.Fill(result);
            return result;
        }
    }
}
```

O con Entity Framework:
```csharp
using (var context = new BDRAG())
{
    var query = context.Solicitudes
        .Where(s => string.IsNullOrEmpty(estado) || s.Estado == estado)
        .Where(s => string.IsNullOrEmpty(empresa) || s.Empresa.NombreEmpresa.Contains(empresa))
        .Where(s => s.FechaCreacion >= fecha)
        .AsEnumerable();  // Evaluar post-cast para LIKE
    
    return query.ToList();
}
```

---

### 5. PERSIST SECURITY INFO HABILITADO

**Severidad**: 🟡 ALTA  
**CVSS**: 6.5 (Information Disclosure)

```xml
<!-- Connection strings en web.config -->
<add name="BDOracle" 
     connectionString="...PERSIST SECURITY INFO=True;USER ID=JOAQUINBERMUDEZ;PASSWORD=..."/>
```

**Impacto**:
- ⚠️ Si aplicación o proceso captura conexión, puede leer contraseña
- ⚠️ Profilers de BD pueden exposer credenciales
- ⚠️ Pool de conexiones puede retener credenciales en memoria

**Remediación**:
```xml
<!-- Cambiar a False (defecto) -->
<add name="BDOracle" 
     connectionString="...PERSIST SECURITY INFO=False;USER ID=JOAQUINBERMUDEZ;PASSWORD=..."/>
```

---

## 📊 MATRIZ DE RIESGOS

| # | Riesgo | Severidad | Impacto | Probabilidad | Score | Plazo |
|---|--------|-----------|---------|-------------|-------|-------|
| 1 | Credenciales hardcodeadas | 🔴 Crítica | 10 | 10 | 100 | **INMEDIATO** |
| 2 | WCF sin autenticación | 🔴 Crítica | 10 | 8 | 80 | **INMEDIATO** |
| 3 | Debug info expuesto | 🔴 Crítica | 8 | 9 | 72 | **INMEDIATO** |
| 4 | SQL Injection | 🔴 Crítica | 10 | 7 | 70 | **INMEDIATO** |
| 5 | PERSIST SECURITY INFO | 🟡 Alta | 6 | 7 | 42 | Mes 1 |
| 6 | Active Directory SPOF | 🟡 Alta | 8 | 3 | 24 | Mes 2 |
| 7 | Caché sin protección | 🟠 Media | 5 | 5 | 25 | Mes 3 |
| 8 | Logs no centralizados | 🟠 Media | 4 | 8 | 32 | Mes 2 |
| 9 | Librerías obsoletas | 🟠 Media | 6 | 6 | 36 | Trimestre 1 |
| 10 | Sin MFA | 🟠 Media | 7 | 5 | 35 | Trimestre 1 |

---

## 🛠️ PLAN DE REMEDIACIÓN

### FASE 1: CRÍTICA (Semanas 1-2)

```
Semana 1:
├─ [ ] Auditoría de git/versioning - ¿Cuándo se expusieron credenciales?
├─ [ ] Análisis de logs de acceso de BD - ¿Acceso no autorizado detectado?
├─ [ ] Cambiar TODAS las contraseñas (SQL, Oracle, ACME)
├─ [ ] Revocar credenciales ADMMID y JOAQUINBERMUDEZ del sistema
└─ [ ] Crear nuevas credenciales de servicio con permisos mínimos

Semana 2:
├─ [ ] Implementar Azure Key Vault o similiar
├─ [ ] Actualizar connection strings a usar Key Vault
├─ [ ] Habilitar HTTPS/TLS 1.2+ en WCF endpoints
├─ [ ] Deshabilitar security mode="None" en WCF
└─ [ ] Pruebas de conectividad post-cambios
```

### FASE 2: ALTA (Mes 1)

```
├─ [ ] Auditoría de código para SQL Injection (scan automático)
├─ [ ] Remediar todas las queries dinámicas
├─ [ ] Deshabilitar includeExceptionDetailInFaults en producción
├─ [ ] Implementar autenticación en todos los WCF endpoints
├─ [ ] Validación de autorización (authorization context)
└─ [ ] Pruebas de penetración básica
```

### FASE 3: MEDIA (Trimestre 1)

```
├─ [ ] Migración de Enterprise Library → Serilog
├─ [ ] Centralizar logs en SIEM (ELK, Splunk, Azure Monitor)
├─ [ ] Implementar Web Application Firewall (WAF)
├─ [ ] Configurar alertas de seguridad
├─ [ ] Audit trail para accesos a datos sensibles
└─ [ ] Capacitación de OWASP Top 10 para equipo
```

---

## 📋 CHECKLIST DE CUMPLIMIENTO

- [ ] Auditoría de seguridad completada
- [ ] Credenciales rotadas
- [ ] WCF con autenticación y autorización
- [ ] SQL Injection remedida
- [ ] HTTPS habilitado
- [ ] Debug information deshabilitado en producción
- [ ] Secrets management implementado
- [ ] WAF configurado
- [ ] Logging centralizado
- [ ] Penetration testing completado
- [ ] GDPR/Compliance audit
- [ ] Documentación de security updated

---

**Documento de seguridad confindencial - No distribuir sin autorización**  
**Próxima revisión**: Post-remediación (2 semanas)

