# 05 - Controladores MVC con CRUD Completo

## 🎯 Objetivo de Aprendizaje
Al finalizar esta sección, el estudiante será capaz de:
- Implementar operaciones CRUD completas usando controladores MVC
- Crear vistas Razor para gestión de datos
- Manejar formularios con validaciones del lado servidor
- Aplicar el patrón MVC de forma correcta y eficiente

## 📚 Contenidos Principales

### 1. ¿Qué es CRUD?
**CRUD** son las cuatro operaciones básicas de cualquier sistema de gestión de datos:

- **C**reate (Crear) - Agregar nuevos registros
- **R**ead (Leer) - Mostrar/consultar datos existentes
- **U**pdate (Actualizar) - Modificar registros existentes
- **D**elete (Eliminar) - Borrar registros

**Mapeo en MVC:**
```csharp
// CREATE
[HttpGet]  public IActionResult Create()           // Mostrar formulario
[HttpPost] public IActionResult Create(Model obj)  // Procesar formulario

// READ
[HttpGet] public IActionResult Index()             // Listar todos
[HttpGet] public IActionResult Details(int id)     // Ver uno específico

// UPDATE
[HttpGet]  public IActionResult Edit(int id)       // Mostrar formulario con datos
[HttpPost] public IActionResult Edit(Model obj)    // Procesar actualización

// DELETE
[HttpGet]  public IActionResult Delete(int id)     // Mostrar confirmación
[HttpPost] public IActionResult DeleteConfirmed(int id) // Ejecutar eliminación
```

### 2. Patrón MVC Explicado
```
┌─────────────────┐    Request    ┌─────────────────┐    Query     ┌─────────────────┐
│      User       │ ────────────> │   Controller    │ ──────────> │   Database      │
│   (Browser)     │ <──────────── │     (C#)        │ <────────── │ (Entity Framework)│
└─────────────────┘   Response    └─────────────────┘   Data      └─────────────────┘
                                          │
                                          ▼ Model
                                  ┌─────────────────┐
                                  │      View       │
                                  │    (Razor)      │
                                  └─────────────────┘
```

## 🔧 Actividad Práctica Completa

### Prerequisitos: Verificar configuración del Módulo 04
**Asegúrate de tener del módulo anterior:**
```
SistemaClinicaMVC/
├── Data/
│   └── ClinicaContext.cs              ✅ DbContext configurado
├── Models/
│   └── Paciente.cs                    ✅ Modelo con validaciones
├── Migrations/                        ✅ Migraciones aplicadas
└── Program.cs                         ✅ EF Core configurado
```

**Verificación rápida:**
```bash
# 1. Verificar que la base de datos existe
dotnet ef database update

# 2. Ejecutar la aplicación
dotnet run

# 3. Debería abrir sin errores
```

---

## Paso 1: Crear el Controlador MVC con CRUD completo

