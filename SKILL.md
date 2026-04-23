---
name: llm-wiki
description: "Construye y mantiene wikis en Markdown persistentes, interconectadas y con trazabilidad de procedencia a partir de fuentes crudas. Úsala cuando el usuario quiera crear una wiki o base de conocimiento desde documentos, ingerir fuentes en una wiki existente, consultar una wiki para obtener respuestas sintetizadas, auditar la salud de una wiki, cruzar entidades o conceptos entre documentos o construir una colección de investigación estructurada. Se activa con menciones de 'wiki', 'base de conocimiento', 'knowledge base', 'ingerir fuente', 'auditar wiki', 'consultar wiki', 'vault Obsidian con LLM', o al pedir convertir notas, artículos o papers dispersos en una colección organizada. Cubre ciclo completo: inicialización, ingesta, consulta, auditoría, evolución de esquema y cierre de sesión. Agnóstica de proveedor LLM (Claude, ChatGPT, Gemini), sin dependencias ejecutables, para Obsidian como cliente principal, exportable a formatos académicos vía Pandoc, con citas en APA 7 y síntesis en español."
---

# Wiki con LLM

Construye y mantiene wikis en Markdown persistentes e interconectadas a partir de fuentes crudas. El LLM escribe y mantiene todo el contenido de la wiki; el usuario curaduriza las fuentes, dirige el análisis y formula preguntas.

Esta skill es **genérica y transversal a cualquier dominio de estudio** (derecho, ciencia política, medicina, historia, ingeniería, humanidades, inteligencia, etc.). No asume un tema específico.

## Principio central

A diferencia de RAG (recuperar y generar en cada consulta), esta skill **compila el conocimiento una sola vez en una wiki persistente** y la mantiene vigente. Las referencias cruzadas, las contradicciones y la síntesis se mantienen de forma incremental; no se rederivan en cada consulta.

Tres capas:

1. **Fuentes crudas** (`sources/`) — documentos de entrada inmutables. El LLM lee pero nunca las modifica.
2. **La wiki** (`wiki/`) — páginas Markdown generadas por el LLM: resúmenes, páginas de entidades, páginas de conceptos, comparativas, síntesis. El LLM es el único dueño de esta capa.
3. **El esquema** (`SCHEMA.md`) — convenciones, plantillas y reglas de flujo específicas de esta wiki. Co-evolucionado por usuario y LLM.

## Principios operativos

Antes de cada operación sobre la wiki el LLM adhiere a cuatro principios. Son reglas de conducta, no pasos de proceso:

1. **Pensar antes de escribir.** No asumir silenciosamente. Si la fuente es ambigua, la instrucción es poco clara o hay varias interpretaciones válidas, detenerse y preguntar. Explicitar supuestos al usuario antes de materializarlos en páginas.
2. **Simplicidad primero.** No crear páginas especulativas ni frontmatter vacío. Si un concepto aparece una sola vez y sin peso analítico, mencionarlo dentro de otra página en lugar de abrirle una propia. Revisar si una página ya existe antes de crearla.
3. **Cambios quirúrgicos.** Al actualizar una página por una nueva fuente, modificar sólo lo que la fuente justifica. No reescribir por estilo, no "mejorar" secciones no relacionadas, no reorganizar silenciosamente.
4. **Orientación a metas verificables.** Cada operación debe terminar con un criterio comprobable (página creada, índice actualizado, contradicción registrada, bitácora escrita). Al cerrar una operación, declarar qué se verificó.

## Estructura de directorios

```
<raiz-wiki>/
├── SCHEMA.md              # Convenciones y reglas específicas de esta wiki
├── sources/               # Documentos fuente (inmutables)
│   └── assets/            # Imágenes y anexos binarios
└── wiki/
    ├── indice.md          # Catálogo de contenido (MOC — Map of Content)
    ├── bitacora.md        # Registro cronológico de operaciones
    ├── panorama.md        # Síntesis transversal de toda la wiki
    ├── pendientes.md      # Tareas y vacíos identificados
    ├── referencias.md     # Bibliografía completa en formato APA 7
    ├── entidades/         # Páginas de personas, organizaciones, lugares
    ├── conceptos/         # Páginas de ideas, teorías, marcos
    ├── fuentes/           # Una ficha por fuente ingerida
    └── analisis/          # Comparativas y consultas archivadas
```

