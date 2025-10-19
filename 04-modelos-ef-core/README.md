# 04 - Modelos y Entity Framework Core

## 🎯 Objetivo de Aprendizaje
Al finalizar esta sección, el estudiante será capaz de:
- Diseñar modelos de datos con Entity Framework Core
- Configurar y conectar una base de datos
- Realizar operaciones CRUD básicas
- Implementar migraciones y actualizaciones de esquema

## 📚 Contenidos Principales

### 1. Introducción a Entity Framework Core
**¿Qué es un ORM?**
ORM significa "Object-Relational Mapping". Es como un "traductor" que convierte objetos de C# en tablas de base de datos automáticamente.

**Analogía:**
Imagina que tienes un traductor que convierte lo que dices en español a inglés automáticamente. Entity Framework hace lo mismo pero entre C# y SQL.

```csharp
// Sin ORM (SQL manual) 😰
string sql = "INSERT INTO Personas (Nombre, Edad) VALUES (@nombre, @edad)";
// Código complejo para ejecutar...

// Con EF Core (automático) 😊
var persona = new Persona { Nombre = "Juan", Edad = 25 };
context.Personas.Add(persona);
await context.SaveChangesAsync();
```

**Ventajas de EF Core:**
- **Menos código**: No escribes SQL manualmente
- **Más seguro**: Previene inyección SQL automáticamente
- **Multiplataforma**: Funciona con SQL Server, PostgreSQL, MySQL, etc.
- **Intellisense**: Autocompletado en Visual Studio

