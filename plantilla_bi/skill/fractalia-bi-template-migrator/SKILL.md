---
name: fractalia-bi-template-migrator
description: Reutiliza la maqueta PBIP corporativa de Fractalia (F:\Automatizacion PBI\plantilla_bi) para crear o migrar páginas de Power BI reusando los componentes ya homologados, sin crear modelos nuevos salvo confirmación expresa. Actívalo cuando se mencione la plantilla, el catálogo, la maqueta, componentes reutilizables, visuales HTML Content, migrar un informe al estilo corporativo, franja de filtros, Home, fondos, KPIs, tokens DS_*, o PBIP/PBIR. Guía la adaptación de DAX y PBIR sin inventar estructuras de Power BI.
---

# Migrador a la plantilla BI de Fractalia

Sirve para **migrar un informe existente** al estándar corporativo y para **crear páginas nuevas** desde la maqueta. La fuente de verdad es `F:\Automatizacion PBI\plantilla_bi`, con `VOCABULARIO.md` (contrato de componentes) y `README.md` (uso y diagnóstico).

---

## 0. El principio: reutilizar, no crear

**Este skill es para reusar los cuadros que ya existen en la plantilla, eligiendo el que más se asemeje a lo que hay que representar. No se crea nada nuevo salvo que el usuario lo confirme expresamente.**

Eso vale para todo, no solo para gráficos: ni segmentadores, ni rejillas, ni notas al pie, ni leyendas, ni iconos, ni columnas, ni páginas. Si al mirar el origen parece que falta algo, **se propone y se espera respuesta**; no se implementa por iniciativa propia.

El costo de agregar de más no es el trabajo extra: es que el informe **deja de ser homologable** y el equipo pierde la referencia única. Un gráfico inventado, aunque sea mejor, rompe eso.

Y si de verdad hace falta un modelo que no existe: **se acuerda primero, y una vez acordado se agrega a la plantilla** (medida + entrada en el CATÁLOGO + fila en `VOCABULARIO.md`) antes de usarlo en el informe destino. Así el próximo que lo necesite lo encuentra homologado.

---

## 1. Reglas que no se negocian

1. **No agregar nada que no se haya pedido.** Si parece faltar, se propone; no se implementa.
2. **Si la hoja original no tiene filtro, la migrada no lleva filtro.** El **modelo de hoja sí va siempre**: fondo, logo, título de página, navegador, franja, rótulo y fecha de carga.
3. **Antes de construir un gráfico, preguntar cuál se va a usar**, para que salga del catálogo.
4. **Preservar la semántica de las medidas de negocio.** Si una medida del origen tiene un defecto, se señala con el arreglo exacto; no se cambia sin autorización, porque mueve números que el usuario ya validó.
5. **Nunca truncar en silencio.** Con `TOPN`, el título dice cuántas quedaron fuera. Si hace falta scroll, el título lo avisa. Los blancos llevan etiqueta explícita: `(Sin dato)`, `(Sin asignar)`, `-`.
6. **Antes de borrar cualquier cosa, comprobar que nadie la referencia**, y respaldarla.

---

## 2. Cómo elegir el modelo: por semejanza

Se parte del **visual nativo del origen** y se busca el modelo homologado equivalente. La pregunta clave no es qué forma tiene el gráfico, sino **de dónde salen sus series**.

| Visual nativo del origen | Modelo que se le asemeja |
|---|---|
| `lineChart` | Líneas con marcadores y valores |
| `areaChart` | Área acumulada |
| `clusteredColumnChart` con **1 medida** partida por una dimensión | Columnas apiladas por periodo *(o agrupadas con una sola serie)* |
| `clusteredColumnChart` con **N medidas** | Columnas agrupadas, series por medida |
| `stackedColumnChart` | Columnas apiladas por periodo |
| `clusteredBarChart` | Barras horizontales simples |
| `stackedBarChart` | Barras horizontales apiladas |
| `donutChart` / `pieChart` | Dona con leyenda al costado |
| `lineStackedColumnComboChart` | Combo barras + línea |
| `gauge` | Gauge semicircular |
| `card` | Tarjeta KPI |
| `matrix` de 2 niveles | Matriz jerárquica |
| `matrix` de 1 nivel | Matriz plana |
| `matrix` con medidas en columnas | Matriz con medidas como columnas |
| `tableEx` de detalle | **Dejar la tabla nativa**, estilada (ver §8) |
| `slicer`, `shape`, `textbox`, `image` de mobiliario | Piezas del modelo de hoja (§6) |

