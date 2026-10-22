# Auditoría — comentarios

Solo lectura · 2026-10-19 · rama `s6/start` (`a87a414`) · datos: `backend/tmp/db.sqlite3` local · orden: precedencia, luego daño ÷ radio.

Rutas abreviadas: `routes.ts` = `backend/start/routes.ts` · `migración` = `backend/database/migrations/1769000000005_create_comentarios_table.ts` · `blog_seeder.ts` = `backend/database/seeders/blog_seeder.ts` · `Comentarios.jsx` / `DetallePost.jsx` = `frontend/src/componentes/…`.

| Qué es | Dónde | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro? |
|---|---|---|---|---|---|
| **C1** · El detalle del post devuelve la fila cruda de cada comentario, con `autor_email` (regla 6). | `routes.ts:153` `select * from comentarios where post_id = ?` | Cualquier lector anónimo lee en el JSON el correo de quien comenta (4 hoy); la web no lo pinta. | 2 | Sí | A ninguno |
| **C13** · DELETE y PATCH autorizan solo con un correo que manda el cliente, sin autenticación. | `routes.ts:350` `const email = request.input('autor_email', '')` | Con el correo que expone C1, cualquier lector anónimo borra comentarios aprobados ajenos. | 2 | Sí | A ninguno |
| **C14** · La clave de moderación está escrita en el código. | `routes.ts:25` `const SECRETO_SESION = 'dev-secret-cambiar'` | Quien lea el repo borra cualquier comentario con la cabecera `x-clave-moderacion`. Sin verificar: qué clave corre en producción. | 2 | Sí | A ninguno |
| **C2a, contención** · El detalle filtra `estado = 'aprobado'` exacto y oculta los aprobados guardados como 'Aprobado' u 'ok'. | `routes.ts:153` `and estado = 'aprobado'` | El lector no ve 3 comentarios (ids 3, 4, 9); quien los escribió cree que se rechazaron. Sin verificar: que 'ok' signifique aprobado. | 1 | Sí | A ninguno |
| **C10** · Tras insertar, el POST lee el último comentario de la tabla en lugar del recién creado. | `routes.ts:330` `select * from comentarios order by id desc limit 1` | Con dos envíos a la vez, quien comenta puede recibir el comentario y el correo de otra persona. Sin verificar: reproducirlo con peticiones concurrentes. | 1 | Sí | A ninguno |
| **C12** · Ninguna ruta aprueba comentarios: el único UPDATE solo cambia `cuerpo`. | `routes.ts:388` `update comentarios set cuerpo = ? where id = ?` | Lo enviado desde la web se queda 'pendiente' para siempre; quien comenta nunca lo ve publicado. Sin verificar: si se modera fuera del repo («panel viejo», `blog_seeder.ts:509`). | 0 (la ruta no existe) | Sí | A ninguno |
| **C18** · Borrar un comentario deja huérfanas sus respuestas. | `routes.ts:359` `delete from comentarios where id = ?` | Borrar el id 1 haría desaparecer de la web la respuesta aprobada id 2; hoy hay 0 huérfanas. | 1 | Sí | A ninguno |
| **C24** · Tras enviar, la sección suma 1 al contador por un comentario pendiente que no se muestra. | `Comentarios.jsx:39` `setTotal(total + 1)` | Quien comenta ve «3 comentarios» con 2 en pantalla y la cabecera en 2; al recargar vuelve a 2. | 1 | Sí | A ninguno |
| **C6** · La validación del correo solo comprueba que haya una «@». | `routes.ts:316` `String(email).indexOf('@') === -1` | Se guardan «@» o «a@»; quien teclea mal su correo no se entera. | 1 | Sí | A ninguno |
| **C22** · El formulario no envía `padre_id`, así que la web pinta hilos pero no deja responder. | `Comentarios.jsx:33` `JSON.stringify({ autor_nombre: nombre, autor_email: email, cuerpo: cuerpo })` | Quien comenta no puede responder a nadie; solo hay respuestas de la semilla. | 2 | Sí | A ninguno |
| **C9** · El POST acepta cualquier `padre_id` sin comprobar que exista, sea del mismo post o esté aprobado. | `routes.ts:308` `request.input('padre_id', null)` | Una respuesta aprobada cuyo padre no está en el listado no se pinta nunca (`Comentarios.jsx:50`); quien responde no se ve. | 1 | Sí | A ninguno |
| **C21** · La web solo pinta un nivel de respuestas. | `Comentarios.jsx:64` `.filter((r) => r.padre_id === c.id)` | Una respuesta a una respuesta, aunque esté aprobada, no la ve ningún lector. Nadie todavía desde la web (C22). | 1 | Sí | A ninguno |
| **C15** · PATCH compara el correo distinguiendo mayúsculas y espacios; DELETE no. | `routes.ts:379` `comentario.autor_email !== email` | Quien comentó como «Ana@…» y teclea «ana@…» puede borrar su comentario pero no editarlo (403). | 1 | Sí | A ninguno |
| **C17** · PATCH acepta un cuerpo de 1 carácter donde el POST exige 5. | `routes.ts:384` `if (!cuerpo) {` | Quien comenta puede dejar su comentario pendiente en «a»; lo recibe el moderador. | 1 | Sí | A ninguno |
| **C8** · Nombre y cuerpo hechos solo de espacios pasan la validación. | `routes.ts:310` `if (!nombre) {` | El moderador recibe comentarios sin autor ni texto; si los aprueba, el lector ve uno en blanco. | 3 | Sí | A ninguno |
| **C5** · Cada fallo de validación del POST usa un código y una forma de cuerpo distintos. | `routes.ts:314` `status(422).send({ message: 'Falta el email' })` | Quien integre la API maneja 400 `error`, 422 `message`, 400 `errores` y 200 `motivo`; la web ya pierde uno (C23). | 4 | Sí | Abarata C4 y C23 |
| **C11** · POST y PATCH devuelven la fila cruda con `autor_email`, `estado` y `post_id`. | `routes.ts:333` `send({ comentario: nuevos[0] })` | Nadie todavía: quien comenta recibe su propio correo; con C10, el de otro. | 2 | Sí | A ninguno |
| **C16** · PATCH no acepta la clave de moderación que DELETE sí acepta. | `routes.ts:379` `if (comentario.autor_email !== email \|\| comentario.estado !== 'pendiente') {` | El moderador no puede corregir un comentario, solo borrarlo. Sin verificar: si el contrato pide que pueda. | 1 | Sí | A ninguno |
| **C20** · `post_id` y `padre_id` no tienen clave foránea. | `migración:14` `table.integer('post_id').notNullable()` | Nadie todavía; la BD no frena C9 ni C18. | 2 | No, porque toca el esquema de una BD ya migrada | A ninguno |
| **C19** · `estado` es texto libre sin CHECK ni enum, y la BD ya guarda 'Aprobado' y 'ok'. | `migración:18` `table.string('estado').notNullable().defaultTo('pendiente')` | Nadie directamente: es el origen de C2a/C2b y C3; hoy hay 3 filas así (ids 3, 4, 9). | 4 | No, porque toca filas ya guardadas | Abarata C2b y C3 |
| **C3** · Los recuentos suman todo estado distinto de 'pendiente', no solo los aprobados (regla 5). | `routes.ts:75` `and c.estado <> '` | El lector ve «4 comentarios» del post 1 en el listado y 2 en su detalle; del post 6, 1 frente a 0. | 2 | Sí | A ninguno |
| **C23** · La web no lee `errores`, así que un correo sin «@» solo muestra un aviso genérico. | `Comentarios.jsx:44` `d.error \|\| d.message \|\| d.motivo` | Quien comenta con el correo mal escrito no sabe qué corregir. | 1 | Sí | A ninguno |
| **C4** · Un cuerpo de menos de 5 caracteres responde 200 en vez de un error. | `routes.ts:320` `return response.send({ ok: false, motivo: 'comentario demasiado corto' })` | Un cliente que mire el código HTTP lo toma por éxito; la web no, porque lee `motivo`. | 1 | Sí | A ninguno |
| **C2b, arreglo de fondo** · El filtro del detalle solo es fiable si `estado` guarda un único valor para «aprobado». | `routes.ts:153` `and estado = 'aprobado'` | Con C2a puesto, nadie; un valor sucio nuevo volvería a ocultar comentarios hasta C19. | 1 | Sí | A ninguno |