**Conceptos básicos:**
- **DbContext**: La "conexión" a tu base de datos
- **DbSet**: Una "tabla" en tu base de datos
- **Entity**: Una "fila" en tu tabla (un objeto C#)

### 2. Diseño de Modelos
**Clases de entidad explicadas:**

```csharp
// Ejemplo de una entidad básica
public class Producto
{
    // Llave primaria (EF Core lo detecta automáticamente por el nombre "Id")
    public int Id { get; set; }
    
    // Propiedades básicas
    public string Nombre { get; set; }
    public decimal Precio { get; set; }
    public DateTime FechaCreacion { get; set; }
    
    // Propiedad opcional (puede ser null)
    public string? Descripcion { get; set; }
    
    // Relación con otra entidad
    public int CategoriaId { get; set; }  // Llave foránea
    public virtual Categoria Categoria { get; set; }  // Navegación
}
```

**Tipos de datos comunes:**
| C# Type | SQL Type | Uso |
|---------|----------|-----|
| `int` | `int` | Números enteros |
| `string` | `nvarchar` | Texto |
| `decimal` | `decimal` | Dinero, precios |
| `DateTime` | `datetime2` | Fechas |
| `bool` | `bit` | Verdadero/Falso |
| `double` | `float` | Números decimales |

**Atributos de validación para el Sistema de Clínica:**
```csharp
public class Paciente
{
    public int Id { get; set; }
    
    [Required(ErrorMessage = "Los nombres son obligatorios")]
    [StringLength(100, ErrorMessage = "Máximo 100 caracteres")]
    public string Nombres { get; set; }
    
    [EmailAddress(ErrorMessage = "Email inválido")]
    public string Email { get; set; }
    
    [Range(0, 120, ErrorMessage = "Edad entre 0 y 120 años")]
    public int Edad { get; set; }
    
    [RegularExpression(@"^\d{10}$", ErrorMessage = "La cédula debe tener 10 dígitos")]
    public string Cedula { get; set; }
}
```

**Tipos de relaciones en el Sistema de Clínica:**
```csharp
// 1. Uno a Muchos (Un médico puede tener muchas citas)
public class Medico
{
    public int Id { get; set; }
    public string Nombres { get; set; }
    public virtual ICollection<CitaMedica> Citas { get; set; } = new HashSet<CitaMedica>();
}

public class CitaMedica
{
    public int Id { get; set; }
    public DateTime FechaHora { get; set; }
    public int MedicoId { get; set; }  // Llave foránea
    public virtual Medico Medico { get; set; }  // Navegación
}

// 2. Muchos a Muchos (Pacientes pueden tener múltiples medicamentos y viceversa)
public class Paciente
{
    public int Id { get; set; }
    public string Nombres { get; set; }
    public virtual ICollection<HistorialMedico> HistorialMedico { get; set; } = new HashSet<HistorialMedico>();
}

public class HistorialMedico
{
    public int Id { get; set; }
    public int PacienteId { get; set; }
    public int MedicoId { get; set; }
    public virtual Paciente Paciente { get; set; }
    public virtual Medico Medico { get; set; }
}

// 2. Muchos a Muchos (Un paciente puede tener múltiples historiales médicos y viceversa)
public class Paciente
{
    public int Id { get; set; }
    public string Nombres { get; set; }
    public virtual ICollection<HistorialMedico> HistorialMedico { get; set; } = new HashSet<HistorialMedico>();
}

public class Medicamento
{
    public int Id { get; set; }
    public string Nombre { get; set; }
    public virtual ICollection<HistorialMedicamento> HistorialMedicamentos { get; set; } = new HashSet<HistorialMedicamento>();
}
```

### 3. Configuración de Base de Datos
**Connection strings explicados:**
```json
// appsettings.json
{
  "ConnectionStrings": {
    // SQL Server LocalDB (para desarrollo)
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=MiApp;Trusted_Connection=true",
    
    // SQL Server completo
    "ProductionConnection": "Server=mi-servidor;Database=MiApp;User Id=usuario;Password=clave123",
    
    // PostgreSQL
    "PostgresConnection": "Host=localhost;Database=MiApp;Username=postgres;Password=clave123"
  }
}
```

**Configuración en Program.cs:**
```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Registrar Entity Framework
builder.Services.AddDbContext<MiDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();
```

**Crear DbContext:**
```csharp
// Data/MiDbContext.cs
public class MiDbContext : DbContext
{
    public MiDbContext(DbContextOptions<MiDbContext> options) : base(options)
    {
    }
    
    // Cada DbSet representa una tabla
    public DbSet<Producto> Productos { get; set; }
    public DbSet<Categoria> Categorias { get; set; }
    public DbSet<Usuario> Usuarios { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configuraciones adicionales
        modelBuilder.Entity<Producto>()
            .Property(p => p.Precio)
            .HasColumnType("decimal(18,2)");  // Precisión para dinero
            
        // Datos iniciales (seed data) para especialidades médicas
        modelBuilder.Entity<Especialidad>().HasData(
            new Especialidad { Id = 1, Nombre = "Medicina General", Descripcion = "Atención médica general" },
            new Especialidad { Id = 2, Nombre = "Cardiología", Descripcion = "Especialidad del corazón" },
            new Especialidad { Id = 3, Nombre = "Pediatría", Descripcion = "Atención médica infantil" }
        );
    }
}
```

### 4. Migraciones en Visual Studio 2022
**¿Qué son las migraciones?**
Las migraciones son como "instrucciones" que le dicen a la base de datos cómo cambiar su estructura (agregar tablas, columnas, etc.).

**Analogía del Hospital:**
Es como una "orden médica" que le dice al laboratorio (base de datos) cómo preparar los análisis (estructura de tablas).

**Comandos en Package Manager Console de Visual Studio 2022:**
```powershell
# 1. Crear una migración (después de cambiar modelos)
Add-Migration NombreDeLaMigracion

# 2. Aplicar cambios a la base de datos
Update-Database

# 3. Ver el estado actual de migraciones
Get-Migration

# 4. Volver a una migración anterior (rollback)
Update-Database -Migration NombreMigracionAnterior

# 5. Generar script SQL (para producción)
Script-Migration
```

**Flujo típico de trabajo en Visual Studio 2022:**
1. **Modificas modelos** → Agregas nueva propiedad a clase `Paciente`
2. **Abres Package Manager Console** → Tools → NuGet Package Manager → Package Manager Console
3. **Creas migración** → `Add-Migration AgregadoCampoAlergias`
4. **Revisas la migración** → EF generó el código SQL automáticamente
5. **Aplicas cambios** → `Update-Database`
6. **¡Listo!** → Tu base de datos tiene el nuevo campo para alergias

**Ejemplo de archivo de migración generado para la clínica:**
```csharp
public partial class AgregadoCampoAlergias : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "Alergias",
            table: "Pacientes",
            type: "nvarchar(500)",
            nullable: false,
            defaultValue: "");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "Email",
            table: "Usuarios");
    }
}
```

## 🔧 Actividad Práctica en Clase (2 horas)

## 🔧 Actividad Práctica en Clase (2 horas)

### 🚨 **ENFOQUE ALTERNATIVO: Database First con Scaffolding**

> **Para profesores:** Este enfoque es más rápido cuando ya tienes una base de datos diseñada y quieres generar automáticamente los modelos, DbContext y controladores.

#### ⚠️ **Situación Actual del Proyecto:**
Si ya han creado controladores y vistas manualmente, necesitamos manejar esto cuidadosamente para evitar conflictos.

### **Opción A: Proyecto Nuevo con Scaffolding Completo** ⭐ **(RECOMENDADO)**

#### Paso 1: Crear Base de Datos de Clínica Primero
```sql
-- Ejecutar este script en SQL Server Management Studio o SQL Server Object Explorer
CREATE DATABASE ClinicAppDB;
GO

USE ClinicAppDB;
GO

-- Tabla Especialidades
CREATE TABLE Especialidades (
    Id int IDENTITY(1,1) PRIMARY KEY,
    Nombre nvarchar(100) NOT NULL UNIQUE,
    Descripcion nvarchar(500) NULL,
    Activa bit NOT NULL DEFAULT 1
);

-- Tabla Pacientes
CREATE TABLE Pacientes (
    Id int IDENTITY(1,1) PRIMARY KEY,
    Cedula nchar(10) NOT NULL UNIQUE,
    Nombres nvarchar(100) NOT NULL,
    Apellidos nvarchar(100) NOT NULL,
    FechaNacimiento date NOT NULL,
    Telefono nvarchar(20) NOT NULL,
    Email nvarchar(150) NOT NULL UNIQUE,
    Direccion nvarchar(250) NULL,
    TipoSangre nvarchar(5) NULL,
    ContactoEmergencia nvarchar(100) NULL,
    TelefonoEmergencia nvarchar(20) NULL,
    FechaRegistro datetime2 NOT NULL DEFAULT GETDATE(),
    Activo bit NOT NULL DEFAULT 1
);

-- Tabla Medicos
CREATE TABLE Medicos (
    Id int IDENTITY(1,1) PRIMARY KEY,
    Cedula nchar(10) NOT NULL UNIQUE,
    Nombres nvarchar(100) NOT NULL,
    Apellidos nvarchar(100) NOT NULL,
    EspecialidadId int NOT NULL,
    NumeroLicencia nvarchar(50) NOT NULL UNIQUE,
    Telefono nvarchar(20) NULL,
    Email nvarchar(150) NULL,
    HorarioInicio time NOT NULL DEFAULT '08:00:00',
    HorarioFin time NOT NULL DEFAULT '17:00:00',
    Activo bit NOT NULL DEFAULT 1,
    FechaRegistro datetime2 NOT NULL DEFAULT GETDATE(),
    FOREIGN KEY (EspecialidadId) REFERENCES Especialidades(Id)
);

-- Tabla CitasMedicas
CREATE TABLE CitasMedicas (
    Id int IDENTITY(1,1) PRIMARY KEY,
    PacienteId int NOT NULL,
    MedicoId int NOT NULL,
    FechaHora datetime2 NOT NULL,
    Motivo nvarchar(500) NULL,
    Estado nvarchar(20) NOT NULL DEFAULT 'Programada',
    Observaciones nvarchar(1000) NULL,
    FechaCreacion datetime2 NOT NULL DEFAULT GETDATE(),
    FOREIGN KEY (PacienteId) REFERENCES Pacientes(Id),
    FOREIGN KEY (MedicoId) REFERENCES Medicos(Id)
);

-- Tabla HistorialesMedicos
CREATE TABLE HistorialesMedicos (
    Id int IDENTITY(1,1) PRIMARY KEY,
    PacienteId int NOT NULL,
    MedicoId int NOT NULL,
    FechaConsulta datetime2 NOT NULL,
    MotivoConsulta nvarchar(500) NULL,
    Diagnostico nvarchar(2000) NULL,
    Tratamiento nvarchar(1000) NULL,
    Medicamentos nvarchar(500) NULL,
    Observaciones nvarchar(1000) NULL,
    ProximaCita datetime2 NULL,
    FOREIGN KEY (PacienteId) REFERENCES Pacientes(Id),
    FOREIGN KEY (MedicoId) REFERENCES Medicos(Id)
);

-- Datos iniciales para Especialidades
INSERT INTO Especialidades (Nombre, Descripcion) VALUES
('Medicina General', 'Atención médica general y preventiva'),
('Cardiología', 'Especialidad en enfermedades del corazón'),
('Pediatría', 'Atención médica para niños y adolescentes'),
('Ginecología', 'Atención médica para la mujer'),
('Dermatología', 'Especialidad en enfermedades de la piel'),
('Traumatología', 'Especialidad en lesiones del sistema músculo-esquelético');

-- Datos de prueba para Medicos
INSERT INTO Medicos (Cedula, Nombres, Apellidos, EspecialidadId, NumeroLicencia, Telefono, Email) VALUES
('1234567890', 'Carlos', 'Ramírez', 2, 'LIC-001', '0987654321', 'carlos.ramirez@clinicapp.com'),
('0987654321', 'Ana', 'López', 1, 'LIC-002', '0976543210', 'ana.lopez@clinicapp.com'),
('1122334455', 'Luis', 'Morales', 3, 'LIC-003', '0965432109', 'luis.morales@clinicapp.com');

-- Datos de prueba para Pacientes
INSERT INTO Pacientes (Cedula, Nombres, Apellidos, FechaNacimiento, Telefono, Email, TipoSangre) VALUES
('1111111111', 'María', 'González', '1985-03-15', '0912345678', 'maria.gonzalez@email.com', 'O+'),
('2222222222', 'Juan', 'Pérez', '1990-07-22', '0923456789', 'juan.perez@email.com', 'A+'),
('3333333333', 'Carmen', 'Silva', '2015-12-10', '0934567890', 'carmen.silva@email.com', 'B+');
```

#### Paso 2: Instalar Paquetes NuGet para Scaffolding
```
🔧 EN VISUAL STUDIO 2022:

1. Clic derecho en el proyecto "ClinicApp"
2. Seleccionar "Manage NuGet Packages..."
3. Buscar e instalar estos paquetes:

📦 Microsoft.EntityFrameworkCore.SqlServer (8.0.0)
📦 Microsoft.EntityFrameworkCore.Tools (8.0.0)
📦 Microsoft.EntityFrameworkCore.Design (8.0.0)
📦 Microsoft.VisualStudio.Web.CodeGeneration.Design (8.0.0)

Opcionalmente, usar Package Manager Console:

Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 8.0.0
Install-Package Microsoft.EntityFrameworkCore.Tools -Version 8.0.0
Install-Package Microsoft.EntityFrameworkCore.Design -Version 8.0.0
Install-Package Microsoft.VisualStudio.Web.CodeGeneration.Design -Version 8.0.0

```

#### Paso 3: Configurar Connection String
```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ClinicAppDB;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

#### Paso 4: Hacer Scaffolding de la Base de Datos
```powershell
# En Package Manager Console de Visual Studio 2022
# Tools → NuGet Package Manager → Package Manager Console

# 🔧 COMANDO PARA GENERAR TODO AUTOMÁTICAMENTE:
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=ClinicAppDB;Trusted_Connection=true;MultipleActiveResultSets=true" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Context ClinicaContext -DataAnnotations -Force

# Este comando crea automáticamente:
# ✅ Todos los modelos en la carpeta Models/
# ✅ El DbContext (ClinicaContext.cs)
# ✅ Con todas las relaciones configuradas
# ✅ Con Data Annotations para validaciones
```

#### Paso 5: Registrar DbContext en Program.cs
```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using ClinicApp.Models; // Namespace generado automáticamente

var builder = WebApplication.CreateBuilder(args);

// Agregar servicios
builder.Services.AddRazorPages();
builder.Services.AddControllers();

// 🔧 CONFIGURAR EL CONTEXTO GENERADO
builder.Services.AddDbContext<ClinicaContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapRazorPages();
app.MapControllers();

app.Run();
```

#### Paso 6: Generar Controladores y Vistas Automáticamente
```
🔧 EN VISUAL STUDIO 2022:

1. Clic derecho en la carpeta "Controllers"
2. Add → Controller...
3. Seleccionar "MVC Controller with views, using Entity Framework"
4. Model class: Paciente
5. Data context class: ClinicaContext
6. Controller name: PacientesController
7. ✅ Generate views
8. Add

Repetir para: Medicos, CitasMedicas, Especialidades
```

### **Opción B: Manejar Conflictos con Trabajo Existente** 🔄

Si ya tienen controladores y vistas creados manualmente:

#### Paso 1: Respaldar Trabajo Existente
```
📂 CREAR CARPETA DE RESPALDO:
- Controllers_Backup/
- Views_Backup/
- Models_Backup/

Copiar archivos existentes antes del scaffolding
```

#### Paso 2: Hacer Scaffolding Selectivo
```powershell
# Solo generar modelos, no controladores
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=ClinicAppDB;Trusted_Connection=true" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Context ClinicaContext -DataAnnotations -Force -NoOnConfiguring

# Luego adaptar controladores existentes para usar los nuevos modelos
```

#### Paso 3: Comparar y Fusionar
```csharp
// Ejemplo: Adaptar controlador existente
public class PacientesController : Controller
{
    private readonly ClinicaContext _context; // Usar el contexto generado

    public PacientesController(ClinicaContext context)
    {
        _context = context;
    }

    // Mantener la lógica existente pero usar los nuevos modelos
    public async Task<IActionResult> Index()
    {
        return View(await _context.Pacientes.ToListAsync());
    }
}
```

---

### ✅ **Resultado Final del Scaffolding**

Después del scaffolding automático tendrás:

#### 🔧 Modelos Generados (Models/)
```csharp
// Models/Paciente.cs (GENERADO AUTOMÁTICAMENTE)
public partial class Paciente
{
    public int Id { get; set; }
    public string Cedula { get; set; } = null!;
    public string Nombres { get; set; } = null!;
    public string Apellidos { get; set; } = null!;
    // ... todas las propiedades
    
    public virtual ICollection<CitasMedica> CitasMedicas { get; set; } = new List<CitasMedica>();
    public virtual ICollection<HistorialesMedico> HistorialesMedicos { get; set; } = new List<HistorialesMedico>();
}

// Models/Medico.cs (GENERADO AUTOMÁTICAMENTE)
public partial class Medico
{
    public int Id { get; set; }
    public string Cedula { get; set; } = null!;
    public string Nombres { get; set; } = null!;
    // ... todas las propiedades
    
    public virtual Especialidade Especialidad { get; set; } = null!;
    public virtual ICollection<CitasMedica> CitasMedicas { get; set; } = new List<CitasMedica>();
}
```

#### 🔧 DbContext Generado (Models/ClinicaContext.cs)
```csharp
public partial class ClinicaContext : DbContext
{
    public ClinicaContext(DbContextOptions<ClinicaContext> options) : base(options) { }

    public virtual DbSet<CitasMedica> CitasMedicas { get; set; }
    public virtual DbSet<Especialidade> Especialidades { get; set; }
    public virtual DbSet<HistorialesMedico> HistorialesMedicos { get; set; }
    public virtual DbSet<Medico> Medicos { get; set; }
    public virtual DbSet<Paciente> Pacientes { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Todas las configuraciones de relaciones generadas automáticamente
    }
}
```

#### 🔧 Controladores Generados (Controllers/)
Si elegiste generar controladores, tendrás:
- `PacientesController.cs` - CRUD completo
- `MedicosController.cs` - CRUD completo  
- `CitasMedicasController.cs` - CRUD completo
- `EspecialidadesController.cs` - CRUD completo

#### 🔧 Vistas Generadas (Views/)
```
Views/
├── Pacientes/
│   ├── Index.cshtml
│   ├── Create.cshtml
│   ├── Edit.cshtml
│   ├── Details.cshtml
│   └── Delete.cshtml
├── Medicos/
│   └── (mismas vistas)
└── CitasMedicas/
    └── (mismas vistas)
```

---

### 🎯 **Ventajas del Enfoque Database First**

✅ **Para Estudiantes:**
- ⚡ **Más rápido**: No crear modelos manualmente
- 🎯 **Menos errores**: Relaciones automáticas
- 📚 **Aprenden SQL**: Primero diseñan BD
- 🔧 **Herramientas reales**: Usan scaffolding profesional

✅ **Para Profesores:**
- ⏰ **Ahorra tiempo**: Generación automática
- 🎓 **Enfoque práctico**: Como en empresas reales
- 🔍 **Fácil corrección**: Estructura predecible
- 📊 **Visualización clara**: BD primero, código después

---

### 🚀 **Siguiente Paso: Probar la Aplicación**

```bash
# Ejecutar la aplicación
dotnet run

# Navegar a:
https://localhost:7xxx/Pacientes
https://localhost:7xxx/Medicos
https://localhost:7xxx/CitasMedicas
```

> **💡 Sugerencia para Clase:** Empezar con Database First permite que los estudiantes vean resultados inmediatos y luego entiendan cómo funciona EF Core "por debajo".
        [StringLength(10, MinimumLength = 10, ErrorMessage = "La cédula debe tener 10 dígitos")]
        public string Cedula { get; set; }
        
        [Required(ErrorMessage = "Los nombres son obligatorios")]
        [StringLength(100, ErrorMessage = "Los nombres no pueden exceder 100 caracteres")]
        public string Nombres { get; set; }
        
        [Required(ErrorMessage = "Los apellidos son obligatorios")]
        [StringLength(100, ErrorMessage = "Los apellidos no pueden exceder 100 caracteres")]
        public string Apellidos { get; set; }
        
        [Required(ErrorMessage = "La fecha de nacimiento es obligatoria")]
        [DataType(DataType.Date)]
        public DateTime FechaNacimiento { get; set; }
        
        [Required(ErrorMessage = "El teléfono es obligatorio")]
        [StringLength(20, ErrorMessage = "El teléfono no puede exceder 20 caracteres")]
        public string Telefono { get; set; }
        
        [Required(ErrorMessage = "El email es obligatorio")]
        [StringLength(150, ErrorMessage = "El email no puede exceder 150 caracteres")]
        [EmailAddress(ErrorMessage = "Formato de email inválido")]
        public string Email { get; set; }
        
        [StringLength(250, ErrorMessage = "La dirección no puede exceder 250 caracteres")]
        public string? Direccion { get; set; }
        
        [StringLength(5, ErrorMessage = "El tipo de sangre no puede exceder 5 caracteres")]
        public string? TipoSangre { get; set; }
        
        [StringLength(100, ErrorMessage = "El contacto de emergencia no puede exceder 100 caracteres")]
        public string? ContactoEmergencia { get; set; }
        
        [StringLength(20, ErrorMessage = "El teléfono de emergencia no puede exceder 20 caracteres")]
        public string? TelefonoEmergencia { get; set; }
        
        public DateTime FechaRegistro { get; set; } = DateTime.Now;
        
        public bool Activo { get; set; } = true;
        
        // Propiedades de navegación (relaciones)
        public virtual ICollection<CitaMedica> Citas { get; set; } = new HashSet<CitaMedica>();
        public virtual ICollection<HistorialMedico> HistorialMedico { get; set; } = new HashSet<HistorialMedico>();
        
        // Propiedad calculada (no se guarda en BD)
        [NotMapped]
        public string NombreCompleto => $"{Nombres} {Apellidos}";
        
        [NotMapped]
        public int Edad 
        {
            get
            {
                var edad = DateTime.Today.Year - FechaNacimiento.Year;
                if (FechaNacimiento.Date > DateTime.Today.AddYears(-edad)) edad--;
                return edad;
            }
        }
    }
}

// Models/Medico.cs
namespace ClinicApp.Models
{
    public class Medico
    {
        public int Id { get; set; }
        
        [Required(ErrorMessage = "La cédula es obligatoria")]
        [StringLength(10, MinimumLength = 10, ErrorMessage = "La cédula debe tener 10 dígitos")]
        public string Cedula { get; set; }
        
        [Required(ErrorMessage = "Los nombres son obligatorios")]
        [StringLength(100, ErrorMessage = "Los nombres no pueden exceder 100 caracteres")]
        public string Nombres { get; set; }
        
        [Required(ErrorMessage = "Los apellidos son obligatorios")]
        [StringLength(100, ErrorMessage = "Los apellidos no pueden exceder 100 caracteres")]
        public string Apellidos { get; set; }
        
        [Required(ErrorMessage = "La especialidad es obligatoria")]
        public int EspecialidadId { get; set; }
        
        [Required(ErrorMessage = "El número de licencia es obligatorio")]
        [StringLength(50, ErrorMessage = "El número de licencia no puede exceder 50 caracteres")]
        public string NumeroLicencia { get; set; }
        
        [StringLength(20, ErrorMessage = "El teléfono no puede exceder 20 caracteres")]
        public string? Telefono { get; set; }
        
        [StringLength(150, ErrorMessage = "El email no puede exceder 150 caracteres")]
        [EmailAddress(ErrorMessage = "Formato de email inválido")]
        public string? Email { get; set; }
        
        public TimeSpan HorarioInicio { get; set; } = new TimeSpan(8, 0, 0); // 8:00 AM
        public TimeSpan HorarioFin { get; set; } = new TimeSpan(17, 0, 0);   // 5:00 PM
        
        public bool Activo { get; set; } = true;
        
        public DateTime FechaRegistro { get; set; } = DateTime.Now;
        
        // Propiedades de navegación
        public virtual Especialidad Especialidad { get; set; }
        public virtual ICollection<CitaMedica> Citas { get; set; } = new HashSet<CitaMedica>();
        public virtual ICollection<HistorialMedico> ConsultasRealizadas { get; set; } = new HashSet<HistorialMedico>();
        
        // Propiedad calculada
        [NotMapped]
        public string NombreCompleto => $"Dr. {Nombres} {Apellidos}";
    }
}

// Models/Especialidad.cs
namespace ClinicApp.Models
{
    public class Especialidad
    {
        public int Id { get; set; }
        
        [Required(ErrorMessage = "El nombre de la especialidad es obligatorio")]
        [StringLength(100, ErrorMessage = "El nombre no puede exceder 100 caracteres")]
        public string Nombre { get; set; }
        
        [StringLength(500, ErrorMessage = "La descripción no puede exceder 500 caracteres")]
        public string? Descripcion { get; set; }
        
        public bool Activa { get; set; } = true;
        
        // Propiedades de navegación
        public virtual ICollection<Medico> Medicos { get; set; } = new HashSet<Medico>();
    }
}

// Models/CitaMedica.cs
namespace ClinicApp.Models
{
    public class CitaMedica
    {
        public int Id { get; set; }
        
        [Required]
        public int PacienteId { get; set; }
        
        [Required]
        public int MedicoId { get; set; }
        
        [Required]
        public DateTime FechaHora { get; set; }
        
        [StringLength(500, ErrorMessage = "El motivo no puede exceder 500 caracteres")]
        public string? Motivo { get; set; }
        
        [Required]
        [StringLength(20, ErrorMessage = "El estado no puede exceder 20 caracteres")]
        public string Estado { get; set; } = "Programada"; // Programada, En Curso, Completada, Cancelada
        
        [StringLength(1000, ErrorMessage = "Las observaciones no pueden exceder 1000 caracteres")]
        public string? Observaciones { get; set; }
        
        public DateTime FechaCreacion { get; set; } = DateTime.Now;
        
        // Propiedades de navegación
        public virtual Paciente Paciente { get; set; }
        public virtual Medico Medico { get; set; }
    }
}

// Models/HistorialMedico.cs
namespace ClinicApp.Models
{
    public class HistorialMedico
    {
        public int Id { get; set; }
        
        [Required]
        public int PacienteId { get; set; }
        
        [Required]
        public int MedicoId { get; set; }
        
        [Required]
        public DateTime FechaConsulta { get; set; }
        
        [StringLength(500, ErrorMessage = "El motivo de consulta no puede exceder 500 caracteres")]
        public string? MotivoConsulta { get; set; }
        
        [StringLength(2000, ErrorMessage = "El diagnóstico no puede exceder 2000 caracteres")]
        public string? Diagnostico { get; set; }
        
        [StringLength(1000, ErrorMessage = "El tratamiento no puede exceder 1000 caracteres")]
        public string? Tratamiento { get; set; }
        
        [StringLength(500, ErrorMessage = "Los medicamentos no pueden exceder 500 caracteres")]
        public string? Medicamentos { get; set; }
        
        [StringLength(1000, ErrorMessage = "Las observaciones no pueden exceder 1000 caracteres")]
        public string? Observaciones { get; set; }
        
        public DateTime? ProximaCita { get; set; }
        
        // Propiedades de navegación
        public virtual Paciente Paciente { get; set; }
        public virtual Medico Medico { get; set; }
    }
}
```
    
    [Required(ErrorMessage = "El teléfono es obligatorio")]
    [MaxLength(20)]
    public string Telefono { get; set; }
    
    [Required(ErrorMessage = "El email es obligatorio")]
    [MaxLength(150)]
    [EmailAddress]
    public string Email { get; set; }
    
    [MaxLength(250)]
    public string Direccion { get; set; }
    
    [MaxLength(5)]
    public string TipoSangre { get; set; }
    
    [MaxLength(100)]
    public string ContactoEmergencia { get; set; }
    
    [MaxLength(20)]
    public string TelefonoEmergencia { get; set; }
    
    public DateTime FechaRegistro { get; set; } = DateTime.Now;
    
    public bool Activo { get; set; } = true;
    
    // Propiedades de navegación
    public virtual ICollection<CitaMedica> Citas { get; set; } = new List<CitaMedica>();
    public virtual ICollection<HistorialMedico> HistorialMedico { get; set; } = new List<HistorialMedico>();
}
```

#### Paso 3: Crear DbContext para la Clínica
```csharp
// Data/ClinicaContext.cs
using Microsoft.EntityFrameworkCore;
using ClinicApp.Models;