**Apiladas o agrupadas es la confusión más frecuente.** Apiladas = una medida partida por una dimensión, la altura total **sí** se lee como suma. Agrupadas = varias medidas comparadas entre sí, **no** se suman. Usar apiladas para comparar dos medidas convierte la altura en una suma sin significado.

Cuando un modelo admite criterio —por ejemplo columnas verticales por dimensión, que se pueden resolver como barras horizontales o como agrupadas de una serie— **se plantean las opciones con sus consecuencias y decide el usuario**.

---

## 3. Catálogo de modelos

Pedir el componente por su **nombre funcional**; el nombre técnico es interno.

| Nombre funcional | Medida de referencia |
|---|---|
| Tarjeta KPI por estado (etiqueta e icono automáticos) | `HTML_Backlog_KPI_EnGestion` |
| Tarjeta KPI de duración | `HTML_KPI_Q2` |
| Columnas apiladas por periodo, series por **dimensión** | `HTML_Volumetria_Apilado` |
| Columnas agrupadas por periodo, series por **medida** | `HTML_Barras_Agrupadas` |
| Barras horizontales apiladas por estado | `HTML_Backlog_Barras_AgenteEstado` |
| Barras horizontales simples (ranking con Top N) | `HTML_Vision_Barras_Cliente` |
| Combo barras + línea | `HTML_Vision_Combo_Volumetria` |
| Dona con leyenda al costado | `HTML_Donut_Estados` |
| Líneas con marcadores y valores | `HTML_Backlog_Lineas_Balance` |
| Área acumulada | `HTML_Backlog_Area_Backlog` |
| Matriz jerárquica de 2 niveles | `HTML_Volumetria_MatrizSoporte` |
| Matriz plana de 1 nivel | `HTML_Volumetria_MatrizAgentes` |
| Matriz con medidas como columnas | `HTML_Matriz_Medidas` |
| Tabla de detalle | `HTML_Registro_Tabla` |
| Box plot de cuartiles | `HTML_Cuartiles_Tiempos` |
| Gauge semicircular | `HTML_Gauge` |
| Chips de categorías | `HTML_Chips` |

**Al reutilizar:** copiar el visual y la medida, conservar los tokens `DS_*` y el acabado, repuntar tabla, clave, dimensiones y medidas al modelo destino, y mantener el nombre funcional visible. **No rediseñar el componente.**

Al portar tokens a otra paleta, **mapear por función y no por nombre**: un trazo fino de icono necesita el tono oscuro o se lava; un borde de caja necesita el medio o pesa demasiado. Si la paleta destino no tiene un tono intermedio, hay que inventarlo.

---

## 4. Las dos trampas que rompen en silencio

Dan el mismo síntoma —**tarjeta sin fondo ni marco y contenido apelotonado arriba**— y ninguna produce error: el visual renderiza contento. Comprobar en este orden.

### Trampa 1: una comilla simple mata todo el CSS que le sigue

El CSS se inyecta como `style='...'`, con comilla **simple** de delimitador. Si un token trae una comilla simple, cierra el atributo antes de tiempo y el navegador **descarta el resto del CSS**.

```dax
DS_Fuente = "font-family:'Segoe UI',system-ui,sans-serif;"   -- ROMPE
DS_Fuente = "font-family:Segoe UI,Arial,sans-serif;"          -- correcto
```

