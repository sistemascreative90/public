# Norma de Nomenclatura y Estándares de Desarrollo Oracle APEX

*Estándar corporativo aplicable a todos los proyectos*

Versión 1.0

## 1. Introducción y propósito

Este documento define la norma de nomenclatura y los estándares de desarrollo para todos los proyectos construidos sobre Oracle APEX. Su objetivo es lograr que cualquier desarrollador pueda abrir una aplicación, leer el nombre de un componente y saber de inmediato qué hace, en qué página vive y con qué intención fue creado, sin necesidad de abrirlo.

La norma no está atada a ningún módulo ni paquete PL/SQL en particular. Es un marco general: los mismos prefijos y patrones se aplican igual a un módulo de facturación, de stock, de recursos humanos o de cualquier otro dominio. Lo que cambia entre proyectos es el contenido del nombre, nunca su estructura.

### 1.1 Beneficios

- Lectura inmediata: el nombre comunica tipo, ubicación e intención.
- Mantenibilidad: cualquier integrante del equipo entiende el código ajeno.
- Búsqueda y filtrado: los prefijos permiten encontrar componentes rápido.
- Separación de responsabilidades: la UI vive en APEX, la lógica de negocio en la base de datos.
- Revisiones consistentes: las auditorías de código se vuelven mecánicas y rápidas.

### 1.2 Alcance

La norma aplica a todos los artefactos de una aplicación APEX:

- Páginas, regiones, ítems y botones.
- Dynamic Actions y sus acciones asociadas.
- Procesos de página (Pre-Rendering, Rendering, Processing).
- Validaciones, computations y branches.
- Procesos AJAX (On Demand).
- Shared Components: listas de valores, listas, autorizaciones.
- Código JavaScript, CSS y PL/SQL invocado desde APEX.

## 2. Principios fundamentales

Antes de las reglas concretas, hay cuatro principios que sostienen toda la norma. Cuando una situación no esté contemplada explícitamente, estos principios deciden.

### 2.1 La lógica de negocio vive en la base de datos

Las reglas, los cálculos, las validaciones de negocio y la persistencia residen en PL/SQL (paquetes, procedimientos y funciones). APEX cumple un rol distinto: dispara procesos, valida la interfaz y llama a la base de datos. APEX no debe contener reglas de negocio complejas escritas "inline".

### 2.2 Un componente se ubica por su nombre

Todo componente debe poder ubicarse leyendo su nombre, que combina tres ideas: el tipo de objeto, la página o módulo donde vive, y la intención o acción que cumple. Un nombre bien puesto evita tener que abrir el componente para saber qué hace.

### 2.3 Una cosa, una responsabilidad

Cada proceso, Dynamic Action o llamada AJAX debe hacer una sola cosa clara. Si un proceso valida, calcula y graba a la vez, conviene dividirlo. Nombres como "PRC_AS_PRO_GUARDAR" describen una intención única.

### 2.4 Nombres técnicos limpios

- Sin tildes ni eñes en nombres técnicos (escribir ANIO, no AÑO).
- Separación de palabras con guion bajo (_).
- Objetos lógicos (procesos, Dynamic Actions, validaciones, branches) en MAYÚSCULAS.
- Static IDs (para JavaScript y CSS) en minúsculas.
- Sin espacios ni caracteres especiales en ningún identificador técnico.

## 3. La regla de los dos registros

Esta es la decisión central de la norma y conviene tenerla clara antes de entrar en cada componente. Existen dos tipos de identificadores y cada uno usa un registro distinto de mayúsculas, sin excepciones ni formatos intermedios.

### 3.1 Nombre del componente: MAYÚSCULAS

Es el nombre lógico que APEX usa internamente y que ve el desarrollador en el árbol de componentes. Va siempre en MAYÚSCULAS, con prefijos y guion bajo.

```
RGN_TREE_ARBOL
BTN_P_GUARDAR
PRC_AS_PRO_GUARDAR
DA_CHG_P52_CANTIDAD_CAL_SUBTOTAL
```

### 3.2 Static ID: minúsculas

Es el identificador que termina en el HTML como atributo id y que se usa desde JavaScript y CSS. Va siempre en minúsculas, con el prefijo `sid_` y guion bajo. Nada de mayúsculas mezcladas, nada de camelCase.

```
sid_tree_arbol
sid_frm_datos_nodo
sid_rep_datos_cliente
```

