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

#### Paso 1: Crear proyecto en Visual Studio 2022
```
1. Abrir Visual Studio Community 2022
2. Crear nuevo proyecto → ASP.NET Core Web App
3. Nombre: "SistemaClinica"
4. Framework: .NET 8.0
5. Authentication: None (por ahora)
6. Configure for HTTPS: ✓
```

#### Paso 2: Estructura básica de la clínica
```csharp
// Program.cs - Configuración inicial del sistema de clínica
var builder = WebApplication.CreateBuilder(args);

// Agregar servicios del sistema de clínica
builder.Services.AddRazorPages();

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

// Rutas básicas del sistema de clínica
app.MapGet("/", () => "🏥 Sistema de Clínica Médica - Bienvenido");
app.MapGet("/pacientes", () => "📋 Módulo de Pacientes");
app.MapGet("/medicos", () => "👩‍⚕️ Módulo de Médicos");
app.MapGet("/citas", () => "📅 Módulo de Citas Médicas");

app.MapRazorPages();
app.Run();
```

#### Paso 3: Crear página de inicio de la clínica
```html
<!-- wwwroot/index.html - Página principal del sistema -->
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Clínica Médica</title>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            margin: 40px; 
            background-color: #f8f9fa;
        }
        .container { 
            max-width: 800px; 
            margin: 0 auto; 
            background: white; 
            padding: 30px; 
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .header { 
            text-align: center; 
            color: #2c5aa0; 
            margin-bottom: 30px;
        }
        .modules { 
            display: grid; 
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); 
            gap: 20px; 
        }
        .module { 
            border: 1px solid #ddd; 
            padding: 20px; 
            border-radius: 8px; 
            text-align: center;
            transition: transform 0.2s;
        }
        .module:hover { 
            transform: translateY(-5px); 
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🏥 Clínica San José</h1>
            <p>Sistema de Gestión Médica</p>
        </div>
        
        <div class="modules">
            <div class="module">
                <h3>👤 Pacientes</h3>
                <p>Registro y gestión de pacientes</p>
                <a href="/pacientes">Ver Pacientes</a>
            </div>
            <div class="module">
                <h3>👩‍⚕️ Médicos</h3>
                <p>Directorio médico de la clínica</p>
                <a href="/medicos">Ver Médicos</a>
            </div>
            <div class="module">
                <h3>📅 Citas</h3>
                <p>Programación de citas médicas</p>
                <a href="/citas">Ver Citas</a>
            </div>
        </div>
    </div>
</body>
</html>
```

### Puntos Clave a Explicar:
- ¿Cómo Visual Studio 2022 facilita el desarrollo?
- ¿Qué hace cada línea de código en Program.cs?
- ¿Cómo se organizan los módulos de una clínica?
- ¿Qué es el middleware y cómo funciona?
- ¿Cómo se sirven archivos estáticos (HTML, CSS)?

## 🏠 Desafío Autónomo (Casa)

### Tarea: "Extensión del Sistema de Clínica"
Continuar el desarrollo del sistema de clínica iniciado en clase:

**Requisitos:**
1. **Ruta de especialidades** (`/especialidades`) - Lista de especialidades médicas
2. **Ruta de emergencias** (`/emergencias`) - Página de contacto de emergencias
3. **Ruta de horarios** (`/horarios/{dia}`) - Muestra horarios de atención por día
4. **Ruta de ubicación** (`/ubicacion`) - Información de contacto y ubicación
5. **Página 404** personalizada con tema médico

**Entregables:**
- Proyecto de Visual Studio 2022 completo
- Screenshots de cada ruta funcionando
- Archivo README.md con instrucciones de ejecución
- Documentación de las rutas implementadas

**Criterios de Evaluación:**
- ✅ Todas las rutas funcionan correctamente
- ✅ Código limpio y comentado
- ✅ Tema coherente de clínica médica
- ✅ Página 404 personalizada con diseño médico
- ✅ Uso correcto de parámetros de ruta

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
