# 07 - Autenticación y Autorización Básica

## 🎯 Objetivo de Aprendizaje
Al finalizar esta sección, el estudiante será capaz de:
- Implementar un sistema de login y registro de usuarios
- Proteger páginas que requieren autenticación
- Usar roles para controlar acceso (Administrador, Doctor, Recepcionista)
- Mantener sesión de usuario activa
- Implementar logout seguro

## 📚 ¿Qué es Autenticación y Autorización?

### 🔑 Autenticación
**"¿Quién eres?"** - Verificar la identidad del usuario

**Ejemplo:** Ingresar usuario y contraseña para entrar al sistema

### 🛡️ Autorización
**"¿Qué puedes hacer?"** - Verificar permisos del usuario

**Ejemplo:** Solo los administradores pueden eliminar pacientes

### Flujo completo:
```
1. Usuario ingresa credenciales (Login)
2. Sistema verifica credenciales → AUTENTICACIÓN
3. Sistema crea sesión y almacena datos del usuario
4. Usuario intenta acceder a una página protegida
5. Sistema verifica si tiene permiso → AUTORIZACIÓN
6. Si tiene permiso → acceso permitido
7. Si NO tiene permiso → redirigir a "Acceso Denegado"
```

---

## 🔧 Actividad Práctica Completa

### Prerequisitos: Módulos 04, 05 y 06 completados
**Debes tener funcionando:**
- ✅ Entity Framework Core configurado
- ✅ CRUD de Pacientes
- ✅ Validaciones avanzadas

---

## Paso 1: Crear el modelo Usuario

```csharp
// Models/Usuario.cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace SistemaClinicaMVC.Models
{
    /// <summary>
    /// Modelo para usuarios del sistema (Admins, Doctores, Recepcionistas)
    /// </summary>
    public class Usuario
    {
        [Key]
        public int Id { get; set; }

        [Required]
        [StringLength(100)]
        public string NombreCompleto { get; set; }

        [Required]
        [StringLength(50)]
        public string NombreUsuario { get; set; } // Usado para login

        [Required]
        [EmailAddress]
        [StringLength(200)]
        public string Email { get; set; }

        [Required]
        [StringLength(255)]
        public string PasswordHash { get; set; } // Password encriptado

        [Required]
        [StringLength(20)]
        public string Rol { get; set; } // "Administrador", "Doctor", "Recepcionista"

        public bool Activo { get; set; } = true;

        public DateTime FechaCreacion { get; set; } = DateTime.Now;

        public DateTime? UltimoAcceso { get; set; }
    }

    /// <summary>
    /// Roles disponibles en el sistema
    /// </summary>
    public static class Roles
    {
        public const string Administrador = "Administrador";
        public const string Doctor = "Doctor";
        public const string Recepcionista = "Recepcionista";
    }
}
```

---

## Paso 2: Actualizar DbContext

```csharp
// Data/ClinicaContext.cs - ACTUALIZAR
using Microsoft.EntityFrameworkCore;
using SistemaClinicaMVC.Models;

namespace SistemaClinicaMVC.Data
{
    public class ClinicaContext : DbContext
    {
        public ClinicaContext(DbContextOptions<ClinicaContext> options)
            : base(options)
        {
        }

        public DbSet<Paciente> Pacientes { get; set; }
        public DbSet<Usuario> Usuarios { get; set; } // NUEVO

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Configurar índices únicos
            modelBuilder.Entity<Usuario>()
                .HasIndex(u => u.NombreUsuario)
                .IsUnique();

            modelBuilder.Entity<Usuario>()
                .HasIndex(u => u.Email)
                .IsUnique();

            // Datos semilla: crear usuario administrador por defecto
            modelBuilder.Entity<Usuario>().HasData(
                new Usuario
                {
                    Id = 1,
                    NombreCompleto = "Administrador del Sistema",
                    NombreUsuario = "admin",
                    Email = "admin@clinica.cl",
                    // Password: "Admin123!" (hasheado con BCrypt)
                    PasswordHash = "$2a$11$hKFQzx/wKdN3J5x5KGZaJuP3xQvZ6qZxvWQBYLYdBXQqVNUZV3YNW",
                    Rol = Roles.Administrador,
                    Activo = true,
                    FechaCreacion = DateTime.Now
                }
            );
        }
    }
}
```

---

## Paso 3: Crear migración y actualizar base de datos

```powershell
# En la terminal de Visual Studio (Package Manager Console)
Add-Migration AgregarUsuarios
Update-Database
```

