# llm-wiki

Una skill para construir y mantener wikis en Markdown persistentes e interconectadas a partir de fuentes crudas. El LLM escribe y mantiene todo el contenido de la wiki; el usuario curaduriza las fuentes, dirige el análisis y formula preguntas.

Inspirada en el [patrón LLM Wiki de Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Diseñada para [Obsidian](https://obsidian.md) como cliente principal. Compatible con cualquier LLM.

> Documentación operativa completa: [`docs/flujo-de-trabajo.md`](docs/flujo-de-trabajo.md)

---

## El problema que resuelve

RAG re-descubre las mismas relaciones desde cero en cada consulta. Nada acumula.

Esta skill compila las fuentes en una wiki una sola vez y la mantiene vigente. Los conceptos tienen páginas propias. Las páginas se enlazan entre sí. Cada afirmación lleva trazabilidad de procedencia. Cuando se archiva una respuesta a una consulta, las consultas futuras la usan como contexto. El conocimiento se compone con el tiempo.

```
RAG:      consulta → buscar fragmentos → respuesta → olvidar
llm-wiki: fuentes → compilar → wiki → consultar → archivar → wiki más rica → mejores respuestas
```

## Para quién es

Cualquier persona que trabaje con múltiples documentos y quiera una base de conocimiento estructurada en lugar de una carpeta de archivos. Sin restricción de dominio: investigación académica, análisis jurídico, inteligencia, periodismo, gestión del conocimiento personal, o cualquier campo donde las fuentes se acumulan y necesitan síntesis.

## Requisitos

- Cualquier LLM con capacidad de leer y escribir archivos Markdown (Claude, ChatGPT, Gemini, u otros)
- [Obsidian](https://obsidian.md) como cliente principal (opcional pero asumido por las convenciones de la skill)
- Sin ejecutables, sin scripts, sin dependencias externas

## Instalación

### Claude (claude.ai / Claude Code)

**Opción A — npx skills:**
```bash
npx skills add git@github.com:hypr-lupo/llm-wiki.git
```

**Opción B — manual:**
```bash
mkdir -p ~/.claude/skills/llm-wiki
curl -fsSL https://raw.githubusercontent.com/hypr-lupo/llm-wiki/main/SKILL.md \
  > ~/.claude/skills/llm-wiki/SKILL.md
```

### ChatGPT / Gemini / Otros LLMs

Pegar el contenido de `SKILL.md` en el system prompt o en las instrucciones personalizadas del modelo.

## Uso

Activar la skill diciéndole al LLM:

| Comando | Acción |
|---|---|
| `inicializar wiki sobre [dominio]` | Crear una wiki nueva para un dominio |
| `ingerir [fuente]` | Procesar e integrar una fuente nueva |
| `consultar: [pregunta]` | Consultar la wiki para obtener una respuesta sintetizada |
| `auditar wiki` | Ejecutar revisión de salud sobre la wiki |
| `cerrar sesión` | Generar registro de pendientes operativos de la sesión |

Ver [`docs/flujo-de-trabajo.md`](docs/flujo-de-trabajo.md) para el flujo operativo completo.

## Estructura de la wiki

```
<raiz-wiki>/
├── SCHEMA.md              # Convenciones y reglas específicas del dominio
├── sources/               # Documentos fuente (inmutables)
│   └── assets/
└── wiki/
    ├── indice.md          # Catálogo de contenido (MOC)
    ├── bitacora.md        # Registro cronológico de operaciones
    ├── panorama.md        # Síntesis transversal
    ├── pendientes.md      # Tareas y preguntas abiertas
    ├── referencias.md     # Bibliografía completa (APA 7)
    ├── entidades/         # Personas, organizaciones, lugares
    ├── conceptos/         # Ideas, teorías, marcos
    ├── fuentes/           # Una ficha por fuente ingerida
    └── analisis/          # Respuestas a consultas archivadas
```

## Sistema de trazabilidad

Cada afirmación en la wiki lleva procedencia declarada mediante notas al pie nombradas — sintaxis Markdown estándar, compatible con exportación a DOCX, LaTeX, PDF y HTML vía Pandoc.

Tres categorías:

| Marca | Significado |
|---|---|
| `[^ext-N]` | Extraído directamente de una fuente |
| `[^inf-N]` | Inferido por el LLM a partir de una o más fuentes (con confianza 0.0–1.0) |
| `[^amb-N]` | Ambiguo — fuentes en conflicto, marcado para revisión humana |

```markdown
La organización fue fundada en 1998 [^ext-1].
Probablemente la estrategia era regional [^inf-1].
Las fechas de creación difieren entre versiones [^amb-1].

[^ext-1]: **Extraído** — [[fuente-a]], p. 45.
[^inf-1]: **Inferido** (confianza 0.7) — combinando [[fuente-a]] y [[fuente-b]].
[^amb-1]: **Ambiguo** — [[fuente-a]] indica 1998; [[fuente-c]] indica 1999.
```

## Referencias y citas

Normas APA 7ma edición en todo el contenido. Exportación académica compatible con Pandoc.

## Pipeline de ingesta en dos fases

La ingesta opera en dos fases conceptuales para eliminar la dependencia del orden de las fuentes:

1. **Extracción (solo lectura):** identificar todas las entidades, conceptos y afirmaciones de la fuente; clasificar cada afirmación como extraída, inferida o ambigua.
2. **Materialización:** crear o actualizar páginas, añadir notas de trazabilidad, registrar contradicciones, actualizar índice y bitácora.

Una fuente promedio toca entre 5 y 15 páginas de la wiki.

## Inspirada en

- [Patrón LLM Wiki de Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [atomicmemory/llm-wiki-compiler](https://github.com/atomicmemory/llm-wiki-compiler)
- [safishamsi/graphify](https://github.com/safishamsi/graphify)
- [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)
- [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)

## Licencia

MIT
