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

**Atributos de validación:**
```csharp
public class Usuario
{
    public int Id { get; set; }
    
    [Required(ErrorMessage = "El nombre es obligatorio")]
    [MaxLength(100, ErrorMessage = "Máximo 100 caracteres")]
    public string Nombre { get; set; }
    
    [EmailAddress(ErrorMessage = "Email inválido")]
    public string Email { get; set; }
    
    [Range(18, 120, ErrorMessage = "Edad entre 18 y 120 años")]
    public int Edad { get; set; }
}
```

**Tipos de relaciones:**
```csharp
// 1. Uno a Muchos (Un autor tiene muchos libros)
public class Autor
{
    public int Id { get; set; }
    public string Nombre { get; set; }
    public virtual ICollection<Libro> Libros { get; set; } = new List<Libro>();
}

public class Libro
{
    public int Id { get; set; }
    public string Titulo { get; set; }
    public int AutorId { get; set; }  // Llave foránea
    public virtual Autor Autor { get; set; }  // Navegación
}

// 2. Muchos a Muchos (Estudiantes y Cursos)
public class Estudiante
{
    public int Id { get; set; }
    public string Nombre { get; set; }
    public virtual ICollection<Curso> Cursos { get; set; } = new List<Curso>();
}

public class Curso
{
    public int Id { get; set; }
    public string Nombre { get; set; }
    public virtual ICollection<Estudiante> Estudiantes { get; set; } = new List<Estudiante>();
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
            
        // Datos iniciales (seed data)
        modelBuilder.Entity<Categoria>().HasData(
            new Categoria { Id = 1, Nombre = "Electrónicos" },
            new Categoria { Id = 2, Nombre = "Ropa" }
        );
    }
}
```

### 4. Migraciones
**¿Qué son las migraciones?**
Las migraciones son como "instrucciones" que le dicen a la base de datos cómo cambiar su estructura (agregar tablas, columnas, etc.).

**Analogía:**
Es como una "receta" que le dice al chef (base de datos) cómo preparar el plato (estructura).

**Comandos básicos:**
```bash
# 1. Crear una migración (después de cambiar modelos)
dotnet ef migrations add NombreDeLaMigracion

# 2. Aplicar cambios a la base de datos
dotnet ef database update

# 3. Ver el estado actual
dotnet ef migrations list

# 4. Volver a una migración anterior (rollback)
dotnet ef database update NombreMigracionAnterior

# 5. Generar script SQL (para producción)
dotnet ef migrations script
```

**Flujo típico de trabajo:**
1. **Modificas modelos** → Agregas nueva propiedad a clase
2. **Creas migración** → `dotnet ef migrations add AgregadaNuevaColumna`
3. **Revisas la migración** → EF generó el código SQL automáticamente
4. **Aplicas cambios** → `dotnet ef database update`
5. **¡Listo!** → Tu base de datos tiene la nueva columna

**Ejemplo de archivo de migración generado:**
```csharp
public partial class AgregadaColumnaEmail : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "Email",
            table: "Usuarios",
            type: "nvarchar(max)",
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

### Ejercicio Guiado: "Sistema de Clínica - Modelos y Base de Datos"

#### Paso 1: Instalar Entity Framework Core en Visual Studio 2022
```xml
<!-- En el archivo .csproj del proyecto SistemaClinica -->
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />
```

#### Paso 2: Crear Modelos de Entidad del Sistema de Clínica
```csharp
// Models/Paciente.cs
using System.ComponentModel.DataAnnotations;

public class Paciente
{
    public int Id { get; set; }
    
    [Required(ErrorMessage = "La cédula es obligatoria")]
    [MaxLength(10)]
    public string Cedula { get; set; }
    
    [Required(ErrorMessage = "Los nombres son obligatorios")]
    [MaxLength(100)]
    public string Nombres { get; set; }
    
    [Required(ErrorMessage = "Los apellidos son obligatorios")]
    [MaxLength(100)]
    public string Apellidos { get; set; }
    
