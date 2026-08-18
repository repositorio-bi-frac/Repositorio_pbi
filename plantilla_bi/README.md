# Plantilla BI Fractalia — biblioteca de componentes reutilizables

Proyecto Power BI en formato PBIP, autocontenido y sin credenciales. Su objetivo es servir como referencia visual y técnica para nuevos proyectos BI: se solicita un componente por el nombre funcional mostrado en el CATÁLOGO y se reutiliza su visual, adaptando únicamente el cálculo DAX y los campos al modelo destino.

## Páginas

| Página | Uso |
|---|---|
| HOME | Navegación corporativa mediante hexágonos hacia el catálogo y el modelo de página. |
| CATÁLOGO | Biblioteca de componentes con nombres funcionales reutilizables. |
| MODELO DE PÁGINA | Maqueta corporativa completa para clonar al iniciar una página de informe. |

## Forma de uso

1. Abrir `plantilla_bi.pbip` en Power BI Desktop.
2. Localizar el componente por su nombre funcional en CATÁLOGO.
3. Consultar `VOCABULARIO.md` para conocer su medida técnica y reglas.
4. Copiar el visual y los tokens `DS_*` requeridos al proyecto destino.
5. Adaptar la tabla, clave, dimensiones y medidas DAX; no rediseñar el componente.
6. Validar filtros, valores vacíos, tamaño y comportamiento en Desktop y Power BI Service.

## Si el componente sale sin marco y con el contenido apelotonado arriba

Son las dos trampas del visual HTML Content. No dan error: el visual renderiza igual, solo mal. Están explicadas en `VOCABULARIO.md`, sección *Dos trampas que rompen en silencio*, y el orden de comprobación es este:

1. **Comillas simples en los tokens de estilo.** El CSS se inyecta como `style='...'`; una comilla simple en un token (el caso típico es `font-family:'Segoe UI'`) cierra el atributo y descarta el resto del CSS, incluido el `display:flex` del que dependen las alturas.
2. **`height:100%` no da altura.** El visual no propaga su alto al HTML. El alto sale siempre de un número: cableado en el DAX, o declarado por el visual en su `filterConfig` sobre `DS_Tamano[Alto]`.

## Si aparece scroll donde había sitio, o si las etiquetas del eje no se ven

Ningún umbral de categorías va como número fijo: se calcula con el ancho real del contenedor, porque un `<= 12` traído de otro gráfico saca scroll en un visual ancho que tenía sitio de sobra. Y cuando el scroll sí corresponde, hay que reservarle el alto: una caja con `overflow-x:auto` y alto automático no crece para alojar la barra, se come ~16 px del contenido y lo que desaparece es la fila de etiquetas del eje. Las dos fórmulas están en `VOCABULARIO.md`, sección *Cuántas categorías entran*.

Al portar tokens a otro informe, revisar el punto 1 antes que nada.

## Componentes destacados

- Tarjeta KPI por estado: resuelve automáticamente etiqueta e icono para Progreso/En curso, Asignado, Pendiente, Cerrado y Cancelado.
- Columnas apiladas por periodo y Gráfico combinado de volumetría: reservan espacio para que los valores no queden recortados.
- Gauge semicircular, Matriz con medidas como columnas y Chips de categorías: enlazados a visuales HTML Content reales.
- Columnas agrupadas con series por medida: compara varias medidas lado a lado sobre el mismo eje de periodo. No confundir con las columnas apiladas, que parten una sola medida por una dimensión y cuya altura total sí se lee como suma.

## Identidad corporativa

Se reutilizan el fondo corporativo, Home con hexágonos, panel lateral, navegación, KPIs, contenedores, paleta, iconografía y tipografía. Los nombres técnicos permanecen internos; los usuarios solicitan los componentes por su nombre funcional.

## Datos

La plantilla utiliza datos ficticios deterministas generados con DAX. Al migrar a un informe real, se reemplazan o repuntan las tablas y medidas, conservando los componentes visuales y tokens.