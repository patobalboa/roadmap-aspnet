# 01 - Introducción a ASP.NET Core

## 🎯 Objetivo de Aprendizaje
Al finalizar esta sección, el estudiante será capaz de:
- Entender qué es ASP.NET Core y sus ventajas
- Crear un proyecto básico de ASP.NET Core
- Configurar el entorno de desarrollo
- Ejecutar y debuggear una aplicación simple

## 📚 Contenidos Principales

### 1. ¿Qué es ASP.NET Core?
**Definición simple:**
ASP.NET Core es un framework (conjunto de herramientas) que te permite crear aplicaciones web modernas usando el lenguaje C#.

**Analogía:** 
Piensa en ASP.NET Core como una "caja de herramientas" para construir sitios web, similar a como WordPress te ayuda a crear blogs, pero mucho más poderoso y flexible.

```csharp
// Ejemplo: Una aplicación web mínima
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "¡Hola Mundo!");
app.Run();
```

**Ventajas principales:**
- **Multiplataforma**: Funciona en Windows, Mac y Linux
- **Rápido**: Optimizado para alta performance
- **Moderno**: Siempre actualizado con las últimas tecnologías
- **Gratuito**: No pagas licencias por usarlo

**Diferencias con ASP.NET Framework (el anterior):**
| ASP.NET Framework | ASP.NET Core |
|-------------------|--------------|
| Solo Windows | Windows, Mac, Linux |
| Pesado | Ligero y rápido |
| Cerrado | Open source |
| Versiones lentas | Actualizaciones frecuentes |

### 2. Configuración del Entorno
**¿Qué necesitas instalar?**
1. **.NET 8 SDK** - Las herramientas para programar
2. **Visual Studio** o **VS Code** - El editor donde escribes código
3. **Git** (opcional) - Para guardar versiones de tu código

**Verificar instalación:**
```bash
# Abrir terminal/consola y escribir:
dotnet --version
# Debería mostrar: 8.0.x
```

**Extensiones recomendadas para VS Code:**
- C# for Visual Studio Code
- C# Extensions
- Auto Rename Tag
- Bracket Pair Colorizer

### 3. Estructura de un Proyecto ASP.NET Core
**Archivos importantes y qué hacen:**

```
MiProyecto/
├── Program.cs          # 🚀 El "motor" de tu aplicación
├── appsettings.json    # ⚙️ Configuración (como BD, APIs)
├── MiProyecto.csproj   # 📦 Lista de dependencias
├── Controllers/        # 🎮 Lógica de la aplicación
├── Views/             # 👁️ Lo que ve el usuario (HTML)
├── Models/            # 📊 Estructura de datos
└── wwwroot/           # 🌐 Archivos públicos (CSS, JS, imágenes)
```

**Program.cs explicado:**
```csharp
// 1. Crear el "constructor" de la aplicación
var builder = WebApplication.CreateBuilder(args);

// 2. Agregar servicios (como piezas LEGO)
builder.Services.AddControllers();        // Para APIs
builder.Services.AddRazorPages();        // Para páginas web

// 3. "Construir" la aplicación
var app = builder.Build();

// 4. Configurar cómo responde a peticiones
app.UseStaticFiles();    // Servir CSS, JS, imágenes
app.UseRouting();        // Decidir qué página mostrar
app.MapControllers();    // Conectar URLs con código

// 5. Iniciar la aplicación
app.Run();
```

### 4. Primer "Hola Mundo"
**Conceptos básicos:**

**¿Qué es el ruteo?**
El ruteo decide qué código ejecutar según la URL que visite el usuario.

```csharp
// Ejemplos de rutas
app.MapGet("/", () => "Página principal");                    // www.miweb.com/
app.MapGet("/sobre-mi", () => "Acerca de mí");               // www.miweb.com/sobre-mi
app.MapGet("/saludo/{nombre}", (string nombre) =>            // www.miweb.com/saludo/juan
    $"¡Hola {nombre}!");
```

**¿Qué es el middleware?**
Son "filtros" que procesan las peticiones web en orden.

```csharp
// Ejemplo de middleware personalizado
app.Use(async (context, next) => 
{
    Console.WriteLine($"Visitando: {context.Request.Path}");  // Log
    await next();  // Continuar al siguiente middleware
});
```

**Orden típico de middleware:**
1. **Manejo de errores** - Captura problemas
2. **Archivos estáticos** - CSS, JS, imágenes
3. **Ruteo** - Decide qué ejecutar
4. **Autenticación** - ¿Usuario válido?
5. **Tu código** - La lógica de tu app

