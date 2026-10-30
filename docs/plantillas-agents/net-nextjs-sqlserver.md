# Nombre del proyecto

> **Cómo usar este archivo:** completa cada sección con los datos reales del proyecto y borra lo que no aplique. Los valores que aparecen son ejemplos. Este archivo lo leen tanto las personas como los agentes de IA (`/spec` y `/spec-impl`), así que todo lo que escribas aquí se toma como regla.

## Descripción

Una o dos oraciones sobre qué hace el sistema y para quién.

Ejemplo: _Plataforma para gestionar pedidos de restaurantes: los clientes hacen pedidos desde la web y el restaurante los administra desde un panel._

---

## Stack

### Backend

| Elemento          | Tecnología                                                        |
| ----------------- | ----------------------------------------------------------------- |
| Lenguaje          | C# 14                                                             |
| Runtime           | .NET 10 LTS                                                       |
| Framework         | ASP.NET Core Web API con controllers (`[ApiController]`)          |
| Arquitectura      | Clean Architecture (Domain, Application, Infrastructure, Api)     |
| ORM               | Entity Framework Core 10 (proveedor SQL Server)                   |
| Validación        | FluentValidation                                                  |
| Casos de uso      | Clases handler propias registradas en DI (sin MediatR)            |
| Mapeo             | Manual, con métodos de extensión (sin AutoMapper)                 |
| Autenticación     | JWT Bearer (access + refresh token)                               |
| Errores           | `ProblemDetails` con un `IExceptionHandler` global                |
| Documentación API | Swagger con Swashbuckle + comentarios XML, en `/swagger`          |
| Logs              | `ILogger` de .NET                                                 |

### Frontend

| Elemento        | Tecnología                                                  |
| --------------- | ----------------------------------------------------------- |
| Lenguaje        | TypeScript 5                                                |
| Framework       | Next.js 16 (App Router)                                     |
| Estilos         | Tailwind CSS 4                                              |
| Componentes     | shadcn/ui                                                   |
| Formularios     | React Hook Form + Zod                                       |
| Notificaciones  | Sonner                                                      |
| Confirmaciones  | `AlertDialog` de shadcn/ui                                  |
| Íconos          | Lucide React                                                |
| Animaciones     | Motion                                                      |
| Peticiones HTTP | fetch nativo con un cliente propio en `src/lib/api.ts`      |
| Tipos de la API | Generados desde Swagger con `openapi-typescript`            |
| Tema y colores  | Variables CSS de shadcn/ui en `src/app/globals.css`         |

### Base de datos

| Elemento        | Valor                                                                  |
| --------------- | ---------------------------------------------------------------------- |
| Motor           | SQL Server 2022                                                        |
| Migraciones     | Migraciones de EF Core en `MiApp.Infrastructure/Persistence/Migrations` |
| Despliegue      | Script idempotente generado con `dotnet ef migrations script --idempotent`; la app nunca aplica migraciones sola en producción |
| Datos iniciales | `HasData()` solo para catálogos fijos (roles, estados)                 |

---

## Estructura de carpetas

### Backend (`back/`)

```
back/
├── MiApp.sln
└── src/
    ├── MiApp.Domain/                 # Entidades y reglas de negocio puras (sin dependencias)
    │   ├── Common/
    │   │   └── EntidadBase.cs
    │   └── <Modulo>/                 # Usuarios, Pedidos...
    │       └── Usuario.cs
    │
    ├── MiApp.Application/            # Casos de uso, DTOs, validaciones e interfaces
    │   ├── Common/
    │   │   ├── Interfaces/           # IUsuarioRepository, IEmailService...
    │   │   └── Excepciones/
    │   ├── <Modulo>/
    │   │   ├── CrearUsuario/
    │   │   │   ├── CrearUsuarioRequest.cs
    │   │   │   ├── CrearUsuarioValidator.cs
    │   │   │   └── CrearUsuarioHandler.cs
    │   │   ├── ObtenerUsuario/
    │   │   └── UsuarioResponse.cs
    │   └── DependencyInjection.cs    # AddApplication()
    │
    ├── MiApp.Infrastructure/         # EF Core, repositorios y servicios externos
    │   ├── Persistence/
    │   │   ├── AppDbContext.cs
    │   │   ├── Configurations/       # Una IEntityTypeConfiguration por entidad
    │   │   ├── Repositories/
    │   │   └── Migrations/
    │   ├── Services/
    │   └── DependencyInjection.cs    # AddInfrastructure()
    │
    └── MiApp.Api/                    # Entrada HTTP
        ├── Controllers/
        ├── Middleware/
        ├── Program.cs
        ├── appsettings.json
        └── appsettings.Development.json
```