Adapta la estructura al dominio. Si el usuario ya tiene un vault con otra organización, respétala y documéntala en `SCHEMA.md`.

## Cliente principal: Obsidian

Esta skill asume **Obsidian** como cliente de lectura y edición. La sintaxis y convenciones por defecto son las de Obsidian Flavored Markdown:

- **Wikilinks**: `[[nombre-pagina]]` y `[[nombre-pagina|alias visible]]`
- **Embeds**: `![[nombre-pagina]]` o `![[pagina#encabezado]]` para incrustar secciones
- **Callouts**: `>[!note]`, `>[!warning]`, `>[!quote]`, `>[!cite]`, `>[!question]`, `>[!danger]`
- **Properties (YAML frontmatter)**: declaradas en la cabecera de cada página para lectura por Dataview y Bases
- **Tags**: `#dominio/subdominio` en jerarquía
- **Notas al pie**: `[^1]` para referencias breves dentro del texto

Si el usuario exporta la wiki a otro cliente (VSCode, Foam, etc.), los wikilinks y callouts se degradan limpiamente; el contenido sigue siendo Markdown válido.

## Frontmatter estándar

Todas las páginas llevan frontmatter YAML. Valores mínimos obligatorios:

```yaml
---
titulo: Título legible
tipo: entidad | concepto | fuente | analisis | pendiente
etiquetas: [dominio/subdominio]
idioma: es
creado: YYYY-MM-DD
actualizado: YYYY-MM-DD
fuentes: [[nombre-fuente-a]], [[nombre-fuente-b]]
estado: borrador | revisado | estable
---
```

Valores opcionales según tipo:

- `autor`, `anio`, `titulo_original`, `editorial`, `doi`, `url`, `acceso` (para `tipo: fuente`)
- `alias` (lista de nombres alternativos para entidades o conceptos)
- `cluster` (agrupación temática; útil si más adelante se activa la extensión de grafo)
- `truncado: true` + `caracteres_originales: N` si la fuente excedió la ventana de contexto y se procesó parcial

## Sistema de trazabilidad y procedencia

Toda afirmación en la wiki debe declarar su procedencia. Tres categorías:

| Marca | Significado | Uso |
|---|---|---|
| `extraído` | Afirmación presente textualmente o parafraseada directamente desde una fuente | Citas directas y paráfrasis cercanas |
| `inferido` | Afirmación deducida por el LLM combinando fuentes o contexto | Requiere nivel de confianza `0.0–1.0` |
| `ambiguo` | Afirmación con interpretación dudosa o fuentes en conflicto | Marcada para revisión humana |

**Sintaxis obligatoria: notas al pie nombradas** (Markdown estándar universal).

Esta convención prioriza la compatibilidad con exportadores académicos (Pandoc → DOCX, LaTeX, PDF, HTML) sobre la comodidad de escritura. Las afirmaciones llevan una referencia corta inline y la procedencia se declara al pie de página:

```markdown
La organización fue fundada en 1998 [^ext-1].
Probablemente el objetivo era regional [^inf-1].
Las fechas de creación difieren entre versiones [^amb-1].

[^ext-1]: **Extraído** — [[fuente-a]], p. 45.
[^inf-1]: **Inferido** (confianza 0.7) — combinando [[fuente-a]] y [[fuente-b]].
[^amb-1]: **Ambiguo** — [[fuente-a]] indica 1998; [[fuente-c]] indica 1999.
```

**Convención de nombrado de las notas**:

- `[^ext-N]` para afirmaciones extraídas
- `[^inf-N]` para afirmaciones inferidas (con confianza entre paréntesis en el cuerpo)
- `[^amb-N]` para afirmaciones ambiguas
- Los prefijos (`ext`, `inf`, `amb`) permiten filtrar por `grep` o buscador integrado.
- La numeración es local a cada página, empieza en `1` y continúa secuencialmente.

Las notas al pie son sintaxis estándar en CommonMark extendido, GitHub Flavored Markdown, Pandoc y Obsidian. Se exportan sin pérdida a todos los formatos académicos habituales.

**Callouts como recurso secundario** (no sustituyen a las notas al pie).

Se reservan para dos casos:

1. **Cita textual en bloque** — siempre que sea necesario mostrar literalidad:

   ```markdown
   > "Texto citado hasta 15 palabras" [traducción si aplica]
   > — Apellido (2024, p. 45)
   ```

   El blockquote estándar funciona en cualquier exportador. Si se prefiere el callout visual de Obsidian, usar `>[!quote]` — al exportar, Pandoc lo degrada a blockquote estándar sin pérdida de contenido.

2. **Contradicciones detectadas durante ingesta** — requieren énfasis visual:

   ```markdown
   > [!warning] Contradicción entre fuentes
   > [[fuente-a]] afirma X; [[fuente-b]] afirma Y.
   > Discusión: [[analisis-titulo]].
   ```

No usar callouts para procedencia de afirmaciones individuales — eso siempre va por nota al pie.

## Normas APA 7ma edición

Las referencias se gestionan según APA 7. Dos niveles:

**1. Frontmatter de fichas de fuente** (`wiki/fuentes/*.md`):

```yaml
---
titulo: Ficha breve para navegar en la wiki
tipo: fuente
autor: [Apellido, N. I.]
anio: 2024
titulo_original: Título completo de la obra
editorial: Editorial o Revista (Vol., pp.)
doi: 10.xxxx/xxxxx
url: https://...
acceso: YYYY-MM-DD
idioma_fuente: en | es | pt | ...
---
```

**2. Citas en el cuerpo de las páginas**:

- Cita parafraseada: `(Apellido, año)` — ejemplo: `(Foucault, 1975)`
- Cita directa: `(Apellido, año, p. X)` o `(Apellido, año, pp. X-Y)`
- Dos autores: `(Apellido y Apellido, año)`
- Tres o más autores: `(Apellido et al., año)`
- Fuente institucional sin autor: `(Nombre de la institución, año)` la primera vez; abreviatura en usos posteriores

**3. `referencias.md`**:

Lista consolidada, ordenada alfabéticamente, con entradas en formato APA 7 completo. Ejemplo:

```markdown
Foucault, M. (1975). *Surveiller et punir: Naissance de la prison*. Gallimard.

Mearsheimer, J. J. (2014). The tragedy of great power politics (Updated ed.). W. W. Norton.

Organización de las Naciones Unidas. (2023). *Informe anual 2023*. https://un.org/...
```

Cada entrada referenciable desde las páginas mediante wikilink a la ficha de fuente (`[[fuente-titulo-abreviado]]`), y contrastable con `referencias.md`.

## Tratamiento multilingüe

**Regla general**: todo el contenido sintetizado por el LLM se escribe en **español**, independientemente del idioma de las fuentes.

**Citas textuales**: se conservan en el idioma original, con traducción adyacente entre corchetes cuando el idioma original no sea español y la comprensión lo requiera.

```markdown
> "Big Brother is watching you" [Gran Hermano te vigila]
> — Orwell (1949, p. 3)
```

**Títulos de obras extranjeras**: nombre original en cursiva, traducción entre corchetes la primera vez que aparecen.

```markdown
*Surveiller et punir* [Vigilar y castigar] (Foucault, 1975) establece...
```

**Nombres propios**: se mantienen en su forma original (Foucault, no "Foucó"). Excepción: entidades con nombre oficial establecido en español (Organización de las Naciones Unidas, no "United Nations").