---

## Paso 4: Instalar BCrypt para encriptar contraseñas

```powershell
# En la terminal de Visual Studio
Install-Package BCrypt.Net-Next
```

**¿Por qué BCrypt?**
- ✅ **Nunca guardes contraseñas en texto plano**
- ✅ BCrypt es un algoritmo seguro de hashing
- ✅ Hace que sea casi imposible descifrar la contraseña
- ✅ Cada hash es único (incluye "salt" automático)

**Ejemplo:**
```
Password: "Admin123!"
Hash: "$2a$11$hKFQzx/wKdN3J5x5KGZaJuP3xQvZ6qZxvWQBYLYdBXQqVNUZV3YNW"

Mismo password, diferente hash:
Hash: "$2a$11$XdKjfLmNvP9zY3QaB1cDeOpqRsT4vWxYz..."
```

---

## Paso 5: Crear DTOs para autenticación

```csharp
// DTOs/AuthDto.cs - Crear en carpeta DTOs
using System.ComponentModel.DataAnnotations;

namespace SistemaClinicaMVC.DTOs
{
    /// <summary>
    /// DTO para login de usuario
    /// </summary>
    public class LoginDto
    {
        [Required(ErrorMessage = "El nombre de usuario es obligatorio")]
        [Display(Name = "Usuario")]
        public string NombreUsuario { get; set; }

        [Required(ErrorMessage = "La contraseña es obligatoria")]
        [DataType(DataType.Password)]
        [Display(Name = "Contraseña")]
        public string Password { get; set; }

        [Display(Name = "Recordarme")]
        public bool Recordarme { get; set; }
    }

    /// <summary>
    /// DTO para registro de nuevo usuario
    /// </summary>
    public class RegistroDto
    {
        [Required(ErrorMessage = "El nombre completo es obligatorio")]
        [StringLength(100, MinimumLength = 3)]
        [Display(Name = "Nombre Completo")]
        public string NombreCompleto { get; set; }

        [Required(ErrorMessage = "El nombre de usuario es obligatorio")]
        [StringLength(50, MinimumLength = 4, ErrorMessage = "El usuario debe tener entre 4 y 50 caracteres")]
        [RegularExpression(@"^[a-zA-Z0-9_]+$", ErrorMessage = "Solo letras, números y guión bajo")]
        [Display(Name = "Usuario")]
        public string NombreUsuario { get; set; }

        [Required(ErrorMessage = "El email es obligatorio")]
        [EmailAddress(ErrorMessage = "Email inválido")]
        [Display(Name = "Email")]
        public string Email { get; set; }

        [Required(ErrorMessage = "La contraseña es obligatoria")]
        [StringLength(100, MinimumLength = 6, ErrorMessage = "La contraseña debe tener al menos 6 caracteres")]
        [DataType(DataType.Password)]
        [RegularExpression(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{6,}$",
            ErrorMessage = "La contraseña debe contener mayúsculas, minúsculas, números y símbolos")]
        [Display(Name = "Contraseña")]
        public string Password { get; set; }

        [Required(ErrorMessage = "Debe confirmar la contraseña")]
        [DataType(DataType.Password)]
        [Compare("Password", ErrorMessage = "Las contraseñas no coinciden")]
        [Display(Name = "Confirmar Contraseña")]
        public string ConfirmarPassword { get; set; }

        [Required(ErrorMessage = "Debe seleccionar un rol")]
        [Display(Name = "Rol")]
        public string Rol { get; set; }
    }

    /// <summary>
    /// DTO para cambiar contraseña
    /// </summary>
    public class CambiarPasswordDto
    {
        [Required(ErrorMessage = "La contraseña actual es obligatoria")]
        [DataType(DataType.Password)]
        [Display(Name = "Contraseña Actual")]
        public string PasswordActual { get; set; }

        [Required(ErrorMessage = "La nueva contraseña es obligatoria")]
        [StringLength(100, MinimumLength = 6)]
        [DataType(DataType.Password)]
        [Display(Name = "Nueva Contraseña")]
        public string NuevaPassword { get; set; }

        [Required(ErrorMessage = "Debe confirmar la nueva contraseña")]
        [DataType(DataType.Password)]
        [Compare("NuevaPassword", ErrorMessage = "Las contraseñas no coinciden")]
        [Display(Name = "Confirmar Nueva Contraseña")]
        public string ConfirmarNuevaPassword { get; set; }
    }
}
```

---

## Paso 6: Crear servicio de autenticación

