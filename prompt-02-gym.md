# Gym — Prompt para Claude Code

Requiere: el `index.html` base con sidebar (ver `prompt-01-dashboard.md` para el sistema de diseño completo si empiezas de cero). Opcional pero recomendado: una cuenta de desarrollador de Polar (gratuita) si quieres sincronizar entrenos reales desde un reloj Polar, y una cuenta de Netlify (gratuita) si vas a desplegar ahí, porque esa parte necesita un pequeño backend.

## Objetivo

Una sección de gimnasio con: seguimiento de peso corporal con gráfico y estimación de composición, un mapa de recuperación muscular, tus rutinas de la semana (una por día) con una vista de detalle y una sesión de entreno guiada con temporizador de descanso, y opcionalmente sincronización real con un reloj Polar (entrenos, sueño, HRV) más un "coach" que analiza tus entrenos aeróbicos.

## Sistema de diseño base

Usa exactamente el mismo sistema que en `prompt-01-dashboard.md` (tema oscuro, tarjetas "glass" con blur, acento naranja en degradado con glow, botones píldora, sin librerías de gráficos externas). Prefijo de clases sugerido para esta sección: `.gt-*`.

## 1. Peso corporal

- Tarjeta con: número grande del último peso registrado + unidad, un input para registrar el peso de hoy (una vez registrado, se bloquea y muestra el valor con un botón para "editar"), un gráfico de línea suave (curvas de Bézier cuadráticas) de los últimos ~30 registros con relleno degradado bajo la línea, un indicador de diferencia vs. el registro anterior (flecha arriba/abajo + color semántico), una racha de días consecutivos registrando peso, y una estimación simple de composición corporal (cuánto del cambio de peso reciente es probablemente músculo vs. grasa, basado en si la fuerza registrada en las sesiones ha subido o bajado en el mismo periodo — dilo claramente como una **estimación aproximada, no un dato clínico**).
- **Toggle kg/lb** visible arriba de la tarjeta (un par de botones tipo píldora). Todo se almacena internamente siempre en kg; el toggle solo convierte en la capa de visualización/entrada (multiplica por 2.20462 para mostrar en lb, divide para volver a guardar en kg) — así nunca se acumulan errores de conversión ni se mezclan unidades en los datos guardados. Recuerda la preferencia en `localStorage`.

## 2. Recuperación muscular

- Un diagrame de silueta corporal (frontal/trasera, con un botón para alternar entre las dos vistas) con 5-6 zonas musculares coloreadas (p. ej. hombros, pecho, espalda, brazos, core, piernas). Para las formas del cuerpo, usa un contorno SVG simplificado que dibujes tú mismo, o busca un set de paths anatómicos de licencia abierta (MIT) — hay varios paquetes open-source de "muscle map" en GitHub pensados para apps de fitness que puedes adaptar como geometría, sin copiar código, solo los paths.
- Cada zona se colorea según un **cálculo de carga por decaimiento de 72 horas**, deliberadamente simple, no científico: cada set registrado en una sesión añade "carga" a las zonas musculares que ese ejercicio tiene etiquetadas, y esa carga decae linealmente a cero a lo largo de 72 horas (`decay = max(0, 1 - horasTranscurridas/72)`). El "% de recuperación" de una zona es `100 - cargaAcumulada × 16` (constante ajustada a ojo para que se note con pocos sets), acotado entre 0 y 100. Colorea verde/lima ≥70%, ámbar 40-69%, rojo <40%.
- Si tienes datos reales de recuperación nocturna (ver sección de Polar más abajo), combínalos como un modificador: la puntuación de recuperación de la última noche (una escala 1-6 tipo "muy mala" a "muy buena") se convierte en un multiplicador acotado (p. ej. 0.75× en la peor noche, 1.1× en la mejor, 1.0× neutro) que se aplica sobre el % de cada zona — así una mala noche baja un poco todos los números y una noche excelente los sube un poco, pero nunca sustituye al cálculo por zona (ningún wearable de consumo da recuperación por grupo muscular, solo un número global del cuerpo).
- Recalcula esto en cada renderizado (no hace falta ningún botón de "activar" — siempre se muestra ya calculado).

