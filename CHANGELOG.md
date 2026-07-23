# Changelog — MCU Tracker

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/).
Este proyecto no usa versionado semántico de paquete (no hay build/release
formal); las entradas se agrupan por fecha de sesión de trabajo.

## [Sin publicar]
### Pendiente
- Protocolo de exclusión mutua para ediciones concurrentes de `index.html`
- Revisar si `DISENO-SPEC.md` (paleta roja/dorada histórica) debe marcarse obsoleto
- Verificación visual en dispositivo real del layout `.item` en ≤560px
- Confirmar si `.stats` debe pasar a 1 columna en <380px (actualmente
  fija a `repeat(3,1fr)` en todos los breakpoints)
- Comprobar `scrollWidth ≤ innerWidth` a 320px en la fila compacta de
  `.item` con duración larga + nota puntuada (posible overflow horizontal)

---

## [2026-07-23] — Parte 2: Rediseño integral "Multiverso"
Rediseño visual y de layout completo ejecutado en 8 fases según
`design/PLAN-REDISENO-PRO.md` (brief de decisiones). Commit `9f3e507`.
Contratos técnicos (KEYs de `localStorage`, identidad `t`, `TAB_IDS`,
orden `buildXmen()`/`buildChecklist()`) intactos.

### Añadido
- `design/PLAN-REDISENO-PRO.md`: plan de decisiones del rediseño (paleta,
  tipografía, fondo, layout escritorio/móvil, componente a componente,
  fases, riesgos y criterios de aceptación)
- Fondo ambiental `.bg-fx` de 2 capas: `.bg-fx__glows` (radial-gradients
  fijos) + `.bg-fx__grain` (SVG `feTurbulence` estático), sustituyendo el
  collage fotográfico de 7 capas anterior
- Escala tipográfica fluida con `clamp()` (`--fs-hero/h2/title/body/meta/
  label/stat/micro`) y `tabular-nums` en toda cifra (stats, %, duraciones,
  años) en `css/tokens.css`/`css/styles.css`
- Layout de 2 columnas en escritorio (`≥900px`): `.grid`/`.action-grid` a
  `grid-template-columns:repeat(auto-fill,minmax(440px,1fr))`, container
  1200px, hero panorámico, hovers reales bajo `@media (hover:hover) and
  (pointer:fine)`
- Bottom navigation fija en móvil (`<900px`, mismos 4 botones `.tab`),
  con iconos SVG + etiqueta corta, indicador de destino activo,
  `env(safe-area-inset-bottom)` y `body{padding-bottom:...}` para no
  tapar el último ítem
- Fila `.item` compacta de 2 filas en móvil (`<600px`) vía
  `grid-template-areas`, sin cambiar clases del HTML
- Nota 0–10 como **bottom sheet** en móvil (`.rating__popover--sheet`,
  `body.sheet-open`, `.rating-scrim`): scrim, foco atrapado, cierre por
  Escape, `overscroll-behavior:contain`; `isMobileSheet()` se evalúa en
  cada apertura del popover
- `<link rel="preload" as="image" ... fetchpriority="high">` del hero
  (LCP), `img.decoding='async'` en pósteres, `@media
  (prefers-reduced-motion:reduce)`, `@media (forced-colors:active)`
- Meta SEO/redes reales: `<title>`/`<meta description>`, Open Graph
  (`og:type/title/description/image/locale`), Twitter Card
  (`summary_large_image`), `<meta name="theme-color">`

### Modificado
- Paleta recolorida a la dirección "Multiverso" en `css/tokens.css`:
  `--bg #0b0b14`, `--surface-1/2/3` escalonadas, `--border`/
  `--border-hover` con alpha, `--accent #a78bfa` / `--accent-fill
  #8b5cf6` / `--accent-2 #60a5fa`, `--especial #e0a94f`,
  `--container-max` 900px → 1200px; alias legacy conservados
  (`--surface`, `--blue`, `--brand-purple*`, `--brand-blue*`,
  `--glow-purple/blue`, `--fs-eyebrow/small/xs`) para no dejar
  selectores sin resolver durante la migración por fases
- `<link>` de Google Fonts reducido de 8 a **5 pesos** (`Anton` +
  `Oswald:500;600` + `Inter:400;500;600`); Oswald reservado a h2 de fase
  y cifras de stats, Inter 600 en tabs/labels/marca/títulos de fila
- `css/background.css`: 364 líneas del collage de 7 capas sustituidas por
  la versión ambiental de 2 capas (~60 líneas netas)
- `css/metal-title.css`: halos `text-shadow` recoloreados a los nuevos
  acentos
- Popover de nota en escritorio restyleado a `--surface-3` + borde
  `--steel-2`; el mecanismo de reposición up/down se mantiene idéntico