```csharp
// Services/AuthService.cs
using SistemaClinicaMVC.Data;
using SistemaClinicaMVC.Models;
using SistemaClinicaMVC.DTOs;
using Microsoft.EntityFrameworkCore;
using BCrypt.Net;

namespace SistemaClinicaMVC.Services
{
    /// <summary>
    /// Servicio para manejar autenticación y registro de usuarios
    /// </summary>
    public class AuthService
    {
        private readonly ClinicaContext _context;
        private readonly ILogger<AuthService> _logger;

        public AuthService(ClinicaContext context, ILogger<AuthService> logger)
        {
            _context = context;
            _logger = logger;
        }

        /// <summary>
        /// Valida credenciales de usuario
        /// </summary>
        public async Task<Usuario> ValidarCredenciales(string nombreUsuario, string password)
        {
            var usuario = await _context.Usuarios
                .FirstOrDefaultAsync(u => u.NombreUsuario == nombreUsuario && u.Activo);

            if (usuario == null)
            {
                _logger.LogWarning("Intento de login con usuario inexistente: {Usuario}", nombreUsuario);
                return null;
            }

            // Verificar password con BCrypt
            bool passwordValido = BCrypt.Net.BCrypt.Verify(password, usuario.PasswordHash);

            if (!passwordValido)
            {
                _logger.LogWarning("Intento de login con password inválido para usuario: {Usuario}", nombreUsuario);
                return null;
            }

            // Actualizar último acceso
            usuario.UltimoAcceso = DateTime.Now;
            await _context.SaveChangesAsync();

            _logger.LogInformation("Login exitoso para usuario: {Usuario}", nombreUsuario);
            return usuario;
        }

        /// <summary>
        /// Registra un nuevo usuario
        /// </summary>
        public async Task<(bool Exito, string Mensaje, Usuario Usuario)> RegistrarUsuario(RegistroDto dto)
        {
            // Verificar si el usuario ya existe
            var existeUsuario = await _context.Usuarios
                .AnyAsync(u => u.NombreUsuario == dto.NombreUsuario);

            if (existeUsuario)
            {
                return (false, "El nombre de usuario ya está en uso", null);
            }

            // Verificar si el email ya existe
            var existeEmail = await _context.Usuarios
                .AnyAsync(u => u.Email == dto.Email);

            if (existeEmail)
            {
                return (false, "El email ya está registrado", null);
            }

            // Hashear password con BCrypt
            string passwordHash = BCrypt.Net.BCrypt.HashPassword(dto.Password);

            // Crear usuario
            var usuario = new Usuario
            {
                NombreCompleto = dto.NombreCompleto.Trim(),
                NombreUsuario = dto.NombreUsuario.Trim().ToLower(),
                Email = dto.Email.Trim().ToLower(),
                PasswordHash = passwordHash,
                Rol = dto.Rol,
                Activo = true,
                FechaCreacion = DateTime.Now
            };

            _context.Usuarios.Add(usuario);
            await _context.SaveChangesAsync();

            _logger.LogInformation("Usuario registrado exitosamente: {Usuario} ({Rol})", 
                usuario.NombreUsuario, usuario.Rol);

            return (true, "Usuario registrado exitosamente", usuario);
        }

        /// <summary>
        /// Cambia la contraseña de un usuario
        /// </summary>
        public async Task<(bool Exito, string Mensaje)> CambiarPassword(
            int usuarioId, string passwordActual, string nuevaPassword)
        {
            var usuario = await _context.Usuarios.FindAsync(usuarioId);

            if (usuario == null)
            {
                return (false, "Usuario no encontrado");
            }

            // Verificar password actual
            bool passwordValido = BCrypt.Net.BCrypt.Verify(passwordActual, usuario.PasswordHash);

            if (!passwordValido)
            {
                _logger.LogWarning("Intento fallido de cambio de contraseña para usuario: {Usuario}", 
                    usuario.NombreUsuario);
                return (false, "La contraseña actual es incorrecta");
            }

            // Hashear nueva password
            usuario.PasswordHash = BCrypt.Net.BCrypt.HashPassword(nuevaPassword);
            await _context.SaveChangesAsync();

            _logger.LogInformation("Contraseña cambiada exitosamente para usuario: {Usuario}", 
                usuario.NombreUsuario);

            return (true, "Contraseña cambiada exitosamente");
        }

        /// <summary>
        /// Obtiene un usuario por su ID
        /// </summary>
        public async Task<Usuario> ObtenerUsuarioPorId(int id)
        {
            return await _context.Usuarios.FindAsync(id);
        }

        /// <summary>
        /// Obtiene un usuario por su nombre de usuario
        /// </summary>
        public async Task<Usuario> ObtenerUsuarioPorNombre(string nombreUsuario)
        {
            return await _context.Usuarios
                .FirstOrDefaultAsync(u => u.NombreUsuario == nombreUsuario);
        }
    }
}
```

