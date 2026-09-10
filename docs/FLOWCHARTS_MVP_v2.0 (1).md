# Laboratory Inventory Management System
## Diagramas de flujo - MVP v2.0

**Fecha:** 2026-09-09.  
**Base:** USE_CASES_MVP_v2.0.md, MVP_Laboratory_Inventory_System_v2.0.md y PROJECT_CHARTER_Laboratory_Inventory_System_v2.0.md.  
**Alcance:** ocho diagramas de casos de uso y CF-03 dividido en dos paneles de detalle. Los paneles de CF-03 no son casos de uso adicionales.

## Cómo leerlos

| Forma | Significado |
| --- | --- |
| Terminal redondeado | Inicio, fin o devolución de resultado. |
| Rectángulo | Acción del sistema o paso de trabajo. |
| Paralelogramo | Capturar o mostrar información. |
| Rombo | Decisión; seguir la flecha con la respuesta correspondiente. |
| Rectángulo con doble borde lateral | Subproceso descrito por otro caso o comportamiento común. |

Cada caso es independiente: registrar daño no exige haber realizado una salida antes. Los rectángulos UC-01/UC-02 dentro de recepción permiten seleccionar un registro existente o invocar el alta cuando falta; no obligan a crearlo cada vez.

Las comprobaciones previas ayudan a corregir formularios. En UC-03/05/06/07, CF-03 repite las comprobaciones determinantes al confirmar y realiza el guardado protegido. Las verificaciones previas por sí solas no garantizan stock ni vigencia.

**Cancelar:** desde una captura o búsqueda puede terminarse sin enviar. Las flechas de cancelación en la revisión representan ese comportamiento común, sin repetirlas en cada campo. Después del envío, cerrar no deshace ni demuestra el fracaso de la operación: se recupera el resultado del mismo intento.

**Resultados de CF-03:** Confirmado termina con el movimiento registrado; Rechazado muestra una causa y permite revisar el formulario; Pendiente conserva identificador y datos y recupera el resultado. La vuelta a revisar una operación conserva los datos que no necesitan corrección. Si el intento fue rechazado, la nueva confirmación usa un identificador nuevo.

**CF-04 en las altas:** guardar con unicidad, recuperar un registro existente y comparar sus datos, o resolver una respuesta perdida por código de producto / combinación producto-lote. Mientras una recuperación no se resuelve, permanece en ese subproceso. Un fallo de consulta no prueba que el alta no exista. La v2.0 no permite sobrescribir metadatos de catálogo.

El PDF contiene los mismos nodos y conexiones; el Markdown conserva Mermaid editable. Los retornos hacia puntos distintos usan canales separados. Un cruce de líneas sin punto de unión no cambia el destino indicado por la flecha. Los saltos entre casos y los detalles de CF-03 se nombran explícitamente para mantener legibles los diagramas.

## UC-01 - Registrar producto

**Actor o ámbito:** Personal de inventario.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_search[/"Buscar coincidencias en el catálogo"/]
    n_found{"¿Ya existe el producto correcto?"}
    n_existing[/"Seleccionar producto existente"/]
    n_endexisting(["Fin: devolver producto"])
    n_input[/"Ingresar código, nombre, categoría, fabricante y unidad"/]
    n_valid{"¿Datos válidos?"}
    n_error["Mostrar campos por corregir"]
    n_confirm{"¿Confirmar alta?"}
    n_cancel(["Fin: alta cancelada"])
    n_save[["CF-04: guardar con código único"]]
    n_result{"¿Resultado del alta?"}
    n_conflict["Revisar registro que ya existe"]
    n_failure["Informar fallo sin alta parcial"]
    n_recover[["CF-04: recuperar por código y comparar datos"]]
    n_show[/"Mostrar producto creado"/]
    n_end(["Fin: devolver producto"])
    n_s --> n_search
    n_search --> n_found
    n_found -->|Sí| n_existing
    n_existing --> n_endexisting
    n_found -->|No| n_input
    n_input --> n_valid
    n_valid -->|No| n_error
    n_error --> n_input
    n_valid -->|Sí| n_confirm
    n_confirm -->|No| n_cancel
    n_confirm -->|Sí| n_save
    n_save --> n_result
    n_result -->|Creado| n_show
    n_show --> n_end
    n_result -->|Existente| n_conflict
    n_conflict --> n_search
    n_result -->|Fallo| n_failure
    n_failure --> n_input
    n_result -->|Sin respuesta| n_recover
    n_recover --> n_result
