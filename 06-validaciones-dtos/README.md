# 06 - Validaciones Avanzadas y DTOs

## 🎯 Objetivo de Aprendizaje
Al finalizar esta sección, el estudiante será capaz de:
- Implementar validaciones personalizadas complejas
- Crear y usar Data Transfer Objects (DTOs)
- Aplicar atributos de validación avanzados
- Validar relaciones entre campos
- Implementar validaciones del lado cliente y servidor

## 📚 Contenidos Principales

### 1. ¿Por qué necesitamos validaciones avanzadas?

**Validaciones básicas vs. avanzadas:**

| Tipo | Ejemplo | Cuándo usar |
|------|---------|-------------|
| **Básicas** | `[Required]`, `[StringLength]` | Campos individuales simples |
| **Avanzadas** | Validar que fecha inicio < fecha fin | Relaciones entre campos |
| **Personalizadas** | Validar RUN chileno | Lógica de negocio específica |

**Escenarios reales que necesitan validaciones avanzadas:**
- ✅ Validar que la fecha de nacimiento no sea futura
- ✅ Validar formato específico de cédula o RUN
- ✅ Validar que un email no esté duplicado en tiempo real
- ✅ Validar rangos de edad (ej: mayor de 18 años)
- ✅ Validar combinaciones de campos

### 2. ¿Qué son los DTOs?

**DTO (Data Transfer Object)** = Objeto para transferir datos entre capas

**Sin DTO:**
```
Vista → Modelo de BD directamente → Base de Datos
```

**Con DTO:**
```
Vista → DTO → Mapeo → Modelo de BD → Base de Datos
```

**Ventajas de usar DTOs:**
- ✅ **Seguridad**: No expones la estructura interna de tu BD
- ✅ **Flexibilidad**: Puedes enviar/recibir solo los campos necesarios
- ✅ **Validaciones específicas**: Validaciones diferentes para crear vs editar
- ✅ **Versionamiento**: Puedes tener diferentes DTOs para diferentes versiones

**Ejemplo práctico:**
```csharp
// Modelo de BD - Tiene TODO
public class Paciente 
{
    public int Id { get; set; }
    public string Nombres { get; set; }
    public DateTime FechaCreacion { get; set; }  // Usuario NO debe modificar esto
    public string PasswordHash { get; set; }      // Usuario NO debe ver esto
}

// DTO para crear - Solo lo que el usuario proporciona
public class CrearPacienteDto 
{
    public string Nombres { get; set; }
    // NO incluye Id, FechaCreacion, ni PasswordHash
}

// DTO para editar - Puede incluir Id pero no campos sensibles
public class EditarPacienteDto 
{
    public int Id { get; set; }
    public string Nombres { get; set; }
    // NO incluye campos sensibles
}
```

---

## 🔧 Actividad Práctica Completa

### Prerequisitos: Módulo 05 completado
**Debes tener funcionando:**
- ✅ CRUD básico de Pacientes
- ✅ Validaciones simples con Data Annotations
- ✅ Entity Framework configurado

---

## Paso 1: Crear atributos de validación personalizados