---

## Paso 7: Registrar servicio en Program.cs

```csharp
// Program.cs - AGREGAR después de agregar DbContext
builder.Services.AddScoped<AuthService>();

// Configurar autenticación con cookies
builder.Services.AddAuthentication("CookieAuth")
    .AddCookie("CookieAuth", options =>
    {
        options.LoginPath = "/Auth/Login";
        options.LogoutPath = "/Auth/Logout";
        options.AccessDeniedPath = "/Auth/AccesoDenegado";
        options.ExpireTimeSpan = TimeSpan.FromHours(8); // Sesión expira en 8 horas
        options.SlidingExpiration = true; // Renovar automáticamente
    });

// IMPORTANTE: Agregar DESPUÉS de var app = builder.Build();
app.UseAuthentication(); // DEBE ir antes de UseAuthorization
app.UseAuthorization();
```

---

## Paso 8: Crear el controlador de autenticación

```csharp
// Controllers/AuthController.cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authorization;
using System.Security.Claims;
using SistemaClinicaMVC.Services;
using SistemaClinicaMVC.DTOs;
using SistemaClinicaMVC.Models;

namespace SistemaClinicaMVC.Controllers
{
    public class AuthController : Controller
    {
        private readonly AuthService _authService;
        private readonly ILogger<AuthController> _logger;

        public AuthController(AuthService authService, ILogger<AuthController> logger)
        {
            _authService = authService;
            _logger = logger;
        }

        // GET: /Auth/Login
        [AllowAnonymous]
        public IActionResult Login(string returnUrl = null)
        {
            ViewData["ReturnUrl"] = returnUrl;
            return View();
        }

        // POST: /Auth/Login
        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Login(LoginDto dto, string returnUrl = null)
        {
            ViewData["ReturnUrl"] = returnUrl;

            if (!ModelState.IsValid)
            {
                return View(dto);
            }

            try
            {
                // Validar credenciales
                var usuario = await _authService.ValidarCredenciales(dto.NombreUsuario, dto.Password);

                if (usuario == null)
                {
                    ModelState.AddModelError("", "Usuario o contraseña incorrectos");
                    return View(dto);
                }

                // Crear claims (información del usuario en la sesión)
                var claims = new List<Claim>
                {
                    new Claim(ClaimTypes.NameIdentifier, usuario.Id.ToString()),
                    new Claim(ClaimTypes.Name, usuario.NombreUsuario),
                    new Claim(ClaimTypes.Email, usuario.Email),
                    new Claim(ClaimTypes.Role, usuario.Rol),
                    new Claim("NombreCompleto", usuario.NombreCompleto)
                };

                var claimsIdentity = new ClaimsIdentity(claims, "CookieAuth");
                var claimsPrincipal = new ClaimsPrincipal(claimsIdentity);

                var authProperties = new AuthenticationProperties
                {
                    IsPersistent = dto.Recordarme, // Mantener sesión si marcó "Recordarme"
                    ExpiresUtc = dto.Recordarme 
                        ? DateTimeOffset.UtcNow.AddDays(30) 
                        : DateTimeOffset.UtcNow.AddHours(8)
                };

                // Iniciar sesión
                await HttpContext.SignInAsync("CookieAuth", claimsPrincipal, authProperties);

                _logger.LogInformation("Usuario {Usuario} inició sesión exitosamente", usuario.NombreUsuario);

                // Redirigir a la página solicitada o al dashboard
                if (!string.IsNullOrEmpty(returnUrl) && Url.IsLocalUrl(returnUrl))
                {
                    return Redirect(returnUrl);
                }

                return RedirectToAction("Index", "Home");
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error durante el login");
                ModelState.AddModelError("", "Ocurrió un error al iniciar sesión");
                return View(dto);
            }
        }

        // GET: /Auth/Registro
        [AllowAnonymous]
        public IActionResult Registro()
        {
            return View();
        }

        // POST: /Auth/Registro
        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Registro(RegistroDto dto)
        {
            if (!ModelState.IsValid)
            {
                return View(dto);
            }

            try
            {
                var (exito, mensaje, usuario) = await _authService.RegistrarUsuario(dto);

                if (!exito)
                {
                    ModelState.AddModelError("", mensaje);
                    return View(dto);
                }

                TempData["Success"] = "Registro exitoso. Por favor, inicie sesión.";
                return RedirectToAction(nameof(Login));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error durante el registro");
                ModelState.AddModelError("", "Ocurrió un error al registrar el usuario");
                return View(dto);
            }
        }

        // POST: /Auth/Logout
        [HttpPost]
        [Authorize]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Logout()
        {
            var nombreUsuario = User.Identity.Name;
            
            await HttpContext.SignOutAsync("CookieAuth");
            
            _logger.LogInformation("Usuario {Usuario} cerró sesión", nombreUsuario);
            
            TempData["Info"] = "Sesión cerrada exitosamente";
            return RedirectToAction(nameof(Login));
        }

        // GET: /Auth/AccesoDenegado
        [AllowAnonymous]
        public IActionResult AccesoDenegado()
        {
            return View();
        }

        // GET: /Auth/Perfil
        [Authorize]
        public async Task<IActionResult> Perfil()
        {
            var usuarioId = int.Parse(User.FindFirst(ClaimTypes.NameIdentifier).Value);
            var usuario = await _authService.ObtenerUsuarioPorId(usuarioId);

            if (usuario == null)
            {
                return NotFound();
            }

            return View(usuario);
        }

        // GET: /Auth/CambiarPassword
        [Authorize]
        public IActionResult CambiarPassword()
        {
            return View();
        }

        // POST: /Auth/CambiarPassword
        [HttpPost]
        [Authorize]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> CambiarPassword(CambiarPasswordDto dto)
        {
            if (!ModelState.IsValid)
            {
                return View(dto);
            }

            try
            {
                var usuarioId = int.Parse(User.FindFirst(ClaimTypes.NameIdentifier).Value);
                
                var (exito, mensaje) = await _authService.CambiarPassword(
                    usuarioId, dto.PasswordActual, dto.NuevaPassword);

                if (!exito)
                {
                    ModelState.AddModelError("", mensaje);
                    return View(dto);
                }

                TempData["Success"] = mensaje;
                return RedirectToAction(nameof(Perfil));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al cambiar contraseña");
                ModelState.AddModelError("", "Ocurrió un error al cambiar la contraseña");
                return View(dto);
            }
        }
    }
}
```