```csharp
// Controllers/PacientesController.cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using SistemaClinicaMVC.Data;
using SistemaClinicaMVC.Models;

namespace SistemaClinicaMVC.Controllers
{
    public class PacientesController : Controller
    {
        private readonly ClinicaContext _context;
        private readonly ILogger<PacientesController> _logger;

        public PacientesController(ClinicaContext context, ILogger<PacientesController> logger)
        {
            _context = context;
            _logger = logger;
        }

        // GET: /Pacientes
        // READ - Mostrar lista de todos los pacientes
        public async Task<IActionResult> Index()
        {
            try
            {
                var pacientes = await _context.Pacientes
                    .OrderBy(p => p.Apellidos)
                    .ThenBy(p => p.Nombres)
                    .ToListAsync();

                _logger.LogInformation("Se cargaron {Count} pacientes", pacientes.Count);
                return View(pacientes);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar la lista de pacientes");
                TempData["Error"] = "Error al cargar la lista de pacientes";
                return View(new List<Paciente>());
            }
        }

        // GET: /Pacientes/Details/5
        // READ - Mostrar detalles de un paciente específico
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null)
            {
                return BadRequest("ID no proporcionado");
            }

            try
            {
                var paciente = await _context.Pacientes.FindAsync(id);

                if (paciente == null)
                {
                    _logger.LogWarning("Paciente con ID {Id} no encontrado", id);
                    return NotFound();
                }

                return View(paciente);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar paciente con ID {Id}", id);
                TempData["Error"] = "Error al cargar los detalles del paciente";
                return RedirectToAction(nameof(Index));
            }
        }

        // GET: /Pacientes/Create
        // CREATE - Mostrar formulario para crear nuevo paciente
        public IActionResult Create()
        {
            return View(new Paciente());
        }

        // POST: /Pacientes/Create
        // CREATE - Procesar la creación de un nuevo paciente
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create(Paciente paciente)
        {
            if (!ModelState.IsValid)
            {
                return View(paciente);
            }

            try
            {
                // Verificar si ya existe un paciente con la misma cédula
                var existePaciente = await _context.Pacientes
                    .AnyAsync(p => p.Cedula == paciente.Cedula);

                if (existePaciente)
                {
                    ModelState.AddModelError("Cedula", "Ya existe un paciente registrado con esta cédula");
                    return View(paciente);
                }

                // Verificar si ya existe un paciente con el mismo email
                if (!string.IsNullOrEmpty(paciente.Email))
                {
                    var existeEmail = await _context.Pacientes
                        .AnyAsync(p => p.Email == paciente.Email);

                    if (existeEmail)
                    {
                        ModelState.AddModelError("Email", "Ya existe un paciente registrado con este email");
                        return View(paciente);
                    }
                }

                _context.Pacientes.Add(paciente);
                await _context.SaveChangesAsync();

                _logger.LogInformation("Paciente creado exitosamente: {Nombres} {Apellidos} (ID: {Id})", 
                    paciente.Nombres, paciente.Apellidos, paciente.Id);

                TempData["Success"] = "Paciente creado exitosamente";
                return RedirectToAction(nameof(Index));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al crear paciente: {Nombres} {Apellidos}", 
                    paciente.Nombres, paciente.Apellidos);
                
                ModelState.AddModelError("", "Ocurrió un error al guardar el paciente. Por favor, intente nuevamente.");
                return View(paciente);
            }
        }

        // GET: /Pacientes/Edit/5
        // UPDATE - Mostrar formulario para editar paciente existente
        public async Task<IActionResult> Edit(int? id)
        {
            if (id == null)
            {
                return BadRequest("ID no proporcionado");
            }

            try
            {
                var paciente = await _context.Pacientes.FindAsync(id);

                if (paciente == null)
                {
                    _logger.LogWarning("Paciente con ID {Id} no encontrado para edición", id);
                    return NotFound();
                }

                return View(paciente);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar paciente para edición con ID {Id}", id);
                TempData["Error"] = "Error al cargar los datos del paciente";
                return RedirectToAction(nameof(Index));
            }
        }

        // POST: /Pacientes/Edit/5
        // UPDATE - Procesar la actualización del paciente
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Edit(int id, Paciente paciente)
        {
            if (id != paciente.Id)
            {
                return BadRequest("ID no coincide");
            }

            if (!ModelState.IsValid)
            {
                return View(paciente);
            }

            try
            {
                // Verificar si existe otro paciente con la misma cédula (excluyendo el actual)
                var existePaciente = await _context.Pacientes
                    .AnyAsync(p => p.Cedula == paciente.Cedula && p.Id != paciente.Id);

                if (existePaciente)
                {
                    ModelState.AddModelError("Cedula", "Ya existe otro paciente registrado con esta cédula");
                    return View(paciente);
                }

                // Verificar email duplicado (excluyendo el actual)
                if (!string.IsNullOrEmpty(paciente.Email))
                {
                    var existeEmail = await _context.Pacientes
                        .AnyAsync(p => p.Email == paciente.Email && p.Id != paciente.Id);

                    if (existeEmail)
                    {
                        ModelState.AddModelError("Email", "Ya existe otro paciente registrado con este email");
                        return View(paciente);
                    }
                }

                _context.Update(paciente);
                await _context.SaveChangesAsync();

                _logger.LogInformation("Paciente actualizado exitosamente: {Nombres} {Apellidos} (ID: {Id})", 
                    paciente.Nombres, paciente.Apellidos, paciente.Id);

                TempData["Success"] = "Paciente actualizado exitosamente";
                return RedirectToAction(nameof(Index));
            }
            catch (DbUpdateConcurrencyException ex)
            {
                if (!PacienteExists(paciente.Id))
                {
                    return NotFound();
                }

                _logger.LogError(ex, "Error de concurrencia al actualizar paciente con ID {Id}", id);
                ModelState.AddModelError("", "El paciente fue modificado por otro usuario. Por favor, recargue la página.");
                return View(paciente);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al actualizar paciente con ID {Id}", id);
                ModelState.AddModelError("", "Ocurrió un error al actualizar el paciente. Por favor, intente nuevamente.");
                return View(paciente);
            }
        }

        // GET: /Pacientes/Delete/5
        // DELETE - Mostrar confirmación de eliminación
        public async Task<IActionResult> Delete(int? id)
        {
            if (id == null)
            {
                return BadRequest("ID no proporcionado");
            }

            try
            {
                var paciente = await _context.Pacientes.FindAsync(id);

                if (paciente == null)
                {
                    _logger.LogWarning("Paciente con ID {Id} no encontrado para eliminación", id);
                    return NotFound();
                }

                return View(paciente);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar paciente para eliminación con ID {Id}", id);
                TempData["Error"] = "Error al cargar los datos del paciente";
                return RedirectToAction(nameof(Index));
            }
        }

        // POST: /Pacientes/Delete/5
        // DELETE - Ejecutar la eliminación
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int id)
        {
            try
            {
                var paciente = await _context.Pacientes.FindAsync(id);

                if (paciente == null)
                {
                    _logger.LogWarning("Paciente con ID {Id} no encontrado para eliminación", id);
                    return NotFound();
                }

                string nombreCompleto = $"{paciente.Nombres} {paciente.Apellidos}";

                _context.Pacientes.Remove(paciente);
                await _context.SaveChangesAsync();

                _logger.LogInformation("Paciente eliminado exitosamente: {NombreCompleto} (ID: {Id})", 
                    nombreCompleto, id);

                TempData["Success"] = $"Paciente {nombreCompleto} eliminado exitosamente";
                return RedirectToAction(nameof(Index));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al eliminar paciente con ID {Id}", id);
                TempData["Error"] = "Error al eliminar el paciente";
                return RedirectToAction(nameof(Index));
            }
        }

        // Método auxiliar para verificar si un paciente existe
        private bool PacienteExists(int id)
        {
            return _context.Pacientes.Any(p => p.Id == id);
        }
    }
}
```

