# Laboratory Inventory Management System
## Casos de uso — v2.0

**Versión documental:** 2.0  
**Estado:** especificación corregida para elaborar diagramas de flujo.  
**Base:** casos v1.0, charter, definición del MVP e informe de revisión del 2026-09-08.  
**Entrega funcional:** MVP inicial; el número 2.0 corresponde al documento, no a una aplicación ya construida o probada.

## 1. Propósito y relación entre documentos

Esta versión incorpora las correcciones de la revisión y define los ocho casos con sus entradas, decisiones, excepciones y resultados. Sustituye funcionalmente el borrador de casos v1.0. Las políticas de unidades enteras, vencimiento inclusivo y registro de material vencido se adoptan explícitamente en esta versión.

El charter v2.0 mantiene la visión y las fases; el MVP v2.0 define el alcance y las reglas; este documento desarrolla sus recorridos. Las versiones v1.0 son antecedentes y no deben combinarse con estas reglas para implementar la entrega actual. Una modificación posterior debe mantener alineadas las reglas del MVP y de estos casos.

## 2. Alcance del MVP

El MVP inicial permite registrar productos y lotes, recibir y entregar inventario, registrar daño y ajustes, buscar existencias y consultar el historial por lote.

Cada confirmación de inventario trabaja con un lote y una unidad base del producto. Una entrega física puede originar varias confirmaciones independientes. No hay carrito ni confirmación conjunta de varios artículos; tampoco se promete que dos operaciones distintas se completen juntas.

La primera versión trabaja con suministros contables en unidades enteras: por ejemplo, guantes individuales, pares, frascos, cajas cerradas o paquetes. La unidad elegida se muestra en todas las operaciones. Un producto gestionado por caja no admite retirar piezas de esa caja. Se requiere vencimiento completo en todos los lotes.

Quedan para fases posteriores FEFO automático o recomendado, alertas de stock bajo o próximo vencimiento, filtros avanzados, dashboards, operaciones de varios artículos, fracciones, conversiones, gestión de artículos sin vencimiento, edición posterior de catálogo, correcciones conjuntas de varios lotes, fechas físicas retroactivas y gestión de rechazos a proveedores.

También quedan fuera autenticación y autorización completas, administración de empleados, solicitudes con aprobación, reservas, devoluciones con su propio flujo, proveedores y compras, facturación y pagos, códigos de barras/QR, ubicaciones, múltiples laboratorios, reportes avanzados, integraciones externas, notificaciones por email/SMS, aplicación móvil, cuarentena y recall. La consulta y el bloqueo de lotes vencidos sí forman parte del MVP.

## 3. Conceptos y cantidades

| Concepto | Significado |
| --- | --- |
| Producto | Tipo de material, con código único, nombre que distingue variantes, categoría, fabricante y una unidad base. |
| Lote | Partida de un producto identificada por producto y número de lote; tiene un vencimiento propio. |
| Saldo del lote | Unidades registradas todavía incluidas en el inventario controlado, después de recepciones, salidas y bajas. |
| Disponible para salida | Saldo si el lote está vigente; cero si está vencido. No es un segundo saldo editable. |
| Primera recepción registrada | Fecha/hora y magnitud del primer RECEIVED confirmado. Antes de él: sin recepción y cantidad cero. |
| Total registrado como recibido | Suma de magnitudes de RECEIVED; conserva lo que se registró originalmente, con correcciones visibles por separado. |
| Operación | Un intento confirmado de recepción, salida, daño o ajuste sobre un lote. Las altas de catálogo son operaciones independientes sin movimiento. |
| Movimiento | Registro inmutable que explica un cambio del saldo. |
| Razón | Explicación obligatoria para daño y ajuste. Una recepción vencida exige nota justificativa. |
| Referencia de corrección | Identificador del movimiento del mismo lote cuya cantidad se corrige; no modifica ni revierte automáticamente el registro referido. |

| Tipo de movimiento | Magnitud almacenada | Dirección | Efecto en saldo |
| --- | --- | --- | --- |
| RECEIVED | Entero positivo q | Determinada por el tipo | +q |
| ISSUED | Entero positivo q | Determinada por el tipo | -q |
| DAMAGED | Entero positivo q | Determinada por el tipo | -q |
| ADJUSTED | Entero positivo q | INCREASE, elegida por el usuario | +q |
| ADJUSTED | Entero positivo q | DECREASE, elegida por el usuario | -q |

`saldo = suma de los efectos de movimientos confirmados del lote`

`disponible_para_salida = saldo si fecha_vencimiento >= fecha_actual_del_laboratorio; 0 en caso contrario`

El total disponible de un producto suma el disponible de sus lotes; no se combinan cantidades de productos ni unidades diferentes. Una consulta no reserva stock.

El vencimiento no demuestra una baja física: puede haber saldo 20 y disponible 0. La baja efectiva se registra mediante un ajuste de disminución con razón «baja por vencimiento». El material ya dado de baja por daño o ajuste queda excluido del conteo del saldo aunque siga físicamente en una zona de descarte.

En una recepción, la cantidad corresponde a las unidades que se incorporan al inventario controlado. Unidades rechazadas por daño antes de incorporarlas no entran en esa cantidad ni reciben una segunda baja DAMAGED; el rechazo puede describirse en una nota. DAMAGED se utiliza cuando las unidades afectadas ya forman parte del saldo.

## 4. Actores y personas registradas

| Actor | Responsabilidades |
| --- | --- |
| Inventory Staff | Registrar productos, lotes, recepciones, daños y ajustes; consultar inventario e historial. |
| Inventory Consumer | Buscar inventario y registrar entregas para uso. |

Estos actores describen formas de uso. No constituyen dos roles con restricciones de acceso implementadas. Para movimientos se registran nombres declarados: quien captura y, en RECEIVED/ISSUED, el participante correspondiente. No se promete identidad verificada ni un módulo de empleados.

## 5. Reglas de negocio y comportamiento común