---

## Paso 9: Crear las vistas de autenticación

### Vista Login
```html
<!-- Views/Auth/Login.cshtml -->
@model LoginDto
@{
    ViewData["Title"] = "Iniciar Sesión";
    Layout = null; // Sin layout para tener diseño personalizado
}

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Sistema Clínica</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    <style>
        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .login-card {
            border-radius: 15px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
        }
        .card-header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border-radius: 15px 15px 0 0 !important;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-5">
                <div class="card login-card">
                    <div class="card-header text-center py-4">
                        <h3 class="mb-0">
                            <i class="bi bi-hospital"></i> Sistema Clínica
                        </h3>
                        <p class="mb-0">Iniciar Sesión</p>
                    </div>
                    <div class="card-body p-4">
                        @if (TempData["Success"] != null)
                        {
                            <div class="alert alert-success alert-dismissible fade show">
                                <i class="bi bi-check-circle"></i> @TempData["Success"]
                                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                            </div>
                        }

                        @if (TempData["Info"] != null)
                        {
                            <div class="alert alert-info alert-dismissible fade show">
                                <i class="bi bi-info-circle"></i> @TempData["Info"]
                                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                            </div>
                        }

                        <form asp-action="Login" method="post">
                            <div asp-validation-summary="ModelOnly" class="alert alert-danger"></div>

                            <div class="mb-3">
                                <label asp-for="NombreUsuario" class="form-label">
                                    <i class="bi bi-person"></i> Usuario
                                </label>
                                <input asp-for="NombreUsuario" class="form-control form-control-lg" 
                                       placeholder="Ingrese su usuario" autofocus />
                                <span asp-validation-for="NombreUsuario" class="text-danger"></span>
                            </div>

                            <div class="mb-3">
                                <label asp-for="Password" class="form-label">
                                    <i class="bi bi-lock"></i> Contraseña
                                </label>
                                <input asp-for="Password" class="form-control form-control-lg" 
                                       placeholder="Ingrese su contraseña" />
                                <span asp-validation-for="Password" class="text-danger"></span>
                            </div>

                            <div class="mb-3 form-check">
                                <input asp-for="Recordarme" class="form-check-input" />
                                <label asp-for="Recordarme" class="form-check-label">
                                    Recordarme por 30 días
                                </label>
                            </div>

                            <div class="d-grid gap-2">
                                <button type="submit" class="btn btn-primary btn-lg">
                                    <i class="bi bi-box-arrow-in-right"></i> Iniciar Sesión
                                </button>
                            </div>
                        </form>
                    </div>
                    <div class="card-footer text-center text-muted">
                        <small>
                            ¿No tiene cuenta? 
                            <a asp-action="Registro">Registrarse aquí</a>
                        </small>
                        <br />
                        <small class="text-muted">
                            Usuario demo: <strong>admin</strong> | Password: <strong>Admin123!</strong>
                        </small>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
</body>
</html>
```