    [Required(ErrorMessage = "La fecha de nacimiento es obligatoria")]
    public DateTime FechaNacimiento { get; set; }
    
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

// Models/Medico.cs
public class Medico
{
    public int Id { get; set; }
    
    [Required(ErrorMessage = "La cédula es obligatoria")]
    [MaxLength(10)]
    public string Cedula { get; set; }
    
    [Required(ErrorMessage = "Los nombres son obligatorios")]
    [MaxLength(100)]
    public string Nombres { get; set; }
    
    [Required(ErrorMessage = "Los apellidos son obligatorios")]
    [MaxLength(100)]
    public string Apellidos { get; set; }
    
    [Required(ErrorMessage = "La especialidad es obligatoria")]
    [MaxLength(100)]
    public string Especialidad { get; set; }
    
    [Required(ErrorMessage = "El número de licencia es obligatorio")]
    [MaxLength(50)]
    public string NumeroLicencia { get; set; }
    
    [MaxLength(20)]
    public string Telefono { get; set; }
    
    [MaxLength(150)]
    [EmailAddress]
    public string Email { get; set; }
    
    public TimeSpan HorarioInicio { get; set; }
    public TimeSpan HorarioFin { get; set; }
    
    public bool Activo { get; set; } = true;
    
    public DateTime FechaRegistro { get; set; } = DateTime.Now;
    
    // Propiedades de navegación
    public virtual ICollection<CitaMedica> Citas { get; set; } = new List<CitaMedica>();
    public virtual ICollection<HistorialMedico> ConsultasRealizadas { get; set; } = new List<HistorialMedico>();
}

// Models/CitaMedica.cs
public class CitaMedica
{
    public int Id { get; set; }
    
    [Required]
    public int PacienteId { get; set; }
    
    [Required]
    public int MedicoId { get; set; }
    
    [Required]
    public DateTime FechaHora { get; set; }
    
    [MaxLength(500)]
    public string Motivo { get; set; }
    
    [Required]
    [MaxLength(20)]
    public string Estado { get; set; } = "Programada"; // Programada, En Curso, Completada, Cancelada
    
    [MaxLength(1000)]
    public string Observaciones { get; set; }
    
    public DateTime FechaCreacion { get; set; } = DateTime.Now;
    
    // Propiedades de navegación
    public virtual Paciente Paciente { get; set; }
    public virtual Medico Medico { get; set; }
}

// Models/HistorialMedico.cs
public class HistorialMedico
{
    public int Id { get; set; }
    
    [Required]
    public int PacienteId { get; set; }
    
    [Required]
    public int MedicoId { get; set; }
    
    [Required]
    public DateTime FechaConsulta { get; set; }
    
    [MaxLength(500)]
    public string MotivoConsulta { get; set; }
    
    [MaxLength(2000)]
    public string Diagnostico { get; set; }
    
    [MaxLength(1000)]
    public string Tratamiento { get; set; }
    
    [MaxLength(500)]
    public string Medicamentos { get; set; }
    
    [MaxLength(1000)]
    public string Observaciones { get; set; }
    
    public DateTime ProximaCita { get; set; }
    
    // Propiedades de navegación
    public virtual Paciente Paciente { get; set; }
    public virtual Medico Medico { get; set; }
}

// Models/Especialidad.cs
public class Especialidad
{
    public int Id { get; set; }
    
    [Required]
    [MaxLength(100)]
    public string Nombre { get; set; }
    
    [MaxLength(500)]
    public string Descripcion { get; set; }
    
    public bool Activa { get; set; } = true;
    
