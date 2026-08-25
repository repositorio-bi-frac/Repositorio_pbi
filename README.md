# Repositorio_pbi

Repositorio para almacenar plantillas, dashboards, modelos de datos, documentación y recursos de Power BI apoyados con Inteligencia Artificial.

---

## Por dónde empezar

| Si querés… | Leé |
|---|---|
| **Instalar el skill** en Claude Code | [`plantilla_bi/skill/INSTALAR.md`](plantilla_bi/skill/INSTALAR.md) |
| **Saber cómo usarlo** en el día a día | [`plantilla_bi/skill/USO.md`](plantilla_bi/skill/USO.md) |
| **Consultar un componente** y cómo pedirlo | [`plantilla_bi/VOCABULARIO.md`](plantilla_bi/VOCABULARIO.md) |
| **Entender la plantilla** y su forma de uso | [`plantilla_bi/README.md`](plantilla_bi/README.md) |

## Contenido

### `plantilla_bi/`

Plantilla BI corporativa en formato **PBIP**, autocontenida y **sin credenciales**: los datos son ficticios y deterministas, generados en DAX, así que se abre en Power BI Desktop sin pedir conexión y las capturas siempre coinciden.

Trae **17 modelos de gráfico** hechos con el visual HTML Content, los tokens de diseño `DS_*` y la maqueta estándar de página. Tres páginas:

- **HOME** — navegación corporativa
- **CATÁLOGO** — muestrario de componentes con su nombre funcional
- **MODELO DE PÁGINA** — maqueta completa para clonar al iniciar una página

La forma de trabajar es **pedir el componente por su nombre funcional**, copiar su visual y su medida, y repuntar tabla, clave, dimensiones y medidas al modelo destino. El componente no se rediseña.

### `plantilla_bi/skill/`

El skill **`fractalia-bi-template-migrator`** para Claude Code. Es la guía de criterio que hace que la reutilización salga homologada: qué modelo corresponde a cada visual nativo, la geometría ya decidida, las trampas conocidas y la checklist de validación.

Su principio es **reutilizar el modelo que más se asemeje y no crear nada nuevo sin confirmación expresa**.

El skill y la plantilla **viven juntos y se actualizan en el mismo commit**: si se desincronizan, el skill empieza a mandar a copiar componentes que ya cambiaron.

## Antes de clonar

El repositorio incluye un `.gitattributes` que **fuerza CRLF** en los archivos del proyecto PBIP (`.tmdl`, `.json`, `.pbip`, `.pbir`, `.pbism`, `.platform`). Es necesario: el parser de TMDL no acepta finales de línea LF, y sin esa regla quien clone en macOS o Linux recibiría un proyecto que no abre.

No hace falta configurar nada; la regla viaja en el repositorio.

## Al aportar

- Un **modelo nuevo** se acuerda antes, y se agrega a la plantilla con su entrada en el CATÁLOGO y su fila en `VOCABULARIO.md`.
- Una **trampa descubierta** va a la sección de diagnóstico del `README.md` de la plantilla.
- Un **defecto corregido** en un motor de la plantilla se corrige ahí, no solo en el informe donde apareció.