## 🔧 Actividad Práctica en Clase (2 horas)

## 🔧 Actividad Práctica en Clase (2 horas)

### Ejercicio Guiado: "Sistema de Clínica Médica - Fundamentos"

#### Paso 1: Crear proyecto MVC en Visual Studio 2022
```
1. Abrir Visual Studio Community 2022
2. Crear nuevo proyecto → "Aplicación web de ASP.NET Core"
3. Nombre: "SistemaClinicaMVC"
4. Plantilla: "Aplicación web de ASP.NET Core (Modelo-Vista-Controlador)"
5. Framework: .NET 8.0
6. Authentication Type: None (por ahora)
7. Configure for HTTPS: ✓
8. Enable Docker: ❌
9. Do not use top-level statements: ❌
```

#### Paso 2: Explorar la estructura MVC generada automáticamente
```
SistemaClinicaMVC/
├── Controllers/
│   └── HomeController.cs          # 🎮 Controlador principal (ya creado)
├── Models/
│   └── ErrorViewModel.cs          # 📊 Modelo de error (ya creado)
├── Views/
│   ├── Home/
│   │   ├── Index.cshtml          # 👁️ Página principal (ya creada)
│   │   └── Privacy.cshtml        # 👁️ Página de privacidad (ya creada)
│   └── Shared/
│       ├── _Layout.cshtml        # 🎨 Plantilla base (ya creada)
│       └── _ViewStart.cshtml     # ⚙️ Configuración de vistas (ya creada)
└── wwwroot/                      # 🌐 Archivos públicos
    ├── css/
    ├── js/
    └── lib/                      # 📚 Librerías (Bootstrap, jQuery)
```

#### Paso 3: Personalizar el controlador principal para la clínica
```csharp
// Controllers/HomeController.cs - Modificar el controlador existente
using Microsoft.AspNetCore.Mvc;
using SistemaClinicaMVC.Models;
using System.Diagnostics;

namespace SistemaClinicaMVC.Controllers
{
    public class HomeController : Controller
    {
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger)
        {
            _logger = logger;
        }

        // Página principal de la clínica
        public IActionResult Index()
        {
            _logger.LogInformation("Usuario visitó la página principal de la clínica");
            
            // Datos de ejemplo para mostrar en la página
            ViewBag.NombreClinica = "Clínica San José";
            ViewBag.TotalPacientes = 150;
            ViewBag.MedicosDisponibles = 8;
            ViewBag.CitasHoy = 25;
            
            return View();
        }

        // Página de información de la clínica
        public IActionResult Privacy()
        {
            ViewBag.TituloSeccion = "Información de la Clínica";
            return View();
        }

        // Nueva acción: Módulos del sistema
        public IActionResult Modulos()
        {
            _logger.LogInformation("Usuario accedió a la página de módulos");
            
            // Lista de módulos disponibles
            var modulos = new List<string>
            {
                "Gestión de Pacientes",
                "Directorio de Médicos", 
                "Sistema de Citas",
                "Historiales Médicos",
                "Reportes y Estadísticas"
            };
            
            ViewBag.ModulosDisponibles = modulos;
            return View();
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
            return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
        }
    }
}
```