```

- El código es único; el usuario debe revisar también las variantes del material. Cambiar el código no justifica duplicar un producto.
- CF-04 comprueba unicidad al guardar. Si se pierde la respuesta, recupera por código y compara datos. No sobrescribe el registro encontrado.
- Crear o seleccionar un producto no genera stock ni movimientos.

**Referencia:** BR-11, BR-12, BR-17, BR-23, BR-24; CF-04.

## UC-02 - Registrar lote

**Actor o ámbito:** Personal de inventario.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_product[/"Seleccionar producto"/]
    n_pvalid{"¿Producto válido?"}
    n_createp[["Seleccionar otro producto o usar UC-01"]]
    n_input[/"Ingresar número de lote y vencimiento"/]
    n_valid{"¿Número y fecha válidos?"}
    n_error["Corregir campos"]
    n_exists{"¿Existe producto + lote?"}
    n_match{"¿Coincide el vencimiento?"}
    n_mismatch["Informar discrepancia; revisar datos"]
    n_existing[/"Devolver lote existente"/]
    n_endexisting(["Fin: sin crear otro lote"])
    n_confirm{"¿Confirmar alta?"}
    n_cancel(["Fin: alta cancelada"])
    n_save[["CF-04: guardar lote único, saldo cero"]]
    n_result{"¿Resultado del alta?"}
    n_failure["Informar fallo sin alta parcial"]
    n_concurrent[/"Comparar lote encontrado"/]
    n_recover[["CF-04: recuperar por producto + lote"]]
    n_show[/"Mostrar lote con saldo cero"/]
    n_end(["Fin: devolver lote"])
    n_s --> n_product
    n_product --> n_pvalid
    n_pvalid -->|No| n_createp
    n_createp --> n_product
    n_pvalid -->|Sí| n_input
    n_input --> n_valid
    n_valid -->|No| n_error
    n_error --> n_input
    n_valid -->|Sí| n_exists
    n_exists -->|Sí| n_match
    n_match -->|Sí| n_existing
    n_existing --> n_endexisting
    n_match -->|No| n_mismatch
    n_mismatch --> n_input
    n_exists -->|No| n_confirm
    n_confirm -->|No| n_cancel
    n_confirm -->|Sí| n_save
    n_save --> n_result
    n_result -->|Creado| n_show
    n_show --> n_end
    n_result -->|Fallo| n_failure
    n_failure --> n_input
    n_result -->|Existente| n_concurrent
    n_concurrent --> n_match
    n_result -->|Sin respuesta| n_recover
    n_recover --> n_result
```

- La clave es producto + número de lote normalizado. Un lote existente conserva su saldo y su historial.
- Crear el lote no es recibir unidades: comienza con saldo cero, sin movimientos y sin fecha de primera recepción.
- Se permite dar de alta un lote vencido. Recibirlo exige la advertencia y aceptación de UC-03. Un vencimiento diferente no se sobrescribe.

**Referencia:** BR-01, BR-02, BR-11, BR-12, BR-13, BR-14; CF-04.

## UC-03 - Registrar recepción

