# Guía de flujo de trabajo — llm-wiki

Documentación operativa para usar la skill `llm-wiki` en el día a día. Asume Obsidian como cliente y cualquier LLM compatible.

---

## Índice

1. [Concepto operativo](#1-concepto-operativo)
2. [Sesión tipo](#2-sesión-tipo)
3. [Inicializar una wiki nueva](#3-inicializar-una-wiki-nueva)
4. [Ingerir una fuente](#4-ingerir-una-fuente)
5. [Consultar la wiki](#5-consultar-la-wiki)
6. [Auditar la wiki](#6-auditar-la-wiki)
7. [Cerrar sesión](#7-cerrar-sesión)
8. [Convenciones de Obsidian](#8-convenciones-de-obsidian)
9. [Exportación académica](#9-exportación-académica)
10. [Antipatrones frecuentes](#10-antipatrones-frecuentes)

---

## 1. Concepto operativo

La wiki no es una colección de resúmenes. Es un grafo de conocimiento en texto plano: entidades y conceptos con páginas propias, interconectadas, con trazabilidad de fuentes en cada afirmación.

El LLM escribe y mantiene las páginas. El usuario decide qué fuentes ingresar, qué preguntas formular, y qué análisis archivar.

**Regla central:** cada vez que el LLM añade una afirmación a la wiki, debe indicar de dónde viene (`[^ext-N]`, `[^inf-N]`, o `[^amb-N]`). Sin procedencia declarada, la afirmación no entra.

---

## 2. Sesión tipo

Una sesión de trabajo sigue este orden:

```
Abrir Obsidian
  → Revisar pendientes.md
    → Ingerir fuentes pendientes (una por vez)
      → Formular consultas si hace falta
        → Archivar análisis útiles
          → Cerrar sesión
```

**Tiempo estimado por fuente:** 10–20 minutos de interacción activa con el LLM para una fuente de 20–50 páginas.

---

## 3. Inicializar una wiki nueva

**Cuándo:** primera vez que se crea una wiki para un dominio.

**Prompt de inicio:**

```
Inicializa una wiki nueva sobre [dominio].
El propósito es [descripción breve del objetivo].
Las categorías principales que anticipo son [categoría 1, categoría 2, categoría 3].
Usa wikilinks [[]] y convención de notas al pie nombradas para trazabilidad.
Citas en APA 7. Contenido sintetizado en español.
```

**Lo que el LLM debe generar:**

- Estructura de directorios completa
- `SCHEMA.md` con convenciones del dominio
- `indice.md`, `bitacora.md`, `pendientes.md`, `panorama.md`, `referencias.md` con encabezados mínimos

**Verificar antes de continuar:**

- [ ] `SCHEMA.md` declara dominio, convenciones de link, idioma, y APA 7
- [ ] `indice.md` tiene secciones vacías para cada categoría
- [ ] `bitacora.md` tiene su primer registro de `inicialización`

---

## 4. Ingerir una fuente

**Cuándo:** siempre que se quiera incorporar un documento nuevo (paper, libro, informe, artículo, nota propia).

### Paso a paso

**Paso 1 — Avisar antes de dar la fuente:**

```
Voy a ingerir una fuente nueva. Espera a que te la entregue antes de escribir nada.
```

**Paso 2 — Entregar la fuente:**

```
Fuente: [pegar texto, subir PDF, o indicar ruta en sources/]
Autor: [si es conocido]
Año: [si es conocido]
```

**Paso 3 — Fase 1 (extracción):**

El LLM debe listar antes de escribir. Si no lo hace solo, pedirlo:

```
Antes de crear o modificar páginas:
lista las entidades, conceptos y afirmaciones clave que identificas.
Clasifica cada afirmación como extraída, inferida o ambigua.
```

Revisar la lista. Corregir si el LLM omitió algo relevante o sobreestimó la importancia de algo menor.

**Paso 4 — Fase 2 (materialización):**

```
Ahora crea o actualiza las páginas correspondientes.
Recuerda: notas al pie nombradas para toda afirmación,
máximo una cita textual por página (hasta 15 palabras),
actualizar indice.md, referencias.md y bitacora.md.
```

**Paso 5 — Revisar el reporte:**

El LLM debe entregar un reporte con páginas creadas, páginas modificadas, y contradicciones detectadas. Si no lo entrega:

```
Dame el reporte de la ingesta: páginas creadas, actualizadas, y contradicciones detectadas.
```

### Señales de ingesta mal hecha

- El LLM crea páginas sin actualizar `indice.md` → pedirle que lo corrija.
- Afirmaciones sin nota al pie → pedirle que añada trazabilidad.
- Contradicción detectada pero no flagueada con `>[!warning]` → pedirle que lo añada en ambas páginas.
- Copia literal de texto fuente → pedirle que parafrasee y atribuya.

### Ejemplo de ingesta completa

```
Voy a ingerir una fuente nueva. Espera a que te la entregue.

[fuente entregada]

Antes de crear páginas: lista entidades, conceptos y afirmaciones clave.
Clasifica cada afirmación como extraída, inferida o ambigua.

[LLM responde con la lista]

De acuerdo. Añade [[crimen-organizado]] a la lista de conceptos y clasifica
la afirmación sobre redes transnacionales como ambigua, no inferida.

Ahora materializa. Notas al pie, máximo una cita por página, APA 7.
Actualiza indice.md, referencias.md y bitacora.md.

Dame el reporte final.
```

---

## 5. Consultar la wiki

**Cuándo:** cuando se necesita una respuesta sintetizada a partir del contenido acumulado.

### Consulta simple

```
Consulta la wiki y responde: [pregunta]
Cita las páginas relevantes y las fuentes originales en APA 7.
```

### Consulta con archivo

Si el análisis tiene valor permanente:

```
Consulta la wiki y responde: [pregunta]
Si la respuesta contiene síntesis útil, archívala en wiki/analisis/.
```

El LLM debe preguntar si archivar o no. Si no pregunta y archiva directamente, está actuando fuera del principio de cambios quirúrgicos — corregirlo.

### Tipos de respuesta útiles

Según la pregunta, orientar al LLM hacia el formato adecuado:

| Tipo de pregunta | Formato recomendado |
|---|---|
| Comparación entre dos conceptos | Tabla |
| Evolución de un fenómeno en el tiempo | Línea de tiempo |
| Relación entre múltiples actores | Prosa con wikilinks |
| Decisión o recomendación | Análisis estructurado con limitaciones |

```
Responde en formato de tabla comparativa.
Responde con una línea de tiempo en Markdown.
```

---

## 6. Auditar la wiki

**Cuándo:** después de ingerir 5 o más fuentes, o cuando se note que las páginas empiezan a crecer sin estructura clara.

```
Audita la wiki. Busca:
- contradicciones no flagueadas
- páginas huérfanas (sin enlaces entrantes)
- páginas faltantes mencionadas pero no creadas
- enlaces rotos
- afirmaciones sin trazabilidad
- vacíos de datos en el dominio
- nodos centrales (páginas con muchos enlaces entrantes)
- conexiones no obvias entre páginas

Entrega un checklist priorizable. No modifiques nada hasta que yo apruebe.
```

Revisar el checklist. Aprobar ítem por ítem:

```
Corrige los puntos 1, 3 y 5.
El punto 2 lo dejo para más adelante, agrégalo a pendientes.md.
```

---

## 7. Cerrar sesión

**Cuándo:** al terminar cada sesión de trabajo, sin importar cuánto se haya avanzado.

```
Cierra la sesión:
- Resume lo que se hizo hoy en 3 a 6 líneas
- Actualiza pendientes.md con fuentes por ingerir,
  páginas en borrador, contradicciones sin resolver,
  y preguntas abiertas
- Añade entrada de cierre a bitacora.md
```

### Para qué sirve

`pendientes.md` es el punto de partida de la siguiente sesión. Al abrir Obsidian la próxima vez, la primera acción es leer `pendientes.md` antes de cualquier otra cosa.

---

## 8. Convenciones de Obsidian

### Wikilinks

- `[[nombre-pagina]]` — enlace estándar
- `[[nombre-pagina|alias visible]]` — cuando el alias es más legible que el nombre de archivo
- `[[pagina#seccion]]` — enlace a sección específica
- `![[pagina]]` — embed de página completa (usar con moderación)

### Frontmatter

Todo archivo de wiki lleva frontmatter YAML. Obsidian lo lee con el plugin **Dataview** o con **Bases**. Ejemplo mínimo:

```yaml
---
titulo: Nombre legible
tipo: entidad
etiquetas: [dominio/subdominio]
estado: borrador
creado: 2026-04-23
---
```

### Tags jerárquicos

Usar `/` para jerarquía: `#seguridad/crimen-organizado`, `#politica/relaciones-exteriores`. Obsidian los muestra en árbol en el panel de etiquetas.

### Callouts

Reservados para dos casos:

**Contradicciones:**
```markdown
> [!warning] Contradicción entre fuentes
> [[fuente-a]] afirma X; [[fuente-b]] afirma Y.
```

**Advertencias operativas:**
```markdown
> [!note] Página en construcción
> Faltan fuentes sobre el período 2010–2015.
```

No usar callouts para procedencia de afirmaciones individuales. Eso va en notas al pie.

### Grafo de Obsidian

El grafo nativo (`Ctrl+G`) muestra la estructura de la wiki. Para obtener algo útil:

- Filtrar por tipo (`tipo: concepto`) con Dataview
- Usar colores por tag para distinguir capas
- Los nodos con muchos enlaces son los candidatos a MOC temático

---

## 9. Exportación académica

Para exportar la wiki (o páginas seleccionadas) a formatos académicos, usar **Pandoc**.

### Exportar una página a DOCX

```bash
pandoc wiki/analisis/mi-analisis.md -o mi-analisis.docx
```

### Exportar varias páginas con bibliografía

```bash
pandoc wiki/panorama.md wiki/conceptos/*.md \
  -o documento.pdf \
  --pdf-engine=xelatex \
  --toc \
  --bibliography=wiki/referencias.bib \
  --csl=apa-7.csl
```

> **Nota:** Para este comando se necesita un archivo `referencias.bib` en formato BibLaTeX y el archivo de estilo `apa-7.csl`. Pedirle al LLM que genere `referencias.bib` desde el contenido de `referencias.md`.

### Qué se preserva en la exportación

| Elemento | ¿Se preserva? |
|---|---|
| Notas al pie `[^ext-N]`, `[^inf-N]`, `[^amb-N]` | ✓ Completamente |
| Citas APA en texto `(Autor, año, p. X)` | ✓ Como texto plano |
| Blockquotes de citas textuales | ✓ Como blockquote |
| Callouts `>[!warning]` | Degradan a blockquote (contenido intacto) |
| Wikilinks `[[página]]` | Degradan a texto plano |
| Frontmatter YAML | No renderizado por defecto |

Para convertir wikilinks a enlaces antes de exportar, pedirle al LLM:

```
Convierte todos los wikilinks [[X]] a enlaces Markdown estándar [X](X.md)
en el archivo wiki/analisis/mi-analisis.md, para exportación a PDF.
```

---

## 10. Antipatrones frecuentes

### El LLM inventa sin fuente

**Síntoma:** afirmaciones sin nota al pie `[^ext-N]` / `[^inf-N]`.

**Corrección:**
```
Revisa la página [[nombre]] y añade notas al pie de procedencia
a todas las afirmaciones que no las tengan.
Si no puedes atribuirlas a ninguna fuente, márcalas como [^amb-N] y flagea para revisión.
```

### El LLM ingesta varias fuentes en una sola operación

**Síntoma:** el LLM procesa un directorio entero sin discutir.

**Corrección:** forzar una fuente por vez. La discusión previa (Fase 1) es parte del valor de la operación, no un trámite.

### El LLM reescribe páginas que no debería

**Síntoma:** al ingerir una fuente nueva, el LLM modifica páginas no relacionadas con esa fuente.

**Corrección:**
```
Principio de cambios quirúrgicos: toca solo las páginas que la fuente actual justifica.
No reorganices, no reescribas por estilo, no "mejores" páginas no relacionadas.
```

### La wiki crece sin orden

**Síntoma:** `indice.md` desactualizado, páginas sin `tipo` en frontmatter, tags inconsistentes.

**Corrección:** auditoría antes de continuar ingiriendo:
```
Antes de ingerir más fuentes, audita la wiki y devuelve un checklist.
Enfócate en: índice desactualizado, frontmatter faltante, tags inconsistentes.
```

### Las respuestas no citan fuentes

**Síntoma:** consulta devuelve prosa sin wikilinks ni referencias APA.

**Corrección:**
```
Repite la respuesta. Incluye wikilinks a las páginas consultadas
y citas APA 7 para todas las afirmaciones factuales.
```

---

*Última actualización: ver `bitacora.md` de la wiki activa.*
