# Simulacro de prueba de desempeño
## Desarrollo de una aplicación fullstack con Next.js 16

### Descripción del proyecto
El objetivo es desarrollar **BidHub**, una plataforma de subastas en línea de artículos de colección (cartas, relojes, vinilos, figuras, etc.) donde los usuarios pueden registrarse, iniciar sesión, publicar sus propios artículos para subasta y ofertar en las subastas de otros usuarios. Los administradores gestionan las categorías y moderan las subastas.

La aplicación debe ser fullstack: no se permite consumir una API externa para obtener los datos. Toda la información debe vivir en una base de datos propia y exponerse mediante route handlers de Next.js.

---

### Requisitos de negocio

#### 1. Roles
- **Usuario:** puede registrarse, iniciar sesión, explorar subastas, publicar sus propias subastas y ofertar en las de otros usuarios. Un mismo usuario puede ser vendedor en unas subastas y postor en otras.
- **Administrador:** además de lo anterior, puede gestionar las categorías y cancelar subastas como moderador. Este rol se crea mediante seed, no desde el registro público.

#### 2. Funcionalidades
- Registro e inicio de sesión de usuarios.
- Catálogo público de subastas, con filtro por categoría y por estado, y vista de detalle de cada subasta con su historial de ofertas.
- Publicación de subastas por parte de un usuario autenticado.
- Ofertas en subastas por parte de un usuario autenticado.
- Visualización de las subastas publicadas y de las ofertas realizadas por el usuario.
- Gestión de categorías y moderación de subastas por parte del administrador.

#### 3. Reglas de negocio

**Publicación**
1. Al publicar una subasta se define un precio inicial (mayor a cero), un incremento mínimo (mayor a cero) y una fecha de cierre.
2. La fecha de cierre debe estar entre 1 hora y 7 días a partir del momento de la publicación.
3. Cada subasta pertenece a una categoría existente.
4. Un usuario puede tener máximo 5 subastas activas al mismo tiempo como vendedor.
5. El vendedor puede editar su subasta únicamente mientras no tenga ofertas, y solo mientras esté activa.

**Ofertas**
1. Un usuario no puede ofertar en su propia subasta.
2. Solo se puede ofertar en una subasta activa, es decir, que no esté cancelada y cuya fecha de cierre no haya llegado.
3. La primera oferta debe ser mayor o igual al precio inicial. Las siguientes deben ser mayores o iguales a la oferta más alta actual más el incremento mínimo.
4. Un usuario que ya tiene la oferta más alta de una subasta no puede volver a ofertar en ella.
5. Las ofertas no se pueden editar ni eliminar.
6. Si se recibe una oferta cuando faltan 5 minutos o menos para el cierre, el cierre se extiende 5 minutos más. Esto puede repetirse tantas veces como lleguen ofertas dentro de ese margen.

**Cierre**
1. Una subasta tiene uno de cuatro estados: activa, cerrada (llegó la fecha de cierre y tiene ofertas, el ganador es el autor de la oferta más alta), desierta (llegó la fecha de cierre sin ofertas) o cancelada.
2. El estado de una subasta debe ser correcto en cualquier momento en que se consulte, sin depender de tareas programadas ni de procesos externos que cierren las subastas.

**Cancelación y moderación**
1. El vendedor puede cancelar su subasta únicamente si no tiene ofertas.
2. El administrador puede cancelar cualquier subasta activa, incluso con ofertas, indicando un motivo obligatorio que queda registrado.
3. Una subasta cancelada o cerrada no se elimina, se conserva con su historial.

**Categorías**
1. Una categoría con subastas asociadas no se puede eliminar.

**Privacidad y seguridad**
1. El historial público de ofertas muestra el monto y la fecha, pero no revela el nombre completo ni el correo de los postores (por ejemplo, se muestra un alias parcial).
2. Un usuario no puede editar ni cancelar subastas de otro usuario, incluso si cambia el id en la URL.
3. Un usuario solo puede ver el listado completo de sus propias ofertas.

---

### Requerimientos técnicos

#### 1. Estructura del proyecto
Configura un proyecto Next.js 16 con App Router y TypeScript, asegurando una estructura bien organizada de carpetas, componentes y layouts reutilizables. El código debe ser limpio, ordenado y entendible. No se exige una arquitectura específica.

#### 2. Rutas y route groups
- Organiza las rutas utilizando route groups: `(public)`, `(auth)`, `(private)` y `(admin)`, de forma que no afecten la URL.
- Implementa rutas dinámicas con `[id]` para visualizar el detalle de cada subasta.
- Páginas mínimas: Home, Catálogo de subastas, Detalle de la subasta, Registro, Login, Dashboard (mis subastas y mis ofertas), Publicar subasta y Panel de administración.

#### 3. Layouts y componentes
- Implementa un root layout con la estructura base de la aplicación.
- Crea un layout para cada route group con un diseño propio:
  - `(public)`: navbar y footer.
  - `(auth)`: diseño centrado y simple.
  - `(private)`: navbar con menú de usuario.
  - `(admin)`: layout propio del panel.