namespace ClinicApp.Data
{
    public class ClinicaContext : DbContext
    {
        public ClinicaContext(DbContextOptions<ClinicaContext> options) : base(options)
        {
        }

        // DbSets - representan las tablas de la base de datos
        public DbSet<Paciente> Pacientes { get; set; }
        public DbSet<Medico> Medicos { get; set; }
        public DbSet<Especialidad> Especialidades { get; set; }
        public DbSet<CitaMedica> CitasMedicas { get; set; }
        public DbSet<HistorialMedico> HistorialesMedicos { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Configuraciones específicas para la clínica
            
            // Configurar tabla Pacientes
            modelBuilder.Entity<Paciente>(entity =>
            {
                entity.ToTable("Pacientes");
                entity.HasKey(e => e.Id);
                entity.HasIndex(e => e.Cedula).IsUnique();
                entity.HasIndex(e => e.Email).IsUnique();
                
                entity.Property(e => e.Cedula)
                    .IsRequired()
                    .HasMaxLength(10)
                    .IsFixedLength();
                    
                entity.Property(e => e.Nombres)
                    .IsRequired()
                    .HasMaxLength(100);
                    
                entity.Property(e => e.Apellidos)
                    .IsRequired()
                    .HasMaxLength(100);
                    
                entity.Property(e => e.Email)
                    .IsRequired()
                    .HasMaxLength(150);
                    
                entity.Property(e => e.Telefono)
                    .IsRequired()
                    .HasMaxLength(20);
                    
                entity.Property(e => e.Direccion)
                    .HasMaxLength(250);
                    
                entity.Property(e => e.TipoSangre)
                    .HasMaxLength(5);
            });

            // Configurar tabla Medicos
            modelBuilder.Entity<Medico>(entity =>
            {
                entity.ToTable("Medicos");
                entity.HasKey(e => e.Id);
                entity.HasIndex(e => e.Cedula).IsUnique();
                entity.HasIndex(e => e.NumeroLicencia).IsUnique();
                
                entity.Property(e => e.Cedula)
                    .IsRequired()
                    .HasMaxLength(10)
                    .IsFixedLength();
                    
                entity.Property(e => e.NumeroLicencia)
                    .IsRequired()
                    .HasMaxLength(50);
                    
                // Relación con Especialidad
                entity.HasOne(e => e.Especialidad)
                    .WithMany(s => s.Medicos)
                    .HasForeignKey(e => e.EspecialidadId)
                    .OnDelete(DeleteBehavior.Restrict);
            });

            // Configurar tabla Especialidades
            modelBuilder.Entity<Especialidad>(entity =>
            {
                entity.ToTable("Especialidades");
                entity.HasKey(e => e.Id);
                entity.HasIndex(e => e.Nombre).IsUnique();
                
                entity.Property(e => e.Nombre)
                    .IsRequired()
                    .HasMaxLength(100);
            });

            // Configurar tabla CitasMedicas
            modelBuilder.Entity<CitaMedica>(entity =>
            {
                entity.ToTable("CitasMedicas");
                entity.HasKey(e => e.Id);
                
                // Relación con Paciente
                entity.HasOne(e => e.Paciente)
                    .WithMany(p => p.Citas)
                    .HasForeignKey(e => e.PacienteId)
                    .OnDelete(DeleteBehavior.Restrict);
                    
                // Relación con Medico
                entity.HasOne(e => e.Medico)
                    .WithMany(m => m.Citas)
                    .HasForeignKey(e => e.MedicoId)
                    .OnDelete(DeleteBehavior.Restrict);
                    
                entity.Property(e => e.Estado)
                    .IsRequired()
                    .HasMaxLength(20)
                    .HasDefaultValue("Programada");
            });

            // Configurar tabla HistorialesMedicos
            modelBuilder.Entity<HistorialMedico>(entity =>
            {
                entity.ToTable("HistorialesMedicos");
                entity.HasKey(e => e.Id);
                
                // Relación con Paciente
                entity.HasOne(e => e.Paciente)
                    .WithMany(p => p.HistorialMedico)
                    .HasForeignKey(e => e.PacienteId)
                    .OnDelete(DeleteBehavior.Restrict);
                    
                // Relación con Medico
                entity.HasOne(e => e.Medico)
                    .WithMany(m => m.ConsultasRealizadas)
                    .HasForeignKey(e => e.MedicoId)
                    .OnDelete(DeleteBehavior.Restrict);
            });

            // Datos semilla (seed data) para especialidades
            modelBuilder.Entity<Especialidad>().HasData(
                new Especialidad { Id = 1, Nombre = "Medicina General", Descripcion = "Atención médica general y preventiva", Activa = true },
                new Especialidad { Id = 2, Nombre = "Cardiología", Descripcion = "Especialidad en enfermedades del corazón", Activa = true },
                new Especialidad { Id = 3, Nombre = "Pediatría", Descripcion = "Atención médica para niños y adolescentes", Activa = true },
                new Especialidad { Id = 4, Nombre = "Ginecología", Descripcion = "Atención médica para la mujer", Activa = true },
                new Especialidad { Id = 5, Nombre = "Dermatología", Descripcion = "Especialidad en enfermedades de la piel", Activa = true },
                new Especialidad { Id = 6, Nombre = "Traumatología", Descripcion = "Especialidad en lesiones del sistema músculo-esquelético", Activa = true }
            );
        }
    }
}
```

#### Paso 4: Configurar la Conexión en Program.cs
```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using ClinicApp.Data;