Lo caro de diagnosticar es que se pierde `display:flex`, y sin él los hijos dejan de ser flex items: todo `flex:1 1 auto` resuelve a altura 0 y el gráfico desaparece. Los nombres de familia con espacios son CSS válido sin comillas. Si hiciera falta una comilla real, va como `&#39;`.

**Al portar tokens a otro informe, revisar esto primero.** Un solo carácter puede romper todos los motores del informe a la vez, porque el token viaja dentro de `DS_Contenedor_Estilo`, `DS_TituloPagina_Estilo` y el rótulo.

### Trampa 2: `height:100%` no da altura

El visual HTML Content **no propaga su alto al HTML**. Un `height:100%` resuelve contra un alto indefinido y la tarjeta colapsa al alto de su contenido. Hay motores que lo llevan escrito y **no hace nada**: se ven bien solo porque su contenido mide en píxeles lo mismo que el visual.

**El alto sale siempre de un número**, por una de dos vías:

| Vía | Cómo | Cuándo |
|---|---|---|
| cableada | `VAR __AltoContenedor = 240` | convención de la plantilla |
| declarada por el visual | `SELECTEDVALUE(DS_Tamano[Alto])` leído con `[DS_Alto]`, y cada visual fija el valor en su `filterConfig` | recomendada: una medida sirve para cualquier tamaño |

Con la segunda vía, **cambiar el alto de un visual obliga a cambiar también su filtro `Alto`**, y el valor tiene que existir en la tabla desconectada.

---

## 5. Política de geometría

### Las tarjetas de una fila miden lo mismo

Todas las tarjetas de una fila **declaran el mismo alto**, y ese alto es el que mide el visual. Si una fila mezcla una tarjeta de 250 con otra que se autoajustó a 90, asoma el fondo de página a media fila. Tres condiciones **juntas**:

1. el contenedor fija su alto **en px**, nunca `height:100%`
2. el cuerpo también en px: `alto − marco − título`, nunca `flex:1 1 auto`
3. el contenido **reparte** ese alto, para que tampoco quede blanco dentro de la tarjeta

Presupuesto vertical: `marco 14` (relleno 6+6, borde 1+1) `+ título 22` (alto 20, margen 2) = **36**. Si el título es de 14 px, son 28.

La geometría cierra en la retícula: las columnas comparten `x` y ancho, las filas comparten `y` y alto, y el borde derecho de la última columna coincide con el fin de la banda.

### El gráfico llena la tarjeta

```dax
VAR __AltoFila = MIN(60, MAX(18, INT(DIVIDE(__AltoCuerpo, __Cantidad, 0))))
VAR __AltoBarra = MAX(10, __AltoFila - 10)
```

Aplicado con `height:100%`, `grid-auto-rows:<__AltoFila>px` y `align-content:center`. El **tope** evita que con una o dos categorías la barra quede desproporcionada; el **piso** es donde aparece el scroll en vez de comprimir la barra. Si el grid lleva `row-gap`, descontarlo: `(__AltoCuerpo − (N−1)·gap) / N`. En una matriz, la cabecera queda en `auto` y solo las filas de datos se reparten: `grid-template-rows:auto repeat(N, Xpx)`.

### Ningún umbral de categorías va como número fijo

Un `<= 12` heredado saca scroll en un visual ancho que tenía sitio de sobra.

```dax
VAR __AnchoMinPeriodo = 63                          -- lo que ocupa una categoría
VAR __AnchoUtil = MAX(120, [DS_Ancho] - 22)
VAR __Caben = MAX(1, INT(DIVIDE(__AnchoUtil, __AnchoMinPeriodo, 0)))
VAR __Entran = __CantidadCategorias <= __Caben
```

`__AnchoMinPeriodo` se calcula sumando lo que ocupa una categoría: barras, hueco, relleno y —si es mayor— **el ancho de la etiqueta del eje**, que con textos largos manda:

```dax
VAR __LargoMax = MAXX(__Cat, LEN(<columna>))
VAR __AnchoMinPeriodo = MAX(40, INT(__LargoMax * 5.6) + 10)   -- ~5,6 px por carácter a 10 px
```