**Actor o ámbito:** Personal de inventario.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_product[["Seleccionar producto o usar UC-01"]]
    n_lot[["Seleccionar lote o usar UC-02"]]
    n_match{"¿Coinciden los datos físicos y registrados?"}
    n_mismatch["Revisar identificación y vencimiento"]
    n_input[/"Ingresar cantidad, quién registra, quién recibió y nota"/]
    n_valid{"¿Cantidad y personas válidas?"}
    n_error["Corregir datos"]
    n_expired{"¿Lote vencido?"}
    n_accept{"¿Acepta registrar como no disponible?"}
    n_cancel_exp(["Fin: recepción cancelada"])
    n_note[/"Ingresar nota obligatoria"/]
    n_noteok{"¿Nota válida?"}
    n_confirm{"¿Confirmar recepción?"}
    n_cancel(["Fin: recepción cancelada"])
    n_cf[["CF-03: confirmar o recuperar RECEIVED"]]
    n_result{"¿Resultado?"}
    n_reject["Mostrar causa y revisar la recepción"]
    n_recover[["Recuperar el mismo intento"]]
    n_end(["Fin: RECEIVED registrado"])
    n_s --> n_product
    n_product --> n_lot
    n_lot --> n_match
    n_match -->|No| n_mismatch
    n_mismatch --> n_product
    n_match -->|Sí| n_input
    n_input --> n_valid
    n_valid -->|No| n_error
    n_error --> n_input
    n_valid -->|Sí| n_expired
    n_expired -->|Sí| n_accept
    n_accept -->|No| n_cancel_exp
    n_accept -->|Sí| n_note
    n_note --> n_noteok
    n_noteok -->|No| n_note
    n_noteok -->|Sí| n_confirm
    n_expired -->|No| n_confirm
    n_confirm -->|No| n_cancel
    n_confirm -->|Sí| n_cf
    n_cf --> n_result
    n_result -->|Confirmado| n_end
    n_result -->|Rechazado| n_reject
    n_reject --> n_product
    n_result -->|Pendiente| n_recover
    n_recover --> n_cf
```

- CF-03 vuelve a validar al confirmar. Si el lote venció desde la revisión, se rechaza ese intento y se exige nota y aceptación antes de una nueva confirmación.
- Un lote ya registrado recibe otro RECEIVED; no se crea de nuevo. La recepción aumenta saldo en q. Si está vencido, disponible sigue en cero.
- Cancelar conserva las altas de producto o lote que ya fueron confirmadas. La cantidad recibida excluye unidades rechazadas por daño antes de incorporarlas.

**Referencia:** BR-01, BR-03, BR-08, BR-10, BR-12, BR-13, BR-15, BR-19, BR-21, BR-23; CF-03.

## UC-04 - Buscar y consultar inventario

**Actor o ámbito:** Personal de inventario y consumidor.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_input[/"Elegir búsqueda e ingresar nombre o lote"/]
    n_query["Consultar catálogo"]
    n_ok{"¿Consulta correcta?"}
    n_error["Mostrar fallo de consulta"]
    n_found{"¿Hay coincidencias?"}
    n_empty[/"Mostrar sin coincidencias"/]
    n_select[/"Seleccionar producto o lote"/]
    n_details["Consultar datos del seleccionado"]
    n_readok{"¿Lectura correcta?"}
    n_readerror["Mostrar fallo de lectura"]
    n_lots{"¿El producto tiene lotes?"}
    n_nolots[/"Mostrar producto sin lotes"/]
    n_show[/"Mostrar lote, unidad, saldo, disponible y vencimiento"/]
    n_next{"¿Qué desea hacer?"}
    n_other[["Iniciar el caso elegido"]]
    n_endother(["Fin de esta consulta"])
    n_end(["Fin"])
    n_s --> n_input
    n_input --> n_query
    n_query --> n_ok
    n_ok -->|No| n_error
    n_error --> n_query
    n_ok -->|Sí| n_found
    n_found -->|No| n_empty
    n_empty --> n_input
    n_found -->|Sí| n_select
    n_select --> n_details
    n_details --> n_readok
    n_readok -->|No| n_readerror
    n_readerror --> n_details
    n_readok -->|Sí| n_lots
    n_lots -->|No| n_nolots
    n_nolots --> n_next
    n_lots -->|Sí| n_show
    n_show --> n_next
    n_next -->|Buscar| n_input
    n_next -->|Otra operación| n_other
    n_other --> n_endother
    n_next -->|Terminar| n_end
```