| Regla | Definición vigente |
| --- | --- |
| BR-01 | Un lote pertenece exactamente a un producto existente. Toda operación verifica que producto y lote seleccionados se correspondan. |
| BR-02 | Un producto puede tener muchos lotes. La combinación producto y número de lote normalizado es única. El mismo número puede existir en productos diferentes. |
| BR-03 | La magnitud de todo movimiento es un entero positivo expresado en la unidad base contable del producto. Se rechazan valores ausentes, no numéricos, no finitos, fraccionarios, cero o negativos; no se redondean. El tipo y, para ADJUSTED, la dirección determinan el efecto. |
| BR-04 | El saldo del lote inicia en cero y equivale a la suma de los efectos de sus movimientos confirmados. Nunca puede ser negativo. El disponible para salida equivale al saldo si el lote está vigente y a cero si está vencido. |
| BR-05 | Una salida solo se confirma si su magnitud no supera el disponible vigente al confirmar. No se realizan salidas parciales ni cambios automáticos de lote. |
| BR-06 | Un lote está vencido cuando su fecha de vencimiento es anterior al día actual del laboratorio. La fecha de vencimiento se incluye como último día vigente. El reloj del sistema y una zona horaria configurada determinan ese día dentro de la validación final protegida. Todo ISSUED de un lote vencido se rechaza. |
| BR-07 | DAMAGED reduce el saldo por unidades dañadas todavía incluidas en él y exige razón. Su cantidad no supera el saldo, incluso si el lote está vencido. No se vuelven a descontar unidades ya dadas de baja. |
| BR-08 | Cada confirmación de inventario afecta exactamente un lote y genera exactamente un movimiento. La validación final y la persistencia del movimiento y de su efecto son indivisibles. El paso del tiempo puede cambiar la disponibilidad sin modificar el saldo ni generar un movimiento. |
| BR-09 | Los movimientos confirmados no se editan ni se eliminan, incluidos tipo, cantidad, dirección, personas, fecha y razón. Una corrección de cantidad agrega un ADJUSTED con razón; si se conoce el movimiento origen, se exige su identificador y se verifica que pertenezca al mismo lote. Una referencia no ejecuta una reversión automática. |
| BR-10 | Todo movimiento registra el nombre declarado de quien lo captura. RECEIVED registra además a quien recibió físicamente el material; ISSUED, a quien se entregó para uso. Los nombres son obligatorios y no vacíos; pueden coincidir. Esta identificación no constituye autenticación. |
| BR-11 | Cada producto tiene un código interno único y estable, distinto de su identificador técnico. Códigos y números de lote se comparan quitando espacios externos y unificando mayúsculas, sin eliminar ceros, signos ni espacios internos. Se tratan como texto. El nombre identifica las variantes del material. La unicidad del código no garantiza detectar materiales equivalentes registrados con códigos diferentes. |
| BR-12 | Crear un producto o lote no genera movimiento ni agrega unidades. Un lote recién creado tiene saldo cero. Su primera fecha y cantidad recibida se derivan del primer RECEIVED confirmado; antes de él se muestra sin recepción y cantidad inicial cero. El total registrado como recibido suma todos los RECEIVED. |
| BR-13 | Las entregas posteriores del mismo producto y lote generan nuevos RECEIVED sobre el lote existente. No reinician cantidades ni modifican su vencimiento. Una discrepancia entre el vencimiento físico informado y el registrado detiene la operación para revisar la identificación y los datos. |
| BR-14 | Todos los lotes del MVP requieren una fecha calendario completa y válida. Los productos sin vencimiento y las fechas incompletas quedan fuera de esta entrega. No se inventa una fecha para aceptarlos. |
| BR-15 | Se puede registrar material ya vencido que existe físicamente: exige advertencia, nota y aceptación explícita de su condición antes de confirmar RECEIVED. Aumenta el saldo y mantiene disponible cero. Si el lote vence entre revisión y confirmación, debe solicitarse esa aceptación antes de un nuevo intento. |
| BR-16 | ADJUSTED exige dirección INCREASE o DECREASE, magnitud positiva y razón. La entrada es una diferencia, no el saldo final. Requiere al menos un RECEIVED previo y no sustituye entregas nuevas. Un lote con recepción histórica y saldo cero admite aumentos justificados. Una baja por vencimiento usa DECREASE con esa razón. |
| BR-17 | Cada producto usa una unidad base contable e indivisible para sus movimientos; no hay conversiones ni fracciones. El catálogo confirmado no ofrece edición ni eliminación en este MVP. Las correcciones de metadatos posteriores al alta requieren una extensión específica y no se simulan con ajustes o duplicados. |
| BR-18 | Preparar una recepción, salida, daño o ajuste no reserva ni cambia stock. Confirmar ISSUED representa una entrega realizada; no crea una solicitud pendiente ni un proceso de aprobación. |
| BR-19 | Las operaciones sobre el mismo lote se coordinan para validar el saldo vigente y registrar el efecto sin interferencias. Dos salidas de 8 sobre saldo 10 no pueden confirmar ambas; dos recepciones distintas conservan ambos incrementos. |
| BR-20 | Un ajuste conserva una referencia del estado del lote revisado. Si ocurrió cualquier movimiento posterior, aunque el saldo vuelva al mismo valor, se rechaza ese intento y se exige revisar el conteo, recalcular el efecto y volver a confirmar. |
| BR-21 | Cada intento enviado de confirmación de inventario tiene un identificador interno de operación. El mismo identificador y contenido se procesa una sola vez; sus reenvíos recuperan el resultado original antes de revalidar saldo o vencimiento. Un contenido distinto con un identificador ya utilizado se rechaza. Un intento todavía en curso no se ejecuta nuevamente. |
| BR-22 | El sistema asigna identificador y fecha/hora a cada movimiento, preserva un orden estable de confirmación por lote y consulta saldo e historial de manera coherente. Primera recepción registrada significa primer registro confirmado, no una fecha física retroactiva. Las cantidades históricas recibidas no se reescriben por ajustes posteriores. |
| BR-23 | Cancelar antes de enviar la confirmación no produce movimiento. Cerrar una pantalla después del envío no cancela una operación en curso ni revierte una confirmada. Las altas de catálogo confirmadas desde una recepción son independientes y permanecen aunque luego se cancele la recepción. |
| BR-24 | Un rechazo conocido o fallo con reversión confirmada conserva cero efectos de ese intento. Una respuesta perdida deja el resultado desconocido hasta consultarlo o reenviarlo con el mismo identificador y datos. Un fallo de consulta no se presenta como inventario cero, producto ausente ni historial vacío. |

