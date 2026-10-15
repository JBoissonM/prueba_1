# Guía de Arquitectura y Estructura de Carpetas

Nuestro proyecto está configurado como un **Monorepo** (un solo repositorio que aloja múltiples proyectos) orquestado con **Bun Workspaces**. Se divide en tres piezas principales dentro de la raíz:

1. `apps/frontend`: La aplicación web visual hecha con **React y Vite**.
2. `apps/backend`: El servidor y API REST hecho con **NestJS, TypeORM y PostgreSQL**.
3. `packages/shared`: Un paquete intermedio para compartir código e interfaces de TypeScript entre el frontend y el backend sin tener que escribirlas dos veces.

---

## Parte 1: Frontend (`apps/frontend/src/`)

El frontend utiliza una arquitectura conocida en la industria como **Feature-Sliced Design (Diseño dividido por funcionalidades)**. En lugar de meter todas las pantallas de todo el sistema en una sola carpeta gigante, agrupamos el código según el **módulo de negocio** al que pertenece.

### 1. Carpeta `app/`

* **¿Por qué se llama así?** Viene de *Application* (Aplicación). Representa el "núcleo" o configuración general que envuelve a todo el proyecto de React.
* **¿A qué se refiere y para qué sirve?** Contiene la configuración que hace que la aplicación funcione como un todo antes de mostrar cualquier pantalla: el enrutador, los estados globales y la seguridad de navegación.
* **¿Qué contiene y cómo se usa?**
* `router.tsx` (*Enrutador*): Es el mapa de URLs del sistema. Aquí se define qué página se muestra cuando el usuario entra a `/login`, `/eventos` o `/mis-boletos`.
* `providers.tsx` (*Proveedores de contexto*): En React, un *Provider* es un componente que "provee" información global a toda la aplicación (por ejemplo, saber qué usuario tiene la sesión iniciada o mostrar alertas flotantes).
* `guards/` (*Guardias de ruta*): Son componentes que actúan como cadeneros en la puerta de una ruta. Antes de dejar pasar al usuario a una página, revisan si tiene sesión activa o el rol necesario; si no, lo redirigen al inicio de sesión.


### 2. Carpeta `assets/`

* **¿Por qué se llama así?** En desarrollo web, *Assets* significa "recursos estáticos" o activos multimedia.
* **¿A qué se refiere y para qué sirve?** Guarda archivos que **no son código** (no son `.ts` ni `.tsx`), sino archivos visuales o multimedia que Vite empaquetará junto con la página.
* **¿Qué contiene y cómo se usa?**
* `icons/`: Iconos en formato `.svg` o `.png`.
* `images/`: Logotipos institucionales, fondos o imágenes por defecto. Se importan directamente en los componentes (`import logo from '@/assets/images/logo.png'`).


### 3. Carpeta `components/`

* **¿Por qué se llama así?** En React, toda la interfaz se construye uniendo piezas llamadas *Components* (Componentes).
* **¿A qué se refiere y para qué sirve?** Almacena **únicamente los componentes visuales genéricos y globales** que se reutilizan en dos o más módulos distintos del sistema.
* **¿Qué contiene y cómo se usa?**
* `layout/` (*Estructura o maquetación*): Contiene el "esqueleto" visual que rodea a las páginas, como la barra de navegación superior (`Navbar.tsx`), el menú lateral o el contenedor base (`MainLayout.tsx`).
* `ui/` (*User Interface / Interfaz de Usuario*): Contiene los "bloques de LEGO" más básicos y tontos del diseño: botones (`Button.tsx`), cuadros de texto (`Input.tsx`), ventanas emergentes (`Modal.tsx`) o tablas vacías.
* **Regla de uso:** Un componente dentro de `ui/` **nunca** debe saber nada del negocio (por ejemplo, no debe llamarse `BotonCancelarEvento.tsx`, sino simplemente `Button.tsx` y recibir el texto por propiedades).


### 4. Carpeta `features/`

