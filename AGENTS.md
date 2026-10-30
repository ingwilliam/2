# Sistema de información para instituciones educativas

Sistema de gestión académica para instituciones educativas.

## Estado actual del repositorio

- **Aún no hay código de aplicación.** No existen `back/` ni `front/`, ni `specs/`, ni tests, ni CI, ni `package.json` raíz. Los comandos de desarrollo de la sección "Comandos" son el plan, no algo ejecutable todavía.
- **No es un repositorio git.** Los comandos `/spec-impl` que usan `git` deben tratarlo como "No es un repositorio git" hasta que alguien ejecute `git init`. No lo inicialices por tu cuenta.
- **No hay `.gitignore` raíz.** Antes del primer commit hay que crearlo para excluir `.env` y `node_modules`. `.opencode/.gitignore` solo cubre la carpeta `.opencode/`.
- Solo debe existir este `AGENTS.md` en la raíz. No crees `CLAUDE.md`, `GEMINI.md` ni otro archivo de instrucciones.

## Flujo Spec-Driven Development (SDD)

- Toda feature empieza con `/spec` y termina con `/spec-impl` solo si la spec está en estado `Aprobado`.
- Las specs van en `specs/NN-slug.md` y siguen `.opencode/spec/template.md`. Los estados válidos son `Borrador`, `En revisión`, `Aprobado`, `Implementado` y `Obsoleto`.
- **Solo una persona cambia el estado a `Aprobado` o `Implementado`.** Los agentes nunca lo hacen.
- Cada spec con persistencia declara `Cambio de esquema: Sí` o `No`.
- `specs/.spec-config.yml` controla `AutoCreateBranch` (por defecto `false`). No lo sobrescribas si existe.
- Los comandos `.opencode/commands/*.md` usan bloques `!` de PowerShell para leer fecha, specs y estado de git. Si esos bloques llegan como texto literal, ejecútalos tú con PowerShell.

## Reglas para agentes de IA

- Respetar el stack, la estructura y las convenciones de este archivo. Si una spec las contradice, detenerse y preguntar.
- No instalar, actualizar ni eliminar dependencias sin preguntar.
- No crear archivos fuera de la estructura definida.
- No aplicar migraciones contra ninguna base de datos; proponer el comando y esperar a que la persona lo ejecute.
- No modificar migraciones existentes, archivos `.env`, `quasar.config.ts`, `app.css` ni la configuración de ESLint o Prettier sin preguntar.
- No hacer commits, push ni cambios de rama salvo que se pida explícitamente.
- No tocar `.opencode/node_modules/`: son dependencias del plugin de OpenCode.
- Todo endpoint nuevo o modificado debe quedar documentado en Swagger. En specs con contrato de API, incluir el criterio: _"El endpoint aparece en `/api/docs` con descripción, ejemplos y todos sus códigos de respuesta."_
- Ante una duda, preguntar en lugar de asumir.

## Stack

### Backend

| Elemento          | Tecnología                                  |
| ----------------- | ------------------------------------------- |
| Lenguaje          | TypeScript 5                                |
| Runtime           | Node.js 22 LTS                              |
| Framework         | NestJS 11                                   |
| ORM               | TypeORM                                     |
| Validación        | class-validator + class-transformer         |
| Autenticación     | JWT (access + refresh token) con Passport   |
| Documentación API | Swagger (`@nestjs/swagger`) en `/api/docs`  |
| Logs              | Logger de NestJS                            |

### Frontend

| Elemento        | Tecnología                                                 |
| --------------- | ---------------------------------------------------------- |
| Lenguaje        | TypeScript 5                                               |
| Framework       | Vue 3 (Composition API con `<script setup>`)               |
| UI              | Quasar 2 (Quasar CLI con Vite)                             |
| Estado global   | Pinia                                                      |
| Rutas           | Vue Router (configurado por Quasar)                        |
| Peticiones HTTP | Axios, configurado en el boot file `src/boot/axios.ts`     |
| Formularios     | `QForm` con reglas de validación de Quasar                 |
| Notificaciones  | Plugin `Notify` de Quasar                                  |
| Confirmaciones  | Plugin `Dialog` de Quasar                                  |
| Íconos          | Material Icons (`@quasar/extras`)                          |
| Estilos         | Tailwind CSS 4 (plugin `@tailwindcss/vite`, sin preflight) |
| Animaciones     | `<Transition>` de Vue con clases de Tailwind               |
| Tema y colores  | `src/css/quasar.variables.scss`, expuestos a Tailwind con `@theme` |

