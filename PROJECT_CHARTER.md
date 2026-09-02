# Laboratory Inventory Management System
## Minimum Viable Product — MVP v1.0

### 1. Problema

Los inventarios de laboratorio necesitan controlar los suministros no solamente por producto, sino también por lote.

Diferentes lotes de un mismo producto pueden tener:

- cantidades diferentes;
- fechas de recepción diferentes;
- fechas de vencimiento diferentes;
- y movimientos diferentes.

Un sistema que únicamente almacene la cantidad total de un producto no permite conocer de dónde proviene el inventario ni explicar por qué cambió.

---

### 2. Objetivo del MVP

Construir una aplicación web capaz de controlar productos de laboratorio a nivel de lote y mantener trazabilidad básica de las entradas y salidas de inventario.

El MVP debe demostrar el siguiente ciclo completo:

```text
PRODUCT
   ↓
LOT
   ↓
RECEPTION
   ↓
AVAILABLE INVENTORY
   ↓
ISSUE
   ↓
MOVEMENT HISTORY
```

---

### 3. Usuarios del MVP

El sistema tendrá conceptualmente dos tipos de interacción.

#### Inventory Staff

Puede:

- registrar productos;
- registrar lotes;
- registrar recepciones;
- consultar inventario;
- consultar movimientos;
- registrar ajustes o material dañado.

#### Inventory Consumer

Puede:

- buscar productos disponibles;
- consultar sus lotes;
- solicitar una cantidad;
- confirmar una salida de inventario.

El MVP no requiere todavía un sistema completo de autenticación y permisos.

Sin embargo, las operaciones relevantes deben registrar quién participó en ellas para mantener trazabilidad.

---

### 4. Entidades conceptuales principales

El MVP tendrá como mínimo los siguientes conceptos:

```text
PRODUCT
   |
   | 1:N
   ↓
LOT
   |
   | 1:N
   ↓
MOVEMENT
```

Además existirán operaciones de:

```text
RECEPTION
ISSUE
```

que pueden generar movimientos sobre los lotes correspondientes.

---

### 5. Producto

Un producto representa el tipo de material.

Información mínima:

- identificador;
- nombre;
- categoría;
- fabricante;
- unidad de medida.

Ejemplo:

```text
Nitrile Gloves
Medical Supply
Manufacturer X
Unit
```

Un producto no debe volver a crearse cuando llega un nuevo lote.

---

### 6. Lote

Un lote representa una partida específica de un producto.

Debe contener como mínimo:

- identificador;
- producto;
- número de lote;
- fecha de recepción;
- fecha de vencimiento;
- cantidad inicialmente recibida.

Un producto puede contener múltiples lotes.

---

### 7. Movimientos de inventario

Todo cambio significativo de cantidad deberá quedar registrado.

Tipos mínimos:

```text
RECEIVED
ISSUED
DAMAGED
ADJUSTED
```

Cada movimiento deberá identificar como mínimo:

- lote afectado;
- tipo de movimiento;
- cantidad;
- fecha y hora;
- persona relacionada con la operación;
- nota o razón cuando corresponda.

Ejemplo:

```text
Lot A234

+500 RECEIVED
 -20 ISSUED
  -5 DAMAGED

Available = 475
```

El historial no debe desaparecer cuando cambia el inventario.

---

### 8. Recepción de inventario

El personal de inventario podrá registrar una recepción.

Una recepción debe permitir:

1. seleccionar o crear un producto;
2. registrar el lote;
3. registrar cantidad;
4. registrar vencimiento;
5. identificar quién recibió el material;
6. confirmar la recepción.

La confirmación deberá generar el movimiento correspondiente:

```text
RECEIVED
```

#### Caso mínimo

```text
Product: Nitrile Gloves
Lot: A234
Quantity: 500
Expiration: 2027-03-01

→ Confirm

Movement:
+500 RECEIVED
```

---

### 9. Consulta de inventario

El usuario podrá buscar un producto y observar sus lotes.

Ejemplo:

```text
Nitrile Gloves

Lot A234
Available: 300
Expires: 2026-10-01

Lot B719
Available: 500
Expires: 2027-02-01
```

Como mínimo debe ser posible buscar utilizando:

- nombre del producto;
- número de lote.

---

### 10. Salida de inventario

El usuario podrá:

1. buscar un producto;
2. seleccionar un lote disponible;
3. ingresar una cantidad;
4. identificar quién recibe o solicita el material;
5. confirmar la salida.

