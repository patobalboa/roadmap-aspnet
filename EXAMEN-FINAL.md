# EXAMEN FINAL - Desarrollo de Aplicaciones ASP.NET Core

## Objetivo del Examen

Evaluar de manera integral los conocimientos adquiridos durante el curso mediante el desarrollo completo de una aplicación web ASP.NET Core que incluya:

- Base de datos relacional con Entity Framework Core
- Operaciones CRUD completas
- Validaciones del lado del cliente y servidor
- DTOs (Data Transfer Objects)
- Autenticación y autorización de usuarios
- Presentación profesional del proyecto

---

## Instrucciones Generales

### Forma de Entrega

**Enviar al correo**: `patricio.balboa@virginiogomez.cl`
**Usar correo institucional obligatoriamente**

**Opciones de entrega** (elegir una):

1. **🔗 Repositorio GitHub** (RECOMENDADO)

   - Subir el proyecto completo a GitHub
   - Enviar el link del repositorio
   - Incluir archivo `README.md` con instrucciones de instalación
   - Asegurarse de que el repositorio sea **público** o agregar al docente como colaborador
2. **☁️ Google Drive / OneDrive**

   - Subir proyecto comprimido (.zip)
   - Enviar link con permisos de visualización
   - Incluir todos los archivos necesarios

**Formato del asunto del correo**:

```
[EXAMEN FINAL] - [Apellido] [Nombre] - [Paralelo]
```

**Ejemplo**:

```
[EXAMEN FINAL] - Gómez Virginio - A
```

---

## Desarrollo del Examen

### Parte 1: Continuación del Certamen 2 (Base)

Debes continuar el desarrollo de tu proyecto del **Certamen 2**, agregando las siguientes funcionalidades:

### Requisitos Obligatorios a Implementar:

#### 1️⃣ **Validaciones Completas** (15 puntos)

Implementar validaciones tanto del lado del **cliente** como del **servidor**:

**Validaciones Requeridas**:

- ✅ Campos obligatorios (no vacíos)
- ✅ Validación de formato de email
- ✅ Validación de longitud de campos (min/max)
- ✅ Validación de números (solo dígitos donde corresponda)
- ✅ Validación de rangos (fechas, precios, cantidades)
- ✅ Validación de unicidad (cédula, email, etc.)
- ✅ Mensajes de error claros y específicos en español

**Ejemplo de implementación**:

```csharp
// Modelo con Data Annotations
public class Producto
{
    [Required(ErrorMessage = "El nombre es obligatorio")]
    [StringLength(100, MinimumLength = 3, 
        ErrorMessage = "El nombre debe tener entre 3 y 100 caracteres")]
    public string Nombre { get; set; }

    [Required(ErrorMessage = "El precio es obligatorio")]
    [Range(0.01, 999999.99, 
        ErrorMessage = "El precio debe estar entre 0.01 y 999,999.99")]
    [DisplayFormat(DataFormatString = "{0:C2}")]
    public decimal Precio { get; set; }

    [EmailAddress(ErrorMessage = "Email inválido")]
    [StringLength(150)]
    public string? EmailContacto { get; set; }
}
```

#### 2️⃣ **DTOs (Data Transfer Objects)** (10 puntos)

Implementar DTOs para separar los modelos de base de datos de los modelos de presentación:

**DTOs Requeridos**:

- ✅ DTOs para operaciones de creación (Create)
- ✅ DTOs para operaciones de actualización (Update)
- ✅ DTOs para respuestas (Response/View)
- ✅ Uso de AutoMapper o mapeo manual

**Ejemplo**:

```csharp
// DTO para crear producto
public class ProductoCreateDto
{
    public string Nombre { get; set; }
    public decimal Precio { get; set; }
    public int CategoriaId { get; set; }
}

// DTO para mostrar producto
public class ProductoResponseDto
{
    public int Id { get; set; }
    public string Nombre { get; set; }
    public decimal Precio { get; set; }
    public string CategoriaNombre { get; set; }
    public string FechaCreacion { get; set; }
}
```

#### 3️⃣ **Autenticación y Autorización** (20 puntos)

Implementar sistema completo de autenticación de usuarios:

**Funcionalidades Requeridas**:

- ✅ Sistema de registro de usuarios
- ✅ Inicio de sesión (Login)
- ✅ Cierre de sesión (Logout)
- ✅ Protección de rutas/controladores con `[Authorize]`
- ✅ Roles de usuario (Administrador, Usuario, etc.)
- ✅ Diferentes permisos según rol
- ✅ Contraseñas encriptadas
- ✅ Validación de credenciales

**Ejemplo básico**:

```csharp
// Controlador protegido
[Authorize]
public class ProductosController : Controller
{
    // Solo usuarios autenticados pueden acceder
  
    [Authorize(Roles = "Administrador")]
    public IActionResult Delete(int id)
    {
        // Solo administradores pueden eliminar
    }
}
```

#### 4️⃣ **Base de Datos Completa** (10 puntos)

Mantener y mejorar la base de datos del Certamen 2:

**Requisitos**:

- ✅ Mínimo 5 tablas relacionadas
- ✅ Relaciones correctamente definidas (Foreign Keys)
- ✅ Atributos coherentes y apropiados
- ✅ Tabla de usuarios para autenticación
- ✅ Índices en campos clave
- ✅ Datos de prueba coherentes

#### 5️⃣ **Mantenedores CRUD Completos** (15 puntos)

Operaciones completas para las entidades principales:

**Operaciones Requeridas**:

- ✅ **Crear** (Create): Formularios con validaciones
- ✅ **Listar** (Read): Tablas con paginación
- ✅ **Editar** (Update): Actualización de registros
- ✅ **Eliminar** (Delete): Con confirmación
- ✅ **Búsqueda/Filtrado**: En al menos una entidad
- ✅ **Ordenamiento**: Por columnas

#### 6️⃣ **Estilos y Diseño** (5 puntos)

Interfaz profesional y consistente:

**Requisitos de Diseño**:

- ✅ Bootstrap 5 o framework CSS similar
- ✅ Diseño responsive (funciona en móvil, tablet, PC)
- ✅ Paleta de colores consistente
- ✅ Navegación clara (menú/navbar)
- ✅ Mensajes de feedback (success, error, warning)
- ✅ Iconos (Font Awesome, Bootstrap Icons)
- ✅ Formularios bien estructurados

#### 7️⃣ **Organización y Código Limpio** (3 puntos)

Código profesional y mantenible:

**Criterios**:

- ✅ Estructura de carpetas clara
- ✅ Nombres descriptivos (variables, métodos, clases)
- ✅ Comentarios en código complejo
- ✅ Separación de responsabilidades (Controllers, Services, Repositories)
- ✅ Uso de async/await apropiado
- ✅ Manejo de errores con try-catch

---

## Presentación

### Formato: "Venta de Software"

Debes presentar tu aplicación como si fueras un **vendedor de software** presentando el producto a un cliente potencial.

### Estructura de la Presentación (10-15 minutos):

#### 1. **Introducción** (2 minutos)

- Nombre del sistema
- Problemática que resuelve
- Público objetivo
- Valor agregado

#### 2. **Demostración en Vivo** (8-10 minutos)

- Navegación por el sistema
- Demostración de registro/login
- Operaciones CRUD de al menos 2 entidades
- Mostrar validaciones funcionando
- Demostrar roles y permisos
- Mostrar búsqueda y filtrado

#### 3. **Preguntas y Respuestas** (5 minutos)


### Consejos para la Presentación:

- ✅ Preparar datos de prueba coherentes y profesionales
- ✅ Tener cuenta de administrador y usuario regular preparadas
- ✅ Practicar el flujo de navegación antes
- ✅ Mostrar confianza en el producto
- ✅ Explicar beneficios, no solo características
- ✅ Mantener contacto visual con la audiencia
- ✅ Vestimenta profesional

---

## Rúbrica de Evaluación (100 puntos)

