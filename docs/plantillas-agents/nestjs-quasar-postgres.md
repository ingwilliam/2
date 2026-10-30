# Nombre del proyecto

> **Cómo usar este archivo:** completa cada sección con los datos reales del proyecto y borra lo que no aplique. Los valores que aparecen son ejemplos. Este archivo lo leen tanto las personas como los agentes de IA (`/spec` y `/spec-impl`), así que todo lo que escribas aquí se toma como regla.

## Descripción

Una o dos oraciones sobre qué hace el sistema y para quién.

Ejemplo: _Plataforma para gestionar pedidos de restaurantes: los clientes hacen pedidos desde la web y el restaurante los administra desde un panel._

---

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
| Documentación API | Swagger (`@nestjs/swagger` + plugin CLI con `introspectComments`) en `/api/docs` |
| Logs              | Logger de NestJS                            |

### Frontend

| Elemento        | Tecnología                       |
| --------------- | -------------------------------- |
| Lenguaje        | TypeScript 5                                         |
| Framework       | Vue 3 (Composition API con `<script setup>`)         |
| UI              | Quasar 2 (Quasar CLI con Vite)                       |
| Estado global   | Pinia                                                |
| Rutas           | Vue Router (configurado por Quasar)                  |
| Peticiones HTTP | Axios, configurado en el boot file `src/boot/axios.ts` |
| Formularios     | `QForm` con reglas de validación de Quasar           |
| Notificaciones  | Plugin `Notify` de Quasar                            |
| Confirmaciones  | Plugin `Dialog` de Quasar                            |
| Íconos          | Material Icons (`@quasar/extras`)                    |
| Estilos         | Tailwind CSS 4 (plugin `@tailwindcss/vite`, sin preflight) |
| Animaciones     | `<Transition>` de Vue con clases de Tailwind         |
| Tema y colores  | `src/css/quasar.variables.scss`, expuestos a Tailwind con `@theme` |

### Base de datos

| Elemento      | Valor                                   |
| ------------- | --------------------------------------- |
| Motor         | PostgreSQL 17                           |
| Migraciones   | Migraciones de TypeORM (nunca `synchronize: true`) |
| Datos iniciales | Seed en `back/src/database/seeds/` solo con datos base (roles, estados) |

---

## Estructura de carpetas

### Backend (`back/`)

```
back/src/
├── main.ts
├── app.module.ts
├── <modulo>/                 # Un módulo por feature (usuarios, pedidos...)
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
├── quasar.config.ts          # Configuración de Quasar (plugins, boot files, build)
└── src/
    ├── App.vue
    ├── assets/
    ├── boot/                 # Inicialización (axios.ts, auth.ts...)
    ├── components/           # Componentes compartidos entre módulos
    ├── composables/          # Lógica reutilizable (useXxx)
    ├── css/
    │   ├── app.css                 # Tailwind (theme + utilities) y @theme con los colores
    │   └── quasar.variables.scss   # Colores de marca (única fuente de verdad)
    ├── layouts/              # MainLayout.vue, AuthLayout.vue
    ├── modules/
    │   └── <modulo>/         # Un directorio por feature (usuarios, pedidos...)
    │       ├── components/
    │       ├── pages/
    │       ├── services/     # Llamadas a la API de ese módulo
    │       ├── stores/       # Store de Pinia del módulo
    │       └── types.ts
    ├── router/
    │   ├── index.ts
    │   └── routes.ts
    └── stores/               # Stores globales (auth, configuración)
```

---

## Convenciones de código

- **Idioma del código:** nombres de entidades, campos y endpoints en español (`usuario`, `telefono`, `/pedidos`). Palabras técnicas del framework en inglés (`controller`, `service`, `dto`).
- **Archivos:** kebab-case con sufijo de rol (`crear-usuario.dto.ts`, `usuarios.service.ts`). Componentes de Vue en PascalCase (`FormularioPerfil.vue`); páginas con sufijo `Page` (`UsuariosPage.vue`) y layouts con sufijo `Layout` (`MainLayout.vue`), como en la convención de Quasar.
- **Clases y tipos:** PascalCase. **Variables y funciones:** camelCase. **Constantes:** UPPER_SNAKE_CASE.
- **Base de datos:** tablas y columnas en snake_case y en plural para tablas (`usuarios`, `fecha_creacion`).
- **Formato:** Prettier y ESLint con la configuración del repositorio. No cambiar esa configuración.

---

## Buenas prácticas

### Backend