### CF-01 — Captura, revisión y rechazo conocido

Validar campos obligatorios, unidad, magnitud y referencias antes de mostrar el resumen; repetir las comprobaciones determinantes al confirmar. Los nombres y razones compuestos solo por espacios son vacíos. El sistema comunica el campo o condición que impide continuar y conserva el formulario para corregirlo o cancelarlo.

Tras un rechazo conocido, una confirmación corregida constituye un nuevo intento con un nuevo identificador. «Sin cambios» significa sin efectos de ese intento; no deshace altas de catálogo ni operaciones independientes de otros usuarios.

### CF-02 — Cancelación y cierre

Antes de enviar, cancelar termina el formulario sin movimiento. Una vez enviado, cerrar o pulsar cancelar no garantiza detenerlo: debe resolverse su resultado con CF-03. Una operación confirmada conserva sus efectos. Una corrección posterior de cantidad utiliza UC-07.

UC-01 y UC-02 pueden confirmar altas independientes dentro de UC-03. Cancelar la recepción posterior conserva esos registros, incluso un lote sin recepciones y con saldo cero.

### CF-03 — Confirmación única y recuperación de resultado

Este protocolo se aplica a UC-03, UC-05, UC-06 y UC-07. Los identificadores y controles internos los administra el sistema; no se exige al usuario inventarlos.

1. Al enviar una confirmación, asociar un identificador interno a sus datos y conservarlo para recuperar el resultado.
2. Antes de ejecutar nuevamente validaciones de negocio, resolver si ese identificador ya existe. Si existe con datos diferentes, rechazar el conflicto. Si está confirmado o rechazado con los mismos datos, devolver su resultado original sin crear otro movimiento. Si sigue en curso, esperar o consultar el mismo intento sin ejecutarlo otra vez.
3. Para un intento nuevo, proteger la operación sobre el lote, obtener sus datos vigentes y aplicar las validaciones específicas del caso. En un ajuste, comparar también la referencia de estado revisada. Tomar el reloj del sistema dentro de esta validación final.
4. Si alguna validación falla, finalizar como rechazo conocido sin movimiento ni efecto en saldo. Indicar el punto de retorno del caso.
5. Si pasa, registrar un movimiento completo y su efecto en saldo de manera indivisible, junto con un resultado recuperable para ese identificador. El identificador de operación y la exclusión de duplicados deben proteger también envíos simultáneos.
6. Confirmar éxito solamente cuando el registro completo está confirmado. Un fallo con reversión confirmada no conserva efectos de ese intento. Una pérdida de respuesta mantiene el resultado desconocido hasta consultar o reenviar el identificador original con sus datos originales.

Una confirmación recuperada muestra el movimiento y el resultado históricos. Si se muestra además el saldo actual, debe identificarse como una consulta actual distinta: otros movimientos o el paso del tiempo pueden haber cambiado ese saldo o disponibilidad. La recuperación no vuelve a bloquear una salida antigua por falta de stock o vencimiento actuales.

Los estados «en curso» o «resultado desconocido» pertenecen a la confirmación técnica; no agregan solicitudes de inventario pendientes ni reservas de negocio. No se debe deducir que un timeout equivale a rechazo.

### CF-04 — Altas de catálogo y consultas

UC-01 y UC-02 comprueban unicidad también al guardar. Si otra alta simultánea creó el mismo código o lote, se recupera el registro existente y se comparan sus datos; nunca se declara creado el contenido del formulario si el registro existente es diferente. Tras una respuesta perdida se busca por esa clave y se verifica la coincidencia antes de dar el alta por resuelta.

UC-04 y UC-08 distinguen resultados vacíos de un fallo de lectura. Consultan registros agotados o vencidos y no realizan ninguna escritura. Un fallo confirmado o una cancelación de alta no conserva un registro parcial creado por ese intento.

## 6. Casos de uso

### UC-01 — Registrar producto

**Actor principal:** Inventory Staff  
**Objetivo:** Identificar un tipo de material sin repetir su código y dejarlo disponible para asociar lotes.  
**Disparador:** Se necesita registrar un producto que no está identificado en el catálogo.

**Precondiciones:** Disponer de los datos que identifican el producto.

**Datos de entrada:** Código interno; nombre con variante relevante; categoría; fabricante; unidad base contable. Todos son obligatorios. El identificador técnico lo genera el sistema.

**Flujo principal**

1. Abrir el alta de producto.
2. Buscar posibles coincidencias en el catálogo por nombre y fabricante y revisar su variante y unidad.
3. Si no corresponde usar un registro existente, ingresar código, nombre, categoría, fabricante y unidad.
4. Validar campos, normalizar el código y mostrar el resumen para revisión.
5. Confirmar el alta.
6. Comprobar de forma protegida que el código sea único y crear el producto.
7. Mostrar el producto creado y devolver su identificador a la operación que solicitó el alta, si existe.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Pasos 3–4: campos vacíos o unidad incompatible con el alcance | Mostrar errores y volver al paso 3. |
| A2 | Paso 2 o 6: producto o código existente | Mostrar el registro existente. Seleccionarlo y terminar sin creación, o volver al paso 3 para corregir una identificación equivocada. No generar otro código solo para eludir un duplicado. |
| A3 | Paso 6: otra alta creó el mismo código | Aplicar CF-04; recuperar y comparar el registro existente. Si no coincide, informar conflicto y volver al paso 2. |
| A4 | Fallo de guardado, respuesta perdida o cancelación | Aplicar CF-02/CF-04. Con fallo confirmado no se crea un producto parcial; con resultado desconocido comprobar el código antes de resolver el alta. |

**Postcondiciones de éxito:** Existe un nuevo producto completo, o se devuelve explícitamente un producto existente seleccionado. No se genera movimiento ni saldo.

**Postcondiciones de fallo:** Sin nuevo producto parcial por el intento fallido. Un registro existente nunca se sobrescribe.

