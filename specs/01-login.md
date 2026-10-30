# SPEC 01 — Inicio de sesión con correo y contraseña

> **Estado:** Borrador
> **Área:** Full-stack
> **Depende de:** Ninguna spec registrada (ver Riesgos: requiere la entidad `usuario`, aún sin spec)
> **Fecha:** 2026-10-30
> **Objetivo:** El usuario inicia sesión con su correo y contraseña, mantiene la sesión con renovación automática y puede cerrarla.

---

## Por qué existe esta spec

El sistema es de acceso restringido por roles (`admin`, `docente`, `estudiante`). Ninguna otra pantalla o endpoint puede protegerse hasta que exista autenticación, por eso es la primera spec.

---

## Alcance

**Incluye:**

- Endpoint `POST /api/v1/auth/login` que valida correo y contraseña y devuelve un access token y un refresh token.
- Endpoint `POST /api/v1/auth/refresh` que emite un nuevo access token a partir de un refresh token válido.
- Endpoint `POST /api/v1/auth/logout` que invalida el refresh token de la sesión.
- Página de inicio de sesión en el frontend (`/login`) con validación de formulario.
- Guard de rutas que redirige a `/login` cuando no hay sesión.
- Dashboard único de inicio tras el login para todos los roles; el menú varía según el rol.

**Fuera de alcance (para specs futuras):**

- Registro público de usuarios. Solo `admin` crea cuentas (spec de usuarios).
- Recuperación de contraseña por correo.
- Bloqueo de cuenta tras intentos fallidos.
- Autenticación de dos factores.
- Gestión de roles y permisos por recurso (solo se valida el rol en el guard).

---

## Modelo de datos

**Backend** — `back/src/usuarios/entities/usuario.entity.ts` (la entidad se crea en la spec de usuarios; esta spec solo usa los campos listados):

```ts
// Campos que usa el login
correo: string;            // único, en minúsculas
contrasena_hash: string;   // bcrypt; nunca se expone
rol: 'admin' | 'docente' | 'estudiante';
activo: boolean;           // un usuario inactivo no puede iniciar sesión
```

**Backend** — nueva entidad `back/src/auth/entities/sesion-refresh.entity.ts`:

```ts
id: string;                // UUID
usuario_id: string;        // FK a usuarios.id
token_hash: string;        // SHA-256 del refresh token; nunca se guarda en texto plano
expira_en: Date;
revocada: boolean;
fecha_creacion: Date;
fecha_actualizacion: Date;
```

**Cambio de esquema:** Sí — requiere migración `CrearSesionesRefresh` (tabla `sesiones_refresh`). La tabla `usuarios` debe existir antes; si no existe, la migración no puede correr.

Convenciones:

- El refresh token se guarda como hash para que una filtración de la base de datos no permita usarlo.
- Cada login crea una fila nueva; el logout y el refresh rotado marcan la fila como `revocada`.

---

## Contrato de API

**`POST /api/v1/auth/login`** — público

- Body: `{ correo: string, contrasena: string }` (DTO `LoginDto`)
- `200` → `{ accessToken, refreshToken, usuario: { id, nombre, correo, rol } }` (DTO `SesionDto`). El refresh token viaja en cookie `httpOnly`, `Secure`, `SameSite=Strict`, y no aparece en el body.
- `400` → el correo no tiene formato válido o falta algún campo.
- `401` → correo o contraseña incorrectos, o usuario inactivo. El mensaje es el mismo en ambos casos para no revelar qué correos existen.

**`POST /api/v1/auth/refresh`** — público con cookie de refresh

- Body: vacío. Lee la cookie de refresh.
- `200` → nuevo `accessToken` y nuevo refresh token en cookie (rotación: el anterior queda `revocada`).
- `401` → no hay cookie, el token no existe, expiró, fue revocado o el usuario está inactivo.

**`POST /api/v1/auth/logout`** — requiere access token válido

- `204` → refresh token revocado y cookie borrada.
- `401` → falta el access token o expiró.

**Retrocompatibilidad:** los endpoints son nuevos; no cambian contratos existentes.

**Swagger:** los tres endpoints deben aparecer en `/api/docs` con descripción, ejemplos y todos sus códigos de respuesta.

---

## Interfaz y estados

**Dónde:** `front/src/modules/auth/pages/LoginPage.vue` (formulario con `QForm`, `QInput`, `QBtn`).

- **Inicial:** campos correo y contraseña vacíos; botón "Ingresar" habilitado.
- **Inválido:** el error aparece bajo el campo (formato de correo, campo vacío). El botón no envía la petición.
- **Cargando:** el botón muestra `loading` y queda deshabilitado.
- **Éxito:** redirige al dashboard único.
- **Credenciales incorrectas (401):** `Notify` con "Correo o contraseña incorrectos". El campo de contraseña se limpia; el correo se conserva.
- **Error de red o 5xx:** `Notify` con "No fue posible iniciar sesión. Intenta de nuevo." El formulario conserva lo escrito.

**Guard de rutas:** si una ruta requiere sesión y no hay access token ni refresh válido, redirige a `/login` y guarda la ruta original para volver a ella tras iniciar sesión.

**Sesión expirada:** si una petición devuelve 401 y el refresh falla, se limpia la sesión y se redirige a `/login` con el aviso "Tu sesión expiró".

**Reutilización:** `QForm` con `:rules`, `Notify` de Quasar. El access token vive en un store de Pinia (`front/src/stores/auth.ts`); no se guarda en `localStorage`.