La confirmación genera:

```text
ISSUED
```

y reduce el inventario disponible.

---

### 11. Reglas de negocio del MVP

#### BR-01
Un lote debe pertenecer a un producto existente.

#### BR-02
Un producto puede tener múltiples lotes.

#### BR-03
No se puede registrar una cantidad de movimiento igual o menor que cero.

#### BR-04
El inventario disponible nunca puede convertirse en negativo.

#### BR-05
No se puede retirar una cantidad mayor que la disponible.

#### BR-06
Un lote vencido no puede utilizarse para una salida normal.

#### BR-07
El inventario dañado deja de formar parte del inventario disponible.

#### BR-08
Todo cambio significativo de cantidad debe generar un movimiento.

#### BR-09
Los movimientos registrados no deben eliminarse para modificar artificialmente el historial.

#### BR-10
Toda recepción y salida debe identificar la persona relacionada con la operación.

---

### 12. Ajustes y daño

El personal de inventario podrá reducir inventario indicando una razón.

Ejemplo:

```text
Lot A234

Available: 100
Damaged: 10

Movement:
-10 DAMAGED

Available: 90
```

Los ajustes manuales deben requerir una razón para preservar la trazabilidad.

---

### 13. Historial

El sistema debe permitir consultar los movimientos de un lote.

Ejemplo:

```text
LOT A234

2026-08-01   +500   RECEIVED
2026-08-03    -20   ISSUED
2026-08-05     -5   DAMAGED
2026-08-08    -30   ISSUED
```

Con este historial debe ser posible explicar por qué el lote tiene su cantidad disponible actual.

---

### 14. Criterios de aceptación

El MVP estará funcional cuando pueda demostrarse el siguiente escenario completo.

#### Scenario A — Reception

Crear:

```text
Product: Nitrile Gloves
Lot: A234
Quantity: 100
Expiration: 2027-03-01
```

Resultado esperado:

```text
Available inventory = 100

Movement:
+100 RECEIVED
```

#### Scenario B — Issue

Retirar:

```text
20 units
```

Resultado esperado:

```text
Available inventory = 80

Movements:
+100 RECEIVED
 -20 ISSUED
```

#### Scenario C — Invalid issue

Intentar retirar:

```text
100 units
```

cuando solamente existen:

```text
80 units
```

Resultado:

```text
Operation rejected.
Inventory remains 80.
```

#### Scenario D — Damaged inventory

Registrar:

```text
5 damaged units
```

Resultado:

```text
Available inventory = 75

Movements:
+100 RECEIVED
 -20 ISSUED
  -5 DAMAGED
```

#### Scenario E — Traceability

Consultar Lot A234.

El sistema debe poder mostrar:

- producto;
- lote;
- vencimiento;
- cantidad recibida;
- cantidad disponible;
- todos sus movimientos;
- fechas;
- cantidades;
- tipos de movimiento;
- personas relacionadas.

---

### 15. Fuera del MVP

Las siguientes funciones no son necesarias para declarar terminado el MVP:

- autenticación completa;
- sistema complejo de roles;
- códigos QR;
- códigos de barras;
- proveedores;
- compras;
- múltiples laboratorios;
- almacenamiento por ubicaciones;
- emails;
- SMS;
- aplicación móvil;
- analítica avanzada;
- reportes avanzados;
- integración con otros sistemas;
- recall workflow;
- quarantine workflow.

También se dejan para una fase posterior:

- FEFO automático;
- recomendación FEFO;
- low-stock alerts;
- dashboards;
- filtros avanzados;
- operaciones sofisticadas de múltiples productos.

Estas características pertenecen a la evolución de la aplicación y no son necesarias para demostrar inicialmente el concepto principal.

---

### 16. Definición de MVP terminado

El MVP se considera terminado cuando dos tipos de usuario pueden completar de principio a fin el siguiente proceso:

```text
Create Product
      ↓
Create Lot
      ↓
Receive Inventory
      ↓
View Available Inventory
      ↓
Issue Inventory
      ↓
Adjust/Damage Inventory
      ↓
View Updated Stock
      ↓
Inspect Complete Movement History
```

y todas las cantidades pueden explicarse mediante los movimientos registrados.

---

### 17. Principio central

La primera versión debe demostrar una sola idea especialmente bien:

> Inventory is not simply a number. Every meaningful inventory change must be traceable.

Una funcionalidad nueva no deberá incorporarse al MVP si no es necesaria para demostrar ese principio.