# Paquete PL/SQL `X` — Manual de uso

> Utilidad de **debug y trazabilidad** para Oracle APEX y PL/SQL.  
> Escribe registros en la tabla `LOG_DEBUG_COMPARTIDO` usando transacciones autónomas, por lo que **no interfiere con el commit/rollback del proceso llamador**.

---

## Tabla de contenidos

1. [Requisitos](#1-requisitos)
2. [Estructura de la tabla de log](#2-estructura-de-la-tabla-de-log)
3. [Referencia de la API](#3-referencia-de-la-api)
   - [I — Iniciar sesión de log](#i--iniciar-sesión-de-log)
   - [E — Escribir etiqueta/valor](#e--escribir-etiquetavalor)
   - [R — Registrar ítem de APEX](#r--registrar-ítem-de-apex)
   - [S — Escribir CLOB](#s--escribir-clob)
   - [A — Volcar todos los ítems de la página actual](#a--volcar-todos-los-ítems-de-la-página-actual)
   - [B — Borrar todo el log](#b--borrar-todo-el-log)
   - [L — Activar/desactivar separador visual](#l--activardesactivar-separador-visual)
   - [H — Activar/desactivar columna de hora](#h--activardesactivar-columna-de-hora)
4. [Flujo típico de uso](#4-flujo-típico-de-uso)
5. [Comportamiento fuera de APEX](#5-comportamiento-fuera-de-apex)
6. [Notas de implementación](#6-notas-de-implementación)
7. [Consultar el log](#7-consultar-el-log)

---

## 1. Requisitos

| Elemento | Descripción |
|---|---|
| Tabla | `LOG_DEBUG_COMPARTIDO` (ver sección 2) |
| Entorno | Oracle APEX (opcional) o PL/SQL puro |
| Privilegios | `INSERT`, `DELETE`, `UPDATE` sobre la tabla de log |

---

## 2. Estructura de la tabla de log

```sql
CREATE TABLE LOG_DEBUG_COMPARTIDO (
    sesion_id  VARCHAR2(100),
    evento     VARCHAR2(100),
    linea      NUMBER,
    etiqueta   VARCHAR2(200),
    valor      VARCHAR2(4000),
    fecha_log  TIMESTAMP
);
```

> **Nota:** El paquete reserva dos valores especiales de `etiqueta`:  
> `__SEPARADOR__` y `__HORA__`. No los uses como etiquetas propias.

---

## 3. Referencia de la API

### `I` — Iniciar sesión de log

```sql
X.I(p_evento IN VARCHAR2);
```

**Debe llamarse siempre primero**, antes de cualquier `E`, `R` o `S`.  
Registra el nombre del evento/proceso y obtiene el ID de sesión (APEX o `'999'` si se corre fuera de APEX).  
Resetea el contador de líneas a `0`.

**Parámetros**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `p_evento` | `VARCHAR2` | Nombre del evento o proceso que se está trazando |

**Ejemplo**

```sql
X.I('PROCESO_FACTURACION');
```

---

### `E` — Escribir etiqueta/valor

```sql
X.E(p_etiqueta IN VARCHAR2, p_valor IN VARCHAR2);
```

Inserta una fila en el log con la etiqueta y el valor indicados.  
Los saltos de línea (`CHR(10)`, `CHR(13)`) se eliminan automáticamente del texto.

**Parámetros**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `p_etiqueta` | `VARCHAR2` | Nombre descriptivo del dato |
| `p_valor` | `VARCHAR2` | Valor a registrar (máx. 4000 caracteres) |

**Ejemplo**

```sql
X.E('v_cliente_id', TO_CHAR(v_cliente_id));
X.E('estado',       v_estado);
X.E('resultado',    'OK');
```

---

### `R` — Registrar ítem de APEX

```sql
X.R(p_item_name IN VARCHAR2, p_valor IN VARCHAR2 DEFAULT NULL);
```

Registra el valor de un ítem de sesión de APEX.  
- Si `p_valor` es `NULL`, lee el estado actual del ítem con `APEX_UTIL.GET_SESSION_STATE`.  
- Si se pasa `p_valor`, lo usa directamente (sin consultar APEX).

**Parámetros**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `p_item_name` | `VARCHAR2` | Nombre del ítem APEX (ej. `P1_CLIENTE`) |
| `p_valor` | `VARCHAR2` | *(Opcional)* Valor explícito a registrar |

**Ejemplo**

```sql
-- Leer desde la sesión APEX
X.R('P1_CLIENTE_ID');
X.R('P1_FECHA_INICIO');

-- Pasar valor explícito
X.R('P1_ESTADO', 'PROCESADO');
```

---

### `S` — Escribir CLOB

```sql
X.S(p_etiqueta IN VARCHAR2, p_texto IN CLOB);
```

Registra el contenido completo de un `CLOB`, dividiéndolo automáticamente en trozos de **3 900 caracteres** para respetar el límite de la columna `valor`.  
Útil para loguear XMLs, JSONs, SQLs dinámicos o cualquier texto largo.

**Parámetros**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `p_etiqueta` | `VARCHAR2` | Nombre descriptivo del dato |
| `p_texto` | `CLOB` | Texto largo a registrar |

**Ejemplo**

```sql
DECLARE
    v_xml CLOB;
BEGIN
    X.I('GENERAR_XML');
    -- ... lógica que construye v_xml ...
    X.S('xml_generado', v_xml);
END;
```

---

### `A` — Volcar todos los ítems de la página actual

```sql
X.A(p_evento IN VARCHAR2);
```

Registra **todos los ítems de la página APEX activa** y sus valores de sesión.  
Itera sobre `APEX_APPLICATION_PAGE_ITEMS` filtrando por `application_id` y `page_id` actuales.  
Si se ejecuta fuera de APEX (sin `APP_SESSION`), no hace nada.

**Parámetros**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `p_evento` | `VARCHAR2` | Nombre del evento bajo el cual se agrupan los ítems |

**Ejemplo**

```sql
-- Snapshot de todos los ítems de la página al inicio del proceso
X.A('SNAPSHOT_INICIO');
```

---

### `B` — Borrar todo el log

```sql
X.B;
```

Elimina **todas las filas** de `LOG_DEBUG_COMPARTIDO` (de todas las sesiones).  
Usar con precaución en ambientes con múltiples usuarios simultáneos.

**Ejemplo**

```sql
-- Limpiar antes de empezar una nueva sesión de debugging
X.B;
X.I('MI_PROCESO');
```

---

### `L` — Activar/desactivar separador visual

```sql
X.L(V_VALOR IN VARCHAR2 DEFAULT NULL);
```

Controla si se muestra una línea separadora (guiones) en la visualización del log para la sesión actual.  
- Pasar un valor activa el separador.  
- Pasar `NULL` (o no pasar nada) lo desactiva/elimina.

**Ejemplo**

```sql
X.L('S');   -- activa separador
X.L();      -- desactiva separador
```

---

### `H` — Activar/desactivar columna de hora

```sql
X.H(V_VALOR IN VARCHAR2 DEFAULT NULL);
```

Controla si se muestra la columna de hora en la visualización del log para la sesión actual.  
- Pasar un valor la activa.  
- Pasar `NULL` (o no pasar nada) la oculta.

**Ejemplo**

```sql
X.H('S');   -- activa columna de hora
X.H();      -- oculta columna de hora
```

---

## 4. Flujo típico de uso

```sql
-- 1. Limpiar log anterior (opcional)
X.B;

-- 2. Iniciar el evento (SIEMPRE antes de E/R/S)
X.I('CALCULAR_DESCUENTO');

-- 3. Registrar datos de entrada
X.R('P1_CLIENTE_ID');
X.R('P1_MONTO');
X.E('v_tipo_cliente', v_tipo_cliente);

-- 4. Lógica del proceso...
v_descuento := calcular(v_tipo_cliente, v_monto);

-- 5. Registrar resultados
X.E('v_descuento',   TO_CHAR(v_descuento));
X.E('resultado',     'OK');

-- 6. Para textos largos, usar S
X.S('query_dinamica', v_sql_clob);
```

---

## 5. Comportamiento fuera de APEX

El paquete funciona en contextos **PL/SQL puro** (scripts, jobs, procedimientos sin sesión APEX).  
En ese caso:

| Situación | Comportamiento |
|---|---|
| `APP_SESSION` no disponible | `sesion_id` se establece como `'999'` |
| `A(p_evento)` | No hace nada (no hay contexto de página) |
| `R(p_item_name)` | Registra `NULL` si el ítem no existe |
| `E`, `S`, `I`, `B`, `L`, `H` | Funcionan normalmente |

---

## 6. Notas de implementación

- **Transacciones autónomas:** todas las escrituras usan `PRAGMA AUTONOMOUS_TRANSACTION`, por lo que los registros persisten aunque el proceso llamador haga `ROLLBACK`.
- **Límite de valor:** la columna `valor` admite hasta **4 000 caracteres**. El procedimiento `S` divide automáticamente los CLOBs en fragmentos de 3 900 caracteres.
- **Limpieza de saltos de línea:** `E` elimina `CHR(10)` y `CHR(13)` de etiqueta y valor para evitar problemas de visualización en tablas.
- **Identificación de sesión:** la función interna `get_session_id` usa `v('APP_SESSION')` con manejo de excepción, devolviendo `'999'` si falla.

---

## 7. Consultar el log

```sql
-- Ver todo el log de la sesión actual
SELECT sesion_id,
       evento,
       linea,
       etiqueta,
       valor,
       TO_CHAR(fecha_log, 'HH24:MI:SS.FF3') AS hora
  FROM LOG_DEBUG_COMPARTIDO
 WHERE sesion_id = NVL(v('APP_SESSION'), '999')
 ORDER BY fecha_log, linea;

-- Ver log de todas las sesiones
SELECT *
  FROM LOG_DEBUG_COMPARTIDO
 ORDER BY fecha_log, linea;

-- Filtrar por evento
SELECT *
  FROM LOG_DEBUG_COMPARTIDO
 WHERE evento = 'CALCULAR_DESCUENTO'
 ORDER BY linea;
```

---

*Paquete interno de debugging — no exponer en ambientes de producción sin control de acceso sobre la tabla `LOG_DEBUG_COMPARTIDO`.*