var builder = WebApplication.CreateBuilder(args);

// Agregar servicios
builder.Services.AddRazorPages();
builder.Services.AddControllers();

// 🔧 CONFIGURAR ENTITY FRAMEWORK PARA LA CLÍNICA
builder.Services.AddDbContext<ClinicaContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configurar pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapRazorPages();
app.MapControllers();

app.Run();
```

#### Paso 5: Configurar Cadena de Conexión
```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ClinicaDB;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

#### Paso 6: Crear Primera Migración en Visual Studio 2022
```
🔧 EN VISUAL STUDIO 2022:

1. Ir al menú "Tools" → "NuGet Package Manager" → "Package Manager Console"
2. Asegúrate de que el proyecto correcto esté seleccionado en "Default project"
3. Ejecutar estos comandos:

📝 COMANDO 1: Crear la migración inicial
Add-Migration InicialClinica

📝 COMANDO 2: Aplicar la migración a la base de datos
Update-Database

⚠️ Si tienes errores, verifica:
- Que los paquetes NuGet estén instalados correctamente
- Que la cadena de conexión sea correcta
- Que SQL Server LocalDB esté funcionando
#### Paso 7: Verificar que todo funciona - Crear Servicio de Prueba
```csharp
// Services/PacienteService.cs
using Microsoft.EntityFrameworkCore;
using ClinicApp.Data;
using ClinicApp.Models;

