# Proyecto del Curso — Entrega I

**Curso:** Construcción de Software y Pruebas — Grupo A, 2026-2
**Profesora:** Claudia Martínez — Pontificia Universidad Javeriana Cali
**Equipo:** Joshua Mendez, Edinson Pedroza
**Sistema bajo prueba:** Demoblaze (https://www.demoblaze.com) — tienda en línea de demostración publicada por BlazeMeter
**Entrega:** Entrega I — vence 22-oct-2026

> **Naturaleza del sistema.** Demoblaze es software de terceros. El equipo no tiene el código fuente ni acceso al servidor o a la base de datos; lo prueba únicamente por caja negra desde el navegador. El frontend es una SPA en JavaScript (jQuery/Bootstrap) servida estáticamente, que no calcula nada por su cuenta: delega toda la lógica en un servicio HTTP aparte (`api.demoblaze.com`) que responde JSON, con autenticación por tokens. Por eso las funcionalidades y reglas de abajo se identifican por el comportamiento observable, no por la implementación.

---

## 5. Lista de funcionalidades del sistema

Una funcionalidad **con lógica de negocio** es aquella en la que el sistema aplica un cálculo, una validación compuesta, un algoritmo de selección o una regla de consistencia; no se limita a crear, leer, actualizar o borrar un registro. A continuación se identifican cuatro funcionalidades de Demoblaze que cumplen ese criterio.

| # | Funcionalidad | Actor | Descripción (lógica que la hace no-CRUD) |
|---|---------------|-------|------------------------------------------|
| F1 | **Cálculo del total del carrito** | Cliente (comprador) | El sistema no almacena un total fijo: lo **calcula** sumando el precio de cada ítem presente en el carrito cada vez que se agrega o elimina un producto. El resultado es una operación aritmética derivada del contenido actual del carrito, no un dato que el usuario introduzca o edite. La prueba verifica que `TOTAL = Σ precios de los ítems` para uno y varios productos (regla R3). |
| F2 | **Validación del flujo de compra (Place Order)** | Cliente (comprador, en sesión) | El sitio decide si una compra procede combinando **tres condiciones**: que haya una sesión iniciada, que el carrito no esté vacío y que los campos obligatorios estén completos. Solo con las tres satisfechas se genera el comprobante; si falta cualquiera, rechaza. Es una **validación compuesta** (modelada como tabla de decisión), no la simple escritura de un formulario. Incorpora además la **precarga del total** desde el carrito y las obligatoriedades de R1, R2 y R5. |
| F3 | **Filtrado del catálogo por categoría** | Visitante / Cliente | Sobre el listado de productos, el sistema **selecciona y reconstruye** el conjunto mostrado según la categoría elegida (Phones, Laptops, Monitors), pidiendo al servicio solo los productos de esa clase y repintando la página. Es un **algoritmo de filtrado** con un criterio de pertenencia, no la lectura plana de una lista completa. |
| F4 | **Consistencia de precio entre listado y detalle** | Visitante / Cliente | El sistema expone el precio de un mismo producto en dos vistas (tarjeta del catálogo y ficha de detalle) que se nutren del mismo servicio. La funcionalidad relevante es que esos dos valores **deben coincidir**: es una **regla de consistencia de datos** entre representaciones (regla R4), no un simple despliegue de información. |

### Funcionalidades excluidas por ser CRUD o acceso (no cuentan)

Las siguientes operaciones quedan **explícitamente fuera** de esta lista porque son CRUD puro o control de acceso, sin lógica de negocio:

- **Registro (Sign up):** alta de un usuario con usuario/contraseña. Es un *create* de registro.
- **Autenticación (Log in / Log out):** validación de credenciales y apertura/cierre de sesión. Es control de acceso, no lógica de negocio del dominio.
- **Lectura del listado de productos y de la ficha de detalle** en sí mismas (el simple *read*), separadas de las reglas de filtrado (F3) y de consistencia (F4) que sí aportan lógica.

> Estas operaciones sí se prueban en la suite (módulos Registro y Autenticación), pero como funcionalidades de tipo CRUD/acceso, no como funcionalidades con lógica de negocio.

---

## 6. Reglas de negocio identificadas

Reglas verificables desde afuera (caja negra), redactadas de forma formal. Cada una indica qué implica para las pruebas. El criterio de referencia de R1–R6 son las **convenciones del dominio** de comercio electrónico declaradas en el plan de pruebas del equipo (apartado 3.4): "que no deje comprar con el carrito vacío, que el total dé igual a la suma de lo que muestra, que el precio del listado sea el mismo del detalle y que un error de login no deje adivinar si el usuario existe".

| Regla | Enunciado | Qué implica para las pruebas |
|-------|-----------|------------------------------|
| **R1** | No se puede completar una compra con el **carrito vacío**: el sistema debe rechazar el flujo de Place Order si no hay al menos un producto. | Caso negativo con carrito vacío y campos completos (combinación de la tabla de decisión): se verifica que no se genere comprobante. |
| **R2** | La compra se realiza **en el contexto de una sesión de usuario**: aunque la ventana Place Order puede abrirse desde el carrito, el flujo de compra válido (el que produce comprobante) está asociado a una sesión iniciada; sin sesión, el sistema exige iniciarla. | Casos que varían la condición "sesión Sí/No" manteniendo carrito y campos; se documenta el comportamiento observado respecto a la sesión. |
| **R3** | El **total** mostrado en el carrito y en Place Order es igual a la **suma de los precios** de los productos agregados; el total de Place Order viene **precargado** desde el carrito y no es editable. | Comparar `TOTAL` con la suma calculada manualmente para 1 y N productos; verificar que el total del comprobante coincide con el del carrito. |
| **R4** | El **precio en el listado** del catálogo es igual al **precio en el detalle** del mismo producto. | Revisión de consistencia: para una muestra de productos, cotejar precio de la tarjeta contra precio de la ficha; cualquier diferencia es defecto. |
| **R5** | En Place Order, **Name y Credit card son obligatorios**; si faltan, el sistema rechaza la compra con el mensaje exacto `Please fill out Name and Creditcard.` | Casos con Name y/o Credit card vacíos (y los demás campos completos): verificar el mensaje exacto, que no se genere comprobante y que el carrito no se vacíe. |
| **R6** | El login con credenciales inválidas **no debe permitir la enumeración de usuarios**: el mensaje para un usuario inexistente debe ser idéntico al de una contraseña errada. | Comparar la respuesta de usuario inexistente contra la de contraseña incorrecta; deben ser indistinguibles. *Observación:* los casos de seguridad del plan (CP-SEG-03 y CP-SEG-05) confirmaron que Demoblaze **viola** esta regla, respondiendo `User does not exist.` frente a `Wrong password.`, lo que sí permite enumerar cuentas. |
| **R7** | El **contenido del carrito persiste** al navegar entre páginas dentro de la misma sesión de navegación: los productos agregados no se pierden al cambiar de vista. | Agregar productos, navegar a otras páginas (catálogo, detalle) y regresar al carrito; verificar que los ítems y el total se mantienen. |

### Nota de coherencia con el plan de pruebas

R1, R3, R4 y R6 son exactamente las cuatro convenciones del dominio declaradas en el apartado 3.4 del plan ya ejecutado. R2 y R5 formalizan las condiciones de la **tabla de decisión de la compra** (apartado 3.3), donde el resultado esperado depende de sesión, carrito y campos. R7 recoge la persistencia del carrito declarada en el alcance del módulo Carrito. Las reglas de abajo sirven de criterio de referencia ("resultado esperado") para los casos de la suite de 60 pruebas del equipo.
