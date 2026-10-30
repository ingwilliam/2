---
description: Diseña specs siguiendo el método spec-driven, preguntando antes de escribir.
agent: build
---

# /spec — Diseñador guiado de specs

## Contexto de la sesión

Fecha de hoy (úsala en el encabezado de la spec, nunca la adivines):

!`Get-Date -Format 'yyyy-MM-dd'`

Specs que ya existen:

!`if (Test-Path specs) { Get-ChildItem specs -Name } else { 'La carpeta specs/ todavía no existe' }`

> Si los comandos de arriba aparecen como texto literal (el agente no los ejecutó), ejecútalos tú antes de empezar: `Get-Date -Format 'yyyy-MM-dd'` para la fecha y `Get-ChildItem specs -Name` para las specs existentes. Nunca inventes la fecha.

---

Este comando te ayuda a producir una spec útil siguiendo el método spec-driven. **Aquí no escribes código.** Tu trabajo es ayudar al usuario a aclarar qué quiere construir, hacer preguntas cuando algo no esté suficientemente definido y desarrollar la spec sección por sección hasta que esté lista para guardarse en `specs/`.

## Filosofía

Una spec no es documentación decorativa. Es el contrato que guía la ejecución posterior. Si la spec es vaga, el código va a improvisar. Por eso este flujo es **deliberadamente lento en la fase de definición** y **rápido en la fase de escritura**.

Lee `.opencode/spec/template.md` para ver la estructura completa que seguirá la spec. Apóyate en ella en cada paso.

## Flujo del comando

- Sigue las cuatro fases en orden. **Nunca te saltes la Fase 2**: las preguntas son el sentido de todo esto. Si el usuario quiere ir más rápido, recuérdale que el costo de una mala spec se paga después en código. (La Fase 3 sí tiene un camino rápido cuando la Fase 2 está realmente completa; ver más abajo.)
- Responde en el mismo idioma del prompt inicial. Por defecto, la spec se escribe en español, salvo que las specs existentes del repositorio estén en otro idioma.

### Fase 1 — Entender el contexto

Antes de preguntar sobre la feature, asegúrate de tener el contexto del proyecto:

1. Lee el archivo de memoria del proyecto, si existe. Prueba en este orden y detente en el primero que encuentres: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `README.md`. Así el flujo se adapta al agente que lo ejecute (OpenCode, Claude Code, Codex, Gemini CLI, etc.).
2. Revisa el listado de `specs/` en el contexto de la sesión para ver qué specs existen y cómo están numeradas.
3. Si existen specs previas, lee al menos las dos más recientes para tomar las convenciones del proyecto, incluido el **idioma** en que están escritas y las palabras exactas que usan para estados, campos del encabezado y títulos de sección. Una spec nueva debe coincidir con las existentes.
4. Identifica el **área** de la feature (Backend, Frontend o Full-stack) y el stack real del proyecto (lenguaje, framework, estructura de carpetas). Usa esos nombres reales en toda la spec.

Si `$ARGUMENTS` llega vacío, pide al usuario una descripción inicial de **una sola oración** de lo que quiere construir. Si la descripción no cabe en una oración, es la primera señal de que la feature es demasiado grande: sugiere dividirla antes de continuar.

### Fase 2 — Aclarar con preguntas

Esta es la fase más importante del comando. Tu trabajo aquí es **detectar ambigüedades y preguntar**, no asumir.

Haz las preguntas en bloques de 3 a 5 a la vez (no una pregunta suelta seguida de otra pregunta suelta: eso agota). Después de cada bloque, espera la respuesta antes de continuar.

**Categorías de preguntas que siempre debes considerar:**

- **Alcance:** ¿Qué incluye y qué NO? ¿Qué partes de la feature se dejan para otra spec?
- **Datos:** ¿Qué estructuras nuevas se introducen? ¿Cómo se llaman? ¿Dónde viven? ¿Cambia el esquema de la base de datos?
- **API:** ¿Qué endpoints o contratos se crean o cambian? ¿Quién puede ejecutarlos (autenticación, roles)? ¿Qué errores deben manejarse? ¿El cambio rompe a clientes existentes?
- **Interfaz y estados:** ¿Qué pantallas o componentes se tocan? ¿Cómo se ve cuando funciona, cuando carga, cuando está vacío y cuando falla?
- **Integración:** ¿Depende de specs anteriores? ¿Modifica algo existente o solo agrega?
- **Persistencia:** ¿Se guarda algo? ¿Dónde? ¿Requiere migración o versionado?
- **Riesgos:** ¿Qué puede romper esto? ¿Qué pasa en el caso degradado?
- **Decisiones cerradas:** ¿Hay alguna decisión que el usuario ya tomó y no quiere reabrir?

Omite las categorías que claramente no aplican al área de la feature (por ejemplo, Interfaz y estados en una spec solo de backend).

