# Respuestas - SQL Gimnasio FitVal

## PARTE A

### A1

**A1.1 ¿Qué columnas no se repiten?**
id, dni y email.

**A1.2 ¿Cuál es la PK y cuáles las claves alternativas?**
La PK es `id`. Las alternativas (UNIQUE) son `dni` y `email`.

**A1.3 ¿Qué columna apunta a otra tabla?**
`plan_id`, que apunta a la tabla `planes`.

**A1.4 ¿Por qué usar id y no el DNI como PK?**
Porque el DNI puede cambiar o tener algún error al escribirlo, mientras que el `id` nunca cambia y al ser un número va más rápido en los JOIN.

### A2 - Tipos de relación

| Pareja | A a B | B a A | Relación |
| --- | --- | --- | --- |
| planes - socios | muchos | uno | 1:N |
| socios - tarjetas | una | uno | 1:1 |
| monitores - clases | muchas | uno | 1:N |
| socios - clases | muchas | muchos | N:M |
| monitores - especialidades | muchas | muchos | N:M |

### A3 - Monitores y clases

**A3.1 Las dos preguntas.**
Un monitor da muchas clases y una clase la da un solo monitor. Es una relación 1:N.

**A3.2 ¿Dónde va la FK?**
En la tabla `clases`, con la columna `monitor_id`.

**A3.3 ¿Qué pasa si va en el lado equivocado?**
Si la ponemos en monitores quedaría algo así:
id: 1 | nombre: Carla | clase_id: 1, 4, 7 (mal)

Falla porque en una celda solo podemos guardar un valor, no varios separados por comas.

**A3.4 ¿NOT NULL?**
Sí, porque toda clase tiene que tener un monitor asignado.

**A3.5 ¿Qué tabla se crea primero?**
Primero `monitores`, porque `clases` tiene la FK que apunta a ella y si no existe daría error.

### A4 - Socios y tarjetas

**A4.1 ¿Qué pasa sin UNIQUE en socio_id?**
Que un mismo socio podría tener dos tarjetas distintas (por ejemplo TJ-0001 y TJ-0099 con `socio_id = 1`) y entonces la relación pasaría a ser 1:N en vez de 1:1.

**A4.2 ¿Qué evita cada UNIQUE?**
- `codigo`: evita que haya dos tarjetas con el mismo código.
- `socio_id`: evita que un mismo socio tenga más de una tarjeta.

**A4.3 Otra relación 1:1.**
Persona y carné de conducir. La FK iría en `carnes` (`persona_id UNIQUE`), porque el carné depende de la persona.

### A5 - Monitores y especialidades

**A5.1 Tipo de relación.**
Un monitor puede tener muchas especialidades y una especialidad la pueden tener muchos monitores. Relación N:M.

**A5.2 ¿Qué tablas hacen falta?**
Hacen falta 3 tablas: `monitores`, `especialidades` y la intermedia `monitor_especialidades`.

**A5.3 Columnas.**
- `monitores`: id (PK), dni (UNIQUE), nombre, email (UNIQUE).
- `especialidades`: id (PK), nombre (UNIQUE).
- `monitor_especialidades`: id (PK), monitor_id (FK), especialidad_id (FK), y el UNIQUE(monitor_id, especialidad_id).

**A5.4 ¿Qué evita el UNIQUE de la intermedia?**
Evita que le asignemos la misma especialidad dos veces al mismo monitor.

**A5.5 ¿En qué se parecen y en qué se diferencian reservas y monitor_especialidades?**
Se parecen en que las dos son tablas intermedias para relaciones N:M con dos FK y un UNIQUE de la pareja. La diferencia es que `reservas` además guarda un dato extra propio de esa relación (`fecha_reserva`).

### A6 - ¿Dónde va la FK?

