# CÓDIGO VULNERABLE vs CÓDIGO SEGURO: EJEMPLOS Y SOLUCIONES

---

## VULNERABILIDAD #1: SQL INJECTION CRÍTICA

### ❌ CÓDIGO VULNERABLE (Actual)

**Archivo:** `Transacciones.cs`, línea 219

```csharp
public bool ActualizarContactoPersona(ContactoPersona contacto)
{
    OracleConnection cnn = new OracleConnection(cadenaconexion);
    cnn.Open();
    try
    {
        if (contpersona != null && contpersona.CEDULA != null)
        {
            // VULNERABLE: Concatenación directa sin parametrización
            OracleCommand sqlc = new OracleCommand(
                "Update LAC.LAT_CONTACTOPERSONA set FechaIniApe = to_date('" + 
                contacto.FECHAINIAPE.Date.Day + "/" + 
                contacto.FECHAINIAPE.Date.Month + "/" + 
                contacto.FECHAINIAPE.Date.Year + 
                "', 'DD/MM/YYYY') where codigoneg = '" + 
                contacto.CODIGONEG + 
                "' and fechafincop is null and codigotco = '" + 
                contacto.CODIGOTCO + 
                "' and identagent = '" + 
                contacto.IDENTAGENT + 
                "'and cedula = '" + 
                contacto.CEDULA + "'", 
                cnn);
            
            OracleDataAdapter dtaa = new OracleDataAdapter();
            dtaa.UpdateCommand = sqlc;
            dtaa.UpdateCommand.ExecuteNonQuery();
        }
        return true;
    }
    catch (Exception ex)
    {
        throw ex;
    }
    finally
    {
        cnn.Close();
    }
}
```

### 🚨 RIESGOS

```
INYECCIÓN SQL POSIBLE:
├── Campo CEDULA: ' OR '1'='1
│   └─ Resultado: Bypasea WHERE clause (retorna todos los registros)
├── Campo CODIGONEG: '; DROP TABLE LAC.LAT_CONTACTOPERSONA; --
│   └─ Resultado: Eliminación de tabla
├── Campo CODIGOTCO: UNION SELECT * FROM USUARIOS --
│   └─ Resultado: Exfiltración de datos
└─ Campo IDENTAGENT: x'; UPDATE LAC.LAT_CONTACTOPERSONA SET ...
    └─ Resultado: Modificación de datos no autorizada

IMPACTO: Pérdida/modificación/exposición de TODOS los datos de contactos
```

### ✅ CÓDIGO SEGURO - SOLUCIÓN #1 (Corto Plazo)

```csharp
public bool ActualizarContactoPersona(ContactoPersona contacto, IOracleDataAccessor dataAccessor)
{
    try
    {
        if (contpersona != null && contpersona.CEDULA != null)
        {
            // SEGURO: Uso de parámetros nombrados
            string sql = @"
                UPDATE LAC.LAT_CONTACTOPERSONA 
                SET FechaIniApe = TO_DATE(:fechaIniApe, 'DD/MM/YYYY')
                WHERE CODIGONEG = :codigoNeg 
                  AND FECHAFINCOP IS NULL 
                  AND CODIGOTCO = :codigoTco 
                  AND IDENTAGENT = :identAgent 
                  AND CEDULA = :cedula";
            
            OracleCommand cmd = new OracleCommand(sql, cnn);
            
            // Parametrización segura - imposible inyectar SQL
            cmd.Parameters.Add(":fechaIniApe", OracleDbType.Date)
                .Value = contacto.FECHAINIAPE.Date;
            cmd.Parameters.Add(":codigoNeg", OracleDbType.Varchar2)
                .Value = contacto.CODIGONEG;
            cmd.Parameters.Add(":codigoTco", OracleDbType.Varchar2)
                .Value = contacto.CODIGOTCO;
            cmd.Parameters.Add(":identAgent", OracleDbType.Varchar2)
                .Value = contacto.IDENTAGENT;
            cmd.Parameters.Add(":cedula", OracleDbType.Varchar2)
                .Value = contacto.CEDULA;
            
            cnn.Open();
            int rowsAffected = cmd.ExecuteNonQuery();
            
            return rowsAffected > 0;
        }
        return false;
    }
    catch (OracleException ex)
    {
        // Logging específico de error de BD
        _logger.LogError($"BD Error al actualizar contacto: {ex.Message}", ex);
        throw new DataAccessException("Error al actualizar contacto", ex);
    }
    finally
    {
        cnn?.Close();
        cnn?.Dispose();
    }
}
```