- Una búsqueda vacía muestra productos. Nombre: coincidencia parcial; número de lote: coincidencia exacta después de normalizar.
- Distinguir sin recepción, agotado y vencido. El lote vencido puede conservar saldo positivo y mostrar disponible cero.
- Los errores de lectura no significan inventario cero. Consultar no reserva unidades. Desde cualquier búsqueda se puede terminar sin modificar datos.

**Referencia:** BR-02, BR-04, BR-06, BR-11, BR-12, BR-18, BR-22, BR-24.

## UC-05 - Registrar salida

**Actor o ámbito:** Consumidor de inventario.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_search[["UC-04: buscar producto y lotes"]]
    n_exists{"¿Hay un lote seleccionable?"}
    n_missing["Mostrar falta de lote o disponibilidad"]
    n_select[/"Seleccionar lote y consultar disponible"/]
    n_expired{"¿Lote vencido?"}
    n_exp_error["Rechazar ese lote"]
    n_input[/"Ingresar cantidad, quién registra y quién recibe"/]
    n_valid{"¿Datos válidos?"}
    n_error["Corregir campos"]
    n_stock{"¿q no supera lo disponible?"}
    n_short["Mostrar disponible insuficiente"]
    n_confirm{"¿Confirmar salida?"}
    n_cancel(["Fin: salida cancelada"])
    n_cf[["CF-03: confirmar o recuperar ISSUED"]]
    n_result{"¿Resultado?"}
    n_reject["Mostrar causa y revisar la salida"]
    n_recover[["Recuperar el mismo intento"]]
    n_end(["Fin: ISSUED registrado"])
    n_s --> n_search
    n_search --> n_exists
    n_exists -->|No| n_missing
    n_missing --> n_search
    n_exists -->|Sí| n_select
    n_select --> n_expired
    n_expired -->|Sí| n_exp_error
    n_exp_error --> n_search
    n_expired -->|No| n_input
    n_input --> n_valid
    n_valid -->|No| n_error
    n_error --> n_input
    n_valid -->|Sí| n_stock
    n_stock -->|No| n_short
    n_short --> n_input
    n_stock -->|Sí| n_confirm
    n_confirm -->|No| n_cancel
    n_confirm -->|Sí| n_cf
    n_cf --> n_result
    n_result -->|Confirmado| n_end
    n_result -->|Rechazado| n_reject
    n_reject --> n_search
    n_result -->|Pendiente| n_recover
    n_recover --> n_cf
