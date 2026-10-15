# Guía de Arquitectura y Estructura de Carpetas

## Parte 2: Backend (`apps/backend/src/`)

El backend está construido con **NestJS**, un framework de backend que obliga a ordenar el código siguiendo una **Arquitectura Modular y en Capas** (Controladores, Servicios y Acceso a Datos). Esto evita el "código espagueti" donde las consultas SQL se mezclan con las rutas HTTP.

### 1. Carpeta `common/`

* **¿Por qué se llama así?** *Common* significa "común" o compartido entre todos los módulos.
* **¿A qué se refiere y para qué sirve?** Contiene las piezas especiales de NestJS que se encargan de la seguridad, validación y formato de todas las peticiones antes o después de que lleguen a la lógica de negocio.
* **¿Qué subcarpetas tiene y qué significa cada concepto en NestJS?**
* `guards/` (*Guardias de seguridad*): Clases que deciden si una petición HTTP tiene permiso de ejecutarse o no. Revisan el token de sesión del usuario y su rol; si no tiene permiso, devuelven un error `401 Unauthorized` o `403 Forbidden` automáticamente.
* `pipes/` (*Tuberías de validación/transformación*): Son filtros por los que pasan los datos que envía el usuario antes de llegar al controlador. Sirven para transformar datos (ej. convertir un `"12"` de texto en el número `12`) o rechazar datos mal formados.
* `decorators/` (*Decoradores personalizados*): En TypeScript, los decoradores son etiquetas con un `@` (como `@Get()` o `@Column()`) que añaden funcionalidad extra. Aquí creamos etiquetas propias, como `@Roles('ADMIN')` para indicar fácilmente qué rol requiere un endpoint.
* `filters/` (*Filtros de excepciones*): Capturan cualquier error que ocurra en el servidor y lo convierten en una respuesta JSON limpia y estandarizada con el mensaje de error correspondiente para el frontend.
* `interceptors/` (*Interceptores*): Funcionan como cámaras de vigilancia que interceptan la petición al entrar y al salir. Son ideales para guardar registros automáticos en la bitácora de auditoría sin ensuciar el código de cada módulo.


### 2. Carpeta `config/`

* **¿Por qué se llama así?** Abreviatura de *Configuration* (Configuración).
* **¿A qué se refiere y para qué sirve?** Centraliza la lectura de las variables de entorno del archivo `.env` (como puertos, claves secretas para tokens o credenciales de correo), asegurando que si falta alguna variable crítica al encender el servidor, avise de inmediato.


### 3. Carpeta `database/`

* **¿Por qué se llama así?** *Database* (Base de datos).
* **¿A qué se refiere y para qué sirve?** Contiene todo lo referente a la conexión con **PostgreSQL** y el historial de estructura de tablas manejado por **TypeORM**.

* **¿Qué contiene y cómo se usa?**
* `data-source.ts` (*Fuente de datos*): Es el archivo que lee las credenciales del `.env` (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`) y establece la conexión única hacia PostgreSQL.
* `migrations/` (*Migraciones*): Como en el proyecto está configurado `synchronize: false` por seguridad, la base de datos nunca se modifica sola. Una migración es un archivo con instrucciones exactas para crear o modificar tablas que se comparte por Git para que todos los integrantes tengan exactamente la misma base de datos al ejecutar `bun run db:migrate`.
* `seeds/` (*Semillas de datos*): Scripts que insertan los datos iniciales obligatorios cuando la base de datos está recién creada y vacía (por ejemplo, crear la primera cuenta de Administrador o el catálogo inicial de recintos).


### 4. Carpeta `modules/`

* **¿Por qué se llama así?** En NestJS, la aplicación se divide en *Modules* (Módulos de dominio). Incluso el propio generador del framework los crea con el comando `bun --bun run nest g resource <nombre>`.
* **¿A qué se refiere y para qué sirve?** Cada subcarpeta dentro de `modules/` encapsula un área del negocio (ej. `auth`, `users`, `events`, `venues`, `reservations`).
* **¿Qué contiene un módulo por dentro y cómo funciona cada pieza?**
* `controllers/` (*Controladores* — `*.controller.ts`): Son los **recepcionistas** del backend. Su único trabajo es escuchar una ruta URL (ej. `POST /api/events`), recibir los datos del cliente, llamar al servicio correspondiente y devolver la respuesta. **Regla:** Un controlador nunca debe hacer consultas directas a la base de datos ni tener cálculos complejos.
* `services/` (*Servicios* — `*.service.ts`): Son el **cerebro** del módulo. Aquí vive toda la **lógica de negocio y las reglas del sistema** (validar que no haya traslape de horarios, verificar que queden lugares disponibles, descontar cupos, etc.) y es quien se comunica con las entidades de la base de datos.
* `dto/` (*Data Transfer Objects / Objetos de Transferencia de Datos* — `*.dto.ts`): Son clases que definen **qué datos exactos esperamos recibir del usuario** en una petición (por ejemplo, en `create-event.dto.ts` especificamos que el nombre debe ser texto obligatorio y la fecha válida). Si el usuario manda basura o deja campos vacíos, el DTO rechaza la petición antes de que toque la base de datos.
* `entities/` (*Entidades* — `*.entity.ts`): Son la representación en código TypeScript de las **tablas de PostgreSQL** usando decoradores de TypeORM (`@Entity()`, `@Column()`, `@PrimaryGeneratedColumn()`). TypeORM detecta automáticamente cualquier archivo terminado en `.entity.ts` gracias a la configuración del proyecto.
* `[nombre].module.ts`: Es la "caja" que registra y conecta a los controladores, servicios y entidades de esa carpeta para que NestJS sepa cómo inyectarlos.


### 5. Archivos raíz (`main.ts` y `app.module.ts`)

* `app.module.ts`: Es el módulo raíz donde se unen la conexión de TypeORM y todos los módulos de la carpeta `modules/`.
* `main.ts`: Es el archivo que arranca el servidor NestJS, aplica el prefijo global `/api` a todas las rutas y pone al servidor a escuchar en el puerto `3000`.

---

## Parte 3: ¿Cómo se conecta todo? (Ejemplo del flujo completo)

Para que cualquier integrante entienda cómo viaja la información cuando programa una tarea, este es el recorrido exacto cuando un usuario hace clic en **"Reservar Lugar"**:

1. **En `features/reservations/pages/` (Frontend):** El usuario hace clic en el botón de confirmar reserva dentro de la página.
2. **En `features/reservations/api/` (Frontend):** La página llama a la función `createReservation(datos)` que envía una petición `POST` a `/api/reservations` usando el cliente configurado en `lib/`.
3. **En `common/guards/` (Backend):** El servidor recibe la petición. Antes de procesar nada, el `JwtAuthGuard` verifica que el usuario tenga una sesión válida y activa.
4. **En `modules/reservations/dto/` (Backend):** El objeto `CreateReservationDto` revisa que el ID del evento y el asiento enviados tengan el formato correcto.
5. **En `modules/reservations/controllers/` (Backend):** El `ReservationsController` recibe la petición ya validada y llama a `this.reservationsService.create(datos, usuario)`.
6. **En `modules/reservations/services/` (Backend):** El `ReservationsService` ejecuta las reglas de negocio (verifica que el usuario no tenga ya un boleto para ese evento y que el asiento siga libre).
7. **En `modules/reservations/entities/` (Backend):** El servicio usa la entidad `Reservation` de TypeORM para guardar el registro en **PostgreSQL** y devuelve la confirmación al controlador, el cual responde al Frontend para mostrar el mensaje de éxito en pantalla.