### Si hay scroll, reservarle el alto

Una caja con `overflow-x:auto` y **alto automático** no crece para alojar la barra: ésta se come ~16 px y, con `overflow-y:hidden`, lo recortado es la fila de etiquetas del eje.

```dax
VAR __EspacioScroll = IF(__Entran, 0, 16)
VAR __AltoPlot = MAX(60, __Alto - <resto> - __EspacioScroll)
VAR __AltoCaja = __AltoPlot + <alto etiquetas> + __EspacioScroll
```

### Orden de las VAR

Como `__EspacioScroll` depende de `__Entran`, y `__Entran` de la cantidad de categorías, **las variables de geometría se declaran después de los datos**, no al principio de la medida.

### Ancho: eso sí es libre

Usar `%` para barras y posiciones. En SVG, `viewBox` estirado con `preserveAspectRatio='none'` más `vector-effect='non-scaling-stroke'` para que el trazo no engorde. **El texto no va dentro de ese SVG**: se deforma. Va en `div` posicionados en `%`.

---

## 6. Mobiliario de página (lienzo 1280×720)

La franja azul de la izquierda **viene dibujada en la imagen de fondo**: no crear ninguna forma. Termina en x=241.

| Pieza | Geometría |
|---|---|
| fondo | `objects.background` en `page.json`, imagen corporativa en `Fit` |
| logo | x=13,8 y=8,5 · 158,8×48,7 |
| título de página | x=261,7 y=12,3 · 936×39,5 |
| navegador (`pageNavigator`) | x=258,7 y=49,6 · 825,8×32,1 |
| rótulo de filtros | x=27,2 · 195,5×45,7 |
| marco de filtro (`shape`) | x=25,5 · 199,1×74,9 |
| control (`slicer`) | x=67,5 · 155,1×70,9, y del marco +2 |
| icono de filtro | x=29,5 · 34×34, y del marco +20,5 |
| botón limpiar (`ClearAllSlicers`) | x=70 y=584,6 · 110,9×37,9, sin bookmarks |
| fecha de actualización | x=31,2 y=636,4 · 187,1×63,8 |

Los filtros van en **pasos de 78**: primer marco en y=124,6, luego 202,6 · 280,6 · 358,6.

**Banda de contenido:** de x=261,7 a 1241,6, o sea **979,9 px**. Primera fila en y=94,8, separación entre filas de 10 px. Distribuciones que cierran exactas: 979,9 · 485+485 · 239×4.

**El navegador tiene que listar todas las páginas.** Es `objects.pages`, con `showByDefault:false` y una entrada `showPage:true` con `selector.id` por página. Al agregar o quitar una hoja hay que actualizarlo en **todas**, o desde alguna no se podrá navegar a ella.

---

## 7. La franja de filtros

**Antes de llevar la franja, comprobar que los segmentadores filtran de verdad el contenido de la hoja.** Un segmentador sobre una tabla que no tiene relación con las tablas que alimentan los gráficos es un control que no hace nada, y es peor que no ponerlo. Revisar `relationships.tmdl`.

Si los segmentadores del origen no sirven y el usuario no pidió otros, **la hoja va sin filtros**: la maqueta sí, los controles no.

**Los controles se clonan del segmentador ya estilado, no del original de la hoja vieja.** Clonar el viejo arrastra su formato blanco y la franja queda con cajas blancas flotando sobre el azul. El molde correcto lleva: fondo del contenedor apagado, texto e ítems en blanco, ítems sobre `#2C57A5`, borde y título ocultos. Se copia ese y se le cambia solo el campo, el `queryRef`, el `nativeQueryRef` y el `displayName`.

**Las etiquetas van cortas y en mayúsculas**, porque en 155 px un texto largo se recorta. Bajar la fuente del encabezado a 9 y activar `titleWrap` ayuda, pero acortar es lo que resuelve.