    // Propiedades de navegación
    public virtual ICollection<Medico> Medicos { get; set; } = new List<Medico>();
}
```
    
    [Required]
    [MaxLength(100)]
    public string Apellido { get; set; }
    
    [Required]
    [EmailAddress]
    [MaxLength(150)]
    public string Email { get; set; }
    
    [MaxLength(15)]
    public string Telefono { get; set; }
    
    public DateTime FechaNacimiento { get; set; }
    
    public DateTime FechaRegistro { get; set; } = DateTime.Now;
    
    public bool Activo { get; set; } = true;
    
    // Relación con préstamos
    public virtual ICollection<Prestamo> Prestamos { get; set; } = new List<Prestamo>();
}

// Models/Prestamo.cs
public class Prestamo
{
    public int Id { get; set; }
    
    public int LibroId { get; set; }
    public virtual Libro Libro { get; set; }
    
    public int UsuarioId { get; set; }
    public virtual Usuario Usuario { get; set; }
    
    public DateTime FechaPrestamo { get; set; } = DateTime.Now;
    
    public DateTime FechaDevolucionPrevista { get; set; }
    
    public DateTime? FechaDevolucionReal { get; set; }
    
    public bool Devuelto { get; set; } = false;
    
    [MaxLength(500)]
    public string Observaciones { get; set; }
}
```

#### Paso 3: Crear DbContext
```csharp
// Data/BibliotecaContext.cs
using Microsoft.EntityFrameworkCore;

public class BibliotecaContext : DbContext
{
    public BibliotecaContext(DbContextOptions<BibliotecaContext> options) : base(options)
    {
    }
    
    public DbSet<Libro> Libros { get; set; }
    public DbSet<Usuario> Usuarios { get; set; }
    public DbSet<Prestamo> Prestamos { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configuración de relaciones
        modelBuilder.Entity<Prestamo>()
            .HasOne(p => p.Libro)
            .WithMany(l => l.Prestamos)
            .HasForeignKey(p => p.LibroId)
            .OnDelete(DeleteBehavior.Restrict);
            
        modelBuilder.Entity<Prestamo>()
            .HasOne(p => p.Usuario)
            .WithMany(u => u.Prestamos)
            .HasForeignKey(p => p.UsuarioId)
            .OnDelete(DeleteBehavior.Restrict);
            
        // Índices únicos
        modelBuilder.Entity<Libro>()
            .HasIndex(l => l.ISBN)
            .IsUnique();
            
        modelBuilder.Entity<Usuario>()
            .HasIndex(u => u.Email)
            .IsUnique();
            
        // Datos semilla
        modelBuilder.Entity<Libro>().HasData(
            new Libro 
            { 
                Id = 1, 
                Titulo = "Don Quijote de la Mancha", 
                Autor = "Miguel de Cervantes",
                ISBN = "9788490011485",
                FechaPublicacion = new DateTime(1605, 1, 16),
                Genero = "Novela",
                NumeroPaginas = 863,
                FechaRegistro = DateTime.Now
            },
            new Libro 
            { 
                Id = 2, 
                Titulo = "Cien años de soledad", 
                Autor = "Gabriel García Márquez",
                ISBN = "9788437604947",
                FechaPublicacion = new DateTime(1967, 5, 30),
                Genero = "Realismo mágico",
                NumeroPaginas = 471,
                FechaRegistro = DateTime.Now
            }
        );
    }
}
```

#### Paso 4: Configurar en Program.cs
```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Agregar servicios
builder.Services.AddRazorPages();

// Configurar Entity Framework
builder.Services.AddDbContext<BibliotecaContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configurar pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
}

app.UseStaticFiles();
app.UseRouting();
app.MapRazorPages();

app.Run();
```

#### Paso 5: Configurar Connection String
```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=BibliotecaDB;Trusted_Connection=true;MultipleActiveResultSets=true"
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

#### Paso 6: Crear y Aplicar Migración
```bash
# Crear primera migración
dotnet ef migrations add InitialCreate

# Actualizar base de datos
dotnet ef database update
```

#### Paso 7: Implementar Operaciones CRUD Básicas
```csharp
// Services/LibroService.cs
public class LibroService
{
    private readonly BibliotecaContext _context;
    
    public LibroService(BibliotecaContext context)
    {
        _context = context;
    }
    
    public async Task<List<Libro>> ObtenerTodosAsync()
    {
        return await _context.Libros.OrderBy(l => l.Titulo).ToListAsync();
    }
    
    public async Task<Libro?> ObtenerPorIdAsync(int id)
    {
        return await _context.Libros.FindAsync(id);
    }
    