### Vista Registro
```html
<!-- Views/Auth/Registro.cshtml -->
@model RegistroDto
@{
    ViewData["Title"] = "Registro de Usuario";
    Layout = null;
}

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Sistema Clínica</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    <style>
        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 30px 0;
        }
        .registro-card {
            border-radius: 15px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
        }
        .card-header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border-radius: 15px 15px 0 0 !important;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-6">
                <div class="card registro-card">
                    <div class="card-header text-center py-4">
                        <h3 class="mb-0">
                            <i class="bi bi-person-plus"></i> Registro de Usuario
                        </h3>
                    </div>
                    <div class="card-body p-4">
                        <form asp-action="Registro" method="post">
                            <div asp-validation-summary="ModelOnly" class="alert alert-danger"></div>

                            <div class="mb-3">
                                <label asp-for="NombreCompleto" class="form-label required">Nombre Completo</label>
                                <input asp-for="NombreCompleto" class="form-control" 
                                       placeholder="Ej: Juan Pérez González" />
                                <span asp-validation-for="NombreCompleto" class="text-danger"></span>
                            </div>

                            <div class="row">
                                <div class="col-md-6 mb-3">
                                    <label asp-for="NombreUsuario" class="form-label required">Usuario</label>
                                    <input asp-for="NombreUsuario" class="form-control" 
                                           placeholder="Ej: jperez" />
                                    <span asp-validation-for="NombreUsuario" class="text-danger"></span>
                                    <small class="form-text text-muted">Solo letras, números y _</small>
                                </div>

                                <div class="col-md-6 mb-3">
                                    <label asp-for="Email" class="form-label required">Email</label>
                                    <input asp-for="Email" class="form-control" 
                                           placeholder="ejemplo@correo.com" />
                                    <span asp-validation-for="Email" class="text-danger"></span>
                                </div>
                            </div>

                            <div class="row">
                                <div class="col-md-6 mb-3">
                                    <label asp-for="Password" class="form-label required">Contraseña</label>
                                    <input asp-for="Password" class="form-control" />
                                    <span asp-validation-for="Password" class="text-danger"></span>
                                    <small class="form-text text-muted">
                                        Mín. 6 caracteres, mayúsculas, minúsculas, números y símbolos
                                    </small>
                                </div>

                                <div class="col-md-6 mb-3">
                                    <label asp-for="ConfirmarPassword" class="form-label required">Confirmar</label>
                                    <input asp-for="ConfirmarPassword" class="form-control" />
                                    <span asp-validation-for="ConfirmarPassword" class="text-danger"></span>
                                </div>
                            </div>

                            <div class="mb-3">
                                <label asp-for="Rol" class="form-label required">Rol</label>
                                <select asp-for="Rol" class="form-select">
                                    <option value="">-- Seleccione un rol --</option>
                                    <option value="Administrador">Administrador</option>
                                    <option value="Doctor">Doctor</option>
                                    <option value="Recepcionista">Recepcionista</option>
                                </select>
                                <span asp-validation-for="Rol" class="text-danger"></span>
                            </div>

                            <div class="d-grid gap-2">
                                <button type="submit" class="btn btn-primary btn-lg">
                                    <i class="bi bi-person-check"></i> Registrarse
                                </button>
                            </div>
                        </form>
                    </div>
                    <div class="card-footer text-center">
                        <small>
                            ¿Ya tiene cuenta? 
                            <a asp-action="Login">Iniciar sesión aquí</a>
                        </small>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
</body>
</html>
```

