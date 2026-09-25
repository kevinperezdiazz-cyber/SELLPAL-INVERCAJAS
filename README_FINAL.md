# SELLPAL integrado — Spring Boot + PostgreSQL + frontend

## Qué se integró

Se incorporó la nueva interfaz de `sellpal-fixed(1)` dentro de `src/main/resources/static`, incluyendo el index actualizado y el asistente visual `chatbot.js`. La capa `js/api.js` utiliza la API REST de Spring Boot con `fetch` y credenciales de sesión; ya no usa `localStorage` como base de datos. La autenticación del frontend utiliza Spring Security mediante `/api/auth/login`, `/api/auth/registro`, `/api/auth/me` y `/api/auth/logout`.

Los módulos administrativos de cargos, categorías, proveedores, productos, clientes y empleados consumen los endpoints reales. Los reportes consultan `/api/reportes/resumen` y `/api/reportes/ventas-estado`. Los pedidos se envían con el documento real del cliente, estado, productos y detalles en el formato de las entidades JPA. El perfil y el cambio de contraseña también se persisten mediante `/api/auth/perfil` y `/api/auth/password`.

## Requisitos

Se recomienda Java 17, Maven 3.9 o superior y PostgreSQL 14 o superior. El proyecto está configurado para Spring Boot 3.2.2 y escucha en el puerto 8080.

## Preparar PostgreSQL

Cree la base de datos `sellpal_db`, confirme el usuario y contraseña en `src/main/resources/application-postgresql.properties` y, si necesita construir el esquema desde cero, ejecute `database/sellpal_postgresql.sql`. Ese script elimina las tablas existentes; úselo únicamente en desarrollo. Si ya tiene una base creada con la versión anterior, ejecute además `database/migration_pedidos_web.sql` para permitir pedidos de la tienda sin empleado interno asignado al momento de compra.

La configuración actual es:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/sellpal_db
spring.datasource.username=postgres
spring.datasource.password=12345kd
```

Para un entorno real, cambie la contraseña por una variable de entorno y no la mantenga en el repositorio.

## Ejecutar desde IntelliJ IDEA

Abra la carpeta del proyecto que contiene `pom.xml`. Seleccione un JDK 17 en Project Structure, permita que IntelliJ importe Maven y ejecute `SellpalApplication`. También puede usar la terminal:

```bash
mvn spring-boot:run
```

Abra `http://localhost:8080/`.

El administrador inicial es:

```text
Correo: admin@sellpal.co
Contraseña: Admin2026!
Rol: ADMIN
```

Cambie esta contraseña antes de usar el sistema en un entorno real.

## Probar desde Postman

Importe `postman/SELLPAL-FINAL.postman_collection.json`. Ejecute primero `Auth / Login administrador`; Postman conservará la cookie de sesión para las solicitudes siguientes. Luego pruebe productos, categorías, clientes, pedidos, perfil, contraseña y reportes. La creación de pedidos necesita que existan cliente, estado y producto; el empleado puede asignarse posteriormente desde administración.

## Arquitectura

La estructura sigue el flujo `frontend → Controller → Service/Repository → JPA/Hibernate → PostgreSQL`. El backend existente conserva sus entidades y controladores CRUD. La interfaz convierte los nombres amigables usados por sus formularios a los nombres de las entidades JPA antes de enviar los cuerpos JSON.

## Limitaciones conocidas

La entidad `Pedido` actual no expone un `PUT` para cambiar solamente el estado; por eso el cliente no inventa una actualización que el backend no soporte. Las entidades de entrega y pago tienen relaciones obligatorias y deben crearse con referencias válidas a sus entidades padre. El módulo visual de registros conserva una vista informativa, pero el backend no tiene todavía una tabla de auditoría.