```

- Las comprobaciones de pantalla son previas. CF-03 repite vigencia, cantidad y disponibilidad sobre el estado actual y protege el guardado frente a operaciones simultáneas.
- Si hay 10 unidades y dos personas intentan retirar 8, solo una salida puede confirmarse. La otra recibe rechazo y debe revisar el disponible.
- No hay reserva, salida parcial ni cambio automático de lote. Reenviar una confirmación ya registrada recupera su resultado sin volver a descontar.

**Referencia:** BR-01, BR-03, BR-04, BR-05, BR-06, BR-08, BR-10, BR-18, BR-19, BR-21; CF-03.

## UC-06 - Registrar daño

**Actor o ámbito:** Personal de inventario.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_select[["UC-04: seleccionar lote y consultar saldo"]]
    n_exists{"¿Lote válido?"}
    n_missing["Revisar selección"]
    n_input[/"Ingresar cantidad dañada, razón y quién registra"/]
    n_valid{"¿Datos válidos?"}
    n_error["Corregir campos"]
    n_stock{"¿q no supera el saldo?"}
    n_short["Mostrar saldo insuficiente"]
    n_confirm{"¿Confirmar daño?"}
    n_cancel(["Fin: daño cancelado"])
    n_cf[["CF-03: confirmar o recuperar DAMAGED"]]
    n_result{"¿Resultado?"}
    n_reject["Mostrar causa y revisar la baja"]
    n_recover[["Recuperar el mismo intento"]]
    n_end(["Fin: DAMAGED registrado"])
    n_s --> n_select
    n_select --> n_exists
    n_exists -->|No| n_missing
    n_missing --> n_select
    n_exists -->|Sí| n_input
    n_input --> n_valid
    n_valid -->|No| n_error
    n_error --> n_input
    n_valid -->|Sí| n_stock
    n_stock -->|No| n_short
    n_short --> n_input
    n_stock -->|Sí| n_confirm
    n_confirm -->|No| n_cancel
    n_confirm -->|Sí| n_cf
    n_cf --> n_result
    n_result -->|Confirmado| n_end
    n_result -->|Rechazado| n_reject
    n_reject --> n_select
    n_result -->|Pendiente| n_recover
    n_recover --> n_cf
```

- Este caso compara contra saldo, no contra disponible para salida. Un lote vencido con saldo positivo admite una baja por daño.
- La razón es obligatoria. Solo se dan de baja unidades todavía incluidas en el saldo; no se descuentan de nuevo unidades ya retiradas.
- CF-03 vuelve a comprobar saldo y campos antes de confirmar un único DAMAGED junto con su efecto.

**Referencia:** BR-01, BR-03, BR-04, BR-07, BR-08, BR-10, BR-19, BR-21; CF-03.

## UC-07 - Registrar ajuste

**Actor o ámbito:** Personal de inventario.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_select[/"Seleccionar lote, leer saldo y conservar estado revisado"/]
    n_receipt{"¿Tiene recepción previa?"}
    n_receive[["UC-03: registrar recepción cuando corresponda"]]
    n_end_norec(["Fin de este ajuste"])
    n_input[/"Ingresar dirección, cantidad, razón, persona y referencia"/]
    n_valid{"¿Datos y referencia válidos?"}
    n_error["Corregir datos"]
    n_calc["Calcular saldo propuesto"]
    n_nonneg{"¿Saldo propuesto no negativo?"}
    n_negative["Revisar magnitud o dirección"]
    n_confirm{"¿Confirmar ajuste?"}
    n_cancel(["Fin: ajuste cancelado"])
    n_cf[["CF-03: verificar estado y confirmar ADJUSTED"]]
    n_result{"¿Resultado?"}
    n_stale{"¿Rechazo por cambios en el lote?"}
    n_recount["Recargar y revisar el conteo"]
    n_other_error["Mostrar causa y corregir selección o datos"]
    n_recover[["Recuperar el mismo intento"]]
    n_end(["Fin: ADJUSTED registrado"])
    n_s --> n_select
    n_select --> n_receipt
    n_receipt -->|No| n_receive
    n_receive --> n_end_norec
    n_receipt -->|Sí| n_input
    n_input --> n_valid
    n_valid -->|No| n_error
    n_error --> n_input
    n_valid -->|Sí| n_calc
    n_calc --> n_nonneg
    n_nonneg -->|No| n_negative
    n_negative --> n_input
    n_nonneg -->|Sí| n_confirm
    n_confirm -->|No| n_cancel
    n_confirm -->|Sí| n_cf
    n_cf --> n_result
    n_result -->|Confirmado| n_end
    n_result -->|Rechazado| n_stale
    n_stale -->|Sí| n_recount
    n_recount --> n_select
    n_stale -->|No| n_other_error
    n_other_error --> n_select
    n_result -->|Pendiente| n_recover
    n_recover --> n_cf