### Validación de RUN Chileno
```csharp
// Validations/ValidarRunAttribute.cs - Crear carpeta Validations
using System.ComponentModel.DataAnnotations;

namespace SistemaClinicaMVC.Validations
{
    /// <summary>
    /// Valida que el RUN chileno sea correcto (incluye dígito verificador)
    /// Ejemplo: 12.345.678-5
    /// </summary>
    public class ValidarRunAttribute : ValidationAttribute
    {
        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            if (value == null || string.IsNullOrWhiteSpace(value.ToString()))
            {
                // Si es null o vacío, dejar que [Required] lo maneje
                return ValidationResult.Success;
            }

            string run = value.ToString().Replace(".", "").Replace("-", "");

            // Debe tener entre 8 y 9 caracteres (7-8 dígitos + verificador)
            if (run.Length < 8 || run.Length > 9)
            {
                return new ValidationResult("RUN debe tener formato válido (ej: 12345678-5)");
            }

            // Separar número y dígito verificador
            string numero = run.Substring(0, run.Length - 1);
            char digitoVerificador = run[run.Length - 1];

            // Validar que la parte numérica sea válida
            if (!int.TryParse(numero, out int runNumero))
            {
                return new ValidationResult("RUN contiene caracteres inválidos");
            }

            // Calcular dígito verificador
            int suma = 0;
            int multiplicador = 2;

            for (int i = numero.Length - 1; i >= 0; i--)
            {
                suma += int.Parse(numero[i].ToString()) * multiplicador;
                multiplicador = multiplicador == 7 ? 2 : multiplicador + 1;
            }

            int resto = suma % 11;
            int digitoEsperado = 11 - resto;
            char digitoEsperadoChar;

            if (digitoEsperado == 11)
                digitoEsperadoChar = '0';
            else if (digitoEsperado == 10)
                digitoEsperadoChar = 'K';
            else
                digitoEsperadoChar = digitoEsperado.ToString()[0];

            // Comparar (case insensitive para la K)
            if (char.ToUpper(digitoVerificador) != char.ToUpper(digitoEsperadoChar))
            {
                return new ValidationResult($"RUN inválido. Dígito verificador debería ser {digitoEsperadoChar}");
            }

            return ValidationResult.Success;
        }
    }
}
```

### Validación de fecha de nacimiento
```csharp
// Validations/EdadMinimaAttribute.cs
using System.ComponentModel.DataAnnotations;

namespace SistemaClinicaMVC.Validations
{
    /// <summary>
    /// Valida que la persona tenga una edad mínima
    /// </summary>
    public class EdadMinimaAttribute : ValidationAttribute
    {
        private readonly int _edadMinima;

        public EdadMinimaAttribute(int edadMinima)
        {
            _edadMinima = edadMinima;
            ErrorMessage = $"Debe tener al menos {edadMinima} años";
        }

        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            if (value == null)
            {
                return ValidationResult.Success;
            }

            if (value is DateTime fechaNacimiento)
            {
                var edad = DateTime.Today.Year - fechaNacimiento.Year;
                
                // Ajustar si aún no ha cumplido años este año
                if (fechaNacimiento.Date > DateTime.Today.AddYears(-edad))
                {
                    edad--;
                }

                if (edad < _edadMinima)
                {
                    return new ValidationResult(ErrorMessage);
                }
            }

            return ValidationResult.Success;
        }
    }
}
```

### Validación de fecha no futura
```csharp
// Validations/FechaNoFuturaAttribute.cs
using System.ComponentModel.DataAnnotations;

namespace SistemaClinicaMVC.Validations
{
    /// <summary>
    /// Valida que la fecha no sea posterior a la fecha actual
    /// </summary>
    public class FechaNoFuturaAttribute : ValidationAttribute
    {
        public FechaNoFuturaAttribute()
        {
            ErrorMessage = "La fecha no puede ser futura";
        }

        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            if (value == null)
            {
                return ValidationResult.Success;
            }

            if (value is DateTime fecha)
            {
                if (fecha.Date > DateTime.Today)
                {
                    return new ValidationResult(ErrorMessage);
                }
            }

            return ValidationResult.Success;
        }
    }
}
```

### Validación de teléfono chileno
```csharp
// Validations/TelefonoChilenoAttribute.cs
using System.ComponentModel.DataAnnotations;
using System.Text.RegularExpressions;

namespace SistemaClinicaMVC.Validations
{
    /// <summary>
    /// Valida formato de teléfono chileno
    /// Acepta: +56912345678, 912345678, +569 1234 5678, etc.
    /// </summary>
    public class TelefonoChilenoAttribute : ValidationAttribute
    {
        public TelefonoChilenoAttribute()
        {
            ErrorMessage = "Formato de teléfono inválido. Use: +56912345678 o 912345678";
        }

        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            if (value == null || string.IsNullOrWhiteSpace(value.ToString()))
            {
                return ValidationResult.Success;
            }

            string telefono = value.ToString().Replace(" ", "").Replace("-", "");

            // Patrón: +56 o 56 seguido de 9 dígitos, o directamente 9 dígitos comenzando con 9
            var patron = @"^(\+?56)?9\d{8}$";

            if (!Regex.IsMatch(telefono, patron))
            {
                return new ValidationResult(ErrorMessage);
            }

            return ValidationResult.Success;
        }
    }
}
```