---

## Paso 2: Crear las Vistas Razor

### Vista Index - Lista de Pacientes
```html
<!-- Views/Pacientes/Index.cshtml -->
@model IEnumerable<Paciente>
@{
    ViewData["Title"] = "Lista de Pacientes";
}

<div class="container-fluid py-4">
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h2 class="h3 mb-0">
            <i class="fas fa-users text-primary me-2"></i>
            Lista de Pacientes
        </h2>
        <a asp-action="Create" class="btn btn-primary">
            <i class="fas fa-plus me-1"></i>
            Nuevo Paciente
        </a>
    </div>

    @* Mostrar mensajes de éxito o error *@
    @if (TempData["Success"] != null)
    {
        <div class="alert alert-success alert-dismissible fade show" role="alert">
            <i class="fas fa-check-circle me-2"></i>@TempData["Success"]
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
    }

    @if (TempData["Error"] != null)
    {
        <div class="alert alert-danger alert-dismissible fade show" role="alert">
            <i class="fas fa-exclamation-triangle me-2"></i>@TempData["Error"]
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
    }

    @if (Model != null && Model.Any())
    {
        <div class="row">
            @foreach (var paciente in Model)
            {
                <div class="col-xl-4 col-lg-6 col-md-6 mb-4">
                    <div class="card shadow-sm h-100">
                        <div class="card-body">
                            <div class="d-flex justify-content-between align-items-start mb-3">
                                <h5 class="card-title mb-0 text-primary">
                                    @paciente.Nombres @paciente.Apellidos
                                </h5>
                                @if (!string.IsNullOrEmpty(paciente.TipoSangre))
                                {
                                    <span class="badge bg-info">@paciente.TipoSangre</span>
                                }
                            </div>

                            <div class="mb-2">
                                <small class="text-muted">
                                    <i class="fas fa-id-card me-1"></i>
                                    <strong>Cédula:</strong> @paciente.Cedula
                                </small>
                            </div>

                            <div class="mb-2">
                                <small class="text-muted">
                                    <i class="fas fa-birthday-cake me-1"></i>
                                    <strong>Edad:</strong> @paciente.Edad años
                                </small>
                            </div>

                            @if (!string.IsNullOrEmpty(paciente.Telefono))
                            {
                                <div class="mb-2">
                                    <small class="text-muted">
                                        <i class="fas fa-phone me-1"></i>
                                        <strong>Teléfono:</strong> @paciente.Telefono
                                    </small>
                                </div>
                            }

                            @if (!string.IsNullOrEmpty(paciente.Email))
                            {
                                <div class="mb-3">
                                    <small class="text-muted">
                                        <i class="fas fa-envelope me-1"></i>
                                        <strong>Email:</strong> @paciente.Email
                                    </small>
                                </div>
                            }
                        </div>

                        <div class="card-footer bg-light">
                            <div class="btn-group w-100" role="group">
                                <a asp-action="Details" asp-route-id="@paciente.Id" 
                                   class="btn btn-outline-info btn-sm">
                                    <i class="fas fa-eye"></i> Ver
                                </a>
                                <a asp-action="Edit" asp-route-id="@paciente.Id" 
                                   class="btn btn-outline-warning btn-sm">
                                    <i class="fas fa-edit"></i> Editar
                                </a>
                                <a asp-action="Delete" asp-route-id="@paciente.Id" 
                                   class="btn btn-outline-danger btn-sm">
                                    <i class="fas fa-trash"></i> Eliminar
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            }
        </div>
    }
    else
    {
        <div class="text-center py-5">
            <i class="fas fa-users fa-4x text-muted mb-4"></i>
            <h4 class="text-muted mb-3">No hay pacientes registrados</h4>
            <p class="text-muted mb-4">Comience agregando su primer paciente al sistema</p>
            <a asp-action="Create" class="btn btn-primary btn-lg">
                <i class="fas fa-plus me-2"></i>
                Registrar Primer Paciente
            </a>
        </div>
    }
</div>
```