### ✅ CÓDIGO SEGURO - SOLUCIÓN #2 (Largo Plazo - Recomendado)

```csharp
// Usar ORM (Oracle Entity Framework / LinqPad contra Oracle)
public class ContactoPersonaRepository : IRepository<ContactoPersona>
{
    private readonly IDbContext _context;
    
    public async Task<bool> UpdateAsync(ContactoPersona contacto)
    {
        try
        {
            // Validar entrada
            ValidarContactoPersona(contacto);
            
            var contactoExistente = await _context.ContactosPersona
                .FirstOrDefaultAsync(c => c.CEDULA == contacto.CEDULA 
                    && c.CODIGONEG == contacto.CODIGONEG);
            
            if (contactoExistente == null)
                throw new EntityNotFoundException("Contacto no encontrado");
            
            // Entity Framework maneja parametrización automáticamente
            contactoExistente.FechaIniApe = contacto.FECHAINIAPE;
            contactoExistente.UltimaModificacion = DateTime.Now;
            
            await _context.SaveChangesAsync();
            return true;
        }
        catch (DbUpdateException ex)
        {
            _logger.LogError($"Error al actualizar contacto: {ex.InnerException?.Message}");
            throw new DataAccessException("No se pudo actualizar el contacto", ex);
        }
    }
    
    private void ValidarContactoPersona(ContactoPersona contacto)
    {
        if (string.IsNullOrWhiteSpace(contacto.CEDULA))
            throw new ValidationException("CEDULA es requerida");
        
        if (contacto.CEDULA.Length > 20)
            throw new ValidationException("CEDULA no puede exceder 20 caracteres");
            
        if (contacto.FECHAINIAPE == default)
            throw new ValidationException("FECHAINIAPE es requerida");
    }
}
```

---

## VULNERABILIDAD #2: CREDENCIALES HARDCODEADAS

### ❌ CÓDIGO VULNERABLE (Actual)

**Archivo:** `web.config`

```xml
<connectionStrings>
    <!-- DEV -->
    <add name="LINQ2MIDConnectionString" 
         connectionString="server=10.250.16.25;uid=ADMMID;password=ADMMID;Initial Catalog=BDMIDXM" 
         providerName="System.Data.SqlClient"/>
    
    <add name="BDRAG" 
         connectionString="metadata=res://*/RAG.csdl|res://*/RAG.ssdl|res://*/RAG.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=10.250.16.25;initial catalog=BDRAGXM;uid=ADMMID;password=ADMMID;...&quot;" 
         providerName="System.Data.EntityClient"/>
    
    <!-- PROD (visible en comentarios) -->
    <!--<add name="LINQ2MIDConnectionString" 
         connectionString="server=172.16.1.163,3052;uid=PASORAG;password=W4Yo#cDvC3xF.XNl9pmLEahJgI1GedO8B2PMbs!Zu70i5kr_;Initial Catalog=BDMIDXM" 
         providerName="System.Data.SqlClient"/>-->
</connectionStrings>

<appSettings>
    <add key="ACME_Usuario" value="XM_S_AcmeAplicacionPrd@xm.com.co"/>
    <add key="ACME_Password" value="C&amp;&amp;8edRa7reN"/>
</appSettings>
```

### 🚨 RIESGOS

```
EXPOSICIÓN DE CREDENCIALES:
├── Visible en repositorio Git (incluido en comentarios)
├── Visible en backups del servidor
├── Visible en logs de compilación
├── Visible en ejecutables decompilados
├── Visible en análisis de memoria
└── Visible en pantalla durante debugging

IMPACTO: Acceso no autorizado a TODAS las bases de datos
```

### ✅ CÓDIGO SEGURO - SOLUCIÓN #1 (Corto Plazo)