**Reglas relacionadas:** BR-11, BR-12, BR-17, BR-23, BR-24.

---

### UC-02 — Registrar lote

**Actor principal:** Inventory Staff  
**Objetivo:** Identificar una partida de un producto sin crear existencias ni duplicar el lote.  
**Disparador:** Se necesita un lote nuevo, directamente o durante una recepción.

**Precondiciones:** Poder seleccionar un producto; si todavía no existe, completar UC-01 antes de guardar el lote.

**Datos de entrada:** Producto existente; número de lote como texto; fecha completa de vencimiento. No se capturan cantidades ni una fecha independiente de recepción.

**Flujo principal**

1. Seleccionar el producto existente.
2. Ingresar número de lote y fecha de vencimiento.
3. Validar datos, normalizar el número de lote y comprobar la combinación producto–lote.
4. Mostrar el resumen, incluido el estado de vencimiento calculado, y confirmar el alta.
5. Volver a validar el producto y la unicidad al guardar; crear el lote con saldo cero y sin movimientos.
6. Mostrar el lote y devolverlo al proceso que solicitó el alta, si existe.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Paso 1 o 5: producto inexistente o inválido | Volver al paso 1; seleccionar otro producto o completar UC-01. |
| A2 | Paso 2–3: número vacío o fecha ausente, incompleta o inválida | Volver al paso 2; no crear el lote. |
| A3 | Paso 3 o 5: mismo producto, lote y vencimiento | Mostrar y devolver el lote existente; terminar sin alta ni reinicio de saldo. |
| A4 | Paso 3 o 5: mismo producto y lote, vencimiento diferente | Informar conflicto y volver a revisar el paso 2. No sobrescribir la fecha ni crear otro lote para evitar el conflicto. |
| A5 | Paso 4: vencimiento anterior al día actual | Mostrar vencido. Se permite el alta de catálogo con saldo cero; una recepción posterior exige BR-15. |
| A6 | Alta simultánea, fallo, respuesta perdida o cancelación | Aplicar CF-02/CF-04. Recuperar por producto–lote y verificar vencimiento; no crear un lote parcial ni afirmar éxito con datos distintos. |

**Postcondiciones de éxito:** Existe un lote nuevo asociado a un producto, con saldo cero y sin primera recepción, o se devuelve explícitamente el lote existente.

**Postcondiciones de fallo:** No se crea un lote inválido ni se modifica un lote existente. No hay efecto de inventario.

**Reglas relacionadas:** BR-01, BR-02, BR-11, BR-12, BR-13, BR-14, BR-17, BR-23, BR-24.

---

### UC-03 — Registrar recepción de inventario

**Actor principal:** Inventory Staff  
**Objetivo:** Incorporar unidades a un lote con un movimiento RECEIVED completo y recuperable.  
**Disparador:** Se registra material recibido físicamente.

**Precondiciones:** Conocer producto, lote, cantidad incorporada y personas participantes. Producto y lote pueden crearse durante el recorrido.

**Datos de entrada:** Producto y lote; cantidad entera; nombre de quien registra; nombre de quien recibió físicamente; nota opcional. Para un lote vencido: nota y aceptación explícita obligatorias. El vencimiento se captura solamente si se crea un lote mediante UC-02.

**Flujo principal**

1. Iniciar recepción y seleccionar el producto. Si falta, completar UC-01 y regresar con el producto.
2. Seleccionar el lote del producto. Si falta, completar UC-02 y regresar con el lote.
3. Mostrar unidad, vencimiento registrado, saldo y disponible. Cotejar la identificación y el vencimiento con el material recibido.
4. Ingresar cantidad incorporada, quién registra, quién recibió y nota cuando corresponda.
5. Validar datos. Si el lote está vencido, mostrar disponible cero, exigir nota y aceptación explícita de registrar material vencido.
6. Mostrar el resumen y enviar la confirmación según CF-03.
7. Para un intento nuevo, validar bajo protección producto–lote, campos, unidad y vencimiento vigentes. Comprobar aceptación y nota si el lote está vencido.
8. Registrar de forma indivisible un RECEIVED con magnitud q y aumentar el saldo vigente en q, según CF-03.
9. Mostrar el movimiento y sus resultados. Obtener del historial la primera recepción y el total registrado como recibido.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Paso 1, 2 o 7: producto o lote inválido | Volver a seleccionar en el paso 1 o 2; conservar cero efectos de esta recepción. |
| A2 | Paso 3: identificación o vencimiento físico no coincide | Detener y revisar el producto y lote desde el paso 1 o 2. No cambiar metadatos ni confirmar mientras persista la discrepancia. |
| A3 | Paso 4, 5 o 7: cantidad, persona o nota obligatoria inválida | Informar el campo y volver al paso 4. |
| A4 | Paso 5: se rechaza registrar material vencido | Cancelar la recepción según CF-02. |
| A5 | Paso 7: el lote venció desde la revisión y falta aceptación | Rechazar ese intento sin movimiento y volver al paso 5. La nueva confirmación tiene nuevo identificador y la aceptación requerida. |
| A6 | Recepción sobre lote con entregas anteriores | Continuar normalmente; agregar un nuevo RECEIVED, sin crear otro lote ni alterar la primera recepción. |
| A7 | Reenvío, fallo, cierre después del envío o respuesta perdida | Aplicar CF-03 antes de volver a ejecutar la operación. Resolver un resultado desconocido sin asumir saldo sin cambios. |

**Postcondiciones de éxito:** Se conserva exactamente un RECEIVED por operación y el saldo aumenta en q. Disponible equivale al saldo si está vigente y a cero si está vencido. Una recuperación de resultado conserva el movimiento original.

**Postcondiciones de fallo:** En rechazo conocido o reversión confirmada no hay movimiento ni aumento de esta recepción. Las altas de catálogo ya confirmadas permanecen. Si falta respuesta, el resultado permanece desconocido hasta resolver CF-03.

**Reglas relacionadas:** BR-01, BR-03, BR-04, BR-08, BR-10, BR-12, BR-13, BR-14, BR-15, BR-18, BR-19, BR-21, BR-22, BR-23, BR-24.

---

### UC-04 — Buscar y consultar inventario

