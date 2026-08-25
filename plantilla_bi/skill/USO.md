# Cómo usan el skill los desarrolladores

Guía práctica para el día a día. Para instalarlo, ver [INSTALAR.md](INSTALAR.md).

---

## 1. Qué es y qué no es

El skill es una **guía de criterio** que se carga sola en Claude Code cuando trabajás sobre informes Power BI del estándar corporativo. No es un generador: no trae DAX adentro ni crea gráficos por su cuenta.

| Es | No es |
|---|---|
| El contrato de cómo se pide y se reutiliza cada componente | Una librería de código |
| Las reglas de geometría, color y validación ya decididas | Un asistente que improvisa diseños |
| El registro de las trampas que ya nos costaron tiempo | Un reemplazo de revisar el resultado en Desktop |

**Los componentes viven en la plantilla**, no en el skill: las medidas `DS_*` y `HTML_*` están en `plantilla_bi.SemanticModel` y los visuales en la página CATÁLOGO. El skill dice qué copiar de ahí y cómo adaptarlo.

## 2. Requisitos antes de empezar

1. **Claude Code** instalado.
2. El skill copiado en `~/.claude/skills/fractalia-bi-template-migrator/` y **Claude Code reiniciado** (el registro de skills se carga al arrancar).
3. **Acceso a la plantilla.** Clonar este repositorio y tener la carpeta `plantilla_bi` disponible en disco. La ruta no está cableada en el skill: cloná donde te convenga y, la primera vez, decile a Claude dónde quedó el clon.
4. **Power BI Desktop cerrado** cuando Claude vaya a editar archivos. Si está abierto, pisa los cambios al guardar.

## 3. El principio, para que no haya sorpresas

> **Se reutiliza el modelo que más se asemeje. No se crea nada nuevo salvo confirmación expresa.**

Vale para todo: gráficos, segmentadores, iconos, rejillas, notas al pie, columnas y páginas. Si Claude detecta que falta algo, **lo propone y espera respuesta**; no lo implementa por su cuenta.

Esto es a propósito. Un gráfico inventado, aunque sea mejor, rompe la homologación y el equipo pierde la referencia única.

## 4. Cómo pedir las cosas

Se pide por el **nombre funcional** del componente, no por el nombre técnico de la medida.

```
Migrá la hoja "Evolutivo" de este informe al estándar corporativo.
```
```
Agregá a esta página un cuadro de barras horizontales con el ranking de clientes.
```
```
Poné fondo, título de página y la franja de filtros a esta hoja nueva.
```
```
Necesito comparar dos medidas por mes: usá columnas agrupadas.
```

**Vocabulario que el skill entiende:** fondo · título de página · menú o navegador · rótulo de filtros · segmento de filtro · icono de filtro · botón limpiar · fecha de actualización · tarjeta KPI · columnas apiladas · columnas agrupadas · barras horizontales · combo · dona · líneas · área · matriz jerárquica · matriz plana · matriz con medidas como columnas · tabla de detalle · box plot · gauge · chips.

La lista completa de modelos con su medida de referencia está en [`VOCABULARIO.md`](../VOCABULARIO.md).

## 5. Qué va a pasar durante el trabajo

**Te va a preguntar antes de construir un gráfico.** No es indecisión: es la regla 3. Cuando un caso admite criterio —por ejemplo columnas verticales por dimensión, que se pueden resolver como barras horizontales o como agrupadas de una serie— te plantea las opciones con sus consecuencias y decidís vos.

**Va a respetar el origen.** Si la hoja original no tiene filtros, la migrada tampoco los lleva, aunque la maqueta sí va completa. Si una medida del informe tiene un defecto, te lo señala con el arreglo exacto pero **no lo cambia**, porque movería números que ya validaste.

**Va a avisar cuando recorte.** Si un gráfico usa `TOPN`, el título dice cuántas categorías quedaron fuera. Si hace falta scroll, el título lo avisa. Nunca se recorta en silencio.

## 6. Lo que no hay que hacer

- **No rediseñar el componente** que se copió. Se le cambia la tabla, la clave, las dimensiones y las medidas; el acabado no se toca.
- **No tocar los tokens `DS_*` de un informe** para arreglar un gráfico puntual: cambian el informe entero.
- **No pasar a HTML la tabla de detalle.** Los visuales HTML no ordenan, no redimensionan columnas y no exportan. Para datos crudos, tabla nativa estilada.
- **No dejar el `SKILL.md` suelto** dentro de `skills\`: el nombre de la carpeta es el nombre del skill.
- **No editar con Desktop abierto.**

## 7. Si el resultado se ve mal

Hay dos fallas que **no dan error**: el visual renderiza contento, solo mal. Y dan el mismo síntoma, así que se comprueban en este orden.

**Síntoma: la tarjeta sale sin fondo ni marco y el contenido apelotonado arriba.**

1. **Comilla simple en un token de estilo.** El CSS se inyecta como `style='...'`; una comilla simple dentro (el caso típico es `font-family:'Segoe UI'`) cierra el atributo y el navegador descarta el resto, incluido el `display:flex` del que dependen las alturas.
2. **`height:100%` no da altura.** El visual no propaga su alto al HTML. El alto sale siempre de un número: cableado en el DAX, o declarado por el visual en su `filterConfig` sobre una tabla desconectada de tamaños.

**Síntoma: los cuadros de una fila no terminan a la misma altura y asoma el fondo de página.** Alguna tarjeta se está dejando colapsar al alto de su contenido. Las tres condiciones van juntas: contenedor en px, cuerpo en px, y el contenido repartiendo ese alto.

**Síntoma: aparece scroll donde había sitio de sobra, o no se ven las etiquetas del eje.** Hay un umbral de categorías puesto como número fijo en vez de calculado del ancho real; y si el scroll corresponde, hay que reservarle ~16 px o se come la fila de etiquetas.

El detalle de las tres está en [`VOCABULARIO.md`](../VOCABULARIO.md) y el orden de diagnóstico en el [`README.md`](../README.md) de la plantilla.

## 8. Un detalle de Power BI Desktop que conviene saber

**Al guardar, Desktop reescribe la geometría de los visuales.** Aparecen alturas como 259,9 o 313,5 donde había 250 o 300. Con el mecanismo de alto declarado eso desfasa el tamaño unos 10-15 px y el síntoma es una franja en blanco abajo del gráfico. No es que el ajuste no funcionó: es el guardado. Se vuelve a encuadrar y listo.

También **reformatea el TMDL** y colapsa medidas de una sola expresión a una línea.

## 9. Cómo se devuelve al equipo lo que se aprende

Es la parte que sostiene la homologación en el tiempo:

- **Modelo nuevo acordado** → se agrega a la plantilla (medida + entrada en el CATÁLOGO + fila en `VOCABULARIO.md`) **antes** de usarlo en el informe destino.
- **Trampa nueva descubierta** → va a la sección de diagnóstico del `README.md` de la plantilla.
- **Defecto corregido en un motor de la plantilla** → se corrige **también ahí**, o el próximo que lo copie se lleva el mismo error.
- Todo eso se comitea a este repositorio, para que el resto lo vea.

Si el skill queda desactualizado respecto de la plantilla, empieza a mandar a copiar cosas que ya cambiaron. Por eso viven juntos en el mismo repositorio y se actualizan en el mismo commit.