### Vista Create - Formulario de Creación
```html
<!-- Views/Pacientes/Create.cshtml -->
@model Paciente
@{
    ViewData["Title"] = "Crear Paciente";
}

<div class="container py-4">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card shadow">
                <div class="card-header bg-primary text-white">
                    <h4 class="mb-0">
                        <i class="fas fa-user-plus me-2"></i>
                        Registrar Nuevo Paciente
                    </h4>
                </div>
                <div class="card-body">
                    <form asp-action="Create" method="post">
                        <div asp-validation-summary="ModelOnly" class="alert alert-danger"></div>

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="Nombres" class="form-label required">Nombres</label>
                                <input asp-for="Nombres" class="form-control" placeholder="Ingrese los nombres">
                                <span asp-validation-for="Nombres" class="text-danger"></span>
                            </div>
                            <div class="col-md-6 mb-3">
                                <label asp-for="Apellidos" class="form-label required">Apellidos</label>
                                <input asp-for="Apellidos" class="form-control" placeholder="Ingrese los apellidos">
                                <span asp-validation-for="Apellidos" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="Cedula" class="form-label required">Cédula</label>
                                <input asp-for="Cedula" class="form-control" placeholder="Ej: 12345678" maxlength="8">
                                <div class="form-text">Ingrese 8 dígitos numéricos</div>
                                <span asp-validation-for="Cedula" class="text-danger"></span>
                            </div>
                            <div class="col-md-6 mb-3">
                                <label asp-for="FechaNacimiento" class="form-label required">Fecha de Nacimiento</label>
                                <input asp-for="FechaNacimiento" class="form-control" type="date">
                                <span asp-validation-for="FechaNacimiento" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="Telefono" class="form-label">Teléfono</label>
                                <input asp-for="Telefono" class="form-control" placeholder="Ej: 0987654321">
                                <span asp-validation-for="Telefono" class="text-danger"></span>
                            </div>
                            <div class="col-md-6 mb-3">
                                <label asp-for="Email" class="form-label">Email</label>
                                <input asp-for="Email" class="form-control" type="email" placeholder="ejemplo@correo.com">
                                <span asp-validation-for="Email" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="TipoSangre" class="form-label">Tipo de Sangre</label>
                                <select asp-for="TipoSangre" class="form-select">
                                    <option value="">-- Seleccionar --</option>
                                    <option value="A+">A+</option>
                                    <option value="A-">A-</option>
                                    <option value="B+">B+</option>
                                    <option value="B-">B-</option>
                                    <option value="AB+">AB+</option>
                                    <option value="AB-">AB-</option>
                                    <option value="O+">O+</option>
                                    <option value="O-">O-</option>
                                </select>
                                <span asp-validation-for="TipoSangre" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="mb-3">
                            <label asp-for="Direccion" class="form-label">Dirección</label>
                            <textarea asp-for="Direccion" class="form-control" rows="3" 
                                      placeholder="Dirección completa del paciente"></textarea>
                            <span asp-validation-for="Direccion" class="text-danger"></span>
                        </div>

                        <div class="d-flex justify-content-end gap-2 mt-4">
                            <a asp-action="Index" class="btn btn-secondary">
                                <i class="fas fa-times me-1"></i>
                                Cancelar
                            </a>
                            <button type="submit" class="btn btn-primary">
                                <i class="fas fa-save me-1"></i>
                                Guardar Paciente
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
    
    <script>
        // Validación de cédula en tiempo real
        document.getElementById('Cedula').addEventListener('input', function(e) {
            const value = e.target.value;
            if (value && !/^\d{1,8}$/.test(value)) {
                e.target.setCustomValidity('Solo se permiten números (máximo 8 dígitos)');
            } else {
                e.target.setCustomValidity('');
            }
        });
    </script>
}

<style>
    .required::after {
        content: " *";
        color: red;
        font-weight: bold;
    }
</style>
```