La correspondencia es directa y mecánica: se toma el nombre del componente, se le quita el prefijo de tipo si se desea abreviar, y se pasa todo a minúsculas con el prefijo `sid_`. Así, `RGN_TREE_ARBOL` tiene como Static ID `sid_tree_arbol`.

### 3.3 Por qué minúsculas para el Static ID

- JavaScript y CSS distinguen mayúsculas de minúsculas; un solo registro elimina errores por descuido.
- Es predecible: sabiendo el nombre del componente se deduce el Static ID sin mirar la configuración.
- Es legible en selectores CSS y llamadas `apex.region()` o `apex.item()`.
- Evita el formato camelCase anterior, más difícil de recordar y de teclear.

> **Regla práctica:** si lo ve el desarrollador en APEX, va en MAYÚSCULAS. Si lo usa el navegador (JS/CSS), va en minúsculas con `sid_`.

## 4. Páginas

El nombre visible de una página describe módulo e intención en lenguaje natural, con la primera letra en mayúscula. No lleva prefijos técnicos porque es lo que ve el usuario final.

| Formato | Descripción | Ejemplo |
| --- | --- | --- |
| Modulo - Accion | Nombre legible para el usuario | Presupuesto de Obra |
| Modulo - Accion | Mantenimiento de catálogos | Mantenimiento de Clientes |
| Modulo - Accion | Consulta o bandeja | Bandeja de Aprobaciones |

### 4.1 Rangos de numeración por módulo

Asignar rangos de números de página por módulo ayuda a no mezclar funcionalidades y deja espacio para crecer. La asignación concreta es decisión de cada proyecto; lo importante es reservar bloques y respetarlos.

| Rango sugerido | Uso |
| --- | --- |
| 1 - 9 | Inicio, login, dashboard, páginas globales |
| 10 - 99 | Módulo principal o más usado de la aplicación |
| 100 - 199 | Segundo módulo |
| 200 - 299 | Tercer módulo, y así sucesivamente |
| 900 - 999 | Administración, utilitarios, configuración |

*Dentro de un módulo conviene agrupar las páginas que colaboran. Por ejemplo, una lista maestra, su detalle y su formulario de edición van juntas y consecutivas.*

## 5. Regiones

El nombre de una región sigue el formato `RGN_{TIPO}_{OBJETO}`. El prefijo `RGN_` marca que es una región; `{TIPO}` indica qué clase de región es; `{OBJETO}` describe su contenido.

| Tipo | Clase de región | Ejemplo |
| --- | --- | --- |
| FRM | Formulario (Form) | RGN_FRM_CLIENTE |
| IR | Interactive Report | RGN_IR_PEDIDOS |
| IG | Interactive Grid | RGN_IG_DETALLE |
| REP | Classic Report | RGN_REP_RESUMEN |
| TREE | Tree (árbol jerárquico) | RGN_TREE_ARBOL |
| CHART | Gráfico | RGN_CHART_VENTAS |
| BTN | Contenedor de botones | RGN_BTN_ACCIONES |
| WK | Región oculta con ítems auxiliares | RGN_WK_OCULTO |
| PRM | Parámetros / filtros de búsqueda | RGN_PRM_FILTROS |
| SUB | Subregión | RGN_SUB_TOTALES |
| MSG | Mensajes / alertas | RGN_MSG_AVISO |
| CARD | Lista de tarjetas (Cards) | RGN_CARD_PRODUCTOS |

### 5.1 Static ID de regiones

Toda región que vaya a manipularse desde JavaScript o CSS debe tener su Static ID. El Static ID se escribe en minúsculas y refleja el nombre de la región.

| Nombre de la región | Static ID |
| --- | --- |
| RGN_TREE_ARBOL | sid_tree_arbol |
| RGN_FRM_CLIENTE | sid_frm_cliente |
| RGN_IR_PEDIDOS | sid_ir_pedidos |
| RGN_REP_RESUMEN | sid_rep_resumen |
| RGN_WK_OCULTO | sid_wk_oculto |

## 6. Ítems de página

APEX antepone automáticamente `P{PAGINA}_` a cada ítem. Sobre esa base, la norma distingue tres clases de ítem según su papel.

| Prefijo | Clase de ítem | Ejemplo |
| --- | --- | --- |
| (ninguno) | Campo de dato real de la tabla | P10_COD_CLIENTE |
| WK_ | Auxiliar oculto (no visible) | P10_WK_MODO |
| SC_ | Auxiliar visible (solo lectura) | P10_SC_TOTAL |

