# Roadmap ASP.NET Core WebAPI con Razor Pages

## 🎯 Objetivo General
Desarrollar una aplicación CRUD funcional utilizando **ASP.NET Core (.NET 8)** con **WebAPI** y **Razor Pages**, aplicable a casos reales como sistemas de gestión de clínica, POS o reservas.

## 👥 Audiencia
Estudiantes técnicos de nivel superior con conocimientos básicos de:
- HTML, CSS y JavaScript
- Programación básica
- Formularios web simples

## 🗂️ Estructura del Roadmap

| Semana | Módulo | Contenido Principal | Actividad Práctica | Desafío Autónomo | Avance Proyecto |
|--------|--------|-------------------|-------------------|------------------|-----------------|
| 1 | [**01-introduccion-aspnet**](./01-introduccion-aspnet/) | Fundamentos de ASP.NET Core | "Mi Primera App Web" | Página Personal con Rutas | Decisión de dominio |
| 2 | [**02-frontend-formularios**](./02-frontend-formularios/) | Frontend con formularios web | Formulario de Contacto | Registro de Usuarios | Frontend del proyecto |
| 3 | [**03-dashboard-razor**](./03-dashboard-razor/) | Dashboard con Razor Pages | Dashboard de Ventas | Dashboard personalizado | Dashboard principal |
| 4 | [**04-modelos-ef-core**](./04-modelos-ef-core/) | Modelos y Entity Framework | Sistema de Biblioteca | Modelado completo | Capa de datos |
| 5 | [**05-api-rest-controladores**](./05-api-rest-controladores/) | API REST y Controladores | API REST Biblioteca | API completa dominio | API integrada |
| 6 | [**06-validaciones-dtos**](./06-validaciones-dtos/) | Validaciones y DTOs | Sistema de Validaciones | Validaciones completas | Validaciones robustas |
| 7 | [**07-autenticacion-basica**](./07-autenticacion-basica/) | Autenticación básica (JWT) | Sistema Auth JWT | Auth completo dominio | Autenticación integrada |
| 8 | [**08-proyecto-final**](./08-proyecto-final/) | Proyecto final aplicado | Optimizaciones avanzadas | Proyecto completo | **Entrega final** |

## 📋 Resumen de Contenidos por Módulo

### [01 - Introducción a ASP.NET Core](./01-introduccion-aspnet/)
**Objetivo**: Configurar entorno y crear primera aplicación  
**Conceptos**: Framework, estructura proyecto, middleware, ruteo  
**Práctica**: App "Hola Mundo" con rutas básicas  
**Entregable**: Proyecto personal con navegación

### [02 - Frontend con Formularios Web](./02-frontend-formularios/)
**Objetivo**: Integrar Bootstrap y crear formularios funcionales  
**Conceptos**: Bootstrap, formularios HTML, validación, POST/GET  
**Práctica**: Formulario de contacto completo  
**Entregable**: Sistema de registro multi-paso

### [03 - Dashboard con Razor Pages](./03-dashboard-razor/)
**Objetivo**: Implementar páginas dinámicas con datos  
**Conceptos**: Razor Pages, ViewModels, navegación, datos simulados  
**Práctica**: Dashboard interactivo de ventas  
**Entregable**: Dashboard específico del dominio elegido

### [04 - Modelos y Entity Framework Core](./04-modelos-ef-core/)
**Objetivo**: Diseñar y conectar base de datos  
**Conceptos**: EF Core, modelos, migraciones, relaciones, CRUD  
**Práctica**: Sistema de biblioteca con BD  
**Entregable**: Modelo de datos completo con servicios

### [05 - API REST y Controladores](./05-api-rest-controladores/)
**Objetivo**: Crear API REST completa con documentación  
**Conceptos**: Controllers, HTTP verbs, Swagger, status codes  
**Práctica**: API REST para biblioteca  
**Entregable**: API completa del dominio con Swagger

### [06 - Validaciones y DTOs](./06-validaciones-dtos/)
**Objetivo**: Implementar validaciones robustas y DTOs  
**Conceptos**: FluentValidation, AutoMapper, DTOs, error handling  
**Práctica**: Sistema de validaciones avanzado  
**Entregable**: Validaciones completas con DTOs