### Vista Edit - Formulario de Edición
```html
<!-- Views/Pacientes/Edit.cshtml -->
@model Paciente
@{
    ViewData["Title"] = "Editar Paciente";
}

<div class="container py-4">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card shadow">
                <div class="card-header bg-warning text-dark">
                    <h4 class="mb-0">
                        <i class="fas fa-edit me-2"></i>
                        Editar Paciente: @Model.Nombres @Model.Apellidos
                    </h4>
                </div>
                <div class="card-body">
                    <form asp-action="Edit" method="post">
                        <div asp-validation-summary="ModelOnly" class="alert alert-danger"></div>
                        
                        <input asp-for="Id" type="hidden">

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="Nombres" class="form-label required">Nombres</label>
                                <input asp-for="Nombres" class="form-control">
                                <span asp-validation-for="Nombres" class="text-danger"></span>
                            </div>
                            <div class="col-md-6 mb-3">
                                <label asp-for="Apellidos" class="form-label required">Apellidos</label>
                                <input asp-for="Apellidos" class="form-control">
                                <span asp-validation-for="Apellidos" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="Cedula" class="form-label required">Cédula</label>
                                <input asp-for="Cedula" class="form-control" maxlength="8">
                                <div class="form-text">Ingrese 8 dígitos numéricos</div>
                                <span asp-validation-for="Cedula" class="text-danger"></span>
                            </div>
                            <div class="col-md-6 mb-3">
                                <label asp-for="FechaNacimiento" class="form-label required">Fecha de Nacimiento</label>
                                <input asp-for="FechaNacimiento" class="form-control" type="date">
                                <span asp-validation-for="FechaNacimiento" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="Telefono" class="form-label">Teléfono</label>
                                <input asp-for="Telefono" class="form-control">
                                <span asp-validation-for="Telefono" class="text-danger"></span>
                            </div>
                            <div class="col-md-6 mb-3">
                                <label asp-for="Email" class="form-label">Email</label>
                                <input asp-for="Email" class="form-control" type="email">
                                <span asp-validation-for="Email" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label asp-for="TipoSangre" class="form-label">Tipo de Sangre</label>
                                <select asp-for="TipoSangre" class="form-select">
                                    <option value="">-- Seleccionar --</option>
                                    <option value="A+">A+</option>
                                    <option value="A-">A-</option>
                                    <option value="B+">B+</option>
                                    <option value="B-">B-</option>
                                    <option value="AB+">AB+</option>
                                    <option value="AB-">AB-</option>
                                    <option value="O+">O+</option>
                                    <option value="O-">O-</option>
                                </select>
                                <span asp-validation-for="TipoSangre" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="mb-3">
                            <label asp-for="Direccion" class="form-label">Dirección</label>
                            <textarea asp-for="Direccion" class="form-control" rows="3"></textarea>
                            <span asp-validation-for="Direccion" class="text-danger"></span>
                        </div>

                        <div class="d-flex justify-content-end gap-2 mt-4">
                            <a asp-action="Index" class="btn btn-secondary">
                                <i class="fas fa-times me-1"></i>
                                Cancelar
                            </a>
                            <button type="submit" class="btn btn-warning">
                                <i class="fas fa-save me-1"></i>
                                Actualizar Paciente
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}

<style>
    .required::after {
        content: " *";
        color: red;
        font-weight: bold;
    }
</style>
```