La idea: si el ítem corresponde a una columna que se graba en la base, lleva el nombre de esa columna sin prefijo extra. Si es una variable de trabajo interna que el usuario no ve, lleva `WK_`. Si es un valor mostrado en pantalla pero que no se edita ni se graba directamente, lleva `SC_`.

### 6.1 Correspondencia con campos de base de datos

Cuando el ítem representa una columna de una tabla, su nombre reproduce el de la columna. Si en la tabla la columna es `COD_PERSONA`, el ítem de la página 10 será `P10_COD_PERSONA`. Esto vale también para sistemas heredados: si en Forms el campo se llamaba `COD_PERSONA`, se mantiene igual.

### 6.2 Tipos de ítem y cuándo usarlos

| Tipo APEX | Cuándo usarlo | Ejemplo |
| --- | --- | --- |
| Hidden | IDs, banderas, valores de sesión | P10_COD_CLIENTE |
| Text Field | Texto libre editable | P10_NOMBRE |
| Number Field | Cantidades, importes, totales | P10_IMPORTE |
| Display Only | Valores calculados no editables | P10_SC_TOTAL |
| Select List | Catálogos cortos / LOVs simples | P10_ESTADO |
| Popup LOV | Catálogos extensos con búsqueda | P10_COD_ARTICULO |
| Text Area | Observaciones o textos largos | P10_OBSERVACION |
| Switch | Banderas Sí/No | P10_ACTIVO |
| Date Picker | Fechas | P10_FEC_ALTA |

### 6.3 Protección de ítems

Los ítems que reciben su valor por URL o por un branch (por ejemplo un identificador que viaja de una página a otra) deben tener desactivada la protección de valor (Value Protected en OFF) para que APEX acepte el valor entrante. Los ítems críticos que no deben manipularse desde el navegador se dejan protegidos.

## 7. Botones

Formato `BTN_{SCOPE}_{ACCION}`. El `{SCOPE}` indica el alcance del botón; `{ACCION}` describe qué hace, idealmente con un verbo.

| Scope | Alcance | Ejemplo |
| --- | --- | --- |
| P | Botón de página (acción principal) | BTN_P_GUARDAR |
| P | Botón de página | BTN_P_CANCELAR |
| P | Botón de página | BTN_P_NUEVO |
| P | Botón de página | BTN_P_VOLVER |
| RGN | Botón dentro de una región concreta | BTN_RGN_AGREGAR |
| IG | Botón de Interactive Grid | BTN_IG_AGREGAR_FILA |
| DLG | Botón de diálogo / modal | BTN_DLG_CONFIRMAR |

*El nombre del botón es, además, el valor de REQUEST que APEX envía al hacer Submit. Por eso conviene que sea claro: un proceso que reacciona a un botón se condiciona con "Request = Value" usando ese mismo nombre.*

### 7.1 Botones generados por el wizard de formulario (CREATE / SAVE)

Cuando el wizard de APEX genera un formulario (Form sobre una tabla), crea dos botones de guardado separados —`CREATE` y `SAVE`— que nunca se muestran al mismo tiempo: `CREATE` aparece cuando el registro es nuevo y `SAVE` cuando ya existe. Como cumplen la misma intención funcional para el usuario, ambos se renombran como `BTN_P_GUARDAR`.

Si APEX exige nombres únicos por página y genera conflicto al usar el mismo nombre dos veces, se diferencia el botón de alta como `BTN_P_CREAR`, manteniendo `BTN_P_GUARDAR` para el de edición.

## 8. Procesos de página

Formato `PRC_{POINT}_{VERBO}_{OBJETO}`. El `{POINT}` indica en qué momento del ciclo de la página se ejecuta; el verbo y el objeto describen la acción.

| Código | Process Point | Cuándo ejecuta |
| --- | --- | --- |
| BH | Before Header | Antes de renderizar: inicializa y carga datos |
| AH | After Header | Justo después del header (poco usado) |
| BS | Before Submit | Antes del envío: prevalidaciones |
| AS | After Submit | Después del envío: DML y lógica de negocio |
| PRO | Processing | Sinónimo de After Submit en APEX moderno |

### 8.1 Ejemplos de nombres de proceso