**Verificar que cada segmentador apunta a la columna que su etiqueta promete.** Es un error frecuente del origen: dos controles rotulados distinto sobre la misma columna. Se señala con la columna correcta identificada y **decide el usuario**, porque cambiarlo mueve el filtrado.

**Cada control lleva su icono**, del juego `HTML_IconoFiltro_*`. Se arman con `DS_BadgeFiltro_Ini` + los `path` + `DS_BadgeFiltro_Fin`: fondo translúcido en el div del 100%, svg de 22×22 y trazo blanco **por figura**, no en el `<svg>`.

Los segmentadores comunes a varias hojas se sincronizan con `visual.syncGroup`, que va **dentro** de `visual`.

**Un cuadro de búsqueda por texto** se resuelve con un segmentador nativo con `general.selfFilterEnabled`, no con un visual custom: el custom no acepta el formato corporativo y su campo de entrada no se puede tematizar. Cambia la mecánica —se busca en la lista y se selecciona, en vez de filtrar por subcadena al tipear—, así que **confirmarlo con el usuario**.

---

## 8. Tabla de detalle: se queda nativa

Para exportar datos crudos **la tabla se deja nativa**, porque los visuales HTML no ordenan, no redimensionan columnas y no exportan. Se estila con los valores del modelo de tabla:

| | |
|---|---|
| cabecera | fondo `#FAFBFD`, texto `#5B6B8C`, negrita, 9 px |
| celdas | texto `#3D4D68`, fondo blanco, 9 px |
| divisores | solo horizontales, `#EEF3FA` a 1 px |
| contorno | `#DCE6F5` |
| relleno de fila | 3 px |
| tarjeta | fondo blanco, borde corporativo con radio 8, sin sombra |
| apagado | líneas verticales, bandeado de filas, fila de totales, cabecera del visual |

Lo único que no se puede replicar del modelo HTML es la cabecera fija al hacer scroll (`position:sticky`): la nativa tiene su propio comportamiento.

---

## 9. Mecánica PBIR

- Estructura: `<informe>.Report/definition/pages/<pageId>/visuals/<visualId>/visual.json`, más `page.json`, `pages.json` y `report.json`.
- **El visual HTML Content debe declararse** en `report.json`, o no carga ninguno:
  ```json
  "publicCustomVisuals": [ "htmlContent443BE3AD55E043BF878BED274D3A6865" ]
  ```
  Ese GUID es el `visualType` de cada `visual.json`.
- **`filterConfig` va en la RAÍZ del contenedor**, no bajo `visual`. `syncGroup` va **dentro** de `visual`.
- Un `filterConfig` permite que **una medida sirva a N visuales** con filtros distintos cableados: es el mecanismo de `DS_Tamano[Alto]` y de la tabla de anchos.
- **No dejar filtros muertos**: si el motor no lee `[DS_Alto]`, su visual no debe declarar `Alto`. Confunde a quien copie el visual.
- **No dejar customs declarados sin usar** en `publicCustomVisuals`: es una dependencia que nadie invoca.
- Al borrar una página, quitarla de `pageOrder` en `pages.json`, revisar `activePageName` y el navegador.
- **Las páginas de tooltip no se pueden reemplazar por visuales HTML**, porque no disparan tooltips de página. Antes de borrarlas, comprobar que ningún visual las referencia.
- Los `.json` del informe usan **CRLF**; los `.tmdl` también. Normalizar al escribir.
- `lineageTag`: validar `medidas + columnas = tags` y cero duplicados. No inventar GUID sin comprobar que están libres.

---

## 10. Lo que los visuales HTML NO pueden hacer

- **No filtran al hacer clic.** El filtrado va por segmentadores.
- **No disparan tooltips de página.** Si el contenido no depende de la fila va al pie del visual; si depende, al atributo `title`.
- **No ordenan ni redimensionan columnas.** Para datos crudos, tabla nativa (§8).
- **No propagan su altura al HTML** (trampa 2).
- **No cargan recursos externos**: los iconos se dibujan a mano con `<path>`, no hay librerías.

---

## 11. Power BI Desktop