### Base de datos

| Elemento        | Valor                                                                  |
| --------------- | ---------------------------------------------------------------------- |
| Motor           | PostgreSQL 17                                                          |
| Migraciones     | TypeORM (nunca `synchronize: true`)                                    |
| Datos iniciales | Seed en `back/src/database/seeds/` solo con datos base (roles, estados) |

## Estructura de carpetas (prevista)

### Backend (`back/`)

```
back/src/
├── main.ts
├── app.module.ts
├── <modulo>/                 # Un módulo por feature (usuarios, cursos...)
│   ├── dto/
│   ├── entities/
│   ├── <modulo>.controller.ts
│   ├── <modulo>.service.ts
│   └── <modulo>.module.ts
├── common/                   # Guards, filters, interceptors, pipes, decorators
├── config/                   # Configuración y variables de entorno
└── database/
    ├── migrations/
    └── seeds/
```

### Frontend (`front/`)

```
front/
├── quasar.config.ts          # Plugins, boot files, build
└── src/
    ├── boot/                 # Inicialización (axios.ts, auth.ts...)
    ├── components/           # Componentes compartidos entre módulos
    ├── composables/          # Lógica reutilizable (useXxx)
    ├── css/
    │   ├── app.css                 # Tailwind (theme + utilities) y @theme con los colores
    │   └── quasar.variables.scss   # Colores de marca (única fuente de verdad)
    ├── layouts/              # MainLayout.vue, AuthLayout.vue
    ├── modules/
    │   └── <modulo>/
    │       ├── components/
    │       ├── pages/
    │       ├── services/     # Llamadas a la API de ese módulo
    │       ├── stores/       # Store de Pinia del módulo
    │       └── types.ts
    ├── router/               # index.ts, routes.ts
    └── stores/               # Stores globales (auth, configuración)
```

## Convenciones de código

- **Idioma del código:** nombres de entidades, campos y endpoints en español (`usuario`, `telefono`, `/cursos`). Palabras técnicas del framework en inglés (`controller`, `service`, `dto`).
- **Archivos:** kebab-case con sufijo de rol (`crear-usuario.dto.ts`, `usuarios.service.ts`). Componentes de Vue en PascalCase (`FormularioPerfil.vue`); páginas con sufijo `Page` (`UsuariosPage.vue`) y layouts con sufijo `Layout` (`MainLayout.vue`).
- **Clases y tipos:** PascalCase. **Variables y funciones:** camelCase. **Constantes:** UPPER_SNAKE_CASE.
- **Base de datos:** tablas y columnas en snake_case; tablas en plural (`usuarios`, `fecha_creacion`).
- **Formato:** Prettier y ESLint con la configuración del repositorio. No cambiarla.
- **Comentarios:** en español. Todo método público de services, guards, interceptors y composables lleva JSDoc (qué hace, qué recibe, qué devuelve, qué errores lanza). Explican el **por qué**, no repiten el código.

## Backend

- Controllers delgados: reciben, validan con DTOs, llaman al service y responden. Sin lógica de negocio.
- Toda entrada se valida con DTOs; nunca se confía en el body sin validar.
- Errores con excepciones de NestJS (`NotFoundException`, `BadRequestException`...); un filtro global da el formato de respuesta.
- Nunca se devuelven entidades: siempre un DTO de respuesta, sin contraseñas ni datos internos.
- Toda operación que modifica varios registros relacionados va en una transacción.
- La configuración sale de variables de entorno.

### Swagger

Cualquiera debe poder consumir un endpoint leyendo solo Swagger.

- `nest-cli.json` debe tener el plugin `@nestjs/swagger` con `introspectComments: true` y `classValidatorShim: true`.
- **DTOs:** cada propiedad con JSDoc (descripción y `@example` realista). Enums con `@ApiProperty({ enum: ... })`.
- **Controllers:** `@ApiTags('nombre-modulo')`. Cada método con JSDoc: primera línea = resumen, resto = descripción.
- **Respuestas:** respuesta exitosa con su DTO (`@ApiOkResponse`, `@ApiCreatedResponse`) y **todos los errores posibles** (`@ApiBadRequestResponse`, `@ApiNotFoundResponse`, `@ApiUnauthorizedResponse`, `@ApiForbiddenResponse`, `@ApiConflictResponse`), cada uno con `description` que explique cuándo ocurre.
- **Parámetros:** rutas y query documentados (`@ApiParam` o DTO de query con JSDoc).
- **Seguridad:** endpoints protegidos con `@ApiBearerAuth()`.
- **Listas:** arreglos con `type: [UsuarioDto]`; paginadas con un DTO de respuesta paginada propio.
- Un endpoint sin documentación completa se considera **no terminado**.