| # | Caso | Relación | Dónde va la FK |
| --- | --- | --- | --- |
| 1 | País - ciudad | 1:N | `ciudades.pais_id` |
| 2 | Persona - pasaporte | 1:1 | `pasaportes.persona_id` (con UNIQUE) |
| 3 | Alumno - asignatura | N:M | Tabla intermedia `matriculas` |
| 4 | Cliente - pedido | 1:N | `pedidos.cliente_id` |
| 5 | Película - actor | N:M | Tabla intermedia `repartos` (con campo `personaje`) |
| 6 | Autor - libro | Depende | Si es 1 autor: `libros.autor_id`. Si pueden ser varios (N:M): tabla intermedia `libros_autores` |
| 7 | User - Order | 1:N | `orders.user_id` |
| 8 | Order - Product | N:M | Tabla intermedia `order_items` (con `cantidad` y `precio_unitario`) |


## PARTE B

### B3 - Romper la base de datos

**B3.1 Socio con el plan 99.**
Va a fallar porque el plan 99 no existe en la tabla `planes`. Salta el error de foreign key `socios_plan_id_fkey` por la restricción en `plan_id`.

**B3.2 Socio con el email de Ana.**
Falla porque el email tiene la restricción UNIQUE. Da error de valor duplicado en `socios_email_key`.

**B3.3 Segunda tarjeta para Ana.**
Falla. Como `socio_id` es UNIQUE en la tabla tarjetas para que sea 1:1, salta el error de restricción única `tarjetas_socio_id_key`.

**B3.4 Ana reserva otra vez la clase 1.**
También falla, porque en la tabla `reservas` pusimos el `UNIQUE(socio_id, clase_id)` para que no pueda apuntarse dos veces a la misma clase. Salta `reservas_socio_id_clase_id_key`.

**B3.5 Reserva de la clase 50.**
Da error de clave foránea (`reservas_clase_id_fkey`) en `clase_id` porque la clase 50 no existe en `clases`.

**B3.6 Borrar el plan Básico.**
Falla porque todavía hay socios que tienen asignado el `plan_id = 1`, así que la FK `socios_plan_id_fkey` no deja borrarlo.

**B3.7 Borrar el plan Mañanas.**
Este sí funciona y devuelve `DELETE 1`.

**B3.8 ¿Por qué B3.6 falla y B3.7 no?**
Porque el plan Básico tiene socios apuntando a él en la tabla `socios`, pero el plan Mañanas no lo tiene ningún socio asignado y por eso deja borrarlo sin problema.

### B4.11 - ¿Por qué Ana sale tres veces?
No es ningún error, sale tres veces porque Ana tiene hechas tres reservas distintas y el JOIN saca una fila por cada reserva que encuentra.

### B5.6 - Baja de Pablo
Si intentas hacer el DELETE directo de Pablo en `socios` falla porque su tarjeta y su reserva siguen apuntando a su `id`. Para borrarlo bien hay que ir en orden:
1. Borrar sus reservas
2. Borrar su tarjeta
3. Borrar al socio

### B6 - Explícalo con tus palabras

**B6.1 PK vs UNIQUE.**
Ninguna de las dos permite valores repetidos, pero la PK es el identificador principal de la fila (solo puede haber una por tabla y nunca puede ser NULL), mientras que columnas con UNIQUE puede haber varias en la misma tabla.

**B6.2 ¿Por qué una FK se puede repetir?**
Porque muchas filas pueden apuntar al mismo registro de la otra tabla (por ejemplo, muchos socios teniendo el mismo plan).

**B6.3 ¿Dónde va la FK en una 1:N?**
En la tabla del lado "muchos" (N), porque si la pusiéramos en el lado "uno" tendríamos que meter varios valores en la misma celda.

**B6.4 ¿Qué convierte una 1:N en 1:1?**
Añadirle un `UNIQUE` a la columna de la FK.

**B6.5 ¿Por qué una N:M necesita tabla intermedia?**
Porque en ninguna de las dos tablas podríamos guardar varios IDs en una sola celda. Con la tabla intermedia se soluciona guardando una fila por cada pareja relacionada.

**B6.6 ¿Por qué fecha_reserva va en reservas?**
Porque esa fecha no es un dato fijo del socio ni de la clase, sino del momento en que ese socio hace esa reserva en concreto.
