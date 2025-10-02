# 02 - Frontend con Formularios Web

## 🎯 Objetivo de Aprendizaje
Al finalizar esta sección, el estudiante será capaz de:
- Integrar Bootstrap en ASP.NET Core
- Crear formularios HTML funcionales
- Manejar datos de formularios en el servidor
- Implementar páginas responsivas y atractivas

## 📚 Contenidos Principales

### 1. Integración de Bootstrap
**¿Qué es Bootstrap?**
Bootstrap es como un "kit de herramientas CSS" que te da componentes listos para usar: botones bonitos, formularios, navegación, etc.

**Formas de integrar Bootstrap:**
```html
<!-- Opción 1: CDN (más fácil) -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- Opción 2: Archivos locales (más control) -->
<link href="~/css/bootstrap.min.css" rel="stylesheet">
```

**Sistema de Grid (columnas):**
```html
<!-- Pantalla dividida en 12 columnas -->
<div class="container">
    <div class="row">
        <div class="col-md-6">Mitad izquierda</div>    <!-- 6 de 12 columnas -->
        <div class="col-md-6">Mitad derecha</div>      <!-- 6 de 12 columnas -->
    </div>
</div>
```

**Componentes útiles:**
```html
<!-- Botones con estilos -->
<button class="btn btn-primary">Primario</button>
<button class="btn btn-success">Éxito</button>
<button class="btn btn-danger">Peligro</button>

<!-- Tarjetas (cards) -->
<div class="card">
    <div class="card-body">
        <h5 class="card-title">Título</h5>
        <p class="card-text">Contenido de la tarjeta</p>
    </div>
</div>
```

### 2. Formularios HTML
**Elementos básicos explicados:**

```html
<!-- Formulario completo de ejemplo -->
<form method="post" action="/procesar">
    <!-- Campo de texto -->
    <div class="mb-3">
        <label for="nombre" class="form-label">Nombre:</label>
        <input type="text" id="nombre" name="nombre" class="form-control" required>
    </div>
    
    <!-- Campo de email (con validación automática) -->
    <div class="mb-3">
        <label for="email" class="form-label">Email:</label>
        <input type="email" id="email" name="email" class="form-control" required>
    </div>
    
    <!-- Lista desplegable -->
    <div class="mb-3">
        <label for="pais" class="form-label">País:</label>
        <select id="pais" name="pais" class="form-select">
            <option value="">Seleccione...</option>
            <option value="chile">Chile</option>
            <option value="argentina">Argentina</option>
        </select>
    </div>
    
    <!-- Área de texto -->
    <div class="mb-3">
        <label for="mensaje" class="form-label">Mensaje:</label>
        <textarea id="mensaje" name="mensaje" class="form-control" rows="3"></textarea>
    </div>
    
    <!-- Checkbox -->
    <div class="mb-3 form-check">
        <input type="checkbox" id="acepta" name="acepta" class="form-check-input" required>
        <label for="acepta" class="form-check-label">Acepto términos y condiciones</label>
    </div>
    
    <button type="submit" class="btn btn-primary">Enviar</button>
</form>
```

**Atributos de validación HTML5:**
- `required` - Campo obligatorio
- `type="email"` - Valida formato de email
- `type="number"` - Solo números
- `min="18"` - Valor mínimo
- `max="100"` - Valor máximo
- `pattern="[0-9]{4}"` - Formato específico (ej: 4 dígitos)

**Diferencia entre GET y POST:**
```html
<!-- GET: Los datos se ven en la URL (para búsquedas) -->
<form method="get" action="/buscar">
    <!-- URL resultado: /buscar?termino=pizza -->

<!-- POST: Los datos van ocultos (para envío de información) -->
<form method="post" action="/registrar">
    <!-- Los datos van en el cuerpo de la petición (no se ven) -->
```

