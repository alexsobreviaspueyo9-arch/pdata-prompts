# Landing — Prompt para Claude Code

Requiere: haber construido ya `prompt-01-dashboard.md` hasta `prompt-09-expenses.md` (el sidebar y las 9 secciones tienen que existir ya) — esta es la última pieza: una pantalla de bienvenida que se muestra antes de entrar a la app, y que conecta con todo lo que ya construiste.

## Objetivo

Una pantalla de bienvenida a pantalla completa que se ve antes de entrar a la app: un logo pequeño arriba, un saludo según la hora del día, una frase motivadora que cambia cada vez que se entra, y un botón que lleva al dashboard con todas las secciones ya construidas. Nada más — sin tarjetas de resumen, sin estadísticas, sin nada debajo del botón.

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md` (tema oscuro, acento naranja en degradado con glow, sin librerías externas). Prefijo de clases sugerido: `.landing-*`.

## Estructura HTML

Un `<div id="landingView">` como hermano directo de `#appRoot` (el que ya tienes con el sidebar y las 9 secciones), visible por defecto — `#appRoot` empieza oculto (`class="hidden"`) hasta que se pulsa el botón.

```html
<div id="landingView">
  <div class="landing-bg-arc" id="landingBgArc" aria-hidden="true"></div>
  <canvas class="landing-particles" id="landingParticles" aria-hidden="true"></canvas>

  <div class="landing-top-logo app-logo" aria-hidden="true"><span class="app-logo-text">PD</span></div>

  <div class="landing-container">
    <div class="landing-hero-section">
      <div class="landing-content">
        <span class="landing-badge lw-in" style="--lw-stagger:0" id="landingGreeting">Good morning</span>
        <div class="landing-date lw-in" style="--lw-stagger:0" id="landingDate"></div>
        <h1 class="landing-title lw-in" style="--lw-stagger:1" id="landingQuote">Small sessions. Big engine.</h1>
        <button type="button" class="landing-cta lw-in" style="--lw-stagger:2" id="landingCta">
          Go to Dashboard
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="12" x2="19" y2="12"></line><polyline points="12 5 19 12 12 19"></polyline></svg>
        </button>
      </div>
    </div>
  </div>
</div>
```

No añadas nada más dentro de `#landingView` — nada de tarjetas, estadísticas rápidas, rachas, ni ningún otro bloque debajo del botón. Solo el logo, el saludo, la fecha, la frase y el botón, todo centrado en el alto completo de la pantalla.

## Estilos

- **`.landing-top-logo`** — reutiliza exactamente la misma receta `.app-logo` del sidebar (cuadrado redondeado de 52px con el glow interior en dos pseudo-elementos, ver `prompt-01-dashboard.md`), fijo arriba a la izquierda (`position: fixed; top: max(20px, env(safe-area-inset-top)); left: max(20px, env(safe-area-inset-left))`), sin interacción (es puramente decorativo aquí, a diferencia del logo del sidebar que sí es un botón).
- **`#landingView`** — pantalla completa (`position: fixed; inset: 0`), fondo casi negro (`#050506`), scroll vertical permitido si hiciera falta, contenido centrado horizontalmente.
- **`.landing-container`** — ancho máximo 1100px, centrado con `margin: 0 auto`.
- **`.landing-hero-section`** — `min-height: 100vh` (y `100dvh` como fallback), `display:flex; align-items:center; justify-content:center` — así el conjunto badge+fecha+frase+botón queda perfectamente centrado verticalmente en la pantalla, sin necesidad de hacer scroll para verlo.
- **`.landing-badge`** — una píldora pequeña: `padding: 7px 16px`, fondo `rgba(255,255,255,0.05)`, borde `1px solid rgba(255,255,255,0.10)`, `border-radius: 999px`, texto 12.5px.
- **`.landing-date`** — texto pequeño (12px), color terciario, debajo del badge.
- **`.landing-title`** — la frase motivadora, como titular grande: `font-size: clamp(28px, 5.5vw, 46px)`, peso 800, `max-width: 60%` de su contenedor (100% en pantallas ≤640px), centrado.
- **`.landing-cta`** — el botón principal de toda la app (píldora, degradado naranja, halo/glow — la misma receta de `.btn-primary` de `prompt-01-dashboard.md`), con una flecha SVG simple a la derecha del texto.
- **`.landing-bg-arc`** — el elemento decorativo más importante de esta pantalla: una gran cúpula naranja brillante pegada a la parte inferior de la pantalla, como un horizonte curvo iluminado. Constrúyela como una caja ancha y baja con solo las esquinas superiores redondeadas al máximo, no como un círculo gigante escondido a medias (esa aproximación es más difícil de encajar bien):
  ```css
  .landing-bg-arc {
    position: fixed; left: 50%; bottom: 0;
    width: 145vw; max-width: 1900px;
    height: 34vh; min-height: 220px; max-height: 380px;
    transform: translateX(-50%);
    border-radius: 50% 50% 0 0 / 100% 100% 0 0;
    border: 1.5px solid rgba(255,146,74,0.85); border-bottom: none;
    background: radial-gradient(ellipse at 50% 0%, rgba(232,73,29,0.42) 0%, rgba(60,22,9,0.3) 45%, rgba(5,5,6,0) 82%);
    box-shadow: 0 -10px 60px 6px rgba(255,120,50,0.28);
    z-index: 0; pointer-events: none;
  }
  ```
  El truco es el shorthand de `border-radius` con dos valores por eje (`horizontal / vertical`): usando el 100% de la propia altura de la caja como radio vertical, una caja plana se convierte en una cúpula/semi-elipse sin ninguna geometría oculta que calcular — la altura de la caja *es* la altura visible del arco.
