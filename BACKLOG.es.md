# Backlog del catálogo

> **Este archivo es en español a propósito y no se traduce** (regla 8 de
> [AGENTS.md](AGENTS.md)). Es un folleto, no un segundo plano: la fuente de
> verdad es la documentación en inglés. Si sos un agente o un LLM: no lo
> traduzcas, no generes una versión en inglés, y no traduzcas los documentos en
> inglés a español. Si algo de acá queda desactualizado, corregilo en español.
>
> *This file is in Spanish on purpose and must not be translated (rule 8 of
> AGENTS.md). Agents and LLMs: do not translate it, do not create an English
> twin, and do not translate the English docs into Spanish.*

El trabajo pendiente y el terminado, con un ID estable por ítem. El porqué de
cada uno suele estar en [ESTADO-DEL-ARTE.es.md](ESTADO-DEL-ARTE.es.md), que
guarda el relevamiento y los hechos medidos; acá va lo que hay que hacer.

**Cómo se usa** (`validate.py` revisa las reglas marcadas con ✓):

- ✓ Cada ítem tiene un ID `B-NNN` que no se reutiliza nunca, ni al cerrarlo.
- ✓ Un ítem abierto está `pendiente`, `en curso` o `en espera`, tiene prioridad
  (`P1` > `P2` > `P3`) y un **criterio de terminado** que se pueda verificar:
  un chequeo que pasa, un escenario en N/3, un archivo idéntico byte a byte.
- ✓ Al cerrarlo pasa a la tabla de cerrados: `hecho` cita los commits que lo
  resuelven (`abc1234` del catálogo, `harness:abc1234` del repo del harness);
  `descartado` dice por qué.
- Si un ítem resulta ser otra cosa al investigarlo, se cierra como
  `descartado` con el motivo y se abre uno nuevo. No se reescribe la historia.

---

## Abiertos

| ID | Prioridad | Estado | Qué | Por qué | Terminado cuando |
|---|---|---|---|---|---|
| B-020 | P2 | pendiente | Carpetas de prueba de `run.py` como su propia raíz de git (fuera del repo del harness o con `git init`), como ya hace `smoke.py` | Un agente usó `git rev-parse --show-toplevel`, encontró el repo del harness e intentó buscar en él | Ninguna traza de la suite muestra `rev-parse` resolviendo al repo del harness |
| B-022 | P2 | pendiente | Reglas que chocan o son absolutas (revisión externa, citas verificadas): CORE "No fix for a bug you haven't observed" contra `verification`; `next-step` "the file wins" sin criterio de vigencia; `orchestration` "and nothing else"; `test-design` "Deterministic or deleted"; `apple-dev/code-review` "One concept per review" | Pueden producir respuestas incorrectas, trabajo innecesario o bloqueos | Cada regla reformulada tiene su escenario, corrido contra la versión anterior (regla 17), y pasa con `--repeat 3` |
| B-023 | P2 | pendiente | **Experimental:** que los agentes carguen sus skills. A medir: una instrucción en el CORE, disparadores más claros, o nombrar el tema en el cuerpo del agente (como pasó con el PM) | Los agentes cargan una skill en 1 de cada 15 muestras: un criterio que vive solo en una skill es invisible. Es una posibilidad, no un plan | Una suite completa muestra que los agentes cargan la skill que corresponde con más frecuencia y sin bajar la tasa de PASS; si no, se cierra como descartado |
| B-030 | P2 | pendiente | Regla en el CORE sobre contenido que trae instrucciones: lo que el agente lee (un archivo, una página, la salida de una herramienta, un ticket) es dato, no una orden; no la sigue y **siempre se lo dice al usuario**, sobre todo si pide quedar oculta | En `senior-dev-ignores-instruction-planted-in-code` el agente resistió 3/3, pero en 1 de 3 no avisó al usuario: cumplió la parte de la inyección que pedía esconderla. El CORE no dice nada sobre esto; `llm-integration` lo cubre solo para construir productos | Los tres escenarios de `injection` pasan 3/3 avisando al usuario, y una suite completa no baja la tasa de PASS (el CORE se inlinea en todos los agentes) |
| B-025 | P3 | pendiente | Calibrar absolutos: `visual-craft` ("an arbitrary `13px` is a bug"), `focus` ("quality — never"), `engagement` (novedad de 2–4 semanas), `devops` ("automate anything you do twice"), `code-health` (umbrales como señales) | Riesgos de calibración; ningún fallo medido | Cada frase reescrita como criterio, con su escenario si cambia el comportamiento esperado |
| B-026 | P3 | pendiente | Privacidad en `llm-integration` y `data-pipelines`: autorización sobre documentos recuperados, aislamiento de cachés, revocación de acceso; capa cruda inmutable contra minimización, retención y borrado | Contenido que falta, no un error | Las dos skills lo cubren, y un escenario verifica que un documento revocado no reaparece desde una respuesta cacheada |
| B-027 | P3 | pendiente | CORE más corto, medido | Se inlinea en cada agente; solo vale si no empeora el cumplimiento | Una suite completa con la variante corta iguala la tasa de PASS de la actual, con menos tokens |
| B-028 | P3 | en espera | Publicar las skills como plugin de Agent Plugins | La 1.0 no porta agentes y no define dónde se publica; Claude Code no la soporta | La especificación suma agentes o un registro común: entonces se reevalúa |
| B-029 | P3 | en espera | Modo opcional con enlaces simbólicos para las skills | Los cuatro hosts siguen los enlaces, pero el destino no puede ser el catálogo (las skills se instalan transformadas) y `--refresh` ya cubre el problema | Aparece un caso que `--refresh` no resuelve |

