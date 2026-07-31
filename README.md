# Liga de Básquet

App estática (un solo `index.html`, sin build ni framework) para llevar la liga de básquet entre Murrey, Miguelón y Mojarrita: tabla de posiciones, carga de partidos, fixture, logros e historial.

## Cómo correrla

No requiere build. Basta con servir el directorio como archivos estáticos:

```
python3 -m http.server 8080
```

y abrir `http://localhost:8080`.

## Deploy en Netlify

El repo ya incluye `netlify.toml` (`publish = "."`, sin build command). Al conectar el repo en Netlify, con el branch `claude/netlify-project-wkqhnj` alcanza con darle "Deploy site" — no hace falta configurar nada más.

## Datos

Todo se guarda en `localStorage` (con fallback a `window.storage` si existiera) bajo la key `ligabasquet.v2`. No hay backend: cada dispositivo/navegador tiene su propia liga.

## Decisiones de diseño no especificadas en el handoff

El handoff de diseño (`.dc.html`) es un mockup visual con datos de ejemplo hardcodeados, no una app funcional. Al portarlo a lógica real con las reglas nuevas (Pussy/Seven/21, modo "todos" vs "1v1"), tuve que resolver algunas reglas de negocio que no estaban explícitas. Si algo no es lo que esperabas, decímelo y lo ajusto:

- **Puntos en la tabla**: cada partido válido (sea de 3 o 1v1) suma 3 puntos solo al ganador (1er puesto en modo "todos", mayor marcador en 1v1). PJ cuenta a todos los participantes, PP a los que no ganaron. La diferencia de puntos (DIF) solo se calcula con partidos 1v1 (los partidos de a 3 no tienen marcador numérico, así que no aportan a la diferencia).
- **Cara a cara**: en partidos de a 3, cada posición del podio "le gana" a las posiciones detrás suyo (1º le gana a 2º y 3º, 2º le gana a 3º) para alimentar la grilla de cara a cara.
- **Sugerencia de partido 1v1** (paso 3 del wizard en modo "Solo 2"): como las reglas nuevas ya no usan el fixture de rondas fijas del original, sugiero la pareja que jugó menos veces entre sí (y en caso de empate, la que hace más tiempo no juega), en vez de una rotación fija de rondas.
- **MPP**: solo se pregunta en modo "Los 3" (partidos de a 3), igual que en el diseño. El logro "Pussy Master" solo cuenta los MPP de partidos de Pussy (por el nombre del logro), aunque el dato de MPP se guarda para cualquier juego jugado de a 3.
- **Mapa de tiros ganadores**: en el paso de "Detalles del partido" del wizard (paso 4), además de los toggles, se toca sobre una media cancha en miniatura para marcar dónde entró el tiro que ganó el partido, y se elige el tipo (Simple/Doble/Triple) — salvo cuando el tipo ya está determinado por la regla del juego: si "Tiro fantasy" está activo (Pussy) se guarda como Fantasy sin preguntar, y en Seven siempre se guarda como Triple (es el único tiro que define ese juego). Esa posición y tipo se guardan por partido (`match.shot`) y alimentan el mapa de calor real (agrupado en hexágonos por zona) y el desglose de % por tipo de tiro que se ve en el detalle de cada jugador — nada de datos de ejemplo. Solo se registra el tiro del ganador, tal como lo pedía el mockup original.
- **Logros** ("de una vez" vs "estado actual"): Invicto y Sequía reflejan la racha actual (se pueden perder). El resto (Fantasy, Escoba, Eterno 2º, Pussy Master, Doble Cara, Rey del Aro) quedan desbloqueados para siempre una vez alcanzados, mostrando el primer jugador que cumplió la condición.
- **Fotos de jugadores**: solo había asset real para Miguelón (`foto-miguelon.png` del handoff). Murrey y Mojarrita usan un avatar con su inicial, igual que el propio mockup de diseño mostraba para Mojarrita.
- **"Avisar al grupo"** (Fixture): en el mockup era un botón sin funcionalidad. Lo conecté a `navigator.share` (o copiar al portapapeles como fallback) para que sea útil.
- Borrar un partido o una fecha del fixture muestra un toast con "Deshacer" por 5s, para evitar pérdidas accidentales de datos (no estaba en el mockup pero me pareció una buena práctica ya que no hay backend/backup).

## Fuentes y tamaños

Tres familias, **auto-hospedadas** en `assets/fonts/*.woff2` (no se cargan desde Google Fonts por red — se embeben en el `<head>` vía `@font-face`). Se eligió así por dos motivos: la app es estática y sin backend, así que no depender de un CDN externo es consistente con esa filosofía; y de paso evita que la tipografía se rompa si el navegador no puede alcanzar `fonts.googleapis.com` (ad-blockers, redes restringidas, etc.) — cosa que efectivamente pasó al probar la app en un entorno con red restringida y confirmó que las capturas que mandé antes se habían renderizado con la fuente de reemplazo del sistema, no con el bug real. Solo se empaquetan los pesos que la app realmente usa:

