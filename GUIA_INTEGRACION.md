# SELLPAL — Guía de integración Interfaz + PostgreSQL + IntelliJ + Postman

## Qué se hizo (paquete D)

1. **SQL**  
   - `database/01_extension_ui_campos.sql` → columnas extra (codigo, marca, imagen, etc.)  
   - `database/02_seed_interfaz.sql` → cargos, categorías, productos Mondelez, empleados y clientes de la UI  

2. **Entidades JPA** actualizadas: `Cargo`, `CategoriaProducto`, `Producto`, `Pedido`, `DetallePedido`  

3. **Interfaz** copiada a `src/main/resources/static/`  
   - `js/api.js` → **fetch real** a `/api/*` con mapeo de nombres  
   - `js/auth.js` → login/registro/logout contra Spring Security  
   - `js/data.js` → modo pasivo (ya no usa localStorage como BD)  

---

## Pasos en tu máquina (orden exacto)

### 1. PostgreSQL

```sql
-- En pgAdmin o psql
CREATE DATABASE sellpal_db;
```

Usuario/contraseña deben coincidir con:

`src/main/resources/application-postgresql.properties`

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/sellpal_db
spring.datasource.username=postgres
spring.datasource.password=12345kd
```

### 2. Schema + extensión + seed

Ejecuta **en este orden** sobre `sellpal_db`:

1. `database/sellpal_postgresql.sql`  
2. `database/01_extension_ui_campos.sql`  
3. `database/02_seed_interfaz.sql`  
4. (opcional) `database/migration_pedidos_web.sql`  

### 3. IntelliJ

1. Abre la carpeta del proyecto (donde está `pom.xml`).  
2. JDK 17.  
3. Ejecuta `SellpalApplication`.  
4. En consola debe aparecer conexión a **PostgreSQL** (no H2).  
5. Al arrancar se crea el admin si no existe:
   - Correo: `admin@sellpal.co`
   - Contraseña: `Admin2026!`

### 4. Interfaz

Abre el navegador:

```
http://localhost:8080/
```

- Login admin → panel `/pages/admin/dashboard.html`  
- Catálogo público → productos desde PostgreSQL  

Crea/edita un producto en el admin y **refresca** en pgAdmin: debe verse el cambio.

### 5. Postman

Importa `postman/SELLPAL-FINAL.postman_collection.json`.

1. **Auth → Login administrador** (misma cookie de sesión).  
2. GET `/api/productos`, POST cargos, etc.  
3. Los mismos datos que ves en la interfaz y en pgAdmin.

---

## Mapeo rápido de nombres

| Interfaz (UI)     | JSON / JPA / BD        |
|-------------------|------------------------|
| cargo.id          | id_cargo               |
| cargo.nombre      | nombre_cargo           |
| categoria.id      | id_categoria           |
| producto.id       | id_producto (campo Java `id`) |
| producto.categoriaId | categoria.id_categoria |
| empleado/cliente documento | documento_unico (PK) |
| pedido.estado (texto) | estado_pedido.nombre_estado |

`api.js` hace este mapeo automáticamente.

---

## Si algo no se guarda

1. Consola IntelliJ: ¿aparece el SQL `insert`/`update` y luego `commit`?  
2. Navegador F12 → Network: ¿la petición a `/api/...` responde 200/201?  
3. ¿Estás logueado? (muchas rutas exigen sesión).  
4. pgAdmin: ¿estás mirando la base `sellpal_db` correcta?

---

## Credenciales

| Rol    | Correo            | Contraseña   |
|--------|-------------------|--------------|
| Admin  | admin@sellpal.co  | Admin2026!   |

Cámbiala antes de cualquier entorno real.