### Vista Acceso Denegado
```html
<!-- Views/Auth/AccesoDenegado.cshtml -->
@{
    ViewData["Title"] = "Acceso Denegado";
}

<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card border-danger">
                <div class="card-header bg-danger text-white text-center">
                    <h3><i class="bi bi-shield-x"></i> Acceso Denegado</h3>
                </div>
                <div class="card-body text-center">
                    <i class="bi bi-exclamation-triangle text-danger" style="font-size: 5rem;"></i>
                    <h4 class="mt-3">No tiene permisos para acceder a esta página</h4>
                    <p class="text-muted">
                        Su rol actual no tiene los permisos necesarios para ver este contenido.
                    </p>
                    <div class="mt-4">
                        <a asp-controller="Home" asp-action="Index" class="btn btn-primary">
                            <i class="bi bi-house"></i> Volver al Inicio
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### Vista Perfil
```html
<!-- Views/Auth/Perfil.cshtml -->
@model Usuario
@{
    ViewData["Title"] = "Mi Perfil";
}

<div class="container mt-4">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header bg-primary text-white">
                    <h4 class="mb-0">
                        <i class="bi bi-person-circle"></i> Mi Perfil
                    </h4>
                </div>
                <div class="card-body">
                    <div class="row mb-3">
                        <div class="col-md-4 text-md-end">
                            <strong>Nombre Completo:</strong>
                        </div>
                        <div class="col-md-8">
                            @Model.NombreCompleto
                        </div>
                    </div>

                    <div class="row mb-3">
                        <div class="col-md-4 text-md-end">
                            <strong>Usuario:</strong>
                        </div>
                        <div class="col-md-8">
                            @Model.NombreUsuario
                        </div>
                    </div>

                    <div class="row mb-3">
                        <div class="col-md-4 text-md-end">
                            <strong>Email:</strong>
                        </div>
                        <div class="col-md-8">
                            @Model.Email
                        </div>
                    </div>

                    <div class="row mb-3">
                        <div class="col-md-4 text-md-end">
                            <strong>Rol:</strong>
                        </div>
                        <div class="col-md-8">
                            <span class="badge bg-info">@Model.Rol</span>
                        </div>
                    </div>

                    <div class="row mb-3">
                        <div class="col-md-4 text-md-end">
                            <strong>Fecha de Registro:</strong>
                        </div>
                        <div class="col-md-8">
                            @Model.FechaCreacion.ToString("dd/MM/yyyy HH:mm")
                        </div>
                    </div>

                    @if (Model.UltimoAcceso.HasValue)
                    {
                        <div class="row mb-3">
                            <div class="col-md-4 text-md-end">
                                <strong>Último Acceso:</strong>
                            </div>
                            <div class="col-md-8">
                                @Model.UltimoAcceso.Value.ToString("dd/MM/yyyy HH:mm")
                            </div>
                        </div>
                    }

                    <hr />

                    <div class="text-center">
                        <a asp-action="CambiarPassword" class="btn btn-primary">
                            <i class="bi bi-key"></i> Cambiar Contraseña
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

---

## Paso 10: Proteger el controlador de Pacientes

```csharp
// Controllers/PacientesController.cs - AGREGAR al inicio de la clase
using Microsoft.AspNetCore.Authorization;

[Authorize] // Requiere estar autenticado para acceder a CUALQUIER acción
public class PacientesController : Controller
{
    // ... resto del código ...

    // Solo administradores pueden eliminar
    [Authorize(Roles = "Administrador")]
    public async Task<IActionResult> Delete(int? id)
    {
        // ... código existente ...
    }

    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    [Authorize(Roles = "Administrador")] // Solo administradores
    public async Task<IActionResult> DeleteConfirmed(int id)
    {
        // ... código existente ...
    }
}
```

---

## Paso 11: Actualizar _Layout.cshtml