**Límite de citación**: cada fuente admite como máximo **una cita textual** por página de wiki, y ninguna cita puede superar **15 palabras**. Para textos más extensos, parafrasear y atribuir.

## Operaciones

### 1. Inicializar

Cuando el usuario quiere crear una wiki nueva:

1. Crear la estructura de directorios descrita arriba.
2. Generar un `SCHEMA.md` inicial con, al menos:
   - Dominio y alcance declarados en una línea
   - Formato de wikilinks (`[[]]` por defecto)
   - Convención de trazabilidad (notas al pie nombradas por defecto: `[^ext-N]`, `[^inf-N]`, `[^amb-N]`)
   - Categorías raíz del índice (adaptadas al dominio)
   - Plantillas de frontmatter por tipo de página
   - Reglas de idioma (español por defecto, excepciones para citas)
   - Declaración de APA 7 como estándar de referencias
   - Política de exportación académica (objetivos de salida: DOCX, LaTeX, PDF vía Pandoc)
3. Crear `indice.md`, `bitacora.md` y `pendientes.md` con encabezados mínimos.
4. Crear `panorama.md` con una frase sobre el alcance.
5. Crear `referencias.md` vacío (sólo encabezado).
6. Copiar la wiki inicial a `/mnt/user-data/outputs/` y presentarla con `present_files`.

Preguntar antes de asumir: dominio, categorías preferidas, si hay un vault preexistente, convenciones especiales del usuario.

### 2. Ingerir (pipeline de dos fases)

Cuando el usuario aporta una nueva fuente para integrar, aplicar un **pipeline conceptual de dos fases** para evitar dependencia del orden y permitir fusionar conceptos compartidos entre fuentes:

**Fase 1 — Extracción (solo lectura)**

1. Leer la fuente completa (o la porción disponible; si excede ventana, marcar `truncado: true` con el conteo original).
2. Enumerar internamente:
   - Entidades mencionadas (personas, organizaciones, lugares, artefactos)
   - Conceptos mencionados (ideas, teorías, métodos, marcos)
   - Afirmaciones verificables de interés
   - Citas textuales candidatas (máximo una por página, hasta 15 palabras)
3. Clasificar cada afirmación: `extraído`, `inferido` (con confianza) o `ambiguo`.
4. Presentar al usuario 3 a 5 hallazgos clave antes de escribir nada.

**Fase 2 — Materialización**

5. Crear ficha de fuente en `wiki/fuentes/` con frontmatter APA 7 completo y resumen sintetizado en español (no copia literal).
6. Para cada entidad o concepto listado:
   - ¿Existe página? → actualizarla con información nueva y trazabilidad.
   - ¿No existe? → crearla si tiene peso analítico; de lo contrario mencionarla en páginas vecinas.
7. Añadir afirmaciones con su marca de procedencia inline.
8. Detectar contradicciones con el contenido existente y registrarlas explícitamente en ambas páginas:

```markdown
> [!warning] Contradicción
> [[fuente-a]] afirma X; [[fuente-b]] afirma Y.
> Ver discusión en [[analisis-contradiccion-fecha]] si existe.
```

9. Actualizar `indice.md` (añadir páginas nuevas, refrescar resúmenes de páginas modificadas).
10. Actualizar `referencias.md` con la entrada APA 7 de la fuente.
11. Añadir una línea a `bitacora.md`:

```markdown
## [YYYY-MM-DD] ingesta | Título de la fuente
- Ficha creada: [[fuente-titulo]]
- Páginas creadas: [[pagina-a]], [[pagina-b]]
- Páginas actualizadas: [[pagina-c]], [[pagina-d]]
- Contradicciones detectadas: 1 (ver [[pagina-c]])
- Notas: descripción breve del cambio
```

12. Reportar al usuario las páginas tocadas. Una fuente promedio toca entre 5 y 15 páginas.

**Regla estricta**: nunca sobrescribir silenciosamente. Cualquier modificación de una afirmación previa debe dejar rastro (en la página o en `bitacora.md`).

### 3. Consultar