### Corregido (QA de cierre, Fase 7)
- Colores heredados de la paleta antigua en el favicon y en el emblema
  SVG de la marca (`.brand-mark`), desalineados de los tokens nuevos:
  recoloreados a `#8b5cf6`/`#60a5fa` (`--accent-fill`/`--accent-2`)
- Área táctil de controles pequeños en móvil verificada/reforzada a
  ≥44×44px (`.rating__trigger::after{inset:-12px}`, grid de 6 columnas
  del bottom sheet con `minmax(44px,1fr)`)
- `z-index` del popover de nota (`.item:has(.rating__popover:not(
  [hidden]))`) insuficiente frente al topbar sticky y la tabs-bar sticky
  de escritorio: subido de `5` a `12` (por encima de topbar `11` y
  tabs-bar `9`)
- Enlaces `window.open(url,'_blank')` sin `rel="noopener"` en los 3
  puntos donde se abren (CTA YouTube, CTA Google, fila clicable de
  Plataforma): añadido

### Pendiente de validar (no cerrado en esta sesión)
- `.stats` a 1 columna en <380px (actualmente fija a `repeat(3,1fr)` en
  todos los breakpoints móviles)
- Posible overflow horizontal de la fila compacta de `.item` a 320px con
  duración larga + nota puntuada (falta medición `scrollWidth ≤
  innerWidth` instrumentada)

---

## [2026-07-23]
### Añadido
- Campo `m` (duración total en minutos; en series, suma de todos los
  episodios) y `est:true` (duración estimada) en los 81 items de `DATA` y
  `XMEN_DATA`, en `index.html`
- Helpers `durationLabel()` y `buildDurationEl()`, y chip `.duration` (con
  icono de reloj) en cada fila del checklist, entre el título y el tag de
  tipo
- Nota aclaratoria sobre duraciones en el `<footer>`
- Pestaña **"Checklist X-Men"** (`#tab-xmen` / `#panel-xmen`) con
  `XMEN_DATA`: 19 títulos de la saga Fox en orden de estreno 1992–2024
  (13 películas + 6 series)
- 19 pósters de la saga X-Men descargados de TMDB (w500) en
  `assets/posters/` y mapeados en `POSTERS_LOCAL` (`js/images-posters.js`)
- `TRACKED_ITEMS` (concatenación de `DATA` + `XMEN_DATA`) como fuente única
  del progreso global (topbar, anillo del hero, stats): total de títulos
  62 → 81
- Separadores `.list-divider` ("Universo Marvel" / "Saga X-Men") en las
  pestañas Resúmenes y Plataforma, en el punto donde empieza la saga X-Men
- `CLAUDE.md`, `DOCUMENTACION.md` y este `CHANGELOG.md` (documentación del
  proyecto, antes inexistente)

### Modificado
- Pestañas renombradas a **"Checklist Marvel"** y **"Checklist X-Men"**, y
  reordenadas para quedar contiguas (orden del DOM y de `TAB_IDS`:
  `checklist, xmen, resumenes, plataforma`)
- Pestañas **Resúmenes** y **Plataforma**: ahora iteran `TRACKED_ITEMS`
  (MCU + X-Men) en vez de solo `DATA`
- `.item` (`css/styles.css`): de fila flex a **grid con
  `grid-template-areas`**; en `@media (max-width:560px)` pasa a 2 filas
  (póster+texto arriba, duración·tag·nota debajo) para descongestionar
  el móvil
- Ritmo vertical: gap de `.grid` 9px → 12px; margen superior de `.phase`
  36px → 48px
- `.duration` rediseñado como texto `muted` plano; la nota (`.rating`)
  queda como único acento de color del clúster derecho de cada fila
- Jerarquía visual título/fecha reforzada en `.item` y `.action-row`
- `.action-row` unificado visualmente con `.item` (mismos radios, hover y
  borde izquierdo de color por tipo)
- `.list-divider` estilizado: tipografía Oswald en mayúsculas + regla
  horizontal
- Tabs: más padding y subrayado activo de 2px
- `footer`: interlineado aumentado para mejorar legibilidad de la nota de
  duraciones

### Corregido
- Typo `wherToWatchUrl` → `whereToWatchUrl` (función y todas sus
  referencias) en el `<script>` inline de `index.html`

### Eliminado
- Array obsoleto `ALL_ITEMS` en `index.html` (sustituido por
  `TRACKED_ITEMS`)
- `.hint` deja de ser una "píldora" de cristal (fondo/borde eliminados;
  pasa a texto plano)

### Incidencias
- Dos sesiones de Claude editaron `index.html` en paralelo, produciendo
  sobrescrituras que descartaron temporalmente la feature de duraciones y
  la pestaña X-Men; ambas se restauraron. Ver detalle y recomendación en
  `DOCUMENTACION.md` (sesión 2026-07-23) y en `CLAUDE.md` § Notas de
  proceso.
