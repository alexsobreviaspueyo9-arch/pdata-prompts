# Books — Prompt para Claude Code

Requiere: haber construido ya `prompt-01-dashboard.md` a `prompt-07-analysis.md` (trae el sidebar y el sistema de diseño). No necesita ninguna cuenta externa salvo, opcionalmente, Supabase para sincronizar.

## Objetivo

Una lista de lectura sencilla: qué estás leyendo/pendiente de leer, y al terminar un libro, un registro de qué aprendiste de él.

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md`. Prefijo de clases sugerido: `.bk-*`.

## Añadir al sidebar

Añade este item al `.sidebar-nav` ya existente, justo debajo de "Analysis":
```html
<button type="button" class="sidebar-item" data-view="bookView">
  <span class="sidebar-item-icon" aria-hidden="true"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"/></svg></span>
  Books
</button>
```
Y un `<div id="bookView" class="hidden">` nuevo dentro de `#appRoot`, con el contenido de esta sección.

## Estructura visual

- Un formulario simple para añadir un libro: título (obligatorio) + autor (opcional).
- Dos listas: **Pendientes** y **Terminados**. Cada fila lleva una insignia circular pequeña con un icono (un libro cerrado para pendientes, un check con un ligero glow para terminados — esa es la única diferencia visual entre ambos estados, aparte de en qué lista/tarjeta vive cada fila).
- En un libro pendiente, un botón "Terminar libro" abre un modal para registrar una lista libre de lecciones/aprendizajes (líneas de texto, añadibles/quitables una a una). Al guardar, el libro se mueve a Terminados y esas lecciones se muestran debajo de su fila como una lista con checks, a modo de recordatorio permanente de lo que sacaste de ese libro.

## Funcionalidad e interacciones

- No hace falta portada, valoración numérica ni fecha de inicio — mantenlo deliberadamente simple: título, autor, estado, y (si terminado) sus lecciones.
- Mover un libro de Pendiente a Terminado es una acción única y no reversible desde la interfaz (no hace falta un botón para "deshacer terminado").

## Estructura de datos

```
localStorage:
  book_list_v1 → [{ id, title, author, status: "toread" | "finished",
                     addedAt, finishedAt?, lessons: string[] }]
```

## Sincronización entre dispositivos (opcional)

Aplica el patrón de Supabase de `prompt-01-dashboard.md`, `APP_KEY: 'book'`, sincronizando `book_list_v1`.

## Notas de implementación

- No hardcodees ningún libro real de nadie en el código publicado — que la lista arranque vacía.
- Reutiliza el mismo chasis de modal (fondo oscuro + tarjeta centrada + campos + botones cancelar/confirmar) descrito en `prompt-01-dashboard.md`, no inventes uno nuevo para el modal de "lecciones aprendidas".