**web.config - Sin credenciales:**
```xml
<connectionStrings>
    <!-- Credenciales cargadas de Azure Key Vault en tiempo de ejecución -->
    <add name="BDRAGConnectionString" 
         connectionString="" 
         providerName="System.Data.SqlClient"/>
    <add name="OracleConnectionString" 
         connectionString="" 
         providerName="System.Data.OracleClient"/>
</connectionStrings>

<appSettings>
    <!-- URLs de Key Vault, no credenciales -->
    <add key="KeyVaultUrl" value="https://mykeyvault.vault.azure.net/"/>
    <add key="KeyVaultClientId" value="[Managed Identity / Service Principal ID]"/>
</appSettings>
```

**C# - Carga segura de credenciales:**
```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

public class SecureConnectionStringProvider : IConnectionStringProvider
{
    private readonly SecretClient _secretClient;
    private readonly ILogger<SecureConnectionStringProvider> _logger;
    
    public SecureConnectionStringProvider(IConfiguration config, ILogger<SecureConnectionStringProvider> logger)
    {
        _logger = logger;
        var keyVaultUrl = new Uri(config["KeyVaultUrl"]);
        
        // Usa Managed Identity en Azure (sin almacenar credenciales)
        _secretClient = new SecretClient(keyVaultUrl, new DefaultAzureCredential());
    }
    
    public async Task<string> GetConnectionStringAsync(string connectionName)
    {
        try
        {
            // Recupera secreto dinámicamente desde Key Vault
            KeyVaultSecret secret = await _secretClient.GetSecretAsync($"CS-{connectionName}");
            
            if (secret == null)
                throw new InvalidOperationException($"Connection string '{connectionName}' no encontrada en Key Vault");
            
            _logger.LogInformation($"Connection string cargado de Key Vault: {connectionName}");
            return secret.Value;
        }
        catch (Exception ex)
        {
            _logger.LogError($"Error al cargar connection string de Key Vault: {ex.Message}");
            throw;
        }
    }
}

// Uso en Startup
public class Startup
{
    public void ConfigureServices(IServiceCollection services, IConfiguration config)
    {
        services.AddSingleton<IConnectionStringProvider, SecureConnectionStringProvider>();
        
        // Entity Framework con string dinámico
        services.AddDbContext<BDRAGContext>((provider, options) =>
        {
            var connStringProvider = provider.GetRequiredService<IConnectionStringProvider>();
            var connString = connStringProvider.GetConnectionStringAsync("BDRAG").Result;
            options.UseSqlServer(connString);
        });
    }
}
```

### ✅ CÓDIGO SEGURO - SOLUCIÓN #2 (Mejor Práctica)

**appsettings.json sin secretos:**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "KeyVault": {
    "Enabled": true,
    "VaultName": "mykeyvault",
    "UseManagedIdentity": true
  },
  "ConnectionStrings": {
    "DefaultConnection": "${KeyVault:CS-BDRAG}"
  }
}
```

**Program.cs (Configuración moderna):**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Agregar Azure Key Vault
var builtConfig = builder.Configuration.Build();
var keyVaultEndpoint = new Uri($"https://{builtConfig["KeyVault:VaultName"]}.vault.azure.net/");

builder.Configuration.AddAzureKeyVault(
    keyVaultEndpoint,
    new DefaultAzureCredential());

// Agregar servicios
builder.Services
    .AddScoped<IConnectionStringProvider, AzureKeyVaultConnectionStringProvider>()
    .AddDbContext<BDRAGContext>()
    .AddLogging(config => 
    {
        config.AddApplicationInsights();
    });

var app = builder.Build();
app.Run();
```

---

## VULNERABILIDAD #3: WCF SIN AUTENTICACIÓN

### ❌ CÓDIGO VULNERABLE (Actual)

```xml
<bindings>
  <wsHttpBinding>
    <binding name="WSHttpBinding_IRealizacionSolicitudes" 
             maxBufferPoolSize="2147483647" 
             maxReceivedMessageSize="2147483647">
      <readerQuotas maxDepth="32" 
                    maxStringContentLength="2147483647" 
                    maxArrayLength="2147483647" 
                    maxBytesPerRead="2147483647" 
                    maxNameTableCharCount="2147483647"/>
      <!-- ❌ SIN AUTENTICACIÓN -->
      <security mode="None">
        <transport clientCredentialType="None" proxyCredentialType="None" realm=""/>
        <message clientCredentialType="UserName" algorithmSuite="Default"/>
      </security>
    </binding>
  </wsHttpBinding>
</bindings>
```