```

- La cantidad es la diferencia: para pasar de 50 a 47 se elige disminuir 3. Magnitud cero no es válida; llegar a saldo cero sí puede serlo.
- CF-03 detecta cualquier movimiento posterior al estado revisado, aunque el saldo vuelva al mismo número. Requiere revisar el conteo y una nueva confirmación.
- La referencia de corrección, cuando corresponde, debe existir y pertenecer al lote. El ajuste agrega un movimiento; no edita el original ni cambia el vencimiento.

**Referencia:** BR-03, BR-04, BR-08, BR-09, BR-10, BR-16, BR-19, BR-20, BR-21; CF-03.

## UC-08 - Consultar historial del lote

**Actor o ámbito:** Personal de inventario.

```mermaid
flowchart TD
    n_s(["Inicio"])
    n_select[["UC-04: buscar y seleccionar lote"]]
    n_exists{"¿Existe el lote?"}
    n_missing["Mostrar lote no encontrado"]
    n_read["Leer lote y movimientos de forma coherente"]
    n_ok{"¿Consulta correcta?"}
    n_error["Mostrar fallo de lectura"]
    n_sum["Sumar efectos de movimientos desde cero"]
    n_coherent{"¿La suma coincide con el saldo?"}
    n_inconsistent["Señalar inconsistencia; conservar evidencia"]
    n_end_error(["Fin: requiere investigación"])
    n_has{"¿Hay movimientos?"}
    n_empty[/"Mostrar saldo cero y sin recepción ni movimientos"/]
    n_end_empty(["Fin: historial vacío"])
    n_show[/"Mostrar saldo, disponible, primera recepción y total recibido"/]
    n_history[/"Mostrar movimientos ordenados, personas y razones"/]
    n_end(["Fin: historial consultado"])
    n_s --> n_select
    n_select --> n_exists
    n_exists -->|No| n_missing
    n_missing --> n_select
    n_exists -->|Sí| n_read
    n_read --> n_ok
    n_ok -->|No| n_error
    n_error --> n_read
    n_ok -->|Sí| n_sum
    n_sum --> n_coherent
    n_coherent -->|No| n_inconsistent
    n_inconsistent --> n_end_error
    n_coherent -->|Sí| n_has
    n_has -->|No| n_empty
    n_empty --> n_end_empty
    n_has -->|Sí| n_show
    n_show --> n_history
    n_history --> n_end
```

- Los lotes agotados o vencidos permanecen consultables. La disponibilidad se explica con saldo y vencimiento; el saldo se explica con los movimientos.
- Sin movimientos, la suma es cero. Un fallo de lectura no equivale a historial vacío. Si existe un saldo almacenado inconsistente, se señala antes de mostrarlo como correcto.
- Esta consulta no edita ni elimina datos. Un ajuste no debe ocultar un fallo interno entre saldo e historial.

**Referencia:** BR-04, BR-06, BR-09, BR-10, BR-12, BR-22, BR-24.

## CF-03A - Confirmación y recuperación de resultado

**Actor o ámbito:** Subproceso común de UC-03, UC-05, UC-06 y UC-07.

```mermaid
flowchart TD
    n_s(["Recibir confirmación o reenvío"])
    n_resolve["Conservar datos y resolver identificador de forma exclusiva"]
    n_new{"¿Es un intento nuevo?"}
    n_same{"¿Coinciden los datos del identificador?"}
    n_conflict(["Devolver rechazo por conflicto; no ejecutar"])
    n_execute[["CF-03B: validar y registrar el intento nuevo"]]
    n_retrieve["Consultar el resultado original"]
    n_known{"¿Hay resultado conocido?"}
    n_pending(["Devolver pendiente; conservar el mismo intento"])
    n_return[/"Devolver confirmado o rechazo conocido"/]
    n_end(["Regresar al caso que llamó"])
    n_s --> n_resolve
    n_resolve --> n_new
    n_new -->|Sí| n_execute
    n_execute --> n_known
    n_new -->|No| n_same
    n_same -->|No| n_conflict
    n_same -->|Sí| n_retrieve
    n_retrieve --> n_known
    n_known -->|No| n_pending
    n_known -->|Sí| n_return
    n_return --> n_end