### Vista Details - Ver Detalles
```html
<!-- Views/Pacientes/Details.cshtml -->
@model Paciente
@{
    ViewData["Title"] = "Detalles del Paciente";
}

<div class="container py-4">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card shadow">
                <div class="card-header bg-info text-white">
                    <h4 class="mb-0">
                        <i class="fas fa-user me-2"></i>
                        Detalles del Paciente
                    </h4>
                </div>
                <div class="card-body">
                    <div class="row">
                        <div class="col-md-6">
                            <h5 class="text-primary">@Model.Nombres @Model.Apellidos</h5>
                            
                            <div class="mb-3">
                                <label class="form-label fw-bold">Cédula:</label>
                                <p class="form-control-plaintext">@Model.Cedula</p>
                            </div>

                            <div class="mb-3">
                                <label class="form-label fw-bold">Fecha de Nacimiento:</label>
                                <p class="form-control-plaintext">@Model.FechaNacimiento.ToString("dd/MM/yyyy")</p>
                            </div>

                            <div class="mb-3">
                                <label class="form-label fw-bold">Edad:</label>
                                <p class="form-control-plaintext">
                                    <span class="badge bg-primary fs-6">@Model.Edad años</span>
                                </p>
                            </div>
                        </div>

                        <div class="col-md-6">
                            @if (!string.IsNullOrEmpty(Model.Telefono))
                            {
                                <div class="mb-3">
                                    <label class="form-label fw-bold">Teléfono:</label>
                                    <p class="form-control-plaintext">
                                        <i class="fas fa-phone text-success me-1"></i>
                                        @Model.Telefono
                                    </p>
                                </div>
                            }

                            @if (!string.IsNullOrEmpty(Model.Email))
                            {
                                <div class="mb-3">
                                    <label class="form-label fw-bold">Email:</label>
                                    <p class="form-control-plaintext">
                                        <i class="fas fa-envelope text-info me-1"></i>
                                        <a href="mailto:@Model.Email">@Model.Email</a>
                                    </p>
                                </div>
                            }

                            @if (!string.IsNullOrEmpty(Model.TipoSangre))
                            {
                                <div class="mb-3">
                                    <label class="form-label fw-bold">Tipo de Sangre:</label>
                                    <p class="form-control-plaintext">
                                        <span class="badge bg-danger fs-6">@Model.TipoSangre</span>
                                    </p>
                                </div>
                            }
                        </div>
                    </div>

                    @if (!string.IsNullOrEmpty(Model.Direccion))
                    {
                        <div class="row">
                            <div class="col-12">
                                <div class="mb-3">
                                    <label class="form-label fw-bold">Dirección:</label>
                                    <p class="form-control-plaintext">
                                        <i class="fas fa-map-marker-alt text-warning me-1"></i>
                                        @Model.Direccion
                                    </p>
                                </div>
                            </div>
                        </div>
                    }

                    <hr>
                    
                    <div class="d-flex justify-content-between">
                        <div>
                            <a asp-action="Index" class="btn btn-secondary">
                                <i class="fas fa-arrow-left me-1"></i>
                                Volver a la Lista
                            </a>
                        </div>
                        <div>
                            <a asp-action="Edit" asp-route-id="@Model.Id" class="btn btn-warning me-2">
                                <i class="fas fa-edit me-1"></i>
                                Editar
                            </a>
                            <a asp-action="Delete" asp-route-id="@Model.Id" class="btn btn-danger">
                                <i class="fas fa-trash me-1"></i>
                                Eliminar
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### Vista Delete - Confirmación de Eliminación
```html
<!-- Views/Pacientes/Delete.cshtml -->
@model Paciente
@{
    ViewData["Title"] = "Eliminar Paciente";
}