- Controllers delgados: reciben, validan con DTOs, llaman al service y responden. Sin lógica de negocio.
- Toda entrada se valida con DTOs; nunca se confía en el body sin validar.
- Los errores se lanzan con las excepciones de NestJS (`NotFoundException`, `BadRequestException`...) y un filtro global da el formato de respuesta.
- Nunca se devuelven entidades directamente: siempre un DTO de respuesta (sin contraseñas ni datos internos).
- Toda operación que modifica varios registros relacionados va en una transacción.
- La configuración sale de variables de entorno, nunca de valores escritos en el código.

### Documentación de la API (Swagger)

Todo lo que expone la API se documenta. El objetivo es que alguien pueda consumir cualquier endpoint leyendo solo Swagger, sin abrir el código.

**Configuración** — en `nest-cli.json`, el plugin de Swagger convierte los comentarios JSDoc en documentación y lee las validaciones de class-validator:

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": { "introspectComments": true, "classValidatorShim": true }
      }
    ]
  }
}
```

**Reglas:**

- **DTOs:** cada propiedad lleva un comentario JSDoc con su descripción y un `@example` realista. Los enums se documentan con `@ApiProperty({ enum: ... })`.
- **Controllers:** cada controller lleva `@ApiTags('nombre-modulo')`. Cada método lleva un comentario JSDoc: la primera línea es el resumen y lo demás, la descripción.
- **Respuestas:** cada endpoint declara su respuesta exitosa con el DTO de respuesta (`@ApiOkResponse({ type: UsuarioDto })`, `@ApiCreatedResponse`...) y **todos sus errores posibles** (`@ApiBadRequestResponse`, `@ApiNotFoundResponse`, `@ApiUnauthorizedResponse`, `@ApiForbiddenResponse`), cada uno con una `description` que explique cuándo ocurre.
- **Parámetros:** los parámetros de ruta y de query se documentan (`@ApiParam`, o un DTO de query con JSDoc).
- **Seguridad:** los endpoints protegidos llevan `@ApiBearerAuth()`.
- **Listas:** las respuestas de arreglos usan `type: [UsuarioDto]`; las paginadas, un DTO de respuesta paginada propio.
- Un endpoint sin documentación completa se considera **no terminado**.

**Ejemplo de DTO:**

```ts
export class CrearUsuarioDto {
  /**
   * Nombre completo del usuario.
   * @example 'Laura Gómez'
   */
  @IsString()
  @Length(3, 100)
  nombre: string;

  /**
   * Teléfono celular de 10 dígitos, sin espacios ni indicativo.
   * @example '3001234567'
   */
  @IsOptional()
  @Matches(/^\d{10}$/)
  telefono?: string | null;
}
```

**Ejemplo de controller:**

```ts
@ApiTags('usuarios')
@ApiBearerAuth()
@Controller('usuarios')
export class UsuariosController {
  /**
   * Crea un usuario.
   *
   * Solo el rol `admin` puede crear usuarios. El correo debe ser único.
   */
  @Post()
  @ApiCreatedResponse({ type: UsuarioDto, description: 'Usuario creado.' })
  @ApiBadRequestResponse({ description: 'Datos inválidos (por ejemplo, teléfono sin 10 dígitos).' })
  @ApiConflictResponse({ description: 'Ya existe un usuario con ese correo.' })
  @ApiUnauthorizedResponse({ description: 'Falta el token o expiró.' })
  @ApiForbiddenResponse({ description: 'El usuario no tiene rol admin.' })
  crear(@Body() dto: CrearUsuarioDto): Promise<UsuarioDto> {
    return this.usuariosService.crear(dto);
  }
}
```

### Comentarios en el código

- Todo método público de services, guards, interceptors y composables lleva JSDoc: qué hace, qué recibe, qué devuelve y qué errores lanza.
- Los comentarios explican el **por qué** de una decisión o una regla de negocio, no repiten lo que el código ya dice (`// suma 1` sobra).
- Comentarios en español, igual que los nombres del dominio.

### Frontend

