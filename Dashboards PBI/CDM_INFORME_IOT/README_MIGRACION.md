# CDM_INFORME_IOT: PBIX → PBIP (estrella sobre gold)

Abrir `CDM_INFORME_IOT.pbip` en Power BI Desktop. Al primer refresh pide credenciales PostgreSQL (`rpt_powerbi_01/02/03`, rol `rol_reporting_pbi`).
Parámetros del modelo: `Servidor_PostgreSQL` = `4.210.66.89:5432`, `Base_Datos` = `bd_esp_tech_iot_21000381`.

## Esquema (todo lee de `dimension` y `hechos`; silver no se expone)

| Tabla del modelo | Origen gold | Rol |
|---|---|---|
| Data_Tickets | hechos.tb_fact_tickets_udo_gold (sin cancelados) | hecho |
| Data_Llamadas | hechos.tb_fact_llamadas_iot_gold (`es_cliente_iot=1` y `n_dedup=1`) | hecho |
| Data_KPI_SLA | hechos.tb_fact_kpi_sla_iot_gold | hecho (control cruzado) |
| Data_Calendario | dimension.tb_dim_calendario (> 2022-12-31) | dim, `jTiempo` igual que el original |
| Data_Horario | dimension.tb_dim_hora | dim |
| Data_Servicio / Severidad / Status / Tipo_Ticket | tb_dim_servicio / severidad / estado_ticket / tipo_ticket _iot | dims |
| Data_Grupo_Responsable / Resolutor / Tecnico / KPI | tb_dim_grupo_responsable / resolutor / tecnico / kpi _iot | dims |
| Data_Measure, Contraseña | — | medidas y contraseña de la Home (igual que el original) |

Relaciones (todas N:1, un solo sentido): ids de gold → dimensiones. Fechas y horas de tickets: **creación activa, resolución inactiva** (`USERELATIONSHIP` en las medidas de resueltos). Llamadas → calendario por `fecha_final` (como el original).

## Qué cambia respecto al PBIX

- `Data_Tickets_Abiertos` + `Data_Tickets_Resueltos` (columna `BASE`) → un solo hecho. Abiertos = fecha de creación; Resueltos = `es_resuelto=1` por fecha de resolución.
- Se cuenta con `COUNTROWS` (un ticket por fila) en vez de `DISTINCTCOUNT(ID_TICKET)`.
- KPI: `SUM(*_cumple)/SUM(*_aplica)` de gold. Las columnas `K_*` quedan como etiquetas de esas marcas (los slicers siguen igual). `VAL_UDO=1` → `Data_Grupo_Responsable[ES_EXCLUIDO_UDO]=0`.
- Filtros `INTRA/INTERSERVICEREFERENCE = ''` → `is blank` (gold guarda NULL).
- Gráfico de llamadas por hora: categoría ahora `Data_Horario[RANGO_HORA2]` (“8-9”) en vez del número de hora.
- KPI de UDO: gold lo cuenta por ticket y con laborables (decisión documentada en `013_silver_tb_kpi_sla_iot_silver.sql`); puede diferir del PBIX.
- Se eliminaron restos huérfanos del PBIX (`All_Data`, `Data_Indicadores`, `Data_Mahou`, `Append1`: anchos de columna de tablas que ya no existían).

## notificable

`sql/bd_esp_tech_iot_21000381/022_gold_tickets_notificable.sql` (aplicado el 2026-11-18, sin commit todavía) agrega `notificable` a `hechos.tb_fact_tickets_udo_gold` y a su procedimiento de carga. En el modelo: columna `Data_Tickets[notificable]`, medida `#Mayor_Notificable` = `CALCULATE([#_Resueltos], Data_Tickets[notificable] = "Yes")` y columna NOTIFICABLE de nuevo en el detalle de la página KPIs. Tras recargar con el histórico del legado hay 104 tickets "Yes" (102 resueltos); PROVDSN, SMARTDIGITS y TOOLS vienen en NULL desde la fuente.

## Histórico de tickets (verificado 2026-11-18)

