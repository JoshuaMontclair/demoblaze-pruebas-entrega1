# 8. Diseño de casos con arreglos ortogonales (Taguchi)

**Sistema bajo prueba:** Demoblaze (https://www.demoblaze.com), tienda en línea de demostración publicada por BlazeMeter.
**Módulo objetivo:** Orden de compra — formulario *Place Order*.
**Técnica:** Arreglos ortogonales (Orthogonal Arrays / Taguchi).
**Curso:** Construcción de Software y Pruebas, Grupo A, 2026-2. Profesora: Claudia Martínez. Pontificia Universidad Javeriana Cali.
**Equipo:** Joshua Mendez, Edinson Pedroza.
**Entregable:** Proyecto del Curso — Entrega I.

---

## 8.1 Por qué esta técnica y qué aporta a la suite

La suite ya ejecutada aplica tres de las cuatro técnicas clásicas de caja negra: partición de equivalencias, análisis de valores límite y tabla de decisión. Falta la cuarta, **arreglos ortogonales**, que es la que permite cubrir de forma sistemática las *combinaciones* entre varios parámetros de entrada. Con este documento la suite alcanza las cuatro técnicas, que es lo que pide la rúbrica.

El arreglo ortogonal resuelve un problema concreto: cuando un formulario tiene varios campos, el número de combinaciones crece de forma exponencial y probarlas todas deja de ser viable. Taguchi demostró que la mayoría de los defectos de interacción se disparan por la combinación de **dos** factores a la vez, no por la de seis simultáneos. Un arreglo ortogonal garantiza que *todas las parejas* de niveles queden cubiertas con una fracción mínima de corridas. Esta propiedad se llama cobertura **pairwise** (de a pares).

El sistema es software de terceros: no tenemos el código fuente ni acceso al servidor o a la base de datos, solo lo probamos por caja negra desde el navegador. Esto no limita la técnica, porque los arreglos ortogonales operan justamente sobre las entradas y salidas observables, que es todo lo que necesitamos.

## 8.2 Función bajo prueba: el formulario *Place Order*

Al pulsar **Place Order** sobre un carrito con productos, Demoblaze abre un formulario con el total precargado y **seis campos editables**:

- **Name** (nombre del titular)
- **Country** (país)
- **City** (ciudad)
- **Credit card** (número de tarjeta)
- **Month** (mes)
- **Year** (año)

Al pulsar **Purchase**, el sistema decide si acepta o rechaza la compra en función de lo que tengan esos seis campos. Ese es exactamente el escenario para el que sirve un arreglo ortogonal: una salida que depende de la combinación de varios campos de entrada.

## 8.3 Factores y niveles

Tomamos los seis campos como **factores**. A cada factor le asignamos **dos niveles**, que es la partición mínima y a la vez la más informativa para una validación de obligatoriedad:

| Factor | Campo del formulario | Nivel 1 (rellenado-válido) | Nivel 2 (vacío) |
|---|---|---|---|
| A | Name | `Joshua Mendez` | *(campo en blanco)* |
| B | Country | `Colombia` | *(campo en blanco)* |
| C | City | `Bogotá` | *(campo en blanco)* |
| D | Credit card | `4111111111111111` | *(campo en blanco)* |
| E | Month | `12` | *(campo en blanco)* |
| F | Year | `2030` | *(campo en blanco)* |

Elegimos dos niveles por factor porque la regla de negocio que se prueba aquí (R5, obligatoriedad de campos) es binaria: un campo está completado o está vacío. El dominio de valores válidos ya se cubre con partición de equivalencias y valores límite en los casos `CP-COM-05` y `CP-COM-06`; aquí no repetimos ese trabajo, sino que estudiamos la **interacción entre los seis campos**.

Conviene posicionar este bloque frente al caso que más se le parece en la suite existente, `CP-COM-04` (rechazo por campos obligatorios vacíos, diseñado con **tabla de decisión**). No hay duplicación: `CP-COM-04` verifica *una* combinación concreta de obligatorios vacíos derivada de la tabla de decisión, mientras que los casos de arreglo ortogonal aportan **cobertura sistemática de la interacción por pares** entre los seis campos —incluyendo combinaciones que la tabla de decisión no enumera, como la corrida 3— con un número acotado de corridas. Las dos técnicas son complementarias: la tabla de decisión documenta las reglas; el arreglo ortogonal explora las interacciones.

### La explosión combinatoria: 2⁶ = 64

Con 6 factores y 2 niveles cada uno, la combinación exhaustiva es:

$$2^6 = 2 \times 2 \times 2 \times 2 \times 2 \times 2 = 64 \text{ corridas}$$

Es decir, para probar literalmente todas las formas de llenar o dejar vacío cada campo haría falta ejecutar **64 casos**. Para un solo formulario de una tienda de demostración, probado a mano sobre un entorno de producción ajeno y compartido (ver restricciones del plan de pruebas), 64 corridas son inviables y, sobre todo, innecesarias: la enorme mayoría no aporta información nueva porque repite parejas de condiciones ya vistas.

## 8.4 Arreglo ortogonal seleccionado: L8 (2⁷)

Para 6 factores de 2 niveles, el arreglo de Taguchi más pequeño que los acomoda es el **L8 (2⁷)**: 8 corridas y hasta 7 columnas de 2 niveles. Usamos 6 de sus 7 columnas (una por factor) y dejamos la séptima sin asignar. El L8 reduce las 64 corridas exhaustivas a solo **8**.

### 8.4.1 Arreglo L8 en forma canónica (niveles 1/2)

Esta es la matriz estándar del L8. Cada fila es una corrida; cada columna, un factor; el valor es el nivel que toma ese factor en esa corrida:

| Corrida | A (Name) | B (Country) | C (City) | D (Credit card) | E (Month) | F (Year) | Col. 7 (sin uso) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 2 | 1 | 1 | 1 | 2 | 2 | 2 | 2 |
| 3 | 1 | 2 | 2 | 1 | 1 | 2 | 2 |
| 4 | 1 | 2 | 2 | 2 | 2 | 1 | 1 |
| 5 | 2 | 1 | 2 | 1 | 2 | 1 | 2 |
| 6 | 2 | 1 | 2 | 2 | 1 | 2 | 1 |
| 7 | 2 | 2 | 1 | 1 | 2 | 2 | 1 |
| 8 | 2 | 2 | 1 | 2 | 1 | 1 | 2 |

### 8.4.2 Arreglo L8 traducido a los campos (nivel = valor concreto)

Nivel 1 = rellenado-válido · Nivel 2 = vacío.

| Corrida | Name | Country | City | Credit card | Month | Year |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | lleno | lleno | lleno | lleno | lleno | lleno |
| 2 | lleno | lleno | lleno | **vacío** | **vacío** | **vacío** |
| 3 | lleno | **vacío** | **vacío** | lleno | lleno | **vacío** |
| 4 | lleno | **vacío** | **vacío** | **vacío** | **vacío** | lleno |
| 5 | **vacío** | lleno | **vacío** | lleno | **vacío** | lleno |
| 6 | **vacío** | lleno | **vacío** | **vacío** | lleno | **vacío** |
| 7 | **vacío** | **vacío** | lleno | lleno | **vacío** | **vacío** |
| 8 | **vacío** | **vacío** | lleno | **vacío** | lleno | lleno |

## 8.5 Justificación de la reducción 64 → 8 y qué garantiza pairwise

La pregunta obligada es: ¿qué perdemos al pasar de 64 a 8 corridas? La respuesta es **nada de la cobertura de a pares**, que es donde se concentran los defectos de interacción.

Un arreglo es *ortogonal* cuando, para **cualquier par de columnas** que se elija, las cuatro combinaciones posibles de sus niveles — (1,1), (1,2), (2,1), (2,2) — aparecen **el mismo número de veces**. En el L8 cada una de esas cuatro combinaciones aparece exactamente **dos veces** en cada par de columnas. Con 6 factores existen C(6,2) = **15 parejas** de campos, y el L8 cubre las cuatro combinaciones de las 15 parejas en solo 8 corridas.

Traducido al formulario: para **cualquier** par de campos (por ejemplo Name–Credit card, o Country–Year) las 8 corridas contienen los cuatro escenarios "ambos llenos", "el primero lleno y el segundo vacío", "el primero vacío y el segundo lleno" y "ambos vacíos". Si existe un defecto que solo se dispara cuando dos campos concretos están en cierta combinación, el L8 lo expone.

Verificación de la pareja crítica **Name (A) – Credit card (D)**, que son los dos campos obligatorios según R5:

| Combinación (Name, Card) | Corridas donde aparece | Veces |
|---|---|:---:|
| (lleno, lleno) | 1, 3 | 2 |
| (lleno, vacío) | 2, 4 | 2 |
| (vacío, lleno) | 5, 7 | 2 |
| (vacío, vacío) | 6, 8 | 2 |

Las cuatro combinaciones quedan cubiertas dos veces cada una. Lo que el L8 **no** garantiza es la cobertura de interacciones de orden superior (tres o más campos simultáneos en una combinación específica). Esa limitación es aceptable y deliberada: la evidencia empírica de Taguchi y de la literatura de *pairwise testing* indica que los fallos por combinación de tres o más factores son marginales frente a los de a pares, y cubrirlos exigiría volver hacia las 64 corridas. Cambiamos un riesgo residual pequeño por una reducción del 87,5 % en el número de casos (de 64 a 8).

**¿Por qué 8 corridas y no 6?** Si el único objetivo fuera la cobertura *pairwise* mínima, un *covering array* lograría los 6 factores binarios en apenas 6 corridas. Usamos 8 porque exigimos la propiedad más fuerte de **arreglo ortogonal**: no solo que cada par de niveles aparezca *al menos* una vez (covering array), sino que aparezca el **mismo número de veces** (cobertura pairwise *uniforme*, que obliga a que el número de corridas sea múltiplo de 4). Esa uniformidad es la diferencia entre un simple conjunto pairwise y un arreglo ortogonal, y aporta además balance estadístico entre los factores —el fundamento del diseño de experimentos de Taguchi—. Las 2 corridas extra respecto al covering array son el precio de esa garantía de equilibrio.

## 8.6 Derivación del resultado esperado (R1, R3, R5)

El resultado esperado de cada corrida no se inventa: se deduce de las reglas de negocio observadas en el sistema.

- **R1 — No se puede completar la compra con el carrito vacío.** Todas las corridas comparten la precondición de carrito con al menos un producto, de modo que R1 queda satisfecha y no es la variable bajo estudio. Así aislamos el efecto de los campos del formulario. Por la misma razón, la **sesión iniciada** se fija como precondición constante en las ocho corridas y **no entra como factor** del arreglo: el objetivo es estudiar la interacción entre los seis campos, no el efecto de la autenticación. Dejar la sesión constante es coherente con el caso `CP-COM-03` de la suite (que prueba por separado la exigencia de sesión); aquí simplemente se da por cumplida.
- **R5 — Name y Credit card son obligatorios.** Si Name o Credit card quedan vacíos, el sistema rechaza la compra con el mensaje exacto **"Please fill out Name and Creditcard."** y no genera comprobante. Esta es la regla que decide el resultado de cada corrida.
- **R3 — El total mostrado = suma de los precios de los productos.** Cuando la compra se completa, el monto del comprobante debe ser igual al total del carrito, que a su vez es la suma de los ítems agregados.

La regla operativa que se desprende de R5 es: **la compra tiene éxito si, y solo si, Name y Credit card están ambos rellenados** (con el carrito no vacío por R1). Los campos Country, City, Month y Year no participan en la validación de obligatoriedad del sistema; que estén llenos o vacíos no bloquea la compra.

Aplicando la regla a las 8 corridas:

| Corrida | Name | Credit card | ¿Ambos obligatorios llenos? | Resultado esperado |
|:---:|:---:|:---:|:---:|---|
| 1 | lleno | lleno | Sí | Compra exitosa + comprobante (R3) |
| 2 | lleno | vacío | No (falta tarjeta) | Rechazo: "Please fill out Name and Creditcard." |
| 3 | lleno | lleno | Sí | Compra exitosa + comprobante (R3) |
| 4 | lleno | vacío | No (falta tarjeta) | Rechazo: "Please fill out Name and Creditcard." |
| 5 | vacío | lleno | No (falta nombre) | Rechazo: "Please fill out Name and Creditcard." |
| 6 | vacío | vacío | No (faltan ambos) | Rechazo: "Please fill out Name and Creditcard." |
| 7 | vacío | lleno | No (falta nombre) | Rechazo: "Please fill out Name and Creditcard." |
| 8 | vacío | vacío | No (faltan ambos) | Rechazo: "Please fill out Name and Creditcard." |

Resultado: **2 compras exitosas** (corridas 1 y 3) y **6 rechazos** (corridas 2, 4, 5, 6, 7 y 8).

La corrida **3** es la más interesante desde el punto de vista del diseño: Name y Credit card están llenos, pero Country, City y Year quedan vacíos, y aun así la compra se completa. Esto evidencia que, de los seis campos, el sistema solo trata como obligatorios dos. Es justo el tipo de hallazgo que el arreglo ortogonal expone de forma económica y que una sola prueba del "camino feliz" (todos los campos llenos) jamás mostraría.

## 8.7 Casos de prueba formales (IEEE 829)

Las 8 corridas se documentan como casos de prueba con las mismas columnas de la suite existente: **ID | Módulo | Prioridad | Técnica de diseño | Precondiciones | Pasos | Datos de prueba | Resultado esperado | Criterio de referencia**. Los identificadores `CP-AO-01` … `CP-AO-08` no colisionan con ningún ID de la suite. En las celdas de *Pasos* y *Datos de prueba* se usa `<br>` como salto de línea para mantener la tabla legible.

**Criterio de priorización.** Las dos corridas de compra exitosa (`CP-AO-01` y `CP-AO-03`) se marcan **Alta** porque ejercitan el camino crítico del negocio —completar una compra—, que es la prioridad máxima según el apartado 3.6 del plan. Las seis corridas de rechazo por R5 se marcan **Media**, en coherencia con los casos negativos equivalentes de la suite (`CP-COM-05` y `CP-COM-06`, también Media): verifican el manejo correcto de entradas inválidas, importante pero de menor impacto que el flujo de compra en sí.

| ID | Módulo | Prioridad | Técnica de diseño | Precondiciones | Pasos | Datos de prueba | Resultado esperado | Criterio de referencia |
|---|---|---|---|---|---|---|---|---|
| CP-AO-01 | Orden de compra | Alta | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar los seis campos con datos válidos.<br>2. Pulsar Purchase. | Name: Joshua Mendez<br>Country: Colombia<br>City: Bogotá<br>Credit card: 4111111111111111<br>Month: 12<br>Year: 2030 | La compra se confirma y se muestra un comprobante con identificador, monto, titular y fecha. El monto es igual al total del carrito (R3). | Convención del dominio / R5: con Name y Credit card completados la compra procede. Coherencia con R3 en el comprobante. |
| CP-AO-02 | Orden de compra | Media | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar Name, Country y City.<br>2. Dejar vacíos Credit card, Month y Year.<br>3. Pulsar Purchase. | Name: Joshua Mendez<br>Country: Colombia<br>City: Bogotá<br>Credit card: (vacío)<br>Month: (vacío)<br>Year: (vacío) | El sistema rechaza la compra con el mensaje "Please fill out Name and Creditcard." No genera comprobante ni vacía el carrito. | Convención del dominio / R5: Credit card es obligatorio. |
| CP-AO-03 | Orden de compra | Alta | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar Name, Credit card y Month.<br>2. Dejar vacíos Country, City y Year.<br>3. Pulsar Purchase. | Name: Joshua Mendez<br>Country: (vacío)<br>City: (vacío)<br>Credit card: 4111111111111111<br>Month: 12<br>Year: (vacío) | La compra se confirma y se muestra el comprobante (monto = total del carrito, R3), aunque Country, City y Year estén vacíos. Evidencia que esos campos no son obligatorios. | Convención del dominio / R5: solo Name y Credit card son obligatorios. Coherencia con R3 en el comprobante. |
| CP-AO-04 | Orden de compra | Media | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar Name y Year.<br>2. Dejar vacíos Country, City, Credit card y Month.<br>3. Pulsar Purchase. | Name: Joshua Mendez<br>Country: (vacío)<br>City: (vacío)<br>Credit card: (vacío)<br>Month: (vacío)<br>Year: 2030 | El sistema rechaza la compra con el mensaje "Please fill out Name and Creditcard." No genera comprobante ni vacía el carrito. | Convención del dominio / R5: Credit card es obligatorio. |
| CP-AO-05 | Orden de compra | Media | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar Country, Credit card y Year.<br>2. Dejar vacíos Name, City y Month.<br>3. Pulsar Purchase. | Name: (vacío)<br>Country: Colombia<br>City: (vacío)<br>Credit card: 4111111111111111<br>Month: (vacío)<br>Year: 2030 | El sistema rechaza la compra con el mensaje "Please fill out Name and Creditcard." No genera comprobante ni vacía el carrito. | Convención del dominio / R5: Name es obligatorio. |
| CP-AO-06 | Orden de compra | Media | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar Country y Month.<br>2. Dejar vacíos Name, City, Credit card y Year.<br>3. Pulsar Purchase. | Name: (vacío)<br>Country: Colombia<br>City: (vacío)<br>Credit card: (vacío)<br>Month: 12<br>Year: (vacío) | El sistema rechaza la compra con el mensaje "Please fill out Name and Creditcard." No genera comprobante ni vacía el carrito. | Convención del dominio / R5: Name y Credit card son obligatorios. |
| CP-AO-07 | Orden de compra | Media | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar City y Credit card.<br>2. Dejar vacíos Name, Country, Month y Year.<br>3. Pulsar Purchase. | Name: (vacío)<br>Country: (vacío)<br>City: Bogotá<br>Credit card: 4111111111111111<br>Month: (vacío)<br>Year: (vacío) | El sistema rechaza la compra con el mensaje "Please fill out Name and Creditcard." No genera comprobante ni vacía el carrito. | Convención del dominio / R5: Name es obligatorio. |
| CP-AO-08 | Orden de compra | Media | Arreglos ortogonales | Sesión iniciada y carrito con al menos un producto; ventana Place Order abierta. | 1. Rellenar City, Month y Year.<br>2. Dejar vacíos Name, Country y Credit card.<br>3. Pulsar Purchase. | Name: (vacío)<br>Country: (vacío)<br>City: Bogotá<br>Credit card: (vacío)<br>Month: 12<br>Year: 2030 | El sistema rechaza la compra con el mensaje "Please fill out Name and Creditcard." No genera comprobante ni vacía el carrito. | Convención del dominio / R5: Name y Credit card son obligatorios. |

## 8.8 Resumen

- **Técnica:** arreglos ortogonales de Taguchi, la cuarta técnica de caja negra que completaba la suite (junto a partición de equivalencias, valores límite y tabla de decisión).
- **Factores:** los 6 campos de *Place Order* (Name, Country, City, Credit card, Month, Year), con 2 niveles cada uno (rellenado-válido / vacío).
- **Combinación exhaustiva:** 2⁶ = 64 corridas; **arreglo elegido:** L8 (2⁷), que cubre todas las parejas de niveles (pairwise) en **8 corridas**, una reducción del 87,5 %.
- **Resultados esperados:** derivados de R1, R3 y R5; 2 compras exitosas (CP-AO-01, CP-AO-03) y 6 rechazos con el mensaje "Please fill out Name and Creditcard.".
- **Hallazgo de diseño:** la corrida 3 muestra que de los 6 campos solo Name y Credit card son obligatorios.
- **Casos nuevos:** CP-AO-01 … CP-AO-08, módulo "Orden de compra", técnica "Arreglos ortogonales", sin colisión con la suite existente.
