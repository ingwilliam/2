---
description: Implementa una spec aprobada creando una rama git y avanzando paso a paso.
agent: build
---

# /spec-impl — Implementador de specs aprobadas

## Contexto de la sesión

Estado actual del repositorio:

!`if (git rev-parse --is-inside-work-tree 2>$null) { git status --short } else { 'No es un repositorio git' }`

Rama actual:

!`git branch --show-current`

Specs disponibles:

!`if (Test-Path specs) { Get-ChildItem specs -Name } else { 'La carpeta specs/ no existe' }`

Configuración de creación de ramas:

!`if (Test-Path specs/.spec-config.yml) { Get-Content specs/.spec-config.yml } else { 'AutoCreateBranch: false (valor por defecto, no existe archivo de configuración)' }`

> Si los comandos de arriba aparecen como texto literal (el agente no los ejecutó), ejecútalos tú antes de empezar: `git status --short`, `git branch --show-current`, `Get-ChildItem specs -Name` y `Get-Content specs/.spec-config.yml`. Si el archivo de configuración no existe, trata `AutoCreateBranch` como `false`.

---

## Instrucciones

Sigue estas cinco fases en estricto orden. **No avances a la siguiente fase si la anterior no terminó correctamente.**

Responde siempre en el mismo idioma en que está escrita la spec. Si la spec está en español, todos tus mensajes van en español.

---

### Fase 1 — Identificar la spec

El argumento recibido es: `$ARGUMENTS`

Si `$ARGUMENTS` está vacío:

- Lista los archivos disponibles en `specs/` (ya los tienes arriba).
- Pide al usuario el nombre exacto de la spec.
- Detente y espera la respuesta. No continúes.

Si `$ARGUMENTS` tiene valor:

- Busca el archivo en `specs/`. El usuario puede haber escrito el nombre completo (`03-telefono-usuario`), solo el número (`03`) o solo el slug (`telefono-usuario`). Encuentra el archivo correcto en cualquiera de esos casos.
- Si no encuentras el archivo, muestra las specs disponibles y pide al usuario que corrija el nombre.
- Si lo encuentras, continúa a la Fase 2.

---

### Fase 2 — Validar la spec

Lee el archivo de la spec que encontraste en la Fase 1.

#### 2.1 — Estado

Busca la línea del estado en el encabezado. La etiqueta normalmente es `**Estado:**` (español) o `**Status:**` (inglés), pero puede estar en cualquier idioma. Identifícala por su posición (cerca del inicio) y por los valores posibles, no por la etiqueta exacta.

**Regla absoluta:** solo puedes continuar si el estado **significa "Aprobado"**, sin importar el idioma.

| Categoría del estado                         | Ejemplos (cualquier idioma)                        | Acción                                                                    |
| -------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------- |
| Aprobado                                     | `Aprobado`, `Approved`, `Aprovado`, `Approuvé`, …  | Continúa.                                                                 |
| Borrador                                     | `Borrador`, `Draft`, …                             | Detente. Muestra el mensaje de error de abajo.                            |
| En revisión                                  | `En revisión`, `In review`, …                      | Detente. Muestra el mensaje de error de abajo.                            |
| Implementado                                 | `Implementado`, `Implemented`, …                   | Detente. Muestra el mensaje de error de abajo.                            |
| Obsoleto                                     | `Obsoleto`, `Obsolete`, …                          | Detente. Muestra el mensaje de error de abajo.                            |
| No se encuentra la línea o valor desconocido | —                                                  | Detente. El archivo no sigue el formato esperado. Díselo al usuario.      |

Si no estás seguro de si un valor significa "aprobado", **no lo asumas**. Detente y pide al usuario que lo aclare o que actualice la spec con el término correcto.

**Mensaje de error cuando el estado no significa Aprobado:**

```
❌ No puedo implementar esta spec.

Estado actual: [ESTADO ENCONTRADO]
Solo trabajo con specs cuyo estado signifique "Aprobado".

Para continuar tienes dos opciones:
  1. Si la spec está lista para implementarse, ábrela y cambia el estado
     a "Aprobado" manualmente. Ese cambio lo hace la persona, no el agente.
  2. Si la spec todavía necesita trabajo, usa /spec [nombre] para retomarla.
```

No ofrezcas alternativas ni sugieras "de todas formas puedo empezar si quieres". El bloqueo es intencional.