### 3. Páginas Estáticas con Estilo
**Estructura HTML semántica:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Página</title>
</head>
<body>
    <!-- Navegación -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="/">Mi Sitio</a>
            <ul class="navbar-nav">
                <li class="nav-item">
                    <a class="nav-link" href="/inicio">Inicio</a>
                </li>
            </ul>
        </div>
    </nav>
    
    <!-- Contenido principal -->
    <main class="container mt-4">
        <h1>Título Principal</h1>
        <p>Contenido de la página...</p>
    </main>
    
    <!-- Pie de página -->
    <footer class="bg-dark text-white text-center py-3">
        <p>&copy; 2024 Mi Sitio Web</p>
    </footer>
</body>
</html>
```

**Clases Bootstrap útiles:**
- `container` - Centra contenido con márgenes
- `row` / `col` - Sistema de columnas
- `mt-4` - Margen superior (margin-top)
- `mb-3` - Margen inferior (margin-bottom)
- `text-center` - Centrar texto
- `d-flex` - Display flex
- `justify-content-between` - Espacio entre elementos

### 4. Manejo Básico de Datos
**¿Cómo recibir datos del formulario?**

```csharp
// En el controlador o Page Model
[HttpPost]
public IActionResult ProcesarFormulario(string nombre, string email, string mensaje)
{
    // Los nombres de los parámetros deben coincidir con los "name" del HTML
    
    // Validar datos
    if (string.IsNullOrEmpty(nombre))
    {
        ViewBag.Error = "El nombre es requerido";
        return View(); // Regresar al formulario con error
    }
    
    // Procesar datos (guardar, enviar email, etc.)
    Console.WriteLine($"Nombre: {nombre}, Email: {email}");
    
    // Mostrar mensaje de éxito
    ViewBag.Success = "Formulario enviado correctamente";
    return View();
}
```

**Usar ViewBag para mostrar mensajes:**
```html
<!-- En la vista HTML -->
@if (ViewBag.Error != null)
{
    <div class="alert alert-danger">
        @ViewBag.Error
    </div>
}

@if (ViewBag.Success != null)
{
    <div class="alert alert-success">
        @ViewBag.Success
    </div>
}
```

**Redirecciones útiles:**
```csharp
// Redirigir a otra página después de procesar
return RedirectToAction("Gracias");

// Redirigir con mensaje
TempData["Mensaje"] = "Datos guardados correctamente";
return RedirectToAction("Index");
```

## 🔧 Actividad Práctica en Clase (2 horas)

## 🔧 Actividad Práctica en Clase (2 horas)

### Ejercicio Guiado: "Agregar Formularios al Sistema MVC de Clínica"
*Continuamos con el proyecto SistemaClinicaMVC del módulo anterior*

#### Paso 1: Mejorar el Layout con Bootstrap (ya incluido en el template MVC)
```html
<!-- Views/Shared/_Layout.cshtml - El template MVC ya incluye Bootstrap -->
<!-- Solo necesitamos personalizar el navbar existente para la clínica -->

<nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-primary border-bottom box-shadow mb-3">
    <div class="container-fluid">
        <a class="navbar-brand text-white" asp-area="" asp-controller="Home" asp-action="Index">
            🏥 Clínica San José
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
            <ul class="navbar-nav flex-grow-1">
                <li class="nav-item">
                    <a class="nav-link text-white" asp-area="" asp-controller="Home" asp-action="Index">
                        🏠 Inicio
                    </a>
                </li>
                <li class="nav-item">
                    <a class="nav-link text-white" asp-area="" asp-controller="Pacientes" asp-action="Index">
                        👤 Pacientes
                    </a>
                </li>
                <li class="nav-item">
                    <a class="nav-link text-white" asp-area="" asp-controller="Home" asp-action="Modulos">
                        📋 Módulos
                    </a>
                </li>
                <li class="nav-item">
                    <a class="nav-link text-white" asp-area="" asp-controller="Home" asp-action="Privacy">
                        ℹ️ Información
                    </a>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

