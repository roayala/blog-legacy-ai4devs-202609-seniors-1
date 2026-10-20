# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

---

## Prompt 1

**Modelo:** Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code (CLI 2.1.292)

```
Vamos a auditar SOLO la feature de comentarios de este blog: crear, editar, borrar y listar comentarios, en sus tres capas (rutas de la API, tabla en la base de datos y lo que la web muestra de ellos). Nada de posts, autores, categorías ni etiquetas salvo donde toquen directamente a comentarios.

Es una auditoría de solo lectura: no modifiques código, no escribas tests, no crees ramas ni commits. Las reglas de TDD y la Definition of Done del CLAUDE.md no aplican a esta tarea; las "Reglas de negocio del dominio" sí: úsalas como contrato para decidir qué es un bug.

Paso 1 de 3: solo inventario, todavía sin hallazgos. Devuélveme en el chat una lista corta:
- API: cada ruta de comentarios (verbo + path) con su archivo:línea.
- BD: la migración de la tabla de comentarios y las columnas que existen de verdad, con archivo:línea; y si la semilla inserta comentarios, dónde.
- Web: cada archivo del frontend que pide, muestra, crea, edita o borra comentarios, con archivo:línea.
Saca cada número de línea abriendo el archivo, no de memoria. Si alguna de las cuatro operaciones no existe en alguna capa, dilo explícitamente en vez de suponerla.
```

**Qué salió:** Funcionó a la primera: inventario con archivo:línea; editar y borrar no existen en la web.

## Prompt 2

**Modelo:** Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code (CLI 2.1.292)

```
Paso 2 de 3: hallazgos. Sobre ese inventario, y solo dentro de la feature de comentarios, busca problemas: bugs, incumplimientos de las reglas de negocio del CLAUDE.md, filtración de datos, validación o autorización que falte, desajustes entre API, BD y web.

Escríbelos en docs/auditoria/comentarios.md (créalo), no en el chat, como UNA tabla con una fila por hallazgo y exactamente estas columnas:

| Qué es | Dónde | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro? |

Reglas por casilla:
- Qué es: un id (C1, C2…) y UNA frase. Si necesitas más, son dos hallazgos: sepáralos.
- Dónde: archivo:línea + el fragmento literal de esa línea entre backticks. Compruébala abriendo el archivo antes de escribirla. Si no puedes citar la línea que lo demuestra, la fila no se escribe.
- Qué se rompe y quién lo nota: efecto concreto y a quién le llega (lector anónimo, quien comenta, autor del post…). "Nadie todavía" es válido.
- Sitios a tocar: un número contado (grep/lectura), no estimado.
- Revert: "Sí" o "No, porque…".
- Bloquea/desbloquea: déjala vacía; se rellena en el paso 3.

Celdas cortas: la tabla tiene que caber en una pantalla. No ordenes, no propongas arreglos en la tabla y no metas nada fuera de comentarios (si ves algo grave fuera de alcance, una línea en el chat y ya). Si una afirmación depende de algo que no se ve en el código (runtime, datos reales, configuración), escribe "sin verificar: <qué faltaría>" en vez de suponerlo.

Al terminar, en el chat solo: el número total de hallazgos y, por cada uno, los archivo:línea que contaste en "Sitios a tocar", para que yo los compruebe.
```

**Qué salió:** Funcionó a la primera: 25 hallazgos en `docs/auditoria/comentarios.md`; el agente comprobó con un script cada fragmento citado y cada línea contada.

## Prompt 3

**Modelo:** Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code (CLI 2.1.292)

```
No añadas hallazgos nuevos ni recuperes los que quité.

1. Rellena "¿Bloquea o desbloquea a otro?" nombrando el id del otro (p. ej. "Desbloquea C4"). "A ninguno" es válido y será lo más frecuente; no inventes dependencias.
2. Reordena las filas:
   - Primero la precedencia: lo que abarata o desbloquea a otro hallazgo va antes que ese otro.
   - Dentro del mismo nivel, daño ÷ radio: daño = gravedad del efecto y a quién llega; radio = cuánto hay que tocar (sitios a tocar; si no se deshace con revert, pesa más).
   - Si un hallazgo tiene el daño más alto pero la precedencia más baja, pártelo en dos filas: "C3a, contención" (lo mínimo que corta el daño hoy, arriba) y "C3b, arreglo de fondo" (en su sitio por precedencia).
3. Lo que sea una propuesta cuyo efecto nadie ha medido (rendimiento, uso, "probablemente…") sácalo de la tabla a una sección "Propuestas sin medir", una línea por propuesta con lo que habría que medir al lado.
4. Debajo de todo deja "## Parte B" con tres viñetas vacías. No las rellenes: son mías.

Si dos hallazgos se disputan el puesto y el criterio no lo decide, no lo fuerces: déjalos contiguos y dime en el chat, en una línea, qué dos cosas chocan.
```

**Qué salió:** Funcionó, pero falló la premisa: el archivo no tenía filas quitadas, así que siguieron todas. Quedaron 24 filas (C2 partido en C2a/C2b; C7, C25 y el índice de C20 a «Propuestas sin medir») y un choque sin decidir entre C1 y C13.

## Prompt 4

**Modelo:** Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code (CLI 2.1.292)

```
Hay que hacer un pr, pero debemos incluir lo siguiente Un PR desde tu fork contra el repo del curso, solo con dos cosas:
- docs/auditoria/comentarios.md
- prompts.md: todos los prompts, incluidos los que fallaron, indicando modelo y herramienta.
```

**Qué salió:** Lo interrumpí a mitad y lo relancé con el prompt 5.

## Prompt 5

**Modelo:** Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code (CLI 2.1.292)

```
REpite la tarea pero no agregues autoria en el commit
```

**Qué salió:** Lo interrumpí a mitad: estaba recogiendo también los prompts de la sesión anterior al `/clear`.

## Prompt 6

**Modelo:** Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code (CLI 2.1.292)

```
No incluyas los prompts de la sesion previa, solo los de esta sesion
```

**Qué salió:** Este PR.