---

## Paso 2: Crear DTOs para Pacientes

```csharp
// DTOs/PacienteDto.cs - Crear carpeta DTOs
using System.ComponentModel.DataAnnotations;
using SistemaClinicaMVC.Validations;

namespace SistemaClinicaMVC.DTOs
{
    /// <summary>
    /// DTO base con propiedades comunes
    /// </summary>
    public class PacienteBaseDto
    {
        [Required(ErrorMessage = "Los nombres son obligatorios")]
        [StringLength(100, MinimumLength = 2, ErrorMessage = "Los nombres deben tener entre 2 y 100 caracteres")]
        [RegularExpression(@"^[a-zA-ZáéíóúÁÉÍÓÚñÑ\s]+$", ErrorMessage = "Los nombres solo pueden contener letras")]
        [Display(Name = "Nombres")]
        public string Nombres { get; set; }

        [Required(ErrorMessage = "Los apellidos son obligatorios")]
        [StringLength(100, MinimumLength = 2, ErrorMessage = "Los apellidos deben tener entre 2 y 100 caracteres")]
        [RegularExpression(@"^[a-zA-ZáéíóúÁÉÍÓÚñÑ\s]+$", ErrorMessage = "Los apellidos solo pueden contener letras")]
        [Display(Name = "Apellidos")]
        public string Apellidos { get; set; }

        [Required(ErrorMessage = "El RUN es obligatorio")]
        [ValidarRun]
        [Display(Name = "RUN")]
        public string Cedula { get; set; }

        [Required(ErrorMessage = "La fecha de nacimiento es obligatoria")]
        [DataType(DataType.Date)]
        [FechaNoFutura]
        [EdadMinima(18)]
        [Display(Name = "Fecha de Nacimiento")]
        public DateTime FechaNacimiento { get; set; }

        [TelefonoChileno]
        [Display(Name = "Teléfono")]
        public string Telefono { get; set; }

        [EmailAddress(ErrorMessage = "Formato de email inválido")]
        [StringLength(200, ErrorMessage = "El email no puede tener más de 200 caracteres")]
        [Display(Name = "Email")]
        public string Email { get; set; }

        [StringLength(20, ErrorMessage = "El tipo de sangre no puede tener más de 20 caracteres")]
        [RegularExpression(@"^(A|B|AB|O)[+-]$", ErrorMessage = "Tipo de sangre inválido. Use formato: A+, A-, B+, B-, AB+, AB-, O+, O-")]
        [Display(Name = "Tipo de Sangre")]
        public string TipoSangre { get; set; }

        [StringLength(500, ErrorMessage = "La dirección no puede tener más de 500 caracteres")]
        [Display(Name = "Dirección")]
        public string Direccion { get; set; }
    }

    /// <summary>
    /// DTO para crear un nuevo paciente
    /// No incluye Id (se genera automáticamente)
    /// </summary>
    public class CrearPacienteDto : PacienteBaseDto
    {
        // Hereda todas las propiedades de PacienteBaseDto
        // No incluye Id porque es auto-generado
    }

    /// <summary>
    /// DTO para editar un paciente existente
    /// Incluye Id para identificar el registro
    /// </summary>
    public class EditarPacienteDto : PacienteBaseDto
    {
        [Required]
        public int Id { get; set; }
    }

    /// <summary>
    /// DTO para mostrar información del paciente (solo lectura)
    /// Incluye propiedades calculadas como Edad
    /// </summary>
    public class PacienteDetalleDto
    {
        public int Id { get; set; }
        public string Nombres { get; set; }
        public string Apellidos { get; set; }
        public string NombreCompleto => $"{Nombres} {Apellidos}";
        public string Cedula { get; set; }
        public DateTime FechaNacimiento { get; set; }
        public int Edad => DateTime.Today.Year - FechaNacimiento.Year - 
            (FechaNacimiento.Date > DateTime.Today.AddYears(-(DateTime.Today.Year - FechaNacimiento.Year)) ? 1 : 0);
        public string Telefono { get; set; }
        public string Email { get; set; }
        public string TipoSangre { get; set; }
        public string Direccion { get; set; }
    }
}
```

