# Laboratory Inventory Management System
## Minimum Viable Product — especificación v2.0

**Versión documental:** 2.0  
**Estado:** alcance y reglas corregidos, alineados con los casos de uso v2.0.  
**Entrega:** MVP inicial; esta revisión documental no anuncia una segunda versión de software.

### 1. Problema

Un inventario que solo guarda el total por producto no permite identificar el lote, su vencimiento ni explicar las variaciones de cantidad. El sistema debe conservar los movimientos de cada lote y distinguir lo registrado de lo habilitado para uso.

### 2. Objetivo del MVP

Construir una aplicación web que permita registrar productos y lotes, recibir material, consultar disponibilidad, registrar entregas, daño y ajustes y reconstruir el saldo mediante el historial. Estas son operaciones independientes; la demostración de extremo a extremo no impone que deban ejecutarse siempre en un orden fijo.

### 3. Usuarios del MVP

| Actor | Responsabilidades |
| --- | --- |
| Inventory Staff | Registrar productos, lotes, recepciones, daños y ajustes; consultar inventario e historial. |
| Inventory Consumer | Buscar inventario y registrar entregas para uso. |

Estos actores describen formas de uso. No constituyen dos roles con restricciones de acceso implementadas. Para movimientos se registran nombres declarados: quien captura y, en RECEIVED/ISSUED, el participante correspondiente. No se promete identidad verificada ni un módulo de empleados.

### 4. Entidades y operaciones conceptuales

Un producto tiene cero o muchos lotes. Cada lote pertenece a un producto y tiene cero o muchos movimientos. Cada movimiento afecta exactamente un lote. Los identificadores internos de confirmación permiten recuperar el resultado de una operación sin duplicarla.

RECEIVED, ISSUED, DAMAGED y ADJUSTED son los tipos de movimiento. Recepción y salida son operaciones que, al confirmarse, generan un movimiento sobre un lote. No se exige una cabecera con múltiples artículos ni se define todavía el esquema físico de tablas.

### 5. Producto

Datos mínimos: identificador técnico generado por el sistema, código interno único, nombre que distingue la variante, categoría, fabricante y unidad base contable. El operador revisa el catálogo antes de crear otro producto. Un código único evita duplicar ese código, pero no reconoce por sí solo materiales equivalentes con nombres o códigos distintos.

La unidad es la misma para todos los movimientos del producto y admite enteros. El catálogo no incluye edición ni eliminación posteriores al alta en este MVP.

### 6. Lote

Datos propios: identificador técnico, producto, número de lote y vencimiento completo. La combinación producto y número normalizado es única. Crear el lote deja saldo cero y no genera movimiento.

Fecha y cantidad de primera recepción se obtienen del primer RECEIVED confirmado. Antes de él, se muestra sin recepción y cantidad inicial cero. Otras entregas del mismo lote agregan nuevos RECEIVED sin reiniciar su historia. Una discrepancia de vencimiento exige revisar datos y no autoriza sobrescribir el lote.

### 7. Movimientos y cantidades

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

Cada movimiento conserva lote, tipo, magnitud, dirección cuando es ADJUSTED, identificador, fecha/hora del sistema, quién registra, participante específico en RECEIVED/ISSUED, nota o razón y referencia de corrección cuando corresponda. La confirmación es recuperable mediante su identificador interno de operación.

### 8. Recepción de inventario

Seleccionar o crear producto y lote; cotejar identificación y vencimiento; ingresar cantidad incorporada y personas; revisar y confirmar. La confirmación genera un RECEIVED y aumenta el saldo de forma indivisible.

La recepción vencida requiere advertencia aceptada y nota; aumenta saldo y mantiene disponible cero. Si cambia el día y el lote vence antes de confirmar, se exige la aceptación correspondiente en un nuevo intento. La primera recepción debe ser RECEIVED, no ADJUSTED.

### 9. Consulta de inventario

Buscar por coincidencia parcial del nombre del producto o coincidencia exacta del número de lote normalizado. Una consulta vacía lista productos. Mostrar producto, variante, fabricante, unidad, lote, vencimiento, saldo y disponible. Distinguir producto inexistente, producto sin lotes, lote sin recepción, lote agotado, lote vencido y fallo de lectura.

Los lotes agotados y vencidos se conservan. Consultar no reserva unidades ni garantiza disponibilidad futura.

### 10. Salida de inventario

Buscar producto, seleccionar lote vigente, ingresar cantidad y personas, revisar y confirmar. La validación final protegida comprueba lote, unidad, vencimiento y disponible actual. El registro indivisible de ISSUED reduce saldo. No hay salidas parciales, reparto automático entre lotes, solicitudes pendientes ni reservas.

