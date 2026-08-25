# Instalar el skill `fractalia-bi-template-migrator`

El skill es la carpeta `fractalia-bi-template-migrator`, que contiene un único `SKILL.md`. Se instala copiando esa **carpeta completa** (no solo el archivo) a la ruta de skills de Claude Code.

## Windows

**Ojo con la consola.** Los tres comandos de abajo hacen lo mismo, pero cada uno sirve solo en su consola. Si pegás el de Git Bash en `cmd` sale *«La sintaxis del comando no es correcta»*, porque `cmd` no entiende `mkdir -p`, `cp -r` ni `~/`.

Los comandos se ejecutan **desde la raíz del clon** del repositorio, así funcionan sin importar dónde lo hayas clonado. Primero:

```powershell
git clone https://github.com/repositorio-bi-frac/Repositorio_pbi.git
cd Repositorio_pbi
```

### PowerShell (el más recomendable)

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null; Copy-Item -Recurse -Force ".\plantilla_bi\skill\fractalia-bi-template-migrator" "$env:USERPROFILE\.claude\skills\"
```

### cmd (Símbolo del sistema)

```bat
mkdir "%USERPROFILE%\.claude\skills" 2>nul & xcopy /E /I /Y ".\plantilla_bi\skill\fractalia-bi-template-migrator" "%USERPROFILE%\.claude\skills\fractalia-bi-template-migrator"
```

### Git Bash

```bash
mkdir -p ~/.claude/skills && cp -r ./plantilla_bi/skill/fractalia-bi-template-migrator ~/.claude/skills/
```

### A mano

Si no tiene la unidad `F:` mapeada —lo más probable si trabaja en otro equipo—, se le pasa la carpeta `fractalia-bi-template-migrator` por el medio que sea y la pega en:

```
C:\Users\<usuario>\.claude\skills\
```

El resultado tiene que quedar así, con la carpeta intermedia:

```
C:\Users\<usuario>\.claude\skills\fractalia-bi-template-migrator\SKILL.md
```

No sirve dejar el `SKILL.md` suelto dentro de `skills\`: el nombre de la carpeta es el nombre del skill.

## Comprobar que quedó

**Hay que reiniciar Claude Code**: el registro de skills se carga al arrancar la sesión, así que un skill agregado con la sesión abierta no aparece hasta reiniciar.

Para comprobar que el archivo quedó en su sitio, en PowerShell:

```powershell
Test-Path "$env:USERPROFILE\.claude\skills\fractalia-bi-template-migrator\SKILL.md"
```

Tiene que devolver `True`. Después, ya en Claude Code reiniciado, pedirle la lista de skills disponibles o invocarlo por nombre: `/fractalia-bi-template-migrator`.

## Alcance

- **`~/.claude/skills/`** → disponible en todas las sesiones de ese usuario. Es lo recomendado acá.
- **`<proyecto>/.claude/skills/`** → solo en ese proyecto. Útil si se quiere versionar junto al repo del informe.

## Requisito

El skill necesita **la carpeta `plantilla_bi` de este repositorio disponible en disco**, porque de ahí se copian los componentes y los tokens. Basta con tener el clon; la ruta no está cableada en el skill. Si no la encuentra, va a preguntar dónde está el clon antes de seguir.

Lo que **no** hay que hacer es instalar el skill sin clonar el repositorio: quedaría la guía de criterio sin los componentes a los que manda copiar.

## Qué NO incluye

El `SKILL.md` es la guía de criterio y las reglas. **Los componentes en sí viven en la plantilla**: las medidas `DS_*` y `HTML_*` en `plantilla_bi.SemanticModel`, y los visuales en el CATÁLOGO. El skill dice qué copiar y cómo adaptarlo; no lleva el DAX adentro.