- **`.landing-particles`** — un `<canvas>` de fondo, `position: fixed; inset: 0`, con ~25 puntos pequeños a baja opacidad flotando muy lentamente hacia arriba (un `requestAnimationFrame` sencillo, `clearRect` + `arc` por punto en cada frame). Pausa el bucle de animación cuando la pestaña no está visible (`visibilitychange`) y no lo arranques en absoluto si `prefers-reduced-motion: reduce` está activo (dibuja un único frame estático en ese caso).
- **Entrada escalonada** (`.lw-in` + `--lw-stagger`): cada bloque del hero aparece con un pequeño fundido + desplazamiento hacia arriba al cargar, con un retraso creciente según su `--lw-stagger` (0, 1, 2...) — dentro de un `@media (prefers-reduced-motion: no-preference)`, para que quien prefiera menos movimiento simplemente vea todo aparecer de golpe.

## Funcionalidad e interacciones

- **Saludo según la hora**, sin ningún nombre — genérico, para que sirva para cualquiera que use la app:
  ```js
  function greeting() {
    const h = new Date().getHours();
    if (h < 12) return 'Good morning';
    if (h < 19) return 'Good afternoon';
    return 'Good evening';
  }
  ```
- **Frase motivadora rotativa**: una lista fija de 8-10 frases cortas y genéricas (escribe las tuyas, o usa algo como "Small steps compound.", "Show up. The rest follows.", "Progress hides in the boring days."), elegida al azar una vez por carga de página, y **vuelta a elegir cada vez que se vuelve a esta pantalla** (al pulsar el logo desde dentro de la app, ver más abajo) — no solo en la primera carga.
- **Intensidad del glow del arco según la hora del día**: cambia solo la opacidad de `.landing-bg-arc` (nunca su color), más intensa a mediodía/tarde, más tenue por la mañana temprano y por la noche.
- **El botón "Go to Dashboard" conecta esta pantalla con el resto de la app ya construida**: al pulsarlo, oculta `#landingView` (`classList.add('hidden')`) y muestra `#appRoot` (`classList.remove('hidden')`) — sin ninguna navegación de verdad, sin entrada en el historial del navegador, y sin lógica de "ya la vi antes" — se vuelve a mostrar esta pantalla en cada carga fresca de la página.
- **El logo del sidebar (`#appLogoBtn`, ya construido en `prompt-01-dashboard.md`) hace el camino inverso**: al pulsarlo desde cualquier sección de la app, oculta `#appRoot` y vuelve a mostrar `#landingView` — y en ese momento, vuelve a elegir un saludo/frase nuevos (ver punto anterior), ya que se está "entrando" de nuevo a esta pantalla. No resetea qué sección del sidebar estaba activa ni si el sidebar estaba colapsado — al volver a pulsar "Go to Dashboard", debe aparecer exactamente donde se dejó.

## Notas de implementación

- Todo en el mismo `index.html`, sin peticiones de red ni imágenes externas — el logo, el arco y las partículas son CSS/SVG/canvas puros.
- No hardcodees ningún nombre de persona en el saludo — esta pantalla debe funcionar igual de bien para cualquiera que use la app.
- No añadas ningún contenido debajo del botón "Go to Dashboard" en este prompt — ni tarjetas de resumen, ni estadísticas rápidas, ni rachas. Esta pantalla termina en el botón.