---

## Plan de implementación

1. **Backend — migración:** crear la tabla `sesiones_refresh` con la migración `CrearSesionesRefresh` y la entidad `SesionRefresh`. Verificación: `npm run migration:run` (lo ejecuta la persona) deja la tabla creada y `npm run migration:revert` la elimina.
2. **Backend — hashing y JWT:** configurar `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET` y sus vigencias desde variables de entorno, y actualizar `.env.example`. Verificación: la app arranca sin esos valores y falla con un mensaje claro en español.
3. **Backend — login:** `LoginDto`, `AuthService.login` (compara con bcrypt, crea sesión) y `POST /auth/login` con Swagger completo. Verificación: credenciales válidas devuelven 200 y cookie; inválidas devuelven 401.
4. **Backend — refresh y logout:** `POST /auth/refresh` con rotación y `POST /auth/logout`, con Swagger completo. Verificación: el refresh anterior devuelve 401 tras rotar; el logout devuelve 204.
5. **Backend — guard JWT:** `JwtAuthGuard` en `common/guards/` que protege por defecto y un decorador `@Publico()` para marcar endpoints públicos. Verificación: un endpoint protegido sin token devuelve 401.
6. **Frontend — cliente y store:** `auth.service.ts` (llamadas a la API) y `stores/auth.ts` (Pinia). Verificación: el store guarda el access token en memoria y el usuario tras el login.
7. **Frontend — página y guard:** `LoginPage.vue`, ruta `/login` con carga diferida y guard de rutas. Verificación: sin sesión, `/` redirige a `/login`; con sesión, muestra el dashboard.

---

## Criterios de aceptación

- [ ] `POST /api/v1/auth/login` con correo y contraseña válidos responde `200` con `accessToken` y `usuario`, sin `contrasena_hash` ni `refreshToken` en el body.
- [ ] La respuesta de login incluye la cookie `refreshToken` con `httpOnly`, `Secure` y `SameSite=Strict`.
- [ ] `POST /api/v1/auth/login` con contraseña incorrecta responde `401` con el mismo mensaje que un correo inexistente.
- [ ] `POST /api/v1/auth/login` con un usuario `activo: false` responde `401`.
- [ ] `POST /api/v1/auth/login` con un correo sin formato válido responde `400`.
- [ ] `POST /api/v1/auth/refresh` con cookie válida responde `200` y el refresh anterior responde `401` en un segundo intento.
- [ ] `POST /api/v1/auth/logout` responde `204` y después el refresh de esa sesión responde `401`.
- [ ] En la base de datos, la columna del refresh token contiene un hash, no el token original.
- [ ] Un endpoint protegido sin access token responde `401`.
- [ ] La página `/login` muestra el error bajo el campo cuando el correo no es válido, sin llamar a la API.
- [ ] Credenciales incorrectas muestran `Notify` con "Correo o contraseña incorrectos" y limpian la contraseña.
- [ ] Sin sesión, entrar a una ruta protegida redirige a `/login`; tras iniciar sesión vuelve a la ruta original.
- [ ] Los endpoints `/api/v1/auth/login`, `/api/v1/auth/refresh` y `/api/v1/auth/logout` aparecen en `/api/docs` con descripción, ejemplos y todos sus códigos de respuesta.

---

## Decisiones tomadas y descartadas

- **Sí:** correo como identificador. Es único y estable; evita un campo de usuario extra.
- **No:** nombre de usuario. Duplica un identificador sin beneficio para el alcance actual.
- **Sí:** access token en memoria (Pinia) y refresh token en cookie `httpOnly`. Limita el daño de un ataque XSS.
- **No:** tokens en `localStorage`. Cualquier script de la página puede leerlos.
- **Sí:** access de 15 minutos y refresh de 7 días. El access corto reduce la ventana si se filtra.
- **Sí:** rotación del refresh token en cada uso. Un refresh robado queda inútil tras el siguiente uso legítimo.
- **Sí:** refresh token guardado como hash en `sesiones_refresh`. Permite revocarlo en logout.
- **No:** JWT sin estado para el refresh. Sin tabla no hay forma de cerrar una sesión antes de que expire.
- **Sí:** mensaje de error idéntico para correo inexistente y contraseña incorrecta. Evita enumerar cuentas.
- **No:** bloqueo de cuenta tras intentos fallidos. Se decidió dejarlo fuera de alcance (ver Riesgos).
- **Sí:** un solo dashboard para todos los roles. El menú cambia según el rol.

---

## Riesgos identificados

| Riesgo | Mitigación |
| ------ | ---------- |
| La entidad `usuario` y su spec no existen todavía, y el login depende de ella. | Antes de `/spec-impl` de esta spec, la spec de usuarios debe estar implementada, o esta spec debe incluir la entidad mínima. Decisión pendiente de la persona que aprueba. |
| Sin límite de intentos, el endpoint de login es vulnerable a fuerza bruta. | Se acepta por decisión explícita. Se recomienda una spec de rate limiting antes de producción. |
| El frontend pierde el access token al recargar la página. | El refresh en cookie se usa al cargar la app para recuperar sesión sin pedir credenciales de nuevo. |

---

## Lo que **no** incluye esta spec

- Registro público de usuarios.
- Recuperación de contraseña.
- Bloqueo de cuenta tras intentos fallidos o rate limiting.
- Autenticación de dos factores.
- Gestión de roles y permisos por recurso.
- CRUD de usuarios (spec de usuarios).

Cada uno, si llega, va en su propia spec.