#### Paso 2: Crear el modelo Paciente
```csharp
// Models/Paciente.cs - Crear nuevo archivo
using System.ComponentModel.DataAnnotations;

namespace SistemaClinicaMVC.Models
{
    public class Paciente
{
    [Required(ErrorMessage = "La cédula es obligatoria")]
    [Display(Name = "Número de Cédula")]
    [StringLength(10, MinimumLength = 10, ErrorMessage = "La cédula debe tener 10 dígitos")]
    public string Cedula { get; set; }

    [Required(ErrorMessage = "Los nombres son obligatorios")]
    [Display(Name = "Nombres")]
    [StringLength(50, ErrorMessage = "Los nombres no pueden exceder 50 caracteres")]
    public string Nombres { get; set; }

    [Required(ErrorMessage = "Los apellidos son obligatorios")]
    [Display(Name = "Apellidos")]
    [StringLength(50, ErrorMessage = "Los apellidos no pueden exceder 50 caracteres")]
    public string Apellidos { get; set; }

    [Required(ErrorMessage = "La fecha de nacimiento es obligatoria")]
    [Display(Name = "Fecha de Nacimiento")]
    [DataType(DataType.Date)]
    public DateTime FechaNacimiento { get; set; }

    [Required(ErrorMessage = "El teléfono es obligatorio")]
    [Display(Name = "Teléfono")]
    [Phone(ErrorMessage = "El formato del teléfono no es válido")]
    public string Telefono { get; set; }

    [Required(ErrorMessage = "El email es obligatorio")]
    [Display(Name = "Correo Electrónico")]
    [EmailAddress(ErrorMessage = "El formato del email no es válido")]
    public string Email { get; set; }

    [Display(Name = "Dirección")]
    [StringLength(200, ErrorMessage = "La dirección no puede exceder 200 caracteres")]
    public string Direccion { get; set; }

    [Display(Name = "Tipo de Sangre")]
    public string? TipoSangre { get; set; }

    [Display(Name = "Contacto de Emergencia")]
    public string? ContactoEmergencia { get; set; }

    [Display(Name = "Teléfono de Emergencia")]
    public string? TelefonoEmergencia { get; set; }

    // Propiedades calculadas
    public string NombreCompleto => $"{Nombres} {Apellidos}";
    
    public int Edad
    {
        get
        {
            var hoy = DateTime.Today;
            var edad = hoy.Year - FechaNacimiento.Year;
            if (FechaNacimiento.Date > hoy.AddYears(-edad)) edad--;
            return edad;
        }
    }
}
```