| Nombre | Qué hace |
| --- | --- |
| PRC_BH_INI_PANTALLA | Carga valores por defecto y estado inicial |
| PRC_BH_CARGA_REGISTRO | Pre-carga datos cuando se edita un registro existente |
| PRC_BS_VAL_PRE_GUARDAR | Validaciones previas al guardado |
| PRC_AS_PRO_GUARDAR | Inserta o actualiza llamando a un paquete PL/SQL |
| PRC_AS_PRO_ELIMINAR | Elimina el registro llamando a un paquete PL/SQL |

### 8.2 Reglas de los procesos

- Un proceso tiene una sola responsabilidad principal.
- Si ejecuta lógica de negocio, llama a un paquete PL/SQL; no escribe reglas complejas inline.
- Los procesos BH se reservan para inicializar: cargar valores por defecto, preparar colecciones y setear variables de contexto.

### 8.3 Procesos AJAX (On Demand)

Formato `AJX_{VERBO}_{OBJETO}`. Cada AJAX devuelve una sola respuesta clara y cumple una sola finalidad.

| Nombre | Qué devuelve |
| --- | --- |
| AJX_OBT_SALDO_CUENTA | El saldo de una cuenta para mostrar en pantalla |
| AJX_CON_LOV_PERSONAS | Una lista de personas para una LOV dinámica |
| AJX_VAL_CODIGO | Si un código existe o está disponible |

*Conviene estandarizar el formato de respuesta, por ejemplo un objeto JSON con un indicador de éxito, los datos y un mensaje.*

## 9. Dynamic Actions

Formato `DA_{EVT}_{ORIGEN}_{VERBO}_{RESULTADO}`. Describe qué evento dispara la acción, sobre qué elemento, qué verbo de negocio aplica y qué resultado produce.

### 9.1 Códigos de evento

| Código | Evento APEX | Uso típico |
| --- | --- | --- |
| CLK | Click | Clic en un botón o elemento |
| CHG | Change | Cambio en un ítem (select, número, texto) |
| INIT | Page Load | Al cargar la página |
| BLUR | Lose Focus | Al perder el foco un campo |
| FOCUS | Get Focus | Al obtener el foco un campo |
| KEYD | Key Down | Al presionar una tecla |

### 9.2 Verbos de acción

| Verbo | Significado | Ejemplo de nombre |
| --- | --- | --- |
| INI | Inicializar / setear defaults | DA_INIT_P10_INI_PANTALLA |
| VAL | Validar | DA_CLK_BTN_P_GUARDAR_VAL_FORM |
| OBT | Obtener datos de la base | DA_CHG_P10_COD_ART_OBT_PRECIO |
| CAL | Calcular (JS o PL/SQL) | DA_CHG_P10_CANTIDAD_CAL_TOTAL |
| REF | Refrescar una región | DA_INIT_RGN_IR_PEDIDOS_REF |
| PRO | Procesar | DA_CLK_BTN_P_APROBAR_PRO_ESTADO |
| ACT | Actualizar la interfaz | DA_CHG_P10_TIPO_ACT_CAMPOS |
| NAV | Navegar a otra página | DA_CLK_BTN_P_VOLVER_NAV_P100 |

## 10. Validaciones y computations

### 10.1 Validaciones

Formato `VAL_{CAMPO}_{REGLA}`. Indica qué ítem valida y bajo qué regla.

| Código | Tipo de regla | Ejemplo |
| --- | --- | --- |
| REQ | Requerido (obligatorio) | VAL_NOMBRE_REQ |
| FMT | Formato (fecha, número, etc.) | VAL_FEC_ALTA_FMT |
| RNG | Rango de valores | VAL_IMPORTE_RNG |
| UK | Único (sin duplicados) | VAL_COD_CLIENTE_UK |
| FK | Existencia de referencia | VAL_COD_ARTICULO_FK |
| CHK | Regla de negocio | VAL_ESTADO_MODIFICABLE |

La validación de interfaz (campos requeridos, formatos) puede resolverse en APEX. La validación de negocio se resuelve en la base de datos y APEX solo muestra el resultado.

### 10.2 Computations

Formato `CMP_{POINT}_{CAMPO}_{ACCION}`. Asigna un valor a un ítem en un momento del ciclo de la página.

| Nombre | Qué hace |
| --- | --- |
| CMP_BH_WK_MODO_INI | Inicializa la variable de modo al cargar la página |
| CMP_AS_SC_TOTAL_CAL | Calcula un total después del envío |

## 11. Branches (navegación)

Formato `BRH_{CONDICION}_TO_P{PAGINA}`. Indica bajo qué condición se navega y hacia qué página.

