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
- **Mapa de tiros / heatmap de cancha**: el mockup de diseño mostraba un gráfico de calor con datos de ejemplo inventados (no hay forma de trackear la ubicación o tipo de cada tiro con el wizard actual, que solo registra el resultado final del partido). En vez de fabricar datos falsos, reemplacé esa sección por un desglose real de rendimiento por juego (Pussy/Seven/21) y contadores de fantasy/MPP/2º puesto — mismo estilo visual, pero con datos reales.
- **Logros** ("de una vez" vs "estado actual"): Invicto y Sequía reflejan la racha actual (se pueden perder). El resto (Fantasy, Escoba, Eterno 2º, Pussy Master, Doble Cara, Rey del Aro) quedan desbloqueados para siempre una vez alcanzados, mostrando el primer jugador que cumplió la condición.
- **Fotos de jugadores**: solo había asset real para Miguelón (`foto-miguelon.png` del handoff). Murrey y Mojarrita usan un avatar con su inicial, igual que el propio mockup de diseño mostraba para Mojarrita.
- **"Avisar al grupo"** (Fixture): en el mockup era un botón sin funcionalidad. Lo conecté a `navigator.share` (o copiar al portapapeles como fallback) para que sea útil.
- Borrar un partido o una fecha del fixture muestra un toast con "Deshacer" por 5s, para evitar pérdidas accidentales de datos (no estaba en el mockup pero me pareció una buena práctica ya que no hay backend/backup).