#### Paso 4: Personalizar la vista principal de la clínica
```html
<!-- Views/Home/Index.cshtml - Modificar la vista existente -->
@{
    ViewData["Title"] = "Sistema de Clínica Médica";
}

<div class="text-center">
    <h1 class="display-4 text-primary">🏥 @ViewBag.NombreClinica</h1>
    <p class="lead">Sistema Integral de Gestión Médica</p>
    <hr class="my-4">
</div>

<!-- Estadísticas principales -->
<div class="row mb-4">
    <div class="col-md-3">
        <div class="card text-white bg-primary mb-3">
            <div class="card-body text-center">
                <h4 class="card-title">@ViewBag.TotalPacientes</h4>
                <p class="card-text">Pacientes Registrados</p>
            </div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="card text-white bg-success mb-3">
            <div class="card-body text-center">
                <h4 class="card-title">@ViewBag.MedicosDisponibles</h4>
                <p class="card-text">Médicos Disponibles</p>
            </div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="card text-white bg-info mb-3">
            <div class="card-body text-center">
                <h4 class="card-title">@ViewBag.CitasHoy</h4>
                <p class="card-text">Citas de Hoy</p>
            </div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="card text-white bg-warning mb-3">
            <div class="card-body text-center">
                <h4 class="card-title">5</h4>
                <p class="card-text">Especialidades</p>
            </div>
        </div>
    </div>
</div>

<!-- Módulos principales -->
<div class="row">
    <div class="col-md-4 mb-3">
        <div class="card">
            <div class="card-body text-center">
                <h5 class="card-title">👤 Pacientes</h5>
                <p class="card-text">Registro y gestión de pacientes de la clínica</p>
                <a href="#" class="btn btn-primary">Gestionar Pacientes</a>
            </div>
        </div>
    </div>
    <div class="col-md-4 mb-3">
        <div class="card">
            <div class="card-body text-center">
                <h5 class="card-title">👩‍⚕️ Médicos</h5>
                <p class="card-text">Directorio y especialidades médicas</p>
                <a href="#" class="btn btn-success">Ver Médicos</a>
            </div>
        </div>
    </div>
    <div class="col-md-4 mb-3">
        <div class="card">
            <div class="card-body text-center">
                <h5 class="card-title">📅 Citas</h5>
                <p class="card-text">Programación y gestión de citas médicas</p>
                <a href="#" class="btn btn-info">Programar Citas</a>
            </div>
        </div>
    </div>
</div>

<!-- Crear nueva vista para módulos -->
<!-- Views/Home/Modulos.cshtml (archivo nuevo) -->
@{
    ViewData["Title"] = "Módulos del Sistema";
}

<div class="container">
    <h2 class="text-center mb-4">📋 Módulos del Sistema de Clínica</h2>
    
    <div class="row">
        @foreach (var modulo in ViewBag.ModulosDisponibles)
        {
            <div class="col-md-6 mb-3">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">✅ @modulo</h5>
                        <p class="card-text">Funcionalidad disponible en desarrollo</p>
                        <small class="text-muted">Próximamente en las siguientes secciones</small>
                    </div>
                </div>
            </div>
        }
    </div>
    
    <div class="text-center mt-4">
        <a asp-action="Index" class="btn btn-secondary">← Volver al Inicio</a>
    </div>
</div>
``` 
#### Paso 5: Actualizar la navegación en el Layout
```html
<!-- Views/Shared/_Layout.cshtml - Modificar la barra de navegación -->
<nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
    <div class="container-fluid">
        <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">
            🏥 Sistema Clínica
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
            <ul class="navbar-nav flex-grow-1">
                <li class="nav-item">
                    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">🏠 Inicio</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Modulos">📋 Módulos</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">ℹ️ Información</a>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

#### Paso 6: Ejecutar y probar el proyecto MVC
```bash
# En la terminal de Visual Studio o externa:
dotnet run

# Navegar a las URLs:
# - https://localhost:5001/ → Página principal con estadísticas
# - https://localhost:5001/Home/Modulos → Lista de módulos
# - https://localhost:5001/Home/Privacy → Información de la clínica
```

### Puntos Clave a Explicar:
- **Patrón MVC**: ¿Cómo se separan las responsabilidades?
  - **Model**: Datos y lógica de negocio (próximo módulo)
  - **View**: Interfaz de usuario (archivos .cshtml)
  - **Controller**: Lógica de control y navegación
- **Razor Syntax**: ¿Cómo mezclar C# con HTML?
- **ViewBag**: ¿Cómo pasar datos del controlador a la vista?
- **Tag Helpers**: `asp-controller`, `asp-action` para navegación
- **Bootstrap**: Framework CSS ya incluido en la plantilla MVC
- **Estructura de carpetas**: Convenciones de ASP.NET Core MVC

## 🏠 Desafío Autónomo (Casa)

### Tarea: "Extensión del Sistema MVC de Clínica"
Continuar el desarrollo del sistema MVC de clínica iniciado en clase:

**Requisitos:**
1. **Nueva acción**: `Especialidades()` en HomeController - Lista de especialidades médicas
2. **Nueva acción**: `Emergencias()` en HomeController - Página de contacto de emergencias  
3. **Nueva acción**: `Horarios(string dia)` en HomeController - Horarios de atención por día
4. **Nueva acción**: `Ubicacion()` en HomeController - Información de contacto y ubicación
5. **Crear las vistas correspondientes** para cada nueva acción
6. **Actualizar la navegación** en _Layout.cshtml para incluir los nuevos enlaces

**Ejemplo de nueva acción:**
```csharp
// Agregar al HomeController.cs
public IActionResult Especialidades()
{
    var especialidades = new List<string>
    {
        "Medicina General",
        "Pediatría", 
        "Cardiología",
        "Dermatología",
        "Ginecología"
    };
    
    ViewBag.EspecialidadesDisponibles = especialidades;
    return View();
}