- **Anton** (peso único 400) — títulos de pantalla y nombres propios, siempre en mayúsculas (`text-transform:uppercase`). 30-34px en títulos de pantalla, 18-24px en nombres/headers de sección, 13-19px en nombres dentro de filas/chips.
- **Teko** (peso 600) — números grandes tipo marcador (puntos, stats). 40-56px en números destacados (puntos de tabla, marcador del wizard), 20-30px en stats secundarias.
- **Barlow** (500/600/700/800) — el resto de la UI: labels, botones, descripciones, chips. 8-9px en labels mayúsculas con letter-spacing .1-.3em, 10-13px en botones/toggles, 14-16px en CTAs principales.

Tamaño exacto por elemento, auditado directamente contra los mockups `.dc.html` del handoff (útil si se toca el CSS y hay que verificar que no se desvíe):

| Pantalla | Elemento | Fuente / peso | Tamaño |
|---|---|---|---|
| Todas | Kicker "Liga de básquet" | Barlow 700 | 9px, letter-spacing .3em |
| Todas (Cargar/Fixture/Logros/Historial) | Título de pantalla | Anton 400 | 30px |
| Tabla | Título "TABLA" | Anton 400 | **32px** (única pantalla con este tamaño) |
| Tabla | Cinturón — label / nombre | Barlow 700 / Anton 400 | 9px / 18px |
| Tabla | Sub-meta ("N partidos jugados") | Barlow 600 | 10px, letter-spacing .04em |
| Tabla | Filter chips (Todos/Pussy/Seven/21) | Barlow 700 | 10px, letter-spacing .04em |
| Tabla | Rank number (1,2,3 en fila) | Teko 600 | 36-40px |
| Tabla | Nombre de jugador en fila | Anton 400 | 20px |
| Tabla | Racha (badge rojo junto al nombre) | Barlow 700 | 9px |
| Tabla | Puntos (columna derecha) | Teko 600 | 40px |
| Tabla | "Próximo sugerido" — nombre / CTA | Anton 400 / Barlow 700 | 15px / 10px |
| Tabla | Cara a cara — celdas | Barlow 700 | 14px |
| Detalle jugador | Foto hero | — | alto **280px** |
| Detalle jugador | Nombre sobre la foto | Anton 400 | **34px** |
| Detalle jugador | Stat tiles (PJ/DIF/RACHA/CINT) | Teko 600 / Barlow 700 | 24px valor / 8px label |
| Detalle jugador | Rendimiento por juego | Anton 400 | 14px |
| Cargar | Step meta / step kicker | Barlow 600 / 700 | 10px / 9px |
| Cargar | Pregunta de paso ("¿Jugaron los 3...?") | Anton 400 | **21px** |
| Cargar | Nombre en choice-chip (juego/modo) | Anton 400 | 19px |
| Cargar | Descripción en choice-chip | Barlow 500 | 11px |
| Cargar | Nombre en podio-slot / pod-chip | Anton 400 | 15-16px |
| Cargar | Marcador (steppers) | Teko 600 | 44px |
| Cargar | Label del juego en paso Confirmar | Barlow 700 | **10px, letter-spacing .14em** (no el label genérico de 9px/.16em) |
| Cargar | Marcador final en Confirmar | Teko 600 | 46px |
| Cargar | CTA principal ("Continuar"/"Guardar") | Barlow 700/800 | 15-16px |
| Fixture | "Próximo encuentro" — fecha / cuenta regresiva | Anton 400 / Teko 600 | 24px / 22px |
| Fixture | Botón "Agendar y avisar" | Barlow 700 | **14px** (no 16px — es más chico que los CTA del wizard) |
| Fixture | Toggle "Se repite todas las semanas" | Barlow 700 / 500 | **12px / 10px** (más chico que los toggles de Cargar, que son 13px/11px) |
| Fixture | Fecha agendada — día / mes | Teko 600 / Barlow 700 | 26px / 9px |
| Logros | Nombre del logro | Anton 400 | 13px |
| Logros | Quién lo ganó / descripción | Barlow 700 / 500 | 11px / 10px |
| Historial | Filter chips | Barlow 700 | **11px, letter-spacing .02em** (distinto del filtro de Tabla: 10px/.04em) |
| Historial | Badge de juego + badges de cinturón/fantasy/MPP | Barlow 700 | **9px** (no 11px — son más chicos que los badges del wizard) |
| Historial | Podio en cada partido (puesto / nombre) | Teko 600 / Barlow 700 | 11px / 12px |

Íconos (todos son los PNG/SVG que ya venían en `assets/` del handoff, recoloreados vía `mask`):

| Ícono | Tamaño |
|---|---|
| Nav inferior (Tabla/Fixture/Logros) | 19×19px |
| Nav inferior "Historial" (SVG inline) | 19×19px |
| FAB central (pelota) | círculo de 58px, ícono al 78% (inactivo) o 100% (activo, en Cargar) |
| Trofeos de Logros | 26×26px |
| Botón "volver" en detalle de jugador | círculo de 34×34px |
| Avatares de jugador | 48×64px (fila de tabla / score card), 36×48px (chips de podio/MPP), 56×76px (matchup 1v1) |
