# 📄 Documentación Técnica — MCU Tracker

---

## Sesión: 2026-07-23 — Duraciones totales, pestaña X-Men y revisión integral

### 🎯 Objetivo
Ampliar el tracker de visionado del MCU con (A) la duración total de cada
título y (B) una pestaña dedicada a la saga X-Men de Fox que sume al
progreso global, y (C) revisar de forma integral la integración de ambas
features en el HTML y el CSS tras detectarse sobrescrituras por edición
paralela.

### 👥 Agentes involucrados
- `frontend-vanilla-agent` — Bloques A y B (datos, helpers, markup del
  checklist y de la pestaña X-Men) y Fase 1 de la revisión (`index.html`)
- `diseno-agent` — Fase 2 de la revisión: layout y estilos de `.item` en
  grid, jerarquía visual del clúster duración·tag·nota (`css/styles.css`)

---

## BLOQUE A — Duraciones totales (feature)

### 📁 Archivos modificados
| Archivo | Cambios |
|---------|---------|
| `index.html` | Campo `m` (minutos totales) y `est:true` (estimada) añadidos a los 81 items de `DATA` y `XMEN_DATA`; helpers `durationLabel()` / `buildDurationEl()`; chip `.duration` insertado en `buildItemRow()` entre el título y el tag; nota aclaratoria añadida al `<footer>` |
| `css/styles.css` | Regla `.duration` (texto plano `muted`, icono reloj, `opacity:.55` cuando `.item.done`) |

### Detalle de implementación

En series, `m` es la **suma de todos los episodios** (p. ej. `WandaVision`
→ `m:360`; `X-Men: La Serie Animada` → `m:1670`). El flag `est:true` marca
títulos sin metraje oficial confirmado (todos los estrenos 2026 en
adelante: `Wonder Man`, `Daredevil: Born Again` T2, `Spider-Man: Brand New
Day`, `VisionQuest`, `Vengadores: Doomsday`).

```javascript
// index.html — helpers de duración
function durationLabel(it){
  if(!it.m) return '';
  const h=Math.floor(it.m/60), min=it.m%60;
  const txt = h===0 ? min+' min' : (min===0 ? h+' h' : h+' h '+min+' min');
  return (it.est?'≈ ':'')+txt;
}
function buildDurationEl(it){
  const dur=durationLabel(it);
  if(!dur) return null;
  const el=document.createElement('span');
  el.className='duration';
  el.title='Duración total'+(it.est?' (estimada)':'');
  el.innerHTML=CLOCK;
  el.appendChild(document.createTextNode(dur));
  return el;
}
```

El chip se inserta en `buildItemRow()` justo entre `meta` (título+fecha) y
`tag`:

```javascript
row.append(poster,meta);
const durEl=buildDurationEl(it);
if(durEl) row.append(durEl);
row.append(tag,buildRatingControl(it));
```

Nota añadida al footer (`index.html` línea 157):
> «Duraciones totales aproximadas (en series, suma de todos los
> episodios); «≈» indica una estimación para títulos sin metraje
> confirmado.»

### 🗃️ Datos — Cambios
Sin base de datos (localStorage puro). Cambio de esquema en los objetos
literales `DATA`/`XMEN_DATA` del script inline: los 81 items ganan `m`
(y 5 de ellos, `est:true`). No requiere migración de `localStorage`
porque `m`/`est` no se persisten — son metadatos estáticos del catálogo,
no estado del usuario.

---

## BLOQUE B — Pestaña X-Men (feature)

### 📁 Archivos creados
| Archivo | Descripción |
|---------|-------------|
| `assets/posters/x-men-*.jpg` (19 ficheros) | Pósters de la saga X-Men descargados de TMDB (w500) |

### ✏️ Archivos modificados
| Archivo | Cambios |
|---------|---------|
| `index.html` | `XMEN_DATA` (19 títulos, 13 películas + 6 series, orden de estreno 1992–2024); `TRACKED_ITEMS` = `DATA`+`XMEN_DATA` como fuente única del progreso global; botón/panel de pestaña `#tab-xmen`/`#panel-xmen`; `buildXmen()` reutilizando `buildPhase()`/`buildItemRow()` |
| `js/images-posters.js` | 19 entradas nuevas en `POSTERS_LOCAL` bajo el comentario `// Saga X-Men (pestaña "X-Men")` |
| `css/styles.css` | `#panel-xmen .panel-intro h2{ border-bottom:2px solid var(--especial); }` |