---

## Paso 3: Crear servicio de mapeo entre DTOs y Modelos

```csharp
// Services/PacienteMapper.cs - Crear carpeta Services
using SistemaClinicaMVC.Models;
using SistemaClinicaMVC.DTOs;

namespace SistemaClinicaMVC.Services
{
    /// <summary>
    /// Servicio para mapear entre modelos y DTOs
    /// </summary>
    public static class PacienteMapper
    {
        // Mapear de CrearPacienteDto a Paciente (para insertar)
        public static Paciente ToModel(this CrearPacienteDto dto)
        {
            return new Paciente
            {
                Nombres = dto.Nombres.Trim(),
                Apellidos = dto.Apellidos.Trim(),
                Cedula = dto.Cedula.Replace(".", "").Replace("-", "").Trim(),
                FechaNacimiento = dto.FechaNacimiento,
                Telefono = dto.Telefono?.Trim(),
                Email = dto.Email?.Trim(),
                TipoSangre = dto.TipoSangre?.Trim(),
                Direccion = dto.Direccion?.Trim()
            };
        }

        // Mapear de EditarPacienteDto a Paciente (para actualizar)
        public static Paciente ToModel(this EditarPacienteDto dto)
        {
            return new Paciente
            {
                Id = dto.Id,
                Nombres = dto.Nombres.Trim(),
                Apellidos = dto.Apellidos.Trim(),
                Cedula = dto.Cedula.Replace(".", "").Replace("-", "").Trim(),
                FechaNacimiento = dto.FechaNacimiento,
                Telefono = dto.Telefono?.Trim(),
                Email = dto.Email?.Trim(),
                TipoSangre = dto.TipoSangre?.Trim(),
                Direccion = dto.Direccion?.Trim()
            };
        }

        // Mapear de Paciente a PacienteDetalleDto (para mostrar)
        public static PacienteDetalleDto ToDetalleDto(this Paciente model)
        {
            return new PacienteDetalleDto
            {
                Id = model.Id,
                Nombres = model.Nombres,
                Apellidos = model.Apellidos,
                Cedula = FormatearRun(model.Cedula),
                FechaNacimiento = model.FechaNacimiento,
                Telefono = model.Telefono,
                Email = model.Email,
                TipoSangre = model.TipoSangre,
                Direccion = model.Direccion
            };
        }

        // Mapear de Paciente a EditarPacienteDto (para editar)
        public static EditarPacienteDto ToEditarDto(this Paciente model)
        {
            return new EditarPacienteDto
            {
                Id = model.Id,
                Nombres = model.Nombres,
                Apellidos = model.Apellidos,
                Cedula = FormatearRun(model.Cedula),
                FechaNacimiento = model.FechaNacimiento,
                Telefono = model.Telefono,
                Email = model.Email,
                TipoSangre = model.TipoSangre,
                Direccion = model.Direccion
            };
        }

        // Método auxiliar para formatear RUN
        private static string FormatearRun(string run)
        {
            if (string.IsNullOrWhiteSpace(run))
                return run;

            // Eliminar formato existente
            run = run.Replace(".", "").Replace("-", "");

            if (run.Length < 2)
                return run;

            // Formatear como 12.345.678-9
            string numero = run.Substring(0, run.Length - 1);
            string dv = run.Substring(run.Length - 1);

            // Agregar puntos cada 3 dígitos de derecha a izquierda
            string numeroFormateado = "";
            int contador = 0;
            for (int i = numero.Length - 1; i >= 0; i--)
            {
                if (contador == 3)
                {
                    numeroFormateado = "." + numeroFormateado;
                    contador = 0;
                }
                numeroFormateado = numero[i] + numeroFormateado;
                contador++;
            }

            return $"{numeroFormateado}-{dv}";
        }
    }
}
```

