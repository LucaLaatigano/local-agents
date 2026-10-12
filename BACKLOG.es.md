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
| B-032 | P3 | pendiente | El chequeo `skill` de la suite de humo en opencode pasa viendo una copia GLOBAL de la skill, no la que escribió la instalación del proyecto | opencode elige una copia por nombre (a veces la global), así que ese PASS no prueba lo que el chequeo dice probar | El chequeo verifica la copia del proyecto (o se aísla de las globales) y lo dice en su detalle |
| B-034 | P2 | pendiente | Piso de modelo por escenario y pruebas de la puerta. (1) Campo `floor:` en el YAML: el modelo mínimo con el que la regla se sostiene, declarado solo con medición ×3; en una corrida con un modelo por debajo del piso el escenario se reporta como "bajo el piso", ni PASS ni FAIL, y `viz.py` lo muestra así. (2) La suite sigue corriendo en Sonnet, la base de uso diario: las hard-rules se corren ×3 en Sonnet como chequeo de release, y Opus entra solo para establecer un piso cuando Sonnet oscila, nunca como corrida de rutina. (3) Gemelo con la puerta cerrada para las hard-rules que se rompen con una herramienta, al estilo de `visionary-cannot-write-files`: `architect-slows-one-way-door` con `deny: Edit,Write` y `unchanged_files`; `agent-does-not-install-tools` con regla de permiso sobre el comando; `qa-never-fixes-product` y `dba-does-not-own-the-machine` solo si el host distingue rutas o comandos en la regla (se mide en `smoke.py`, no se asume). `security-defensive-only` y `gamification-speaks-product` son semánticas: piso medido y nada más | B-033 mostró que una hard-rule puede sostenerse en Opus y no en Sonnet (3/3 contra 1/13), y hoy eso se lee como regresión del catálogo. Regla 16 aplicada al harness: lo que una puerta del host puede enforcear se prueba sin depender del modelo; lo que no, se mide con su piso. La guía del README ("lo que guía a Sonnet guía a los más fuertes") necesita esa excepción escrita | `floor:` existe y el reporte lo muestra; las siete hard-rules tienen resultado ×3 en Sonnet con su piso anotado donde oscilan; los gemelos de architect y de instalación de herramientas pasan 3/3 en Sonnet con la puerta cerrada; el README del harness documenta piso y práctica |
| B-025 | P3 | pendiente | Calibrar absolutos: `visual-craft` ("an arbitrary `13px` is a bug"), `focus` ("quality — never"), `engagement` (novedad de 2–4 semanas), `devops` ("automate anything you do twice"), `code-health` (umbrales como señales) | Riesgos de calibración; ningún fallo medido | Cada frase reescrita como criterio, con su escenario si cambia el comportamiento esperado |
| B-026 | P3 | pendiente | Privacidad en `llm-integration` y `data-pipelines`: autorización sobre documentos recuperados, aislamiento de cachés, revocación de acceso; capa cruda inmutable contra minimización, retención y borrado | Contenido que falta, no un error | Las dos skills lo cubren, y un escenario verifica que un documento revocado no reaparece desde una respuesta cacheada |
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
| B-031 | hecho | Rediseño de los reportes del harness (`viz.py`): portada con la vigencia de la evidencia, la última línea base y la matriz de hosts; página de corrida con procedencia antes que resultados, resultado por chequeo (pass/fail/vacuous) y criterio del juez guardado. Solo modo oscuro y estimaciones a partir de corridas anteriores. Experimentos: se declaran al crearlos (`--experiment` o `experiment:` en el escenario), no corren en la suite, no cuentan para la vigencia y vencen con su ítem del backlog (`until:`). Verificado con una corrida real (run `20261012-103902`) | `harness:37ea6b7` |
| B-027 | hecho | CORE más corto, medido: `generalist/CORE.md` de 1313 a 986 palabras (regla más cláusula discriminante, sin racional ni anécdotas; SPEND como bullet propio; `senior-dev/CORE.md` intacto). Suite completa `20261012-111015` (Sonnet agente y juez): 69/73; en los 68 escenarios comunes con la línea base `20261011-220947`, 65/68 contra 60/68, costo del agente 9,32 contra 9,87 USD (−5,6 %; es el proxy de tokens, los trials no guardan `usage`). De los tres PASS→FAIL, dos fueron muestreo o duda del juez y el tercero falla igual con el CORE largo (B-033). Restaurar frases una por una ("before taking it", "and the order matters") no movió nada: antes de atribuir un rojo al prompt va el control con el prompt viejo. Las frases "Measured:" quitadas viven ahora como comentarios en sus escenarios del harness; la de SPEND no tiene escenario (promesa sin medir, pendiente de ítem) | `7c414c2`, `b6a92d3`, `harness:be7cda6` |
| B-033 | hecho | `architect-slows-one-way-door-before-editing` es inestable en Sonnet con sesión neutral (1/1 en la línea base, 0/7 con el CORE corto, 0/3 con el largo en el control `20261012-113332`, 1/3 con el harness en `44e655c`) y estable en Opus (3/3, `20261012-115241`, sin Edit, costo y camino compatible primero). Descartados con evidencia: el recorte del CORE, el `architect.md` instalado y el harness. Es capacidad del modelo; el límite queda declarado como comentario en la entrada del escenario, y la práctica de piso por escenario pasa a B-034 | `6f3f252`, `1903394`, `harness:e6e861d` |
