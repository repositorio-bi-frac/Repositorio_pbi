# Plantilla BI — vocabulario y catálogo

Proyecto PBIP **autocontenido** (sin servidores ni credenciales) con **un componente por modelo**. Sirve para dos cosas:

1. Que el equipo copie piezas ya resueltas en vez de rehacer CSS y DAX.
2. Que al pedir un cambio en lenguaje corriente —"ponele fondo", "agregá un segmento de filtro", "un cuadro de barras"— quede claro qué medida y qué visual hay que crear.

Abrilo en Power BI Desktop: no pide credenciales, los datos se generan en DAX.

**Tres páginas:**

- **HOME** — menú corporativo de navegación hacia el catálogo y el modelo de página.
- **CATÁLOGO** — muestrario con nombres funcionales. Ese nombre es el contrato que se solicita en otros proyectos; el ID técnico queda documentado en este archivo.
- **MODELO DE PÁGINA** — una página armada con la maqueta estándar completa, para clonar de una sola vez.

---

## Cómo pedir las cosas

### Mobiliario de página

| Si pedís… | Se hace |
|---|---|
| **fondo** | `objects.background` en el `page.json` con la imagen `Fondo_Fractalia_BI-paginas…png` en `Fit`. La imagen **ya trae la franja azul de la izquierda dibujada**: no hay que crear ninguna forma. La franja termina en **x=241** de 1280. |
| **título de página** | `HTML_Titulo_Volumetria` en **x=261,7 y=12,3 · 936×39,5**. El texto sale de `TITULO_SOC_VOLUMETRIA_GENERAL`; la píldora de nota usa `DS_NotaFiltro_Estilo`. |
| **menú / navegador** | visual nativo `pageNavigator` en **x=258,7 y=49,6 · 825,8×32,1**. Se copia y se cambian los ids del array `pages`. |
| **rótulo de filtros** | `HTML_Rotulo_Filtros` en **x=27,2 y=32,1 · 195,5×45,7**. Icono, texto blanco y una línea de 2 px debajo. |
| **segmento de filtro** | tres visuales por filtro: `shape` de marco (**x=25,5 · 199,1×74,9**), `slicer` nativo (**x=67,5 · 155,1×70,9**) e icono HTML (**x=29,5 · 34×34**). Las y van en pasos de **78**: 78,7 / 156,7 / 234,7. |
| **icono de filtro** | `HTML_IconoFiltro_Anio`, `_Mes`, `_Dia`, `_Estado`. Se arman con `DS_BadgeFiltro_Ini` + los `path` + `DS_BadgeFiltro_Fin`. Para un icono nuevo se copia uno y se cambia el `path`. |
| **botón limpiar** | `actionButton` con `visualLink.type = ClearAllSlicers`, en **x=70 y=584,6 · 110,9×37,9**. Sin bookmarks. |
| **fecha de actualización** | `HTML_UltimaActualizacion` en **x=31,2 y=636,4 · 187,1×63,8**. |

**Banda de contenido:** de **x=261,7** a **1241,6**, o sea **979,9 px**. Primera fila en **y=94,8**.

### Los 14 modelos de gráfico

Uno por modelo. Si hace falta otro igual, se copia la medida y se le cambian la tabla, la columna y el título.

| Si pedís… | Medida | Alto cableado |
|---|---|---|
| **Tarjeta KPI por estado** con etiqueta e icono automáticos | `HTML_Backlog_KPI_EnGestion` | libre |
| **tarjeta KPI de duración** (hh:mm:ss desde un percentil) | `HTML_KPI_Q2` | libre |
| **columnas apiladas** por periodo, series por dimensión | `HTML_Volumetria_Apilado` | 290 |
| **columnas agrupadas** por periodo, series **por medida** | `HTML_Barras_Agrupadas` | 240 |
| **barras horizontales apiladas** por estado | `HTML_Backlog_Barras_AgenteEstado` | 190 |
| **barras horizontales simples** (ranking con Top N) | `HTML_Vision_Barras_Cliente` | 180 |
| **combo** barras + línea | `HTML_Vision_Combo_Volumetria` | 200 |
| **dona** con leyenda al costado | `HTML_Donut_Estados` | 280 |
| **líneas** con marcadores y valores | `HTML_Backlog_Lineas_Balance` | 190 |
| **área** acumulada | `HTML_Backlog_Area_Backlog` | 190 |
| **matriz jerárquica** de 2 niveles, periodos en columnas | `HTML_Volumetria_MatrizSoporte` | 290 |
| **matriz plana** de 1 nivel | `HTML_Volumetria_MatrizAgentes` | 180 |
| **tabla de detalle** | `HTML_Registro_Tabla` | 280 |
| **box plot de cuartiles** con descripción automática | `HTML_Cuartiles_Tiempos` | 230 |