---

## Paso 4: Actualizar el Controlador para usar DTOs

```csharp
// Controllers/PacientesController.cs - ACTUALIZAR
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using SistemaClinicaMVC.Data;
using SistemaClinicaMVC.Models;
using SistemaClinicaMVC.DTOs;
using SistemaClinicaMVC.Services;

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
        public async Task<IActionResult> Index()
        {
            try
            {
                var pacientes = await _context.Pacientes
                    .OrderBy(p => p.Apellidos)
                    .ThenBy(p => p.Nombres)
                    .Select(p => p.ToDetalleDto()) // Usar DTO para mostrar
                    .ToListAsync();

                _logger.LogInformation("Se cargaron {Count} pacientes", pacientes.Count);
                return View(pacientes);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar la lista de pacientes");
                TempData["Error"] = "Error al cargar la lista de pacientes";
                return View(new List<PacienteDetalleDto>());
            }
        }

        // GET: /Pacientes/Details/5
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

                var pacienteDto = paciente.ToDetalleDto();
                return View(pacienteDto);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar paciente con ID {Id}", id);
                TempData["Error"] = "Error al cargar los detalles del paciente";
                return RedirectToAction(nameof(Index));
            }
        }

        // GET: /Pacientes/Create
        public IActionResult Create()
        {
            return View(new CrearPacienteDto());
        }

        // POST: /Pacientes/Create
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create(CrearPacienteDto dto)
        {
            if (!ModelState.IsValid)
            {
                return View(dto);
            }

            try
            {
                // Normalizar cédula para comparación
                string cedulaNormalizada = dto.Cedula.Replace(".", "").Replace("-", "").Trim();

                // Verificar duplicados
                var existePaciente = await _context.Pacientes
                    .AnyAsync(p => p.Cedula == cedulaNormalizada);

                if (existePaciente)
                {
                    ModelState.AddModelError("Cedula", "Ya existe un paciente registrado con este RUN");
                    return View(dto);
                }

                if (!string.IsNullOrEmpty(dto.Email))
                {
                    var existeEmail = await _context.Pacientes
                        .AnyAsync(p => p.Email == dto.Email.Trim());

                    if (existeEmail)
                    {
                        ModelState.AddModelError("Email", "Ya existe un paciente registrado con este email");
                        return View(dto);
                    }
                }

                // Mapear DTO a modelo
                var paciente = dto.ToModel();

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
                    dto.Nombres, dto.Apellidos);
                
                ModelState.AddModelError("", "Ocurrió un error al guardar el paciente. Por favor, intente nuevamente.");
                return View(dto);
            }
        }

        // GET: /Pacientes/Edit/5
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

                var dto = paciente.ToEditarDto();
                return View(dto);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar paciente para edición con ID {Id}", id);
                TempData["Error"] = "Error al cargar los datos del paciente";
                return RedirectToAction(nameof(Index));
            }
        }

        // POST: /Pacientes/Edit/5
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Edit(int id, EditarPacienteDto dto)
        {
            if (id != dto.Id)
            {
                return BadRequest("ID no coincide");
            }

            if (!ModelState.IsValid)
            {
                return View(dto);
            }

            try
            {
                // Normalizar cédula para comparación
                string cedulaNormalizada = dto.Cedula.Replace(".", "").Replace("-", "").Trim();

                // Verificar duplicados (excluyendo el actual)
                var existePaciente = await _context.Pacientes
                    .AnyAsync(p => p.Cedula == cedulaNormalizada && p.Id != dto.Id);

                if (existePaciente)
                {
                    ModelState.AddModelError("Cedula", "Ya existe otro paciente registrado con este RUN");
                    return View(dto);
                }

                if (!string.IsNullOrEmpty(dto.Email))
                {
                    var existeEmail = await _context.Pacientes
                        .AnyAsync(p => p.Email == dto.Email.Trim() && p.Id != dto.Id);

                    if (existeEmail)
                    {
                        ModelState.AddModelError("Email", "Ya existe otro paciente registrado con este email");
                        return View(dto);
                    }
                }

                // Mapear DTO a modelo
                var paciente = dto.ToModel();
                
                _context.Update(paciente);
                await _context.SaveChangesAsync();

                _logger.LogInformation("Paciente actualizado exitosamente: {Nombres} {Apellidos} (ID: {Id})", 
                    paciente.Nombres, paciente.Apellidos, paciente.Id);

                TempData["Success"] = "Paciente actualizado exitosamente";
                return RedirectToAction(nameof(Index));
            }
            catch (DbUpdateConcurrencyException ex)
            {
                if (!PacienteExists(dto.Id))
                {
                    return NotFound();
                }

                _logger.LogError(ex, "Error de concurrencia al actualizar paciente con ID {Id}", id);
                ModelState.AddModelError("", "El paciente fue modificado por otro usuario. Por favor, recargue la página.");
                return View(dto);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al actualizar paciente con ID {Id}", id);
                ModelState.AddModelError("", "Ocurrió un error al actualizar el paciente. Por favor, intente nuevamente.");
                return View(dto);
            }
        }

        // GET: /Pacientes/Delete/5
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

                var pacienteDto = paciente.ToDetalleDto();
                return View(pacienteDto);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cargar paciente para eliminación con ID {Id}", id);
                TempData["Error"] = "Error al cargar los datos del paciente";
                return RedirectToAction(nameof(Index));
            }
        }

        // POST: /Pacientes/Delete/5
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

        private bool PacienteExists(int id)
        {
            return _context.Pacientes.Any(p => p.Id == id);
        }
    }
}
```