| Criterio                      | Excelente                                                                          | Bueno                                                    | Regular                                                 | Insuficiente                      | Puntos         |
| ----------------------------- | ---------------------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------- | --------------------------------- | -------------- |
| **Base de Datos**       | BD completa, relaciones coherentes, diseño normalizado, incluye usuarios (10 pts) | BD con relaciones básicas, falta normalización (7 pts) | BD incompleta, faltan relaciones (4 pts)                | No hay BD o no funciona (0 pts)   | /10            |
| **Mantenedores CRUD**   | CRUD completo en todas las entidades, funciona perfectamente (15 pts)              | CRUD completo pero con errores menores (10 pts)          | CRUD incompleto o con errores importantes (5 pts)       | No hay CRUD o no funciona (0 pts) | /15            |
| **Validaciones**        | Validaciones completas cliente/servidor, mensajes claros (15 pts)                  | Validaciones básicas implementadas (10 pts)             | Validaciones incompletas o inconsistentes (5 pts)       | No hay validaciones (0 pts)       | /15            |
| **DTOs**                | DTOs bien implementados en todas las operaciones (10 pts)                          | DTOs parcialmente implementados (6 pts)                  | DTOs incorrectamente implementados (3 pts)              | No usa DTOs (0 pts)               | /10            |
| **Autenticación**      | Sistema completo: registro, login, roles, permisos (20 pts)                        | Sistema básico de autenticación funcionando (13 pts)   | Autenticación parcial o con errores (7 pts)            | No hay autenticación (0 pts)     | /20            |
| **Estilos y Diseño**   | Diseño profesional, responsive, consistente (5 pts)                               | Diseño básico pero funcional (3 pts)                   | Diseño mínimo o inconsistente (1 pt)                  | Sin diseño (0 pts)               | /5             |
| **Organización**       | Código muy bien organizado, limpio, comentado (3 pts)                             | Código organizado pero mejorable (2 pts)                | Código desorganizado (1 pt)                            | Código muy desordenado (0 pts)   | /3             |
| **Presentación**       | Presentación profesional, dominio del tema, respuestas claras (12 pts)            | Presentación correcta, respuestas adecuadas (8 pts)     | Presentación básica, dificultad en respuestas (4 pts) | Presentación deficiente (0 pts)  | /12            |
| **Documentación**      | README completo, instrucciones claras, diagramas (5 pts)                           | README básico con instrucciones (3 pts)                 | README incompleto (1 pt)                                | Sin documentación (0 pts)        | /5             |
| **Funcionalidad Extra** | Implementaciones adicionales valiosas (5 pts bonus)                                | -                                                        | -                                                       | -                                 | /5             |
| **TOTAL**               |                                                                                    |                                                          |                                                         |                                   | **/100** |

### 🌟 Puntos Extra (Opcional - hasta 5 puntos):

- ⭐ **+2 pts**: Implementar paginación en listas
- ⭐ **+2 pts**: Implementar exportación a Excel/PDF
- ⭐ **+1 pt**: Implementar búsqueda avanzada con múltiples filtros
- ⭐ **+1 pt**: Implementar cambio de contraseña
- ⭐ **+1 pt**: Implementar dashboard con estadísticas

---

## 📋 Checklist de Entrega

Antes de enviar, verificar que:

- [ ] El proyecto compila sin errores
- [ ] Todas las operaciones CRUD funcionan
- [ ] Las validaciones están implementadas
- [ ] Los DTOs están en uso
- [ ] El sistema de autenticación funciona
- [ ] Los roles y permisos están implementados
- [ ] El diseño es profesional y responsive
- [ ] El README.md está completo
- [ ] El script SQL de la base de datos está incluido
- [ ] Los datos de prueba son coherentes
- [ ] He probado en una instalación limpia
- [ ] Tengo cuentas de prueba preparadas (admin y usuario)
- [ ] He practicado la presentación

## Recursos de Apoyo

### Documentación Oficial:

- [ASP.NET Core Documentation](https://docs.microsoft.com/en-us/aspnet/core/)
- [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Identity](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity)
- [Data Annotations](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations)

### Material del Curso:

- Módulos 01-08 del repositorio roadmap-aspnet
- Ejemplos vistos en clase
- Proyectos de certámenes anteriores

---

## 🎯 Criterios de Éxito

Un proyecto **exitoso** debe:

- ✅ Funcionar completamente sin errores críticos
- ✅ Tener una base de datos bien diseñada
- ✅ Implementar TODAS las funcionalidades requeridas
- ✅ Tener validaciones robustas
- ✅ Usar DTOs apropiadamente
- ✅ Tener sistema de autenticación seguro
- ✅ Presentar diseño profesional
- ✅ Estar bien documentado
- ✅ Ser presentado con confianza y claridad

---