**Actor principal:** Inventory Staff e Inventory Consumer  
**Objetivo:** Localizar productos o lotes y distinguir su saldo de la cantidad habilitada para salida.  
**Disparador:** Se necesita conocer o localizar inventario.

**Precondiciones:** Ninguna.

**Datos de entrada:** Consulta por nombre de producto o por número de lote. Una consulta vacía muestra el catálogo de productos.

**Flujo principal**

1. Abrir la consulta de inventario.
2. Seleccionar búsqueda por nombre o número de lote e ingresar el término; permitir consulta vacía para listar productos.
3. Buscar coincidencias parciales de nombre sin distinguir mayúsculas, o coincidencia exacta del número de lote normalizado.
4. Mostrar coincidencias con código, nombre que incluye variante, fabricante y unidad; si son lotes, identificar también su producto.
5. Seleccionar un producto o lote y recuperar una lectura coherente de sus datos.
6. Mostrar lotes, vencimientos, saldos y disponibilidad, incluidos lotes agotados, vencidos o sin recepción. Si se presenta total disponible del producto, sumar la disponibilidad de sus lotes.
7. Terminar la consulta, realizar otra búsqueda o iniciar explícitamente otra operación.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Paso 3: sin coincidencias | Informar que no hay coincidencias y volver al paso 2. |
| A2 | Paso 5–6: producto sin lotes | Mostrar el producto y sin lotes; no inventar registros de lote. Permitir volver al paso 2. |
| A3 | Paso 6: lote sin recepción o agotado | Mostrar saldo cero y distinguir sin recepción de saldo agotado con historial. |
| A4 | Paso 6: lote vencido con saldo | Mostrar vencido, saldo conservado y disponible cero; no permitir seleccionarlo como lote habilitado para salida. |
| A5 | Paso 3 o 5: fallo de lectura | Informar que no se pudo consultar y permitir reintento. No mostrar el fallo como cero o como ausencia de resultados. |
| A6 | Cierre o cancelación de consulta | Terminar sin modificaciones. |

**Postcondiciones de éxito:** Se obtiene información de la lectura realizada. No existe reserva ni garantía de que la disponibilidad siga igual al confirmar una salida.

**Postcondiciones de fallo:** No se modifica producto, lote, saldo ni historial. Un fallo de consulta no aporta una conclusión sobre las existencias.

**Reglas relacionadas:** BR-02, BR-04, BR-06, BR-11, BR-12, BR-18, BR-22, BR-24.

---

### UC-05 — Registrar salida de inventario

**Actor principal:** Inventory Consumer  
**Objetivo:** Registrar una entrega para uso desde un lote vigente sin exceder lo disponible.  
**Disparador:** Se entrega material para consumo o uso.

**Precondiciones:** Conocer cantidad y personas participantes. La existencia del lote y la disponibilidad se verifican durante el recorrido y al confirmar.

**Datos de entrada:** Producto y lote; magnitud entregada; nombre de quien registra; nombre de quien recibe para uso; nota opcional.

**Flujo principal**

1. Buscar el producto y sus lotes mediante UC-04.
2. Seleccionar un lote habilitado y mostrar unidad, saldo, disponible y vencimiento.
3. Ingresar cantidad, quién registra y a quién se entrega; agregar nota si se necesita.
4. Validar campos y mostrar el resumen. Su preparación no reserva unidades.
5. Enviar la confirmación según CF-03.
6. Para un intento nuevo, validar bajo protección producto–lote, campos, vigencia al confirmar y q no superior al disponible actual.
7. Registrar de forma indivisible un ISSUED con magnitud q y disminuir el saldo en q, según CF-03.
8. Mostrar el movimiento y el resultado de la salida.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Paso 1–2: no existe un lote aplicable o hay disponible cero | Volver a buscar o terminar sin salida. |
| A2 | Paso 3–4 o 6: campos inválidos | Volver al paso 3 con la causa del rechazo. |
| A3 | Paso 6: lote inexistente o ajeno al producto | Rechazar y volver al paso 1 o 2. |
| A4 | Paso 6: el lote está vencido, incluso si era vigente al abrir | Rechazar y volver al paso 2. No existe salida excepcional de vencidos. |
| A5 | Paso 6: disponible insuficiente, incluso por otra operación simultánea | Rechazar, mostrar el disponible actual y volver al paso 3 o seleccionar otro lote. No retirar parcialmente ni distribuir entre lotes automáticamente. |
| A6 | Reenvío, fallo, cancelación o respuesta perdida | Aplicar CF-02/CF-03. Un reenvío ya confirmado recupera la salida original antes de evaluar stock o vencimiento actuales. |

**Postcondiciones de éxito:** Se conserva exactamente un ISSUED por operación y el saldo disminuye en q sin ser negativo. La entrega queda identificada; no se crea una solicitud ni una reserva.

**Postcondiciones de fallo:** Rechazo conocido o reversión confirmada: ningún efecto de esta salida. Resultado desconocido: recuperar el intento original, sin afirmar que no se entregó o que no se guardó.

**Reglas relacionadas:** BR-01, BR-03, BR-04, BR-05, BR-06, BR-08, BR-10, BR-18, BR-19, BR-21, BR-22, BR-23, BR-24.

---

### UC-06 — Registrar inventario dañado

**Actor principal:** Inventory Staff  
**Objetivo:** Dar de baja unidades dañadas incluidas en el saldo y conservar la explicación.  
**Disparador:** Se identifica daño en unidades que ya forman parte del inventario controlado.

**Precondiciones:** Poder identificar el lote y la cantidad dañada. La suficiencia de saldo se comprueba al confirmar.

**Datos de entrada:** Producto y lote; magnitud dañada; razón obligatoria; nombre de quien registra.

**Flujo principal**