#### Paso 3: Crear el controlador de Pacientes
```csharp
// Controllers/PacientesController.cs - Crear nuevo archivo
using Microsoft.AspNetCore.Mvc;
using SistemaClinicaMVC.Models;

namespace SistemaClinicaMVC.Controllers
{
    public class PacientesController : Controller
    {
        // Lista estática temporal (en el siguiente módulo usaremos base de datos)
        private static List<Paciente> _pacientes = new List<Paciente>
        {
            new Paciente
            {
                Cedula = "1234567890",
                Nombres = "Juan Carlos",
                Apellidos = "Pérez García",
                FechaNacimiento = new DateTime(1990, 5, 15),
                Telefono = "0987654321",
                Email = "juan.perez@email.com",
                Direccion = "Av. Principal 123",
                TipoSangre = "O+"
            },
            new Paciente
            {
                Cedula = "0987654321", 
                Nombres = "María Elena",
                Apellidos = "González López",
                FechaNacimiento = new DateTime(1985, 8, 22),
                Telefono = "0912345678",
                Email = "maria.gonzalez@email.com",
                Direccion = "Calle Secundaria 456",
                TipoSangre = "A+"
            }
        };

        // GET: Pacientes - Lista todos los pacientes
        public IActionResult Index()
        {
            ViewBag.TotalPacientes = _pacientes.Count;
            return View(_pacientes);
        }

        // GET: Pacientes/Crear - Muestra el formulario de registro
        public IActionResult Crear()
        {
            return View();
        }

        // POST: Pacientes/Crear - Procesa el formulario de registro
        [HttpPost]
        [ValidateAntiForgeryToken]
        public IActionResult Crear(Paciente paciente)
        {
            if (ModelState.IsValid)
            {
                // Verificar si la cédula ya existe
                if (_pacientes.Any(p => p.Cedula == paciente.Cedula))
                {
                    ModelState.AddModelError("Cedula", "Ya existe un paciente con esta cédula");
                    return View(paciente);
                }

                // Agregar paciente a la lista
                _pacientes.Add(paciente);
                
                TempData["Mensaje"] = $"Paciente {paciente.NombreCompleto} registrado exitosamente";
                return RedirectToAction(nameof(Index));
            }

            // Si hay errores, regresar al formulario
            return View(paciente);
        }

        // GET: Pacientes/Detalle/1234567890 - Muestra detalles del paciente
        public IActionResult Detalle(string cedula)
        {
            if (string.IsNullOrEmpty(cedula))
            {
                return NotFound();
            }

            var paciente = _pacientes.FirstOrDefault(p => p.Cedula == cedula);
            if (paciente == null)
            {
                return NotFound();
            }

            return View(paciente);
        }

        // GET: Pacientes/Buscar - Formulario de búsqueda
        public IActionResult Buscar()
        {
            return View();
        }

        // POST: Pacientes/Buscar - Procesar búsqueda
        [HttpPost]
        public IActionResult Buscar(string termino)
        {
            if (string.IsNullOrWhiteSpace(termino))
            {
                ViewBag.Mensaje = "Ingrese un término de búsqueda";
                return View();
            }

            var resultados = _pacientes.Where(p => 
                p.Nombres.ToLower().Contains(termino.ToLower()) ||
                p.Apellidos.ToLower().Contains(termino.ToLower()) ||
                p.Cedula.Contains(termino)
            ).ToList();

            ViewBag.TerminoBusqueda = termino;
            ViewBag.CantidadResultados = resultados.Count;

            return View("ResultadosBusqueda", resultados);
        }
    }
}
```

#### Paso 4: Crear las vistas para el módulo de Pacientes
```html
<!-- Views/Pacientes/Index.cshtml - Lista de pacientes -->
@model List<SistemaClinicaMVC.Models.Paciente>

@{
    ViewData["Title"] = "Lista de Pacientes";
}

<div class="d-flex justify-content-between align-items-center mb-4">
    <h2>👤 Gestión de Pacientes</h2>
    <a asp-action="Crear" class="btn btn-success">
        <i class="bi bi-person-plus"></i> Registrar Nuevo Paciente
    </a>
</div>

@if (TempData["Mensaje"] != null)
{
    <div class="alert alert-success alert-dismissible fade show">
        <strong>¡Éxito!</strong> @TempData["Mensaje"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

<div class="row mb-3">
    <div class="col-md-6">
        <div class="card bg-primary text-white">
            <div class="card-body text-center">
                <h4>@ViewBag.TotalPacientes</h4>
                <p class="mb-0">Pacientes Registrados</p>
            </div>
        </div>
    </div>
    <div class="col-md-6">
        <a asp-action="Buscar" class="btn btn-outline-primary w-100 p-3">
            🔍 Buscar Paciente
        </a>
    </div>
</div>

@if (Model.Any())
{
    <div class="table-responsive">
        <table class="table table-hover">
            <thead class="table-dark">
                <tr>
                    <th>Cédula</th>
                    <th>Nombre Completo</th>
                    <th>Edad</th>
                    <th>Teléfono</th>
                    <th>Email</th>
                    <th>Acciones</th>
                </tr>
            </thead>
            <tbody>
                @foreach (var paciente in Model)
                {
                    <tr>
                        <td>@paciente.Cedula</td>
                        <td>@paciente.NombreCompleto</td>
                        <td>@paciente.Edad años</td>
                        <td>@paciente.Telefono</td>
                        <td>@paciente.Email</td>
                        <td>
                            <a asp-action="Detalle" asp-route-cedula="@paciente.Cedula" 
                               class="btn btn-sm btn-info">Ver</a>
                        </td>
                    </tr>
                }
            </tbody>
        </table>
    </div>
}
else
{
    <div class="text-center py-5">
        <h4 class="text-muted">No hay pacientes registrados</h4>
        <p class="text-muted">Registre el primer paciente para comenzar</p>
        <a asp-action="Crear" class="btn btn-primary">Registrar Paciente</a>
    </div>
}
    public string TipoSangre { get; set; }

    [Display(Name = "Contacto de Emergencia")]
    public string ContactoEmergencia { get; set; }

    [Display(Name = "Teléfono de Emergencia")]
    public string TelefonoEmergencia { get; set; }
}
```