---

## Paso 5: Actualizar las Vistas para usar DTOs

### Vista Index actualizada
```html
<!-- Views/Pacientes/Index.cshtml - ACTUALIZAR el @model -->
@model IEnumerable<PacienteDetalleDto>
@{
    ViewData["Title"] = "Lista de Pacientes";
}

<!-- El resto del código de la vista permanece igual -->
<!-- Los PacienteDetalleDto ya tienen la propiedad NombreCompleto y Edad calculadas -->
```

### Vista Create actualizada
```html
<!-- Views/Pacientes/Create.cshtml - ACTUALIZAR el @model -->
@model CrearPacienteDto
@{
    ViewData["Title"] = "Crear Paciente";
}

<!-- El resto del HTML es el mismo, pero ahora con validaciones mejoradas -->
<!-- Los mensajes de error serán más específicos gracias a las validaciones personalizadas -->
```

### Vista Edit actualizada
```html
<!-- Views/Pacientes/Edit.cshtml - ACTUALIZAR el @model -->
@model EditarPacienteDto
@{
    ViewData["Title"] = "Editar Paciente";
}

<!-- El resto del HTML es el mismo -->
```

### Vista Details actualizada
```html
<!-- Views/Pacientes/Details.cshtml - ACTUALIZAR el @model -->
@model PacienteDetalleDto
@{
    ViewData["Title"] = "Detalles del Paciente";
}

<!-- Ahora puedes usar Model.NombreCompleto directamente -->
<h5 class="text-primary">@Model.NombreCompleto</h5>
```

### Vista Delete actualizada
```html
<!-- Views/Pacientes/Delete.cshtml - ACTUALIZAR el @model -->
@model PacienteDetalleDto
@{
    ViewData["Title"] = "Eliminar Paciente";
}

<!-- El resto del HTML es el mismo -->
```

---

## Paso 6: Agregar validación del lado cliente con JavaScript

