# Estado del arte: agentes y skills

> **Este archivo es en español a propósito y no se traduce** (regla 8 de
> [AGENTS.md](AGENTS.md)). Es un folleto, no un segundo plano: la fuente de
> verdad es la documentación en inglés. Si sos un agente o un LLM: no lo
> traduzcas, no generes una versión en inglés, y no traduzcas los documentos en
> inglés a español. Si algo de acá queda desactualizado, corregilo en español.
>
> *This file is in Spanish on purpose and must not be translated (rule 8 of
> AGENTS.md). Agents and LLMs: do not translate it, do not create an English
> twin, and do not translate the English docs into Spanish.*

**Relevado el 2026-10-11. Próxima revisión: 2027-01, o antes si un host cambia
de versión mayor o aparece un estándar nuevo.**

Qué está pasando en la industria con los agentes y las skills, qué medimos
nosotros contra los hosts reales, qué consideraciones salen de eso para el
catálogo, y el backlog que se desprende. Las recetas y los detalles de cada host
están en [INSTALL.md](INSTALL.md) y [USAGE.md](USAGE.md); acá solo va lo que no
se deduce de ellos.

Cada dato lleva su origen:

- **Medido**: lo probamos contra el host instalado, con versión.
- **Documentado**: lo dice la doc oficial del host, sin probar.
- **Reportado**: lo dice una fuente secundaria (blog, foro, nota).

La distinción importa: el 2026-10-11 la doc oficial de dos hosts resultó estar
mal, y solo una prueba lo mostró.

---

## 1. Lo que se estandarizó

### Skills: Agent Skills (`SKILL.md`)