Gold tenía tickets solo desde 2024-12 porque bronce solo traía 2025-04-01..2025-05-12 del SFTP. Se cargaron además las fotos de fin de enero, febrero y marzo de 2025 del SFTP (`--sftp`), y se corrigió `silver.sp_transformar_tb_tickets_udo_silver` (`sql/.../023_silver_tickets_legado_columnas.sql`): su rama `legado` (histórico del legado `m_cdm_iot`, `silver.tb_sftp_udo_legado_iot_bronce_21000381`) pedía 4 columnas con nombre equivocado y por eso silver no recargaba. Resultado: `hechos.tb_fact_tickets_udo_gold` = 146.296 tickets, 2021-04-28 a 2025-05-10. Frente a `m_cdm_iot.tickets_abiertos/cerrados` (sin cancelados) los volúmenes mensuales coinciden en todos los meses salvo 2025-03 abiertos (-1) y 2025-04 resueltos (-2).

## Comparación PBIX original vs PBIP (2026-11-18, mismas 21 medidas por periodo)

Coinciden en todos los meses de 2023-01 a 2025-03: abiertos, resueltos, gestionados N1, proactividad, reactividad, KPI de detección temprana, tiempo de atención y restaurados N1, y mayor notificable. Diferencias de 1–2 tickets: 2025-03 abiertos (3.589 → 3.588), 2025-04 resueltos (1.635 → 1.633) y reactividad (897 → 895).

Diferencias aceptadas (decisión del usuario, 2026-11-18: **se mantiene la definición de gold**):
- **Llamadas atendidas, TME, TMA y NDA.** En la tabla legada `m_data_global.llamadas_consolidado_entrantes`, algunas llamadas COMPLETECALLER tienen `FECHA_INICIO` nula aunque sí tienen `TIMESTAMP_INICIO` (34 en enero de 2024) y la regla del legado (`SP_33`: `FECHA_INICIO IS NOT NULL`) no las contaba como atendidas. Gold calcula `fecha_inicio` desde el timestamp y sí las cuenta, de modo que recibidas = atendidas + escapadas + abandonadas. Ej. 2024-01: atendidas 199 (PBIX 165), TME 4,93 (4,75), TMA 45,8 (32,8), NDA 96,2 % (80,2 %). Llamadas recibidas, escapadas y abandonadas coinciden salvo 2024-08 (166 frente a 162) y 2025-04 (87 frente a 86).
- **KPI de información actualizada de UDO.** El PBIX muestra "-" todos los meses; gold lo calcula por ticket y con días laborables (013).
- **Gráfico de llamadas por hora:** categoría `Data_Horario[RANGO_HORA2]`.

La referencia es `CDM_INFORME_IOT.pbix` (el original de esta carpeta, con los datos guardados en el archivo) frente al PBIP nuevo.

## Pendiente

- No se ha refrescado el PBIP contra la base (sin credenciales en esta sesión): validar cifras en Desktop contra el PBIX publicado.
- `#KPI_InformacionActualizada_UDO`: gold solo tiene 33 tickets con `udo_aplica = 1` (8 cumplen), porque los tickets abiertos de KITE/JASPER/GLOBAL SIM en la última foto (2025-05-12) están casi todos en grupos excluidos. El KPI será muy pequeño o "-" según el filtro. No es un fallo del PBIP.
- Tras la recarga apareció un estado nuevo, `New` (262 tickets), con las tres marcas en 0 en `dimension.tb_dim_estado_ticket_iot`; revisar si debe contar como abierto (`es_abierto`). No afecta a las medidas del PBIP, solo a los atributos de `Data_Status`.
- `Data_Calendario` se lee con consulta nativa (`PostgreSQL.Database(..., [Query = ...])`) porque `dimension.tb_dim_calendario` es una tabla foránea (FDW) y el navegador del conector no las lista ("La clave no coincidió con ninguna fila de la tabla"). Desktop pide aprobar la consulta nativa la primera vez.
- `Data_Calendario` conserva el filtro `fecha > 2022-12-31` del PBIX, así que el informe muestra 2023 en adelante aunque gold tenga desde 2021.