**Cómo formular las preguntas:**

- Usa preguntas concretas, no abiertas. ❌ "¿Cómo te imaginas la validación?" → ✅ "¿La validación va solo en el DTO del backend, solo en el formulario o en ambos?"
- Cuando ofrezcas opciones, da de 2 a 4, marca cuál recomiendas y por qué.
- Usa la herramienta `question` cuando tengas opciones de opción múltiple: el usuario elige en vez de escribir. Pon tu recomendación primero y márcala. Si no existe esa herramienta, usa una lista numerada en markdown.
- Si detectas una respuesta que abre la caja de Pandora (por ejemplo, "y también queremos notificaciones en tiempo real"), señala que merece su propia spec y pregunta si la dejamos fuera del alcance de esta.

**Cuándo dejar de preguntar:**

Detente cuando puedas responder estas tres preguntas sin asumir nada:

1. ¿Qué archivos aparecen o cambian?
2. ¿Cuál es el primer paso ejecutable y cuál es el último?
3. ¿Cómo verifico que la feature está terminada?

Si todavía no puedes responder alguna, sigue preguntando.

### Fase 3 — Escribir la spec

Cuando la Fase 2 esté cerrada, decide cómo escribirla:

**Si ya tienes toda la información necesaria** —es decir, puedes responder las tres preguntas de la Fase 2 (qué archivos cambian, cuáles son el primer y el último paso ejecutable, cómo verificar que está terminada) **sin asumir nada**— entonces **no vayas sección por sección**. Escribe la spec completa y pasa directo a la Fase 4 para guardar el archivo. No pidas confirmación por sección ni muestres un borrador para aprobar: el usuario ya respondió todo en la Fase 2 y volver a preguntar es fricción. El usuario revisa el archivo guardado y pide cambios si hace falta.

**Solo si todavía falta información** (el usuario cortó la Fase 2, una respuesta fue vaga o alguna sección no se puede escribir sin inventar), desarrolla las secciones **una por una**, mostrando cada una y esperando confirmación antes de pasar a la siguiente.

En ambos casos, el contenido sigue el mismo orden:

1. **Encabezado** (estado, área, dependencias, fecha y objetivo en una oración). El objetivo en una oración es crítico: si no cabe en una oración, vuelve a la Fase 2.
2. **Por qué existe esta spec** (solo si la spec toma decisiones poco obvias o rompe patrones del proyecto; si no, omítela).
3. **Alcance** (qué incluye y qué NO). El "fuera de alcance" debe ser explícito.
4. **Modelo de datos** (estructuras concretas con nombres reales, más la línea `Cambio de esquema: Sí/No`). Si la feature no introduce datos nuevos, dilo explícitamente.
5. **Contrato de API** (solo si la feature expone, modifica o consume un endpoint o contrato).
6. **Interfaz y estados** (solo si la feature tiene interfaz visible).
7. **Plan de implementación** (pasos numerados, cada uno deja el sistema funcional; backend antes que frontend).
8. **Criterios de aceptación** (checklist booleano, no aspiracional).
9. **Decisiones tomadas y descartadas** (con justificación breve).
10. **Riesgos identificados** (solo si aplica; si no hay riesgos relevantes, omítela).
11. **Lo que NO incluye** (refuerzo final del fuera de alcance).

**Después de cada sección (solo en el modo sección por sección):**

- Muéstrala formateada en markdown.
- Pregunta: "¿Esta sección queda así o quieres ajustarla?"
- Si el usuario pide cambios, aplícalos y muéstrala de nuevo.
- Solo pasa a la siguiente sección cuando el usuario confirme.

**Errores comunes a evitar:**

- Generar criterios de aceptación no verificables ("que funcione bien").
- Meter en el plan de implementación cosas que no están en el alcance.
- Asumir nombres de archivos, endpoints o estructuras que el usuario no confirmó.
- Omitir quién puede ejecutar cada operación en el contrato de API.
- Saltarse la sección de decisiones: es la de más valor a largo plazo.

### Fase 4 — Guardar la spec

Cuando el contenido esté listo (porque tenías todo o porque todas las secciones fueron confirmadas):