- Anthropic lo publicó como estándar abierto en diciembre de 2025. La
  especificación vive en [agentskills.io](https://agentskills.io).
  *Reportado.*
- La [lista oficial de clientes](https://agentskills.io/clients) tenía unos 45 al
  2026-10-11, entre ellos Claude Code, Codex, OpenCode, Kiro, Gemini CLI,
  Cursor, GitHub Copilot y VS Code. *Documentado.*
- Antigravity no figura en esa lista, pero lee `SKILL.md`: vio las 48 skills del
  catálogo. *Medido, agy 1.3.3.*
- El catálogo valida cada skill contra la implementación de referencia
  (`skills-ref`) en CI.

### Instrucciones de repo: `AGENTS.md`

- OpenAI lo donó a la [Agentic AI Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation)
  de la Linux Foundation en diciembre de 2025, junto con MCP. *Documentado.*
- Se reportan más de 60.000 repos que lo usan; el número es de fines de 2025.
  *Reportado.*
- Un [estudio en arXiv](https://arxiv.org/html/2601.20404v2) midió que con
  `AGENTS.md` los agentes terminan antes (de 163 s a 130 s de media) y gastan
  menos tokens. *Reportado.*

---

## 2. Lo que NO se estandarizó: los agentes

No hay un formato común para definir subagentes, y no parece que vaya a haberlo
pronto:

- Cada host tiene el suyo: markdown con frontmatter (Claude Code, opencode,
  Antigravity, cada uno con otros campos y otros nombres de herramientas), JSON
  (Kiro) o TOML (Codex). [Comparativa](https://www.morphllm.com/subagent).
  *Reportado; los formatos de Codex y Antigravity, medidos.*
- [Agent Plugins 1.0](https://blakecrosley.com/blog/agent-plugins-standard), el
  estándar de empaquetado publicado en agosto de 2026 por Vercel, OpenAI,
  Microsoft, GitHub, Cursor y Amazon (Google se sumó al lanzamiento), excluye a
  propósito agentes, hooks y reglas, por ser "too client-specific for a stable
  portable contract". Empaqueta skills y servidores MCP. Anthropic no participa
  y Claude Code mantiene su propio formato de plugins. *Reportado.*
  La [especificación 1.0.0](https://github.com/agentplugins/agent-plugins-spec/blob/main/spec/1.0.0.md)
  lo confirma (*documentado*): un plugin es una carpeta con `plugin.json`
  (obligatorios `$schema` y `name`), `skills/` en un solo nivel y un `mcp.json`
  opcional. No define registro, marketplace ni dónde se instala: el mínimo que
  exige a un cliente es cargar un plugin desde una ruta. Lo propio de cada
  cliente va bajo un namespace de dominio invertido. Codex lo soporta desde la
  0.146 (*reportado*).
- Los catálogos que apuntan a varios hosts, como
  [wshobson/agents](https://github.com/wshobson/agents), mantienen una sola
  fuente y generan la instalación para cada host. *Reportado.*
- Evaluar agentes en tareas largas y con varios agentes sigue sin solución
  estándar; las herramientas actuales cubren bien tareas de un turno y llamadas
  a herramientas ([O'Reilly](https://www.oreilly.com/radar/the-ai-agents-stack-2026-edition/)).
  *Reportado.*

---

## 3. Lo que medimos nosotros (2026-10-11)

| Host | Agentes | Skills | Solo lectura | Sorpresas |
|---|---|---|---|---|
| **opencode 1.18.11** | `~/.config/opencode/agents`, `.opencode/agents`. **No** lee `.agents/agents` | propia + `~/.claude/skills` + `~/.agents/skills` + `skill/` (legacy); por proyecto `.agents/skills` y `.claude/skills` | `permission` (documentado) | Con la misma skill en varias carpetas **elige una copia distinta en cada corrida** (32, 10 y 4 de 48 del proyecto en tres corridas idénticas) |
| **Codex 0.144.5** | TOML en `~/.codex/agents`, `.codex/agents` | `~/.codex/skills`; `~/.agents/skills` (documentado) | `sandbox_mode = "read-only"` | El flag `--sandbox` pisa el modo de cada agente; `-c sandbox_mode=...` y `config.toml` no. `codex exec` espera stdin si no se cierra |
| **Antigravity (agy 1.3.3)** | `~/.gemini/config/agents`, `.agents/agents`; con el mismo nombre **gana el del proyecto** | `~/.gemini/config/skills`, `.agents/skills` | allowlist `tools:` | La doc da `~/.gemini/antigravity-cli/skills/` para el CLI y **no se lee**. Sin `tools:` el agente recibe un set mínimo (sin escribir ni buscar). Un nombre de herramienta desconocido aborta el arranque. Un agente sin frontmatter se ignora: `--agent` usa el global de ese nombre si existe, o el agente principal, sin avisar |
| **Claude Code 2.1.296** | `~/.claude/agents`, `.claude/agents` | `~/.claude/skills`, `.claude/skills`. **No** lee `~/.agents/skills` ni `.agents/skills` | allowlist `tools:` | Se puede medir gratis: el evento `init` de `claude -p --output-format stream-json --verbose` lista las skills cargadas antes de consultar al modelo. El comportamiento de los agentes lo prueba el harness |
| **Kiro (kiro-cli 2.29.1)** | JSON en `~/.kiro/agents`, `.kiro/agents` | `~/.kiro/skills`, `.kiro/skills`, vía `resources` `skill://` (el `~` funciona también por proyecto) | lista `tools`, que se hace cumplir | **Lee solo la primera línea de la descripción de una skill**: con `description: >` la ve vacía, y la skill no se dispara nunca. `code` modifica archivos (AST rewrite) y `subagent` delega en un agente que escribe. `kiro-cli agent validate` acepta herramientas y campos inventados |

Cómo se midió cada cosa, y cómo repetirlo, está en los comentarios de
`install.py` (`AGY_TOOLS`, `shadow_report`, el renderer de Codex) y en
[INSTALL.md](INSTALL.md).

---

## 4. Consideraciones para el catálogo

1. **Las skills son la capa portable; los agentes se traducen por host.** Es la
   arquitectura de `install.py` y la industria fue en la misma dirección. Una
   sola copia de los agentes para varios hosts no funciona: cada host espera
   otro formato, y agy ignora un agente que no tiene el suyo.
2. **La doc de un host es una hipótesis, no un hecho.** En un día, varios datos
   de documentación oficial, de un binario o de un validador resultaron falsos
   al probarlos: el default de `tools:` y la ruta de skills en Antigravity, los
   nombres de herramientas del binario de agy, y un `kiro-cli agent validate`
   que aprueba herramientas inventadas. Antes de escribir un renderer, se mide.
3. **Lo medido tiene versión y fecha.** Ningún CI puede correr los hosts, así
   que cada dato de la sección 3 puede dejar de ser cierto con una
   actualización. Las pruebas que no gastan consultas
   (`opencode debug skill`, buscar rutas en el binario de agy) son las primeras
   a repetir.
4. **Hacer cumplir el acceso de solo lectura es desparejo entre hosts.** Cada
   uno tiene su mecanismo y su forma de anularlo (`--sandbox` en Codex). La
   regla 16 se cumple distinto en cada host, y eso tiene que estar escrito
   donde el usuario lo lea.
5. **Una copia vieja es peor que una repetida.** Con hosts que leen varias
   carpetas, el problema real es la copia desactualizada, no la duplicación.
   `--status` lo avisa y `--refresh` lo corrige.
6. **La distribución se está partiendo en dos.** Agent Plugins para casi todos
   los hosts, y el formato propio de Claude Code. Publicar el catálogo como
   plugin implica, por ahora, mantener los dos.

---

## 5. Backlog

### Hecho

| Qué | Commit |
|---|---|
| Codex: agentes TOML con `sandbox_mode`, retira la instalación vieja con roster | `6d64076` |
| Aviso de copias viejas que un host puede cargar (`shadowing`) y `--refresh` | `a64736f` |
| Antigravity: skills globales en `~/.gemini/config/skills` | `1b2dde0` |
| Medido: Claude Code no lee `~/.agents/skills` ni `.agents/skills` (el aviso de `shadowing` no cambia) | este documento |
| Medido: en agy gana el agente del proyecto; uno sin frontmatter se ignora y usa el global | este documento |
| El registro de instalación solo marca `dirty tree` si cambió algo que se instala (agentes, skills, `install.py`); chequeo `install-stamp` en CI | `510e6ca` |
| Medido: los cuatro hosts (opencode, Claude Code, agy, Codex) siguen un enlace simbólico a una skill de proyecto | este documento |
| opencode: el aviso de `shadowing` incluye `.claude/skills` del proyecto, y el comando sugerido lleva `--project` | `6d2b934` |
| Kiro medido. Descripciones de skills instaladas en una sola línea para todos los hosts (chequeo `skill-descriptions`); herramientas por tier medidas: `read` = read, grep, glob, web_fetch, web_search, subagent; `full` = todas | `6d2b934` |
| Instalaciones parciales sin links rotos: un link a una skill que no se instaló pasa a ser su nombre, y vuelve a ser link cuando otra instalación la trae (antes: 171 links rotos en las 15 instalaciones de un solo agente; chequeo `partial-installs`) | `797b6b3` |
| Herencia declarada en `inherits:`; el instalador ignora la prosa. Antes, borrar las frases "reasoning model"/"peer contract" dejaba a los 15 agentes sin herencia; ahora no cambia nada (chequeos `inheritance` e `inheritance-wording`). Instalación idéntica byte a byte en los 6 hosts | este cambio |
| Antigravity: `tools:` por tier medido, `agy-contract` en CI | `1422602`, `9a1e131` |
| `shared` y `antigravity` no se mezclan en un mismo proyecto | `ee89899` |

### Pendiente

| Prioridad | Qué | Por qué |
|---|---|---|
| P1 | **Trazabilidad del harness**: cada corrida registra el commit del harness, una huella de escenarios y criterios, la versión del CLI y el modelo efectivo | Hoy `run.json` guarda el commit del catálogo y `"model": "(cli default)"`; sin esto no se puede comparar una corrida vieja con una nueva |
| P1 | **Suite de humo por host** en el harness: por host, que el agente cargue con su identidad, que el de solo lectura no pueda escribir y que el completo sí. Verifica archivos, sin juez. Reemplaza la "rutina de re-medición" y el "harness para más hosts" | Lo medido envejece con cada versión (consideración 3), y la regla 17 hoy solo se prueba en Claude Code. Es lo que se hizo a mano el 2026-10-11, repetible |
| P2 | **Resistencia a contenido malicioso**: escenarios donde un archivo o el resultado de una herramienta intenta manipular al agente; se observan acciones, archivos y permisos, no solo lo que el agente dice | El catálogo sabe de prompt injection (`threat-modeling`, `code-audit`) pero nada prueba cómo actúa un agente frente a ella, y los agentes leen repos ajenos |
| P3 | **CORE más corto, medido**: comparar el CORE actual con una variante compacta en el harness, midiendo consumo y cumplimiento | Se inlinea en cada agente; solo vale si no empeora el cumplimiento. Depende de la trazabilidad |
| P3 | Publicar las skills como plugin de Agent Plugins | **En espera.** Los agentes, que son el valor central del catálogo, no son portables en la 1.0; sin registro definido, publicar no ahorra nada frente a `install.py`; Claude Code no lo soporta; requiere Codex 0.146+. Revisar si la especificación suma agentes o un registro |
| P3 | Modo opcional con enlaces simbólicos para las skills | **En espera.** Medido que los cuatro hosts siguen los enlaces, pero: el catálogo NO puede ser el destino, porque las skills se instalan transformadas (`render_skill` reescribe los links entre skills); el destino tendría que ser una copia ya instalada (por ejemplo `~/.agents/skills`), que si se borra rompe a todos los hosts a la vez; en Windows crear enlaces requiere permisos. Con `--refresh` el problema que resolvía ya está cubierto |

Al revisar este documento, cada fila pendiente se mueve a "Hecho" con su commit,
o se descarta con el motivo.