Cuando el usuario hace una pregunta contra la wiki:

1. Leer `indice.md` para identificar páginas relevantes (no escanear todo).
2. Leer las páginas candidatas.
3. Sintetizar una respuesta en español con citas a páginas específicas mediante wikilinks, y a fuentes originales mediante formato APA 7.
4. **Ofrecer archivar la respuesta** como nueva página en `wiki/analisis/`. El usuario decide. Si acepta:
   - Crear página con frontmatter `tipo: analisis`
   - Actualizar `indice.md`
   - Añadir línea a `bitacora.md`
   - El análisis archivado queda disponible como contexto para consultas futuras — las consultas se componen con el tiempo.

Formato de respuesta según pregunta: prosa, tabla comparativa, línea de tiempo, árbol de decisiones. Ajustar.

### 4. Auditar

Cuando el usuario pide revisión de salud, o proactivamente tras crecimiento significativo:

Escanear la wiki en busca de:

- **Contradicciones** no flagueadas entre páginas
- **Afirmaciones obsoletas** superadas por fuentes más recientes
- **Páginas huérfanas** sin enlaces entrantes
- **Páginas faltantes** — entidades o conceptos mencionados sin página propia
- **Enlaces rotos** — wikilinks que apuntan a páginas inexistentes
- **Referencias cruzadas faltantes** entre páginas relacionadas
- **Vacíos de datos** — áreas del dominio con pocas fuentes
- **Deriva del índice** — páginas no catalogadas o entradas que apuntan a páginas eliminadas
- **Inconsistencias APA** — entradas en `referencias.md` sin ficha correspondiente, o viceversa
- **Falta de trazabilidad** — afirmaciones sin marca de procedencia
- **Nodos centrales** (equivalente a "god nodes"): páginas con alta cantidad de enlaces entrantes, candidatas a MOC temático
- **Conexiones inesperadas**: pares de páginas que comparten múltiples fuentes pero no se enlazan entre sí
- **Preguntas que la wiki puede responder de forma única** — útiles para sugerir al usuario

Reportar hallazgos como checklist priorizable. Ejecutar correcciones sólo con aprobación explícita.

### 5. Evolucionar el esquema

`SCHEMA.md` es un documento vivo. Cuando emerjan patrones durante el uso:

- Nuevos tipos de página necesarios → proponer plantilla
- Convenciones de nomenclatura que se quiebran → proponer revisión
- Campos de frontmatter omitidos sistemáticamente → proponer valores por defecto o eliminación

**Proponer siempre los cambios al usuario antes de aplicarlos.** Documentar cada revisión con fecha al pie de `SCHEMA.md`.

### 6. Cerrar sesión

Operación de utilidad al final de una sesión de trabajo. No es un registro de seguridad; es un artefacto operativo para retomar el trabajo después.

1. Resumir en 3 a 6 líneas lo hecho en la sesión (qué se ingirió, qué se consultó, qué se archivó).
2. Actualizar `pendientes.md` con:
   - Fuentes mencionadas pero no ingeridas aún
   - Páginas marcadas como `estado: borrador` que requieren cierre
   - Contradicciones pendientes de resolución
   - Preguntas abiertas planteadas durante la sesión
3. Añadir entrada a `bitacora.md` con tipo `cierre`.

Formato de `pendientes.md`:

```markdown
# Pendientes

## Fuentes por ingerir
- [ ] [Título] — mencionada en [[fuente-a]] y [[analisis-b]]

## Páginas borrador
- [ ] [[pagina-x]] — falta ampliar sección sobre Y

## Contradicciones sin resolver
- [ ] [[pagina-z]] — conflicto entre [[fuente-c]] y [[fuente-d]]

## Preguntas abiertas
- [ ] ¿Cómo afecta X a Y en el contexto Z?
```

## Convenciones del índice y la bitácora

### `indice.md`

Catálogo orientado al contenido. Una línea por página:

```markdown
# Índice de la wiki

## Entidades
- [[nombre-entidad]] — descripción breve (N fuentes)

## Conceptos
- [[nombre-concepto]] — descripción breve

## Fuentes
- [[titulo-fuente]] — Autor, año. Descripción breve

## Análisis
- [[titulo-analisis]] — Fecha. Descripción breve

## Pendientes destacados
- Ver [[pendientes]]
```

Actualizar en cada ingesta. Mantener los resúmenes a una línea.

### `bitacora.md`

Cronológica, sólo añadir (append-only). Cada entrada:

```markdown
## [YYYY-MM-DD] operación | Título
- Páginas creadas: [[pagina-a]], [[pagina-b]]
- Páginas actualizadas: [[pagina-c]], [[pagina-d]]
- Notas: breve descripción
```

Tipos de operación: `ingesta`, `consulta`, `analisis`, `auditoria`, `esquema`, `cierre`.

## Reglas de referencia cruzada

- Cada página de entidad o concepto enlaza a las fuentes que la respaldan.
- Cada ficha de fuente enlaza a las entidades y conceptos que menciona.
- Usar el formato de wikilink definido en `SCHEMA.md` (`[[]]` por defecto).
- Al crear una página nueva, escanear las páginas existentes y añadir retroenlaces donde se mencione la entidad o concepto nuevo.

## Escalamiento

- **< 50 páginas**: `indice.md` es suficiente. No se requieren herramientas de búsqueda.
- **50–200 páginas**: el grafo nativo de Obsidian y su búsqueda integrada son adecuados. Considerar el plugin Dataview para vistas dinámicas.
- **200+ páginas**: evaluar la extensión opcional de grafo consultable (ver siguiente sección).
- La wiki es una colección de archivos Markdown compatible con git; el historial de versiones viene sin costo adicional.

## Extensión opcional: grafo consultable

Para wikis grandes o análisis multi-fuente complejos, la convención de trazabilidad ya establecida (marcas `extraído` / `inferido` / `ambiguo`, frontmatter con `fuentes`, wikilinks estrictos) permite derivar un grafo consultable sin infraestructura adicional:

- **En Obsidian**: el grafo nativo muestra nodos y aristas de wikilinks. Filtrable por tag y tipo.
- **Con Dataview**: consultas tipo `LIST FROM [[concepto]] WHERE tipo = "fuente"` producen subgrafos temáticos.
- **Con JSON Canvas** (`.canvas`): crear mapas visuales curados de áreas específicas, manteniendo wikilinks a las páginas fuente.

Esta skill no genera grafos ejecutables por sí misma; establece las convenciones que hacen que el grafo sea trivialmente derivable desde Obsidian o herramientas externas.

## Exportación académica

La skill está pensada para producir documentación académica que requiere referencias absolutas. Todas las convenciones (frontmatter YAML, wikilinks, notas al pie nombradas, blockquotes estándar, citas APA 7 en el cuerpo) son Markdown estricto y se exportan sin pérdida mediante Pandoc.

**Formatos de salida soportados de forma directa**:

| Formato | Comando Pandoc base | Preserva |
|---|---|---|
| DOCX | `pandoc pagina.md -o pagina.docx` | Notas al pie, cursivas, blockquotes, títulos |
| LaTeX / PDF | `pandoc pagina.md -o pagina.pdf --pdf-engine=xelatex` | Idem + tipografía académica |
| HTML | `pandoc pagina.md -o pagina.html --standalone` | Idem + enlaces |
| EPUB | `pandoc pagina.md -o pagina.epub` | Idem |

**Para exportar la wiki completa o una selección**:

```bash
pandoc wiki/panorama.md wiki/conceptos/*.md wiki/fuentes/*.md \
  -o documento.pdf \
  --pdf-engine=xelatex \
  --toc \
  --bibliography=wiki/referencias.bib \
  --csl=apa-7.csl
```

(El comando es referencial; la skill no ejecuta nada.)

**Qué se degrada al exportar**:

- Los wikilinks `[[pagina]]` se convierten en texto plano. Para mantener navegación, reemplazarlos por enlaces Markdown estándar antes de exportar (conversión mecánica con `sed` o con el plugin Obsidian Export).
- Los callouts `>[!warning]` / `>[!note]` se exportan como blockquote simple (el contenido se mantiene; se pierde el icono).
- Las *properties* de Obsidian (frontmatter YAML) no se renderizan por defecto en Pandoc, pero se pueden mapear a variables de plantilla (`--template`).

**Lo que NO se degrada**:

- Notas al pie `[^ext-N]`, `[^inf-N]`, `[^amb-N]` — se exportan a todos los formatos como notas al pie reales.
- Citas APA en el cuerpo `(Autor, año, p. X)` — texto plano, válido en cualquier formato.
- Blockquotes para citas textuales — universales.
- Cursivas, negritas, tablas, encabezados — universales.

**Para una gestión bibliográfica formal**: mantener en paralelo un archivo `referencias.bib` en formato BibLaTeX con las mismas entradas que `referencias.md`. Esto permite el procesamiento automático de citas por Pandoc con estilos CSL (Citation Style Language) oficiales de APA 7. La skill no genera el `.bib` por defecto, pero puede hacerlo si el usuario lo pide.

## Compatibilidad con múltiples LLM

La skill está redactada sin dependencias ejecutables y sin sintaxis propietaria. Funciona con Claude, ChatGPT, Gemini o cualquier otro LLM que admita lectura y escritura de archivos Markdown. Las convenciones de frontmatter, wikilinks, notas al pie y callouts son Markdown estándar o extensiones ampliamente soportadas.

Si el usuario migra de proveedor o de cliente (de Obsidian a VSCode, Foam, MkDocs, Quartz), la wiki se mantiene: sólo cambia quién la opera y cómo se renderiza. El contenido y su trazabilidad son portables.

## Qué NO hacer

- No modificar archivos en `sources/`. Son inmutables.
- No sobrescribir contradicciones en silencio. Siempre registrarlas.
- No crear páginas sin actualizar `indice.md` y `bitacora.md`.
- No asumir ingesta por lotes. Por defecto, una fuente por vez con discusión previa.
- No reproducir texto fuente literal más allá del límite de citación (1 cita por página, máximo 15 palabras).
- No redactar contenido sintetizado en idioma distinto al español.
- No generar afirmaciones sin marca de procedencia.
- No crear páginas especulativas de entidades o conceptos mencionados una sola vez y sin peso analítico.
- No reorganizar, reescribir o "mejorar" secciones no relacionadas con la operación en curso.
- No asumir que el usuario quiere que se ejecuten scripts o herramientas externas. La skill es autocontenida.

## Plantillas mínimas

### Página de entidad

```markdown
---
titulo: Nombre de la entidad
tipo: entidad
etiquetas: [dominio/subdominio]
idioma: es
creado: YYYY-MM-DD
actualizado: YYYY-MM-DD
fuentes: [[fuente-a]], [[fuente-b]]
estado: borrador
alias: [nombre alternativo 1, nombre alternativo 2]
---

# Nombre de la entidad

Resumen de una o dos líneas.

## Descripción

Párrafo sintético en español. Afirmaciones con marcas de procedencia al pie [^ext-1].

## Hechos clave

- Fecha de fundación/nacimiento [^ext-2]
- Rol o función principal [^inf-1]

## Relaciones

- Vinculada con [[otra-entidad]] por [[concepto-puente]]
- Mencionada en el contexto de [[concepto-x]]

## Fuentes que la mencionan

- [[fuente-a]] — contexto breve
- [[fuente-b]] — contexto breve

---

[^ext-1]: **Extraído** — [[fuente-a]], p. 12.
[^ext-2]: **Extraído** — [[fuente-a]], p. 3.
[^inf-1]: **Inferido** (confianza 0.8) — combinando [[fuente-a]] y [[fuente-b]].
```

### Página de concepto