* **¿Por qué se llama así?** *Feature* significa "funcionalidad" o "módulo de negocio".
* **¿A qué se refiere y para qué sirve?** Es el corazón del frontend. Cada subcarpeta aquí dentro representa un módulo funcional del proyecto (por ejemplo: `auth`, `events`, `tickets`). Su objetivo es que cada subequipo trabaje dentro de su propia carpeta sin chocar ni generar conflictos en Git con los demás equipos.
* **¿Cómo se organiza una *feature* por dentro y cómo se usa?**
* `pages/` (*Páginas*): Son las pantallas completas de esa funcionalidad (ej. `EventsPage.tsx`). Son las únicas que se conectan directamente al `router.tsx`.
* `components/` (*Componentes locales*): Son piezas visuales que **solo le importan a este módulo** (ej. `EventCard.tsx` para mostrar el recuadro de un evento). Al ponerlos aquí, no ensuciamos la carpeta global `src/components/`.
* `api/` (*Interfaz de Programación de Aplicaciones*): Archivos con las funciones que hacen peticiones HTTP (`fetch` o `axios`) hacia el backend para pedir o enviar datos de este módulo (ej. `getEvents()`, `createEvent()`).
* `hooks/`: Lógica y manejo de estado exclusiva de este módulo (ver explicación de *hooks* más abajo).
* `types/`: Interfaces de TypeScript exclusivas para los formularios o componentes de esta pantalla.
* `index.ts` (*Archivo barril / Barrel file*): Es la "puerta de salida" del módulo. Exporta únicamente lo que el resto de la aplicación necesita ver de esta *feature* (como sus páginas para el `router.tsx`).


### 5. Carpeta `hooks/`

* **¿Por qué se llama así?** En React, los *Hooks* ("ganchos") son funciones especiales cuyos nombres siempre empiezan con el prefijo `use` (como `useState` o `useEffect`) que permiten "enganchar" estado y ciclo de vida a los componentes.
* **¿A qué se refiere y para qué sirve?** Guarda **Custom Hooks (Hooks personalizados) globales** que resuelven problemas técnicos repetitivos en cualquier pantalla.
* **¿Cómo se usa?** Por ejemplo, si necesitan detectar cuando el usuario lleva 30 minutos sin mover el ratón para cerrar su sesión por inactividad, se crea aquí un archivo `useIdleTimer.ts` y se invoca donde se necesite.


### 6. Carpeta `lib/`

* **¿Por qué se llama así?** Es la abreviatura estándar de *Libraries* (Librerías o bibliotecas externas).
* **¿A qué se refiere y para qué sirve?** Cuando instalamos una herramienta externa (por ejemplo, un cliente para hacer peticiones al servidor o una librería para manejar fechas), no queremos configurarla desde cero en cada archivo. En `lib/` guardamos la **configuración base (wrapper)** de esas herramientas.
* **¿Cómo se usa?** Aquí se crea, por ejemplo, `apiClient.ts`, donde se configura una sola vez que todas las peticiones apunten al prefijo `/api` (el cual Vite redirige automáticamente a `http://localhost:3000` en desarrollo) y que adjunten el token de sesión del usuario.


### 7. Carpeta `styles/`

* **¿Por qué se llama así?** Viene de *Stylesheets* (Hojas de estilo).
* **¿A qué se refiere y para qué sirve?** Contiene los archivos `.css` globales de toda la aplicación, como la paleta de colores en variables CSS, las fuentes tipográficas y el reseteo de márgenes del navegador.


### 8. Carpeta `types/`

* **¿Por qué se llama así?** Viene de *Data Types* (Tipos de datos de TypeScript).
* **¿A qué se refiere y para qué sirve?** Define la forma que tienen los objetos en el código para que el editor nos avise si escribimos mal una propiedad.
* **¿Cómo se usa?**
* Si un tipo de dato lo usan tanto el Backend como el Frontend (por ejemplo, la interfaz `Evento`), se coloca en el paquete compartido `packages/shared` y se importa con `@eventoos/shared`.
* Si un tipo es exclusivo del diseño visual de toda la aplicación de React (por ejemplo, las opciones de un menú de navegación), se coloca en esta carpeta `src/types/`.


### 9. Carpeta `utils/`

* **¿Por qué se llama así?** Es la abreviatura de *Utilities* (Utilidades o funciones auxiliares).
* **¿A qué se refiere y para qué sirve?** Contiene **funciones puras de JavaScript/TypeScript** (funciones simples que reciben un dato, lo transforman y devuelven un resultado, sin depender de React ni de la base de datos).
* **¿Cómo se usa?** Aquí van funciones como `formatDateToCDMX(date)` para mostrar fechas en formato de 24 horas, o validadores de formato de correo electrónico.


### 10. Archivos raíz (`main.tsx` y `App.tsx`)

* `main.tsx`: Es el punto de entrada técnico que busca el `<div id="root">` en el archivo `index.html` e inyecta nuestra aplicación de React en el navegador.
* `App.tsx`: Es el componente raíz de React donde se unen los `providers.tsx` y el `router.tsx`.

---

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