### 🚨 RIESGOS

```
SIN AUTENTICACIÓN SIGNIFICA:
├── Cualquiera puede llamar a los servicios
├── Sin autorización por roles
├── Ataques DDoS sin protección
├── Imposible auditar quién hizo qué
└── Datos sensibles accesibles sin credenciales

IMPACTO: Acceso completo al sistema sin restricciones
```

### ✅ CÓDIGO SEGURO - SOLUCIÓN: Autenticación Windows + SSL

```xml
<bindings>
  <wsHttpBinding>
    <!-- Binding seguro con autenticación Windows -->
    <binding name="WSHttpBinding_IRealizacionSolicitudes_Secure" 
             maxBufferPoolSize="2147483647" 
             maxReceivedMessageSize="2147483647">
      <readerQuotas maxDepth="32" 
                    maxStringContentLength="2147483647" 
                    maxArrayLength="2147483647" 
                    maxBytesPerRead="2147483647" 
                    maxNameTableCharCount="2147483647"/>
      
      <!-- ✅ SEGURO: Transport Security + Windows Auth -->
      <security mode="TransportWithMessageCredential">
        <transport clientCredentialType="Windows" proxyCredentialType="None"/>
        <message clientCredentialType="UserName" algorithmSuite="Default"/>
      </security>
    </binding>
    
    <!-- Binding para autenticación con certificados -->
    <binding name="WSHttpBinding_IRealizacionSolicitudes_Certificate" 
             maxBufferPoolSize="2147483647" 
             maxReceivedMessageSize="2147483647">
      <readerQuotas maxDepth="32" 
                    maxStringContentLength="2147483647" 
                    maxArrayLength="2147483647" 
                    maxBytesPerRead="2147483647" 
                    maxNameTableCharCount="2147483647"/>
      
      <!-- ✅ SEGURO: SSL + Certificate Authentication -->
      <security mode="TransportWithMessageCredential">
        <transport clientCredentialType="Certificate" proxyCredentialType="None"/>
        <message negotiateServiceCredential="true" algorithmSuite="Default"/>
      </security>
    </binding>
  </wsHttpBinding>
</bindings>

<!-- Configurar servicio con binding seguro -->
<services>
  <service name="XM.RAG.Servicios.RealizacionSolicitudes" 
           behaviorConfiguration="XM.RAG.Servicios.RealizacionSolicitudesBehavior_Secure">
    
    <endpoint address="" 
              binding="wsHttpBinding" 
              bindingConfiguration="WSHttpBinding_IRealizacionSolicitudes_Secure"
              contract="XM.RAG.ContratosServicios.RealizacionSolicitudes.IRealizacionSolicitudes">
      <identity>
        <dns value="localhost"/>
      </identity>
    </endpoint>
    
    <endpoint address="mex" binding="mexHttpsBinding" contract="IMetadataExchange"/>
    
    <host>
      <baseAddresses>
        <add baseAddress="https://srvragsql/RAG/RealizacionSolicitudes/"/> <!-- HTTPS! -->
      </baseAddresses>
    </host>
  </service>
</services>

<!-- Configurar comportamiento con rutas autorizadas -->
<behaviors>
  <serviceBehaviors>
    <behavior name="XM.RAG.Servicios.RealizacionSolicitudesBehavior_Secure">
      <serviceMetadata httpGetEnabled="false" httpsGetEnabled="true"/>
      <!-- ❌ CAMBIAR includeExceptionDetailInFaults a false -->
      <serviceDebug includeExceptionDetailInFaults="false"/>
      
      <!-- ✅ Agregar autorización -->
      <serviceAuthorization principalPermissionMode="UseWindowsGroups">
        <authorizationPolicy>
          <!-- Configurar política de autorización -->
        </authorizationPolicy>
      </serviceAuthorization>
      
      <!-- ✅ Agregar throttling para proteger contra DDoS -->
      <serviceThrottling maxConcurrentCalls="100" 
                         maxConcurrentSessions="50" 
                         maxConcurrentInstances="100"/>
    </behavior>
  </serviceBehaviors>
</behaviors>
```