public IActionResult Horarios(string dia)
{
    ViewBag.DiaSeleccionado = dia ?? "lunes";
    ViewBag.HorarioAtencion = "8:00 AM - 6:00 PM";
    return View();
}
```

**Entregables:**
- Proyecto MVC de Visual Studio 2022 completo y funcional
- Screenshots de cada nueva vista funcionando
- Archivo README.md con instrucciones de ejecución
- Navegación funcional entre todas las páginas

**Criterios de Evaluación:**
- ✅ Todas las acciones del controlador funcionan correctamente
- ✅ Vistas creadas siguiendo las convenciones MVC
- ✅ Navegación coherente usando Tag Helpers
- ✅ Tema consistente de clínica médica
- ✅ Uso correcto de ViewBag para pasar datos
- ✅ Código limpio y comentado

## 🚀 Avance en el Proyecto Final

### Proyecto Guiado: Sistema de Clínica Médica
Durante todo el roadmap desarrollaremos un **Sistema de Clínica Médica completo** que incluirá:

**Módulos del Sistema:**
- 👤 **Gestión de Pacientes** - Registro, historiales médicos
- 👩‍⚕️ **Gestión de Médicos** - Especialidades, horarios, disponibilidad
- 📅 **Sistema de Citas** - Programación, recordatorios, cancelaciones
- 💊 **Recetas Médicas** - Emisión, historial, medicamentos
- 📊 **Reportes** - Estadísticas, dashboards, análisis

**Entidades Principales:**
```csharp
// Adelanto de las entidades que desarrollaremos
public class Paciente
{
    public int Id { get; set; }
    public string Cedula { get; set; }
    public string Nombres { get; set; }
    public string Apellidos { get; set; }
    public DateTime FechaNacimiento { get; set; }
    public string Telefono { get; set; }
    public string Email { get; set; }
    public string Direccion { get; set; }
    // Más propiedades...
}

public class Medico
{
    public int Id { get; set; }
    public string Cedula { get; set; }
    public string Nombres { get; set; }
    public string Apellidos { get; set; }
    public string Especialidad { get; set; }
    public string NumeroLicencia { get; set; }
    // Más propiedades...
}

public class CitaMedica
{
    public int Id { get; set; }
    public int PacienteId { get; set; }
    public int MedicoId { get; set; }
    public DateTime FechaHora { get; set; }
    public string Motivo { get; set; }
    public string Estado { get; set; } // Programada, Completada, Cancelada
    // Más propiedades...
}
```

**Tecnologías que usaremos:**
- ⚙️ **Visual Studio Community 2022** como IDE principal
- 🌐 **ASP.NET Core 8** para el backend
- 🎨 **Bootstrap 5** para interfaces responsivas
- 🗄️ **Entity Framework Core** para base de datos
- 🔐 **JWT Authentication** para seguridad
- 📝 **Swagger** para documentación de APIs

### Tareas para el Proyecto:
1. **Elegir el dominio** de aplicación
2. **Crear el proyecto base** con la estructura inicial
3. **Documentar la elección** en un archivo `PROYECTO.md`
4. **Subir a GitHub** el proyecto inicial

### Estructura Inicial del Proyecto:
```
MiProyectoFinal/
├── Controllers/
├── Models/
├── Views/
├── wwwroot/
├── Program.cs
├── appsettings.json
├── PROYECTO.md
└── README.md
```

## 📝 Material de Apoyo
- [Documentación oficial ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/)
- [Tutorial paso a paso](./tutorial-paso-a-paso.md)
- [Ejemplos de código](./ejemplos/)
- [Troubleshooting común](./troubleshooting.md)

## ✅ Checklist de Completado
- [ ] Entorno de desarrollo configurado
- [ ] Primer proyecto ASP.NET Core creado y ejecutado
- [ ] Comprende la estructura básica del proyecto
- [ ] Desafío autónomo completado
- [ ] Proyecto final iniciado y subido a GitHub

---
**Siguiente:** [02 - Frontend con Formularios Web](../02-frontend-formularios/)