### [07 - Autenticación Básica](./07-autenticacion-basica/)
**Objetivo**: Implementar autenticación y autorización  
**Conceptos**: JWT, roles, claims, [Authorize], BCrypt  
**Práctica**: Sistema de auth con JWT  
**Entregable**: Autenticación completa con roles

### [08 - Proyecto Final](./08-proyecto-final/)
**Objetivo**: Integrar y optimizar aplicación completa  
**Conceptos**: Cache, búsqueda avanzada, exportación, testing  
**Práctica**: Optimizaciones y funcionalidades avanzadas  
**Entregable**: **Proyecto final completo documentado**

## 🛠️ Tecnologías Utilizadas
- **Lenguaje**: C#
- **Framework**: .NET 8 / ASP.NET Core
- **Base de datos**: SQL Server / PostgreSQL
- **ORM**: Entity Framework Core
- **Frontend**: Razor Pages + Bootstrap 5
- **API**: RESTful WebAPI con Swagger
- **Autenticación**: JWT Bearer Tokens
- **Validaciones**: FluentValidation
- **Mapeo**: AutoMapper
- **IDE**: Visual Studio / VS Code

## 📋 Opciones de Proyecto Final

### 🏥 Sistema de Gestión de Clínica
**Entidades**: Pacientes, Médicos, Citas, Historiales  
**Usuarios**: Pacientes, Médicos, Recepcionistas, Administradores  
**Funcionalidades**: Agenda médica, historiales, dashboard clínico

### 🛒 Sistema POS (Punto de Venta)
**Entidades**: Productos, Categorías, Ventas, Clientes  
**Usuarios**: Vendedores, Supervisores, Gerentes, Administradores  
**Funcionalidades**: Terminal ventas, inventario, reportes comerciales

### 📅 Sistema de Reservas
**Entidades**: Espacios, Reservas, Usuarios, Horarios  
**Usuarios**: Usuarios, Moderadores, Administradores  
**Funcionalidades**: Calendario reservas, gestión espacios, estadísticas

## 🎯 Criterios de Evaluación Final

| Categoría | Peso | Aspectos Evaluados |
|-----------|------|-------------------|
| **Funcionalidad** | 40% | CRUD completo, Autenticación, Dashboard, Búsquedas, Validaciones |
| **Calidad Técnica** | 30% | Código limpio, Manejo errores, Arquitectura, Base de datos |
| **Documentación** | 20% | README, API docs, Comentarios, Diagramas |
| **Presentación** | 10% | Video demo, UI/UX, Casos de uso |

## 🚀 Cómo Usar Este Roadmap
1. **Sigue el orden secuencial** de los módulos (1-8)
2. **Completa las actividades prácticas** en clase con el docente
3. **Realiza los desafíos autónomos** en casa antes del siguiente módulo
4. **Construye progresivamente** tu proyecto final desde el módulo 1
5. **Consulta los recursos** adicionales cuando sea necesario
6. **Documenta tu progreso** en cada etapa

## 📚 Recursos Adicionales
- [**RECURSOS.md**](./RECURSOS.md) - Enlaces útiles, herramientas y documentación oficial
- [**ejemplos/**](./ejemplos/) - Código de ejemplo y plantillas de referencia
- [**PROYECTO-FINAL.md**](./PROYECTO-FINAL.md) - Especificaciones detalladas del proyecto final

## ✅ Checklist General de Progreso

### Fundamentos (Módulos 1-2):
- [ ] Entorno ASP.NET Core configurado
- [ ] Primera aplicación web funcionando
- [ ] Formularios con Bootstrap implementados
- [ ] Dominio del proyecto final elegido

### Desarrollo Backend (Módulos 3-5):
- [ ] Razor Pages con datos dinámicos
- [ ] Base de datos con Entity Framework
- [ ] API REST completa con Swagger
- [ ] CRUD funcional en toda la aplicación

### Refinamiento (Módulos 6-8):
- [ ] Validaciones robustas implementadas
- [ ] Sistema de autenticación funcionando
- [ ] Proyecto final optimizado y documentado
- [ ] Video demostración completado

---

**🎓 ¡Objetivo Final**: Al completar este roadmap, tendrás una aplicación web profesional que demuestra competencias sólidas en desarrollo ASP.NET Core y estará listo para desafíos más avanzados en el desarrollo web moderno!**

---
*Roadmap diseñado para estudiantes técnicos de jornada vespertina - Enfoque práctico y progresivo* 🚀