### 11. Reglas de negocio del MVP

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

Las rutas comunes de validación, cancelación, recuperación de confirmación, fallos y consultas están desarrolladas en CF-01 a CF-04 de USE_CASES_MVP_v2.0.md. La recuperación de un identificador confirmado ocurre antes de repetir las validaciones de saldo o vencimiento actuales.

### 12. Ajustes y daño

DAMAGED registra unidades dañadas ya incluidas en el saldo, con magnitud positiva y razón obligatoria. Su límite es el saldo, incluso cuando el lote está vencido.

ADJUSTED requiere dirección INCREASE o DECREASE, magnitud positiva, razón y recepción previa. Si el sistema muestra 50 y el conteo correcto es 47, se registra DECREASE 3 y se revisa el resultado 47. Si hubo cualquier movimiento desde esa revisión, el intento se rechaza y exige revisar el conteo y confirmar nuevamente.

Las correcciones de cantidades registradas agregan ajustes, conservando el movimiento original y refiriéndolo cuando se conoce. La referencia debe existir y corresponder al mismo lote. La baja efectiva por vencimiento usa ADJUSTED DECREASE con esa razón. Ajustar un lote vencido no lo habilita para uso.

### 13. Historial

Consultar datos y movimientos de un lote en una lectura coherente. Mostrar identificadores, tipos, magnitudes, efectos, fecha/hora, personas, notas/razones y referencias en orden estable de confirmación. Desde cero, sus efectos explican el saldo; el vencimiento explica la disponibilidad.

La primera recepción y el total registrado como recibido conservan los datos originales aunque haya ajustes correctivos. Ningún movimiento confirmado se edita o elimina. Una discrepancia interna entre saldo e historial se comunica e investiga; no se inventa un ajuste para ocultarla.

### 14. Criterios de aceptación

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

### 15. Fuera del MVP y límites de esta entrega

El MVP inicial permite registrar productos y lotes, recibir y entregar inventario, registrar daño y ajustes, buscar existencias y consultar el historial por lote.

Cada confirmación de inventario trabaja con un lote y una unidad base del producto. Una entrega física puede originar varias confirmaciones independientes. No hay carrito ni confirmación conjunta de varios artículos; tampoco se promete que dos operaciones distintas se completen juntas.

La primera versión trabaja con suministros contables en unidades enteras: por ejemplo, guantes individuales, pares, frascos, cajas cerradas o paquetes. La unidad elegida se muestra en todas las operaciones. Un producto gestionado por caja no admite retirar piezas de esa caja. Se requiere vencimiento completo en todos los lotes.

Quedan para fases posteriores FEFO automático o recomendado, alertas de stock bajo o próximo vencimiento, filtros avanzados, dashboards, operaciones de varios artículos, fracciones, conversiones, gestión de artículos sin vencimiento, edición posterior de catálogo, correcciones conjuntas de varios lotes, fechas físicas retroactivas y gestión de rechazos a proveedores.

También quedan fuera autenticación y autorización completas, administración de empleados, solicitudes con aprobación, reservas, devoluciones con su propio flujo, proveedores y compras, facturación y pagos, códigos de barras/QR, ubicaciones, múltiples laboratorios, reportes avanzados, integraciones externas, notificaciones por email/SMS, aplicación móvil, cuarentena y recall. La consulta y el bloqueo de lotes vencidos sí forman parte del MVP.

### 16. Definición de MVP terminado

El MVP se considera funcional cuando los dos tipos de interacción pueden completar sus operaciones y la implementación demuestra los criterios de la sección 14, incluidos vencimiento, ajustes, guardado indivisible, confirmaciones repetidas y simultaneidad.

Una demostración mínima puede crear producto y lote, recibir 100, entregar 20, registrar 5 dañadas, ajustar +3 y -2 y consultar saldo 76 y sus cinco movimientos. La demostración debe incluir rechazos que no alteren el inventario. Cada operación puede iniciarse de manera independiente cuando se cumplen sus condiciones.

Estos son criterios a implementar y verificar; la documentación por sí sola no demuestra que funcionen.

### 17. Principio central y control de cambios

Todo cambio del saldo debe tener una explicación mediante movimientos. La disponibilidad puede cambiar por vencimiento sin que cambie el saldo.

El charter v2.0 fija la visión por fases y USE_CASES_MVP_v2.0.md desarrolla los ocho recorridos. Estas versiones sustituyen las reglas ambiguas de v1.0 para la entrega inicial. FEFO y las operaciones con varios artículos continúan en la evolución prevista, sin ser criterios de terminación de este MVP.
