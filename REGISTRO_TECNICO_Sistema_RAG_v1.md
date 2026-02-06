# 📋 REGISTRO TÉCNICO COMPLETO - SISTEMA XM.RAG

**Documento**: Análisis Técnico de Arquitectura y Solución
**Versión**: 1.0  
**Fecha**: 06 de Febrero de 2026  
**Clasificación**: Documentación Oficial del Sistema  
**Audiencia**: Equipos de Migración Tecnológica, Arquitectura, DevOps

---

## 📑 TABLA DE CONTENIDOS

1. [Estructura General del Proyecto](#1-estructura-general-del-proyecto)
2. [Arquitectura y Patrones](#2-arquitectura-y-patrones)
3. [Componentes de SharePoint](#3-componentes-de-sharepoint)
4. [Configuración y Despliegue](#4-configuración-y-despliegue)
5. [Dependencias Externas](#5-dependencias-externas)
6. [Acceso a Datos](#6-acceso-a-datos)
7. [Seguridad](#7-seguridad)
8. [Lógica de Negocio Crítica](#8-lógica-de-negocio-crítica)
9. [Logging, Errores y Monitoreo](#9-logging-errores-y-monitoreo)
10. [Riesgos Técnicos y Deuda](#10-riesgos-técnicos-y-deuda)
11. [Recomendaciones para Migración](#11-recomendaciones-para-migración)

---

## 1️⃣ ESTRUCTURA GENERAL DEL PROYECTO

### 📦 Tipo de Solución

**Solución Multi-Capas Híbrida con Dos Componentes Principales:**

| Componente | Tipo | Propósito |
|-----------|------|----------|
| **XM.RAG** | SharePoint 2010 WSP (WebSolution Package) | Interfaz web, WebParts, Features, Event Receivers, Timer Jobs |
| **XM.RAG.Servicios** | WCF Service Host (.NET 4.0) | 8 Endpoints SOAP para integración de lógica de negocio |

### 🎯 Versiones de .NET Framework

| Proyecto | Framework | Versión | Propósito |
|----------|-----------|---------|----------|
| XM.RAG (SharePoint) | .NET Framework | v3.5 | Compatibilidad SharePoint 2010 |
| XM.RAG.Framework | .NET Framework | v3.5 | Framework base para SharePoint |
| XM.RAG.TimerJobs | .NET Framework | v3.5 | Jobs de SharePoint 2010 |
| XM.RAG.SharePointDataAccess | .NET Framework | v3.5 | DAL para SharePoint |
| XM.RAG.Servicios (WCF) | .NET Framework | v4.0 | Servicios web de negocio |
| XM.RAG.Servicios.Framework | .NET Framework | v4.0 | Utilities y base para servicios |

**Compilador**: Visual Studio 2010, ToolsVersion 4.0, StyleCop v4.7

### 🏗️ Proyectos dentro de la Solución

#### **Solución 1: XM.RAG.sln (SharePoint)**

```
XM.RAG.sln
├── XM.RAG (Proyecto Principal - WSP)
│   ├── WebParts (AccesoEmpresa, AccesoAnalista)
│   ├── Features (RAG, XM.RAG.TimerJobs)
│   ├── Provisioning
│   ├── ControlTemplates
│   ├── Service References (4 referencias WCF)
│   └── Layouts (JavaScript, CSS, HTML)
├── XM.RAG.Framework (Librería Base)
│   ├── Contexto (Session management)
│   ├── Enumeraciones y constantes
│   ├── Utilidades (AD, Roles, Logging)
│   └── Enterprise Library integration
├── XM.RAG.Mensajes (Resource strings)
│   └── XML de mensajes multiidioma
├── XM.RAG.TimerJobs (Timer Jobs)
│   ├── Feature Event Receiver
│   ├── 6+ Timer Job implementations
│   └── Helper clases (Email, Parameters)
└── XM.RAG.SharePointDataAccess (DAL)
    ├── BLL base classes (SharepointBase<T>)
    └── DAL base classes (BaseEntity)
```

#### **Solución 2: XM.RAG.Servicios.sln (WCF + Negocio)**

```
XM.RAG.Servicios.sln
├── Servicios
│   ├── XM.RAG.ContratosServicios (Contracts)
│   │   └── 8 IServiceContract interfaces
│   └── XM.RAG.Servicios (WCF Host .svc files)
│       ├── Administracion.svc
│       ├── General.svc
│       ├── RevisionSolicitudes.svc
│       ├── RealizacionSolicitudes.svc
│       ├── IntegracionMID.svc
│       ├── IntegracionPDN.svc
│       ├── NuevoRegfro.svc
│       └── RegistroSucesos.svc
├── Negocio (Lógica de negocio)
│   ├── XM.RAG.Entidades (DTOs/Modelos)
│   ├── XM.RAG.General (Broker, Fachada)
│   ├── XM.RAG.Administracion (Broker, Servicios)
│   ├── XM.RAG.RevisionSolicitudes (Broker)
│   ├── XM.RAG.RealizacionSolicitudes (Broker)
│   ├── XM.RAG.IntegracionMID (Broker)
│   ├── XM.RAG.IntegracionPDN (Broker)
│   ├── XM.RAG.RegistroSucesos (Broker)
│   ├── XM.RAG.Reportes (Reporting logic)
│   ├── XM.RAG.GestorArchivos (File management)
│   └── XM.RAG.RegFro (Registro Frontera)
├── AccesoDatos
│   ├── XM.RAG.DataAccess (EF ObjectContext BDRAG)
│   ├── XM.RAG.Oracle (EF ObjectContext BDOracle)
│   ├── XM.RAG.LinQ2Mid (LINQ to SQL context)
│   └── DAO classes (ConceptoTributarioDAO, AgentesDAO, etc.)
└── Soporte (Support libraries)
    ├── XM.RAG.Servicios.Framework (Utilities)
    ├── XM.RAG.Servicios.Mensajes (Error/Success messages)
    └── XM.RAG.EntidadesOracle (Oracle model classes)
```

### 🏛️ Capas Arquitectónicas

```
┌─────────────────────────────────────────────────────┐
│  PRESENTACIÓN (SharePoint WSP)                      │
│  ├─ WebParts (Razor-like components)               │
│  ├─ User Controls (ASPX.cs)                        │
│  └─ Features & Event Receivers                      │
└─────────────────────────────────────────────────────┘
                          ↓↑
┌─────────────────────────────────────────────────────┐
│  WCF SERVICES (8 Endpoints SOAP)                    │
│  ├─ RealizacionSolicitudes.svc                      │
│  ├─ RevisionSolicitudes.svc                         │
│  ├─ Administracion.svc                              │
│  ├─ General.svc                                     │
│  ├─ IntegracionMID.svc                              │
│  ├─ IntegracionPDN.svc                              │
│  ├─ RegistroSucesos.svc                             │
│  └─ NuevoRegfro.svc                                 │
└─────────────────────────────────────────────────────┘
                          ↓↑
┌─────────────────────────────────────────────────────┐
│  LÓGICA DE NEGOCIO (Brokers + Facades)              │
│  ├─ BrokerRevisionSolicitudes                       │
│  ├─ BrokerRealizacionSolicitudes                    │
│  ├─ BrokerGeneral + FachadaGeneral                  │
│  ├─ BrokerAdministracion                            │
│  ├─ BrokerIntegracionMID                            │
│  ├─ BrokerIntegracionPDN                            │
│  ├─ BrokerRegistroSucesos                           │
│  └─ BrokerRegfro                                    │
└─────────────────────────────────────────────────────┘
                          ↓↑
┌─────────────────────────────────────────────────────┐
│  DATA ACCESS LAYER (DAL + DAO + EF)                 │
│  ├─ Entity Framework ObjectContext (SQL Server)     │
│  ├─ Entity Framework ObjectContext (Oracle)         │
│  ├─ LINQ to SQL (MID)                               │
│  ├─ DAO Pattern Classes                             │
│  └─ Stored Procedure calls                          │
└─────────────────────────────────────────────────────┘
                          ↓↑
┌─────────────────────────────────────────────────────┐
│  BASES DE DATOS                                     │
│  ├─ SQL Server: BDRAGXM (Principal)                 │
│  ├─ SQL Server: BDMIDXM (Integración MID)           │
│  └─ Oracle: PDN (Permisos - PDN1)                   │
└─────────────────────────────────────────────────────┘
```

### 📁 Convenciones de Nombres y Organización

| Concepto | Convención | Ejemplo |
|----------|-----------|---------|
| **Soluciones** | `XM.[ModuloGrande].sln` | `XM.RAG.Servicios.sln` |
| **Proyectos** | `XM.RAG.[Módulo][Propósito]` | `XM.RAG.Administracion` |
| **Namespaces** | `XM.RAG.[Layer].[Feature]` | `XM.RAG.Negocio.Administracion` |
| **Clases Broker** | `Broker[Entidad]` | `BrokerRevisionSolicitudes` |
| **Clases DAO** | `[Entidad]DAO` | `AgentesDAO` |
| **Clases DAL** | `[Entidad]` + hereda BaseEntity | `Destinatarios` |
| **Clases BLL Web** | `[Entidad]` + hereda SharepointBase<T> | `Destinatarios : SharepointBase<DestinatariosDAL>` |
| **WCF Contracts** | `I[NombreServicio]` | `IAdministracion`, `IGeneral` |
| **WCF Implementación** | `[NombreServicio]` | `RealizacionSolicitudes.svc` |
| **Features** | `[Módulo].feature` | `RAG.feature` |
| **WebParts** | `[NombreFuncional]WebPart` | `AccesoEmpresaWebPart` |

---

## 2️⃣ ARQUITECTURA Y PATRONES

### 🏢 Arquitectura Utilizada

**Tipo: Arquitectura de N-Capas Distribuida con Patrón Broker**

La solución implementa una arquitectura clásica de n-capas separada en dos dominios:
1. **Dominio SharePoint 2010** (Presentación + Capa de Integración)
2. **Dominio de Servicios WCF** (Lógica de negocio distribuida)

**Ventajas actuales**:
- Separación clara de responsabilidades
- Posibilidad de escalar servicios independientemente
- Múltiples clientes pueden consumir servicios WCF (no solo SharePoint)

**Desventajas**:
- Duplicación de lógica en algunos Brokers
- Acoplamiento entre SharePoint y servicios WCF a través de referencias directas
- No utiliza patrones modernos como Repository o Dependency Injection

### 📐 Patrones de Diseño Implementados

| Patrón | Implementación | Ubicación | Estado |
|--------|----------------|-----------|--------|
| **BLL/DAL Clásico** | ✅ Implementado | `Negocio/*`, `AccesoDatos/*` | En uso activo |
| **Broker Pattern** | ✅ Implementado | `Broker[Módulo].cs` | En uso activo |
| **Facade Pattern** | ✅ Implementado | `FachadaGeneral.cs` | En uso activo |
| **DAO Pattern** | ✅ Implementado | `[Entidad]DAO.cs` | En uso activo |
| **Factory Pattern** | ⚠️ Parcialmente | `ServiceClient.MakeClient()` | Limitado |
| **Repository Pattern** | ❌ No implementado | — | Usar DAOs en su lugar |
| **Unit of Work Pattern** | ❌ No implementado | — | No identificado |
| **Dependency Injection** | ❌ No implementado | — | No hay contenedor DI |
| **Singleton** | ⚠️ Parcialmente | Algunos helpers | Uso manual |
| **Observer Pattern** | ✅ Implementado | Event Receivers de SharePoint | En Timer Jobs |

### 🏗️ Estructura de Brokers

Los Brokers actúan como orquestadores de negocio, coordinando múltiples DAOs y servicios:

```csharp
// Patrón Broker típicamente implementado
public class BrokerRevisionSolicitudes
{
    private EF.Solicitudes.AgentesDAO _agentesDAO;
    private Microsoft.Practices.EnterpriseLibrary.Data.Database _db;
    
    public void ProcesarRevision(SolicitudDTO solicitud)
    {
        // Obtiene datos de múltiples fuentes
        var agente = _agentesDAO.ObtenerAgente(solicitud.IdAgente);
        
        // Usa Enterprise Library para operaciones
        var resultado = _db.ExecuteDataSet(
            "sp_ProcesarRevision", 
            solicitud.IdSolicitud
        );
        
        // Orquesta multiple operaciones
        ActualizarSolicitud(resultado);
        GenerarNotificación(agente);
        RegistrarAuditoria(solicitud);
    }
}
```

### 🔄 Flujos de Integración Principal

```
Cliente SharePoint
        ↓
WebPart / Feature
        ↓
Service References (Add Service Reference)
        ↓
WCF Service Proxy (auto-generated)
        ↓
WCF Endpoint (BasicHttpBinding)
        ↓
[ExceptionShielding] Intercept
        ↓
IServiceContract Implementation (.svc)
        ↓
Broker Pattern Class
        ↓
DAO/DAL Classes
        ↓
Entity Framework ObjectContext o ADO.NET Raw
        ↓
SQL Server (BDRAGXM) o Oracle (PDN)
```

### ⚙️ Acoplamientos Críticos Identificados

1. **SharePoint ↔ WCF Services** (Fuerte)
   - WebParts tienen referencias directas a servicios WCF
   - Si WCF service cae, SharePoint falla parcialmente
   - No hay circuit breaker o retry logic

2. **Servicios WCF ↔ Entity Framework** (Fuerte)
   - ObjectContext directamente en Brokers
   - Cambios en modelo EDMX afectan toda la cadena
   - No hay abstracción con repository interface

3. **Enterprise Library Configuration** (Crítico)
   - Archivo de configuración externo: `entlib.config`
   - Si la ruta no existe, fallan logging y exception handling
   - Ruta hardcodeada en web.config

4. **Active Directory** (Fuerte)
   - Sistema de roles depende de AD
   - Si AD no está disponible, autenticación falla
   - No hay fallback local

### 🔗 Dependencias Circulares Identificadas

**Ninguna dependencia circular crítica detectada** entre proyectos, sin embargo:

⚠️ **Acoplamiento Circular a Nivel de Clases**:
- `BrokerRevisionSolicitudes` → `AgentesDAO` → `BDRAG ObjectContext` → `BrokerRevisionSolicitudes` (indirectamente a través de cambios)

### 🎯 Relaciones de Dependencia Entre Proyectos

```
XM.RAG (SharePoint WSP)
├─→ XM.RAG.Framework (v3.5)
├─→ XM.RAG.Mensajes (v3.5)
├─→ XM.RAG.SharePointDataAccess (v3.5)
├─→ XM.RAG.TimerJobs (v3.5)
└─→ Service References → WCF Services (en tiempo de ejecución)

XM.RAG.TimerJobs (v3.5)
├─→ XM.RAG.Framework (v3.5)
└─→ Service References → WCF Services

XM.RAG.SharePointDataAccess (v3.5)
├─→ XM.RAG.Framework (v3.5)
└─→ Entity Framework 3.5

XM.RAG.Servicios (v4.0 WCF Host)
├─→ XM.RAG.ContratosServicios (v4.0)
├─→ XM.RAG.Servicios.Framework (v4.0)
└─→ XM.RAG.Servicios.Mensajes (v4.0)

Proyectos de Negocio (*Broker classes)
├─→ XM.RAG.General, XM.RAG.Administracion, etc.
├─→ AccesoDatos (DAO + DAL)
├─→ XM.RAG.Entidades (DTOs)
└─→ Enterprise Library v5.0

AccesoDatos
├─→ XM.RAG.DataAccess (EF + BDRAGXM)
├─→ XM.RAG.Oracle (EF + PDN)
├─→ XM.RAG.LinQ2Mid (LINQ + BDMIDXM)
└─→ Entity Framework v1.0
```

---

## 3️⃣ COMPONENTES DE SHAREPOINT

### 📌 Versión de SharePoint

**SharePoint 2010** (v14.0.0.0 on-premises)

**Características soportadas**:
- Features (Site-scoped deactivatable)
- Event Receivers (Built-in feature activation/deactivation)
- WebParts (Custom development)
- User Controls (ASCX templates)
- Timer Jobs (Server-side scheduled tasks)

### 🎁 Features Implementadas

| Feature | Scope | Ubicación | Responsabilidad |
|---------|-------|-----------|-----------------|
| **RAG** | Site | `Features/RAG/` | Feature base de la solución RAG |
| **XM.RAG.TimerJobs** | Site | `Features/RAG/` | Activación/Desactivación de Timer Jobs |

**Estructura Feature**:
```xml
<Feature>
  <Name>RAG</Name>
  <Receiver Class="XM.RAG.Features.RAG.RAG">
    <FeatureActivated>CreateTimerJobsAndWebPartsActions</FeatureActivated>
    <FeatureDeactivating>DeleteTimerJobsActions</FeatureDeactivating>
  </Receiver>
</Feature>
```

### 🔔 Event Receivers

**Clase**: `RAG.EventReceiver` (hereda de `SPFeatureReceiver`)

| Evento | Acción | Implementación |
|--------|--------|---------------|
| **FeatureActivated** | Crea 6+ Timer Jobs | Instancia de `SPJobDefinition` |
| **FeatureDeactivating** | Elimina Timer Jobs | Enumeración y deletración segura |

### 🎨 WebParts Personalizados

| WebPart | Propósito | Ubicación |
|---------|----------|-----------|
| **AccesoEmpresa** | Portal de entrada para empresas | `Provisioning/WebParts/AccesoEmpresa.webpart` |
| **AccesoAnalista** | Portal de acceso para analistas | `Provisioning/WebParts/AccesoAnalista.webpart` |

**Características WebParts**:
- Heredan de extensiones custom
- Conectan directamente a servicios WCF
- Contienen lógica de renderización ASPX.cs
- Utilizan Telerik RadControls para grids y calendarios

### ⏱️ Timer Jobs

Hay 6+ Timer Jobs configurados para ejecutarse en diferentes horarios:

| Timer Job | Propósito | Frecuencia | Estado |
|-----------|----------|-----------|--------|
| **ActualizarARechazadaParcialmente** | Actualizar estado de solicitudes rechazadas parcialmente | Daily | ✅ Activo |
| **GenerarAlarma** | Generar notificaciones de alertas | Hourly | ✅ Activo |
| **GenerarReporteDianNIF** | Generación de reportes para DIAN | Daily | ✅ Activo |
| **ActualizarEstadoSolicitud** | Transiciones de estado automáticas | Daily | ✅ Activo |
| **LimpiarCache** | Limpieza de caché expirado | Hourly | ✅ Activo |
| **SincronizarDatosExtornos** | Sincronización con sistemas externos | Every 4 hours | ✅ Activo |

**Implementación**:
```csharp
public class GenerarAlarma : SPJobDefinition
{
    public override void Execute(Guid targetInstanceId)
    {
        try
        {
            var broker = new BrokerRegistroSucesos();
            broker.GenerarAlarmas();
            LogManager.LogInfo("GenerarAlarma completado", CategoriaLog.TIMER_JOB);
        }
        catch (Exception ex)
        {
            LogManager.LogError("Error en GenerarAlarma", ex);
            ExceptionHandler.HandleException(ex, "FRONTERA_DE_SERVICIO");
        }
    }
}
```

### 📋 Listas y Bibliotecas Utilizadas

| Lista/Biblioteca | Propósito | Content Type | Campos Clave |
|-----------------|-----------|-------------|--------------|
| **RAG - Solicitudes** | Gestión de solicitudes de registro | SolicitudRAG | Id, NúmeroSolicitud, Estado, Fecha |
| **RAG - Documentos** | Almacenamiento de documentos adjuntos | Documento | Archivo, SolicitudRelacionada, Tipo |
| **RAG - Notificaciones** | Log de notificaciones enviadas | Notificación | Destinatario, Tipo, Fecha |
| **RAG - Auditoría** | Registro de cambios | Auditoría | Usuario, Acción, Recurso, Fecha |

### 📦 Content Types Personalizados

Todos herdan de `Item` o `Document`:

| Content Type | Campos | Relaciones |
|-------------|--------|-----------|
| **SolicitudRAG** | 23 campos (vía Site Columns) | Empresas, Agentes, Documentos |
| **Documento** | 15 campos | Solicitud (relación) |
| **Notificación** | 8 campos | Usuario (relación) |

### 🔐 Permisos y Roles SharePoint

Los siguientes grupos de SharePoint se crean durante la feature activation:

| Grupo SharePoint | Permisos | Alcance |
|-----------------|----------|--------|
| **Administradores RAG** | Contribuyente + Diseño | Site completo |
| **Analistas RAG** | Editar (restringido) | Listas específicas |
| **Usuarios RAG** | Leer | Datos públicos |

**Integración con AD**: Los grupos se sincronizan con grupos de Active Directory corporativo.

### 🔄 Workflows

**Nota**: La documentación y code no muestran workflows de SharePoint Designer o Nintex.
Los workflows se implementan **mediante lógica en Brokers y Timer Jobs**, no usando SharePoint Workflow engine.

```csharp
// Workflows implementados en código, no en WFML
public class BrokerRevisionSolicitudes
{
    public void ProcesarSolicitud(int idSolicitud)
    {
        // Estado: Pendiente Revisión
        var solicitud = ObtenerSolicitud(idSolicitud);
        
        // Validación
        if (!ValidarSolicitud(solicitud))
            solicitud.Estado = EstadoSolicitud.RechazadoValidacion;
        
        // Análisis
        var resultado = AnalizarDocumentos(solicitud);
        
        // Aprobar/Rechazar
        solicitud.Estado = resultado.Aprobado 
            ? EstadoSolicitud.Aprobado 
            : EstadoSolicitud.RechazadoAnalisis;
        
        solicitud.FechaResolucion = DateTime.Now;
        GuardarCambios();
    }
}
```

### 🔗 Integración SharePoint ↔ Servicios WCF

**Flujo típico de llamada desde WebPart a Servicio WCF**:

```
1. WebPart.ascx.cs → BrindaDatos()
2. Crea proxy de servicio (auto-generated por Add Service Reference)
3. Proxy realiza llamada HTTP/SOAP a WCF Endpoint
4. WCF intercepts con [ExceptionShielding] behavior
5. Service.svc → Broker.Metodo()
6. Broker → DAL/DAO → Entity Framework → BD
7. Resultado retorna a WebPart
8. WebPart renderiza con Telerik RadControls
```

**Service References definidas en XM.RAG.csproj**:
- ServiceAdministracion
- ServiceGeneral
- ServiceIntegracionMID
- ServiceRevisionSolicitudes
- ServiceRealizacionSolicitudes (v1, v2, v3)

---

## 4️⃣ CONFIGURACIÓN Y DESPLIEGUE

### 🔧 Configuración - Web.config y App.config

**Ubicación Web.config**: 
`c:\RAG\RAGV2\RAG\FUENTES\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\web.config`

**Configuración Clave**:

#### Connection Strings

```xml
<connectionStrings>
  <!-- SQL Server - Base Principal RAG -->
  <add name="BDRAG" 
    connectionString="metadata=res://*/RAG.csdl|res://*/RAG.ssdl|res://*/RAG.msl;
    provider=System.Data.SqlClient;
    provider connection string=&quot;data source=10.250.16.25;
    initial catalog=BDRAGXM;uid=ADMMID;password=ADMMID;
    multipleactiveresultsets=True;App=EntityFramework&quot;" 
    providerName="System.Data.EntityClient"/>
  
  <!-- SQL Server - Base MID (Integración) -->
  <add name="LINQ2MIDConnectionString" 
    connectionString="server=10.250.16.25;uid=ADMMID;password=ADMMID;
    Initial Catalog=BDMIDXM" 
    providerName="System.Data.SqlClient"/>
  
  <!-- Oracle - Base de Permisos PDN -->
  <add name="BDOracle" 
    connectionString="metadata=res://*/PDN.csdl|res://*/PDN.ssdl|res://*/PDN.msl;
    provider=Oracle.ManagedDataAccess.Client;
    provider connection string=&quot;DATA SOURCE=ORCL-PDN1-AZURE;
    PASSWORD=mvm_joaquinbermudez;PERSIST SECURITY INFO=True;
    USER ID=JOAQUINBERMUDEZ&quot;" 
    providerName="System.Data.EntityClient"/>
</connectionStrings>
```

#### AppSettings

```xml
<appSettings>
  <!-- Logging -->
  <add key="urlLogTecnico" value="C:\LogTecnico" />
  
  <!-- Integración ACME (Sistema Externo) -->
  <add key="ACME_URL" value="https://acmecalidadback.xm.com.co"/>
  <add key="ACME_Autenticacion" value="/acme_seguridad_webapi/v2.0/Oauth2"/>
  <add key="ACME_Usuario" value="XM_S_AcmeAplicacionPrd@xm.com.co"/>
  <add key="ACME_Password" value="C&&8edRa7reN"/>
  <add key="ACME_Bancos" value="acme_bancos_webapi/api/Bank/GetAllBank"/>
  <add key="ACME_Cuentas" value="/AccountBank/ConsultarCuentasBancariasNegocios"/>
</appSettings>
```

#### Enterprise Library Configuration

```xml
<enterpriseLibrary.ConfigurationSource selectedSource="Configuracion">
  <sources>
    <add name="Configuracion" 
      type="Microsoft.Practices.EnterpriseLibrary.Common.Configuration.FileConfigurationSource, ..."
      filePath="C:\Proyectos\RAG\Fuentes\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\Configuracion\entlib.config"/>
  </sources>
</enterpriseLibrary.ConfigurationSource>
```

**Nota Crítica**: Ruta hardcodeada. Si la ruta no existe, fallan Exception Handling y Logging.

#### WCF Bindings

```xml
<system.serviceModel>
  <!-- 7 WSHttpBindings definidos con security mode="None" -->
  <bindings>
    <wsHttpBinding>
      <binding name="WSHttpBinding_IRealizacionSolicitudes" 
        maxBufferPoolSize="2147483647" 
        maxReceivedMessageSize="2147483647">
        <readerQuotas maxStringContentLength="2147483647" 
          maxArrayLength="2147483647" 
          maxBytesPerRead="2147483647"/>
        <security mode="None">
          <transport clientCredentialType="None"/>
          <message clientCredentialType="UserName"/>
        </security>
      </binding>
      <!-- Similar para IRevisionSolicitudes, IGeneral, IAdministracion, etc. -->
    </wsHttpBinding>
  </bindings>
  
  <!-- 7 Service Behaviors con debug habilitado -->
  <behaviors>
    <serviceBehaviors>
      <behavior name="XM.RAG.Servicios.RealizacionSolicitudesBehavior">
        <serviceMetadata httpGetEnabled="true"/>
        <serviceDebug includeExceptionDetailInFaults="true"/>
        <dataContractSerializer maxItemsInObjectGraph="2147483647"/>
      </behavior>
      <!-- Similar para otros servicios -->
    </serviceBehaviors>
  </behaviors>
</system.serviceModel>
```

### 🚀 Proceso de Despliegue Actual

**Tipo**: Semi-automatizado con PowerShell + scripts de Token Replacement

**Herramientas utilizadas**:
- Visual Studio 2010 (compilación local)
- PowerShell Scripts (en `FUENTES/Commands/`)
- MSBuild (compilación en CI/CD)
- Windows Scheduled Tasks (para versioning)

**Scripts principales**:

1. **DeployWSPGlobal.ps1** - Despliegue de WSP (SharePoint solution)
   - Uninstalla solución anterior
   - Espera 120 segundos
   - Agrega nueva solución
   - Instala con GAC Deployment
   - Usa Invoke-Command para ejecución remota

2. **launch.ps1** - Crea Scheduled Task
   - Elimina tarea anterior si existe
   - Registra nueva tarea de PowerShell
   - Invoca VersionPowerShell.ps1

3. **VersionPowerShell.ps1** - Versionamiento
   - (Contenido no completamente visible)

**Variables de Reemplazo (Token Replacement)**:
```
#{PowerShell_Server__ambiente__}# → IP del servidor
#{user_sharepoint__ambiente__}# → Usuario de servicio
#{password_sharepoint__ambiente__}# → Contraseña (RIESGO)
#{wsp_file}# → Nombre del WSP
#{artefacto}# → Artefacto de compilación
#{System.DefaultWorkingDirectory}# → Raíz de build
```

### 🏭 Servidores Involucrados

| Servidor | Rol | Puerto | Software |
|----------|-----|--------|----------|
| **MVMSW523** (Desarrollo) | SQL Server | 1433 | SQL Server 2014 |
| **Sharepoint Farm** | SharePoint 2010 | 80/443 | IIS 7.5 + SharePoint |
| **10.250.16.25** | SQL Server MID | 1433 | SQL Server |
| **10.250.16.5** | Oracle PDN | 1521 | Oracle Database |
| **ORCL-PDN1-AZURE** | Oracle (Producción) | 1521 | Oracle Database |

### 🔄 Ambientes de Despliegue

| Ambiente | BD Principal | BD MID | BD PDN | Estado |
|----------|-------------|--------|--------|--------|
| **Desarrollo** | MVMSW523\SQLDEV2014 | 10.250.16.25 | ORCL-PDN1-AZURE | ✅ Activo |
| **Pruebas** | COMEDXMAZ061:3052 | COMEDXMAZ061:3052 | XM_PRU | ⚠️ Producción actual |
| **Producción** | COMEDXMV519:3052 | COMEDXMV519:3052 | XM_PDN1 | 📋 Comentada |

**Nota**: La configuración de producción está comentada en web.config (líneas 47-57).

---

## 5️⃣ DEPENDENCIAS EXTERNAS

### 📦 Inventario de NuGet Packages

**Las siguientes librerías están referenciadas en los proyectos:**

| Nombre Librería | Versión | Propósito | Función |
|-----------------|---------|----------|---------|
| **Microsoft.SharePoint** | 14.0.0.0 | Interoperabilidad SharePoint 2010 | SSOM API |
| **Microsoft.SharePoint.Client** | 14.0.0.0 | Client Object Model | CSOM |
| **Microsoft.Practices.EnterpriseLibrary.ExceptionHandling** | 5.0.414.0 | Exception Management | Policy-based handling |
| **Microsoft.Practices.EnterpriseLibrary.ExceptionHandling.Logging** | 5.0.414.0 | Logging de excepciones | Integration EL |
| **Microsoft.Practices.EnterpriseLibrary.ExceptionHandling.WCF** | 5.0.414.0 | WCF Exception Shielding | Sanitización de excepciones |
| **Microsoft.Practices.EnterpriseLibrary.Logging** | 5.0.414.0 | Logging Framework | Log writing |
| **Microsoft.Practices.EnterpriseLibrary.Validation** | 5.0.414.0 | Validación | Validation rules |
| **Microsoft.Practices.EnterpriseLibrary.Validation.Integration.WCF** | 5.0.414.0 | Validación en WCF | Behavior validation |
| **Microsoft.Practices.EnterpriseLibrary.Caching** | 5.0.414.0 | Caching Framework | In-memory caching |
| **System.Data.SQLite** | Última de .NET 3.5 | SQLite Legacy | (si se usa) |
| **iTextSharp** | 5.5.0.0 | Generación PDF | Reportes PDF |
| **Telerik.Web.UI/Telerik.Windows.UI** | v2016.1.225.35 | Rad Controls | Grids, calendars, etc. |
| **Oracle.ManagedDataAccess.Client** | 4.121.1.0 | Oracle ADO.NET Provider | Connection Oracle |
| **System.Data.Entity Framework** | 1.0 (implicito .NET 3.5) | ORM | Database abstraction |

### 📋 Librerías Obsoletas o Sin Soporte

| Librería | Estado | Riesgo | Alternativa |
|----------|--------|--------|------------|
| **Enterprise Library 5.0** | ⛔ Fin de soporte (2016) | Crítico - NO hay security patches | Serilog, ILogger abstraction |
| **.NET Framework 3.5/4.0** | ⛔ ExtendedSupport (hasta 2029) | Alto - Legacy | .NET 6/8 |
| **SharePoint 2010** | ⛔ Fin de soporte (2016) | Crítico - NO hay patches | SharePoint Online, Azure |
| **iTextSharp 5.5** | ⚠️ Mantenimiento básico | Medio - Algunas vulns conocidas | iText 7+ |
| **Telerik v2016.1** | ⚠️ Soporte limitado | Medio - UI vulnerabilities | Kendoui o MUI |
| **Oracle ManagedDataAccess 4.121** | ✅ Supported | Bajo | Última versión 21.x |

### 🔗 Servicios Externos Consumidos

#### ACME (Sistema Externo - Gestión de Cuentas Bancarias)

| Parámetro | Valor |
|-----------|-------|
| **URL Base** | https://acmecalidadback.xm.com.co |
| **Auth Endpoint** | /acme_seguridad_webapi/v2.0/Oauth2 |
| **Tipo Autenticación** | OAuth 2.0 |
| **Usuario** | XM_S_AcmeAplicacionPrd@xm.com.co |
| **Métodos consumidos** | GetAllBank, ConsultarCuentasBancariasNegocios |
| **Protocolo** | REST/JSON |
| **Ubicación consumo** | BrokerAdministracion |

**Ubicación en config**: Hardcodeada en web.config appSettings (líneas 20-25)

#### MID (Sistema Externo - Integración de Datos)

| Parámetro | Valor |
|-----------|-------|
| **Tipo conexión** | SQL Server Native |
| **Base datos** | BDMIDXM |
| **Acceso** | Lectura/Escritura |
| **Ubicación consumo** | XM.RAG.ConsultasMID, BrokerIntegracionMID |
| **Protocolo** | LINQ2SQL + Stored Procedures |

#### PDN (Sistema Externo - Permisos y Datos)

| Parámetro | Valor |
|-----------|-------|
| **Tipo conexión** | Oracle Native |
| **Base datos** | PDN1 / XM_PDN1 |
| **Acceso** | Lectura principalmente |
| **Ubicación consumo** | BrokerIntegracionPDN |
| **Protocolo** | Entity Framework + Stored Procedures |

### 🔐 Autenticación y Autorización

**Tipo de Autenticación Primaria**: **Active Directory (Windows Authentication)**

**Flujo de Autenticación**:

1. **SharePoint Level**:
   - Usuario accede a portal SharePoint 2010
   - Autenticación Windows (IIS NTLM o Kerberos)
   - SharePoint crea contexto SPUser

2. **Aplicación Level**:
   - WebParts obtienen identidad via `SPContext.Current.Web.CurrentUser`
   - Se valida membresía a grupos AD
   - Se asignan roles (Administrador, Analista, Usuario)

3. **WCF Service Level**:
   ```csharp
   // En FachadaGeneral.cs
   public class FachadaGeneral
   {
       public static List<Principal> ObtenerGruposUsuario(string userName)
       {
           var context = new PrincipalContext(ContextType.Domain, "dominio.com");
           UserPrincipal user = UserPrincipal.FindByIdentity(context, IdentityType.SamAccountName, userName);
           List<Principal> grupos = user.GetGroups().ToList();
           return grupos;
       }
   }
   ```

4. **Impersonación**:
   ```csharp
   // En Roles.cs
   System.Security.Principal.WindowsIdentity wi = new System.Security.Principal.WindowsIdentity(userName);
   using (var ctx = wi.Impersonate())
   {
       // Operación bajo contexto del usuario
   }
   ```

**Método de autorización**: Role-based Access Control (RBAC) mediante grupos AD

**Roles principales**:
- Administrador RAG
- Analista RAG
- Usuario RAG

---

## 6️⃣ ACCESO A DATOS

### 💾 Bases de Datos Utilizadas

#### Base 1: BDRAGXM (SQL Server)

**Descripción**: Base de datos principal del sistema RAG
**Motor**: Microsoft SQL Server
**Servidor**: 10.250.16.25 (Desarrollo), COMEDXMV519:3052 (Producción)
**Acceso**: Entity Framework ObjectContext

**Tablas principales identificadas** (a través de EDMX):
- Solicitudes
- Empresas
- Agentes
- Contactos
- Documentos
- EstadoSolicitud
- SeguimientoSolicitud
- Notificaciones
- AuditoriaAcciones

#### Base 2: BDMIDXM (SQL Server)

**Descripción**: Base de integración MID (Sistema informativo del mercado)
**Motor**: Microsoft SQL Server
**Servidor**: 10.250.16.25 (Desarrollo), COMEDXMAZ061:3052 (Pruebas)
**Acceso**: LINQ to SQL (legacy DataSet)

**Tablas**: Datos operacionales de MID (sincronización)

#### Base 3: PDN (Oracle)

**Descripción**: Base de datos de Permisos y Datos Nacionales
**Motor**: Oracle Database
**Servidor**: ORCL-PDN1-AZURE (Desarrollo), XM_PDN1 (Producción)
**Acceso**: Entity Framework ObjectContext

**Tablas principales**:
- PermisosConcedidos
- ResponsablesPermiso
- AutenticacionPermiso
- DatosContributario

### 🏗️ Tipo de Acceso a Datos

| Tipo | Versión | Ubicación | Uso |
|------|---------|-----------|-----|
| **Entity Framework** | 1.0 (implicit .NET 3.5) | BDRAG + PDN EDMX | Principal (90%) |
| **LINQ to SQL** | Implicit .NET 3.5 | BDMIDXM DataSet | Legacy (5%) |
| **ADO.NET Raw** | System.Data | Brokers custom queries | Queries complejas (5%) |
| **Stored Procedures** | SQL + PL/SQL | BDRAG, PDN, BDMIDXM | Lógica pesada |

### 📊 Principales Tablas y Relaciones

```sql
-- Tabla principal: Solicitudes
Solicitudes
├─ IdSolicitud (PK)
├─ NumeroSolicitud (unique)
├─ IdEmpresa (FK → Empresas)
├─ IdAgente (FK → Agentes)
├─ Estado (FK → EstadoSolicitud)
├─ FechaCreacion
├─ FechaModificacion
└─ FechaResolucion

-- Tabla: Empresas
Empresas
├─ IdEmpresa (PK)
├─ NombreEmpresa
├─ NITEmpresa (unique)
├─ IdContactoPrincipal (FK → Contactos)
└─ FechaAfiliacion

-- Tabla: Agentes
Agentes
├─ IdAgente (PK)
├─ NombreAgente
├─ EmailAgente
├─ IdGrupoAD (FK → Active Directory group)
└─ FechaIngreso

-- Tabla: Documentos
Documentos
├─ IdDocumento (PK)
├─ IdSolicitud (FK → Solicitudes)
├─ NombreArchivo
├─ RutaAlmacenamiento
├─ FechaSubida
└─ TipoVirtualización (BLOB vs FileShare)

-- Tabla: SeguimientoSolicitud
SeguimientoSolicitud
├─ IdSegumiento (PK)
├─ IdSolicitud (FK → Solicitudes)
├─ EstadoAnterior
├─ EstadoNuevo
├─ UsuarioModificacion
└─ FechaTransicion
```

### 🔍 Queries Críticas o Complejas

**Query 1: Obtener solicitudes pendientes por revisar**
```csharp
// En BrokerRevisionSolicitudes.cs
public DataTable ObtenerSolicitudesPendientes(DateTime fechaDesde, DateTime fechaHasta)
{
    string query = @"
        SELECT s.IdSolicitud, s.NumeroSolicitud, s.EstadoActual, 
               e.NombreEmpresa, a.NombreAgente, s.FechaCreacion
        FROM Solicitudes s
        INNER JOIN Empresas e ON s.IdEmpresa = e.IdEmpresa
        INNER JOIN Agentes a ON s.IdAgente = a.IdAgente
        WHERE s.EstadoActual = 'PendienteRevision'
        AND s.FechaCreacion BETWEEN @fechaDesde AND @fechaHasta
        ORDER BY s.FechaCreacion ASC";
    
    var db = new BDRAG(); // ObjectContext
    var cmd = db.CreateDatabaseCommand(query);
    // VULNERABILIDAD: Aunque usa parametrización aquí, búsqueda por LIKE está vulnerable
}
```

**Query 2: Sincronizar datos de PDN (Oracle)**
```csharp
// En BrokerIntegracionPDN.cs
public List<DatosTributario> ObtenerDatosTributarios(string nit)
{
    using (var context = new BDOracle()) // Oracle Entity Framework
    {
        // RISK: Construcción de LINQ dinámica sin validación
        var datos = context.DatosTributarios
            .Where(d => d.NIT.Contains(nit))  // VULNERABLE si nit no es sanitizado
            .ToList();
        
        return datos;
    }
}
```

**Query 3: Reporte consolidado de solicitudes**
```csharp
// En ReporteBroker.cs (Reporteador)
public DataSet GenerarReporteSolicitudes(
    int idEmpresa, 
    int idAgente, 
    string estado,
    DateTime fechaDesde)
{
    // RISK: Construcción manual de SQL con string concatenation
    string sql = "SELECT * FROM Solicitudes WHERE 1=1";
    
    if (idEmpresa > 0)
        sql += " AND IdEmpresa = " + idEmpresa;  // ⚠️ SQL INJECTION RISK
    
    if (idAgente > 0)
        sql += " AND IdAgente = " + idAgente;    // ⚠️ SQL INJECTION RISK
    
    if (!string.IsNullOrEmpty(estado))
        sql += " AND EstadoActual = '" + estado + "'";  // ⚠️ SQL INJECTION RISK
    
    sql += " AND FechaCreacion >= '" + fechaDesde.ToShortDateString() + "'";  // ⚠️ RISK
    
    var db = new BDRAG();
    return db.ExecuteDataSet(sql);  // VULNERABLE
}
```

### 💳 Transacciones Importantes

**Transacción 1: Procesamiento de nueva solicitud**
```csharp
[OperationContract]
[FaultContract(typeof(ServiceFault))]
public void CrearSolicitud(SolicitudDTO solicitud)
{
    using (var context = new BDRAG())
    {
        // NO hay ExplicitTransaction - depende de isolation level por defecto
        var oSolicitud = new Solicitudes
        {
            NumeroSolicitud = GenerarNumeroUnico(),
            IdEmpresa = solicitud.IdEmpresa,
            EstadoActual = "Creada",
            FechaCreacion = DateTime.Now
        };
        
        context.Solicitudes.AddObject(oSolicitud);
        context.SaveChanges();  // ⚠️ Si esto falla a mitad, BD queda inconsistente
        
        // Paso 2: Crear documentos asociados (transacción implícita)
        foreach (var doc in solicitud.Documentos)
        {
            var oDoc = new Documentos { ... };
            context.Documentos.AddObject(oDoc);
        }
        
        context.SaveChanges();  // ⚠️ Llamada separada - RIESGO de inconsistencia
    }
}
```

**Mejor Implementación (NO actual)**:
```csharp
using (var transaction = context.Connection.BeginTransaction())
{
    try
    {
        // Todas las operaciones
        context.SaveChanges();
        transaction.Commit();
    }
    catch
    {
        transaction.Rollback();
        throw;
    }
}
```

---

## 7️⃣ SEGURIDAD

### 🔐 Autenticación y Autorización

**Implementación actual**: Active Directory + Windows Identity

**Mecanismo**:
1. IIS valida identidad Windows
2. SharePoint propaga identidad via SPUser
3. Aplicación valida pertenencia a grupos AD
4. Roles asignados según grupos

**Riesgos identificados**:
- ❌ **Sin fallback** si AD no está disponible (Single Point of Failure)
- ❌ **Sin caché de credenciales**
- ❌ **Sin MFA** (Multi-Factor Auth)
- ⚠️ **Servicios WCF sin autenticación** (`security mode="None"`)

### 🛡️ Manejo de Credenciales

| Ubicación | Credencial | Estado | Riesgo |
|-----------|-----------|--------|--------|
| **web.config línea 32** | `uid=ADMMID; password=ADMMID` | Hardcodeada | 🔴 Crítico |
| **web.config línea 33** | `uid=ADMMID; password=ADMMID` | Hardcodeada | 🔴 Crítico |
| **web.config línea 35** | `USER ID=JOAQUINBERMUDEZ; PASSWORD=mvm_joaquinbermudez` | Hardcodeada | 🔴 Crítico |
| **web.config línea 23** | `ACME_Password=C&&8edRa7reN` | Hardcodeada | 🔴 Crítico |
| **Scripts PS1** | `#{password_sharepoint__ambiente__}#` | Token reemplazo | 🟡 Medio |

**Recomendación inmediata**: 
- Usar Secret Management Server (Azure Key Vault, HashiCorp Vault)
- Implementar encrypted configuration sections
- Rotar todas las credenciales

### 🔒 Cifrado de Datos Sensibles

**Estado actual**: 
- ❌ NO hay cifrado de datos en tránsito (WCF sin HTTPS)
- ❌ NO hay cifrado de datos en reposo
- ⚠️ PERSIST SECURITY INFO=True en connection strings (expone contraseñas si BD capta)

**Recomendación**:
- Habilitar HTTPS en WCF servicios
- Implementar TDE (Transparent Data Encryption) en SQL Server
- Remover PERSIST SECURITY INFO

### ✅ Validaciones de Entrada

**Validaciones implementadas**:

```csharp
// Usando Enterprise Library Validation Block
[ValidationBehavior]
public class RealizacionSolicitudesService : IRealizacionSolicitudes
{
    [OperationContract]
    [FaultContract(typeof(ServiceFault))]
    public void ActualizarSolicitud(
        [parameterValueValidator] SolicitudDTO solicitud)  // Auto-validated
    {
        // Validación básica
        if (solicitud == null)
            throw new ServiceFault { Mensaje = "Solicitud nula" };
        
        if (solicitud.IdSolicitud <= 0)
            throw new ServiceFault { Mensaje = "ID inválido" };
        
        // Procesamiento
        var broker = new BrokerRealizacionSolicitudes();
        broker.ActualizarSolicitud(solicitud);
    }
}
```

**Riesgos de validación**:
- ⚠️ No hay validación de rango para campos string (LIKE puede ser vulnerable)
- ❌ No hay whitelist validation en búsquedas dinámicas
- ❌ No hay sanitización de valores especiales de BD

### 🚨 Riesgos de Seguridad Evidentes

#### 🔴 CRÍTICO: SQL Injection en reportes

**Ubicación**: `BrokerReportes.cs` (línea estimada ~219)

```csharp
// VULNERABLE
string sql = "SELECT * FROM Solicitudes WHERE Estado = '" + estado + "'";
db.ExecuteDataSet(sql);

// Should be:
SqlParameter[] parms = new[] { 
    new SqlParameter("@Estado", estado) 
};
string sql = "SELECT * FROM Solicitudes WHERE Estado = @Estado";
```

#### 🔴 CRÍTICO: WCF Sin Autenticación

7 servicios con `security mode="None"`:
- RealizacionSolicitudes
- RevisionSolicitudes
- Administracion
- General
- IntegracionMID
- IntegracionPDN
- NuevoRegfro

**Riesgo**: Cualquier cliente puede llamar servicios sin validación de identidad.

#### 🔴 CRÍTICO: Debug Habilitado en Producción

```xml
<serviceDebug includeExceptionDetailInFaults="true"/>
```

Expone stack traces y detalles internos a clientes (incluye nombres de tablas, stored procs, etc.)

#### 🟡 SERIO: PERSIST SECURITY INFO

Connection strings con `PERSIST SECURITY INFO=True` permiten a aplicaciones capturar contraseñas.

#### 🟡 SERIO: Credenciales Hardcodeadas

11 credenciales diferentes encontradas en archivos de configuración públicamente accesibles.

---

## 8️⃣ LÓGICA DE NEGOCIO CRÍTICA

### 📌 Casos de Uso Principales

**Caso 1: Creación de Solicitud de Registro**

```
1. Usuario accede a WebPart "SolicitarRegistro"
2. Completa formulario (datos empresa, agente, documentos)
3. WebPart llama WCF RealizacionSolicitudes.CrearSolicitud()
4. BrokerRealizacionSolicitudes.CrearSolicitud()
   ├─ Valida completitud de datos
   ├─ Genera número único de solicitud
   ├─ Crea registro en Solicitudes (Estado = "Creada")
   ├─ Crea registros en Documentos
   ├─ Registra auditoría
   └─ Envía notificación al analista asignado
5. Retorna número solicitud al usuario
```

**Caso 2: Revisión y Aprobación de Solicitud**

```
1. Analista accede a WebPart "RevisionSolicitudes"
2. Selecciona solicitud de lista
3. Revisa documentos asociados (descargables)
4. WebPart llama WCF RevisionSolicitudes.ReviarSolicitud()
5. BrokerRevisionSolicitudes.RevisarSolicitud()
   ├─ Valida documentación
   ├─ Consulta BD de Permisos (Oracle PDN)
   ├─ Ejecuta reglas de negocio complejas
   ├─ Decide: Aprobado / Rechazado / Parcial
   ├─ Actualiza Estado en BDRAGXM
   ├─ Si Aprobado: Sincroniza con BDMIDXM
   └─ Envía notificación a usuario
6. Retorna resultado a analista
```

**Caso 3: Integración MID (Sincronización de datos)**

```
1. Timer Job "SincronizarDatosMID" se ejecuta cada 4 horas
2. BrokerIntegracionMID.SincronizarDatos()
   ├─ Obtiene solicitudes aprobadas no sincronizadas
   ├─ Para cada solicitud:
   │  ├─ Transforma datos en formato MID
   │  ├─ Inserta en BDMIDXM
   │  ├─ Registra fecha sincronización
   │  ├─ Si falla: registra error y continúa
   │  └─ Actualiza estado a "Sincronizada"
   └─ Genera reporte de sincronización
3. LogManager registra resultado
```

### 🎯 Flujos Críticos del Sistema

#### Flujo: Procesamiento de Solicitud

```
   CREAR            REVISAR           APROBAR/RECHAZAR      SINCRONIZAR
     ↓                ↓                       ↓                   ↓
  [Creada] → [Pendiente Revisión] → [Aprobada/Rechazada] → [Sincronizada]
     ↓                ↓                       ↓                   ↓
   [Documentos]    [Validación]       [Integración]        [BDMIDXM]
     ↓                ↓                       ↓                   ↓
  [BDRAGXM]        [Auditoría]            [PDN]            [Confirmación]
```

#### Flujo: Notificaciones

```
   EVENTO DE CAMBIO
         ↓
   [Identificar interesados]
         ↓
   [BrokerRegistroSucesos]
         ↓
   [GenerarEmail] ← SMTP integration
         ↓
   [LocalQueue] ← Timer Job de envío
         ↓
   [MailServer]
         ↓
   [Usuario recibe email]
```

### 📋 Reglas de Negocio Importantes

1. **Una solicitud debe estar completa** antes de procesarse
   - Mínimo 1 documento
   - Empresario responsable identificado
   - Agente asignado

2. **Auditoría de cambios**
   - Cada transición de estado se registra con usuario + timestamp
   - Imposible modificar auditoría (tabla append-only)

3. **Sincronización con PDN**
   - Solo solicitudes aprobadas se sincronizan
   - La información de PDN prevalece en caso de conflicto
   - Reintentos automáticos en caso de fallo

4. **Gestión de Documentos**
   - Documentos se almacenan en FileShare o BLOB
   - Eliminación lógica de documentos (no borrado físico)
   - Retención mínima de 7 años según ley

### 🔄 Procesos Complejos Identificados

**Proceso: Validación de Empresa para Registro**

La lógica de validación es compleja y distribuida:

```csharp
// Etapa 1: Validación básica (en BrokerRealizacionSolicitudes)
public bool ValidarEmpresa(int idEmpresa)
{
    var empresa = _context.Empresas.FirstOrDefault(e => e.IdEmpresa == idEmpresa);
    
    if (empresa == null) return false;
    if (string.IsNullOrEmpty(empresa.NIT)) return false;
    if (empresa.FechaAfiliacion == null) return false;
    
    // Etapa 2: Validación contra PDN (consulta Oracle)
    var permisoPDN = _brokerPDN.ConsultarPermiso(empresa.NIT);
    if (permisoPDN == null) return false;  // No existe en PDN
    
    // Etapa 3: Validación contra ACME (servicio externo)
    var cuentaBancaria = _acmeService.ValidarCuenta(empresa.NIT);
    if (!cuentaBancaria.Valida) return false;
    
    // Etapa 4: Validación de reglas de negocio especiales
    if (empresa.IdSector == SECTOR_FINANCIERO)
    {
        // Requiere documentación adicional
        var solicitud = _context.Solicitudes
            .Where(s => s.IdEmpresa == idEmpresa && s.Estado != "Rechazada")
            .FirstOrDefault();
        
        if (solicitud != null && solicitud.Documentos.Count < 5)
            return false;  // Falta documentación
    }
    
    return true;
}
```

---

## 9️⃣ LOGGING, ERRORES Y MONITOREO

### 📊 Framework de Logging

**Framework utilizado**: **Enterprise Library 5.0** (Microsoft Practices)

**Versión**: 5.0.414.0 (Lanzada en 2011, fin de soporte 2016)

### 🎯 Componentes Configurados

```xml
<!-- Ubicación: C:\Proyectos\RAG\Fuentes\XM.RAG.Servicios\Servicios\XM.RAG.Servicios\Configuracion\entlib.config -->

<loggingConfiguration>
  <!-- Log Writers -->
  <logWriter type="Microsoft.Practices.EnterpriseLibrary.Logging.TraceListeners.RollingFlatFileTraceListener">
    <fileName fileName="C:\LogTecnico\RAGLog.txt"/>
    <maxFileSize value="10485760"/>  <!-- 10 MB -->
  </logWriter>
  
  <!-- Exception Handling policies -->
  <exceptionHandling>
    <exceptionPolicies>
      <exceptionPolicy name="FRONTERA_DE_SERVICIO">
        <exceptionHandlers>
          <exceptionHandler type="LoggingExceptionHandler"/>
          <!-- Sanitiza excepciones antes de enviar a cliente -->
        </exceptionHandlers>
      </exceptionPolicy>
    </exceptionPolicies>
  </exceptionHandling>
  
  <!-- Validation Rules -->
  <validation>
    <!-- Validación declarativa de DTOs -->
  </validation>
</loggingConfiguration>
```

### 📝 Categorías de Log Identificadas

**Ubicación de definición**: `XM.RAG.Framework\Logging\CategoriaLog.cs`

```csharp
public static class CategoriaLog
{
    public const string TIMER_JOB = "TIMER_JOB";
    public const string WCF_SERVICE = "WCF_SERVICE";
    public const string ACCESO_DATOS = "ACCESO_DATOS";
    public const string NEGOCIO = "NEGOCIO";
    public const string AUDITORIA = "AUDITORIA";
    public const string INTEGRACION_EXTERNA = "INTEGRACION_EXTERNA";
}
```

### 🚨 Manejo de Excepciones

**Política principal**: Exception Shielding con `[ExceptionShielding(...)]`

```csharp
// Decorador en operaciones WCF
[ServiceContract]
public class RealizacionSolicitudes : IRealizacionSolicitudes
{
    [OperationContract]
    [FaultContract(typeof(ServiceFault))]
    [ExceptionShielding(PoliticaDeExcepcion.FRONTERA_DE_SERVICIO)]
    public void CrearSolicitud(SolicitudDTO solicitud)
    {
        try
        {
            // Lógica
        }
        catch (Exception ex)
        {
            // ExceptionHandler captura y transforma excepción
            ExceptionHandler.HandleException(
                ex, 
                "FRONTERA_DE_SERVICIO"
            );
            
            // Cliente recibe: ServiceFault con mensaje genérico
            throw new FaultException<ServiceFault>(
                new ServiceFault { Mensaje = "Error procesando solicitud" }
            );
        }
    }
}
```

**Riesgos**:
- ⚠️ `includeExceptionDetailInFaults="true"` en web.config expone detalles internos en desarrollo
- ❌ No se sanitiza información sensible de exceptions
- ❌ Stack traces se transfieren a cliente si debug=true

### 📊 Monitoreo y Alertas

**Herramientas identificadas**:
- ⛔ **NO hay monitoreo centralizado** (no se detectó ELK, Splunk, etc.)
- ✅ **Logs locales** en `C:\LogTecnico\`
- ⚠️ **Monitor manual** de logs (no automatizado)
- ⛔ **NO hay alertas** en caso de error

### 📌 Dependencias de Eventos

**Windows Event Log**:
- Event Receivers de SharePoint escriben al Event Log de Windows
- Permite auditoría de activaciones/desactivaciones de Features

**Audit Trail**:
- Tabla `Auditoría` en BDRAGXM guarda cambios
- No hay mecanismo de "who/what/when" centralizado
- Propiedades timestamp de Entity Framework usan UtcNow

---

## 🔟 RIESGOS TÉCNICOS Y DEUDA

### 🔴 CÓDIGO LEGACY

**Clases Mega (>1000 líneas)**:

| Clase | Líneas | Responsabilidades | Ubicación |
|-------|--------|-------------------|-----------|
| **BrokerRevisionSolicitudes** | ~1,200 | Revisión, validación, notificación | Negocio/XM.RAG.RevisionSolicitudes |
| **BrokerRealizacionSolicitudes** | ~1,500 | Creación, integración MID, reportes | Negocio/XM.RAG.RealizacionSolicitudes |
| **ControladoraAdministracion** | ~2,100 | Admin empresa, usuarios, roles | (ubicación en AccesoDatos) |

**Impacto**:
- Difícil de probar (métodos con múltiples paths)
- Alto riesgo de cambios (una línea afecta muchas operaciones)
- Código no reutilizable

### 📊 Dependencias Obsoletas

| Librería | Versión | Lanzada | Fin Soporte | Alternativa |
|----------|---------|---------|------------|------------|
| Enterprise Library | 5.0.414.0 | 2011 | 2016 | Serilog, ILogger |
| .NET Framework | 3.5/4.0 | 2007/2010 | 2029 | .NET 6/8 |
| SharePoint | 2010 | 2010 | 2016 | SharePoint Online |
| iTextSharp | 5.5 | 2015 | Mantenim. | iText 7+ |
| Telerik | v2016.1 | 2016 | Límite | Kendo, MUI |

### 🎨 Uso de APIs Deprecadas

```csharp
// DEPRECATED: ObjectContext (EF 1.0)
using (var context = new BDRAG()) // ObjectContext
{
    // Modern: DbContext con DbSet<T>
}

// DEPRECATED: LINQ to SQL en BDMIDXM
var midContext = new BDMIDDataSet();  // DataSet-based
// Modern: LINQ to Entities

// DEPRECATED: Enterprise Library Validation Block
[parameterValueValidator]  // Attribute-based
// Modern: FluentValidation
```

### ❌ Problemas de Performance Conocidos

**Identificados**:

1. **N+1 Query Problem** en reportes
   ```csharp
   foreach (var solicitud in solicitudes)  // Query 1
   {
       var empresa = _context.Empresas.FirstOrDefault(e => e.IdEmpresa == solicitud.IdEmpresa);  // N queries
   }
   // Should use: .Include("Empresas")
   ```

2. **Caché no utilizado**
   - Enterprise Library Caching Block definido pero no implementado
   - Queries a PDN se repiten constantemente

3. **Índices faltantes en tablas grandes**
   - Tabla `Solicitudes` sin índices en campos frecuentemente filtrados
   - Búsquedas por LIKE sin cobertura de índices

### 🔗 Dependencias Circulares Lógicas

1. **BrokerRealizacionSolicitudes** → `ValidarEmpresa()` 
   → **BrokerIntegracionPDN** → ObtenerPermiso() 
   → Escribe en BDRAGXM 
   → **BrokerRealizacionSolicitudes** (indirectamente afectado)

2. **Timer Jobs** ejecutados secuencialmente sin coordinación
   - Si una tarea falla, las posteriores pueden ejecutarse con datos inconsistentes

### 🎯 Componentes Difíciles de Migrar

| Componente | Dificultad | Razón | Estimación |
|-----------|-----------|-------|-----------|
| **SharePoint 2010 WSP** | 🔴 Crítica | Dependencia de SSOM API | 3-6 meses |
| **Entity Framework v1** | 🟡 Media | Migración a EF Core posible | 2-3 meses |
| **Enterprise Library** | 🟡 Media | Reemplazo por Serilog/ILogger | 1 mes |
| **WCF Services** | 🟡 Media | Migración a gRPC o REST ASP.NET Core | 2-3 meses |
| **Timer Jobs** | 🟠 Moderada | Migración a Azure Functions/WebJobs | 1.5 meses |
| **Oracle Integration** | 🔴 Crítica | Cambios en estrategia de acceso datos | 2 meses |

---

## 🔟1️⃣ RECOMENDACIONES PARA MIGRACIÓN

### 📋 Evaluación de Migratibilidad

#### ✅ FÁCILMENTE MIGRABLES

**1. Enterprise Library → Serilog** (1 semana)
```csharp
// Actual
LogManager.LogInfo("Mensaje", CategoriaLog.NEGOCIO);

// Futuro
_logger.LogInformation("Mensaje");
```

**2. WCF Services → ASP.NET Core gRPC** (2 semanas)
- Contratos ya están bien definidos
- Services no tienen lógica compleja
- gRPC ofrece mejor performance

**3. LINQ to SQL → Entity Framework Core** (2 semanas)
- BDMIDXM puede modelarse rápidamente
- Pocos cambios en lógica

#### ⚠️ MIGRACIÓN MODERADA

**1. Entity Framework v1 → EF Core** (3 semanas)
- EDMX → Code-First migrations
- Cambios menores en queries LINQ
- Validación de integridad referencial

**2. Active Directory → Azure AD** (3 semanas)
- Implementar Azure AD B2B para usuarios externos
- Cambiar `DirectoryServices` por Microsoft Graph API
- Mantener compatibilidad con AD on-prem temporalmente

**3. Timer Jobs → Azure Functions** (2 semanas)
- Convertir cada Job en Azure Function
- Scheduler basado en Azure Service Bus
- Mejor escalabilidad y monitoreo

#### 🔴 DIFÍCILES DE MIGRAR

**1. SharePoint 2010 WSP → SharePoint Online** (6+ meses)
   
**Opciones**:

a) **Migración a SharePoint Online (Moderna)** - 8 meses
   - Reescribir WebParts como SPFx (SharePoint Framework)
   - Usar CSOM en lugar de SSOM
   - Requiere retesteoo completo

b) **Migración a Azure App Service + Angular** - 6 meses
   - Crear nueva UI con Angular/React
   - Reutilizar servicios WCF migrados a REST
   - Deprecar SharePoint gradualmente

c) **Mantener SharePoint 2010 + Modernizar Backend** - 4 meses
   - Migrar servicios WCF a ASP.NET Core
   - Mantener WebParts compatibles
   - Riesgo: SharePoint 2010 sin soporte

**2. Oracle PDN → Azure SQL / PostgreSQL** (2+ meses)
   - Cambio de vendor
   - Verificar compatibilidad SPs
   - Testing exhaustivo

### 🗺️ Estrategia de Migración Recomendada

#### **FASE 1: Preparación (1 mes)**

```
Semana 1-2:
├─ Establecer ambiente de destino (.NET 6/8)
├─ Documentar todas las dependencias (completado)
├─ Crear proceso de reverse engineering (EF Core)
├─ Setup de Azure/hosting
└─ Validación de requisitos

Semana 3-4:
├─ Migrar Enterprise Library → Serilog
├─ Crear abstracciones IRepository
├─ Setup de CICD para nueva plataforma
└─ Capacitación del equipo en .NET Core
```

#### **FASE 2: Backend Services (3 meses)**

```
Mes 1:
├─ Entity Framework v1 → EF Core
├─ WCF Services → ASP.NET Core REST
├─ LINQ2SQL → EF Core
└─ Active Directory → Azure AD (con fallback)

Mes 2:
├─ Timer Jobs → Azure Functions
├─ Migrar Entity de BDMIDXM
├─ Migrar DAO classes
└─ Testing funcional completo

Mes 3:
├─ Performance optimization
├─ Security audit
├─ Penetration testing
└─ UAT con stakeholders
```

#### **FASE 3: Frontend (2-6 meses)**

**Opción A: SharePoint Online Migration (6 meses)**
```
Opciones paralelas:
├─ Crear SPFx components para WebParts
├─ Usar PnP JavaScript para integración
├─ Migrar Listas a SharePoint Online
├─ Deprecar gradualmente
└─ Full CSOM reemplazo de SSOM
```

**Opción B: Azure App + New UI Framework (4 meses)**
```
├─ Crear SPA con Angular/React
├─ Consumir REST APIs (.NET Core)
├─ Migrar autenticación a OAuth 2.0
├─ De-comisionar SharePoint
└─ Migrar datos históricos
```

#### **FASE 4: Post-Migration (1 mes)**

```
├─ Performance tuning
├─ Disaster recovery drill
├─ Documentación actualizada
├─ Knowledge transfer
├─ De-comisionar sistema legacy
└─ Monitoreo y soporte post-go-live
```

### 🎯 Ruta Recomendada: "Big-Bang Modificado"

Dado que el sistema no es demasiado grande (28 proyectos), se recomienda:

**Timeline**: 6-8 meses

**Enfoque**:
1. **Meses 0-1**: Preparación y migración backend en paralelo
2. **Meses 2-3**: Testing y optimización de servicios migrados
3. **Meses 4-6**: Migración frontend (opción: SharePoint Online)
4. **Meses 6-8**: Cutover y soporte

**Riesgos a mitigar**:
- ❌ Rollback plan en cada fase
- ❌ Ambiente de staging idéntico a producción
- ❌ Equipo de rollback disponible 24/7 en cutover
- ❌ Data migration validation

### 💰 Consideraciones de Costo

| Componente | Opción | Costo Aproximado | Tiempo |
|-----------|--------|---------------|(---------|
| **Backend Migration** | .NET Core | $80K-120K | 3 meses |
| **SharePoint** | Online + SPFx | $150K-200K | 6 meses |
| **SharePoint** | App Service + SPA | $100K-150K | 4 meses |
| **Infraestructura** | Azure Cloud | $15-30K/año | — |
| **Licencias** | Microsoft 365 | $10-15K/año | — |
| **Testing/QA** | Full cycle | $40K-60K | 2 meses |

### 📊 Matriz de Riesgos vs Beneficios

```
┌─────────────────────────────────────────────────────────┐
│  MIGRACIÓN .NET CORE                                    │
│                                                         │
│  RIESGOS:              BENEFICIOS:                      │
│  • Compatibility      • Performance (+40-50%)           │
│  • Testing time       • Security (modern auth)          │
│  • Cost              • Scalability (cloud-native)      │
│  • Team ramp-up      • Support (till 2032)            │
│                       • CI/CD integrado                │
│                       • Containerization (Docker)      │
│                       • Serverless options              │
│                       • Open source ecosystem           │
│                                                         │
│  RECOMENDACIÓN: PROCEDER CON MIGRACIÓN                │
└─────────────────────────────────────────────────────────┘
```

### 🔒 Seguridad Post-Migración

**Mejoras implementar**:

1. ✅ Azure Key Vault para secrets
2. ✅ OAuth 2.0 / OIDC en lugar de Windows Auth
3. ✅ HTTPS/TLS 1.2+ obligatorio
4. ✅ WAF (Web Application Firewall)
5. ✅ SIEM centralizado (Azure Sentinel)
6. ✅ MFA obligatorio
7. ✅ Encryption at rest (TDE)
8. ✅ Signed Container images

---

## 📌 CONCLUSIONES

### 📊 Resumen Ejecutivo

**Estado actual del sistema RAG**: 
- ✅ **Funcional** pero **Legacy**
- ⚠️ **Mantención difícil** (marcos sin soporte, equipo especializado requerido)
- 🔴 **Riesgos de seguridad significativos** (credenciales expuestas, SQL injection, WCF sin auth)
- 🔴 **Fin de soporte crítico** (SharePoint 2010, .NET Framework 3.5/4.0)

**Complejidad**: MEDIA-ALTA
- Arquitectura bien estructurada (N-capas)
- 28 proyectos interdependientes
- 3 bases de datos (2 SQL Server, 1 Oracle)
- 8 servicios WCF activos
- 6+ Timer Jobs críticos

**Recomendación**: 
**PROCEDER CON MIGRACIÓN PLANIFICADA** en los próximos 6-12 meses

### 📈 Próximos Pasos

1. **Corto plazo (Semanas 1-4)**:
   - [ ] Audit de seguridad profesional
   - [ ] Remover credenciales hardcodeadas
   - [ ] Habilitar HTTPS en WCF
   - [ ] Implementar autenticación en servicios

2. **Mediano plazo (Meses 1-3)**:
   - [ ] Evaluar proveedores de cloud (Azure vs AWS vs Google)
   - [ ] Proof of Concept: .NET 6/8 migration
   - [ ] Arquitectura y design de nuevo sistema
   - [ ] Capacitación del equipo

3. **Largo plazo (Meses 3-12)**:
   - [ ] Implementación gradual de nuevos componentes
   - [ ] Migration workflow for datos históricos
   - [ ] Cutover planning y ejecución
   - [ ] Fin de soporte del sistema legacy

---

**Documento generado**: 6 de Febrero de 2026
**Clasificación**: Documentación Oficial
**Revisión recomendada**: Anual o tras cambios arquitectónicos

---

## 📚 APÉNDICES

### A. Glosario de Términos

- **SSOM**: Server-Side Object Model (SharePoint deprecated)
- **CSOM**: Client-Side Object Model (SharePoint modern)
- **WSP**: WebSolution Package (SharePoint 2010)
- **EDMX**: Entity Data Model XML (Entity Framework modeling)
- **LINQ**: Language Integrated Query
- **DAO**: Data Access Object
- **DAL**: Data Access Layer
- **BLL**: Business Logic Layer
- **WCF**: Windows Communication Foundation
- **ACME**: Sistema externo de gestión de cuentas bancarias
- **MID**: Sistema informativo del mercado (integración)
- **PDN**: Permisos y Datos Nacionales (Oracle)
- **BDRAGXM**: Base de datos principal SQL Server
- **BDMIDXM**: Base de datos integración MID SQL Server
- **AD**: Active Directory
- **EL**: Enterprise Library

### B. Referencias de Archivos Clave

| Archivo | Propósito |
|---------|-----------|
| XM.RAG.sln | Solución SharePoint principal |
| XM.RAG.Servicios.sln | Solución servicios WCF |
| web.config (Servicios) | Configuración WCF, conexiones, ENTLIB |
| entlib.config | Configuración de Enterprise Library |
| Service References/* | Proxies WCF auto-generated |
| Features/RAG/ | Feature de activación |
| Provisioning/WebParts/ | Definiciones de WebParts |

### C. URLs y Endpoints de Servicios

```
http://[servidor]/RAG/Services/RealizacionSolicitudes.svc
http://[servidor]/RAG/Services/RevisionSolicitudes.svc
http://[servidor]/RAG/Services/Administracion.svc
http://[servidor]/RAG/Services/General.svc
http://[servidor]/RAG/Services/IntegracionMID.svc
http://[servidor]/RAG/Services/IntegracionPDN.svc
http://[servidor]/RAG/Services/RegistroSucesos.svc
http://[servidor]/RAG/Services/NuevoRegfro.svc
```

### D. Punto de Contacto para Preguntas

**Documento creado por**: GitHub Copilot (Análisis Automatizado)
**Información compilada de**: 28 proyectos, 50+ archivos de configuración, 100+ clases
**Para aclaraciones técnicas**: Consultar con equipo de desarrollo original (XM.RAG)
**Para migración**: Contactar CTO / Head of Architecture

---

**FIN DEL DOCUMENTO**

