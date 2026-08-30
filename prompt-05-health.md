# Health — Prompt para Claude Code

Requiere: el `index.html` base con sidebar (ver `prompt-01-dashboard.md`). Opcional: si ya tienes montada una sección de tipo "despensa/cocina" con un registro de qué comes y cuándo (como en `prompt-03-kitchen.md`), esta sección puede leer esos datos para estimar ingesta de nutrientes — si no la tienes, construye igualmente esta sección y deja esa parte concreta como un cálculo que simplemente no tendrá datos hasta que exista esa otra sección.

## Objetivo

Un registro de analíticas de sangre a lo largo del tiempo, con las hormonas/vitaminas más importantes destacadas arriba con su tendencia, un checklist diario de suplementos, y (si tienes una sección de cocina/despensa) una estimación de si tu dieta cubre ciertos nutrientes clave y un desglose de macronutrientes de lo que has comido.

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md`. Prefijo de clases sugerido: `.lb-*`.

## Estructura visual (de arriba abajo)

1. **Suplementos diarios** — una tarjeta con una lista fija de suplementos (defínela tú mismo — nombre + dosis, p. ej. "Vitamina D3+K2, 1 cápsula"), cada uno con una píldora "Marcar tomado" que alterna el estado de hoy, y un contador de cuántos días de los últimos 7 lo has tomado. Esta lista es fija/hardcodeada, no editable desde la interfaz.
2. **Tendencias de macronutrientes** (solo si tienes datos de una sección de cocina) — un gráfico de barras apiladas (proteína/carbohidratos/grasa en gramos, un color sólido distinto por macro, nunca transparencias que se mezclen entre sí) que alterna entre ventana de 7 y 30 días, con 4 mini-tarjetas de resumen (calorías/proteína/carbohidratos/grasa medias al día) y una línea honesta de cobertura (p. ej. "7 de 21 registros (33%) tenían datos nutricionales") — dilo siempre así de explícito, para que el gráfico nunca dé la sensación de ser más completo de lo que realmente es. Solo cuentan para el cálculo los registros de consumo que tengan datos nutricionales asociados (por ejemplo, procedentes de una búsqueda por código de barras en la sección de cocina) y que estén medidos por peso/volumen, no por "unidades".
3. **Cuadrícula de analíticas destacadas** — una tarjeta pequeña por cada hormona/vitamina que te importe seguir (defínelas tú: p. ej. TSH, Vitamina D, Testosterona, Cortisol, Hierro, Ferritina...), cada una con: el último valor registrado, un punto de color (verde si está dentro de rango, rojo si no) y un mini-gráfico de línea con la tendencia de todos los registros históricos de esa prueba. Si una prueba de la lista nunca se ha registrado, muestra igualmente su tarjeta vacía (para que quede claro qué se está siguiendo, en vez de que desaparezca).
   - **Medidores de "cobertura dietética"** (opcional, solo si tienes datos de cocina): en las tarjetas de hierro y de función tiroidea concretamente, un pequeño medidor circular que estima el % de la ingesta diaria recomendada de un nutriente relevante (hierro, yodo, selenio, zinc) a partir de los últimos 7 días de consumo registrado, cruzado contra una tabla propia de nutrientes aproximados por ingrediente que definas tú mismo (no hace falta que sea exhaustiva, unos pocos ingredientes habituales bastan). **No inventes un medidor así para ninguna otra hormona** (testosterona, cortisol, etc.) — no hay una fórmula dietética honesta para esas, así que esas tarjetas solo muestran el valor y su tendencia, sin ningún cálculo añadido.
4. **Botón "+ Añadir analítica"** — abre un modal para registrar una analítica completa con fecha, nombre del laboratorio, y una lista libre de resultados (categoría, nombre de la prueba, valor, unidad, rango de referencia bajo/alto), con filas que se pueden añadir/quitar dinámicamente y autocompletado de categorías/nombres de pruebas ya usados antes.
5. **Historial completo** — debajo de todo, cada analítica registrada se sigue mostrando entera, agrupada por categoría, no solo las destacadas de arriba.

## Funcionalidad e interacciones

- El registro de analíticas es de **esquema libre**: no hay una lista fija de "campos válidos", cualquier nombre de prueba se puede registrar, para que sirva igual de bien con un informe de laboratorio distinto al tuyo.
- Los medidores de cobertura dietética y el gráfico de macros deben volver a calcularse cada vez que se visita esta sección (no solo al cargar la página entera), porque los datos de consumo se generan en otra sección distinta — si tu app tiene varias pestañas que no comparten estado en memoria entre sí, vuelve a leer `localStorage` y a repintar al hacer clic en el propio item de navegación de esta sección.

## Estructura de datos

```
localStorage:
  health_labs_v1        → [{ id, date, labName, results: [{ category, name, value, unit,
                              refLow, refHigh }] }]
  health_supplements_v1 → [{ id: nombreSuplemento, date: "YYYY-MM-DD" }]   // una fila por toma
```

Si conectas con una sección de cocina, lee (sin escribir nunca) sus claves de consumo/nutrición — documenta claramente en tu código qué claves espera encontrar para que quede claro que es una integración de solo lectura entre dos secciones independientes.

## Sincronización entre dispositivos (opcional)

Aplica el patrón de Supabase de `prompt-01-dashboard.md`, `APP_KEY: 'health'`, sincronizando `health_labs_v1` y `health_supplements_v1`.

## Notas de implementación

- No hardcodees ninguna analítica, valor ni suplemento real de nadie en el código publicado — deja la lista de suplementos con 2-3 entradas de ejemplo genéricas, y sin ninguna analítica precargada (o, como mucho, una entrada de ejemplo claramente ficticia).
- Todos los gráficos son SVG dibujado a mano, sin librerías externas.
