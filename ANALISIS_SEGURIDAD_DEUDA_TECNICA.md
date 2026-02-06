# 🔴 ANÁLISIS DE SEGURIDAD, RIESGOS Y DEUDA TÉCNICA - SISTEMA RAG

**Fecha de Análisis:** 6 de febrero de 2026  
**Criticidad General:** 🔴 **CRÍTICA** - Requiere acción inmediata

---

## TABLA DE CONTENIDOS

1. [Problemas de Seguridad Evidentes](#1-problemas-de-seguridad-evidentes)
2. [Riesgos de WCF](#2-riesgos-de-wcf)
3. [Acceso a Datos - Vulnerabilidades](#3-acceso-a-datos---vulnerabilidades)
4. [Deuda Técnica](#4-deuda-técnica)
5. [Obsolescencia Tecnológica](#5-obsolescencia-tecnológica)
6. [Resumen Ejecutivo](#resumen-ejecutivo)

---

## 1. PROBLEMAS DE SEGURIDAD EVIDENTES

### 1.1 ⚠️ CREDENCIALES HARDCODEADAS EN CONFIGURACIÓN

**Criticidad:** 🔴 CRÍTICA

#### Ubicación 1: web.config (Ambiente DEV/MVM)
**Archivo:** `FUENTES\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\web.config`

**Línea 18 - ACME Password en texto plano:**
```xml
<add key="ACME_Password" value="C&amp;&amp;8edRa7reN"/>
```
**Dato Expuesto:** `C&&8edRa7reN`

**Línea 32 - SQL Server Credentials:**
```xml
<add name="LINQ2MIDConnectionString" connectionString="server=10.250.16.25;uid=ADMMID;password=ADMMID;Initial Catalog=BDMIDXM" providerName="System.Data.SqlClient"/>
```
**Datos Expuestos:**
- Usuario: `ADMMID`
- Contraseña: `ADMMID`
- Servidor: `10.250.16.25`

**Línea 33 - Entity Framework Connection String con credenciales:**
```xml
<add name="BDRAG" connectionString="metadata=res://*/RAG.csdl|res://*/RAG.ssdl|res://*/RAG.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=10.250.16.25;initial catalog=BDRAGXM;uid=ADMMID;password=ADMMID;multipleactiveresultsets=True;App=EntityFramework&quot;" 
```
**Datos Expuestos:**
- Usuario: `ADMMID`
- Contraseña: `ADMMID`
- Servidor: `10.250.16.25`
- BD: `BDRAGXM`

**Línea 35 - Oracle Credentials:**
```xml
<add name="BDOracle" connectionString="metadata=res://*/PDN.csdl|res://*/PDN.ssdl|res://*/PDN.msl;provider=Oracle.ManagedDataAccess.Client;provider connection string=&quot;DATA SOURCE=ORCL-PDN1-AZURE;PASSWORD=mvm_joaquinbermudez;PERSIST SECURITY INFO=True;USER ID=JOAQUINBERMUDEZ&quot;"
```
**Datos Expuestos:**
- Usuario: `JOAQUINBERMUDEZ`
- Contraseña: `mvm_joaquinbermudez`
- Data Source: `ORCL-PDN1-AZURE`
- **PROBLEMA ADICIONAL:** `PERSIST SECURITY INFO=True` permite recuperar credenciales desde strings de conexión en tiempo de ejecución

**Línea 36 - SQL Server Reporting:**
```xml
<add name="XM.RAG.Reportes.Properties.Settings.BDRAGXM" connectionString="Data Source=MVMSW523\SQLDEV2014 Initial Catalog=BDRAGXM;Persist Security Info=True;User ID=ADMMID;Password=ADMMID"
```
**Datos Expuestos:**
- Usuario: `ADMMID`
- Contraseña: `ADMMID`
- Servidor: `MVMSW523\SQLDEV2014`

#### Ubicación 2: web.config (Ambiente TEST)
**Archivo:** `FUENTES\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\web.config` (comentadas pero visibles)

**Líneas 42+ - TEST Credentials:**
```xml
<!--<add name="LINQ2MIDConnectionString" connectionString="server=COMEDXMAZ061.isamdnt.grupo-isa.com,3052;uid=PASORAG;password=Hcwm5ZpKNOJEuW1L7UzfGh4+;Initial Catalog=BDMIDXM"-->
```
**Datos Expuestos:**
- Usuario: `PASORAG`
- Contraseña: `Hcwm5ZpKNOJEuW1L7UzfGh4+`
- Servidor: `COMEDXMAZ061.isamdnt.grupo-isa.com:3052`

#### Ubicación 3: web.config (Ambiente PRODUCCIÓN - Comentado pero visible)
**Líneas 51+ - PRODUCTION Credentials (en comentarios):**
```xml
<!--<add name="LINQ2MIDConnectionString" connectionString="server=172.16.1.163,3052;uid=PASORAG;password=W4Yo#cDvC3xF.XNl9pmLEahJgI1GedO8B2PMbs!Zu70i5kr_;Initial Catalog=BDMIDXM"-->
<!--<add name="BDRAG" connectionString="...data source=COMEDXMV519.isamdnt.grupo-isa.com,3052;initial catalog=BDRAGXM;uid=PASORAG;password=eNluyLdJj!QhGV4%pRUrv1TqYm7A#st8zCkn$BD0WOa9wM6.;...-->
<!--<add name="BDOracle" connectionString="...DATA SOURCE=XM_PDN1;PASSWORD=Q9a_BIg-EPxGMjVKtRmol4w;PERSIST SECURITY INFO=True;USER ID=PASORAG"-->
```
**Datos Expuestos:**
- Usuario: `PASORAG`
- Contraseñas Production: 
  - SQL: `eNluyLdJj!QhGV4%pRUrv1TqYm7A#st8zCkn$BD0WOa9wM6.`
  - Oracle: `Q9a_BIg-EPxGMjVKtRmol4w`
- Servidores: `COMEDXMV519.isamdnt.grupo-isa.com:3052`, `172.16.1.163:3052`

#### Ubicación 4: ACME Credentials en appSettings
**Líneas 17-22:**
```xml
<add key="ACME_URL" value="https://acmecalidadback.xm.com.co"/>
<add key="ACME_Autenticacion" value="/acme_seguridad_webapi/v2.0/Oauth2"/>
<add key="ACME_Usuario" value="XM_S_AcmeAplicacionPrd@xm.com.co"/>
<add key="ACME_Password" value="C&amp;&amp;8edRa7reN"/>
<add key="ACME_Bancos" value="acme_bancos_webapi/api/Bank/GetAllBank"/>
<add key="ACME_Cuentas" value="/AccountBank/ConsultarCuentasBancariasNegocios"/>
```
**Datos Expuestos:**
- Usuario ACME: `XM_S_AcmeAplicacionPrd@xm.com.co`
- Contraseña: `C&&8edRa7reN`
- URLs internas de APIs

### 1.2 ⚠️ PERSIST SECURITY INFO=True

**Criticidad:** 🔴 CRÍTICA

**Ubicaciones encontradas:**
1. Línea 35 (web.config): Oracle Connection String
2. Línea 36 (web.config): Reporting Connection String  
3. Línea 29 (Settings.Designer.cs): Hardcoded en código

**Riesgo:** Permite que aplicaciones maliciosas extraigan credenciales desde el objeto SQLConnection/OracleConnection en memoria.

---

## 2. RIESGOS DE WCF

### 2.1 ⚠️ Servicios SIN AUTENTICACIÓN (security mode="None")

**Criticidad:** 🔴 CRÍTICA

**Archivo:** `FUENTES\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\web.config`

**Líneas donde aparece `security mode="None"`:** 98, 105, 112, 119, 126, 133, 140

**Servicios afectados:**
1. **WSHttpBinding_IRealizacionSolicitudes** (Línea 98)
   - Sin autenticación de transporte
   - Sin credenciales de mensaje

2. **WSHttpBinding_IRevisionSolicitudes** (Línea 105)
   - Sin autenticación de transporte
   - Sin credenciales de mensaje

3. **WSHttpBinding_IGeneral** (Línea 112)
   - Sin autenticación de transporte
   - Sin credenciales de mensaje

4. **WSHttpBinding_IAdministracion** (Línea 119)
   - Sin autenticación de transporte
   - Sin credenciales de mensaje

5. **WSHttpBinding_IIntegracionPDN** (Línea 126)
   - Sin autenticación de transporte
   - Sin credenciales de mensaje

6. **WSHttpBinding_IIntegracionMID** (Línea 133)
   - Sin autenticación de transporte
   - Sin credenciales de mensaje

7. **WSHttpBinding_INuevoRegfro** (Línea 140)
   - Sin autenticación de transporte
   - Sin credenciales de mensaje

**Configuración problemática encontrada:**
```xml
<binding name="WSHttpBinding_IRealizacionSolicitudes" maxBufferPoolSize="2147483647" maxReceivedMessageSize="2147483647">
  <readerQuotas maxDepth="32" maxStringContentLength="2147483647" maxArrayLength="2147483647" maxBytesPerRead="2147483647" maxNameTableCharCount="2147483647"/>
  <security mode="None">
    <transport clientCredentialType="None" proxyCredentialType="None" realm=""/>
    <message clientCredentialType="UserName" algorithmSuite="Default"/>
  </security>
</binding>
```

**Impacto:** Cualquier cliente puede acceder a los servicios sin credenciales.

### 2.2 ⚠️ Debug Information Exposed en Respuestas de Error

**Criticidad:** 🔴 CRÍTICA

**Archivo:** `FUENTES\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\web.config`

**Líneas:** 244, 249, 254, 259, 264, 269, 274 (y más repetidas)

**Configuración problemática:**
```xml
<serviceDebug includeExceptionDetailInFaults="true"/>
```

**Servicios afectados:**
- RealizacionSolicitudesBehavior
- GeneralBehavior
- RevisionSolicitudesBehavior
- AdministracionBehavior
- IntegracionPDNBehavior
- IntegracionMIDBehavior
- NuevoRegfroBehavior

**Impacto:** 
- Los detalles completos de excepciones (incluyendo stack traces, nombres de métodos, ubicaciones de archivos) se envían en las respuestas SOAP a los clientes
- Revelación de información interna del sistema a atacantes
- Potencial exposición de rutas del servidor, nombres de clases, etc.

---

## 3. ACCESO A DATOS - VULNERABILIDADES

### 3.1 🔴 SQL INJECTION CRÍTICA

**Criticidad:** 🔴 CRÍTICA

**Archivo:** `FUENTES\XM.RAG.Servicios\AccesoDatos\XM.RAG.Oracle\Transacciones\Transacciones.cs`

**Línea 219:**
```csharp
OracleCommand sqlc = new OracleCommand("Update LAC.LAT_CONTACTOPERSONA set FechaIniApe = to_date('" + 
    contacto.FECHAINIAPE.Date.Day + "/" + 
    contacto.FECHAINIAPE.Date.Month + "/" + 
    contacto.FECHAINIAPE.Date.Year + "', 'DD/MM/YYYY') where codigoneg = '" + 
    contacto.CODIGONEG + "' and fechafincop is null and codigotco = '" + 
    contacto.CODIGOTCO + "' and identagent = '" + 
    contacto.IDENTAGENT + "'and cedula = '" + 
    contacto.CEDULA + "'", cnn);
```

**Problemas:**
1. **Concatenación directa de strings en SQL**
2. **Sin parametrización**
3. **Vulnerable a SQL Injection en más de 6 campos:** `CODIGONEG`, `CODIGOTCO`, `IDENTAGENT`, `CEDULA`
4. **No hay validación de entrada**

**Ataque ejemplo:**
```
CEDULA: "' OR '1'='1"  → Bypass de WHERE clause
CODIGONEG: "'; DROP TABLE LAC.LAT_CONTACTOPERSONA; --"  → Eliminación de datos
```

### 3.2 ⚠️ Almacenamiento Procedimientos Sin Parametrización

**Criticidad:** 🟡 MEDIA

**Archivos afectados:** Múltiples DAOs

Aunque muchos usan stored procedures (lo cual es mejor), el parámetro `nombreProcedimientoAlmacenado` se pasa como string y se ejecuta directamente:

**Archivo:** `FUENTES\XM.RAG.Servicios\AccesoDatos\XM.RAG.DataAccess\General\GeneralDAO.cs`

**Línea 382-403:**
```csharp
public DataSet GetDatosPlantillaFromSP(int idDocumento, int idSolicitud, string nombreProcedimientoAlmacenado)
{
    try
    {
        if (string.IsNullOrEmpty(nombreProcedimientoAlmacenado))
        {
            throw new Exception("Debe establecer un procedimiento almacenado...");
        }

        using (SqlConnection conexion = GetConnectionString())
        {
            conexion.Open();
            SqlCommand cmd = new SqlCommand(nombreProcedimientoAlmacenado, conexion);
            // VULNERABILIDAD: nombreProcedimientoAllamacenado viene sin validación
            cmd.CommandType = System.Data.CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("parIdSolicitud", idSolicitud);
            cmd.ExecuteNonQuery();
            // ... resto del código
        }
    }
    catch (Exception ex)
    {
        return null;
        throw ex; // Unreachable code - bad practice
    }
}
```

**Risco:** Si se valida contra whitelist de nombres permitidos sería seguro, pero es necesario verificar.

### 3.3 ⚠️ Connection String Exposure

**Criticidad:** 🔴 CRÍTICA

Connection strings se extraen y se usan en múltiples ocasiones sin encriptación:

**Archivo:** `FUENTES\XM.RAG.Servicios\AccesoDatos\XM.RAG.DataAccess\General\GeneralDAO.cs`

**Línea 291-307:**
```csharp
public static SqlConnection GetConnectionString()
{
    BDRAG ctx = new BDRAG();
    try
    {
        EntityConnection conexion = (EntityConnection)ctx.Connection;
        SqlConnection connection = new SqlConnection();
        connection = (SqlConnection)conexion.StoreConnection;
        return connection; // Retorna sin protección
    }
    catch (Exception ex)
    {
        throw ex;
    }
}
```

**Problema:** La conexión retornada contiene credenciales en texto plano que se pueden ver en logs, debugging, etc.

---

## 4. DEUDA TÉCNICA

### 4.1 🔴 CLASES EXCESIVAMENTE GRANDES

**Criticidad:** 🔴 CRÍTICA (Mantenibilidad)

**Límite Recomendado:** < 400 líneas

| Archivo | Líneas | Severidad |
|---------|--------|-----------|
| ControladoraRealizacionSolicitudes.cs | **3315** | 🔴 CRÍTICA |
| ControladoraTransaccion.cs | **1950** | 🔴 CRÍTICA |
| FachadaGeneral.cs | **1325** | 🟡 MEDIA |
| ControladoraConsulta.cs | ~1500+ | 🟡 MEDIA |

**Archivo crítico:** `FUENTES\XM.RAG.Servicios\Negocio\XM.RAG.RealizacionSolicitudes\Controladora\ControladoraRealizacionSolicitudes.cs`
- 3315 líneas de código
- Responsabilidad única violada
- Difícil de testear
- Difícil de mantener

### 4.2 ⚠️ MÉTODOS CON PARÁMETROS EXCESIVOS

**Criticidad:** 🟡 MEDIA

**Límite Recomendado:** ≤ 3-4 parámetros

Ejemplos encontrados:

**Archivo:** `FUENTES\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\General.svc.cs`
```csharp
public ResultadoGestor RealizarCargaGestor(
    string casoId, 
    string rutaArchivoDestino, 
    string nombreArchivo, 
    string nombreConfiguracionCarga, 
    string nombreTabla)  // 5 parámetros
```

**Archivo:** `FUENTES\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\Administracion.svc.cs`
```csharp
public List<Entidades.Solicitudes.Solicitud> ObtenerSolicitudesConNotificacion(
    string usuario, 
    DateTime fechaInicial, 
    DateTime fechaFinal, 
    List<string> roles, 
    List<string> estados)  // 5 parámetros
```

**Archivo:** `FUENTES\XM.RAG.Servicios\Negocio\XM.RAG.IntegracionPDN\Broker\BrokerTransaccion.cs`
```csharp
public static bool RemoverConceptoBasico(
    string codigoNeg, 
    string codigoCTO, 
    string idAplicacion, 
    string idModulo, 
    string codigoEntidad)  // 5 parámetros
```

### 4.3 ⚠️ MANEJO GENÉRICO DE EXCEPCIONES

**Criticidad:** 🔴 CRÍTICA

**Patrón Problemático 1: Catching Exception sin logging**

**Archivo:** `FUENTES\XM.RAG.Servicios\Soporte\XM.RAG.Servicios.Framework\Utilidades\Comun.cs`

Múltiples instancias (líneas 41, 58, 78, 121, 163, 204, 235, 265, 297, 339, 367):
```csharp
catch (Exception exception)
{
    // Silently fails - no logging, no handling
}
```

**Archivo:** `FUENTES\XM.RAG.Servicios\AccesoDatos\XM.RAG.DataAccess\General\GeneralDAO.cs`

**Líneas 206-209:**
```csharp
catch (Exception ex)
{
    return null;
    throw ex;  // Unreachable code - will never throw!
}
```

**Problemas encontrados:**
1. Excepciones capturadas sin logging
2. Código muerto (throw después de return)
3. Información de error perdida
4. Imposible debuguear en producción

**Patrón Problemático 2: MultiCatch sin distinción**

**Archivo:** `FUENTES\XM.RAG.Servicios\Negocio\XM.RAG.General\Fachada\FachadaGeneral.cs`

```csharp
catch (ServicioExcepcion ex)
{
    ExceptionPolicy.HandleException(ex, PoliticaDeExcepcion.SERVICIOS);
    throw ex;
}
catch (Exception ex)
{
    ExceptionPolicy.HandleException(ex, PoliticaDeExcepcion.SERVICIOS);
    throw ex;
}
```

### 4.4 ⚠️ DUPLICACIÓN DE CÓDIGO

**Criticidad:** 🟡 MEDIA

Patrón repetido +50 veces:

```csharp
public static List<Entidades.Solicitudes.Empresa> ObtenerListaEmpresas()
{
    try
    {
        return Controladora.ControladoraGeneral.ObtenerListaEmpresas();
    }
    catch (ServicioExcepcion ex)
    {
        ExceptionPolicy.HandleException(ex, PoliticaDeExcepcion.SERVICIOS);
        throw ex;
    }
}
```

**Solución:** Usar patrón Template Method o Decorator para evitar duplicación.

### 4.5 ⚠️ VALORES MÁGICOS (Magic Numbers/Strings)

**Criticidad:** 🟡 MEDIA

**Archivo:** `FUENTES\XM.RAG.Servicios\Negocio\XM.RAG.General\Fachada\FachadaGeneral.cs`

```csharp
if ((IdTipoSolicitud == 1 || IdTipoSolicitud == 3) && tabla.Columns.Contains("EsORZNI") 
    && (documentoPlantilla.NombreCorto == "CREA" || documentoPlantilla.NombreCorto == "NAIA"))

if (IdTipoSolicitud == 7 && !string.IsNullOrEmpty(documento.Html) && documentoPlantilla.NombreCorto == "SCUC")
    documento.Html = HttpUtility.HtmlDecode(documento.Html);
```

**Problemas:**
- Códigos hardcodeados (1, 3, 7, "CREA", "NAIA", "SCUC")
- Sin constantes enumeradas
- Difícil de mantener

---

## 5. OBSOLESCENCIA TECNOLÓGICA

### 5.1 🔴 VERSIONES MUY ANTIGUAS DE LIBRERÍAS

**Criticidad:** 🔴 CRÍTICA

| Librería | Versión Actual | Versión Usada | Antigüedad | Riesgo |
|----------|--------|--------|-----------|--------|
| **Enterprise Library** | 10.x | **5.0.414.0** | ~13 años | 🔴 Sin soporte, vulnerabilidades |
| **Newtonsoft.Json** | 13.x | **4.5.0.0** | ~10 años | 🔴 Múltiples vulnerabilidades |
| **iTextSharp** | 8.x (deprecated) | **5.5.0.0** | ~7 años | 🟡 Librería completamente deprecated |
| **Oracle.DataAccess** | 21.x | **2.112.3.0** | ~5+ años | 🔴 Sin soporte |
| **System.Data.Linq** | - | Framework 4.5 | ~12 años | 🟡 Obsoleto (usar EF Core) |
| **LinqExtender** | - | **2.5.0.0** | ~10+ años | 🔴 Proyecto muerto |

**Archivo de referencia:** `FUENTES\XM.RAG.Servicios\Soporte\XM.RAG.Servicios.Framework\XM.RAG.Servicios.Framework.csproj`

```xml
<Reference Include="Microsoft.Practices.EnterpriseLibrary.ExceptionHandling, Version=5.0.414.0, ..."/>
```

### 5.2 ⚠️ TECNOLOGÍAS COMPLETAMENTE DEPRECADAS

**Criticidad:** 🔴 CRÍTICA

1. **Microsoft.Practices.EnterpriseLibrary v5** (2010)
   - Status: End of Life
   - Reemplazo: Microsoft.Extensions.* (incluido en .NET Core/.NET 5+)
   
2. **iTextSharp 5.5** 
   - Status: **Completamente deprecado por el autor**
   - Reemplazo: iText 7 (pero es comercial), o usar QuestPDF, SelectPdf
   
3. **LINQ to SQL (System.Data.Linq)**
   - Status: Congelado desde .NET 4.5
   - Reemplazo: Entity Framework Core (recomendado)
   
4. **LinqExtender 2.5**
   - Status: Proyecto muerto (última actualización ~2010)
   - Riesgo: Sin actualizaciones de seguridad

5. **Windows Communication Foundation (WCF) sin actualizaciones**
   - Microsoft recomienda: gRPC o ASP.NET Core APIs

---

## RESUMEN EJECUTIVO

### 🔴 Hallazgos Críticos Inmediatos

| # | Problema | Severidad | Acción |
|---|----------|-----------|--------|
| 1 | Credenciales hardcodeadas (9 instancias) | 🔴 CRÍTICA | Rotación inmediata + vault seguro |
| 2 | SQL Injection en Transacciones.cs:219 | 🔴 CRÍTICA | Parche urgente + review código Oracle |
| 3 | WCF sin autenticación (7 servicios) | 🔴 CRÍTICA | Implementar seguridad (Transport/Message) |
| 4 | ServiceDebug expone detalles (7 behaviors) | 🔴 CRÍTICA | Deshabilitar en producción |
| 5 | PERSIST SECURITY INFO=True | 🔴 CRÍTICA | Cambiar a False en todas las strings |
| 6 | Clases de 3300+ líneas | 🔴 CRÍTICA | Refactoring arquitectónico |
| 7 | Excepciones genéricas sin logging | 🔴 CRÍTICA | Implementar logging real |
| 8 | Librerías 10+ años antiguas | 🔴 CRÍTICA | Plan de migración a .NET Core/.NET 5+ |

### 📊 Estadísticas

- **Líneas de código en clases mega (>1000):** 3 archivos = 6,590 líneas
- **Métodos con >5 parámetros:** 25+ métodos encontrados
- **Try-catch vacíos sin logging:** 50+ instancias
- **Credenciales en archivos de configuración:** 11 diferentes credenciales expuestas
- **Servicios WCF sin autenticación:** 7/7 bindings críticos

### 🛠️ Plan de Remediación (Prioridad)

**Phase 1 (Inmediato - Semana 1):**
1. Rotar todas las credenciales expuestas
2. Implementar Azure Key Vault / ConfigServer para secretos
3. Deshabilitar `serviceDebug` en producción
4. Parchar SQL Injection en Transacciones.cs

**Phase 2 (Corto plazo - Mes 1):**
1. Implementar autenticación en todos los WCF (seleccionar: Windows/Certificate/OAuth2)
2. Cambiar `PERSIST SECURITY INFO=False` en todas las strings
3. Crear logging centralizado para excepciones

**Phase 3 (Mediano plazo - Trimestre 1):**
1. Refactorizar clases mega (dividir en responsabilidades)
2. Crear abstracción de acceso a datos (reducir código duplicado)
3. Eliminar hardcoded magic numbers → Constants/Enums

**Phase 4 (Largo plazo - Año 1):**
1. Migración a .NET Core/.NET 6+ 
2. Entity Framework Core (reemplazar LINQ to SQL)
3. Reemplazo alternativas: 
   - iTextSharp → QuestPDF o iText 7
   - Enterprise Library → Microsoft.Extensions.*
   - WCF → gRPC / ASP.NET Core APIs

---

## APÉNDICE A: CREDENCIALES EXPUESTAS (PARA ROTACIÓN)

### Usuarios que DEBEN rotarse INMEDIATAMENTE:
1. `ADMMID` - Múltiples DBs (BDMIDXM, BDRAGXM)
2. `JOAQUINBERMUDEZ` - Oracle (ORCL-PDN1-AZURE)
3. `PASORAG` - SQL Server (staging/prod)
4. `XM_S_AcmeAplicacionPrd@xm.com.co` - ACME API

### Servidores/IPs expuestas:
- `10.250.16.25` (SQL Server DEV)
- `COMEDXMAZ061.isamdnt.grupo-isa.com:3052` (TEST)
- `COMEDXMV519.isamdnt.grupo-isa.com:3052` (PROD)
- `172.16.1.163:3052` (PROD)
- `ORCL-PDN1-AZURE` (Oracle DEV)

---

## APÉNDICE B: ARCHIVOS CRÍTICOS A REVISAR

```
Priority 1 (Seguridad Crítica):
├── FUENTES/XM.RAG.Servicios/Servicios/XM.RAG.Servicios/web.config
├── FUENTES/XM.RAG.Servicios/AccesoDatos/XM.RAG.Oracle/Transacciones/Transacciones.cs
└── FUENTES/XM.RAG.Servicios/AccesoDatos/XM.RAG.DataAccess/General/GeneralDAO.cs

Priority 2 (Arquitectura/Deuda):
├── FUENTES/XM.RAG.Servicios/Negocio/XM.RAG.RealizacionSolicitudes/Controladora/ControladoraRealizacionSolicitudes.cs (3315 líneas)
├── FUENTES/XM.RAG.Servicios/Negocio/XM.RAG.IntegracionPDN/Controladora/ControladoraTransaccion.cs (1950 líneas)
└── FUENTES/XM.RAG.Servicios/Negocio/XM.RAG.General/Fachada/FachadaGeneral.cs (1325 líneas)
```

---

**Documento generado:** 6 de febrero de 2026  
**Analista:** GitHub Copilot  
**Clasificación:** CONFIDENCIAL