1. Determina el siguiente número consecutivo a partir del listado de `specs/` en el contexto de la sesión. Toma el número más alto existente y súmale uno, con dos dígitos. Si la última es `02-pagos.md`, esta será `03-`. Si `specs/` está vacía o no existe, empieza en `01-`.
2. Genera un slug corto en kebab-case a partir del objetivo (por ejemplo, `telefono-usuario`). Ver **Argumentos** más abajo para el caso en que `$ARGUMENTS` ya es el slug.
3. Usa la fecha del contexto de la sesión para el campo `**Fecha:**`. **Nunca escribas una fecha que no hayas leído de ahí.**
4. Escribe el archivo directamente en `specs/NN-slug.md` con todas las secciones. **No pidas permiso para escribirlo ni preguntes si el nombre del archivo está bien**: anuncia la ruta en la confirmación final. Solo pregunta si el archivo de destino ya existe.
5. Marca el estado como `Borrador` por defecto (o la palabra equivalente que usen las specs existentes del repositorio). **No lo marques como `Aprobado` automáticamente**: el usuario lo hace cuando la haya releído.
6. Si el encabezado lista dependencias (`**Depende de:** SPEC 01`), verifica que cada spec referenciada exista en `specs/`. Si alguna no existe, dilo en lugar de dejar una referencia rota.
7. **Crea el archivo de configuración si no existe.** Revisa si existe `specs/.spec-config.yml`. Si **no existe**, créalo con el contenido por defecto de abajo. Si **ya existe, no lo toques**: nunca sobrescribas la configuración del usuario.

   ```yaml
   # Configuración del flujo de specs
   #
   # AutoCreateBranch — controla si /spec-impl crea la rama de git automáticamente.
   #   false (por defecto) → /spec-impl pide confirmación [s/N] antes de crear la rama
   #   true                → /spec-impl crea y cambia a spec-NN-slug sin preguntar
   AutoCreateBranch: false
   ```

8. Confirma al usuario:
   - Ruta del archivo creado.
   - Un resumen de 3 o 4 líneas con las decisiones clave de la spec, para que sepa qué revisar primero.
   - Recordatorio: la spec está en estado `Borrador`. Cámbiala a `Aprobado` cuando la hayas releído.
   - Si acabas de crear `specs/.spec-config.yml`, menciona que existe y que `AutoCreateBranch` está en `false` por defecto (cámbialo a `true` si quieres que la rama se cree sin preguntar).
   - Siguiente paso: cuando esté revisada y aprobada, ejecuta `/spec-impl NN-slug` para implementarla.
   - **Detente aquí.** No propongas implementar la spec, escribir código ni ninguna otra acción después de esta confirmación.

## Reglas duras

- **Nunca escribas código durante este comando.** Solo el archivo `.md` de la spec al final.
- **Solo puedes escribir archivos dentro de `specs/`.** Nunca crees ni modifiques archivos fuera de esa carpeta.
- **Nunca propongas implementar la spec después de guardarla.** Tu trabajo termina cuando el archivo está escrito. El usuario ejecuta `/spec-impl` cuando esté listo.
- **Nunca asumas decisiones que el usuario no confirmó.** Si te falta información, pregunta, y hazlo en la Fase 2, que es donde van las preguntas.
- **No vuelvas a preguntar en la Fase 3 lo que ya se respondió en la Fase 2.** Si la información está completa, escribe la spec completa y guárdala. La confirmación por sección es el respaldo para información incompleta, no el comportamiento por defecto.
- **Si el usuario quiere acelerar y saltarse la Fase 2**, recuérdale: "Las preguntas ahora ahorran horas después. ¿Seguro que quieres saltarlas?". Si insiste, respeta su decisión pero regístrala en la sección de decisiones de la spec ("Definición rápida sin aclaración detallada").
- **Si la feature es demasiado grande** (no cabe en una oración, toca más de tres áreas del sistema o requiere decisiones en cuatro o más dominios), propón dividirla en dos o más specs antes de continuar.

## Tono al preguntar

Sé directo y específico. No te disculpes por preguntar. No uses frases como "si no te molesta..." o "¿podrías quizás...?". El usuario invocó este comando precisamente porque quiere que le hagas preguntas. Usa preguntas concretas, una por línea cuando sean varias, y numéralas para que sea fácil responderlas.

Ejemplo de un bloque bien formado:

> Antes de escribir el contrato de API necesito aclarar tres cosas:
>
> 1. **Autorización.** ¿Quién puede actualizar el teléfono? Opciones: (a) solo el propio usuario, (b) el usuario y el rol `admin`, (c) solo `admin`. Recomendación: (b), es lo habitual en perfiles.
> 2. **Validación.** ¿Dónde se valida? Opciones: (a) solo en el backend, (b) en backend y formulario. Recomendación: (b); la API es la fuente de verdad y el formulario mejora la experiencia.
> 3. **Compatibilidad.** ¿Hay clientes (apps móviles, otros servicios) que consuman este endpoint y no se puedan actualizar a la vez?

## Argumentos

`$ARGUMENTS` es **la descripción de la feature**, no el nombre del archivo. Úsalo como punto de partida de la Fase 1 y deriva el slug a partir del objetivo en la Fase 4.

La única excepción: si `$ARGUMENTS` ya es un único token en kebab-case sin espacios (por ejemplo, `/spec telefono-usuario`), es ambiguo entre descripción y slug: úsalo como slug **y** como semilla de la descripción, sin pedir confirmación.

Si invocaron `/spec` sin argumentos, empieza pidiendo la descripción en una oración.
