# 03 - Dashboard con Razor Pages

## 🎯 Objetivo de Aprendizaje
Al finalizar esta sección, el estudiante será capaz de:
- Implementar Razor Pages en ASP.NET Core
- Crear dashboards interactivos con datos dinámicos
- Utilizar ViewModels para organizar datos
- Implementar navegación entre páginas

## 📚 Contenidos Principales

### 1. Introducción a Razor Pages
**¿Qué son las Razor Pages?**
Las Razor Pages son una forma simplificada de crear páginas web en ASP.NET Core. Cada página combina HTML con código C# usando la sintaxis Razor.

```html
<!-- Ejemplo básico de sintaxis Razor -->
@page
@model IndexModel
@{
    ViewData["Title"] = "Mi Dashboard";
}

<h1>Bienvenido @Model.NombreUsuario</h1>
<p>Tienes @Model.TotalNotificaciones notificaciones pendientes</p>
```

**Diferencias con MVC tradicional:**
- **MVC**: Controller → View (separados)
- **Razor Pages**: PageModel + View (unidos en un archivo .cshtml.cs)

**Conceptos clave:**
- `@page` - Define que es una Razor Page
- `@model` - Conecta con el PageModel (código C#)
- `@{}` - Bloque de código C#
- `@variable` - Mostrar variables en HTML

### 2. Creación de Dashboards
**¿Qué es un Dashboard?**
Un dashboard es una página que muestra información importante de forma visual y resumida, como métricas, gráficos y tablas.

**Elementos típicos de un dashboard:**
```html
<!-- Tarjeta de métrica -->
<div class="card">
    <div class="card-body">
        <h5 class="card-title">Ventas del Mes</h5>
        <h2 class="text-success">$@Model.VentasMes.ToString("N2")</h2>
        <small class="text-muted">+15% vs mes anterior</small>
    </div>
</div>
```

**Tipos de visualizaciones:**
- **Tarjetas**: Para métricas importantes (números, porcentajes)
- **Tablas**: Para listas de datos detallados
- **Gráficos**: Para tendencias y comparaciones
- **Barras de progreso**: Para mostrar completitud

### 3. ViewModels y Organización de Datos
**¿Qué es un ViewModel?**
Un ViewModel es una clase que contiene solo los datos que necesita una vista específica.

```csharp
// Ejemplo de ViewModel para dashboard
public class DashboardViewModel
{
    public int TotalUsuarios { get; set; }
    public decimal VentasDelDia { get; set; }
    public List<VentaResumen> VentasRecientes { get; set; } = new();
    public string MensajeBienvenida => $"Dashboard actualizado: {DateTime.Now:HH:mm}";
}
```

**¿Por qué usar ViewModels?**
- **Seguridad**: No expones datos sensibles de la base de datos
- **Performance**: Solo cargas los datos que necesitas
- **Flexibilidad**: Puedes combinar datos de diferentes fuentes

### 4. Navegación y Enlaces
**TagHelpers para navegación:**
Los TagHelpers son atributos especiales que ayudan a generar HTML dinámico.

```html
<!-- Navegación con TagHelpers -->
<a asp-page="/Productos/Index" class="btn btn-primary">Ver Productos</a>
<a asp-page="/Ventas/Detalle" asp-route-id="@venta.Id">Ver Detalle</a>

<!-- Formulario con TagHelper -->
<form asp-page="/Buscar" method="get">
    <input asp-for="TerminoBusqueda" class="form-control" />
    <button type="submit">Buscar</button>
</form>
```

**Ventajas de los TagHelpers:**
- **URLs amigables**: Se generan automáticamente
- **Validación**: Detecta rutas inexistentes en tiempo de compilación
- **Intellisense**: Autocompletado en Visual Studio

## 🔧 Actividad Práctica en Clase (2 horas)

## 🔧 Actividad Práctica en Clase (2 horas)

### Ejercicio Guiado: "Dashboard de la Clínica Médica"

#### Paso 1: Configurar Razor Pages en el Sistema de Clínica
```csharp
using Microsoft.Extensions.FileSystemGlobbing.Internal.Patterns;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();
builder.Services.AddRazorPages();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();
app.MapRazorPages();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");


app.Run();

```

#### Paso 2: Crear ViewModel para Dashboard de la Clínica
```csharp
using System.Collections.Generic;

namespace ClinicApp.Models
{
    public class DashboardViewModel
    {
        public int TotalPacientes { get; set; }
        public int CitasHoy { get; set; }
        public int MedicosDisponibles { get; set; }
        public int SalasOcupadas { get; set; }
        public decimal IngresosMensuales { get; set; }
        public List<CitaDelDia> CitasRecientes { get; set; } = new();
        public List<PacienteReciente> PacientesRecientes { get; set; } = new();
        public List<MedicoStats> MedicosActivos { get; set; } = new();
        public List<EspecialidadStats> EspecialidadesPopulares { get; set; } = new();
    }

    public class CitaDelDia
    {
        public int Id { get; set; }
        public DateTime FechaHora { get; set; }
        public string PacienteNombre { get; set; }
        public string MedicoNombre { get; set; }
        public string Especialidad { get; set; }
        public string Estado { get; set; } // Programada, En curso, Completada, Cancelada
        public string Motivo { get; set; }
    }

    public class PacienteReciente
    {
        public int Id { get; set; }
        public string NombreCompleto { get; set; }
        public string Cedula { get; set; }
        public DateTime FechaRegistro { get; set; }
        public string UltimaCita { get; set; }
        public string Estado { get; set; }
    }

    public class MedicoStats
    {
        public string NombreCompleto { get; set; }
        public string Especialidad { get; set; }
        public int CitasHoy { get; set; }
        public string Estado { get; set; } // Disponible, Ocupado, No disponible
        public TimeSpan ProximaCita { get; set; }
    }

    public class EspecialidadStats
    {
        public string Especialidad { get; set; }
        public int CitasEsteMes { get; set; }
        public decimal Ingresos { get; set; }
        public int PacientesAtendidos { get; set; }
    }

   }

```

#### Paso 3: Crear la Razor Page
```csharp
// Pages/Dashboard.cshtml.cs
using Microsoft.AspNetCore.Mvc.RazorPages;

public class DashboardModel : PageModel
{
    public DashboardViewModel Dashboard { get; set; } = new();

    public void OnGet()
    {
        // Simular datos del dashboard
        Dashboard = new DashboardViewModel
        {
            TotalVentas = 1250,
            IngresosMensuales = 45670.50m,
            ProductosVendidos = 3420,
            ClientesActivos = 156,
            VentasRecientes = GenerarVentasRecientes(),
            ProductosPopulares = GenerarProductosPopulares()
        };
    }

    private List<VentaDelDia> GenerarVentasRecientes()
    {
        return new List<VentaDelDia>
        {
            new() { Id = 1001, Fecha = DateTime.Today, Cliente = "Juan Pérez", Total = 150.00m, Estado = "Completada" },
            new() { Id = 1002, Fecha = DateTime.Today.AddHours(-2), Cliente = "María García", Total = 280.50m, Estado = "Pendiente" },
            new() { Id = 1003, Fecha = DateTime.Today.AddHours(-4), Cliente = "Carlos López", Total = 95.75m, Estado = "Completada" },
            new() { Id = 1004, Fecha = DateTime.Today.AddHours(-6), Cliente = "Ana Martínez", Total = 320.00m, Estado = "Completada" },
            new() { Id = 1005, Fecha = DateTime.Yesterday, Cliente = "Luis Rodríguez", Total = 180.25m, Estado = "Cancelada" }
        };
    }

    private List<ProductoTop> GenerarProductosPopulares()
    {
        return new List<ProductoTop>
        {
            new() { Nombre = "Laptop Dell", CantidadVendida = 25, Ingresos = 18750.00m },
            new() { Nombre = "Mouse Logitech", CantidadVendida = 150, Ingresos = 4500.00m },
            new() { Nombre = "Teclado Mecánico", CantidadVendida = 80, Ingresos = 8000.00m },
            new() { Nombre = "Monitor 24\"", CantidadVendida = 35, Ingresos = 10500.00m }
        };
    }
}
```

#### Paso 4: Vista del Dashboard
```html
@page
@model ClinicApp.Pages.DashboardModel
@{
	ViewData["Title"] = "Dashboard de Clinica";
}

<div class="d-flex justify-content-between align-items-center mb-4">
    <h1 class="h3 mb-0 text-gray-800">Dashboard de Clinica</h1>
    <div>
        <span class="text-muted">Última actualización: @DateTime.Now.ToString("dd/MM/yyyy HH:mm")</span>
    </div>
</div>

<!-- Tarjetas de Métricas -->
<div class="row mb-4">
    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-primary shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-primary text-uppercase mb-1">
                            Total Pacientes
                        </div>
                        <div class="h5 mb-0 font-weight-bold text-gray-800">@Model.Dashboard.TotalPacientes</div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-users fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-success shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-success text-uppercase mb-1">
                            Citas de Hoy
                        </div>
						<div class="h5 mb-0 font-weight-bold text-gray-800">@Model.Dashboard.CitasHoy</div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-calendar-check fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-info shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-info text-uppercase mb-1">
                            Médicos Disponibles
                        </div>
                        <div class="h5 mb-0 font-weight-bold text-gray-800">@Model.Dashboard.MedicosDisponibles</div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-user-md fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-warning shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-warning text-uppercase mb-1">
                            Salas Ocupadas
                        </div>
                        <div class="h5 mb-0 font-weight-bold text-gray-800">@Model.Dashboard.SalasOcupadas</div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-door-open fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- Tablas de Datos -->
<div class="row">
    <!-- Citas Recientes -->
    <div class="col-lg-6">
        <div class="card shadow mb-4">
            <div class="card-header py-3 d-flex justify-content-between align-items-center">
                <h6 class="m-0 font-weight-bold text-primary">Citas Recientes</h6>
                <a asp-page="/Citas/Index" class="btn btn-primary btn-sm">Ver Todas</a>
            </div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table table-bordered">
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Fecha</th>
                                <th>Paciente</th>
                                <th>Médico</th>
                                <th>Estado</th>
                                <th>Acciones</th>
                            </tr>
                        </thead>
                        <tbody>
							@foreach (var cita in Model.Dashboard.CitasRecientes)
                            {
                                <tr>
                                    <td>@cita.Id</td>
                                    <td>@cita.FechaHora.ToString("dd/MM HH:mm")</td>
                                    <td>@cita.PacienteNombre</td>
                                    <td>@cita.MedicoNombre</td>
                                    <td>
                                        <span class="badge @(cita.Estado == "Completada" ? "bg-success" :
                                                            cita.Estado == "Pendiente" ? "bg-warning" : "bg-danger")">
                                            @cita.Estado
                                        </span>
                                    </td>
                                    <td>
                                        <a asp-page="/Citas/Detalle" asp-route-id="@cita.Id" class="btn btn-sm btn-outline-primary">Ver</a>
                                    </td>
                                </tr>
                            }
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <!-- Pacientes Recientes -->
    <div class="col-lg-6">
		<div class="card shadow mb-4">
			<div class="card-header py-3 d-flex justify-content-between align-items-center">
				<h6 class="m-0 font-weight-bold text-primary">Pacientes Recientes</h6>
				<a asp-page="/Pacientes/Index" class="btn btn-primary btn-sm">Ver Todos</a>
			</div>
			<div class="card-body">
				<div class="table-responsive">
					<table class="table table-bordered">
						<thead>
							<tr>
								<th>Cédula</th>
								<th>Nombre</th>
								<th>Fecha Registro</th>
								<th>Estado</th>
								<th>Acciones</th>
							</tr>
						</thead>
						<tbody>
							@foreach (var paciente in Model.Dashboard.PacientesRecientes)
							{
								<tr>
									<td>@paciente.Cedula</td>
									<td>@paciente.NombreCompleto</td>
									<td>@paciente.FechaRegistro.ToString("dd/MM/yyyy")</td>
									<td>@paciente.Estado</td>
									<td>
										<a asp-page="/Pacientes/Detalle" asp-route-cedula="@paciente.Cedula"
										   class="btn btn-sm btn-info">Ver</a>
									</td>
								</tr>
							}
						</tbody>
					</table>
				</div>
			</div>
		</div>
    </div>
</div>
```
### Paso 5°: Enlazar Layout de Views e Importar Modelos en Pages
```html
// Añadir esto en /Pages/_ViewStart.cshtml
@{
   Layout = "../Views/Shared/_Layout.cshtml";
}

```

```html
// Añadir esto en /Pages/_ViewImports.cshtml
@using ClinicApp
@using ClinicApp.Models
@namespace ClinicApp.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

### Puntos Clave a Explicar:
- Diferencia entre PageModel y Controller
- Uso de @model y @page
- ViewModels para estructurar datos
- TagHelpers asp-page y asp-route

## 🏠 Desafío Autónomo (Casa)

### Tarea: "Dashboard Personalizado para tu Dominio"
Crear un dashboard completo para el proyecto elegido anteriormente.

**Para Sistema de Clínica:**
1. **Métricas principales**: Pacientes registrados, Citas del día, Médicos disponibles, Consultas completadas
2. **Citas próximas**: Tabla con paciente, médico, hora, tipo de consulta
3. **Pacientes recientes**: Lista de últimos pacientes registrados
4. **Médicos**: Cards con especialidad y disponibilidad

**Para Sistema POS:**
1. **Métricas principales**: Ventas del día, Productos en stock bajo, Clientes registrados, Ingresos mensuales
2. **Ventas recientes**: Como en el ejemplo de clase
3. **Productos con stock bajo**: Alertas de inventario
4. **Top clientes**: Mejores compradores del mes

**Para Sistema de Reservas:**
1. **Métricas principales**: Reservas del día, Espacios ocupados, Usuarios activos, Ingresos del mes
2. **Reservas próximas**: Tabla con usuario, espacio, fecha/hora
3. **Espacios disponibles**: Grid visual de disponibilidad
4. **Estadísticas de uso**: Espacios más solicitados

**Requisitos Técnicos:**
- ✅ Usar Razor Pages correctamente
- ✅ Implementar ViewModels apropiados
- ✅ Datos simulados realistas
- ✅ Diseño responsivo y atractivo
- ✅ Al menos 4 métricas principales
- ✅ 2 tablas o listas de datos
- ✅ Enlaces de navegación funcionales

**Entregables:**
- Código fuente del dashboard
- Screenshots de diferentes tamaños de pantalla
- Documento explicando las decisiones de diseño
- Video demo de la funcionalidad (3-5 min)

## 🚀 Avance en el Proyecto Final

### Implementación del Dashboard Principal
Convertir la página de inicio de tu proyecto en un dashboard funcional.

### Tareas Específicas:
1. **Migrar a Razor Pages** la estructura actual
2. **Crear ViewModels** para organizar los datos
3. **Implementar métricas** relevantes para tu dominio
4. **Agregar tablas dinámicas** con datos simulados
5. **Crear navegación** entre páginas principales
6. **Diseñar layout** consistente y profesional

### Estructura de Archivos:
```
MiProyectoFinal/
├── Pages/
│   ├── Shared/
│   │   └── _Layout.cshtml
│   ├── Dashboard.cshtml
│   ├── Dashboard.cshtml.cs
│   └── ...
├── Models/
│   ├── ViewModels/
│   │   ├── DashboardViewModel.cs
│   │   └── ...
│   └── ...
└── ...
```

### Criterios de Evaluación:
- Dashboard funcional y atractivo
- Uso correcto de Razor Pages
- ViewModels bien estructurados
- Datos simulados coherentes
- Navegación clara y funcional

## 📝 Material de Apoyo
- [Razor Pages Documentation](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/)
- [Chart.js para gráficos](./integracion-chartjs.md)
- [Font Awesome icons](./iconos-fontawesome.md)
- [Ejemplos de dashboards](./ejemplos-dashboards/)

## ✅ Checklist de Completado
- [ ] Razor Pages configuradas y funcionando
- [ ] Dashboard con métricas implementado
- [ ] ViewModels creados y utilizados
- [ ] Navegación entre páginas funcionando
- [ ] Desafío autónomo completado
- [ ] Dashboard del proyecto final implementado

---
**Anterior:** [02 - Frontend con Formularios](../02-frontend-formularios/)  
**Siguiente:** [04 - Modelos y Entity Framework Core](../04-modelos-ef-core/)
