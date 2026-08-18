# Contexto completo: continuar el desarrollo del pbip SOC Telefónica en una sesión nueva

Este documento es un prompt/briefing para pegar como primer mensaje en una sesión NUEVA de Claude Code dedicada exclusivamente al desarrollo de este reporte Power BI. La sesión original (proyecto `F:\prueba`) sigue existiendo pero de ahora en adelante se dedica solo al ETL/Postgres de `fractalia-data-platform` — este reporte se separa a su propia sesión.

## 1. Qué es este proyecto

Migración del reporte Power BI **"B2B SOC Telefonica.pbix"** (modelo antiguo, con transformaciones pesadas en Power Query) a una **plantilla de marca nueva**, en formato **PBIP** (Power BI Project — carpeta de texto/JSON, no un `.pbix` binario), conectado **directo a Postgres** sin transformaciones: las tablas ya llegan limpias desde `hechos.tb_fact_remedy_abiertos_mesa_soc_gold` / `tb_fact_remedy_cerrados_cancelados_mesa_soc_gold` (proyecto `fractalia-data-platform`, subproyecto `soc`).

## 2. Ubicaciones clave

- **Proyecto PBIP** (donde se trabaja): `C:\Users\juanjair.macedo\Downloads\soc_telefonica_pbip\` — **fuera de cualquier repo git**, decisión explícita del usuario.
- **Pbix de referencia de diseño** (extraer plantillas reales de ahí, NUNCA inventar el schema JSON de Power BI de memoria):
  - `C:\Users\juanjair.macedo\Downloads\Informe Gestion Reporting.pbix` — mismo formato/plantilla de marca, con visuales reales funcionando (cards, slicers, tableEx, pivotTable, gráficos, pageNavigator, y la página "Home" con menú de hexágonos). Es un ZIP normal — se puede leer con `zipfile`/`json` en Python sin abrir Power BI Desktop.
  - `C:\Users\juanjair.macedo\Downloads\modelo_pbi.pbix` — plantilla en blanco original (solo fondo de marca).
  - `C:\Users\juanjair.macedo\Downloads\B2B SOC Telefonica.pbix` — el modelo antiguo que se está reemplazando (usar `pbixray` para leer Power Query/DAX si hace falta comparar una fórmula original).
- **Fuente de datos**: Postgres `4.210.66.89:5432`, base `bd_ecu_jaguar_nocb2bpe`, esquemas `hechos`/`dimension`. Repo del ETL que la puebla: `F:\GITLAB\fractalia-data-platform` (rama del trabajo de la sesión original — NO tocar esa parte desde esta sesión nueva salvo que el usuario lo pida explícitamente).

## 3. Estado actual del modelo semántico (COMPLETO, no se necesitan más tablas)

- 2 tablas gold (`tb_fact_remedy_abiertos_mesa_soc_gold`, `tb_fact_remedy_cerrados_cancelados_mesa_soc_gold`) + 3 dimensiones (`tb_dim_priority`, `tb_dim_urgency`, `tb_dim_status`), relacionadas por FK.
- `Tabla_Combinada`: tabla calculada `UNION` de las 2 gold, con columnas de fecha truncada (`fecha_apertura_dia`, `fecha_cierre_dia`) para relacionarse con `Calendario`, y `HoraCierre`.
- `Calendario`: tabla calculada DAX (`CALENDAR`), relación activa sobre apertura, inactiva sobre cierre (`USERELATIONSHIP` en la medida `Cerrados`). Columnas: `Anio`, `Mes`, `NombreMes` (con `sortByColumn: Mes` para que ordene cronológico, no alfabético), `Dia`, `NombreDia`, `Date`.
- `FTE`: roster manual (`Nombre en Remedy` → `Rol`/`Grupo de Soporte`/`Empresa`), tabla calculada `DATATABLE` con 24 filas, relacionada `Tabla_Combinada[assignee] → FTE[Nombre en Remedy]`. **Grupos de Soporte reales confirmados**: `ESPECIALISTAS`, `OPERACIONES SOC`, `SOPORTE SOC`, blanco/"(Sin grupo)".
- `Actualizacion`: tabla de 1 fila con la hora de última actualización.
- `Medidas`: ~24 medidas DAX (Abiertos/Cerrados/Balance/RT_Balance, percentiles Q0-Q4, títulos dinámicos por página, `HTML_Volumetria_Apilado`, etc.).
- Todas las columnas con `summarizeBy: none` — las agregaciones son medidas DAX explícitas, nunca implícitas.

## 4. Páginas del reporte (7 totales, todas construidas)

| Página | ID interno | Contenido |
|---|---|---|
| Home | `4d46897cc501616acc62` | Menú de navegación con 6 hexágonos (estándar corporativo Fractalia BI — ver sección 6) |
| VOLUMETRIA | `c0848bc9cbcc7973402e` | Volumetría por año/mes/día/grupo de soporte, tabla "En Gestión" |
| BACKLOG | `52eb6090557ce072c53f` | Balance de producción, backlog acumulado, KPI por estado |
| REQ LOCATION WORLD_EC | `25c61dc8a5917d3d0b72` | Registro y tiempos de resolución de Requerimientos |
| INC LOCATION WORLD_EC | `1a363ba07bd9a1d5b5c7` | Igual que REQ, para Incidentes (+ vista comparativa) |
| ACT | `573eb9790efb9e6be17a` | Igual que REQ, para Actividades (+ vista comparativa) |
| VISION_TELEFÓNICA | `d40de5b6399cbb90d6a8` | Volumetría por agente, categorías, SIEM/Cyberthreats/SOC por cliente |

**Fuera de alcance, confirmado por el usuario**: página "Proyecciones" (7 medidas de regresión lineal del modelo antiguo) — sus cálculos ya estaban rotos en el pbix original, no se migra.

**"Página 2"** (plantilla en blanco original) fue **eliminada** del proyecto a pedido del usuario — si aparece mencionada en algún archivo viejo, ya no existe.

## 5. Metodología de trabajo de este proyecto (MUY IMPORTANTE — actuar igual en la sesión nueva)

1. **Nunca inventar el schema JSON de un visual de Power BI (PBIR) de memoria.** Antes de construir un tipo de visual nuevo, extraer un ejemplo REAL ya funcionando de `Informe Gestion Reporting.pbix` (es un ZIP, leer con Python) y adaptarlo — cambiar campos/colores/posición, pero mantener la estructura exacta del JSON real.
2. **Cuando un campo del modelo antiguo no tiene mapeo obvio a la fuente nueva, preguntar al usuario** — no asumir. Ejemplos ya resueltos así: "Agente Cierra"→`assignee`, "Agente Apertura"→`submitter`, "TIPO"→`tipo_definitivo`.
3. **Si se detecta un bug/inconsistencia en el modelo antiguo** (ej. un filtro copiado mal al clonar una página), mostrárselo al usuario y preguntar si corregirlo o replicarlo tal cual — no decidir en silencio.
4. **Construir en lotes y pedir validación en Desktop** antes de seguir con el siguiente lote de visuales. El usuario abre el proyecto en Power BI Desktop, prueba, y reporta el resultado o el error exacto.
5. **Cuidado con ediciones concurrentes**: Power BI Desktop reescribe los archivos del proyecto al abrir/guardar. Si Desktop está abierto mientras el asistente edita un archivo, pedirle al usuario que cierre Desktop primero (sin perder cambios sin guardar), confirmar que está cerrado, y recién ahí editar.
6. **Generar los archivos JSON con scripts Python** (no escribirlos a mano uno por uno) cuando hay que crear varios visuales parecidos — mucho más confiable y reproducible. Guardar esos scripts en el scratchpad de la sesión.
7. **Para íconos/imágenes nuevas**: copiar el `.png` a `soc_telefonica.Report/StaticResources/RegisteredResources/` **Y ADEMÁS** agregar la entrada correspondiente en `soc_telefonica.Report/definition/report.json` → `resourcePackages[1].items[]` (`{"name":"...", "path":"...", "type":"Image"}`) — si se omite este segundo paso, Power BI muestra "imagen no encontrada" aunque el archivo exista. Catálogo de íconos disponibles en la memoria `reference_iconos_fractalia_bi_landing.md`.
8. **Cuando una propiedad de formato/DAX no está documentada** (ej. alineación de texto en un textbox), no seguir adivinando el valor — pedirle al usuario que la aplique a mano en Desktop, guarde, y leer el archivo resultante para ver el valor real que escribió Power BI.
9. **Para gráficos con el visual "HTML Content"**: usar SIEMPRE `<div>` + CSS flexbox/grid, **NUNCA SVG con `viewBox`/`preserveAspectRatio`** para contenido que dependa de filtros dinámicos — ver el patrón completo y la razón en la memoria `reference_html_content_chart_pattern.md` (fue un problema real, debugueado a fondo). El HTML sale de una medida DAX conectada al rol "Values" del visual; no acepta HTML fijo en el panel de propiedades. La versión certificada del visual bloquea `<script>` y URLs externas — todo tiene que ser HTML/CSS/SVG puro, sin JavaScript.
10. **Guardar en memoria** cada decisión de diseño, mapeo de campo confirmado, o bug encontrado — usar el sistema de memoria de Claude Code normalmente.

## 6. Convenciones de diseño establecidas

- **Colores de marca** (literales hex, no `ThemeDataColor`): `#0F2D68` (navy, títulos), `#1E4FAF` (azul marino, bordes), `#4A7DFF` (azul claro, acentos), `#0E9AA7` (teal), `#DCE8FF` (fondo de página, celeste muy claro), `#F0883E` (warning), `#16A34A` (good).
- **Layout de página de contenido**: slicers apilados en la barra lateral izquierda (`x=27.5`, `width=195.1`, `height=70.9`, separados ~72-80px verticalmente), contenido principal desde `x=261.6` (`width≈980`), título arriba (`x=261.6,y=12,height=40`, medida DAX, no texto fijo), `pageNavigator` justo debajo del título (`x≈264,y≈57,height≈32,width≈825`) — **agregar cada página nueva a los `pageNavigator` de TODAS las páginas existentes**, no solo a la nueva.
- **Home**: es SOLO un menú de navegación (estándar corporativo Fractalia BI, skill `fractalia-bi-corporate-dashboard-designer`) — hexágonos de forma/tamaño/espaciado fijos (149x126, patrón panal 2 filas x 3 columnas), navegación vía `visualLink` nativo (`type: PageNavigation`) en el shape Y en la imagen, no `actionButton`. Fondo `Fondo Fractalia BI.png` con `transparency: 30D`. Título "SOC TIGO" debajo del logo (que está horneado en la imagen de fondo).
- **Fondo de páginas de contenido**: `Fondo Fractalia BI-paginas.png`, `scaling: 'Fit'`, `transparency: 0D`.
- **Tarjetas/gráficos nativos**: borde `#1E4FAF` redondeado (`radius: 8D`) + `dropShadow` sutil — mismo acabado en todos los visuales nuevos.