```html
<!-- Views/Shared/_Layout.cshtml - ACTUALIZAR navbar -->
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
    <div class="container-fluid">
        <a class="navbar-brand" href="/">
            <i class="bi bi-hospital"></i> Sistema Clínica
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav me-auto">
                <li class="nav-item">
                    <a class="nav-link" asp-controller="Home" asp-action="Index">Inicio</a>
                </li>
                @if (User.Identity.IsAuthenticated)
                {
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="Pacientes" asp-action="Index">Pacientes</a>
                    </li>
                }
            </ul>

            <ul class="navbar-nav">
                @if (User.Identity.IsAuthenticated)
                {
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" id="userDropdown" 
                           data-bs-toggle="dropdown">
                            <i class="bi bi-person-circle"></i> 
                            @User.FindFirst("NombreCompleto")?.Value
                            <span class="badge bg-light text-dark">@User.FindFirst(System.Security.Claims.ClaimTypes.Role)?.Value</span>
                        </a>
                        <ul class="dropdown-menu dropdown-menu-end">
                            <li>
                                <a class="dropdown-item" asp-controller="Auth" asp-action="Perfil">
                                    <i class="bi bi-person"></i> Mi Perfil
                                </a>
                            </li>
                            <li>
                                <a class="dropdown-item" asp-controller="Auth" asp-action="CambiarPassword">
                                    <i class="bi bi-key"></i> Cambiar Contraseña
                                </a>
                            </li>
                            <li><hr class="dropdown-divider"></li>
                            <li>
                                <form asp-controller="Auth" asp-action="Logout" method="post">
                                    <button type="submit" class="dropdown-item text-danger">
                                        <i class="bi bi-box-arrow-right"></i> Cerrar Sesión
                                    </button>
                                </form>
                            </li>
                        </ul>
                    </li>
                }
                else
                {
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="Auth" asp-action="Login">
                            <i class="bi bi-box-arrow-in-right"></i> Iniciar Sesión
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="Auth" asp-action="Registro">
                            <i class="bi bi-person-plus"></i> Registrarse
                        </a>
                    </li>
                }
            </ul>
        </div>
    </div>
</nav>
```

---

## 🎯 Pruebas del Sistema

### 1. Probar Login
```
Usuario: admin
Password: Admin123!
```

### 2. Probar Registro
- Registrar un nuevo usuario con rol "Doctor"
- Verificar que se hashea la contraseña
- Verificar que no permite duplicados

### 3. Probar Autorización
- Iniciar sesión como Doctor
- Intentar eliminar un paciente
- Verificar que muestra "Acceso Denegado"

### 4. Probar Protección de Rutas
- Cerrar sesión
- Intentar acceder a `/Pacientes`
- Verificar que redirige a `/Auth/Login`

---

## 🔐 Conceptos de Seguridad Aprendidos

### 1. Hashing de Contraseñas
```csharp
// ❌ NUNCA HACER ESTO
Password = "Admin123!" // Guardado en texto plano

// ✅ SIEMPRE HACER ESTO
PasswordHash = BCrypt.HashPassword("Admin123!")
// Resultado: "$2a$11$hKFQzx/wKdN3J5x5KGZaJu..."
```

### 2. Claims (Reclamaciones)
Son datos del usuario guardados en la sesión:
- **NameIdentifier**: ID del usuario
- **Name**: Nombre de usuario
- **Role**: Rol del usuario
- **Email**: Email del usuario

### 3. Atributos de Autorización
```csharp
[Authorize] // Requiere estar autenticado
[Authorize(Roles = "Administrador")] // Requiere rol específico
[Authorize(Roles = "Administrador,Doctor")] // Múltiples roles
[AllowAnonymous] // Permite acceso sin autenticación
```

---

## 📊 Roles y Permisos

| Acción | Administrador | Doctor | Recepcionista |
|--------|--------------|--------|---------------|
| Ver pacientes | ✅ | ✅ | ✅ |
| Crear pacientes | ✅ | ✅ | ✅ |
| Editar pacientes | ✅ | ✅ | ❌ |
| Eliminar pacientes | ✅ | ❌ | ❌ |

---

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Agregar validación de fuerza de contraseña visual
Crear un indicador que muestre si la contraseña es débil/media/fuerte

### Ejercicio 2: Implementar "Olvidé mi contraseña"
Permitir resetear contraseña con un token temporal

### Ejercicio 3: Restringir más acciones por rol
- Solo doctores pueden ver historial médico
- Solo recepcionistas pueden agendar citas

---

## ➡️ Próximo módulo
**Módulo 08: Proyecto Final** - Integrarás todo lo aprendido en un sistema completo de gestión de clínica.

---

## 📝 Checklist de Implementación

- [ ] Instalar BCrypt.Net-Next
- [ ] Crear modelo Usuario
- [ ] Actualizar DbContext y crear migración
- [ ] Crear DTOs de autenticación
- [ ] Crear AuthService
- [ ] Configurar autenticación en Program.cs
- [ ] Crear AuthController
- [ ] Crear vistas de Login y Registro
- [ ] Proteger PacientesController
- [ ] Actualizar _Layout.cshtml
- [ ] Probar login con usuario admin
- [ ] Probar registro de nuevo usuario
- [ ] Probar autorización por roles
- [ ] Verificar protección de rutas

¡Felicitaciones! Ahora tu sistema tiene autenticación y autorización completa. 🎉