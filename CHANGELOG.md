# Changelog — MCU Tracker

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/).
Este proyecto no usa versionado semántico de paquete (no hay build/release
formal); las entradas se agrupan por fecha de sesión de trabajo.

## [Sin publicar]
### Pendiente
- Protocolo de exclusión mutua para ediciones concurrentes de `index.html`
- Revisar si `DISENO-SPEC.md` (paleta roja/dorada histórica) debe marcarse obsoleto
- Verificación visual en dispositivo real del layout `.item` en ≤560px

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
