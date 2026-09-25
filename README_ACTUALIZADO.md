# SELLPAL actualizado – Spring Boot + seguridad + registro + reportes

## Cambios implementados

El proyecto incorpora Spring Security, registro real de usuarios, validación de contraseñas, login con sesión HTTP, logout, usuario administrador inicial y un reporte real de ventas agrupadas por estado.

La contraseña se guarda cifrada con BCrypt. Nunca se guarda en `localStorage`. El navegador conserva únicamente los datos públicos de la sesión para pintar la interfaz.

## Ejecución con PostgreSQL

1. Cree la base de datos configurada en `application-postgresql.properties`.
2. Verifique usuario, contraseña y puerto de PostgreSQL.
3. Ejecute:

```bash
mvn spring-boot:run
```

La aplicación queda disponible en `http://localhost:8080`.

## Ejecución con H2 para pruebas

El perfil H2 evita depender de PostgreSQL:

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=h2
```

La aplicación queda disponible en `http://localhost:8080`.

## Usuario administrador inicial

Al iniciar por primera vez se crea automáticamente:

```text
Correo: admin@sellpal.co
Contraseña: Admin2026!
Rol: ADMIN
```

Cambie la contraseña antes de utilizar el sistema en un entorno real.

## Registro

Ruta:

```text
POST /api/auth/registro
```

Ejemplo:

```json
{
  "nombre": "Tienda La Esperanza",
  "email": "tienda@correo.com",
  "password": "Tienda2026!",
  "passwordConfirm": "Tienda2026!",
  "documento": "901234567",
  "telefono": "3000000000",
  "direccion": "Calle 10 # 20-30",
  "ciudad": "Bogotá"
}
```

La contraseña debe tener mínimo ocho caracteres, una mayúscula, una minúscula, un número y un símbolo. El backend también comprueba que el correo y el documento no estén registrados.

## Inicio de sesión

Ruta:

```text
POST /api/auth/login
```

Ejemplo:

```json
{
  "email": "admin@sellpal.co",
  "password": "Admin2026!"
}
```

El login crea una sesión HTTP. El frontend utiliza `credentials: include` para conservar la sesión.

## Sesión actual y logout

```text
GET  /api/auth/me
POST /api/auth/logout
```

## Reportes

Las rutas están protegidas y necesitan una sesión válida:

```text
GET /api/reportes/resumen
GET /api/reportes/ventas-estado
```

`/api/reportes/resumen` devuelve cantidad de pedidos, suma de ventas y agrupación por estado. `/api/reportes/ventas-estado` devuelve una lista adecuada para tablas y gráficos del dashboard.

El dashboard administrativo incluye una sección de reporte real y lo carga desde `/api/reportes/ventas-estado`.

## Validación del frontend

El formulario de registro verifica coincidencia de contraseñas, fuerza mínima, campos obligatorios y formato de correo. El formulario de login muestra los errores enviados por el backend. Se eliminaron las credenciales precargadas del HTML.

## Pruebas de compilación

```bash
mvn -DskipTests package
```

La compilación verificada genera:

```text
target/sellpal-0.0.1-SNAPSHOT.jar
```

También se verificó el flujo completo con H2: registro `201`, login exitoso, consulta de sesión y reporte protegido `200`.

## Estructura de las nuevas clases

| Archivo | Función |
|---|---|
| `entity/Usuario.java` | Usuario persistente con rol y contraseña cifrada. |
| `repository/UsuarioRepository.java` | Búsqueda por email y documento. |
| `service/UsuarioService.java` | Registro, validación y `UserDetailsService`. |
| `controller/AuthController.java` | Registro, login, sesión y logout. |
| `config/SecurityConfig.java` | Rutas públicas, protegidas y sesiones. |
| `service/ReporteService.java` | Agregación de ventas por estado. |
| `controller/ReporteController.java` | API de reportes. |
| `static/js/auth.js` | Cliente frontend de autenticación real. |

## Nota de integración

El módulo de autenticación y los reportes ya utilizan el backend real. El módulo heredado de CRUD del frontend conserva funciones antiguas basadas en `localStorage` para no romper las pantallas existentes mientras se migran sus campos al modelo relacional completo. La siguiente fase recomendada es migrar progresivamente productos, clientes, pedidos, pagos y entregas a sus endpoints REST reales.