namespace ClinicApp.Services
{
    public class PacienteService
    {
        private readonly ClinicaContext _context;

        public PacienteService(ClinicaContext context)
        {
            _context = context;
        }

        public async Task<List<Paciente>> ObtenerTodosPacientesAsync()
        {
            return await _context.Pacientes
                .Where(p => p.Activo)
                .OrderBy(p => p.Apellidos)
                .ThenBy(p => p.Nombres)
                .ToListAsync();
        }

        public async Task<Paciente?> ObtenerPacientePorIdAsync(int id)
        {
            return await _context.Pacientes
                .Include(p => p.Citas)
                .ThenInclude(c => c.Medico)
                .ThenInclude(m => m.Especialidad)
                .FirstOrDefaultAsync(p => p.Id == id && p.Activo);
        }

        public async Task<Paciente> CrearPacienteAsync(Paciente paciente)
        {
            _context.Pacientes.Add(paciente);
            await _context.SaveChangesAsync();
            return paciente;
        }

        public async Task<bool> ExistePacienteConCedulaAsync(string cedula)
        {
            return await _context.Pacientes
                .AnyAsync(p => p.Cedula == cedula);
        }

        public async Task<bool> ExistePacienteConEmailAsync(string email)
        {
            return await _context.Pacientes
                .AnyAsync(p => p.Email == email);
        }

