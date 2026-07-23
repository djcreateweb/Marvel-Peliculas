# CLAUDE.md — MCU Tracker

## Qué es

Tracker de visionado del Universo Cinematográfico Marvel (MCU) + saga X-Men.
Aplicación de un único fichero HTML (`index.html`) con CSS y JS separados en
carpetas propias, sin build step, sin dependencias, sin backend. Se abre
directamente en el navegador (`file://` o cualquier hosting estático).

- HTML5 + CSS3 + JavaScript vanilla (ES2017, IIFE, `'use strict'`)
- Persistencia 100% en `localStorage` del navegador (sin servidor, sin cuentas)
- Fuentes: Google Fonts (Anton, Oswald, Inter) vía `<link>` en `<head>`
- Repo git **anidado** dentro del monorepo del Orquestador — no hacer commits
  automáticos aquí salvo petición explícita del usuario

## Estructura de ficheros

```
index.html                  Documento único: markup + <script> inline con
                             todo el JS de la app (DATA, XMEN_DATA, lógica).
css/
  tokens.css                :root — paleta "Endgame" (lila/azul + acero),
                             tipografía, espaciado, sombras. Debe cargarse
                             antes que metal-title.css y styles.css.
  metal-title.css           Efecto metálico del <h1> del hero (.metal-title).
  styles.css                Layout y componentes (tabs, .item, .action-row,
                             rating popover, responsive ≤560px).
  background.css            Fondo animado fijo (.bg-fx), independiente del
                             resto — no se toca desde styles.css.
js/
  images-posters.js         POSTERS_LOCAL (mapa título → ruta jpg) +
                             HERO_LOCAL (imagen de fondo del hero). Se
                             carga con <script src> ANTES del <script>
                             inline de index.html.
assets/posters/*.jpg        81 pósters descargados de TMDB (w500), uno por
                             cada item de DATA + XMEN_DATA.
assets/hero-vengadores.jpg  Imagen de fondo del hero cinematográfico.
design/, DISENO-*.md,
disney-links.md             Documentos de diseño/spec previos (histórico,
                             no autoritativos frente al código real).
```

No hay `package.json`, build tool ni tests automatizados. Toda la lógica de
la app vive en el `<script>` inline de `index.html` (líneas ~161–856).

## Datos: `DATA` y `XMEN_DATA`

Ambos son arrays de fases (`{phase, years, items:[...]}`) definidos dentro
del `<script>` inline de `index.html`. Cada item tiene:

| Campo | Tipo | Significado |
|-------|------|--------------|
| `t` | string | **Identidad del título.** Ver contrato intocable abajo. |
| `d` | string | Fecha/rango de estreno mostrada en la UI. |
| `type` | `"film"` \| `"serie"` \| `"especial"` | Determina tag, color y tinte de póster. |
| `m` | number | Duración total en minutos (en series: suma de todos los episodios). |
| `est` | boolean (opcional) | `true` = duración estimada, se muestra con prefijo «≈». |
| `q` | string (opcional) | Término de búsqueda alternativo para Google/YouTube si `t` no basta. |

`DATA` = 6 fases del MCU (Fase 1 a Fase 6), 62 items.
`XMEN_DATA` = 1 fase ("Películas y series", 1992–2024), 19 items.
`TRACKED_ITEMS` = concatenación de todos los items de `DATA` + `XMEN_DATA`
(81 en total) y es la fuente única del progreso global (topbar, anillo del
hero, stats, y el orden de listado en Resúmenes/Plataforma).

## Contratos intocables

Estas reglas existen porque romperlas corrompe datos de usuario ya
guardados o rompe la navegación por pestañas. Cualquier cambio futuro debe
respetarlas o migrar explícitamente los datos existentes.

1. **`localStorage` keys**: `"mcu_checklist_v3"` (progreso de visto/no
   visto) y `"mcu_ratings_v1"` (notas 0–10). No renombrar ni versionar sin
   escribir una migración explícita — de lo contrario el usuario pierde
   todo su progreso guardado.
2. **Identidad = campo `t`**: el título (`it.t`) es la clave primaria en
   `state`, `ratings`, `POSTERS_LOCAL`, `itemRefs` y `titleToPhase`. No
   renombrar el texto de un `t` ya publicado — eso desconecta el progreso
   guardado de ese título (queda huérfano en localStorage y el nuevo título
   aparece como "no visto").
3. **`TAB_IDS` debe ir siempre en el mismo orden que los botones `.tab`
   del DOM.** Actualmente: `['checklist','xmen','resumenes','plataforma']`,
   igual que `#tab-checklist`, `#tab-xmen`, `#tab-resumenes`,
   `#tab-plataforma` en el HTML. El routing por hash (`tabNameFromHash`) y
   la navegación por teclado (flechas/Home/End) dependen de este orden.
4. **`buildXmen()` debe llamarse después de `buildChecklist()`** en el
   bootstrap final del script. `buildChecklist()` limpia (`.clear()`)
   `itemRefs` y `phaseRefs` antes de reconstruirlos; si `buildXmen()` se
   ejecutara antes, `buildChecklist()` borraría sus referencias.
5. **`.rating` necesita `position:relative`** (ancla del popover
   absolutamente posicionado) **y ni `.item` ni `.grid` deben tener
   `overflow:hidden`** — el popover de nota (0–10) se renderiza fuera de
   los límites de la fila y necesita poder desbordar visualmente.

## Notas de proceso

- Cuando se trabaje en `index.html`, usar **una sola sesión/agente a la
  vez**. Editar el mismo fichero en paralelo desde dos sesiones produce
  sobrescrituras silenciosas (ver incidencia registrada en
  `DOCUMENTACION.md`, sesión 2026-07-23).
- Tras cualquier edición de `index.html`, verificar con `grep`/lectura que
  `DATA`, `XMEN_DATA`, `TAB_IDS` y las funciones `build*` siguen presentes
  y completas antes de dar la tarea por cerrada.
