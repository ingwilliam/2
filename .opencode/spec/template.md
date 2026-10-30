# Plantilla para una spec útil

Este archivo es la referencia que consulta el skill `/spec` al generar specs. Cada sección incluye su propósito y un ejemplo mínimo. **No es texto para copiar tal cual**: es la forma que el skill debe respetar.

Los ejemplos usan una sola feature de principio a fin: **"El usuario puede registrar un teléfono opcional"**, que toca backend (entidad, DTO, endpoint) y frontend (formulario de perfil). Los ejemplos están en TypeScript; usa siempre el lenguaje, el framework y los nombres de archivo reales del proyecto.

---

## Encabezado

Toda spec empieza con metadatos en un blockquote (sin tablas ni bloques, simple como se muestra):

```markdown
# SPEC NN — Título corto y descriptivo

> **Estado:** Borrador
> **Área:** Backend | Frontend | Full-stack
> **Depende de:** SPEC 01, SPEC 02
> **Fecha:** AAAA-MM-DD
> **Objetivo:** Una sola oración. Si necesitas dos, la feature es demasiado grande.
```

**Estados válidos:** `Borrador`, `En revisión`, `Aprobado`, `Implementado`, `Obsoleto`.

> Usa un solo conjunto de estados por repositorio y mantente consistente.

**Regla del objetivo:** una oración que una persona lee en 5 segundos y entiende qué se va a construir. Si no cabe en una oración, divide la feature.

Ejemplo: _"El usuario puede registrar y editar un teléfono opcional de 10 dígitos desde su perfil."_

---

## Por qué existe esta spec (opcional)

Para specs que toman decisiones poco obvias o rompen patrones del proyecto, una sección breve que explique el **por qué** del trabajo. No el qué: el qué viene después.

En specs simples, omítela.

---

## Alcance

Dos bloques explícitos. **Ambos son obligatorios.**

```markdown
## Alcance

**Incluye:**

- Campo opcional `telefono` en la entidad Usuario.
- Aceptar `telefono` en los endpoints de crear y actualizar usuario.
- Campo de teléfono en el formulario de perfil.

**Fuera de alcance (para specs futuras):**

- Verificación por SMS.
- Números internacionales e indicativos de país.
```

**Por qué importa el "fuera de alcance":** registra lo que el usuario mencionó durante la fase de preguntas pero se decidió dejar para después. Sin ese registro, durante la implementación aparece la tentación de meterlo "ya que estamos".

---

## Modelo de datos

Las estructuras concretas que aparecen o cambian. Usa código real, no pseudocódigo abstracto. Cubre solo las capas que toca la feature.

~~~markdown
## Modelo de datos

**Backend** — `src/usuarios/entities/usuario.entity.ts`:

```ts
// Nueva columna
telefono: string | null; // 10 dígitos, opcional
```

**Cambio de esquema:** Sí — requiere migración (`AgregarTelefonoUsuario`).

**Frontend** — `src/features/usuarios/types.ts`:

```ts
type Usuario = {
  id: string;
  nombre: string;
  email: string;
  telefono: string | null; // nuevo
};
```

Convenciones:

- El teléfono se guarda solo con dígitos, sin espacios ni símbolos.
- `null` significa "no registrado"; nunca se guarda un string vacío.
~~~

**Regla del cambio de esquema:** toda spec que toque persistencia debe decir explícitamente `Cambio de esquema: Sí` o `Cambio de esquema: No`. Esa línea define si la migración forma parte del plan.

Si la feature no introduce datos nuevos, escríbelo explícitamente: _"Esta feature no introduce estructuras de datos nuevas. Reutiliza el modelo de la SPEC 01."_

---

## Contrato de API (cuando aplique)

Obligatoria cuando la feature expone, modifica o consume un endpoint. Omítela en features que no pasan por la red.

```markdown
## Contrato de API

**`POST /usuarios`** y **`PATCH /usuarios/:id`**

- **Autorización:** usuario autenticado, solo sobre su propio perfil; rol `admin` sobre cualquiera.
- El body agrega: `telefono?: string | null`
- `201` / `200` → `UsuarioDto` incluyendo `telefono`
- `400` → `telefono` viene y no tiene exactamente 10 dígitos
- `404` → (solo PATCH) el usuario no existe

**`GET /usuarios/:id`**

- `200` → `UsuarioDto` ahora incluye `telefono` (`null` si no está registrado)
```

**Reglas:**

- Indica siempre quién puede ejecutar cada operación (autenticación, roles o permisos). Si es pública, escríbelo.
- Lista todos los códigos de respuesta que el cliente debe manejar, no solo el caso exitoso.
- Indica si el cambio es **retrocompatible**. Agregar un campo opcional normalmente lo es; renombrar o quitar uno no, y debe decirse.
- Nombra el DTO o tipo exacto que viaja en la petición y la respuesta.
- Si no es REST, describe el contrato equivalente: query o mutation de GraphQL, método gRPC, mensaje de WebSocket o evento de cola, con su payload y sus errores.

---

## Interfaz y estados (cuando aplique)

Obligatoria cuando la feature tiene interfaz visible. Omítela en specs solo de backend.