## 3. Mis rutinas

- 7 tarjetas, una por día de la semana, ancladas al día real (no un ciclo rotativo) — usa `new Date().getDay()` para resaltar la de hoy (a color) y aplicar `filter: grayscale(1)` al resto. Cada tarjeta muestra una foto de cabecera (usa una foto de stock propia o un color de fondo liso si no tienes fotos) y el nombre corto de la rutina de ese día.
- **Estructura de ejemplo** (sustitúyela por tu propia rutina semanal — esto es solo para tener algo con lo que arrancar, no una prescripción):
  - Día 1: cardio suave + movilidad
  - Día 2: fuerza tren superior
  - Día 3: descanso o cardio ligero
  - Día 4: fuerza tren inferior
  - Día 5: cardio + core
  - Día 6: fuerza full-body
  - Día 7: descanso / movilidad
- Cada ejercicio de una rutina tiene: nombre, número de sets, rango de repeticiones objetivo, peso inicial y "step" de progresión (cuánto sube el peso cuando corresponde), las zonas musculares que trabaja (para el diagrama de recuperación de arriba), y un flag `checkOnly` para ejercicios sin peso/reps que solo se marcan como "hecho" (calentamientos, estiramientos, series de técnica, cardio continuo).
- Al tocar una tarjeta se abre una **vista de detalle** a pantalla completa (overlay que se desliza desde abajo): cabecera con la foto, duración/calorías estimadas (heurística simple: ~30min/250kcal por bloque `checkOnly`, ~2.5min/8kcal por set normal — dilo como estimación, no exacta), lista de ejercicios con un icono genérico según el tipo (carrera/natación/estiramiento/fuerza — 4 iconos de trazo simple bastan, no hace falta uno por ejercicio), y un botón grande "▶ Empezar" que abre la sesión activa. Un icono de lápiz abre un modal para añadir/editar/quitar ejercicios de ese día.

## 4. Sesión activa

- Overlay a pantalla completa con un cronómetro que cuenta desde que empezó la sesión, y una fila por ejercicio: para ejercicios normales, una fila editable de peso+reps por cada set con un botón de check; para ejercicios `checkOnly`, un único botón "Marcar hecho".
- Al marcar un set, se guarda en un registro de sesión y arranca un **temporizador de descanso visual** (p. ej. 120 segundos, sin sonido salvo que quieras añadirlo) que cambia de color/pulsa cerca del final.
- La sesión se guarda en `localStorage` tras cada cambio, así que si recargas la página o cambias de pestaña, se recupera exactamente donde la dejaste.
- **Coach de sobrecarga progresiva**: al empezar una sesión nueva, el peso inicial de cada ejercicio no es siempre el mismo fijo — se calcula mirando el registro de la última vez que se hizo ese ejercicio: si en todos los sets se llegó al objetivo de repeticiones, el peso sube un "step"; si no, se mantiene igual (nunca baja automáticamente). Cuidado con un fallo típico: si el "step" de un ejercicio es legítimamente `0` (por ejemplo un estiramiento sin peso), comprueba con `!= null` y no con `||`, porque `0 || 2.5` da `2.5` por error.
- Cerrar la sesión (con la X o con "Terminar entreno") no debe pedir confirmación si todo lo marcado ya está guardado en el momento en que se marca — no hay nada que "perder" al cerrar.

## 5. Integración con Polar Flow (opcional)

Esto es lo único de toda la app que necesita un pequeño backend, porque el intercambio de token OAuth requiere una clave secreta que nunca debe llegar al navegador.