| Nombre | Cuándo se dispara |
| --- | --- |
| BRH_OK_TO_P100 | Tras guardar con éxito, vuelve a la página 100 |
| BRH_CANCEL_TO_P100 | Al cancelar, vuelve a la página 100 |
| BRH_NUEVO_TO_P101 | Tras crear un registro, va al detalle en la 101 |
| BRH_VOLVER_TO_P100 | El botón volver regresa a la lista |

## 12. Listas de valores (LOVs)

Las LOVs reutilizables y compartidas siguen el formato `LV_{OBJETO}`. Las LOVs que dependen de ítems de la página (por ejemplo una lista en cascada) se definen como SQL inline en el propio ítem, no en Shared Components.

| Nombre | Origen | Tipo |
| --- | --- | --- |
| LV_ESTADO | Lista estática de estados | Shared (estática) |
| LV_COD_CLIENTE | Consulta a la tabla de clientes | Shared (SQL) |
| LV_UNID_MEDIDA | Consulta a unidades de medida | Shared (SQL) |
| (SQL inline) | Filtrada por un ítem de la página | En el ítem |

### 12.1 Reglas de las LOVs

- En Shared Components solo si es independiente de la página y reutilizable.
- Si depende de un ítem de la página (cascada), se define inline en el ítem.
- El valor mostrado es legible para el usuario; el valor de retorno es la clave técnica.
- Siempre ordenar el resultado por el valor mostrado o por un orden lógico del negocio.

## 13. PL/SQL invocado desde APEX

APEX debe llamar a interfaces estables en la base de datos, no contener SQL extenso y suelto. La lógica vive en paquetes, procedimientos y funciones. La norma no impone un paquete concreto: cada proyecto puede tener los suyos. Lo que se estandariza es la forma de nombrar y el manejo de errores.

### 13.1 Convención de nombres

Se sugiere una convención corta y coherente, aplicada de igual forma en todos los proyectos. Cada equipo puede elegir su juego de prefijos, pero debe usarlo de manera uniforme. Dos esquemas habituales:

| Elemento | Esquema A | Esquema B |
| --- | --- | --- |
| Paquete | PA_{MODULO} | PKG_{MODULO} |
| Procedimiento | pr_{accion} | pp_{accion} |
| Función | fu_{accion} | fp_{accion} |

*Lo importante no es cuál de los dos se elija, sino que dentro de un mismo sistema se use uno solo, de principio a fin.*

### 13.2 Objetos atados a una página

Cuando un procedimiento o función se consume desde una sola página y depende de ella, se le agrega el sufijo del número de página: `PR_{VERBO}_{OBJETO}_P{PAGINA}`. Si el objeto es reutilizable y no depende de ninguna página, se omite el sufijo.

| Nombre | Significado |
| --- | --- |
| PR_INS_MOVIMIENTO_P27 | Inserta un movimiento, usado solo en la página 27 |
| FU_OBT_DATOS_P27 | Obtiene datos, usado solo en la página 27 |
| PR_VALIDAR_RUC | Valida un RUC; reutilizable, sin sufijo de página |

### 13.3 Identificadores y límite de longitud

En bases Oracle anteriores a la versión 12.2 los identificadores tienen un máximo de **30 caracteres**. Conviene mantenerse dentro de ese límite por compatibilidad, sobre todo si hay versiones en Oracle Forms. Si un nombre se pasa, se abrevia el verbo o el objeto, nunca el prefijo que da significado.

### 13.4 Manejo de errores

Todo procedimiento o función consumido desde APEX debe manejar errores de forma uniforme, devolviendo un mensaje funcional para la interfaz y registrando el punto técnico de la falla. El patrón recomendado usa:

- `p_mensaje OUT VARCHAR2`: mensaje funcional, claro, para mostrar al usuario (nunca un ORA- crudo).
- `v_lugar VARCHAR2`: rastro del punto de ejecución, para diagnóstico.
- `e_error EXCEPTION`: excepción controlada para cortar el flujo cuando hay un error de negocio.

**Esqueleto del patrón:**