```markdown
## Interfaz y estados

**Dónde:** `src/features/usuarios/components/FormularioPerfil.tsx`

- **Inicial:** el campo muestra el teléfono actual o aparece vacío.
- **Inválido:** mensaje "El teléfono debe tener 10 dígitos" bajo el campo; botón de guardar deshabilitado.
- **Guardando:** el botón muestra un spinner y queda deshabilitado.
- **Éxito:** toast "Perfil actualizado".
- **Error (API 4xx/5xx):** toast con el mensaje de la API; el formulario conserva lo escrito.
```

**Reglas:**

- Cada pantalla o componente lista como mínimo los estados de carga, vacío (si muestra datos), error y éxito.
- Nombra los componentes o librerías existentes que se reutilizan; no inventes nuevos sin registrarlo en Decisiones.

---

## Plan de implementación

Pasos numerados. Cada paso debe dejar el sistema en un estado **funcional y ejecutable**. Nada de "implementar la mitad y seguir mañana".

```markdown
## Plan de implementación

1. Agregar la columna `telefono` en `usuario.entity.ts` y generar la migración `AgregarTelefonoUsuario`. Verificación: la app arranca y la columna existe.
2. Agregar `telefono` validado en `create-usuario.dto.ts` y `update-usuario.dto.ts`. Verificación: `POST /usuarios` con `"123"` responde 400.
3. Incluir `telefono` en `UsuarioDto`. Verificación: `GET /usuarios/:id` devuelve `telefono`.
4. Agregar `telefono` al tipo `Usuario` y al cliente de API del frontend.
5. Agregar el campo de teléfono y sus estados en `FormularioPerfil.tsx`.
```

**Reglas:**

- Cada paso debe poder commitearse por sí solo.
- Si un paso requiere más de 30–50 líneas de código, divídelo.
- Ordena los pasos para que el contrato del backend exista antes de que el frontend lo consuma.
- Cada paso termina con una verificación manual corta.
- El último paso del plan **no** es "probar todo": eso son los criterios de aceptación.

---

## Criterios de aceptación

Checklist booleano. Cada ítem se verifica con sí o no.

```markdown
## Criterios de aceptación

- [ ] `POST /usuarios` con `telefono: "3001234567"` guarda y devuelve el teléfono.
- [ ] `POST /usuarios` con `telefono: "123"` responde 400.
- [ ] `POST /usuarios` sin `telefono` guarda el usuario con `telefono: null`.
- [ ] Los usuarios existentes siguen cargando después de la migración.
- [ ] El formulario de perfil muestra el error si hay menos de 10 dígitos.
- [ ] Guardar un teléfono válido muestra el toast "Perfil actualizado".
```

**Antipatrones a evitar:**

- ❌ "Que funcione bien." → no es verificable.
- ❌ "Buena experiencia de usuario." → es subjetivo.
- ❌ "Sin errores." → no es operativo.
- ✅ "Enviar un teléfono de 9 dígitos muestra 'El teléfono debe tener 10 dígitos' y no llama a la API." → verificable y booleano.

---

## Decisiones tomadas y descartadas

La sección con más valor dentro de 3 meses. Registra **lo que consideraste**, no solo lo que elegiste.

```markdown
## Decisiones

- **Sí:** columna opcional (nullable). Los usuarios existentes no tienen teléfono y no deben romperse.
- **No:** tabla separada `contactos`. Es sobreingeniería para un solo campo.
- **Sí:** validación en el DTO y en el formulario. La API es la fuente de verdad; el formulario solo mejora la experiencia.
- **No:** librería de máscaras en el frontend. Una validación simple basta por ahora.
```

Cada decisión idealmente lleva una razón breve. Las decisiones sin razón son las primeras que se cuestionan después.

---

## Riesgos identificados (opcional)

Solo cuando hay riesgos poco obvios. Tabla simple:

```markdown
## Riesgos

| Riesgo                                          | Mitigación                                                              |
| ----------------------------------------------- | ----------------------------------------------------------------------- |
| La migración corre sobre una tabla grande       | Columna nullable sin valor por defecto; no reescribe la tabla.          |
| Versiones antiguas del frontend ignoran el campo | El campo es opcional; los clientes antiguos siguen funcionando sin él. |
```

En specs pequeñas o features muy acotadas, omítela.

---

## Lo que NO incluye (refuerzo)

Repite explícitamente al final lo que **no** se hará en esta spec. La repetición es deliberada: la sección de Alcance ya lo dice, pero al final del documento sirve de recordatorio para quien solo lee las últimas líneas.

```markdown
## Lo que **no** incluye esta spec

- Verificación por SMS.
- Números internacionales e indicativos de país.
- Mostrar el teléfono en el listado de usuarios del administrador.

Cada uno, si llega, va en su propia spec.
```

---

## Reglas globales del documento

- **Una oración por idea.** Si una oración tiene dos comas y un punto y coma, divídela.
- **Nombres concretos.** Si dices "el módulo de usuarios", escribe `src/usuarios/`. Si dices "un endpoint", da el método y la ruta. Si dices "un componente", da el archivo.
- **Solo las capas que aplican.** Una spec solo de backend omite Interfaz y estados; una spec de frontend que consume un endpoint existente lo referencia en lugar de redefinirlo.
- **Sin TODOs.** Un TODO en una spec significa que la decisión no se tomó. Tómala o regístrala como decisión pendiente con su razón.
- **Sin código ejecutable largo.** La spec describe; el código se escribe después. Fragmentos cortos para ilustrar estructuras de datos están bien; funciones completas no.
- **Markdown estándar.** Sin extensiones raras. Debe verse bien en GitHub sin sorpresas.