<div class="container py-4">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card shadow border-danger">
                <div class="card-header bg-danger text-white">
                    <h4 class="mb-0">
                        <i class="fas fa-exclamation-triangle me-2"></i>
                        Confirmar Eliminación
                    </h4>
                </div>
                <div class="card-body text-center">
                    <i class="fas fa-user-times fa-4x text-danger mb-4"></i>
                    
                    <h5 class="mb-3">¿Está seguro de que desea eliminar este paciente?</h5>
                    
                    <div class="alert alert-warning">
                        <strong>Esta acción no se puede deshacer</strong>
                    </div>

                    <div class="card bg-light">
                        <div class="card-body">
                            <h6 class="text-primary">@Model.Nombres @Model.Apellidos</h6>
                            <p class="mb-1"><strong>Cédula:</strong> @Model.Cedula</p>
                            <p class="mb-1"><strong>Edad:</strong> @Model.Edad años</p>
                            @if (!string.IsNullOrEmpty(Model.Telefono))
                            {
                                <p class="mb-0"><strong>Teléfono:</strong> @Model.Telefono</p>
                            }
                        </div>
                    </div>

                    <form asp-action="Delete" method="post" class="mt-4">
                        <div class="d-flex justify-content-center gap-3">
                            <a asp-action="Index" class="btn btn-secondary btn-lg">
                                <i class="fas fa-times me-1"></i>
                                Cancelar
                            </a>
                            <button type="submit" class="btn btn-danger btn-lg">
                                <i class="fas fa-trash me-1"></i>
                                Eliminar Definitivamente
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
```

---

## Paso 3: Actualizar la navegación

### Agregar menú en _Layout.cshtml
```html
<!-- Views/Shared/_Layout.cshtml - Actualizar la sección de navegación -->
<div class="navbar-nav">
    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Inicio</a>
    <a class="nav-link text-dark" asp-area="" asp-controller="Pacientes" asp-action="Index">Pacientes</a>
