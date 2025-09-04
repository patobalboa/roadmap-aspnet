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

### Ejercicio Guiado: "Formulario de Registro de Pacientes"

#### Paso 1: Configurar Bootstrap en el Sistema de Clínica
```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>🏥 Sistema de Clínica Médica</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
</head>
<body>
    <!-- Navbar de la clínica -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="/">
                <i class="fas fa-hospital"></i> Clínica San José
            </a>
            <div class="navbar-nav">
                <a class="nav-link" href="/"><i class="fas fa-home"></i> Inicio</a>
                <a class="nav-link" href="/pacientes"><i class="fas fa-user-injured"></i> Pacientes</a>
                <a class="nav-link" href="/medicos"><i class="fas fa-user-md"></i> Médicos</a>
                <a class="nav-link" href="/citas"><i class="fas fa-calendar-alt"></i> Citas</a>
            </div>
        </div>
    </nav>

    <!-- Contenido principal -->
    <main class="container mt-4">
        @RenderBody()
    </main>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

#### Paso 2: Crear modelo de Paciente
```csharp
// Models/PacienteViewModel.cs
using System.ComponentModel.DataAnnotations;

public class PacienteViewModel
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
    public string TipoSangre { get; set; }

    [Display(Name = "Contacto de Emergencia")]
    public string ContactoEmergencia { get; set; }

    [Display(Name = "Teléfono de Emergencia")]
    public string TelefonoEmergencia { get; set; }
}
```

#### Paso 3: Crear formulario de registro de pacientes
```html
<!-- Views/Pacientes/Registro.cshtml -->
@model PacienteViewModel
@{
    ViewData["Title"] = "Registro de Paciente";
}

<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0">
                    <i class="fas fa-user-plus"></i> Registro de Nuevo Paciente
                </h4>
            </div>
            <div class="card-body">
                <form asp-action="Registro" method="post">
                    <div class="row">
                        <!-- Información Personal -->
                        <div class="col-md-12">
                            <h5 class="text-primary mb-3">📋 Información Personal</h5>
                        </div>
                        
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

                        <!-- Información de Contacto -->
                        <div class="col-md-12">
                            <h5 class="text-primary mb-3 mt-3">📞 Información de Contacto</h5>
                        </div>
                        
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

                        <!-- Información Médica -->
                        <div class="col-md-12">
                            <h5 class="text-primary mb-3 mt-3">🩺 Información Médica</h5>
                        </div>
                        
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

                        <!-- Contacto de Emergencia -->
                        <div class="col-md-12">
                            <h5 class="text-primary mb-3 mt-3">🚨 Contacto de Emergencia</h5>
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
                            <button type="submit" class="btn btn-primary btn-lg me-2">
                                <i class="fas fa-save"></i> Registrar Paciente
                            </button>
                            <a href="/pacientes" class="btn btn-secondary btn-lg">
                                <i class="fas fa-times"></i> Cancelar
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
- Bootstrap grid system y componentes
- Diferencia entre GET y POST
- Validación HTML5 vs servidor
- ViewBag para mensajes temporales

## 🏠 Desafío Autónomo (Casa)

### Tarea: "Formulario de Registro Completo"
Crear un sistema de registro de usuarios con múltiples pasos.

**Requisitos Funcionales:**
1. **Página de Registro** (`/registro`)
   - Datos personales: nombre, apellido, email, teléfono
   - Fecha de nacimiento (con validación de edad mínima 18 años)
   - Género (select con opciones)
   - País y ciudad
   - Checkbox de términos y condiciones

2. **Página de Confirmación** (`/registro/confirmacion`)
   - Mostrar todos los datos ingresados
   - Botón "Confirmar" y "Editar"
   - Mensaje de éxito después de confirmar

3. **Página de Lista** (`/usuarios`)
   - Tabla con todos los usuarios registrados
   - Información básica en formato de cards o tabla

**Requisitos Técnicos:**
- ✅ Diseño responsivo con Bootstrap
- ✅ Validación HTML5 en el cliente
- ✅ Validación en el servidor
- ✅ Manejo de errores y mensajes de éxito
- ✅ Navegación clara entre páginas

**Entregables:**
- Código fuente completo
- Screenshots de todas las páginas
- Video corto (2-3 min) demostrando la funcionalidad
- Documento con reflexiones sobre dificultades encontradas

## 🚀 Avance en el Proyecto Final

### Diseño del Frontend
Aplicar lo aprendido al proyecto elegido en el módulo anterior.

#### Para Sistema de Clínica:
1. **Página de inicio** con dashboard básico
2. **Formulario de registro de pacientes**
3. **Formulario de agendar citas**
4. **Página de listado** (pacientes o citas)

#### Para Sistema POS:
1. **Página de inicio** con resumen de ventas
2. **Formulario de registro de productos**
3. **Formulario de nueva venta**
4. **Página de inventario**

#### Para Sistema de Reservas:
1. **Página de inicio** con espacios disponibles
2. **Formulario de registro de usuarios**
3. **Formulario de nueva reserva**
4. **Página de mis reservas**

### Tareas Específicas:
1. **Crear layout principal** con navegación
2. **Implementar 2-3 formularios** relacionados con tu dominio
3. **Usar Bootstrap** para diseño responsivo
4. **Agregar validación básica** en cliente y servidor
5. **Documentar el progreso** en `PROYECTO.md`

### Estructura Esperada:
```
MiProyectoFinal/
├── Controllers/
│   └── HomeController.cs
├── Views/
│   ├── Shared/
│   │   └── _Layout.cshtml
│   └── Home/
│       ├── Index.cshtml
│       ├── Formulario1.cshtml
│       └── Formulario2.cshtml
├── wwwroot/
│   └── css/
│       └── custom.css
└── ...
```

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