Dependencias permitidas: `Api → Application → Domain` e `Infrastructure → Application`. `Api` referencia a `Infrastructure` solo para registrar servicios en `Program.cs`. `Domain` no referencia a ningún otro proyecto.

### Frontend (`front/`)

```
front/src/
├── app/                      # Rutas (App Router) y globals.css
├── components/
│   └── ui/                   # Componentes de shadcn/ui
├── features/
│   └── <feature>/            # Un directorio por feature (usuarios, pedidos...)
│       ├── components/
│       ├── hooks/
│       ├── services/         # Llamadas a la API de esa feature
│       └── schemas.ts        # Esquemas de Zod de los formularios
└── lib/
    ├── api.ts                # Cliente de API
    └── api-types.ts          # Tipos generados desde Swagger (no editar a mano)
```

---

## Convenciones de código

- **Idioma del código:** nombres de entidades, propiedades y endpoints en español (`Usuario`, `Telefono`, `/pedidos`). Palabras técnicas del framework en inglés (`Controller`, `Handler`, `Request`, `Response`, `Validator`).
- **C#:** una clase por archivo, con el mismo nombre de la clase. Clases, métodos y propiedades en PascalCase; parámetros y variables en camelCase; campos privados con `_camelCase`. Interfaces con prefijo `I`. Métodos asíncronos con sufijo `Async`.
- **Namespaces:** reflejan la ruta de carpetas (`MiApp.Application.Usuarios.CrearUsuario`), con *file-scoped namespaces*.
- **Nullable:** `<Nullable>enable</Nullable>` en todos los proyectos; no se ignoran sus advertencias.
- **Frontend:** archivos en kebab-case (`usuarios-service.ts`); componentes de React en PascalCase (`FormularioPerfil.tsx`); variables y funciones en camelCase.
- **Base de datos:** tablas en PascalCase y plural (`Usuarios`), columnas en PascalCase (`FechaCreacion`), como es habitual en SQL Server.
- **Formato:** `.editorconfig` en el backend y Prettier + ESLint en el frontend, con la configuración del repositorio. No cambiar esa configuración.

---

## Buenas prácticas

### Backend

- Controllers delgados: reciben la petición, llaman al handler y responden. Sin lógica de negocio ni acceso a datos.
- Toda entrada se valida con un validador de FluentValidation; nunca se confía en el body sin validar.
- Los errores de negocio se lanzan con excepciones propias (`NoEncontradoException`, `ConflictoException`...) y el `IExceptionHandler` global las convierte en `ProblemDetails` con el código HTTP correcto.
- Nunca se devuelven entidades directamente: siempre un `Response` (sin contraseñas ni datos internos).
- Todo método asíncrono recibe y propaga un `CancellationToken`.
- Consultas de solo lectura con `AsNoTracking()`; nunca cargar tablas completas en memoria para filtrarlas.
- Toda operación que modifica varios registros relacionados va en una transacción.
- La configuración se lee con `IOptions<T>` desde `appsettings.json` y variables de entorno; los secretos nunca van en `appsettings.json`.
- Inyección de dependencias por constructor; nada de `new` para servicios.

### Documentación de la API (Swagger)

Todo lo que expone la API se documenta. El objetivo es que alguien pueda consumir cualquier endpoint leyendo solo Swagger, sin abrir el código.

**Configuración** — en `MiApp.Api.csproj` y `MiApp.Application.csproj`, generar el archivo de documentación XML:

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

En `Program.cs`, Swagger incluye los comentarios de ambos proyectos (los controllers viven en `Api` y los DTOs en `Application`):

```csharp
builder.Services.AddSwaggerGen(opciones =>
{
    opciones.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, "MiApp.Api.xml"));
    opciones.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, "MiApp.Application.xml"));
});
```

**Reglas:**