**Apiladas o agrupadas, según de dónde salgan las series.** Es la distinción que más se confunde:

- **apiladas** (`HTML_Volumetria_Apilado`): **una** medida partida por una **dimensión**. Los segmentos se suman, y la altura total de la columna es el total del periodo.
- **agrupadas** (`HTML_Barras_Agrupadas`): **varias medidas** comparadas entre sí, lado a lado. No se suman: cada barra se lee por separado. Es el modelo para "SCCM vs PANDA" o "aperturados vs cerrados".

Si se usa el apilado para comparar dos medidas, la altura total pasa a leerse como una suma que no significa nada. Para una tercera serie en el agrupado se duplica el bloque de la barra, igual que el motor de líneas hace con sus tres.

### El alto de cada visual no es libre

Las medidas llevan la altura **cableada** en el DAX (`VAR __AltoContenedor`). Si al visual se le da más alto queda hueco abajo; si se le da menos, el contenido se corta. Son los valores de la columna de arriba.

Las tarjetas KPI, los iconos, el rótulo y la fecha de actualización **no** tienen alto cableado: se adaptan.

**No se puede migrar a `height:100%` para librarse de esa columna.** En el visual HTML Content `height:100%` **es inerte**: el visual no le propaga su altura al HTML, así que un `height:100%` resuelve contra un alto indefinido y la tarjeta colapsa al alto de su contenido. Varios motores de esta plantilla lo llevan escrito y **no hace nada**: se ven bien solo porque su contenido mide en píxeles lo mismo que el visual.

Por lo tanto **el alto siempre sale de un número**, por una de dos vías:

| Vía | Cómo | Cuándo |
|---|---|---|
| cableada en el DAX | `VAR __AltoContenedor = 290` | la de esta plantilla |
| declarada por el visual | `SELECTEDVALUE(DS_Tamano[Alto])` leído con `[DS_Alto]`, y cada visual fija el valor en su `filterConfig` | la recomendada: una medida sirve para cualquier tamaño |

Con la segunda vía, al cambiar el alto de un visual hay que cambiar **también** su filtro `Alto`, y el valor tiene que existir en la tabla `DS_Tamano`.

Lo que sí es libre es el **ancho**: usar `%` para las barras y para las posiciones, y en los SVG un `viewBox` estirado con `preserveAspectRatio='none'` más `vector-effect='non-scaling-stroke'` para que el trazo no engorde. El texto **no** va dentro de ese SVG: se deforma. Va en `div` posicionados en `%`.

### El gráfico llena la tarjeta

Un grid cuyas filas miden lo que mide su contenido deja media tarjeta en blanco: tres categorías de 21 px dentro de una tarjeta de 250. El alto disponible se reparte entre las categorías, con tope y piso:

```dax
VAR __AltoFila = MIN(60, MAX(18, INT(DIVIDE(__AltoCuerpo, __Cantidad, 0))))
VAR __AltoBarra = MAX(10, __AltoFila - 10)
```

Y el grid lo aplica con `height:100%`, `grid-auto-rows:<__AltoFila>px` y `align-content:center`, para que el sobrante se reparta arriba y abajo en vez de acumularse al final.

- **el tope** evita que con una o dos categorías la barra quede como un bloque desproporcionado
- **el piso** es el punto a partir del cual aparece el scroll vertical, en vez de comprimir la barra hasta hacerla ilegible
- si el grid lleva `row-gap`, hay que descontarlo: `(__AltoCuerpo - (__Cantidad - 1) * gap) / __Cantidad`

### Cuántas categorías entran: se calcula, no se fija

**Ningún umbral de categorías va como número fijo.** Un `<= 12` heredado de otro gráfico hace aparecer scroll en un visual ancho que tenía sitio de sobra, o recorta en uno angosto. Se calcula siempre con el ancho real:

```dax
VAR __AnchoMinPeriodo = 63                                   -- lo que ocupa una categoría
VAR __AnchoUtil = MAX(120, __AnchoContenedor - 22)            -- menos relleno y borde
VAR __Caben = MAX(1, INT(DIVIDE(__AnchoUtil, __AnchoMinPeriodo, 0)))
VAR __Entran = __CantidadCategorias <= __Caben
```

`__AnchoMinPeriodo` sale de sumar lo que de verdad ocupa una categoría: las barras, el hueco entre ellas, el relleno de la columna y, si es mayor, el ancho de la etiqueta del eje. En el modelo de columnas agrupadas son 63 px (dos barras de 26, hueco de 3, relleno de 8).

De dónde viene `__AnchoContenedor`, las mismas dos vías que el alto:

| Vía | Cómo |
|---|---|
| cableada | `VAR __AnchoContenedor = 400` — la de esta plantilla. Al cambiar el ancho del visual hay que cambiarla. |
| declarada por el visual | tabla desconectada de anchos leída con `SELECTEDVALUE`, y cada visual fija el valor en su `filterConfig`. Un mismo motor sirve en un visual de 239 px y en uno de 980. |

### Y si hay scroll, reservarle el alto

Una caja con `overflow-x:auto` y **alto automático** no crece para alojar la barra de desplazamiento: ésta se come ~16 px del contenido y, con `overflow-y:hidden`, lo recortado es justo la fila de etiquetas del eje. Hay que reservarle el espacio y dar altura explícita a la caja:

```dax
VAR __EspacioScroll = IF(__Entran, 0, 16)
VAR __AltoPlot = MAX(60, __AltoContenedor - <resto> - __EspacioScroll)
VAR __AltoCaja = __AltoPlot + <alto de las etiquetas> + __EspacioScroll
```

Como `__EspacioScroll` depende de `__Entran`, y `__Entran` de la cantidad de categorías, **las variables de geometría van declaradas después de conocer los datos**, no al principio de la medida.

---

## Paleta y tokens

Todo el color sale de `Medidas` → carpeta `Diseno HTML\Tokens`. **Cambiar un token cambia el informe entero.**

| Token | Para qué |
|---|---|
| `DS_Fuente` | familia tipográfica |
| `DS_Color_TituloTexto` | texto de títulos y valores |
| `DS_Color_TextoSecundario` | etiquetas y notas — **declarado pero sin cablear** |
| `DS_Color_Borde` | borde de contenedores y trazo de iconos |
| `DS_Color_Divisor` | líneas divisorias — **declarado pero sin cablear** |
| `DS_Sombra` | sombra de contenedor |
| `DS_Contenedor_Estilo` | la tarjeta: marco, radio, sombra, columna flex |
| `DS_Titulo_Estilo` | título dentro de un gráfico |
| `DS_TituloPagina_Estilo` | título de página |
| `DS_NotaFiltro_Estilo` | píldora de nota al lado del título |
| `DS_SinDatos_Estilo` / `_Mensaje` / `_Conflicto` | estado vacío |
| `DS_KPI_Tile_Estilo` / `_Header_` / `_Label_` / `_Valor_` | tarjetas KPI |
| `DS_Color_Grupo_Oscuro` / `_Pastel` / `_Claro` / `DS_Orden_Grupo` | color y orden por grupo |
| `DS_BadgeFiltro_Ini` / `_Fin` | placa de los iconos de filtro |
| `DS_Icono_EnGestion` / `_Q2` / `_Actualizacion` | tres glifos de muestra |

Los dos marcados **sin cablear** están declarados pero los motores hardcodean esos colores. Al tocar un motor, conviene reemplazar el literal por el token.

### Al portar a otra paleta, mapear por FUNCIÓN y no por nombre

La lección de portabilidad de paleta. Esta plantilla usa **un solo azul medio** para bordes, iconos y acentos. Si la paleta destino tiene un navy muy oscuro y un cian claro, ninguno sirve para las tres cosas:

- **trazo fino de icono** → necesita el **oscuro**, o se lava
- **borde de caja** → necesita el **medio o claro**, o pesa demasiado

Si el destino no tiene un tono intermedio, hay que inventarlo.

