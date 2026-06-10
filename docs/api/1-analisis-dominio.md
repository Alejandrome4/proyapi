# Análisis de Dominio - Proyecto de Alquiler de Campos de Fútbol

He preparado este documento para la parte de la API de la entrega intermodular. La idea es ver cómo se conectan las tablas que diseñamos en Bases de Datos con los datos que se van a mover en el Frontend mediante archivos JSON para el tema de los alquileres de los campos (fútbol 7, fútbol 11 y fútbol sala).

## 1. Qué tablas de la Base de Datos usamos
Para que funcione el flujo de las reservas de los partidos (el CRUD básico), tocamos tres tablas:
* **reservas:** Es la tabla principal donde se guarda el día, las horas en las que se juega, si está pagada/pendiente y lo que cuesta el alquiler.
* **usuarios:** Para guardar los datos del chaval o el capitán que alquila el campo.
* **recursos / campos:** Para listar las pistas que tiene el polideportivo (por ejemplo, el campo de césped 1 o la pista de fútbol sala cubierta).

## 2. Campos que se envían por la API vs Base de Datos
He estado revisando qué campos de la tabla `reservas` hacen falta en la API y cuáles maneja el backend por su cuenta para que sea seguro:

* `id`: En la base de datos es la clave primaria autoincremental. El frontend no lo envía cuando alguien rellena el formulario de reservar (POST), pero el backend nos lo tiene que devolver en la respuesta para poder identificar el partido si luego queremos borrarlo o cambiar la hora (`/reservas/{id}`).
* `usuario_id`: Tampoco lo va a poner el usuario a mano en la web. El backend lo pilla directamente del token de la sesión cuando el usuario inicia sesión. Lo dejamos en el JSON de respuesta para comprobar de quién es el partido.
* `recurso_id`: Este sí se expone. El usuario tiene que marcar obligatoriamente el ID del campo que quiere (por ejemplo, el ID 2 para el campo de Fútbol 7).
* `fecha_inicio` y `fecha_fin`: Son obligatorios para saber cuándo empieza y termina el partido. Los ponemos con el formato ISO 8601 (con la T y la Z) como vimos en los apuntes de JSON.
* `num_personas`: Lo pedimos para tener controlado cuánta gente va a entrar a las instalaciones y que no se saturen los vestuarios.
* `estado`: Al hacer el POST para pedir pista, el backend le pone automáticamente el valor "pendiente". No dejamos que el frontend lo mande como "confirmada" directamente para que nadie reserve gratis sin pasar por pasarela de pago o validación.
* `precio`: Lo calcula el servidor por detrás. Multiplica el tiempo que dura el partido por lo que cuesta la hora de ese campo concreto (el césped artificial sale más caro que el pabellón de cemento). El frontend solo lo lee.

## 3. Lógica y validaciones que tiene que hacer el Backend
Para que la aplicación no rompa la base de datos, el backend tiene que comprobar estas tres reglas antes de guardar nada:
1. **Que las fechas tengan sentido:** El partido tiene que ser en un día y hora futuros, y la hora de acabar no puede ser antes de la de empezar.
2. **El solapamiento (lo más importante):** El sistema tiene que comprobar en la BBDD que el campo que se pide no esté ya ocupado por otro partido a esa misma hora. Si hay choque de horarios, frena la reserva y salta el error `400 Bad Request`.
3. **Mínimo de jugadores:** Que al menos vaya a jugar una persona.

## 4. Captura de Swagger UI
Aquí abajo dejo la captura de pantalla de cómo se ve la documentación de las rutas de fútbol en local usando lo de starting-swagger.

![Swagger UI Fútbol](./captura-swagger-futbol.png)