- **No suprimir la advertencia CS1591** (miembro público sin comentario XML) en `Api` ni en `Application`: es la alarma de que falta documentación.
- **DTOs:** cada clase y cada propiedad lleva `<summary>` con su descripción y `<example>` con un valor realista.
- **Controllers:** cada método lleva `<summary>` (resumen), `<remarks>` (detalle y reglas de negocio) y un `<response code="...">` por cada código posible.
- **Respuestas:** cada endpoint declara su respuesta exitosa con `[ProducesResponseType<T>]` y **todos sus errores posibles** (400, 401, 403, 404, 409...), con `ProblemDetails` o `ValidationProblemDetails` como tipo.
- **Seguridad:** el esquema Bearer está configurado en Swagger y los endpoints protegidos lo muestran.
- **Listas:** las respuestas paginadas usan un `RespuestaPaginada<T>` propio y documentado.
- Un endpoint sin documentación completa se considera **no terminado**.

**Ejemplo de DTO:**

```csharp
namespace MiApp.Application.Usuarios.CrearUsuario;

/// <summary>Datos para crear un usuario.</summary>
public sealed class CrearUsuarioRequest
{
    /// <summary>Nombre completo del usuario.</summary>
    /// <example>Laura Gómez</example>
    public required string Nombre { get; init; }

    /// <summary>Teléfono celular de 10 dígitos, sin espacios ni indicativo.</summary>
    /// <example>3001234567</example>
    public string? Telefono { get; init; }
}
```

**Ejemplo de controller:**

```csharp
namespace MiApp.Api.Controllers;

[ApiController]
[Route("api/v1/usuarios")]
[Authorize]
public sealed class UsuariosController(CrearUsuarioHandler crearUsuario) : ControllerBase
{
    /// <summary>Crea un usuario.</summary>
    /// <remarks>Solo el rol admin puede crear usuarios. El correo debe ser único.</remarks>
    /// <response code="201">Usuario creado.</response>
    /// <response code="400">Datos inválidos (por ejemplo, teléfono sin 10 dígitos).</response>
    /// <response code="401">Falta el token o expiró.</response>
    /// <response code="403">El usuario no tiene rol admin.</response>
    /// <response code="409">Ya existe un usuario con ese correo.</response>
    [HttpPost]
    [Authorize(Roles = "admin")]
    [ProducesResponseType<UsuarioResponse>(StatusCodes.Status201Created)]
    [ProducesResponseType<ValidationProblemDetails>(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    [ProducesResponseType(StatusCodes.Status403Forbidden)]
    [ProducesResponseType<ProblemDetails>(StatusCodes.Status409Conflict)]
    public async Task<ActionResult<UsuarioResponse>> CrearAsync(
        CrearUsuarioRequest request, CancellationToken cancellationToken)
    {
        var usuario = await crearUsuario.EjecutarAsync(request, cancellationToken);
        return Created($"/api/v1/usuarios/{usuario.Id}", usuario);
    }
}
```

### Comentarios en el código

- Todo miembro público de `Domain`, `Application` e `Infrastructure` lleva comentario XML (`<summary>`, `<param>`, `<returns>`, `<exception>`). En el frontend, las funciones de `services`, `hooks` y `lib` llevan JSDoc.
- Los comentarios explican el **por qué** de una decisión o una regla de negocio, no repiten lo que el código ya dice.
- Comentarios en español, igual que los nombres del dominio.

### Frontend

- Server Components por defecto; `"use client"` solo cuando haga falta interactividad.
- Las llamadas a la API van en los `services` de la feature y usan los tipos de `api-types.ts`; nunca se escriben a mano los tipos que ya define Swagger.
- Cada pantalla contempla los estados de carga (`Skeleton`), vacío, error y éxito.
- Formularios con React Hook Form y Zod; el error se muestra junto al campo.
- Notificaciones con Sonner; confirmaciones destructivas con `AlertDialog`.
- Reutilizar los componentes de `components/ui` antes de crear uno nuevo o instalar una librería.
- **Tailwind para todo el estilo:** nada de `<style>`, CSS Modules ni `style={{...}}` salvo casos que Tailwind no cubra, registrados en la spec.
- **Colores desde una sola fuente:** las variables de tema de shadcn/ui en `globals.css` (`bg-primary`, `text-destructive`). Nunca colores sueltos como `bg-[#1976d2]` o `text-blue-600`.
- Animaciones sutiles con Motion (200–300 ms); nada de animar por animar.

### Base de datos