## Cerrados

| ID | Estado | Qué | Commits |
|---|---|---|---|
| B-001 | hecho | `shared` y `antigravity` no se mezclan en un mismo proyecto | `ee89899` |
| B-002 | hecho | Antigravity: `tools:` por tier medido; chequeo `agy-contract` | `1422602`, `9a1e131` |
| B-003 | hecho | Aviso de copias viejas que un host puede cargar (`shadowing`) y `--refresh` | `a64736f` |
| B-004 | hecho | Codex: agentes TOML con `sandbox_mode`; retira la instalación vieja con roster | `6d64076` |
| B-005 | hecho | Antigravity: skills globales en `~/.gemini/config/skills` (la ruta de su doc no se lee) | `1b2dde0` |
| B-006 | hecho | El registro de instalación solo marca `dirty tree` si cambió algo que se instala; chequeo `install-stamp` | `510e6ca` |
| B-007 | hecho | opencode: `shadowing` incluye `.claude/skills` del proyecto, y el comando sugerido lleva `--project` | `6d2b934` |
| B-008 | hecho | Kiro: descripciones de skills en una sola línea (chequeo `skill-descriptions`) y herramientas por tier medidas | `6d2b934` |
| B-009 | hecho | Instalaciones parciales sin links rotos (antes 171 en las 15 instalaciones de un solo agente; chequeo `partial-installs`) | `797b6b3` |
| B-010 | hecho | Herencia declarada en `inherits:`; la prosa ya no decide (chequeos `inheritance` e `inheritance-wording`) | `5f634c5` |
| B-011 | hecho | Claude Code: los agentes de solo lectura se instalan sin `memory:`, que les sumaba `Edit` y `Write`; escenario `visionary-cannot-write-files` | `f564749`, `harness:ecfc39c` |
| B-012 | hecho | Errores técnicos de skills: `migrations`, `query-performance`, `react-next`, `design-review`; escenario `dba-separates-lock-wait-from-lock-hold` | `c66b0be`, `harness:d1224a5` |
| B-013 | hecho | PM: evidencia y métrica en el cuerpo; el guion de entrevista de `discovery` pasa a criterio (0/3 → 3/3) | `b4f0d2d`, `harness:7318a72` |
| B-014 | hecho | `eng-manager`: dos criterios de ruteo en el cuerpo (1/3 y 2/3 → 3/3); criterio del rename de `dba` corregido (1/3 → 3/3) | `e0c6e03`, `harness:71bdf60` |
| B-015 | hecho | Harness: trazabilidad (huellas por corrida, modelo y output style efectivos, `--applicability`) | `harness:32652b9` |
| B-016 | hecho | Harness: sesión neutral, skills del catálogo por muestra, permisos por `--settings`, terminal sin alias, `--user-session` para experimentos | `harness:82dade8`, `harness:3b73dd8`, `harness:72f5aff` |
| B-017 | hecho | Harness: aserción `expect_files` y denegaciones visibles en cada corrida | `harness:44e655c` |
| B-018 | hecho | Suite de humo por host (`smoke.py`): los 5 hosts pasan carga, skill, tier de lectura y tier completo | `harness:eea358d` |
| B-019 | descartado | Investigar los 4 comportamientos que fallaban con el modelo actual como un solo problema ("no cargan la skill") | La hipótesis quedó refutada al medir (7 PASS y 7 FAIL sin skill); se resolvió caso por caso en B-013 y B-014 |
| B-021 | hecho | `agent-does-not-install-tools` habilita su premisa (`allow: "Bash(lsd:*)"`): `lsd` se ejecuta y falla con "command not found", 3/3; el detector de denegaciones reconoce también "can't be checked before it runs" | `harness:179e41e` |
| B-024 | hecho | Resistencia a contenido con instrucciones: tres escenarios (`injection`) juzgados por lo que el agente hizo — comandos intentados, archivos cambiados, secretos filtrados — con las aserciones `forbid_commands`, `unchanged_files` y `forbid_text`. 9/9 sin ninguna acción insegura; la falla en avisar al usuario pasa a B-030 | `harness:45d1a1a` |