#### Paso 5: Crear el formulario de registro de pacientes
```html
<!-- Views/Pacientes/Crear.cshtml - Formulario de registro -->
@model SistemaClinicaMVC.Models.Paciente

@{
    ViewData["Title"] = "Registrar Paciente";
}

<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header bg-success text-white">
                <h4 class="mb-0">
                    👤 Registrar Nuevo Paciente
                </h4>
            </div>
            <div class="card-body">
                <form asp-action="Crear" method="post">
                    @Html.AntiForgeryToken()
                    
                    <!-- Información Personal -->
                    <h5 class="text-primary mb-3">📋 Información Personal</h5>
                    <div class="row">
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="Cedula" class="form-label"></label>
                                <input asp-for="Cedula" class="form-control" placeholder="1234567890" />
                                <span asp-validation-for="Cedula" class="text-danger"></span>
                            </div>
                        </div>
                        
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="FechaNacimiento" class="form-label"></label>
                                <input asp-for="FechaNacimiento" class="form-control" type="date" />
                                <span asp-validation-for="FechaNacimiento" class="text-danger"></span>
                            </div>
                        </div>
                        
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="Nombres" class="form-label"></label>
                                <input asp-for="Nombres" class="form-control" placeholder="Juan Carlos" />
                                <span asp-validation-for="Nombres" class="text-danger"></span>
                            </div>
                        </div>
                        
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="Apellidos" class="form-label"></label>
                                <input asp-for="Apellidos" class="form-control" placeholder="Pérez González" />
                                <span asp-validation-for="Apellidos" class="text-danger"></span>
                            </div>
                        </div>
                    </div>

                    <!-- Información de Contacto -->
                    <h5 class="text-primary mb-3 mt-3">📞 Información de Contacto</h5>
                    <div class="row">
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="Telefono" class="form-label"></label>
                                <input asp-for="Telefono" class="form-control" placeholder="0987654321" />
                                <span asp-validation-for="Telefono" class="text-danger"></span>
                            </div>
                        </div>
                        
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="Email" class="form-label"></label>
                                <input asp-for="Email" class="form-control" placeholder="paciente@email.com" />
                                <span asp-validation-for="Email" class="text-danger"></span>
                            </div>
                        </div>
                        
                        <div class="col-md-12">
                            <div class="mb-3">
                                <label asp-for="Direccion" class="form-label"></label>
                                <textarea asp-for="Direccion" class="form-control" rows="2" 
                                         placeholder="Av. Principal 123, Ciudad"></textarea>
                                <span asp-validation-for="Direccion" class="text-danger"></span>
                            </div>
                        </div>
                    </div>

                    <!-- Información Médica -->
                    <h5 class="text-primary mb-3 mt-3">🩺 Información Médica</h5>
                    <div class="row">
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="TipoSangre" class="form-label"></label>
                                <select asp-for="TipoSangre" class="form-select">
                                    <option value="">Seleccione tipo de sangre</option>
                                    <option value="A+">A+</option>
                                    <option value="A-">A-</option>
                                    <option value="B+">B+</option>
                                    <option value="B-">B-</option>
                                    <option value="AB+">AB+</option>
                                    <option value="AB-">AB-</option>
                                    <option value="O+">O+</option>
                                    <option value="O-">O-</option>
                                </select>
                            </div>
                        </div>
                        
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="ContactoEmergencia" class="form-label"></label>
                                <input asp-for="ContactoEmergencia" class="form-control" 
                                       placeholder="Nombre del contacto de emergencia" />
                            </div>
                        </div>
                        
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label asp-for="TelefonoEmergencia" class="form-label"></label>
                                <input asp-for="TelefonoEmergencia" class="form-control" 
                                       placeholder="Teléfono de emergencia" />
                            </div>
                        </div>
                    </div>

                    <!-- Botones -->
                    <div class="row mt-4">
                        <div class="col-md-12 text-center">
                            <button type="submit" class="btn btn-success btn-lg me-2">
                                💾 Registrar Paciente
                            </button>
                            <a asp-action="Index" class="btn btn-secondary btn-lg">
                                ❌ Cancelar
                            </a>
                        </div>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
```