- Registra una app gratuita en el panel de desarrolladores de Polar AccessLink (`admin.polaraccesslink.com`), con la URI de redirección apuntando a tu propio dominio desplegado.
- Crea 5 funciones serverless pequeñas (Netlify Functions u otro proveedor equivalente):
  - `polar-connect`: redirige al usuario a la pantalla de autorización de Polar.
  - `polar-callback`: recibe el código, lo intercambia por un token (usando tu client id/secret, guardados como variables de entorno, nunca en el código), registra al usuario en la API si hace falta, y guarda el token en un almacén del lado servidor (p. ej. Netlify Blobs) — **nunca en localStorage ni en tu base de datos compartida**, porque cualquiera con la clave pública de Supabase podría leerlo.
  - `polar-status`: dice si hay un token guardado (sin revelarlo).
  - `polar-sync`: usando el token guardado, llama a `GET /v3/exercises?zones=true` (entrenos, con desglose de tiempo en cada zona de frecuencia cardiaca), `GET /v3/users/sleep` (sueño), y `GET /v3/users/nightly-recharge` (HRV, frecuencia cardiaca en reposo, y una puntuación de recuperación nocturna 1-6) — devuelve los tres como JSON.
  - `polar-disconnect`: revoca el registro y borra el token guardado.
- Importante (confirmado contra la documentación real de Polar): solo se devuelven entrenos subidos a Flow **después** de que tu app quedara registrada con esa cuenta — no hay backfill de historial antiguo, y desconectar/reconectar reinicia esa fecha de corte.
- En el frontend: un botón "Conectar Polar" (enlace directo a `polar-connect`), y una vez conectado, un botón "Sincronizar ahora" que llama a `polar-sync` y guarda el resultado en `localStorage`. Muestra solo un resumen compacto ("Última sincronización hace X · N entrenos · M noches de sueño"), no una lista larga de cada entrada.

## 6. Coach de entrenos aeróbicos (opcional, necesita el punto 5)

- Analiza solo las sesiones de carrera/natación/ciclismo sincronizadas (clasifícalas por el campo de deporte que devuelve Polar — descarta fuerza/gimnasio, ese lado ya lo cubre el mapa de recuperación).
- Compara el tiempo real pasado en zonas altas de frecuencia cardiaca (zona 4-5 de 5) contra lo que ese día de tu rutina estaba pensado para ser (defínelo tú mismo por día: "técnica", "intenso", "moderado", "suave"). Si un día pensado como suave se pasó de vueltas en zonas altas, o un día de intervalos se quedó corto, muéstralo con un comentario breve tipo "podrías haber apretado más" o "te pasaste de esfuerzo para lo que tocaba" — dilo siempre como una lectura orientativa, no una ciencia exacta.

## Estructura de datos

```
localStorage:
  gym_units_v1            → "kg" | "lb"
  gym_routines_v1          → [{ id, name, day, sets, repMin, repMax, step, startWeight,
                                 checkOnly?, muscles: string[] }]
  gym_session_log_v1       → [{ date, day, exerciseId, exerciseName, muscles, checkOnly,
                                 weight, reps }]   // una fila por set marcado
  gym_active_session_v1    → { day, startedAt, sets: {...} } | null
  po_coach_weights         → [{ dateKey: "YYYY-MM-DD", weight (kg) }]
  gym_polar_log_v1         → [{ id, date, sport, sportType, duration, calories, avgHr, maxHr,
                                 zones: [{index, lower, upper, sec}] }]   // si usas Polar
  gym_polar_sleep_v1       → [{ date, sleepScore, totalSleepSec, deepSec, remSec, lightSec,
                                 awakeSec, startTime, endTime }]
  gym_polar_recharge_v1    → [{ date, nightlyRechargeStatus, ansCharge, hrvAvg,
                                 heartRateAvg, breathingRateAvg }]
```

## Sincronización entre dispositivos (opcional)

Aplica el mismo patrón de Supabase descrito en `prompt-01-dashboard.md`, con `APP_KEY: 'gym'` y sincronizando las claves de `localStorage` de arriba (excepto la del token de Polar, que nunca debe salir del backend).

## Notas de implementación

- Todo el gráfico/anillo/diagrama corporal es SVG dibujado a mano, sin librerías externas.
- No hardcodees ninguna rutina, peso ni foto personal en el código publicado — dale al usuario una estructura de ejemplo vacía o mínima y que la rellene él mismo la primera vez que abre la app.