### Crear archivo de validación personalizada
```html
<!-- Views/Pacientes/Create.cshtml y Edit.cshtml - Agregar en @section Scripts -->
@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
    
    <script>
        $(document).ready(function() {
            // Validación de RUN en tiempo real
            $('#Cedula').on('blur', function() {
                var run = $(this).val();
                if (run) {
                    validarRunChileno(run, $(this));
                }
            });

            // Formatear RUN automáticamente
            $('#Cedula').on('input', function() {
                var run = $(this).val().replace(/\./g, '').replace(/-/g, '');
                if (run.length > 1) {
                    var cuerpo = run.slice(0, -1);
                    var dv = run.slice(-1);
                    
                    // Formatear con puntos
                    cuerpo = cuerpo.replace(/\B(?=(\d{3})+(?!\d))/g, '.');
                    
                    $(this).val(cuerpo + '-' + dv);
                }
            });

            // Validación de edad en tiempo real
            $('#FechaNacimiento').on('change', function() {
                var fechaNac = new Date($(this).val());
                var hoy = new Date();
                var edad = hoy.getFullYear() - fechaNac.getFullYear();
                var mes = hoy.getMonth() - fechaNac.getMonth();
                
                if (mes < 0 || (mes === 0 && hoy.getDate() < fechaNac.getDate())) {
                    edad--;
                }

                if (edad < 18) {
                    mostrarError($(this), 'Debe tener al menos 18 años');
                } else if (fechaNac > hoy) {
                    mostrarError($(this), 'La fecha no puede ser futura');
                } else {
                    limpiarError($(this));
                    mostrarEdad(edad);
                }
            });

            // Validación de teléfono
            $('#Telefono').on('blur', function() {
                var telefono = $(this).val();
                if (telefono) {
                    var patron = /^(\+?56)?9\d{8}$/;
                    telefono = telefono.replace(/\s/g, '').replace(/-/g, '');
                    
                    if (!patron.test(telefono)) {
                        mostrarError($(this), 'Formato inválido. Use: +56912345678 o 912345678');
                    } else {
                        limpiarError($(this));
                    }
                }
            });

            // Validación de email único (llamada AJAX)
            $('#Email').on('blur', function() {
                var email = $(this).val();
                var pacienteId = $('#Id').val() || 0;
                
                if (email) {
                    $.ajax({
                        url: '/Pacientes/VerificarEmailUnico',
                        type: 'GET',
                        data: { email: email, id: pacienteId },
                        success: function(disponible) {
                            if (!disponible) {
                                mostrarError($('#Email'), 'Este email ya está registrado');
                            } else {
                                limpiarError($('#Email'));
                            }
                        }
                    });
                }
            });
        });

        function validarRunChileno(run, elemento) {
            run = run.replace(/\./g, '').replace(/-/g, '');
            
            if (run.length < 8 || run.length > 9) {
                mostrarError(elemento, 'RUN debe tener formato válido');
                return false;
            }

            var cuerpo = run.slice(0, -1);
            var dv = run.slice(-1).toUpperCase();

            if (!/^\d+$/.test(cuerpo)) {
                mostrarError(elemento, 'RUN contiene caracteres inválidos');
                return false;
            }

            // Calcular dígito verificador
            var suma = 0;
            var multiplo = 2;

            for (var i = cuerpo.length - 1; i >= 0; i--) {
                suma += parseInt(cuerpo.charAt(i)) * multiplo;
                multiplo = multiplo === 7 ? 2 : multiplo + 1;
            }

            var dvEsperado = 11 - (suma % 11);
            dvEsperado = dvEsperado === 11 ? '0' : dvEsperado === 10 ? 'K' : dvEsperado.toString();

            if (dv !== dvEsperado) {
                mostrarError(elemento, 'RUN inválido. Dígito verificador debería ser ' + dvEsperado);
                return false;
            }

            limpiarError(elemento);
            return true;
        }

        function mostrarError(elemento, mensaje) {
            elemento.addClass('is-invalid');
            elemento.siblings('.text-danger').text(mensaje);
        }

        function limpiarError(elemento) {
            elemento.removeClass('is-invalid');
            elemento.siblings('.text-danger').text('');
        }

        function mostrarEdad(edad) {
            var mensaje = '<small class="text-success">✓ Edad: ' + edad + ' años</small>';
            $('#FechaNacimiento').siblings('.form-text').html(mensaje);
        }
    </script>

    <style>
        .required::after {
            content: " *";
            color: red;
            font-weight: bold;
        }
    </style>
}
```

---

## Paso 7: Agregar endpoint para validación AJAX