1. Buscar y seleccionar el lote mediante UC-04, incluso si está vencido.
2. Mostrar unidad, saldo y vencimiento.
3. Ingresar magnitud dañada, razón y quién registra; excluir unidades dadas de baja previamente.
4. Validar campos, revisar el efecto sobre el saldo y enviar la confirmación según CF-03.
5. Para un intento nuevo, validar bajo protección producto–lote y campos, y comprobar q no superior al saldo vigente.
6. Registrar de forma indivisible un DAMAGED y disminuir el saldo en q, según CF-03.
7. Mostrar el movimiento, el saldo y la disponibilidad resultantes.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Paso 1 o 5: lote inválido o no corresponde al producto | Volver a seleccionar en el paso 1. |
| A2 | Paso 3–5: cantidad, razón o persona inválida | Volver al paso 3 con el error. |
| A3 | Paso 5: q supera el saldo | Rechazar y revisar la cantidad desde el paso 3; no registrar una baja parcial. |
| A4 | Lote vencido con saldo suficiente | Continuar normalmente; la baja utiliza saldo y deja disponible cero. |
| A5 | Reenvío, fallo, cancelación o respuesta perdida | Aplicar CF-02/CF-03. |

**Postcondiciones de éxito:** Un DAMAGED reduce el saldo en q y conserva una razón. Las unidades dadas de baja dejan de formar parte del conteo; no se descuentan de nuevo por vencimiento.

**Postcondiciones de fallo:** Rechazo conocido o reversión confirmada: ningún efecto de este daño. Si falta respuesta, resolver el intento por CF-03.

**Reglas relacionadas:** BR-01, BR-03, BR-04, BR-07, BR-08, BR-10, BR-19, BR-21, BR-22, BR-23, BR-24.

---

### UC-07 — Registrar ajuste de inventario

**Actor principal:** Inventory Staff  
**Objetivo:** Corregir una diferencia de cantidad o registrar una baja por vencimiento mediante un movimiento adicional.  
**Disparador:** Se detecta discrepancia, error de cantidad en un registro previo o retiro efectivo por vencimiento.

**Precondiciones:** Identificar el lote afectado y la causa del ajuste. El lote debe tener al menos un RECEIVED; se valida antes de confirmar.

**Datos de entrada:** Producto y lote; dirección INCREASE o DECREASE; magnitud positiva; razón; nombre de quien registra. Si se corrige un movimiento conocido, su identificador es obligatorio. El sistema conserva la referencia del estado revisado.

**Flujo principal**

1. Seleccionar el lote y mostrar unidad, saldo, vencimiento y recepción previa; conservar la referencia de su estado para detectar movimientos posteriores.
2. Seleccionar aumentar o disminuir e ingresar una magnitud positiva. El dato es la diferencia que se aplicará, no el saldo final.
3. Ingresar razón, quién registra y referencia del movimiento origen cuando exista. Revisar que el conteo excluya unidades ya dadas de baja.
4. Mostrar saldo anterior, efecto y saldo resultante; verificar la corrección propuesta.
5. Enviar la confirmación según CF-03.
6. Para un intento nuevo, validar bajo protección producto–lote, campos, recepción previa, referencia de corrección del mismo lote y ausencia de movimientos posteriores a la revisión.
7. Comprobar que el saldo resultante no sea negativo y registrar de forma indivisible un ADJUSTED con dirección, magnitud, razón y efecto, según CF-03.
8. Mostrar el nuevo movimiento y el resultado, conservando intactos todos los movimientos anteriores.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Paso 1 o 6: lote inválido | Volver al paso 1. |
| A2 | Paso 1 o 6: lote sin recepción previa | Rechazar y dirigir a UC-03 cuando corresponda. No usar ADJUSTED como recepción inicial. |
| A3 | Paso 2–4 o 6: magnitud cero/inválida, dirección, persona o razón ausente | Volver al campo correspondiente en el paso 2 o 3. Disminuir todo el saldo hasta cero sí es válido. |
| A4 | Paso 3 o 6: referencia de movimiento inexistente o de otro lote | Rechazar y corregir el paso 3; el ajuste no cambia el lote de un movimiento. |
| A5 | Paso 6: hubo cualquier movimiento después de la revisión | Rechazar el intento sin efecto. Recargar el lote, revisar el conteo y volver al paso 2 para recalcular; exigir un nuevo resumen y una nueva confirmación. |
| A6 | Paso 7: saldo propuesto negativo | Rechazar y volver al paso 2. |
| A7 | Lote vencido o saldo cero con recepción histórica | El vencimiento no impide ajustar y no se elimina mediante un aumento. Con saldo cero solo un aumento justificado puede pasar las validaciones. |
| A8 | Reenvío, fallo, cancelación o respuesta perdida | Aplicar CF-02/CF-03. Recuperar un ajuste ya confirmado antes de comprobar si el lote cambió desde su revisión original. |

**Postcondiciones de éxito:** Un nuevo ADJUSTED explica el cambio y conserva su dirección y razón. El saldo no es negativo. En lote vencido, disponible permanece cero.

**Postcondiciones de fallo:** Rechazo conocido o reversión confirmada: ningún efecto del ajuste intentado; el historial anterior permanece intacto. Un resultado desconocido se recupera según CF-03.

**Reglas relacionadas:** BR-01, BR-03, BR-04, BR-08, BR-09, BR-10, BR-16, BR-19, BR-20, BR-21, BR-22, BR-23, BR-24.

---

### UC-08 — Consultar historial de movimientos del lote

**Actor principal:** Inventory Staff  
**Objetivo:** Explicar el saldo mediante movimientos y la disponibilidad mediante saldo y vencimiento.  
**Disparador:** Se revisa el inventario o se investiga una discrepancia.

**Precondiciones:** Disponer de datos de búsqueda; la existencia del lote se verifica durante el recorrido.

**Datos de entrada:** Producto o referencia de lote para buscar. La consulta no admite modificar movimientos.

**Flujo principal**

1. Buscar y seleccionar un lote mediante UC-04, incluso si está agotado o vencido.
2. Solicitar el historial.
3. Recuperar una lectura coherente de los datos del lote y sus movimientos.
4. Mostrar producto, unidad, lote, vencimiento, fecha/cantidad de primera recepción registrada, total registrado como recibido, saldo y disponible.
5. Mostrar los movimientos en orden estable de confirmación por lote, resolviendo empates de fecha/hora mediante ese orden.
6. Por movimiento, mostrar identificador, tipo, magnitud, efecto con signo, fecha/hora, quién registra, participante específico de recepción/salida, razón o nota y referencia de corrección cuando corresponda.
7. Mostrar o permitir reconstruir el efecto acumulado desde cero y comprobar que explica el saldo; aplicar el vencimiento para explicar el disponible.