- Crea componentes compartidos y reutilizables (navbar, footer, botones, formularios, tarjetas, etc.), con una organización clara de carpetas. Los estilos son libres.

#### 4. Estrategias de renderizado
Aplica la estrategia de renderizado adecuada en cada página:
- **SSG**: Home.
- **SSR**: catálogo y detalle de la subasta.
- **CSR**: dashboard del usuario.

#### 5. Route handlers
Crea route handlers con Next.js para:
- **Autenticación**: registro, login, refresh y logout.
- **Subastas**: listar, ver detalle, crear, editar y cancelar.
- **Ofertas**: crear una oferta, listar el historial de una subasta y listar las ofertas propias.
- **Categorías**: listar (público), crear, editar y eliminar (administrador).
- **Moderación**: cancelar una subasta como administrador.

Todos deben validar la entrada y responder con un formato de error consistente y códigos HTTP correctos (400, 401, 403, 404, 409).

#### 6. Base de datos y ORM
- Diseña un modelo relacional coherente con los requisitos de negocio.
- Utiliza un ORM a elección. Por ejemplo: Prisma, TypeORM, MikroORM o Drizzle.
- Implementa migraciones y un seed que cree el administrador, categorías y subastas de ejemplo en distintos estados (activas, cerradas, desiertas y canceladas).
- Base de datos PostgreSQL alojada en Supabase.
- Las reglas de negocio deben garantizarse en el backend, no solo en la interfaz.

#### 7. Autenticación y autorización
- Registro e inicio de sesión con contraseñas almacenadas de forma segura (hasheadas).
- Implementa access token y refresh token, ambos enviados en cookies httpOnly, secure y sameSite. No se permite guardarlos en localStorage ni devolverlos en el body.
- El refresh token debe almacenarse hasheado en la base de datos, y el logout debe revocarlo.
- Implementa el flujo de renovación del access token sin que el usuario deba volver a iniciar sesión.
- Los route handlers validan la sesión y el rol por su cuenta, sin depender únicamente del proxy.

#### 8. Protección de rutas con proxy
Utiliza `proxy.ts` para proteger las rutas privadas y de administrador, redirigiendo al login a quien no tenga sesión y bloqueando a los usuarios sin rol de administrador en las rutas de admin.

#### 9. Internacionalización (i18n)
Implementa soporte multilingüe para que la interfaz esté disponible en español e inglés, con un selector de idioma visible. La solución de i18n es libre.

#### 10. Hidratación
Incluye al menos un componente que dependa de datos del cliente, como una cuenta regresiva del tiempo restante de la subasta, sin generar errores de hidratación.

#### 11. Optimización
Utiliza `next/image` para las imágenes de los artículos.

#### 12. Docker
Incluye un Dockerfile que construya y ejecute la aplicación correctamente. El `docker-compose` es opcional.

#### 13. Variables de entorno
Incluye un archivo `.env.example` con todas las variables que necesita el proyecto, sin valores reales ni secretos. Ejemplo:
```env
DATABASE_URL=
JWT_ACCESS_SECRET=
JWT_REFRESH_SECRET=
ACCESS_TOKEN_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=7d
```

#### 14. README
El README es un entregable evaluado. Debe permitir que cualquier persona, partiendo de cero y siguiendo solo ese documento, ejecute el proyecto completo. Debe incluir:
1. Descripción breve del proyecto.
2. Requisitos previos: qué debe tener instalado o creado la persona (versión de Node.js, gestor de paquetes, Docker, cuenta de Supabase, etc.).
3. Variables de entorno: una explicación de cada variable del `.env.example`: para qué sirve, qué valor lleva y cómo obtenerlo (por ejemplo, de dónde se saca la cadena de conexión en Supabase y cómo generar los secretos de los tokens).
4. Ejecución en desarrollo, con los comandos exactos y en orden:
  1. Clonar el repositorio e instalar dependencias.
  2. Crear el `.env` a partir del `.env.example` y completarlo.
  3. Ejecutar las migraciones.
  4. Ejecutar el seed.
  5. Levantar el proyecto en modo desarrollo.
5. Credenciales del administrador creado por el seed.
6. Ejecución con Docker: comandos para construir la imagen y correr el contenedor, indicando cómo pasarle las variables de entorno.
7. Scripts disponibles en el `package.json` y qué hace cada uno.
8. Estructura de carpetas del proyecto, explicada brevemente.
9. URL de la aplicación desplegada.
Si no es posible ejecutar el proyecto siguiendo el README, se penaliza.

#### 15. Despliegue
Despliega la base de datos en Supabase y la aplicación en Vercel, asegurando que todas las funcionalidades (route handlers, rutas dinámicas, autenticación) funcionen correctamente.

#### 16. Calidad del código
- TypeScript sin `any`.
- Nombres claros, funciones con una sola responsabilidad y código sin duplicación.

---

### Puntos extra
- Rate limiting en el login y en las ofertas.
- Idempotencia al crear ofertas.
- Manejo de concurrencia cuando dos usuarios ofertan al mismo tiempo.
- `docker-compose`.
- Pipeline de CI.
- Pruebas de al menos una regla de negocio.
