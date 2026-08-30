# Sleep — Prompt para Claude Code

Requiere: el `index.html` base con sidebar (ver `prompt-01-dashboard.md`). Funciona igual de bien sin ninguna integración externa (con datos de muestra generados), pero para tener datos reales necesita una sección de gimnasio con integración de Polar Flow ya montada (ver `prompt-02-gym.md`, punto 5) — de ahí lee, sin escribir nunca, los entrenos/sueño/recuperación nocturna sincronizados.

## Objetivo

Una vista de sueño y recuperación al estilo de un reloj deportivo moderno: puntuación de sueño, fases de sueño, una tendencia de varias semanas, y una cuadrícula de señales de recuperación (HRV, frecuencia cardiaca en reposo, frecuencia respiratoria, regularidad de horarios, carga de entreno vs. sueño).

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md`. Prefijo de clases sugerido: `.sl-*`. Usa `--good`/`--warn`/`--bad` (verde/ámbar/rojo) para los indicadores de estado de esta sección en vez del acento naranja general — aquí el color es información real (bien/regular/mal), no decoración.

## Estructura visual

1. **Tarjeta de puntuación de sueño** — número grande de la puntuación de la noche más reciente (0-100), una insignia comparándola con la media de la última semana (arriba/abajo/igual), y un selector "Esta semana / Este mes" que cambia un gráfico de barras entre 7 barras diarias o 4 barras de media semanal. Al pasar el ratón por una barra, un tooltip flotante muestra fecha, horas dormidas y hora de despertar.
2. **Tarjeta de fases de sueño** — un gráfico de barras apiladas de las últimas 7 noches (de abajo a arriba: sueño profundo → REM → ligero → despierto), con colores sólidos claramente distintos entre sí (nada de variaciones de opacidad de un mismo color — con el blur de las tarjetas de fondo, se confunden), y 4 mini-tarjetas debajo con el tiempo medio en cada fase y el cambio % frente a la semana anterior. Ojo con una trampa fácil: para "tiempo despierto", menos es mejor, así que la lógica de "verde si sube / rojo si baja" tiene que invertirse solo para esa fase en concreto.
3. **Tarjeta de tendencia y recuperación** — un gráfico de línea de 8 semanas comparando la duración media de sueño con una puntuación de recuperación compuesta (mezcla de puntuación de sueño y HRV si tienes datos reales). Normaliza ambas series a su propio rango 0-1 antes de dibujarlas (son unidades distintas — horas vs. una puntuación 0-100), y añade una frase corta de interpretación automática según si la recuperación ha subido, bajado o se ha mantenido plana entre la primera y la segunda mitad de esas 8 semanas.
4. **Cuadrícula de "Señales de recuperación"** — 7 mini-tarjetas: estado de recuperación nocturna (una insignia de texto tipo "Buena/Regular/Mala"), HRV media de 7 noches, frecuencia cardiaca en reposo durante el sueño, continuidad del sueño (minutos despierto / número de interrupciones), carga de entreno vs. sueño (heurística: si el entreno ha subido mucho pero el sueño no ha acompañado, avisa), regularidad de horarios (basada en cuánto varían tus horas de acostarte/despertarte en las últimas 2 semanas — menos variación, puntuación más alta), y frecuencia respiratoria.

## Funcionalidad e interacciones

- **Datos reales si existen, datos de muestra si no.** Genera un set de datos de muestra determinista (usa un generador pseudoaleatorio con semilla fija, no `Math.random()` puro, para que no cambie en cada recarga) que cubra unas 8 semanas, y úsalo como base **solo si no hay datos reales sincronizados**. Si tienes la integración de Polar de `prompt-02-gym.md`, lee (sin escribir) sus claves de sueño/entrenos/recuperación nocturna y prefiérelas sobre los datos de muestra en cuanto existan.
- Cruza el sueño con la recuperación nocturna por fecha (ambos vienen de fuentes/llamadas distintas de la API, pero comparten el mismo campo de fecha) para poder rellenar HRV/frecuencia cardiaca/frecuencia respiratoria en la misma tarjeta que la puntuación de sueño de esa noche.
- **Cuidado con las comparaciones "vs. semana anterior" cuando hay pocos datos reales todavía**: si acabas de conectar la integración real, puede que aún no haya 14 noches de historial para calcular una comparación con la semana previa — en ese caso, no muestres una división por cero disfrazada de "NaN%"; simplemente omite la comparación y muestra solo el dato de esta semana.
- Vuelve a leer y repintar al hacer clic en el propio item de navegación de esta sección (por si se acaba de sincronizar algo nuevo en la sección de gimnasio).

## Estructura de datos

Esta sección **no guarda nada propio** — solo lee (nunca escribe) las claves de sueño/recuperación de la sección de gimnasio si existen (ver `prompt-02-gym.md`: `gym_polar_sleep_v1`, `gym_polar_log_v1`, `gym_polar_recharge_v1`), y si no existen, genera sus propios datos de muestra en memoria. No necesita ninguna clave de `localStorage` ni sincronización propia con Supabase.

## Notas de implementación

- Todos los gráficos son SVG dibujado a mano, sin librerías externas.
- Un hueco honesto que no tiene solución con estas fuentes: el **número exacto de veces que te despiertas** por noche no lo da ninguna API de wearable habitual (solo el tiempo total despierto sí es un dato real) — muestra ese conteo como "necesita más datos" de forma permanente en vez de fingir un número.