```sql
DECLARE
  v_lugar  VARCHAR2(255) := 'INICIO';
  e_error  EXCEPTION;
BEGIN
  p_mensaje := NULL;
  v_lugar   := 'Validacion X';
  -- si falla: p_mensaje := 'Mensaje funcional'; RAISE e_error;

  v_lugar := 'Llamada a subproceso Y';
  pr_subproceso(..., p_mensaje => p_mensaje);
  IF p_mensaje IS NOT NULL THEN
     RAISE e_error;
  END IF;
EXCEPTION
  WHEN e_error THEN
     p_mensaje := p_mensaje || ' lugar: ' || v_lugar;
     RETURN;
  WHEN OTHERS THEN
     p_mensaje := SQLERRM || ' lugar: ' || v_lugar;
     RETURN;
END;
```

## 14. Grillas editables (Interactive Grid)

Toda grilla editable debe trabajar contra una colección APEX, no contra las tablas productivas directamente. Esto da control sobre el momento de aplicar los cambios y permite validar antes de tocar la base.

- **Carga:** un proceso BH crea, limpia y carga la colección. Nombre sugerido: `PRC_BH_COLL_{OBJETO}_CARGAR`.
- **Edición:** las altas, modificaciones y bajas se reflejan en la colección, no en la tabla.
- **Aplicación:** un único proceso valida y aplica en orden (eliminar, insertar, actualizar). Nombre sugerido: `PRC_PRO_COLL_{OBJETO}_APLICAR`.
- **Identificador técnico:** la grilla usa un `SEQ_ID` como clave y lo mantiene oculto.
- **Nombre de la colección:** `COLL_{OBJETO}_P{PAGINA}`.

*Si la aplicación falla, no se aplica nada y la colección se conserva para que el usuario corrija.*

## 15. Presentación de datos al usuario

### 15.1 Encabezados de columna

- Primera letra mayúscula y el resto en minúscula: Persona, Documento, Fecha, Monto, Estado.
- En títulos de varias palabras, los conectores van en minúscula: Tipo de persona, Fecha de proceso.
- Las siglas van en mayúscula: RUC, ID, SQL, API.
- Los nombres técnicos no se muestran nunca al usuario.

### 15.2 Alineación

La alineación del encabezado debe coincidir con la del dato. Esta es una regla obligatoria.

| Tipo de dato | Alineación | Ejemplo |
| --- | --- | --- |
| Texto | Izquierda | Nombre, dirección |
| Números e importes | Derecha | Cantidad, total |
| Fechas | Centrado (consistente) | Fecha de alta |
| Indicadores Sí/No | Centrado | Activo, vigente |

## 16. Seguridad

- Definir qué páginas y requests exigen checksum (Session State Protection) y cuándo.
- Proteger los ítems críticos (Value Protected) para evitar manipulación desde la consola del navegador.
- Los ítems que reciben valores legítimos por URL o branch se dejan sin proteger, pero validados en el servidor.

## 17. Tabla resumen de prefijos

Referencia rápida de todos los prefijos de la norma, para tener a mano durante el desarrollo.

| Componente | Formato | Ejemplo |
| --- | --- | --- |
| Región | RGN_{TIPO}_{OBJETO} | RGN_FRM_CLIENTE |
| Static ID | sid_{tipo}_{objeto} | sid_frm_cliente |
| Ítem de dato | P{PAG}_{CAMPO} | P10_NOMBRE |
| Ítem auxiliar oculto | P{PAG}_WK_{NOMBRE} | P10_WK_MODO |
| Ítem auxiliar visible | P{PAG}_SC_{NOMBRE} | P10_SC_TOTAL |
| Botón | BTN_{SCOPE}_{ACCION} | BTN_P_GUARDAR |
| Proceso | PRC_{POINT}_{VERBO}_{OBJETO} | PRC_AS_PRO_GUARDAR |
| Proceso AJAX | AJX_{VERBO}_{OBJETO} | AJX_OBT_SALDO |
| Dynamic Action | DA_{EVT}_{ORIGEN}_{VERBO}_{RES} | DA_CHG_P10_CANT_CAL_TOTAL |
| Validación | VAL_{CAMPO}_{REGLA} | VAL_NOMBRE_REQ |
| Computation | CMP_{POINT}_{CAMPO}_{ACCION} | CMP_BH_WK_MODO_INI |
| Branch | BRH_{COND}_TO_P{PAG} | BRH_OK_TO_P100 |
| LOV compartida | LV_{OBJETO} | LV_ESTADO |
| Colección | COLL_{OBJETO}_P{PAG} | COLL_DETALLE_P27 |

> **Para recordar:** nombres de componentes en MAYÚSCULAS, Static IDs en minúsculas con `sid_`, lógica de negocio en la base de datos, y cada componente con una sola responsabilidad clara.