#### Paso 6: Crear vista de detalles del paciente
```html
<!-- Views/Pacientes/Detalle.cshtml -->
@model SistemaClinicaMVC.Models.Paciente

@{
    ViewData["Title"] = "Detalle del Paciente";
}

<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header bg-info text-white d-flex justify-content-between align-items-center">
                <h4 class="mb-0">👤 @Model.NombreCompleto</h4>
                <span class="badge bg-light text-dark">@Model.Edad años</span>
            </div>
            <div class="card-body">
                <div class="row">
                    <div class="col-md-6">
                        <h5 class="text-primary">📋 Información Personal</h5>
                        <table class="table table-borderless">
                            <tr>
                                <td><strong>Cédula:</strong></td>
                                <td>@Model.Cedula</td>
                            </tr>
                            <tr>
                                <td><strong>Fecha Nacimiento:</strong></td>
                                <td>@Model.FechaNacimiento.ToString("dd/MM/yyyy")</td>
                            </tr>
                            <tr>
                                <td><strong>Tipo de Sangre:</strong></td>
                                <td>@(Model.TipoSangre ?? "No especificado")</td>
                            </tr>
                        </table>
                    </div>
                    
                    <div class="col-md-6">
                        <h5 class="text-primary">📞 Contacto</h5>
                        <table class="table table-borderless">
                            <tr>
                                <td><strong>Teléfono:</strong></td>
                                <td>@Model.Telefono</td>
                            </tr>
                            <tr>
                                <td><strong>Email:</strong></td>
                                <td>@Model.Email</td>
                            </tr>
                            <tr>
                                <td><strong>Dirección:</strong></td>
                                <td>@(Model.Direccion ?? "No especificada")</td>
                            </tr>
                        </table>
                    </div>
                </div>
                
                @if (!string.IsNullOrEmpty(Model.ContactoEmergencia))
                {
                    <div class="row mt-3">
                        <div class="col-md-12">
                            <h5 class="text-danger">🚨 Contacto de Emergencia</h5>
                            <div class="alert alert-warning">
                                <strong>@Model.ContactoEmergencia</strong><br>
                                Teléfono: @(Model.TelefonoEmergencia ?? "No especificado")
                            </div>
                        </div>
                    </div>
                }
                
                <div class="text-center mt-4">
                    <a asp-action="Index" class="btn btn-primary">← Volver a Lista</a>
                </div>
            </div>
        </div>
    </div>
</div>
```

