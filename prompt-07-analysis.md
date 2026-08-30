# Analysis — Prompt para Claude Code

Requiere: haber construido ya `prompt-01-dashboard.md` a `prompt-06-sleep.md` (trae el sidebar y el sistema de diseño). No necesita ninguna cuenta externa salvo, opcionalmente, Supabase para sincronizar.

## Objetivo

Una autoevaluación periódica de las grandes áreas de tu vida (salud, relaciones, desarrollo personal, salud mental, propósito...) representada como un gráfico de radar/araña, para ver de un vistazo qué área está más floja y necesita atención.

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md`. Prefijo de clases sugerido: `.an-*`.

## Añadir al sidebar

Añade este item al `.sidebar-nav` ya existente, justo debajo de "Sleep":
```html
<button type="button" class="sidebar-item" data-view="analysisView">
  <span class="sidebar-item-icon" aria-hidden="true"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg></span>
  Analysis
</button>
```
Y un `<div id="analysisView" class="hidden">` nuevo dentro de `#appRoot`, con el contenido de esta sección.

## Estructura visual

- Un botón "Puntuar" abre un modal con un slider de 0 a 10 por cada sub-área, agrupados visualmente bajo su pilar. Define tú mismo los pilares y sub-áreas — una estructura de ejemplo razonable:
  ```
  Salud física: nutrición, ejercicio, descanso
  Relaciones: familia, amistades, comunidad
  Desarrollo personal: aprendizaje, objetivos, crecimiento
  Salud mental: disciplina, paz interior
  Propósito: sentido, pasión, ayudar a otros
  ```
- Al guardar, cada pilar se calcula como la media de sus sub-áreas, y esos 4-5 valores se dibujan en un **gráfico de radar/araña** (SVG dibujado a mano: un eje por pilar repartido en círculo, líneas de rejilla concéntricas de fondo en gris muy tenue, y un polígono relleno con el color de acento de la app conectando el valor de cada eje).
- El pilar con la puntuación media más baja se resalta automáticamente (un borde/glow especial en su tarjeta o etiqueta, más una pequeña marca de "necesita atención") — pero solo una vez que ya se ha puntuado al menos una vez; antes de eso, no resaltes nada.

## Funcionalidad e interacciones

- Guardar una puntuación nueva sobrescribe siempre el conjunto completo de valores actuales — no es un histórico con fecha por entrada, es "tu última autoevaluación".
- El propio gráfico de radar y los sliders del modal deben derivarse de la misma lista de pilares/sub-áreas — si añades o quitas un sub-área de esa lista, tanto el modal como el gráfico deben reflejarlo sin tocar nada más.

## Estructura de datos

```
localStorage:
  analysis_scores_v1 → { subScores: { [idSubArea]: number (0-10) }, lastScoredAt: ISOString }
```

## Sincronización entre dispositivos (opcional)

Aplica el patrón de Supabase de `prompt-01-dashboard.md`, `APP_KEY: 'analysis'`, sincronizando `analysis_scores_v1`.

## Notas de implementación

- El gráfico de radar es SVG dibujado a mano, sin librerías externas.
- No hardcodees ninguna puntuación real de nadie en el código publicado — que arranque sin ninguna puntuación guardada, listo para que el usuario haga su primera autoevaluación.