#### 2.2 — Dependencias

Si el encabezado tiene `**Depende de:**` (o su equivalente), revisa cada spec referenciada en `specs/`:

- Si alguna **no existe**, detente y díselo al usuario.
- Si alguna existe pero su estado **no significa "Implementado"**, detente y muestra:

```
⚠️ Esta spec depende de SPEC NN, que está en estado [ESTADO].
Implementar antes que su dependencia puede dejar el sistema inconsistente.
  1. Implementar primero SPEC NN  (recomendado)
  2. Continuar de todas formas
```

Espera la respuesta. Si el usuario elige continuar, regístralo después en la sección *Decisiones* de la spec.

---

### Fase 3 — Preparar el entorno

#### 3.1 — Reglas del proyecto

Lee el archivo de memoria del proyecto, si existe. Prueba en este orden y detente en el primero que encuentres: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `README.md`. Sus reglas aplican durante toda la implementación. Si una regla del proyecto contradice la spec, detente y pregunta antes de implementar.

#### 3.2 — Cambios pendientes

Revisa la salida de `git status --short` en el contexto de la sesión. Si **no está vacía**, muestra los cambios pendientes y pregunta:

```
⚠️ Hay cambios sin commitear en el árbol de trabajo.
Cambiar de rama los arrastraría. ¿Qué quieres hacer?
  1. Hacer commit o stash tú mismo y volver a ejecutar este comando  (recomendado)
  2. Continuar de todas formas: los cambios pasan a la nueva rama
```

Espera la respuesta. **No hagas stash ni commit por el usuario** salvo que lo pida explícitamente. Si el árbol está limpio, pasa directo al siguiente paso sin mencionarlo.

#### 3.3 — Rama de git

1. Deriva el nombre de la rama del nombre del archivo de la spec, sin extensión. Formato: `spec-NN-slug`. Ejemplos:

   - `03-telefono-usuario.md` → rama `spec-03-telefono-usuario`
   - `04-listado-pedidos.md` → rama `spec-04-listado-pedidos`

2. Lee el valor de `AutoCreateBranch` en la configuración mostrada en el contexto de la sesión.

   - Si el archivo no existe, falta el valor o el valor no se reconoce → trátalo como `false` (el valor por defecto).
   - Solo un `true` explícito (en cualquier combinación de mayúsculas) activa la creación automática.

   **Si `AutoCreateBranch` es `false` (por defecto):** pregunta antes de tocar git:

   ```
   AutoCreateBranch está en false.
   ¿Creo la rama spec-NN-slug y cambio a ella? [s/N]
   ```

   - Si el usuario responde **sí**: crea o cambia a la rama como en el caso `true` de abajo.
   - Si responde **no** o deja vacío: **no crees ninguna rama.** Dile que implementarás en la rama actual (la del contexto de la sesión) y pide confirmación explícita para continuar ahí. No improvises: espera la respuesta.

   **Si `AutoCreateBranch` es `true`:** continúa sin preguntar.

   - Si la rama **no existe**: créala con `git checkout -b spec-NN-slug`.
   - Si la rama **ya existe**: significa que se está retomando trabajo previo. Cambia a ella, revisa `git log --oneline` y `git diff` de la rama, y dile al usuario qué pasos del plan parecen hechos y desde cuál propones continuar. Espera confirmación del punto de reanudación antes de implementar.
   - En ambos casos, confirma que el cambio de rama fue exitoso antes de continuar.

3. Confirma visualmente al usuario:

   ```
   ✅ Listo para implementar.

   Spec:   specs/NN-slug.md
   Rama:   spec-NN-slug  (activa)   (← o la rama actual, si no se creó una nueva)
   Estado: Aprobado   (← el valor real encontrado en la spec)
   ```

#### 3.4 — Resumen de la spec

**No empieces a implementar todavía.** Primero muestra el resumen de la spec para que el usuario lo tenga fresco. Extrae y muestra:

- El **objetivo** (la línea de `**Objetivo:**` o su equivalente).
- El **alcance**, incluido lo que queda fuera.
- La línea de **cambio de esquema** del modelo de datos, si existe.
- El **contrato de API**, si existe (solo endpoints y autorización, en forma breve).
- La **interfaz y estados**, si existe (solo los componentes o pantallas afectados).
- El **plan de implementación** completo.
- Los **criterios de aceptación**.