---

## VULNERABILIDAD #4: EXCEPCIONES GENÉRICAS SIN LOGGING

### ❌ CÓDIGO VULNERABLE

```csharp
// Archivo: Comun.cs - Línea 41
public static void MetodoConError()
{
    try
    {
        // ... código ...
        int resultado = 10 / int.Parse(entrada); // Puede fallar
    }
    catch (Exception ex)
    {
        // ❌ Excepción silenciada - NO SE REGISTRA NADA
        // El error simplemente desaparece
        // Imposible debuguear qué pasó
    }
}

// Archivo: GeneralDAO.cs - Línea 206-209
catch (Exception ex)
{
    return null;  // Retorna null
    throw ex;     // ❌ CÓDIGO MUERTO - Nunca se ejecuta
}
```

### 🚨 RIESGOS

```
EXCEPCIONES SILENCIADAS:
├── No se sabe qué falló en producción
├── Debugging imposible
├── Auditoría ausente
├── El sistema falla silenciosamente
└── Acumulación de errores no detectados
```

### ✅ CÓDIGO SEGURO

```csharp
using Microsoft.Extensions.Logging;
using System;

public class ContactoService : IContactoService
{
    private readonly ILogger<ContactoService> _logger;
    
    public ContactoService(ILogger<ContactoService> logger)
    {
        _logger = logger;
    }
    
    // ✅ Manejo específico de excepciones
    public async Task<ContactoDto> ObtenerContactoAsync(string cedula)
    {
        try
        {
            if (string.IsNullOrWhiteSpace(cedula))
            {
                _logger.LogWarning("Se intentó obtener contacto con cédula vacía");
                throw new ValidationException("Cédula es requerida");
            }
            
            // Validar formato
            if (!ValidarFormatoCedula(cedula))
            {
                _logger.LogWarning($"Formato de cédula inválido: {cedula}");
                throw new ValidationException("Formato de cédula inválido");
            }
            
            var contacto = await _repository.GetByUniqueIdentifierAsync(cedula);
            
            if (contacto == null)
            {
                _logger.LogInformation($"Contacto no encontrado para cédula: {cedula}");
                throw new EntityNotFoundException($"Contacto con cédula {cedula} no encontrado");
            }
            
            return MapToDto(contacto);
        }
        catch (ValidationException ex)
        {
            _logger.LogWarning($"Validación falló: {ex.Message}");
            throw; // Re-lanzar después de loguear
        }
        catch (EntityNotFoundException ex)
        {
            _logger.LogInformation($"Entidad no encontrada: {ex.Message}");
            throw;
        }
        catch (DbUpdateException ex)
        {
            // Error de BD - loguear detalles
            _logger.LogError(
                ex,
                "Error de base de datos al obtener contacto con cédula: {cedula}",
                cedula);
            throw new DataAccessException("Error al acceder a la base de datos", ex);
        }
        catch (Exception ex)
        {
            // Excepción inesperada
            _logger.LogError(
                ex,
                "Error inesperado en ObtenerContactoAsync con cédula: {cedula}",
                cedula);
            
            // No revelar detalles internos al cliente
            throw new ApplicationException("Error al procesar la solicitud", ex);
        }
    }
    
    // ✅ Validación específica
    private bool ValidarFormatoCedula(string cedula)
    {
        try
        {
            return cedula.Length >= 8 && cedula.Length <= 20 && 
                   cedula.All(char.IsLetterOrDigit);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error validando formato de cédula: {cedula}", cedula);
            return false;
        }
    }
}

// ✅ Excepciones personalizadas
public class ValidationException : ApplicationException
{
    public ValidationException(string message) : base(message) { }
    public ValidationException(string message, Exception inner) 
        : base(message, inner) { }
}

public class EntityNotFoundException : ApplicationException
{
    public EntityNotFoundException(string message) : base(message) { }
}

public class DataAccessException : ApplicationException
{
    public DataAccessException(string message, Exception inner) 
        : base(message, inner) { }
}
```

---