- **Cerrarlo antes de editar** cualquier `.tmdl` o `.json`: si está abierto, pisa los cambios al guardar. Comprobar los procesos `PBIDesktop` y `msmdsrv` antes de escribir y **abortar** si están.
- **Al guardar, Desktop reescribe la geometría**: aparecen alturas como 259,9 o 313,5 en vez de 250 o 300 (los números salen con denominador 41, típico de un factor de zoom). Con el alto declarado eso desfasa el `Alto` unos 10-15 px; el síntoma es una franja en blanco abajo del gráfico. Reencuadrar cuando se note.
- Desktop **reformatea el TMDL** al guardar y colapsa medidas de una sola expresión a una línea: volver a leer el archivo antes de editar con anclas de texto.
- **Convertir PBIP a `.pbix` es una operación de Desktop** (Archivo → Guardar como). No se puede hacer editando archivos.

---

## 12. Antes de dar por terminado

1. ¿Ningún token de estilo tiene comilla simple? (trampa 1)
2. ¿El alto sale de un número y no de un `height:100%` inerte? (trampa 2)
3. ¿Todas las tarjetas de la fila declaran el mismo alto y coincide con el tamaño real del visual?
4. ¿El contenido reparte el alto, sin blanco dentro de la tarjeta?
5. ¿Los umbrales de categorías se calculan del ancho real, no de una constante?
6. ¿Reserva de ~16 px cuando hay scroll horizontal?
7. ¿Reacciona a los segmentadores sin `ALL` ni `REMOVEFILTERS`? Usar `TREATAS` + `KEEPFILTERS`.
8. ¿Las categorías son solo las que tienen datos, nunca una lista fija?
9. ¿Mensaje de "sin datos" en vez de recuadro vacío?
10. ¿Todos los `FORMAT` numéricos que van a HTML llevan `"en-US"`? Si no, el separador decimal rompe el CSS.
11. ¿Los nombres de categoría se ven completos, sin `…`? Columna en `max-content` + `overflow-x:auto`.
12. ¿`DISTINCTCOUNT` de la clave de la entidad, nunca `COUNT` de filas? Si el origen usa `COUNT`, **verificarlo con el usuario antes de cambiarlo**: puede que no haya duplicados y entonces no hay nada que corregir.
13. ¿Cada segmentador apunta a la columna que su etiqueta promete, y filtra de verdad las tablas de la hoja?
14. ¿El navegador lista todas las páginas?
15. ¿Ninguna medida quedó huérfana y ningún visual referencia una medida inexistente?
16. ¿Ningún visual se sale del lienzo? ¿Ningún solape? ¿Ningún filtro muerto?
17. ¿Paréntesis, corchetes y comillas balanceados, y cero LF sueltos en los archivos tocados?
18. Probado con **una sola** categoría, con valores todos iguales, con blancos y con duraciones > 24 h.

---

## 13. Forma de trabajar

Cada cambio en `.tmdl` se hace con **reemplazo acotado al bloque de la medida** y **conteo obligatorio de coincidencias**: si el patrón no aparece exactamente las veces esperadas, abortar sin escribir. Varios motores comparten texto idéntico —los de barras son un caso típico—, así que un reemplazo global toca lo que no debe.

Respaldar antes de escribir y **validar después**: recuentos de medidas y tags, balance de paréntesis y comillas, finales de línea, y que cada medida referenciada por un visual exista.

Cuidado al delimitar el bloque de una medida: los comentarios `///` que preceden a la medida **siguiente** caen dentro del rango si se corta en la línea `measure`. Hay que descartarlos antes de analizar el contenido, o se atribuyen al motor anterior.

Al terminar, **dejar escrito en la plantilla lo aprendido**: un modelo nuevo acordado va al CATÁLOGO y a `VOCABULARIO.md`; una trampa descubierta va a la sección de diagnóstico del `README.md`. Si se corrige un defecto de un motor de la plantilla, corregirlo **también ahí**, o el próximo que lo copie se lleva el mismo error.