### Detalle: `XMEN_DATA`

```javascript
const XMEN_DATA=[
 {phase:"Películas y series",years:"1992 – 2024",items:[
  {t:"X-Men: La Serie Animada",d:"1992 – 1997",type:"serie",q:"X-Men serie animada 1992",m:1670},
  {t:"X-Men",d:"Agosto 2000",type:"film",q:"X-Men 2000 pelicula",m:104},
  // ... 17 items más, hasta:
  {t:"X-Men '97",d:"Marzo 2024",type:"serie",q:"X-Men 97 serie Disney+",m:290},
 ]},
];
```

El progreso global ahora suma `DATA` (62 items) + `XMEN_DATA` (19 items) =
**81 items** (antes 62), vía:

```javascript
const ALL_PHASES = DATA.concat(XMEN_DATA);
const TRACKED_ITEMS = ALL_PHASES.reduce((acc,ph)=>acc.concat(ph.items),[]);
```

`buildXmen()` reutiliza `buildPhase()` y por tanto `buildItemRow()` — misma
fila con tick, póster, duración, tag y control de nota que en el checklist
MCU:

```javascript
function buildXmen(){
  const cont=document.getElementById('xmenList');
  cont.innerHTML='';
  XMEN_DATA.forEach(ph=>cont.appendChild(buildPhase(ph)));
}
```

### 🗃️ Datos — Cambios
81 pósters (62 MCU + 19 X-Men) verificados en disco en
`assets/posters/*.jpg`; todos mapeados por título exacto en
`js/images-posters.js` → `POSTERS_LOCAL`.

---

## BLOQUE C — Revisión integral (plan del orquestador, 2 fases)

### Fase 1 — `frontend-vanilla-agent` (`index.html`)
- Pestañas renombradas: **"Checklist Marvel"** y **"Checklist X-Men"**,
  colocadas de forma contigua tanto en el DOM (orden de botones `.tab` y
  paneles `.tab-panel`) como en `TAB_IDS`:
  `['checklist','xmen','resumenes','plataforma']`.
- Paneles reordenados para que `#panel-xmen` siga inmediatamente a
  `#panel-checklist`.
- **Resúmenes** y **Plataforma** dejan de listar solo `DATA`: ahora iteran
  `TRACKED_ITEMS` completo (MCU primero, X-Men al final), insertando un
  separador `.list-divider` («Universo Marvel» / «Saga X-Men») en el punto
  de transición:

  ```javascript
  function buildResumenes(){
    const cont=document.getElementById('resumenesList');
    cont.innerHTML='';
    const firstXmenTitle=XMEN_DATA[0].items[0].t;
    TRACKED_ITEMS.forEach((it,idx)=>{
      if(idx===0){ /* ...append divider "Universo Marvel"... */ }
      if(it.t===firstXmenTitle){ /* ...append divider "Saga X-Men"... */ }
      cont.appendChild(buildActionRow(it));
    });
  }
  ```
  (Mismo patrón en `buildPlataforma()`.)
- Limpieza de JS: eliminado el array obsoleto `ALL_ITEMS` (sustituido por
  `TRACKED_ITEMS`); `wherToWatchUrl` renombrada a `whereToWatchUrl` (typo
  corregido, y actualizadas todas sus referencias); comentario obsoleto de
  cabecera compactado.

### Fase 2 — `diseno-agent` (`css/styles.css`)
- `.item` pasa de fila flex a **grid con `grid-template-areas`**
  (`"poster meta duration tag rating"` en escritorio). En `@media
  (max-width:560px)` se reorganiza a 2 filas —
  `"poster meta meta meta meta"` / `"poster duration tag rating ."` —
  descongestionando el móvil (póster+texto arriba, duración·tag·nota
  debajo, sin tocar ninguna clase del HTML):

  ```css
  .item{
    display:grid;
    grid-template-columns:var(--poster-w) 1fr auto auto auto;
    grid-template-areas:"poster meta duration tag rating";
    ...
  }
  @media (max-width:560px){
    .item{
      grid-template-columns:var(--poster-w) auto auto auto 1fr;
      grid-template-areas:
        "poster meta     meta  meta   meta"
        "poster duration tag   rating .";
      row-gap:8px; column-gap:10px;
    }
  }
  ```