</div>
```

---

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Probar el CRUD completo
1. Ejecuta la aplicación: `dotnet run`
2. Navega a `/Pacientes`
3. Crea 5 pacientes con datos diferentes
4. Edita uno de ellos
5. Ve los detalles de otro
6. Elimina uno

### Ejercicio 2: Mejorar las validaciones
Agrega validaciones personalizadas:

```csharp
// En el modelo Paciente.cs
[RegularExpression(@"^\d{8}$", ErrorMessage = "La cédula debe tener exactamente 8 dígitos")]
public string Cedula { get; set; }

[Range(18, 120, ErrorMessage = "La edad debe estar entre 18 y 120 años")]
public int Edad => DateTime.Now.Year - FechaNacimiento.Year;
```

### Ejercicio 3: Agregar búsqueda
Implementa un campo de búsqueda en la vista Index:

```csharp
// En el controlador
public async Task<IActionResult> Index(string searchString)
{
    var pacientes = from p in _context.Pacientes select p;
    
    if (!String.IsNullOrEmpty(searchString))
    {
        pacientes = pacientes.Where(p => p.Nombres.Contains(searchString) 
                                      || p.Apellidos.Contains(searchString)
                                      || p.Cedula.Contains(searchString));
    }
    
    return View(await pacientes.OrderBy(p => p.Apellidos).ToListAsync());
}
```

---

## 🔍 Debugging y Troubleshooting

### Errores comunes:

**1. Error "DbContext no registrado":**
- Verifica que `AddDbContext` esté en Program.cs
- Confirma que la cadena de conexión sea correcta

**2. Error de validación en formularios:**
- Asegúrate de que `ModelState.IsValid` se evalúe correctamente
- Verifica que las validaciones del modelo estén bien configuradas

**3. Error 404 en rutas:**
- Confirma que el controlador tenga el nombre correcto
- Verifica que las vistas estén en la carpeta correcta

**4. Problemas con fechas:**
- Usa `type="date"` en inputs de fecha
- Configura formato de fecha en el modelo si es necesario

---

## 📖 Conceptos Clave Aprendidos

### 1. Patrón MVC
- **Modelo**: Representa los datos y lógica de negocio
- **Vista**: Presenta la interfaz de usuario
- **Controlador**: Maneja la lógica de aplicación y coordinación

### 2. Entity Framework Core
- **DbContext**: Punto de entrada para interactuar con la base de datos
- **LINQ**: Consultas expresivas y legibles
- **Async/Await**: Operaciones no bloqueantes

### 3. Razor Views
- **Tag Helpers**: Sintaxis limpia para formularios
- **Model Binding**: Enlace automático entre vista y modelo
- **Validaciones**: Feedback automático de errores

---

## 🎯 Evaluación del Módulo

### Criterios según la rúbrica:

**Base de datos (15 puntos):**
- ✅ Atributos coherentes y relaciones necesarias
- ✅ Migraciones aplicadas correctamente

**Mantenedores (30 puntos):**
- ✅ CRUD completo: Crear, Modificar, Eliminar, Listar
- ✅ Todas las operaciones funcionando correctamente

**Validaciones (30 puntos):**
- ✅ Validación de campos vacíos
- ✅ Validación de espacios y formato
- ✅ Validación de números (cédula)
- ✅ Validación de fechas y email
- ✅ Validación de duplicados (cédula, email)

**Menú (7 puntos):**
- ✅ Acceso directo a los mantenedores

**Estilos (5 puntos):**
- ✅ Sistema profesional con Bootstrap
- ✅ Interfaz limpia y consistente

---

## ➡️ Próximo módulo
**Módulo 06: Validaciones Avanzadas y DTOs** - Aprenderás a implementar validaciones más complejas y crear modelos de transferencia de datos para mejorar la arquitectura de tu aplicación.