**Flujos alternativos y excepciones**

| Rama | Condición y punto de entrada | Acción y retorno |
| --- | --- | --- |
| A1 | Paso 1 o 3: lote inexistente | Informar y volver al paso 1. |
| A2 | Paso 3: lote sin movimientos | Mostrar saldo y total recibido cero, sin fecha de primera recepción e historial vacío; terminar correctamente. |
| A3 | Paso 3: fallo de consulta | Informar el fallo y permitir reintento; no mostrar un historial vacío como resultado del fallo. |
| A4 | Paso 7: saldo no coincide con el efecto neto del historial | Señalar inconsistencia y conservar la evidencia; no modificar, borrar ni inventar movimientos para cuadrarla. Una incoherencia interna requiere investigación técnica, no un ajuste que oculte el fallo. |
| A5 | Cierre de consulta | Terminar sin modificaciones. |

**Postcondiciones de éxito:** El usuario puede inspeccionar los registros y explicar saldo y disponibilidad correspondientes a la lectura realizada. Primera recepción y total recibido conservan los registros originales.

**Postcondiciones de fallo:** Ningún registro ni cantidad se modifica; se informa la causa que impidió consultar o explicar el saldo.

**Reglas relacionadas:** BR-04, BR-06, BR-09, BR-10, BR-12, BR-22, BR-24.

## 7. Criterios de aceptación

Cada fila utiliza sus propios datos iniciales, salvo la primera, que describe una secuencia. Para los ejemplos de fecha se fija el día del laboratorio en **2026-09-08**; no dependen del día real en que alguien ejecute la prueba.

| ID | Situación | Resultado esperado en v2.0 |
| --- | --- | --- |
| T-01 | Lote vigente: recibir 100, entregar 20, dañar 5, ajustar +3 y ajustar -2 | Saldo y disponible 76; cinco movimientos con razones en daño y ajustes. |
| T-02 | Crear un lote sin recibir material | Saldo 0, sin movimientos, sin fecha de primera recepción. |
| T-03 | Mismo producto y lote: recibir 100 y después 50 | Un lote, dos `RECEIVED`, saldo 150; primera recepción 100 y total recibido 150. |
| T-04 | Registrar el mismo producto y lote con diferente vencimiento | Conflicto explícito; no crear otro lote ni modificar el vencimiento existente. |
| T-05 | Dos productos distintos usan el número A234 | Se permiten dos lotes; cada resultado de búsqueda identifica su producto. |
| T-06 | Repetir código de producto con cambios de mayúsculas o espacios externos | No se crea otro producto; se ofrece el existente. |
| T-07 | Cantidad vacía, texto, 0, -1 o 1,5 unidades contables | Rechazo; sin redondeo ni movimiento. |
| T-08 | Lote vigente con saldo 8; intentar entregar 9 | Rechazo, saldo 8 y ningún `ISSUED` nuevo. |
| T-09 | Lote vigente con saldo 8; entregar exactamente 8 | Confirmación, saldo 0; historial consultable. |
| T-10 | Saldo 20; vencimiento 2026-09-07 | Saldo 20, disponible 0 y salida rechazada. El vencimiento por sí solo no crea movimiento. |
| T-11 | Vencimiento 2026-09-08; confirmar durante ese día | La fecha todavía permite salida, si se cumplen las demás reglas. Al comenzar el día siguiente queda vencido. |
| T-12 | Abrir salida el último día vigente y confirmar después de medianoche | Se rechaza por el día vigente al confirmar, aunque el formulario mostrara disponibilidad. |
| T-13 | Recibir 10 unidades ya vencidas | Exigir advertencia aceptada y nota; un `RECEIVED`, saldo +10 y disponible 0. Si se cancela, no hay recepción. |
| T-14 | Lote vencido con saldo 20; registrar 5 dañadas | Se permite: saldo 15, disponible 0 y un `DAMAGED` de 5. |
| T-15 | Lote vencido con saldo 15; dar de baja todo por vencimiento | Un `ADJUSTED` disminuir 15 con razón; saldo y disponible 0. |
| T-16 | Lote con recepción previa y saldo 0; ajuste aumentar 3 justificado | Saldo 3; disponible 3 si está vigente o 0 si está vencido. |
| T-17 | Lote sin ninguna recepción; intentar ajuste aumentar 3 | Rechazo: no sustituye la recepción inicial. |
| T-18 | Dos personas intentan retirar 8 sobre saldo 10 | Una salida confirmada y otra rechazada; saldo final 2 y un solo movimiento de salida. |
| T-19 | Dos recepciones distintas, de 5 y 7, se confirman sobre saldo 10 | Saldo 22 y ambos movimientos presentes, sin pérdida de una actualización. |
| T-20 | Reenviar dos veces la misma confirmación válida | Un movimiento; el reenvío recupera el resultado original. |
| T-21 | Reutilizar identificador de confirmación con una cantidad diferente | Conflicto; no registrar otra modificación con ese identificador. |
| T-22 | Revisar ajuste sobre saldo 50; otro movimiento ocurre antes de confirmarlo | Detener ajuste y exigir nueva revisión del conteo y del efecto. No aplicar la propuesta anterior automáticamente. |
| T-23 | Guardado falla antes de completar la operación y se confirma la reversión | No se conserva movimiento ni cambio de saldo de esa operación. |
| T-24 | Guardado termina, pero se pierde la respuesta | Consultar/reintentar con el mismo identificador; reconocer el movimiento existente sin duplicarlo. |
| T-25 | Crear lote desde recepción y cancelar después el formulario de recepción | El lote puede permanecer con saldo 0; no se crea `RECEIVED`. |
| T-26 | Daño o ajuste sin razón; cualquier movimiento sin quién registra | Rechazo antes de guardar. Recepción/salida también exigen a su participante específico. |
| T-27 | Corregir una salida capturada con 2 unidades de más en el mismo lote | Agregar ajuste aumentar 2 con razón y referencia; conservar la salida original y mostrar saldo neto corregido. |
| T-28 | Consulta de un lote sin movimientos frente a un fallo de conexión | El primero muestra historial vacío y saldo 0; el segundo informa error, sin afirmar que no existe inventario. |
| T-29 | Reenviar una salida confirmada después de que el lote se agotó o venció | Recuperar la salida original; no rechazarla por el estado actual ni registrar otra salida. |
| T-30 | Dos solicitudes simultáneas con el mismo identificador de confirmación | Se ejecuta una sola; ambas recuperan el mismo resultado cuando se resuelve. |
| T-31 | Otro movimiento modifica el lote y luego una segunda operación devuelve el saldo al valor revisado | El ajuste antiguo sigue rechazándose: cambió el estado aunque coincida el número. |
| T-32 | Ajuste intenta referir un movimiento inexistente o perteneciente a otro lote | Rechazo y ningún ADJUSTED nuevo. |
| T-33 | Crear lote con fecha ausente, fecha incompleta o 2026-02-30 | Rechazo; no inventar ni corregir silenciosamente el vencimiento. |
| T-34 | Un alta de producto pierde la respuesta y se reintenta con el mismo código | Recuperar el registro y comparar datos; no crear otro producto. |
| T-35 | Cambiar la zona horaria del navegador para intentar usar un lote vencido | El servidor mantiene la decisión según el día configurado del laboratorio. |
| T-36 | Recepción de 10 unidades incorporadas y 2 rechazadas por daño antes del ingreso | RECEIVED cuenta 10; las 2 rechazadas no se suman ni vuelven a restarse. Su rechazo puede constar en nota. |
| T-37 | Producto con lote vigente de saldo 8 y lote vencido de saldo 12 | Disponible total del producto 8; los saldos por lote se conservan y la suma de saldos es 20. |
| T-38 | Reenviar un ajuste ya confirmado después de movimientos posteriores del lote | Recuperar el ajuste original antes de comparar la referencia de estado; no duplicarlo ni convertirlo en un nuevo rechazo por desactualización. |