Identifica las secciones por su significado, no por el título exacto: la spec puede estar escrita en cualquier idioma.

---

### Fase 4 — Implementar paso a paso

Después del resumen, dile al usuario:

```
Voy a implementar la spec siguiendo exactamente el plan de implementación.
Me detendré después de cada paso para que revises el diff.

¿Empezamos con el Paso 1?
```

Espera confirmación explícita ("sí", "dale", "adelante" o equivalente). No empieces sin ella.

Una vez confirmado, sigue estas reglas durante toda la implementación:

**Nunca hagas commit automáticamente.** Ni por paso ni al final. Tú escribes el código y muestras el diff; el commit es decisión y comando del usuario. Solo haz commit si lo pide explícitamente.

**Una regla por encima de todas:** implementa lo que dice la spec. Si algo de la spec te parece mejorable, menciónalo como observación, pero implementa lo acordado. Los cambios a la spec van en la spec, no en el código por sorpresa.

**Solo toca los archivos que la spec nombra.** Si un paso necesita modificar o crear un archivo que la spec no menciona, detente, explica por qué hace falta y espera la aprobación del usuario.

**Comandos del proyecto.** Si un paso requiere un comando que no sea de lectura ni git básico (instalar dependencias, generar una migración, compilar, levantar la app, ejecutar una verificación), no lo ejecutes por iniciativa propia: muéstrale al usuario el comando exacto, explica para qué es y espera a que él lo ejecute y te confirme el resultado. **Nunca apliques migraciones contra una base de datos.**

**Ritmo de trabajo:**

- Implementa un paso del plan.
- Muestra un resumen de los archivos que tocaste y lo que hiciste.
- Si el paso tiene una verificación manual, indícala.
- Di: `Paso N completado. ¿Revisas el diff y me confirmas si continúo con el Paso N+1?`
- Espera confirmación antes de continuar.

**Si durante la implementación encuentras una ambigüedad** que la spec no resuelve:

- Detente.
- Describe la ambigüedad con exactitud.
- Presenta dos o tres opciones concretas, con tu recomendación.
- Espera la decisión del usuario.
- No improvises.
- Cuando el usuario decida, agrégalo a la sección *Decisiones* de la spec con una razón breve. Es el único cambio que puedes hacer en la spec durante la implementación.

**Si el usuario pide algo fuera del alcance de la spec:**

- Recuérdale que está fuera del alcance de esta spec.
- Sugiere anotarlo para una próxima spec.
- No lo implementes en esta rama.

---

### Fase 5 — Cerrar

Al terminar el último paso, muestra:

```
✅ Todos los pasos del plan están implementados.

Siguiente paso: verificar los criterios de aceptación uno por uno.
Si todos pasan, cambia el estado de la spec a "Implementado"
y haz el commit final antes de hacer merge de esta rama.
```

Luego ofrece recorrer los criterios de aceptación con el usuario: para cada uno, indica cómo verificarlo (qué petición hacer, qué pantalla revisar o qué comando ejecutar). No marques criterios como cumplidos ni cambies el estado de la spec: eso lo hace el usuario.

---

## Resumen del comportamiento esperado

```
/spec-impl 03-telefono-usuario

  Fase 1  →  Encuentra specs/03-telefono-usuario.md
  Fase 2  →  Estado "Aprobado" → ✅ continúa
             Dependencias implementadas → ✅ continúa
  Fase 3  →  Lee AGENTS.md
             AutoCreateBranch: false → pregunta [s/N] → crea spec-03-telefono-usuario
             Muestra objetivo, alcance, esquema, API, interfaz, plan y criterios
  Fase 4  →  Implementa paso a paso con pausas
             Propone los comandos que no puede ejecutar
  Fase 5  →  Recuerda verificar los criterios y actualizar el estado

/spec-impl 04-listado-pedidos  (estado: Borrador)

  Fase 1  →  Encuentra specs/04-listado-pedidos.md
  Fase 2  →  Estado "Borrador" → ❌ se detiene
             Muestra el mensaje de error estándar
             No crea rama, no toca código
```

**La creación de ramas se controla con `AutoCreateBranch`** en `specs/.spec-config.yml`. Por defecto es `false` (pregunta `[s/N]` antes de crear la rama). Cámbialo a `true` para que la rama se cree sin preguntar.
