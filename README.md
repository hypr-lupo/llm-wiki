[Español](README.md) | [English](README.es.md)

# llm-wiki

Una skill para construir y mantener wikis en Markdown persistentes e interconectadas a partir de fuentes crudas. El LLM escribe y mantiene todo el contenido de la wiki; el usuario selecciona las fuentes, dirige el análisis y formula preguntas.

Inspirada en el [patrón LLM Wiki de Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Diseñada para [Obsidian](https://obsidian.md) como cliente principal. Compatible con cualquier LLM.

> Documentación completa: [`docs/flujo-de-trabajo.md`](docs/flujo-de-trabajo.md)

---

## El problema que resuelve

RAG re-descubre las mismas relaciones desde cero en cada consulta. Nada acumula.

Esta skill compila las fuentes en una wiki una sola vez y la mantiene vigente. Los conceptos tienen páginas propias. Las páginas se enlazan entre sí. Cada afirmación declara su procedencia. Cuando se archiva una respuesta a una consulta, las consultas futuras la usan como contexto. El conocimiento crece con cada sesión.

```
RAG:      consulta → buscar fragmentos → respuesta → olvidar
llm-wiki: fuentes → compilar → wiki → consultar → archivar → wiki más rica → mejores respuestas
```

## Para quién es

Investigadores, analistas, periodistas y cualquier persona que trabaje con múltiples documentos y necesite síntesis, no solo recuperación. Investigación académica, análisis jurídico, inteligencia, periodismo, gestión del conocimiento personal — cualquier campo donde las fuentes se acumulan.

## Requisitos

- Cualquier LLM con capacidad de leer y escribir archivos Markdown (Claude, ChatGPT, Gemini, u otros)
- [Obsidian](https://obsidian.md) como cliente principal (opcional, pero las convenciones de la skill lo asumen)
- Sin ejecutables, sin scripts, sin dependencias externas

## Instalación

### Claude (claude.ai / Claude Code)

**Opción A:** npx skills
```bash
npx skills add git@github.com:hypr-lupo/llm-wiki.git
```

**Opción B:** manual
```bash
mkdir -p ~/.claude/skills/llm-wiki
curl -fsSL https://raw.githubusercontent.com/hypr-lupo/llm-wiki/main/SKILL.md \
  > ~/.claude/skills/llm-wiki/SKILL.md
```

### ChatGPT / Gemini / Otros LLMs

Pegar el contenido de `SKILL.md` en el system prompt o en las instrucciones personalizadas del modelo.

## Uso

Dile a tu LLM:

| Comando | Acción |
|---|---|
| `inicializar wiki sobre [dominio]` | Crear una wiki nueva para un dominio |
| `ingerir [fuente]` | Procesar e integrar una fuente nueva |
| `consultar: [pregunta]` | Consultar la wiki para obtener una respuesta sintetizada |
| `auditar wiki` | Ejecutar revisión de salud sobre la wiki |
| `cerrar sesión` | Generar registro de pendientes de la sesión |

Flujo completo: [`docs/flujo-de-trabajo.md`](docs/flujo-de-trabajo.md)

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

Cada afirmación en la wiki declara su procedencia mediante notas al pie nombradas, compatibles con exportación a DOCX, LaTeX, PDF y HTML vía Pandoc.

Tres categorías:

| Marca | Significado |
|---|---|
| `[^ext-N]` | Extraído directamente de una fuente |
| `[^inf-N]` | Inferido por el LLM a partir de una o más fuentes (con confianza 0.0–1.0) |
| `[^amb-N]` | Fuentes en conflicto — requiere revisión |

```markdown
La organización fue fundada en 1998 [^ext-1].
Probablemente la estrategia era regional [^inf-1].
Las fechas de creación difieren entre versiones [^amb-1].

[^ext-1]: **Extraído** — [[fuente-a]], p. 45.
[^inf-1]: **Inferido** (confianza 0.7) — combinando [[fuente-a]] y [[fuente-b]].
[^amb-1]: **Ambiguo** — [[fuente-a]] indica 1998; [[fuente-c]] indica 1999.
```

## Referencias y citas

Citas y referencias en APA 7. Exportación académica vía Pandoc.

## Ingesta en dos fases

La ingesta opera en dos fases:

1. **Extracción (solo lectura):** identificar todas las entidades, conceptos y afirmaciones de la fuente; clasificar cada afirmación como extraída, inferida o ambigua.
2. **Materialización:** crear o actualizar páginas, añadir notas de procedencia, registrar contradicciones, actualizar índice y bitácora.

Una fuente promedio toca entre 5 y 15 páginas de la wiki.

## Inspirada en

- [Patrón LLM Wiki de Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [atomicmemory/llm-wiki-compiler](https://github.com/atomicmemory/llm-wiki-compiler)
- [safishamsi/graphify](https://github.com/safishamsi/graphify)
- [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)
- [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)

## Licencia

MIT