## 8. Relaciones entre casos y preparación de diagramas

Los casos son operaciones independientes. Una cadena de demostración no impone navegación ni obliga a emitir inventario antes de registrar daño o ajuste.

| Caso | Relación con otros casos | Decisiones que necesita el diagrama |
| --- | --- | --- |
| UC-01 | Puede invocarse desde UC-02 o UC-03 si falta producto. | Datos completos, coincidencia de código, recuperación de alta existente. |
| UC-02 | Requiere producto; puede invocarse desde UC-03. | Producto válido, lote existente, coincidencia de vencimiento, fecha válida. |
| UC-03 | Selecciona o reutiliza UC-01/02; puede apoyarse en UC-04. | Selección/alta, cotejo de lote, cantidad, vencimiento, aceptación de recepción vencida y CF-03. |
| UC-04 | Puede abrirse directamente o desde UC-03/05/06/07/08. | Búsqueda con o sin coincidencias, producto sin lotes, lotes sin recepción/agotados/vencidos, fallo de lectura. |
| UC-05 | Usa UC-04; no obliga a ejecutar daño, ajuste o historial después. | Campos válidos, lote vigente, disponible suficiente y CF-03. |
| UC-06 | Usa UC-04; no requiere una salida previa. | Razón, cantidad, saldo suficiente y CF-03. |
| UC-07 | Usa UC-04; requiere recepción histórica y puede referir un movimiento. | Dirección, razón, referencia válida, estado revisado vigente, saldo final no negativo y CF-03. |
| UC-08 | Usa UC-04 y puede consultarse sin cambiar stock. | Existencia de lote, consulta correcta, historial vacío y coherencia de cantidades. |

Crear un diagrama por caso. Puede representarse CF-03 como subproceso compartido de los cuatro movimientos. Validar una cantidad, comprobar vencimiento o mostrar un error son pasos internos, no nuevos casos de negocio.

Cada rama de rechazo debe volver al punto de corrección indicado o terminar. El guardado de movimiento y su efecto se representa como una operación indivisible. Cancelar antes de enviar y cerrar después del envío tienen rutas diferentes. La recuperación de un reenvío ocurre antes de repetir las validaciones dependientes del estado actual.

## 9. Cierre de la revisión

| Hallazgo del informe | Resolución incorporada |
| --- | --- |
| H-01: alcance contradictorio | Sección 2; charter y MVP v2.0 alineados. |
| H-02: identidad de producto | BR-11; UC-01. |
| H-03: duplicación y entregas repetidas del lote | BR-02/13; UC-02/03. |
| H-04: alta mezclada con recepción | BR-12/23; UC-02/03. |
| H-05: dirección del ajuste | BR-03/16; UC-07. |
| H-06: unidades y fracciones | BR-03/17; sección 2. |
| H-07: saldo frente a disponible | BR-04/07/08; sección 3. |
| H-08: vencimiento incompleto | BR-06/14/15; UC-02/03/05/06/07. |
| H-09: daño y correcciones históricas | BR-07/09/16; UC-06/07/08. |
| H-10: personas y solicitudes | BR-10/18; sección 4. |
| H-11: simultaneidad y ajuste desactualizado | BR-19/20; CF-03. |
| H-12: cancelación y reintentos | BR-21/23/24; CF-02/03/04. |
| H-13: consultas ambiguas | BR-22/24; UC-04/08. |
| H-14: mapa secuencial incorrecto | Sección 8. |
| H-15: criterios insuficientes | Sección 7: 38 escenarios de aceptación. |
| H-16: saldo almacenado o derivado | Invariantes BR-04/08/19 y nota de diseño siguiente. |

**Nota de diseño:** queda para el ERD elegir si el saldo se calcula al consultar o se conserva como un valor actualizado. Esto no es una regla de negocio pendiente: ambas alternativas deben respetar la misma suma de movimientos, la protección frente a simultaneidad y el protocolo de confirmación. Si se guarda saldo, se actualiza de forma indivisible con el movimiento.

**Verificación de esta versión:** revisión documental de consistencia, referencias, caminos alternativos y aritmética de ejemplos. Los escenarios especifican resultados que deberán comprobarse en la aplicación; no se declara que exista una implementación validada.