## Propuestas sin medir

- Longitud máxima para nombre, correo y cuerpo (era C7, `routes.ts:310`, `:316`, `:319`, `:384`) — medir: el tope real del parser JSON por defecto (`config/bodyparser.ts` no fija `limit` en `json`) y el tamaño de los comentarios que llegan.
- Índice en `comentarios.post_id` (salía en C20, `migración:14`) — medir: tiempo de las subconsultas de recuento (`routes.ts:75`, `:223`) con el volumen real de comentarios.
- Una sola petición de `GET /posts/:slug` por visita al detalle (era C25, `Comentarios.jsx:19` y `DetallePost.jsx:34`) — medir: coste de la segunda petición (latencia y carga de la API) por visita.

## Parte B

1. **Hallazgos devueltos por el agente:** 25  **Comprobados abriendo el archivo:** 9 
2. **No supe decidir la prioridad de** tuve dudas en que era lo que pretendia lograr este código, las instrucciones siguen siendo muy ambiguas con poca claridadlo que dificulta entender que es lo que se pide. Para mi la prioridad fue entregar los prompts que lograran lo que se entendió **porque** C1 contra C13: uno expone el correo de quien comenta y el otro deja borrar comentarios usando justo ese correo. Si tú tampoco supiste ordenarlos, es la respuesta que piden. 
3. **El agente afirmó y no pude verificar:** C El agente no afirmó nada, entendió lo que debía entender, más bien yo, Raúl, no enentdí muy bien que era lo que se quería lograr con el objetivo de este ejercicio, espero realmente que el día de la clase se pueda revisar a detalle que esra lo que se esperaba

   - C14: qué clave de moderación corre en producción.
   - C2a: si "ok" significa aprobado.
   - C10: lo de las peticiones concurrentes.
   - C12: lo de moderar desde un "panel viejo".
   - C16: qué pide el contrato.