```csharp
// Controllers/PacientesController.cs - AGREGAR este método
[HttpGet]
public async Task<JsonResult> VerificarEmailUnico(string email, int id = 0)
{
    if (string.IsNullOrWhiteSpace(email))
    {
        return Json(true);
    }

    var existe = await _context.Pacientes
        .AnyAsync(p => p.Email == email.Trim() && p.Id != id);

    return Json(!existe);
}
```

---

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Probar todas las validaciones
1. Intenta crear un paciente menor de 18 años
2. Ingresa un RUN inválido
3. Intenta usar un email duplicado
4. Prueba teléfonos con diferentes formatos

### Ejercicio 2: Crear validación personalizada de Comuna
```csharp
// Validations/ComunaValidaAttribute.cs
public class ComunaValidaAttribute : ValidationAttribute
{
    private static readonly string[] ComunasValidas = {
        "Santiago", "Providencia", "Las Condes", "Vitacura",
        "Ñuñoa", "La Reina", "Maipú", "Puente Alto"
        // Agregar más comunas...
    };

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (value == null || string.IsNullOrWhiteSpace(value.ToString()))
        {
            return ValidationResult.Success;
        }

        string comuna = value.ToString();
        
        if (!ComunasValidas.Contains(comuna, StringComparer.OrdinalIgnoreCase))
        {
            return new ValidationResult($"Comuna no válida. Comunas disponibles: {string.Join(", ", ComunasValidas)}");
        }

        return ValidationResult.Success;
    }
}
```

### Ejercicio 3: Agregar campo Comuna al DTO
```csharp
// DTOs/PacienteDto.cs - Agregar a PacienteBaseDto
[ComunaValida]
[StringLength(100)]
[Display(Name = "Comuna")]
public string Comuna { get; set; }
```

---

## 📊 Comparación: Antes y Después

### Antes (Módulo 05):
```csharp
// Validaciones básicas
[Required]
[StringLength(100)]
public string Nombres { get; set; }

[Required]
public string Cedula { get; set; } // Acepta cualquier texto
```

### Después (Módulo 06):
```csharp
// Validaciones avanzadas
[Required(ErrorMessage = "Los nombres son obligatorios")]
[StringLength(100, MinimumLength = 2)]
[RegularExpression(@"^[a-zA-ZáéíóúÁÉÍÓÚñÑ\s]+$", ErrorMessage = "Solo letras")]
public string Nombres { get; set; }

[Required]
[ValidarRun] // Valida RUN chileno completo con dígito verificador
public string Cedula { get; set; }

[FechaNoFutura]
[EdadMinima(18)]
public DateTime FechaNacimiento { get; set; }
```

---

## 🔍 Conceptos Clave Aprendidos

### 1. Validaciones Personalizadas
- Crear atributos heredando de `ValidationAttribute`
- Implementar lógica de negocio específica
- Mensajes de error descriptivos

### 2. DTOs (Data Transfer Objects)
- Separar modelos de BD de modelos de vista
- Diferentes DTOs para diferentes operaciones
- Mapeo entre DTOs y modelos

### 3. Validaciones del Cliente
- JavaScript para validación en tiempo real
- AJAX para validaciones que requieren BD
- Feedback inmediato al usuario

### 4. Arquitectura por Capas
```
Vista (Razor) ↔ DTO ↔ Controlador ↔ Modelo ↔ Base de Datos
```

---

## 🎯 Evaluación del Módulo

### Validaciones implementadas (según rúbrica):

✅ **Campos vacíos**: Required en todos los campos obligatorios
✅ **Espacios**: Trim() en mapeos
✅ **Solo números**: ValidarRun para cédula
✅ **Validación de Run**: Algoritmo completo con dígito verificador
✅ **Validación de fechas**: FechaNoFutura y EdadMinima
✅ **Validación de email**: EmailAddress + verificación de duplicados
✅ **No repetir campos**: Verificación en BD de RUN y email únicos

---

## ➡️ Próximo módulo
**Módulo 07: Autenticación y Autorización** - Aprenderás a implementar login, registro de usuarios y control de acceso basado en roles.