## VULNERABILIDAD #5: PERSIST SECURITY INFO=True

### ❌ CÓDIGO VULNERABLE

```xml
<!-- web.config -->
<add name="BDOracle" 
     connectionString="metadata=res://*/PDN.csdl|...;DATA SOURCE=ORCL-PDN1-AZURE;PASSWORD=mvm_joaquinbermudez;PERSIST SECURITY INFO=True;USER ID=JOAQUINBERMUDEZ;" 
     providerName="System.Data.EntityClient"/>

<add name="BDRAGXM" 
     connectionString="Data Source=MVMSW523\SQLDEV2014;Persist Security Info=True;User ID=ADMMID;Password=ADOMMID" 
     providerName="System.Data.SqlClient"/>
```

### 🚨 RIESGOS

```
PERSIST SECURITY INFO=True PERMITE:
├── Extraer credenciales de objeto SqlConnection en memoria
├── Acceso mediante reflexión desde aplicación comprometida
├── Exposición en dumps de memoria
├── Vulnerabilidad en procesos que usan la conexión
└── Difícil de detectar después del compromiso

Código Atacante:
    SqlConnection conn = new SqlConnection(connectionString);
    string credentials = conn.ConnectionString; // La contraseña está visible!
```

### ✅ CÓDIGO SEGURO

```xml
<!-- web.config - CORRECTO -->
<add name="BDOracle" 
     connectionString="metadata=res://*/PDN.csdl|...;DATA SOURCE=ORCL-PDN1-AZURE;USER ID=JOAQUINBERMUDEZ;" 
     providerName="System.Data.EntityClient"/>
     <!-- ✅ Sin PASSWORD en el string - Password viene de Key Vault
          ✅ PERSIST SECURITY INFO=False (por defecto) -->

<add name="BDRAGXM" 
     connectionString="Data Source=MVMSW523\SQLDEV2014;User ID=ADMMID;Persist Security Info=False;" 
     providerName="System.Data.SqlClient"/>
     <!-- ✅ PERSIST SECURITY INFO=False
          ✅ Sin password en el string -->
```

**C# - Construcción segura de connection strings:**

```csharp
public class SecureConnectionStringBuilder
{
    private readonly IConnectionStringProvider _provider;
    
    public string BuildSqlServerConnectionString(string serverName, string databaseName, string userId)
    {
        var password = _provider.GetSecretAsync($"sql-{userId}-password").Result;
        
        var builder = new SqlConnectionStringBuilder
        {
            DataSource = serverName,
            InitialCatalog = databaseName,
            UserID = userId,
            Password = password,
            PersistSecurityInfo = false,  // ✅ SEGURO
            ConnectTimeout = 30,
            Encrypt = true,               // ✅ Encriptar conexión
            TrustServerCertificate = true // ✅ Verificar certificado
        };
        
        return builder.ConnectionString;
    }
    
    public string BuildOracleConnectionString(string dataSource, string userId)
    {
        var password = _provider.GetSecretAsync($"oracle-{userId}-password").Result;
        
        var builder = new OracleConnectionStringBuilder
        {
            DataSource = dataSource,
            UserID = userId,
            Password = password,
            PersistSecurityInfo = false,  // ✅ SEGURO
            ConnectTimeout = 30
        };
        
        return builder.ConnectionString;
    }
}
```

---

## RESUMEN DE REMEDIACIÓN INMEDIATA

| Vulnerabilidad | Cambio Mínimo | Verificación |
|---|---|---|
| SQL Injection | `Línea 219: Transacciones.cs` - Usar parámetros | Test: `' OR '1'='1` no causa bypass |
| Creds Hardcoded | Migrar a Key Vault + Managed Identity | Creds rotadas, archivos sin secretos |
| WCF sin Auth | Cambiar `security mode="None"` a `"Transport"` | Requerir certificados/Windows auth |
| Debug Info | `web.config:274` - `includeExceptionDetailInFaults="false"` | Verify WSDL no contiene stack traces |
| Persist Info | Cambiar `True` a `False` en 3 lugares | Test que conexión funciona |

---

**Documento:** Ejemplos de Código Vulnerable vs Seguro  
**Generado:** 6 de febrero de 2026  