## 7. Bugs reales encontrados y corregidos en el modelo antiguo (no volver a introducirlos)

- Filtro de "SOC POR CLIENTE" copiado mal (filtraba SIEM en vez de SOC) al clonar de otra página — corregido.
- Tarjetas Q0-Q4 de las páginas INC/ACT heredaban el filtro de Requerimiento en vez de su propio tipo — corregido (decisión confirmada del usuario, no replicar el bug).
- Fondo de página con `scaling: Normal` en vez de `Fit` (imagen desalineada) — corregido en todas las páginas.

## 8. Pendiente / próximos pasos posibles

- Convertir más gráficos nativos al visual "HTML Content" (el usuario pidió avanzar más rápido en esto — usar la plantilla de `reference_html_content_chart_pattern.md`).
- Evaluar (pausado, no iniciado) construir un custom visual `.pbiviz` real si se necesita JavaScript/interactividad que "HTML Content" no permite.
- Cuando el desarrollo esté conforme, el usuario genera el `.pbix` final manualmente desde Desktop (Archivo → Guardar como → Informe de Power BI) — Claude no puede generar ese binario.

## 9. Memoria persistente ya disponible

Si esta sesión nueva corre en el mismo entorno de Claude Code, ya debería tener acceso automático a estos archivos de memoria (no hace falta pedírselos al usuario, pero si por algún motivo no aparecen, este documento ya cubre lo esencial):
- `project_soc_pbi_redesign.md` — historial completo y detallado de todo el desarrollo (la fuente más completa, cronológica).
- `reference_html_content_chart_pattern.md` — plantilla DAX/CSS para gráficos HTML Content.
- `reference_iconos_fractalia_bi_landing.md` — catálogo de íconos para hexágonos.