- Todo cambio de esquema se hace con una migración nueva de EF Core. Nunca se edita una migración ya aplicada.
- Cada migración generada se revisa antes de aplicarse, especialmente si contiene `DropColumn` o `DropTable`.
- Antes del primer despliegue a producción, las migraciones de desarrollo se consolidan en una sola `InitialCreate`.
- Las columnas nuevas en tablas existentes son opcionales (nullable) o llevan valor por defecto.
- Claves primarias `Guid` generadas con `Guid.CreateVersion7()` para evitar la fragmentación de índices.
- Toda tabla tiene `FechaCreacion` y `FechaActualizacion`.
- El mapeo de cada entidad va en su propia clase de `Configurations/`, nunca con atributos en las entidades de `Domain`.

---

## Contrato de API

- Base: `/api/v1`.
- Formato JSON en camelCase (comportamiento por defecto de ASP.NET Core).
- Respuesta de error estándar con `ProblemDetails` (RFC 9457):

  ```json
  {
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "Datos inválidos",
    "status": 400,
    "errors": { "telefono": ["El teléfono debe tener 10 dígitos."] }
  }
  ```

- Listados paginados con `?pagina=1&limite=20` y respuesta `{ datos, total, pagina, limite }`.
- Fechas en formato ISO 8601 y en UTC (`DateTimeOffset` en C#).

---

## Seguridad

- Contraseñas con un hash seguro (`PasswordHasher<T>` de ASP.NET Core Identity o BCrypt); nunca se guardan ni se registran en texto plano.
- `[Authorize]` en todos los controllers; los endpoints públicos se marcan explícitamente con `[AllowAnonymous]`.
- Roles del sistema: `admin`, `cliente`.
- Secretos en *User Secrets* para desarrollo y variables de entorno en producción; nunca en el repositorio.
- CORS configurado solo para los orígenes del frontend.

---

## Comandos

| Acción                 | Backend (`back/`)                                                     | Frontend (`front/`)  |
| ---------------------- | --------------------------------------------------------------------- | -------------------- |
| Restaurar dependencias | `dotnet restore`                                                      | `npm install`        |
| Desarrollo             | `dotnet watch --project src/MiApp.Api`                                | `npm run dev`        |
| Compilar               | `dotnet build`                                                        | `npm run build`      |
| Generar migración      | `dotnet ef migrations add NombreCambio --project src/MiApp.Infrastructure --startup-project src/MiApp.Api` | — |
| Aplicar en local       | `dotnet ef database update --project src/MiApp.Infrastructure --startup-project src/MiApp.Api` | — |
| Script para producción | `dotnet ef migrations script --idempotent --project src/MiApp.Infrastructure --startup-project src/MiApp.Api -o migraciones.sql` | — |
| Generar tipos de la API | —                                                                    | `npm run api:types`  |

---

## Git

- Rama principal: `main`. Nadie hace push directo a `main`.
- Ramas de trabajo: `spec-NN-slug` (creadas por `/spec-impl`) o `fix/descripcion-corta`.
- Commits en español y en imperativo: `Agrega teléfono al usuario`.

---

## Reglas para agentes de IA

- Este proyecto trabaja con **Spec-Driven Development**: toda feature empieza con `/spec` y se implementa con `/spec-impl` solo cuando la spec está en estado `Aprobado`.
- Respetar el stack, la estructura y las convenciones de este archivo. Si algo de una spec las contradice, detenerse y preguntar.
- Respetar las dependencias entre proyectos de Clean Architecture; nunca agregar una referencia de `Domain` o `Application` hacia `Infrastructure` o `Api`.
- No instalar, actualizar ni eliminar paquetes NuGet o npm sin preguntar.
- No crear archivos fuera de la estructura definida.
- No aplicar migraciones contra ninguna base de datos; proponer el comando y esperar a que la persona lo ejecute.
- No modificar migraciones existentes, `appsettings.*.json`, los `.csproj`, `Program.cs`, `globals.css` ni la configuración de `.editorconfig`, ESLint o Prettier sin preguntar.
- No hacer commits ni push salvo que se pida explícitamente.
- Todo endpoint nuevo o modificado debe quedar documentado en Swagger según las reglas de este archivo. En las specs con contrato de API, incluir el criterio de aceptación: _"El endpoint aparece en `/swagger` con descripción, ejemplos y todos sus códigos de respuesta."_
- Después de cambiar el contrato de un endpoint, recordar a la persona que ejecute `npm run api:types` en el frontend.
- Ante una duda, preguntar en lugar de asumir.