- Composition API con `<script setup lang="ts">` en todos los componentes. No usar Options API.
- Usar primero los componentes de Quasar (`QTable`, `QForm`, `QInput`, `QDialog`...) antes de crear uno propio o instalar una librería.
- Las llamadas a la API van en los `services` del módulo, nunca directamente dentro de un componente.
- El estado compartido va en Pinia; el estado local de un componente se queda en el componente.
- Cada página contempla los estados de carga (`QSpinner`, `QSkeleton` o `loading` del componente), vacío, error y éxito.
- Formularios con `QForm` y reglas (`:rules`) en cada campo; el error se muestra junto al campo.
- Notificaciones con `Notify` y confirmaciones destructivas con `Dialog`.
- **Tailwind para todo el estilo propio:** layout, espaciado, tipografía, colores y responsive (`flex`, `gap-4`, `p-6`, `md:grid-cols-2`...).
- **Quasar solo para componentes:** no usar sus clases utilitarias de CSS (`row`, `col-6`, `q-pa-md`, `q-mt-sm`...). Así hay una sola forma de dar estilo.
- **Sin estilos escritos a mano:** nada de `<style>` con CSS propio ni `style="..."` salvo casos que Tailwind no cubra, registrados en la spec.
- **Colores desde una sola fuente:** se definen en `quasar.variables.scss` y Tailwind los usa a través de `@theme` (`bg-primary`, `text-negative`). Nunca colores sueltos como `bg-[#1976d2]` o `text-blue-600`.
- Rutas con carga diferida (`component: () => import(...)`).
- Animaciones sutiles con `<Transition>` y clases de Tailwind (`transition`, `duration-200`); nada de animar por animar.

### Configuración de Tailwind con Quasar

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

En `quasar.config.ts`, el plugin `@tailwindcss/vite` va en `build.vitePlugins` y `app.css` en la lista `css`. Si aparece un conflicto de nombres de clases entre Quasar y Tailwind, la solución acordada es usar el prefijo `tw:` de Tailwind en todo el proyecto, no mezclar soluciones.

### Base de datos

- Todo cambio de esquema se hace con una migración nueva. Nunca se edita una migración ya aplicada.
- Las migraciones se revisan antes de aplicarse, especialmente si contienen `DROP`.
- Las columnas nuevas en tablas existentes son opcionales (nullable) o llevan valor por defecto.
- Claves primarias de tipo UUID.
- Toda tabla tiene `fecha_creacion` y `fecha_actualizacion`.

---

## Contrato de API

- Base: `/api/v1`.
- Formato JSON en camelCase.
- Respuesta de error estándar:

  ```json
  { "statusCode": 400, "message": "El teléfono debe tener 10 dígitos", "error": "Bad Request" }
  ```

- Listados paginados con `?pagina=1&limite=20` y respuesta `{ datos, total, pagina, limite }`.
- Fechas en formato ISO 8601 y en UTC.

---

## Seguridad

- Contraseñas con bcrypt; nunca se guardan ni se registran en texto plano.
- Endpoints protegidos por defecto; los públicos se marcan explícitamente.
- Roles del sistema: `admin`, `cliente`.
- Nunca subir archivos `.env` al repositorio; mantener `.env.example` actualizado.

---

## Comandos

| Acción                 | Backend (`back/`)                           | Frontend (`front/`) |
| ---------------------- | ------------------------------------------- | ------------------- |
| Instalar dependencias  | `npm install`                               | `npm install`       |
| Desarrollo             | `npm run start:dev`                         | `npx quasar dev`    |
| Compilar               | `npm run build`                             | `npx quasar build`  |
| Generar migración      | `npm run migration:generate -- NombreCambio` | —                  |
| Aplicar migraciones    | `npm run migration:run`                     | —                   |
| Ejecutar seeds         | `npm run seed`                              | —                   |

---

## Git

- Rama principal: `main`. Nadie hace push directo a `main`.
- Ramas de trabajo: `spec-NN-slug` (creadas por `/spec-impl`) o `fix/descripcion-corta`.
- Commits en español y en imperativo: `Agrega teléfono al usuario`.

---

## Reglas para agentes de IA

- Este proyecto trabaja con **Spec-Driven Development**: toda feature empieza con `/spec` y se implementa con `/spec-impl` solo cuando la spec está en estado `Aprobado`.
- Respetar el stack, la estructura y las convenciones de este archivo. Si algo de una spec las contradice, detenerse y preguntar.
- No instalar, actualizar ni eliminar dependencias sin preguntar.
- No crear archivos fuera de la estructura definida.
- No aplicar migraciones contra ninguna base de datos; proponer el comando y esperar a que la persona lo ejecute.
- No modificar migraciones existentes, archivos `.env`, `quasar.config.ts`, `app.css` ni la configuración de ESLint o Prettier sin preguntar.
- No hacer commits ni push salvo que se pida explícitamente.
- Todo endpoint nuevo o modificado debe quedar documentado en Swagger según las reglas de este archivo. En las specs con contrato de API, incluir el criterio de aceptación: _"El endpoint aparece en `/api/docs` con descripción, ejemplos y todos sus códigos de respuesta."_
- Ante una duda, preguntar en lugar de asumir.