### Colores de serie

La biblioteca usa el token `DS_Paleta`: una cadena delimitada por `|` que los motores leen con `PATHITEM`, en vez de repetir un `SWITCH` de 8 colores dentro de cada motor.

```dax
DS_Paleta = "#0F2D68|#1E4FAF|#4A78F6|#1198A5|#7B93D6|#8FADF9|#5FC2CE|#2F5FC4"
VAR __Color = PATHITEM([DS_Paleta], MOD(__Orden - 1, 8) + 1)
```

---

## Reglas ya decididas

1. **Contar `DISTINCTCOUNT`** de la clave de la entidad, nunca `COUNT` ni `COUNTA` de filas.
2. **Los nombres de categoría se ven completos.** Columna en `max-content` + `overflow-x:auto`. Nunca `…`.
3. **Nunca truncar en silencio.** Si hay `TOPN`, el título dice cuántos quedaron fuera. Si hace falta scroll, el título lo avisa.
4. **Los blancos llevan etiqueta explícita**: `(Sin asignar)`, `(Sin grupo)`, `(Sin dato)`, `-`.
5. **Los `FORMAT` numéricos que van a HTML llevan `"en-US"`**, o el separador decimal rompe el CSS.
6. **Calendario derivado del dato**, cubriendo **todas** las columnas de fecha relacionadas.
7. **Publicar con solo el Home visible** y navegar con el `pageNavigator`.
8. **Segmentadores comunes sincronizados** con `visual.syncGroup`.
9. **Los percentiles se calculan con `PERCENTILE.INC` dentro del motor.** No usar `FORMAT(x, "HH:NN:SS")` para duraciones: pierde los días cuando pasa de 24 h.
10. **Las tarjetas de una misma fila miden lo mismo y no dejan huecos en el lienzo.** Ver abajo.

### Política de alineación

Todas las tarjetas de una fila **declaran el mismo alto** y ese alto es el que mide el visual. Ninguna se deja colapsar al alto de su contenido: si una fila mezcla una tarjeta de 250 y otra que se autoajusta a 90, queda un hueco a media fila que se ve en cuanto el fondo no es blanco.

Las tres condiciones, juntas:

1. **El contenedor fija su alto en px**, nunca con `height:100%` — que es inerte y hace que la tarjeta mida lo que mide su contenido.
2. **El cuerpo también en px**, `alto del contenedor − marco − título`, en vez de `flex:1 1 auto`, que sin altura definida resuelve a cero.
3. **El contenido reparte ese alto** (ver *El gráfico llena la tarjeta*), para que tampoco quede blanco dentro de la tarjeta.

Y la geometría se cierra en la retícula: dentro de la banda de contenido, las columnas comparten `x` y ancho, y las filas comparten `y` y alto. El borde derecho de la última columna coincide con el fin de la banda.

---

## Lo que los visuales HTML NO pueden hacer

- **No filtran al hacer clic.** El filtrado va por segmentadores.
- **No disparan tooltips de página.** Si el contenido no depende de la fila va al pie del visual; si depende, al atributo `title`.
- **No ordenan ni redimensionan columnas.** Para exportar datos crudos, dejar la tabla nativa.
- **No propagan su altura al HTML.** Ver la sección del alto: `height:100%` no sirve.

## Dos trampas que rompen en silencio

Las dos se manifiestan igual —la tarjeta sale sin fondo ni marco y el contenido apelotonado arriba— y ninguna da error: el visual renderiza contento.

### 1. Nunca una comilla simple dentro de un token de estilo

Los estilos se inyectan como `style='...'`, con comilla **simple** de delimitador. Si un token trae una comilla simple, cierra el atributo antes de tiempo y el navegador **descarta todo el CSS que venía después**, sin avisar.

```dax
DS_Fuente = "font-family:'Segoe UI',system-ui,sans-serif;"   -- ROMPE
DS_Fuente = "font-family:Segoe UI,Arial,sans-serif;"          -- correcto
```

Con la versión rota sobrevive solo lo que va **antes** del token; se pierden `background`, `border` y, lo más difícil de diagnosticar, `display:flex` — y sin `display:flex` los hijos dejan de ser flex items, así que cualquier `flex:1 1 auto` resuelve a altura 0 y el gráfico desaparece.