        public async Task<List<Paciente>> BuscarPacientesAsync(string termino)
        {
            return await _context.Pacientes
                .Where(p => p.Activo && 
                    (p.Nombres.Contains(termino) || 
                     p.Apellidos.Contains(termino) || 
                     p.Cedula.Contains(termino)))
                .OrderBy(p => p.Apellidos)
                .ToListAsync();
        }
    }
}
```

#### Paso 8: Registrar el Servicio en Program.cs
```csharp
// Program.cs - Agregar después de AddDbContext
builder.Services.AddDbContext<ClinicaContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// 📝 REGISTRAR SERVICIOS DE LA CLÍNICA
builder.Services.AddScoped<PacienteService>();
```

### Puntos Clave a Explicar:
- ✅ **NuGet en Visual Studio 2022**: Cómo instalar paquetes correctamente
- ✅ **Modelos con namespaces**: Organización correcta del código
- ✅ **Validaciones Data Annotations**: [Required], [StringLength], etc.
- ✅ **Propiedades calculadas**: [NotMapped] para campos que no van a BD
- ✅ **Relaciones EF Core**: Cómo configurar Foreign Keys
- ✅ **Package Manager Console**: Comandos Add-Migration y Update-Database
- ✅ **LocalDB**: Base de datos local para desarrollo
- ✅ **Datos semilla**: HasData() para datos iniciales

## 🏠 Desafío Autónomo (Casa)

## 🏠 **Desafío Autónomo (Para Casa)**

### 📋 **Tarea: "Expandir el Sistema de Clínica"**

#### 🎯 **Objetivo:**
Agregar nuevas entidades al sistema de clínica para completar la funcionalidad básica.

#### 📊 **Entidades a Crear:**

**1. Entidad Sala:**
```csharp
// Models/Sala.cs
public class Sala
{
    public int Id { get; set; }
    
**1. Entidad Sala:**
```csharp
// Models/Sala.cs
namespace ClinicApp.Models
{
    public class Sala
    {
        public int Id { get; set; }
        
        [Required]
        [StringLength(10)]
        public string Numero { get; set; }        // Ej: "CONS-01"
        
        [Required]
        [StringLength(50)]
        public string Tipo { get; set; }          // Consultorio, Urgencias, Quirófano
        
        [StringLength(200)]
        public string? Equipamiento { get; set; }  // Lista de equipos disponibles
        
        public bool Disponible { get; set; } = true;
        public bool Activa { get; set; } = true;
        
        // Relación: Una sala puede tener muchas citas
        public virtual ICollection<CitaMedica> Citas { get; set; } = new HashSet<CitaMedica>();
    }
}
```

**2. Entidad Medicamento:**
**2. Entidad Medicamento:**
```csharp
// Models/Medicamento.cs
namespace ClinicApp.Models
{
    public class Medicamento
    {
        public int Id { get; set; }
        
        [Required]
        [StringLength(100)]
        public string Nombre { get; set; }
        
        [StringLength(200)]
        public string? Descripcion { get; set; }
        
        [Required]
        [StringLength(50)]
        public string Concentracion { get; set; }  // Ej: "500mg"
        
        [Required]
        [StringLength(50)]
        public string Forma { get; set; }          // Tableta, Jarabe, Inyección
        
        public int Stock { get; set; } = 0;
        public int StockMinimo { get; set; } = 10;
        
        [Column(TypeName = "decimal(10,2)")]
        public decimal Precio { get; set; }
        
        public DateTime? FechaVencimiento { get; set; }
        public bool Activo { get; set; } = true;
    }
}
```

#### 📋 **Entregables:**
1. **Modelos completos** con validaciones y relaciones
2. **Configuración en DbContext** con Fluent API  
3. **Migración aplicada** correctamente
4. **Servicios CRUD** para Sala y Medicamento
5. **Screenshots** de la base de datos creada
6. **Código de prueba** funcional

#### ⭐ **Criterios de Evaluación:**
- ✅ **Modelos bien diseñados** (30%)
- ✅ **Relaciones correctas** (25%)
- ✅ **Migración exitosa** (20%)
- ✅ **Servicios funcionando** (15%)
- ✅ **Código limpio** (10%)

---
[CONTROLLERS] ← (Próximo módulo - APIs REST)
```

---

### 📋 **Guía Rápida: Database First vs Trabajo Existente**

#### 🚨 **Para Profesores: Manejo de la Situación Actual**

Si tus estudiantes ya avanzaron con Code First y controladores manuales, esta es la estrategia recomendada:

**✅ OPCIÓN A - RECOMENDADA (Clase Nueva):**
```sql
-- 1. Crear nueva base de datos completa
-- 2. Hacer scaffolding completo
-- 3. Continuar solo con Database First
```

**⚠️ OPCIÓN B - HÍBRIDA (Preservar Avance):**
```powershell
# 1. Respaldar trabajo existente
# 2. Scaffolding solo de modelos
# 3. Adaptar controladores existentes
Scaffold-DbContext "..." -OutputDir Models -NoOnConfiguring
```

#### 🎯 **Ventajas del Cambio a Database First:**
- ⚡ **50% menos tiempo** en crear CRUD completo
- 🎯 **Consistencia garantizada** en relaciones
- 🏢 **Enfoque empresarial real** (BD → Código)
- 🧪 **Fácil testing** con datos reales

#### 💡 **Consejos para la Transición:**
1. **Mostrar ambos enfoques** (Code First vs Database First)
2. **Explicar cuándo usar cada uno** en proyectos reales
3. **Database First = Rapidez en prototipado**
4. **Code First = Control total del esquema**

---

**Entregables:**
- Código completo de los modelos
- Diagram ER de las entidades y relaciones
- Screenshots de la base de datos creada
- Servicios CRUD para al menos 2 entidades
- Documento con decisiones de diseño

## 🚀 Avance en el Proyecto Final

### Implementación de la Capa de Datos
Integrar los modelos creados en el desafío con tu proyecto existente.

### Tareas Específicas:
1. **Instalar EF Core** y configurar la conexión
2. **Migrar los modelos** del desafío a tu proyecto
3. **Crear el DbContext** específico para tu dominio
4. **Aplicar migraciones** y verificar la base de datos
5. **Implementar servicios CRUD** para las entidades principales
6. **Actualizar el dashboard** para usar datos reales (aunque sean pocos)

### Integración con Pages Existentes:
- Modificar las Razor Pages para usar los servicios de datos
- Reemplazar datos simulados con consultas a la base de datos
- Implementar formularios funcionales que guarden datos reales

### Estructura Esperada:
```
MiProyectoFinal/
├── Data/
│   └── MiDominioContext.cs
├── Models/
│   ├── Entidad1.cs
│   ├── Entidad2.cs
│   └── ...
├── Services/
│   ├── Entidad1Service.cs
│   ├── Entidad2Service.cs
│   └── ...
├── Migrations/
│   └── (archivos de migración)
└── ...
```

---

### � **Guía Rápida: Database First vs Trabajo Existente**

#### 🚨 **Para Profesores: Manejo de la Situación Actual**

Si tus estudiantes ya avanzaron con Code First y controladores manuales, esta es la estrategia recomendada:

**✅ OPCIÓN A - RECOMENDADA (Clase Nueva):**
```sql
-- 1. Crear nueva base de datos completa
-- 2. Hacer scaffolding completo
-- 3. Continuar solo con Database First
```

**⚠️ OPCIÓN B - HÍBRIDA (Preservar Avance):**
```powershell
# 1. Respaldar trabajo existente
# 2. Scaffolding solo de modelos
# 3. Adaptar controladores existentes
Scaffold-DbContext "..." -OutputDir Models -NoOnConfiguring
```

#### 🎯 **Ventajas del Cambio a Database First:**
- ⚡ **50% menos tiempo** en crear CRUD completo
- 🎯 **Consistencia garantizada** en relaciones
- 🏢 **Enfoque empresarial real** (BD → Código)
- 🧪 **Fácil testing** con datos reales

#### 💡 **Consejos para la Transición:**
1. **Mostrar ambos enfoques** (Code First vs Database First)
2. **Explicar cuándo usar cada uno** en proyectos reales
3. **Database First = Rapidez en prototipado**
4. **Code First = Control total del esquema**

---

## 📝 **Tareas para Casa**

### 🎯 **Tarea Individual (2 horas)**
1. **Expandir la base de datos:**
   ```sql
   -- Agregar tabla de Medicamentos
   CREATE TABLE Medicamentos (
       Id int IDENTITY(1,1) PRIMARY KEY,
       Nombre nvarchar(200) NOT NULL,
       PrincipioActivo nvarchar(200) NOT NULL,
       Concentracion nvarchar(50) NULL,
       FormaFarmaceutica nvarchar(50) NOT NULL,
       Laboratorio nvarchar(100) NULL,
       Precio decimal(10,2) NULL,
       Stock int NOT NULL DEFAULT 0,
       FechaVencimiento date NULL,
       RequiereReceta bit NOT NULL DEFAULT 0,
       Activo bit NOT NULL DEFAULT 1
   );
   ```

2. **Hacer scaffolding actualizado:**
   ```powershell
   # Actualizar modelos con nueva tabla
   Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=ClinicAppDB;Trusted_Connection=true" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Context ClinicaContext -DataAnnotations -Force
   ```

3. **Crear CRUD para Medicamentos**

### 🎯 **Investigación Adicional**
- Diferencias entre **Eager Loading** vs **Lazy Loading**
- **Índices de base de datos** para optimización
- **Stored Procedures** vs **Consultas EF**

---

## 📚 **Recursos Adicionales**

### 🔗 **Enlaces Esenciales**
- [EF Core Database First](https://docs.microsoft.com/en-us/ef/core/managing-schemas/scaffolding?tabs=dotnet-core-cli)
- [Scaffolding en Visual Studio 2022](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/adding-model)
- [SQL Server LocalDB Setup](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/sql-server-express-localdb)

### 📖 **Lectura Complementaria**
- "Entity Framework Core Database First vs Code First"
- "Diseño de BD para sistemas médicos - Buenas prácticas"
- "Scaffolding y productividad en ASP.NET Core"

---

## 🎯 **Evaluación del Módulo**

### ✅ **Criterios de Evaluación**
1. **Diseño de Base de Datos (30%)**
   - Estructura apropiada de tablas
   - Relaciones correctas (FK, PK)
   - Datos de prueba coherentes

2. **Implementación con EF Core (35%)**
   - Scaffolding ejecutado correctamente
   - DbContext configurado apropiadamente
   - Modelos generados sin errores

3. **Funcionalidad CRUD (35%)**
   - Operaciones básicas funcionando
   - Navegación entre entidades
   - Validaciones activas

### 📊 **Rúbrica de Calificación**
- **Excelente (9-10)**: Sistema completo con Database First y funcionalidad avanzada
- **Bueno (7-8)**: CRUD básico funcionando correctamente con scaffolding
- **Regular (5-6)**: Funcionalidad parcial con errores menores
- **Insuficiente (0-4)**: No logra implementar el enfoque Database First

---

> **💡 Nota Importante:** El enfoque Database First mostrado aquí es especialmente útil cuando se trabaja con esquemas de BD existentes o cuando se prioriza la rapidez en el desarrollo de prototipos funcionales.

## 📝 Material de Apoyo
- [EF Core Documentation](https://docs.microsoft.com/en-us/ef/core/)
- [Data Annotations Guide](./data-annotations.md)
- [Fluent API Examples](./fluent-api-examples.md)
- [Migration Commands Reference](./migration-commands.md)

## ✅ Checklist de Completado
- [ ] Base de datos ClinicAppDB creada con todas las tablas
- [ ] Entity Framework Core instalado y configurado para Database First
- [ ] Scaffolding ejecutado correctamente (modelos + contexto)
- [ ] Controladores y vistas generados automáticamente
- [ ] Aplicación funcionando con CRUD completo
- [ ] Tarea individual completada (tabla Medicamentos)
- [ ] Comprensión de Database First vs Code First

---
**Anterior:** [03 - Dashboard con Razor Pages](../03-dashboard-razor/)  
**Siguiente:** [05 - API REST y Controladores](../05-api-rest-controladores/)