```markdown
---
titulo: Nombre del concepto
tipo: concepto
etiquetas: [dominio/subdominio]
idioma: es
creado: YYYY-MM-DD
actualizado: YYYY-MM-DD
fuentes: [[fuente-a]], [[fuente-b]]
estado: borrador
---

# Nombre del concepto

Definición operativa de una o dos líneas.

## Desarrollo

Elaboración sintética en español, con trazabilidad.

## Variantes y tensiones

- [[fuente-a]] lo define como X [^ext-1]
- [[fuente-b]] lo define como Y [^ext-2]
- Tensión: ver [[analisis-divergencia-concepto]]

## Ejemplos

- [[entidad-x]] ilustra el concepto en el contexto Z.

## Relacionado

- [[concepto-vecino]], [[concepto-opuesto]]

---

[^ext-1]: **Extraído** — [[fuente-a]], p. 45.
[^ext-2]: **Extraído** — [[fuente-b]], p. 112.
```

### Ficha de fuente

```markdown
---
titulo: Título corto para navegación
tipo: fuente
autor: [Apellido, N. I.]
anio: 2024
titulo_original: Título completo exacto
editorial: Editorial o Revista
doi: 10.xxxx/xxxxx
url: https://...
acceso: YYYY-MM-DD
idioma_fuente: en
idioma: es
etiquetas: [dominio/subdominio]
creado: YYYY-MM-DD
actualizado: YYYY-MM-DD
estado: revisado
truncado: false
---

# Título corto

**Referencia APA 7**: Apellido, N. I. (2024). *Título completo exacto*. Editorial.

## Síntesis

Resumen estructurado en español: tesis central, argumentos principales, evidencia aportada, limitaciones declaradas por el autor.

## Afirmaciones clave

- Afirmación 1 [^ext-1]
- Afirmación 2 [^ext-2]

## Cita relevante

> "Texto citado ≤15 palabras" [traducción si aplica]
> — Apellido (2024, p. X)

## Entidades mencionadas

- [[entidad-a]] — rol en el texto
- [[entidad-b]] — rol en el texto

## Conceptos tratados

- [[concepto-a]], [[concepto-b]]

## Conexiones con otras fuentes

- Coincide con [[fuente-c]] en X.
- Contradice [[fuente-d]] en Y — ver discusión en [[analisis-contradiccion]].

---

[^ext-1]: **Extraído** — p. X.
[^ext-2]: **Extraído** — pp. X-Y.
```

### Página de análisis

```markdown
---
titulo: Pregunta o tema del análisis
tipo: analisis
etiquetas: [dominio/subdominio]
idioma: es
creado: YYYY-MM-DD
actualizado: YYYY-MM-DD
fuentes: [[fuente-a]], [[fuente-b]], [[fuente-c]]
estado: revisado
---

# Título del análisis

## Pregunta

Enunciado preciso de la consulta original.

## Respuesta sintética

Prosa, tabla, línea de tiempo, o el formato más adecuado a la pregunta.

## Evidencia y trazabilidad

Detalle de afirmaciones con marcas de procedencia.

## Limitaciones

- Vacíos identificados
- Supuestos que la respuesta asume
- Fuentes que convendría incorporar
```

## Resumen operativo

| Operación | Entrada | Salida |
|---|---|---|
| Inicializar | Dominio, alcance | Estructura base + `SCHEMA.md` |
| Ingerir | Fuente + discusión previa | Ficha de fuente + páginas tocadas + entrada en bitácora |
| Consultar | Pregunta | Respuesta sintetizada (opcionalmente archivada) |
| Auditar | — | Checklist priorizable de hallazgos |
| Evolucionar esquema | Patrón recurrente | Revisión propuesta de `SCHEMA.md` |
| Cerrar sesión | — | `pendientes.md` actualizado + entrada en bitácora |

La skill es operativa sin scripts, sin servidores y sin dependencia de un proveedor de LLM específico. Su valor reside en las convenciones que impone y en la disciplina de trazabilidad que exige.