    public async Task<Libro> CrearAsync(Libro libro)
    {
        _context.Libros.Add(libro);
        await _context.SaveChangesAsync();
        return libro;
    }
    
    public async Task<bool> ActualizarAsync(Libro libro)
    {
        _context.Libros.Update(libro);
        return await _context.SaveChangesAsync() > 0;
    }
    
    public async Task<bool> EliminarAsync(int id)
    {
        var libro = await _context.Libros.FindAsync(id);
        if (libro == null) return false;
        
        _context.Libros.Remove(libro);
        return await _context.SaveChangesAsync() > 0;
    }
    
    public async Task<List<Libro>> BuscarAsync(string termino)
    {
        return await _context.Libros
            .Where(l => l.Titulo.Contains(termino) || l.Autor.Contains(termino))
            .ToListAsync();
    }
}
```

### Puntos Clave a Explicar:
- Convenciones de EF Core (Id como PK, navegación automática)
- Diferencia entre virtual y no virtual en propiedades de navegación
- OnDelete behaviors y su importancia
- Datos semilla para testing

## 🏠 Desafío Autónomo (Casa)

### Tarea: "Modelado Completo de tu Dominio"
Implementar el modelo de datos completo para tu proyecto elegido.

#### Para Sistema de Clínica:
**Entidades requeridas:**
1. **Paciente**: Id, Nombre, Apellido, Documento, FechaNacimiento, Telefono, Email, Direccion, Seguro
2. **Medico**: Id, Nombre, Apellido, Especialidad, NumeroLicencia, Telefono, Email, HorarioDisponible
3. **Cita**: Id, PacienteId, MedicoId, FechaHora, TipoConsulta, Estado, Observaciones, Costo
4. **HistorialMedico**: Id, PacienteId, MedicoId, Fecha, Diagnostico, Tratamiento, Observaciones

#### Para Sistema POS:
**Entidades requeridas:**
1. **Producto**: Id, Nombre, Descripcion, Precio, Stock, Categoria, CodigoBarras, FechaVencimiento
2. **Cliente**: Id, Nombre, Apellido, Documento, Telefono, Email, Direccion, FechaRegistro
3. **Venta**: Id, ClienteId, Fecha, Total, Descuento, MetodoPago, Estado
4. **DetalleVenta**: Id, VentaId, ProductoId, Cantidad, PrecioUnitario, Subtotal

#### Para Sistema de Reservas:
**Entidades requeridas:**
1. **Usuario**: Id, Nombre, Apellido, Email, Telefono, TipoUsuario, FechaRegistro
2. **Espacio**: Id, Nombre, Descripcion, Capacidad, Ubicacion, TarifaHora, Disponible
3. **Reserva**: Id, UsuarioId, EspacioId, FechaInicio, FechaFin, TotalHoras, Costo, Estado
4. **Pago**: Id, ReservaId, Monto, MetodoPago, FechaPago, Estado

**Requisitos Técnicos:**
- ✅ Todas las entidades con validaciones apropiadas
- ✅ Relaciones correctamente configuradas
- ✅ Índices únicos donde corresponda
- ✅ Datos semilla para testing
- ✅ Al menos un servicio CRUD implementado
- ✅ Migraciones creadas y aplicadas

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

## 📝 Material de Apoyo
- [EF Core Documentation](https://docs.microsoft.com/en-us/ef/core/)
- [Data Annotations Guide](./data-annotations.md)
- [Fluent API Examples](./fluent-api-examples.md)
- [Migration Commands Reference](./migration-commands.md)

## ✅ Checklist de Completado
- [ ] Entity Framework Core instalado y configurado
- [ ] Modelos de entidad creados con validaciones
- [ ] DbContext implementado con relaciones
- [ ] Migraciones creadas y aplicadas exitosamente
- [ ] Servicios CRUD básicos funcionando
- [ ] Desafío autónomo completado
- [ ] Proyecto final integrado con base de datos

---
**Anterior:** [03 - Dashboard con Razor Pages](../03-dashboard-razor/)  
**Siguiente:** [05 - API REST y Controladores](../05-api-rest-controladores/)