## Frontend

- `<script setup lang="ts">` en todos los componentes. No usar Options API.
- Primero los componentes de Quasar (`QTable`, `QForm`, `QInput`, `QDialog`...); después un componente propio o una librería.
- Las llamadas a la API van en los `services` del módulo, nunca dentro de un componente.
- Estado compartido en Pinia; estado local en el componente.
- Cada página contempla carga (`QSpinner`, `QSkeleton` o `loading`), vacío, error y éxito.
- Formularios con `QForm` y `:rules` en cada campo; el error se muestra junto al campo.
- Notificaciones con `Notify`; confirmaciones destructivas con `Dialog`.
- **Estilo con Tailwind** (layout, espaciado, tipografía, colores, responsive). **Quasar solo para componentes**: no usar sus clases utilitarias (`row`, `col-6`, `q-pa-md`).
- Sin `<style>` propio ni `style="..."`, salvo casos que Tailwind no cubra, registrados en la spec.
- Colores solo desde `quasar.variables.scss` vía `@theme` (`bg-primary`, `text-negative`). Nunca `bg-[#1976d2]` ni `text-blue-600`.
- Rutas con carga diferida: `component: () => import(...)`.
- Animaciones sutiles con `<Transition>` y clases de Tailwind (`transition`, `duration-200`).

### Tailwind con Quasar

`src/css/app.css`:

```css
/* Sin preflight: el reset de Tailwind rompe estilos de los componentes de Quasar */
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/utilities.css" layer(utilities);

/* Colores de Quasar disponibles como clases de Tailwind */
@theme inline {
  --color-primary: var(--q-primary);
  --color-secondary: var(--q-secondary);
  --color-accent: var(--q-accent);
  --color-positive: var(--q-positive);
  --color-negative: var(--q-negative);
  --color-warning: var(--q-warning);
  --color-info: var(--q-info);
}
```

En `quasar.config.ts`, `@tailwindcss/vite` va en `build.vitePlugins` y `app.css` en `css`. Si hay conflicto de nombres de clases entre Quasar y Tailwind, usar el prefijo `tw:` en todo el proyecto; no mezclar soluciones.

## Base de datos

- Todo cambio de esquema es una migración nueva. Nunca se edita una migración ya aplicada.
- Revisar las migraciones antes de aplicarlas, sobre todo si contienen `DROP`.
- Columnas nuevas en tablas existentes: nullable o con valor por defecto.
- Claves primarias UUID.
- Toda tabla tiene `fecha_creacion` y `fecha_actualizacion`.

## Contrato de API

- Base: `/api/v1`. JSON en camelCase. Fechas en ISO 8601 y UTC.
- Error estándar: `{ "statusCode": 400, "message": "El teléfono debe tener 10 dígitos", "error": "Bad Request" }`
- Listados paginados: `?pagina=1&limite=20` y respuesta `{ datos, total, pagina, limite }`.

## Seguridad

- Contraseñas con bcrypt; nunca en texto plano, ni guardadas ni en logs.
- Endpoints protegidos por defecto; los públicos se marcan explícitamente.
- Roles del sistema: `admin`, `docente`, `estudiante`.
- Nunca subir `.env` al repositorio; mantener `.env.example` actualizado.

## Comandos (previstos, aún no ejecutables)

| Acción                | Backend (`back/`)                            | Frontend (`front/`) |
| --------------------- | -------------------------------------------- | ------------------- |
| Instalar dependencias | `npm install`                                | `npm install`       |
| Desarrollo            | `npm run start:dev`                          | `npx quasar dev`    |
| Compilar              | `npm run build`                              | `npx quasar build`  |
| Generar migración     | `npm run migration:generate -- NombreCambio` | —                   |
| Aplicar migraciones   | `npm run migration:run`                      | —                   |
| Ejecutar seeds        | `npm run seed`                               | —                   |

Verifica que cada script exista en el `package.json` correspondiente antes de usarlo.

## Git

- Rama principal: `main`. Nadie hace push directo a `main`.
- Ramas de trabajo: `spec-NN-slug` (creadas por `/spec-impl`) o `fix/descripcion-corta`.
- Commits en español, en imperativo: `Agrega teléfono al usuario`.