- Ritmo vertical ajustado: gap de `.grid` 9→12px, margen superior de
  `.phase` 36→48px.
- `.duration` rediseñado como **texto muted plano** (ya no compite
  visualmente); la nota (`.rating`) queda como único acento de color del
  clúster derecho.
- Jerarquía título/fecha reforzada (`.title` en negrita, `.date` en
  `--muted`).
- `.hint` deja de ser una "píldora" de cristal — pasa a texto plano
  centrado, sin fondo ni borde (`background:none;border:none;`).
- `.action-row` unificado visualmente con `.item` (mismos radios, mismo
  hover con `translateY(-1px)` + `box-shadow`, mismo borde izquierdo de
  color por tipo).
- `.list-divider` con tipografía Oswald uppercase + regla horizontal
  (`::after{ flex:1 1 auto; height:1px; background:var(--border); }`).
- Tabs con más padding (`14px 22px`) y subrayado activo de 2px
  (`.tab.active::after`).
- `footer` con mejor interlineado (`line-height:1.75`) para la nota de
  duraciones añadida en el Bloque A.
- **Paleta y tipografías intactas**: `css/tokens.css` sin cambios en esta
  fase.

---

## 🐛 Incidencia — Edición paralela de `index.html`

**Qué pasó**: durante la jornada del 2026-07-23, dos sesiones de Claude
distintas editaron `index.html` en paralelo (sin coordinación entre
ellas). Una sesión sobrescribió a la otra, perdiendo temporalmente tanto
la feature de duraciones (Bloque A) como la pestaña X-Men (Bloque B), que
tuvieron que restaurarse.

**Causa raíz**: no hay bloqueo de fichero ni coordinación entre sesiones
de agente que editan el mismo `index.html` de forma concurrente; la
última sesión en guardar gana y descarta los cambios de la otra.

**Solución aplicada**: se restauraron manualmente ambas features
(duraciones + pestaña X-Men) en `index.html`.

**Recomendación** (ver también `CLAUDE.md` § Notas de proceso):
1. Una sola sesión/agente trabajando sobre `index.html` a la vez.
2. Tras cada edición, verificar con `grep`/lectura completa que las
   piezas clave (`DATA`, `XMEN_DATA`, `TAB_IDS`, funciones `build*`)
   siguen presentes antes de cerrar la tarea.

---

## ✅ Verificaciones realizadas (estado al cierre de la sesión)

| Verificación | Resultado |
|---|---|
| `node --check` del `<script>` inline extraído de `index.html` | OK — sin errores de sintaxis |
| Llaves `{`/`}` de `css/styles.css` | Balanceadas — 192 aperturas / 192 cierres |
| Pósters de los 81 items mapeados en `POSTERS_LOCAL` | Los 81 ficheros existen en `assets/posters/` |
| Orden de pestañas en el DOM (`.tab`) vs. `TAB_IDS` | Coinciden: `checklist, xmen, resumenes, plataforma` |

---

## 📚 Referencias de diseño internas
- `design/DISENO-SISTEMA-ENDGAME.md` — paleta "Endgame" (lila/azul/acero) y
  tabla de migración de tokens (base de `css/tokens.css`)
- `design/DISENO-RATING-V2.md` — patrón ARIA "Collapsible Dropdown
  Listbox" del control de nota 0–10 (`.rating`)

## ⏭️ Próxima sesión — Pendiente
- [ ] Definir protocolo de exclusión mutua (o al menos un checklist de
      verificación post-edición) para evitar nuevas sobrescrituras de
      `index.html` en trabajos paralelos.
- [ ] Revisar si `DISENO-SPEC.md` (paleta "Marvel clásica" roja/dorada,
      histórica) debe marcarse como obsoleto ahora que `tokens.css` usa
      la paleta "Endgame".
- [ ] Confirmar visualmente en dispositivo real el layout de `.item` en
      ≤560px (Bloque C, Fase 2) tras el cambio a grid de 2 filas.

---
*Documentado por: documentacion-agent · 2026-07-23*