Los nombres de familia con espacios son CSS válido sin comillas, así que la solución no cuesta nada. Si algún día hace falta una comilla de verdad, va como `&#39;`.

**Al portar tokens a otro informe, revisar esto primero.** Es la comprobación más barata y la que más tiempo ahorra.

### 2. `height:100%` no da altura

Ver la sección del alto. El síntoma es idéntico al de la trampa 1, por lo que conviene descartar primero las comillas y después la altura.

---

## Reglas al migrar una hoja existente

1. **No se agrega nada que no se haya pedido.** Ni segmentadores, ni rejillas, ni notas al pie, ni leyendas de más. Si algo parece faltar, se propone; no se implementa.
2. **Si la hoja original no tiene filtro, la hoja migrada no lleva filtro.** El **modelo de hoja sí va** en todos los casos: fondo, logo, título de página, navegador, franja, rótulo y fecha de carga.
3. **Antes de construir un gráfico se pregunta cuál se va a usar**, para que salga de los modelos de este catálogo y no sea una variante nueva.
4. Solo se reutilizan los modelos de la tabla de arriba. Un modelo nuevo se acuerda antes, y una vez acordado se agrega a esta plantilla para que quede homologado.

## Requisito al empezar o migrar un informe

El visual HTML Content tiene que estar declarado en `<informe>.Report/definition/report.json`:

```json
"publicCustomVisuals": [ "htmlContent443BE3AD55E043BF878BED274D3A6865" ]
```

Sin eso ningún visual carga. Ese GUID es el `visualType` de cada `visual.json`.

---

## Los datos ficticios

| Tabla | Qué es |
|---|---|
| `Tabla_Combinada` | 600 filas con `GENERATESERIES` y aritmética modular. **Deterministas**: siempre los mismos números, las capturas coinciden. 18 columnas + 2 calculadas de fecha. |
| `tb_dim_status` | 4 estados: Cerrado, Asignado, Progreso, Pendiente |
| `tb_dim_fte` | 8 personas en 3 grupos. `assigne` cruza, `assigne_normalizado` se muestra |
| `Calendario` | derivado del mínimo y máximo del dato |
| `Actualizacion` | `NOW()` |

**Al migrar un informe real se reemplaza `Tabla_Combinada` y las dimensiones**, y los motores se repuntan a las columnas del destino. Los tokens y el mobiliario no se tocan.

Los valores de `grupo_soporte` son `ESPECIALISTAS`, `OPERACIONES SOC` y `SOPORTE SOC` **a propósito**: los tokens de color hacen `SWITCH` sobre esos literales y con cualquier otro texto los grupos se van al gris.

---

## Componentes adicionales de la biblioteca

Tres modelos ya disponibles y enlazados a visuales HTML Content:

- **gauge semicircular** — `HTML_Gauge_*`, arco SVG de 180°
- **matriz con medidas como columnas** — `HTML_Matriz_Estado`; esta plantilla solo tiene "una medida partida por dimensión"
- **chips de categorías** — lista compacta de categorías con su cantidad

## Contrato de reutilización por nombre

En otros proyectos se solicita el componente por el nombre funcional visible en el CATÁLOGO, por ejemplo: **Tarjeta KPI por estado**, **Columnas apiladas por periodo** o **Matriz con medidas como columnas**.

Al reutilizar:

1. Copiar el visual HTML Content y la medida indicada en este vocabulario.
2. Conservar los tokens `DS_*` y el acabado corporativo.
3. Repuntar tabla, clave, dimensiones y medidas al modelo destino.
4. Mantener el nombre funcional visible; no mostrar IDs técnicos al consumidor.
5. Validar valores, filtros, estados vacíos y redimensionamiento en Power BI Desktop y Service.

### Estados de la Tarjeta KPI por estado

La medida de ejemplo usa `VAR __Estado = "Progreso"`. Cambiar únicamente ese literal adapta automáticamente la etiqueta y el icono:

| Estado del modelo | Etiqueta visible | Icono |
|---|---|---|
| `Progreso` | En curso | Reproducción/progreso |
| `Asignado` | Asignado | Usuario |
| `Pendiente` | Pendiente | Reloj |
| `Cerrado` | Cerrado | Verificación |
| `Cancelado` | Cancelado | Cierre |