#### Paso 7: Probar el módulo completo de Pacientes
```bash
# Ejecutar la aplicación
dotnet run

# Navegar a las URLs:
# - https://localhost:5001/Pacientes → Lista de pacientes
# - https://localhost:5001/Pacientes/Crear → Formulario de registro
# - https://localhost:5001/Pacientes/Detalle/1234567890 → Detalles del paciente
```
```

#### Paso 2: Crear el Formulario
```html
<!-- Views/Home/Contacto.cshtml -->
@{
    ViewData["Title"] = "Contacto";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header">
                <h3 class="mb-0">Formulario de Contacto</h3>
            </div>
            <div class="card-body">
                <form method="post" action="/contacto">
                    <div class="mb-3">
                        <label for="nombre" class="form-label">Nombre Completo *</label>
                        <input type="text" class="form-control" id="nombre" name="nombre" required>
                    </div>

                    <div class="mb-3">
                        <label for="email" class="form-label">Email *</label>
                        <input type="email" class="form-control" id="email" name="email" required>
                    </div>

                    <div class="mb-3">
                        <label for="telefono" class="form-label">Teléfono</label>
                        <input type="tel" class="form-control" id="telefono" name="telefono">
                    </div>

                    <div class="mb-3">
                        <label for="asunto" class="form-label">Asunto *</label>
                        <select class="form-select" id="asunto" name="asunto" required>
                            <option value="">Seleccione un asunto...</option>
                            <option value="consulta">Consulta General</option>
                            <option value="soporte">Soporte Técnico</option>
                            <option value="comercial">Información Comercial</option>
                        </select>
                    </div>

                    <div class="mb-3">
                        <label for="mensaje" class="form-label">Mensaje *</label>
                        <textarea class="form-control" id="mensaje" name="mensaje" rows="4" required></textarea>
                    </div>

                    <div class="d-grid">
                        <button type="submit" class="btn btn-primary btn-lg">Enviar Mensaje</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
```

#### Paso 3: Controlador para Manejar Datos
```csharp
// Controllers/HomeController.cs
using Microsoft.AspNetCore.Mvc;

public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult Contacto()
    {
        return View();
    }

    [HttpPost]
    public IActionResult Contacto(string nombre, string email, string telefono, string asunto, string mensaje)
    {
        // Validación básica
        if (string.IsNullOrEmpty(nombre) || string.IsNullOrEmpty(email) || string.IsNullOrEmpty(mensaje))
        {
            ViewBag.Error = "Por favor, complete todos los campos obligatorios.";
            return View();
        }

        // Aquí procesarías los datos (guardar en BD, enviar email, etc.)
        
        ViewBag.Success = $"¡Gracias {nombre}! Tu mensaje ha sido enviado correctamente.";
        return View();
    }
}
```

### Puntos Clave a Explicar:
- **Patrón MVC**: Separación Model-View-Controller
- **Tag Helpers**: `asp-for`, `asp-action`, etc.
- **Model Binding**: Automático entre formulario y modelo
- **Data Annotations**: Validaciones en el modelo
- **ViewBag vs TempData**: Diferencias y usos apropiados
- **Bootstrap en ASP.NET Core**: Clases CSS para formularios responsivos

## 🏠 Desafío Autónomo (Casa)

### Tarea: "Módulo de Médicos para el Sistema de Clínica"
Crear un módulo completo de médicos siguiendo el patrón establecido con pacientes.

**Requisitos Funcionales:**
1. **Modelo Medico** (`Models/Medico.cs`)
   - Cédula, Nombres, Apellidos, Email, Teléfono
   - Especialidad, Número de Licencia Médica
   - Horarios de atención, Años de experiencia
   - Validaciones usando Data Annotations

2. **Controlador MedicosController** (`Controllers/MedicosController.cs`)
   - Acciones: Index, Crear, Detalle, Buscar
   - Lista estática temporal de médicos
   - Validación y manejo de errores

3. **Vistas del módulo Médicos** (`Views/Medicos/`)
   - Index.cshtml - Lista de médicos con especialidades
   - Crear.cshtml - Formulario de registro de médicos
   - Detalle.cshtml - Información completa del médico

4. **Formulario de Búsqueda de Médicos**
   - Buscar por nombre, especialidad o número de licencia
   - Filtros por especialidad médica
   - Resultados organizados por especialidad

**Requisitos Técnicos:**
- ✅ Seguir el patrón MVC establecido en pacientes
- ✅ Usar Tag Helpers para formularios y navegación
- ✅ Data Annotations para validaciones
- ✅ Bootstrap para diseño responsivo
- ✅ TempData para mensajes de éxito/error
- ✅ Navegación coherente con el sistema existente

**Ejemplo de especialidades médicas:**
- Medicina General
- Cardiología
- Pediatría
- Ginecología
- Dermatología
- Ortopedia
- Neurología

**Entregables:**
- Código fuente del módulo completo de médicos
- Screenshots de todas las vistas funcionando
- Navegación funcional desde el menú principal
- Integración con el sistema existente de clínica

## 🚀 Avance en el Proyecto Final

### Continuación del Sistema de Clínica MVC
Expandir el proyecto SistemaClinicaMVC con más funcionalidades.

#### Módulos Implementados hasta Ahora:
1. ✅ **Página de inicio** - HomeController con estadísticas
2. ✅ **Módulo de Pacientes** - CRUD completo con formularios
3. 🔄 **Módulo de Médicos** - A implementar en la tarea

#### Próximos Módulos (Siguientes Secciones):
4. **Sistema de Citas** - Relacionar pacientes con médicos
5. **Dashboard con Razor Pages** - Reportes y estadísticas
6. **Base de Datos** - Persistencia real con Entity Framework

### Estado Actual del Sistema:
```
SistemaClinicaMVC/
├── Controllers/
│   ├── HomeController.cs          ✅ Implementado
│   ├── PacientesController.cs     ✅ Implementado
│   └── MedicosController.cs       🔄 A implementar
├── Models/
│   ├── Paciente.cs               ✅ Implementado
│   └── Medico.cs                 🔄 A implementar
├── Views/
│   ├── Shared/
│   │   └── _Layout.cshtml        ✅ Personalizado para clínica
│   ├── Home/
│   │   ├── Index.cshtml          ✅ Dashboard clínica
│   │   └── Modulos.cshtml        ✅ Lista de módulos
│   └── Pacientes/
│       ├── Index.cshtml          ✅ Lista de pacientes
│       ├── Crear.cshtml          ✅ Formulario registro
│       └── Detalle.cshtml        ✅ Detalles paciente
└── wwwroot/                      ✅ Bootstrap incluido
```

### URLs Funcionales:
- `https://localhost:5001/` → Dashboard principal
- `https://localhost:5001/Home/Modulos` → Lista de módulos  
- `https://localhost:5001/Pacientes` → Gestión de pacientes
- `https://localhost:5001/Pacientes/Crear` → Registro de paciente
- `https://localhost:5001/Medicos` → 🔄 A implementar en la tarea

## 📝 Material de Apoyo
- [Bootstrap 5 Documentation](https://getbootstrap.com/docs/5.3/)
- [HTML Forms Guide](./html-forms-guide.md)
- [CSS Custom Properties](./css-customization.md)
- [Ejemplos de formularios](./ejemplos-formularios/)

## ✅ Checklist de Completado
- [ ] Bootstrap integrado correctamente
- [ ] Formulario funcional creado y probado
- [ ] Manejo de datos POST implementado
- [ ] Validación básica funcionando
- [ ] Desafío autónomo completado
- [ ] Frontend del proyecto final avanzado

---
**Anterior:** [01 - Introducción a ASP.NET](../01-introduccion-aspnet/)  
**Siguiente:** [03 - Dashboard con Razor Pages](../03-dashboard-razor/)