```

- Un identificador repetido con los mismos datos recupera su resultado. No pasa otra vez por CF-03B ni vuelve a validar el stock o el vencimiento actuales.
- En curso, timeout o respuesta perdida no significan rechazo. La recuperación puede quedar pendiente y reanudarse; no se crea un identificador nuevo a ciegas.
- Tras un rechazo conocido, corregir y confirmar constituye un nuevo intento. El resultado recuperado es histórico; cualquier saldo actual se muestra como consulta separada.

**Referencia:** BR-08, BR-19, BR-21, BR-23, BR-24; CF-03.

## CF-03B - Validación final y guardado indivisible

**Actor o ámbito:** Solo para un intento nuevo admitido por CF-03A.

```mermaid
flowchart TD
    n_s(["Intento nuevo"])
    n_read["Proteger lote y leer estado y fecha actuales"]
    n_base{"¿Producto, lote y campos válidos?"}
    n_base_error["Preparar rechazo con causa"]
    n_base_end(["Devolver rechazo sin efectos"])
    n_adjust{"¿Es un ajuste?"}
    n_changed{"¿Hubo movimientos desde la revisión?"}
    n_stale["Exigir revisar el conteo"]
    n_stale_end(["Devolver rechazo sin efectos"])
    n_rules{"¿Cumple las reglas de su movimiento?"}
    n_rule_error["Preparar rechazo específico"]
    n_rule_end(["Devolver rechazo sin efectos"])
    n_save["Confirmar juntos movimiento, efecto y resultado recuperable"]
    n_result{"¿Resultado del guardado?"}
    n_fail["Reversión confirmada; sin efectos"]
    n_fail_end(["Devolver fallo conocido"])
    n_unknown["Resultado aún desconocido"]
    n_unknown_end(["Volver a CF-03A con el mismo intento"])
    n_end(["Devolver confirmado"])
    n_s --> n_read
    n_read --> n_base
    n_base -->|No| n_base_error
    n_base_error --> n_base_end
    n_base -->|Sí| n_adjust
    n_adjust -->|Sí| n_changed
    n_changed -->|Sí| n_stale
    n_stale --> n_stale_end
    n_changed -->|No| n_rules
    n_adjust -->|No| n_rules
    n_rules -->|No| n_rule_error
    n_rule_error --> n_rule_end
    n_rules -->|Sí| n_save
    n_save --> n_result
    n_result -->|Revertido| n_fail
    n_fail --> n_fail_end
    n_result -->|Desconocido| n_unknown
    n_unknown --> n_unknown_end
    n_result -->|Confirmado| n_end
```

- RECEIVED: entero positivo, participantes, datos de lote coherentes; si vencido, nota y aceptación. ISSUED: vigente y q no supera disponible.
- DAMAGED: razón y q no supera saldo. ADJUSTED: recepción previa, dirección, razón, referencia válida y saldo resultante no negativo.
- La protección incluye lectura final, validación y guardado; no la captura humana ni la espera de respuesta. Si no puede conocerse el resultado, se recupera mediante CF-03A. Nunca hay éxito parcial.

**Referencia:** BR-01 a BR-10, BR-13 a BR-16, BR-19 a BR-24; CF-03.

## Comprobación de correspondencia

Los ocho casos conservan sus decisiones de la v2.0. Cada nodo es alcanzable desde su inicio; cada rombo tiene salidas identificadas; los rechazos permiten corregir o terminar. Las rutas de confirmación repetida no ejecutan otra vez CF-03B.

Los escenarios de aceptación que deben recorrerse al implementar siguen siendo los 38 de la v2.0. La revisión de estos dibujos verifica correspondencia y legibilidad; no constituye una prueba del software.
