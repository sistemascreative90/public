# Módulo STK — Contexto técnico de documentos y costo promedio

> Documento de ingeniería de contexto para el sistema ACREA.  
> Describe la estructura, flujo de datos, triggers y procedimientos del módulo de stock.

---

## Tabla de contenidos

1. [Tablas principales](#1-tablas-principales)
2. [Tablas relacionadas clave](#2-tablas-relacionadas-clave)
3. [Tipos de operación (STK_OPERACION)](#3-tipos-de-operación-stk_operacion)
4. [Flujo completo de un documento](#4-flujo-completo-de-un-documento)
5. [Triggers](#5-triggers)
6. [Procedimientos almacenados](#6-procedimientos-almacenados)
7. [Cálculo del costo promedio](#7-cálculo-del-costo-promedio)
8. [Tablas auxiliares de stock](#8-tablas-auxiliares-de-stock)
9. [Consultas de referencia](#9-consultas-de-referencia)
10. [Reglas de negocio importantes](#10-reglas-de-negocio-importantes)

---

## 1. Tablas principales

### `STK_DOCUMENTO` — Cabecera del documento

Representa cualquier movimiento de stock: compra, venta, transferencia, devolución, etc.

| Columna | Tipo | Descripción |
|---|---|---|
| `DOCU_CLAVE` | NUMBER(14) PK | Clave única del documento |
| `DOCU_EMPR` | NUMBER(2) | Empresa |
| `DOCU_CODIGO_OPER` | NUMBER(2) FK | Tipo de operación → `STK_OPERACION` |
| `DOCU_NRO_DOC` | NUMBER(13) | Número de documento (factura, remisión, etc.) |
| `DOCU_SUC_ORIG` | NUMBER(2) | Sucursal origen |
| `DOCU_DEP_ORIG` | NUMBER(2) | Depósito origen |
| `DOCU_SUC_DEST` | NUMBER(2) | Sucursal destino (transferencias) |
| `DOCU_DEP_DEST` | NUMBER(2) | Depósito destino (transferencias) |
| `DOCU_MON` | NUMBER(2) FK | Moneda → `GEN_MONEDA` |
| `DOCU_PROV` | NUMBER(6) FK | Proveedor → `FIN_PROVEEDOR` (compras) |
| `DOCU_CLI` | NUMBER(8) FK | Cliente → `FIN_CLIENTE` (ventas) |
| `DOCU_CLI_NOM` | VARCHAR2(40) | Nombre cliente (desnormalizado) |
| `DOCU_CLI_RUC` | VARCHAR2(11) | RUC cliente (desnormalizado) |
| `DOCU_LEGAJO` | NUMBER(6) FK | Vendedor → `FAC_VENDEDOR` |
| `DOCU_FEC_EMIS` | DATE | Fecha de emisión del documento |
| `DOCU_TIPO_MOV` | NUMBER(2) FK | Tipo de movimiento → `GEN_TIPO_MOV` |
| `DOCU_GRAV_NETO_LOC` | NUMBER(14,4) | Importe gravado neto en moneda local |
| `DOCU_GRAV_NETO_MON` | NUMBER(14,4) | Importe gravado neto en moneda extranjera |
| `DOCU_EXEN_NETO_LOC` | NUMBER(14,4) | Importe exento neto en moneda local |
| `DOCU_EXEN_NETO_MON` | NUMBER(14,4) | Importe exento neto en moneda extranjera |
| `DOCU_GRAV_BRUTO_LOC` | NUMBER(14,4) | Importe gravado bruto en moneda local |
| `DOCU_IVA_LOC` | NUMBER(14,4) | IVA en moneda local |
| `DOCU_IVA_MON` | NUMBER(14,4) | IVA en moneda extranjera |
| `DOCU_TASA_US` | NUMBER(10,4) | Cotización del dólar al momento del documento |
| `DOCU_IND_CUOTA` | VARCHAR2(1) | Indicador de venta en cuotas |
| `DOCU_IND_CONSIGNACION` | VARCHAR2(1) | Indicador de consignación |
| `DOCU_OBS` | VARCHAR2(40) | Observaciones |
| `DOCU_CLAVE_PADRE` | NUMBER(14) FK | Documento padre (devoluciones/anulaciones) — ON DELETE CASCADE |
| `DOCU_LOGIN` | VARCHAR2(8) | Usuario que grabó |
| `DOCU_FEC_GRAB` | DATE | Fecha y hora de grabación |
| `DOCU_SIST` | VARCHAR2(3) | Sistema que originó el documento |
| `DOCU_OPERADOR` | VARCHAR2(1) | Tipo de operador (default `'2'`) |
| `DOCU_NRO_PED` | NUMBER(8) FK | Pedido relacionado → `STK_PEDIDO` |
| `DOCU_LEGAJO_EMPL` | NUMBER(6) FK | Empleado → `PER_EMPLEADO` |
| `DOCU_IMP_DTO_LOC` | NUMBER(14,2) | Importe descuento en moneda local |
| `DOCU_PORC_DTO` | NUMBER(5,2) | Porcentaje de descuento |
| `DOCU_PORC_REC_GF` | NUMBER(5,2) | Porcentaje recargo gastos financieros |
| `DOCU_RUBRO` | NUMBER(2) | Rubro del documento |
| `DOCU_NRO_OT` | NUMBER(13) | Número de orden de taller |
| `DOCU_FORMA_PAGO` | NUMBER | Forma de pago |
| `DOCU_IND_PAGADO` | VARCHAR2(1) | Indicador de pagado |

**Índice:** `XAKSTK_DOC_FEC_OPER` sobre (`DOCU_CLAVE`, `DOCU_CODIGO_OPER`, `DOCU_FEC_EMIS`)

---

### `STK_DOCUMENTO_DET` — Detalle del documento

Cada ítem (artículo) que compone un documento.

| Columna | Tipo | Descripción |
|---|---|---|
| `DETA_CLAVE_DOC` | NUMBER(14) PK/FK | Clave del documento → `STK_DOCUMENTO` |
| `DETA_NRO_ITEM` | NUMBER(14) PK | Número de ítem dentro del documento |
| `DETA_ART` | NUMBER(14) FK | Artículo → `STK_ARTICULO` |
| `DETA_EMPR` | NUMBER(2) | Empresa |
| `DETA_NRO_REM` | NUMBER(18) | Nro de remisión asociada (si aplica) |
| `DETA_CANT` | NUMBER(15,6) | Cantidad del movimiento |
| `DETA_IMP_NETO_LOC` | NUMBER(14,4) | Importe neto en moneda local (precio × cant, sin IVA) |
| `DETA_IMP_NETO_MON` | NUMBER(14,4) | Importe neto en moneda extranjera |
| `DETA_IMPU` | VARCHAR2(1) | Indicador de impuesto |
| `DETA_IVA_LOC` | NUMBER(14,4) | IVA en moneda local |
| `DETA_IVA_MON` | NUMBER(14,4) | IVA en moneda extranjera |
| `DETA_PORC_DTO` | NUMBER(5,2) | Porcentaje de descuento por ítem |
| `DETA_IMP_BRUTO_LOC` | NUMBER(14,4) | Importe bruto en moneda local (con IVA) |
| `DETA_IMP_BRUTO_MON` | NUMBER(14,4) | Importe bruto en moneda extranjera |
| `DETA_CANT_REM` | NUMBER(15,6) | Cantidad de remisión |
| `DETA_PERIODO` | NUMBER(6) FK | Período contable → `STK_PERIODO` (asignado por trigger) |
| `DETA_IMP_DTO_BON_LOC` | NUMBER(14,2) | Importe descuento/bonificación local |
| `DETA_IND_ART_BONIF` | VARCHAR2(1) | `'S'` = artículo es bonificación/regalo |
| `DETA_IND_MODIFICADO` | VARCHAR2(1) | Indica si el ítem fue modificado |
| `DETA_PRECIO_VTA` | NUMBER(14,4) | Precio de venta unitario |
| `DETA_CCOSTO` | NUMBER(10) | Centro de costo |
| `DETA_LOT` | VARCHAR2(100) | Código de lote (opcional) |
| `DETA_FEC_VTO` | DATE | Fecha de vencimiento del lote |
| `DETA_CANT_LOT` | NUMBER(15,6) | Cantidad del lote (se sincroniza con `DETA_CANT`) |

**PK:** (`DETA_CLAVE_DOC`, `DETA_NRO_ITEM`)

> **Nota:** `DETA_PERIODO` es asignado automáticamente por el trigger `STK_DOC_DET_BE_ID` al insertar, buscando el período cuya fecha inicio/fin contiene `DOCU_FEC_EMIS`.

---

## 2. Tablas relacionadas clave

| Tabla | Relación | Propósito |
|---|---|---|
| `STK_OPERACION` | `DOCU_CODIGO_OPER` | Define el tipo de operación y si es entrada o salida |
| `STK_ARTICULO` | `DETA_ART` | Maestro de artículos |
| `STK_DEPOSITO` | `DOCU_SUC/DEP_ORIG/DEST` | Depósitos de origen y destino |
| `STK_ARTICULO_DEPOSITO` | — | Stock actual por artículo/empresa/sucursal/depósito |
| `STK_ARTICULO_DEPOSITO_LOT` | — | Stock por artículo/lote |
| `STK_ART_EMPR_PERI` | — | **Costo promedio y existencias por período** |
| `STK_PERIODO` | `DETA_PERIODO` | Períodos contables (mensual) |
| `STK_CONFIGURACION` | — | Período activo y siguiente por empresa |
| `STK_AUD_DOCUMENTO` | — | Auditoría de documentos eliminados |
| `STK_AUD_DOCUMENTO_DET` | — | Auditoría de ítems eliminados |
| `STK_ESTADISTICA` | — | Estadísticas de ventas/producción por mes |
| `STK_INVENTARIO_DET` | — | Detalle de tomas de inventario físico |
| `STK_REMISION_DET` | — | Control de cantidades facturadas vs remitidas |
| `FIN_DOCUMENTO` | — | Vínculo con presupuestos financieros |
| `TAL_ORDEN_COMPRA_DET` | — | Control de cantidades compradas vs ordenadas |

---

## 3. Tipos de operación (STK_OPERACION)

La columna `OPER_ENT_SAL` define si el movimiento suma (`'E'` = Entrada) o resta (`'S'` = Salida) del stock.

| OPER_DESC | Tipo | Descripción |
|---|---|---|
| `COMPRA` | E | Compra a proveedor |
| `DEV_COM` | S | Devolución de compra |
| `VENTA` | S | Venta a cliente |
| `VENTA FUTURA` | S | Venta anticipada |
| `DEV_VENTA` | E | Devolución de venta |
| `DEV_VENTA FUTURA` | E | Devolución de venta anticipada |
| `ENT_PROD` | E | Entrada por producción |
| `SAL_PROD` | S | Salida por producción |
| `DEV_PROD` | E | Devolución por producción |
| `ENT_KIT` | E | Entrada por armado de kit |
| `SAL_KIT` | S | Salida por armado de kit |
| `ENT_ARM_DES` | E | Entrada por armado/desarmado |
| `SAL_ARM_DES` | S | Salida por armado/desarmado |
| `TRAN_ENT` | E | Transferencia entrada (depósito destino) |
| `TRAN_SAL` | S | Transferencia salida (depósito origen) |
| `REMISION` | — | Remisión de mercadería |
| `PERDIDA` | S | Pérdida/merma |
| `CONSUMO` | S | Consumo interno |
| `DIF_MAS` | E | Diferencia de inventario positiva |
| `DIF_MEN` | S | Diferencia de inventario negativa |

> **Regla:** Las operaciones `COMPRA`, `DEV_COM`, `ENT_PROD`, `DEV_PROD`, `ENT_KIT`, `ENT_ARM_DES` **requieren** `DOCU_TASA_US` (cotización) distinta de cero, porque afectan el costo promedio en moneda extranjera.

---

## 4. Flujo completo de un documento

### Al insertar un ítem (`STK_DOCUMENTO_DET` INSERT)

```
INSERT INTO STK_DOCUMENTO_DET
        ↓
TRIGGER: STK_DOC_DET_BE_ID (BEFORE INSERT)
        ↓
    ├── STK_ACT_ART_DEP('I', art, doc, cant)
    │       └── STK_ACT_ART_DEP_IU(empr, suc, dep, art, cant_ent, cant_sal)
    │               → INSERT o UPDATE en STK_ARTICULO_DEPOSITO
    │                 (ARDE_CANT_ACT, ARDE_CANT_ENT, ARDE_CANT_SAL)
    │
    ├── STK_ACT_ART_DEP_LOT('I', ...) [solo si tiene lote y fecha de vto]
    │               → INSERT o UPDATE en STK_ARTICULO_DEPOSITO_LOT
    │
    ├── STK_ACT_ART_EMP('I', art, doc, cant, imp_neto_loc)
    │               → UPDATE STK_ART_EMPR_PERI
    │                 (actualiza cantidades + RECALCULA COSTO PROMEDIO)
    │
    ├── STK_ACT_REM_DET('I', ...) [solo si tiene remisión]
    │               → UPDATE STK_REMISION_DET (DETR_CANT_FACT)
    │
    └── SELECT INTO :N.DETA_PERIODO [asigna período automáticamente]

TRIGGER: STK_DOC_DET_TAL_PRES_REP_BE_ID (BEFORE INSERT)
        → Actualiza presupuesto si el documento tiene cargo de presupuesto

TRIGGER: STK_DOC_DET_TAL_ORD_COM_BE_ID (BEFORE INSERT)
        → Actualiza TAL_ORDEN_COMPRA_DET si es oper. 1 o 2
```

### Al eliminar un ítem (`STK_DOCUMENTO_DET` DELETE)

```
DELETE FROM STK_DOCUMENTO_DET
        ↓
TRIGGER: STK_DOC_DET_BE_ID (BEFORE DELETE)
        ↓
    ├── INSERT INTO STK_AUD_DOCUMENTO_DET [auditoría]
    ├── STK_ACT_ART_DEP('D', ...) → revierte stock en STK_ARTICULO_DEPOSITO
    ├── STK_ACT_ART_DEP_LOT('D', ...) → revierte stock lote
    ├── STK_ACT_ART_EMP('D', ...) → revierte costo promedio
    └── STK_ACT_REM_DET('D', ...) → revierte cantidad facturada
```

### Al eliminar el documento cabecera (`STK_DOCUMENTO` DELETE)

```
DELETE FROM STK_DOCUMENTO
        ↓
TRIGGER: STK_DOC_BE_ID (BEFORE DELETE)
        → INSERT INTO STK_AUD_DOCUMENTO [auditoría del encabezado]
        
CONSTRAINT ON DELETE CASCADE
        → Elimina automáticamente todos los STK_DOCUMENTO_DET hijos
        → Cada eliminación dispara STK_DOC_DET_BE_ID (ver arriba)
```

---

## 5. Triggers

### `STK_DOC_DET_BE_ID`
- **Tabla:** `STK_DOCUMENTO_DET`
- **Evento:** `BEFORE INSERT OR DELETE` — FOR EACH ROW
- **Responsabilidades en INSERT:**
  - Llama `STK_ACT_ART_DEP` → actualiza existencias en depósito
  - Llama `STK_ACT_ART_DEP_LOT` → actualiza existencias por lote (si aplica)
  - Llama `STK_ACT_ART_EMP` → actualiza `STK_ART_EMPR_PERI` y recalcula costo promedio
  - Llama `STK_ACT_REM_DET` → actualiza remisión (si aplica)
  - Asigna `DETA_PERIODO` automáticamente desde `STK_PERIODO`
- **Responsabilidades en DELETE:**
  - Graba auditoría en `STK_AUD_DOCUMENTO_DET`
  - Revierte todas las actualizaciones anteriores con signo inverso

### `STK_DOC_DET_TAL_PRES_REP_BE_ID`
- **Tabla:** `STK_DOCUMENTO_DET`
- **Evento:** `BEFORE INSERT OR DELETE` — FOR EACH ROW
- **Responsabilidad:** Actualiza el presupuesto financiero (`FIN_DOCUMENTO`) si el documento tiene cargo de presupuesto

### `STK_DOC_DET_TAL_ORD_COM_BE_ID`
- **Tabla:** `STK_DOCUMENTO_DET`
- **Evento:** `BEFORE INSERT OR DELETE` — FOR EACH ROW
- **Responsabilidad:** Actualiza `TAL_ORDEN_COMPRA_DET.DEOC_CANT_COMPRADA` si el documento corresponde a una orden de compra (oper. 1 = suma, oper. 2 = resta)

### `STK_DOC_BE_ID`
- **Tabla:** `STK_DOCUMENTO`
- **Evento:** `BEFORE INSERT OR DELETE` — FOR EACH ROW
- **Responsabilidad en DELETE:** Graba auditoría en `STK_AUD_DOCUMENTO`
- **Nota:** Las secciones de INSERT están comentadas (inactivas)

---

## 6. Procedimientos almacenados

### `STK_ACT_ART_DEP(V_TIPO_OPER, V_ART, V_CLAVE_DOC, V_CANT)`
Actualiza el stock en `STK_ARTICULO_DEPOSITO`.

- Lee el documento para obtener empresa, sucursal, depósito y tipo de operación
- Determina si es entrada o salida según `OPER_ENT_SAL`
- Si `V_TIPO_OPER = 'I'` (insertar): suma entrada o salida
- Si `V_TIPO_OPER = 'D'` (eliminar): invierte los signos
- Delega en `STK_ACT_ART_DEP_IU` para el INSERT/UPDATE real

### `STK_ACT_ART_DEP_IU(V_EMPR, V_SUC, V_DEP, V_ART, V_CANT_ENT, V_CANT_SAL)`
Hace el INSERT o UPDATE efectivo en `STK_ARTICULO_DEPOSITO`.

```
ARDE_CANT_ACT = ARDE_CANT_ACT + V_CANT_ENT - V_CANT_SAL
ARDE_CANT_ENT = ARDE_CANT_ENT + V_CANT_ENT
ARDE_CANT_SAL = ARDE_CANT_SAL + V_CANT_SAL
```
Si no existe el registro, hace INSERT con `ARDE_CANT_INI = 0`.

### `STK_ACT_ART_DEP_LOT(V_TIPO_OPER, V_ART, V_CLAVE_DOC, V_CANT, V_LOT, V_FVTO)`
Igual a `STK_ACT_ART_DEP` pero sobre `STK_ARTICULO_DEPOSITO_LOT`, filtrando por lote y fecha de vencimiento.

### `STK_ACT_ART_EMP(V_TIPO_OPER, V_ART, V_CLAVE_DOC, V_CANT, V_IMP_NETO_LOC)`
**Procedimiento central del costo promedio.** Ver sección 7.

### `STK_ACT_ART_EMPR_PERI`
Procedimiento de apertura de período. Crea los registros del período siguiente en `STK_ART_EMPR_PERI` copiando el stock y costo promedio actual como valores iniciales.

### `STK_ACT_ART_DEP_ACT(V_EMPRESA, V_FEC_INI, V_FEC_FIN)`
Recalcula `STK_ARTICULO_DEPOSITO` para un rango de fechas. Útil para sincronización o corrección.

### `STK_ACT_ART_DEP_HIST(V_EMPRESA, V_PERIODO, V_FEC_INI, V_FEC_FIN)`
Genera el histórico en `STK_ARTICULO_DEPOSITO_HIST` para el cierre de período.

### `STK_ACT_REM_DET(V_TIPO_OPER, V_EMPR, V_NRO_REM, V_ART, V_CANT)`
Actualiza `STK_REMISION_DET.DETR_CANT_FACT`. Valida que exista saldo en la remisión antes de permitir la facturación.

### `STK_ACT_STK_EST(V_TIPO_OPER, V_ART, V_CLAVE_DOC, V_CANT)`
Actualiza estadísticas de venta/producción en `STK_ESTADISTICA` por mes.  
**Actualmente comentado en el trigger** (se desactivó por performance).

### `STK_ACT_TOMA_INVENTARIO(V_TIPO_OPER, V_EMPR, V_SUC, V_DEP, V_FEC, V_ART, V_CANT, V_ENT_SAL)`
Actualiza `STK_INVENTARIO_DET.INVD_CANT_ACT` para tomas de inventario posteriores a la fecha del movimiento.

### `STK_ACT_CANC_PED(Clave, Empresa, Sucursal, Deposito, Tipo_ped, NroItem, Articulo, cantidad)`
Cancela pedidos pendientes en `STK_PED_SUC_IMP_DET` insertando registros en `STK_PED_CANC`.

### `STK_ACT_ARTICULO(V_CODIGO_OPER, V_ART, V_CLAVE)`
Actualiza `STK_ARTICULO.ART_SITUACION` (1=Vendido, 3=Disponible).  
**Actualmente comentado en el trigger** (se desactivó por performance).

---

## 7. Cálculo del costo promedio

El costo promedio se gestiona en `STK_ART_EMPR_PERI` mediante el procedimiento `STK_ACT_ART_EMP`.

### Tabla `STK_ART_EMPR_PERI` (columnas relevantes)

| Columna | Descripción |
|---|---|
| `AEP_PERIODO` | Período contable |
| `AEP_EMPR` | Empresa |
| `AEP_ART` | Artículo |
| `AEP_TOT_EXIST` | Existencia total actual |
| `AEP_EXIST_INI` | Existencia inicial del período |
| `AEP_COSTO_PROM_LOC` | **Costo promedio en moneda local** |
| `AEP_COSTO_PROM_MON` | **Costo promedio en moneda extranjera (U$S)** |
| `AEP_COSTO_PROM_INI_LOC` | Costo promedio inicial (arrastrado del período anterior) |
| `AEP_COSTO_PROM_INI_MON` | Costo promedio inicial en moneda extranjera |
| `AEP_COSTO_INI_LOC` | Valor total del stock inicial en moneda local |
| `AEP_EC_COMPRA` | Cant. entrada por compra |
| `AEP_EL_COMPRA` | Importe entrada compra local |
| `AEP_EM_COMPRA` | Importe entrada compra moneda |
| `AEP_SC_DEV_COMPRA` | Cant. salida por devolución de compra |
| `AEP_SC_VTA` | Cant. salida por venta |
| `AEP_EC_DEV_VTA` | Cant. entrada por devolución de venta |
| `AEP_EC_PROD` | Cant. entrada por producción |
| `AEP_SC_PROD` | Cant. salida por producción |
| *(y más columnas por cada tipo de operación)* | |

### Fórmula del costo promedio ponderado

```
TOT_CANT = Exist_Ini + EC_COMPRA + EC_PROD + EC_DEV_PROD + EC_KIT + EC_ARM_DES
                     - SC_DEV_COMPRA

TOT_IMP_LOC = Costo_Ini_Loc + EL_COMPRA + EL_PROD + EL_DEV_PROD + EL_KIT + EL_ARM_DES
                             - SL_DEV_COMPRA

COSTO_PROM_LOC = TOT_IMP_LOC / TOT_CANT
COSTO_PROM_MON = TOT_IMP_MON / TOT_CANT
```

> **Regla:** Si `TOT_CANT <= 0` o `TOT_IMP <= 0`, el sistema conserva el último costo promedio válido (no lo pone en cero).

### Alcance del recálculo

- Solo se recalcula si el documento pertenece al **período activo** (`CONF_PERIODO_ACT`) o al **período siguiente** (`CONF_PERIODO_SGTE`).
- Documentos de períodos anteriores (cerrados) **no modifican** el costo promedio.
- Al recalcular el período activo, también se actualiza el **período siguiente** propagando el nuevo costo como costo inicial.

### Operaciones que afectan el costo promedio

| Operación | Efecto |
|---|---|
| COMPRA | Recalcula promedio (suma cant + importe) |
| DEV_COM | Reduce cant + importe (resta de numerador y denominador) |
| ENT_PROD | Recalcula promedio |
| DEV_PROD | Recalcula promedio |
| ENT_KIT | Recalcula promedio |
| ENT_ARM_DES | Recalcula promedio |
| VENTA, SAL_PROD, SAL_KIT, PERDIDA, CONSUMO, etc. | Solo afecta existencias, **NO modifica el costo promedio** |

### Apertura de período

El procedimiento `STK_ACT_ART_EMPR_PERI` crea los registros del período siguiente tomando:

```
AEP_EXIST_INI  = AEP_TOT_EXIST  (del período actual)
AEP_COSTO_PROM_INI_LOC = AEP_COSTO_PROM_LOC (del período actual)
AEP_COSTO_INI_LOC      = AEP_TOT_EXIST * AEP_COSTO_PROM_LOC
```

---

## 8. Tablas auxiliares de stock

### `STK_ARTICULO_DEPOSITO`
Stock físico por artículo/empresa/sucursal/depósito.

| Columna | Descripción |
|---|---|
| `ARDE_CANT_INI` | Cantidad inicial del período |
| `ARDE_CANT_ACT` | Cantidad actual (= INI + ENT - SAL) |
| `ARDE_CANT_ENT` | Total de entradas del período |
| `ARDE_CANT_SAL` | Total de salidas del período |
| `ARDE_CANT_INV` | Cantidad según último inventario |

### `STK_ARTICULO_DEPOSITO_LOT`
Igual pero desagregado por lote y fecha de vencimiento.

### `STK_ARTICULO_DEPOSITO_HIST`
Histórico por período cerrado. Se genera con `STK_ACT_ART_DEP_HIST`.

---

## 9. Consultas de referencia

### Ver existencia y costo promedio actual

```sql
SELECT aep.aep_art,
       art.art_desc,
       aep.aep_tot_exist              AS existencia_actual,
       aep.aep_costo_prom_loc         AS costo_prom_gs,
       aep.aep_costo_prom_mon         AS costo_prom_usd,
       aep.aep_tot_exist * aep.aep_costo_prom_loc AS valor_mercaderia_gs,
       aep.aep_tot_exist * aep.aep_costo_prom_mon AS valor_mercaderia_usd
  FROM stk_art_empr_peri aep
  JOIN stk_articulo art ON art.art_codigo = aep.aep_art
  JOIN stk_configuracion conf ON conf.conf_empr = aep.aep_empr
                              AND conf.conf_periodo_act = aep.aep_periodo
 WHERE aep.aep_empr = :p_empr
 ORDER BY art.art_desc;
```

### Ver movimientos de un artículo con tipo de operación

```sql
SELECT d.docu_fec_emis,
       o.oper_desc,
       o.oper_ent_sal,
       dt.deta_cant,
       dt.deta_imp_neto_loc,
       d.docu_tasa_us,
       d.docu_nro_doc
  FROM stk_documento d
  JOIN stk_documento_det dt ON dt.deta_clave_doc = d.docu_clave
  JOIN stk_operacion o      ON o.oper_codigo = d.docu_codigo_oper
 WHERE dt.deta_art = :p_art
   AND d.docu_empr = :p_empr
 ORDER BY d.docu_fec_emis, d.docu_clave;
```

### Ver stock por depósito

```sql
SELECT ard.arde_empr,
       ard.arde_suc,
       ard.arde_dep,
       art.art_desc,
       ard.arde_cant_ini,
       ard.arde_cant_ent,
       ard.arde_cant_sal,
       ard.arde_cant_act
  FROM stk_articulo_deposito ard
  JOIN stk_articulo art ON art.art_codigo = ard.arde_art
 WHERE ard.arde_empr = :p_empr
   AND ard.arde_art  = :p_art
 ORDER BY ard.arde_suc, ard.arde_dep;
```

### Ver documentos con sus ítems

```sql
SELECT d.docu_clave,
       d.docu_nro_doc,
       d.docu_fec_emis,
       o.oper_desc,
       dt.deta_nro_item,
       art.art_desc,
       dt.deta_cant,
       dt.deta_imp_neto_loc,
       dt.deta_periodo
  FROM stk_documento d
  JOIN stk_documento_det dt ON dt.deta_clave_doc = d.docu_clave
  JOIN stk_operacion o      ON o.oper_codigo = d.docu_codigo_oper
  JOIN stk_articulo art     ON art.art_codigo = dt.deta_art
 WHERE d.docu_empr = :p_empr
   AND d.docu_codigo_oper = :p_oper
 ORDER BY d.docu_fec_emis, d.docu_clave, dt.deta_nro_item;
```

---

## 10. Reglas de negocio importantes

1. **Anulación de documentos:** Se hace eliminando el `STK_DOCUMENTO`. El cascade elimina los detalles, y cada detail dispara el trigger que revierte stock y costo. El encabezado queda en `STK_AUD_DOCUMENTO` y los ítems en `STK_AUD_DOCUMENTO_DET`.

2. **DOCU_CLAVE_PADRE:** Se usa para documentos relacionados (ej. devolución que referencia a la venta original). El `ON DELETE CASCADE` significa que si se elimina el padre, los hijos también se eliminan.

3. **Períodos cerrados:** Si el documento no pertenece al período activo ni al siguiente, `STK_ACT_ART_EMP` hace `RAISE SALIR` (exit silencioso). El stock en `STK_ARTICULO_DEPOSITO` sí se actualiza siempre.

4. **Cotización obligatoria:** Las operaciones de entrada con valor (COMPRA, ENT_PROD, etc.) requieren `DOCU_TASA_US > 0`. Las ventas no lo requieren.

5. **Lotes:** Son opcionales por artículo. Solo se actualiza `STK_ARTICULO_DEPOSITO_LOT` si el ítem tiene `DETA_LOT IS NOT NULL AND DETA_FEC_VTO IS NOT NULL`.

6. **Período asignado automáticamente:** `DETA_PERIODO` nunca se ingresa manualmente; el trigger busca en `STK_PERIODO` el período que contiene la fecha de emisión del documento.

7. **Costo promedio y ventas:** Las salidas (ventas, consumos, pérdidas) **NO modifican** el costo promedio. Solo las entradas con precio lo modifican.

8. **Procedimientos comentados:** `STK_ACT_ARTICULO` y `STK_ACT_STK_EST` están comentados en el trigger por razones de performance. Si se necesitan, deben llamarse externamente.

9. **Consignación:** `DOCU_IND_CONSIGNACION = 'S'` puede indicar mercadería en consignación. La lógica específica depende del sistema que genere el documento.

10. **Multi-sucursal:** `GEN_PACK_SUC.GEN_CONF_IND_SUC = 'S'` hace que ciertos procedimientos omitan su lógica (la sucursal tiene su propio control).


