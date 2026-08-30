# Kitchen — Prompt para Claude Code

Requiere: el `index.html` base con sidebar (ver `prompt-01-dashboard.md` para el sistema de diseño). No necesita ninguna cuenta externa salvo, opcionalmente, Supabase para sincronizar.

## Objetivo

Un inventario de despensa: registra lo que compras, réstalo cuando lo consumes, define un plan de comidas semanal con botones de "consumir" por comida, y una lista de la compra que se genera sola a partir de lo que se está agotando.

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md`. Prefijo de clases sugerido: `.kx-*`.

## Estructura visual

Cinco sub-pestañas dentro de la sección, en este orden: **Comprar**, **Consumir**, **Plan semanal**, **Despensa**, **Lista de la compra**.

- **Comprar**: filas dinámicas (una o varias a la vez, con botón "+ añadir otra") con campo de nombre de ingrediente (con autocompletado de ingredientes ya conocidos), cantidad, unidad (g/ml/unidad), y un botón de icono de código de barras (ver más abajo). Botón "Añadir a la despensa" al final.
- **Consumir**: misma estructura de fila, pero pensada para restar. Debajo de cada fila, una fila de **botones de porción rápida** que aparecen en cuanto escribes el nombre de un ingrediente conocido — pensados para no tener que pesar nada al dedo: para ingredientes que se cuentan por unidades (huevos, piezas de fruta) muestra botones "1 / 2 / 3"; para el resto, 2-3 porciones realistas en su propia unidad (p. ej. "1 cucharada (15g)", "ración (150g)") — si no tienes una porción concreta definida para ese ingrediente, calcula un par de porciones genéricas a partir de su umbral de "stock bajo" (ver estructura de datos) en vez de dejarlo sin nada. Al tocar un botón, rellena directamente los campos de cantidad y unidad de esa fila.
- **Plan semanal**: selector de día (Lun-Dom) y, para el día elegido, tres bloques (Desayuno/Comida/Cena) cada uno con su lista de ingredientes+cantidades y un botón "Consumir desayuno/comida/cena" que resta todos los ingredientes de ese bloque de golpe. Rellena esto con un **plan de ejemplo de 2 días** (no hace falta el resto de la semana) para que quien lo use vea cómo tiene que ser el formato y lo sustituya por el suyo:
  ```
  Ejemplo día 1 — desayuno: avena + leche + fruta; comida: proteína + arroz + verdura; cena: pescado + patata + verdura.
  Ejemplo día 2 — desayuno: huevos + tostada; comida: legumbre + verdura; cena: proteína + ensalada.
  ```
- **Despensa**: lista de todo lo que tienes, ordenada alfabéticamente, con la cantidad actual y su unidad; los ingredientes en o por debajo de su umbral de "stock bajo" se resaltan con un color de aviso y una etiqueta "bajo". Cada fila tiene un botón de eliminar.
- **Lista de la compra**: se calcula sola — todo ingrediente con `cantidad <= umbral de stock bajo` **y que no lo hayas "descartado" recientemente** (ver más abajo). Cada fila lleva: un botón circular de check ("ya lo he comprado" — restablece la cantidad a un nivel razonable, el doble del umbral, y lo quita de la lista), el nombre (con una marca recomendada opcional debajo, en texto pequeño y gris, un campo puramente decorativo que el usuario puede rellenar), la cantidad actual/umbral, y un botón de eliminar (para un ingrediente que ya no quieres seguir, distinto del check de "ya comprado").
- **Búsqueda por código de barras**: en las filas de Comprar/Consumir, un icono de código de barras abre un modal que pide un número de código. Al buscarlo, llama a la API pública y gratuita de **Open Food Facts** (`https://world.openfoodfacts.org/api/v2/product/{codigo}.json`, sin necesidad de clave de API) y, si encuentra el producto, rellena el nombre, intenta interpretar su cantidad ("400 g", "1 L") en el par cantidad+unidad de la fila, y cachea sus datos nutricionales por 100g. Muestra un resumen de marca/cantidad/nutrientes encontrados antes de cerrar el modal — no lo cierres solo automáticamente, deja que el usuario lo confirme.

## Funcionalidad e interacciones

- **Comprar** suma cantidad al stock existente (o crea el ingrediente si no existía, con un umbral de stock bajo por defecto = 20% de la cantidad comprada si no hay uno definido). Comprar también marca el ingrediente como "recién atendido" para que no siga apareciendo en la Lista de la compra aunque matemáticamente siga bajo (por ejemplo si compraste solo una reposición parcial) — esa marca se borra automáticamente la próxima vez que consumas algo de ese ingrediente, así nunca se queda escondido para siempre.
- **Consumir** resta cantidad, sin bajar nunca de 0, y si el ingrediente no existía en la despensa lo crea a cantidad 0 (para que aparezca inmediatamente en la Lista de la compra). Cada consumo real (no las compras) se añade también a un registro histórico con fecha, para que otras secciones puedan leer "qué comiste y cuándo" si lo necesitan.
- **Plan semanal → Consumir [comida]** simplemente llama al mismo consumo de cada ingrediente del bloque, uno por uno, y muestra un aviso si algún ingrediente no tenía suficiente stock (lo deja en 0 en vez de fallar).

## Estructura de datos

```
localStorage:
  kitchen_storage     → [{ name (siempre en minúsculas), quantity, unit: "g"|"ml"|"unidad",
                           lowThreshold, brand?, groceryDismissed? }]
  kitchen_log_v1      → [{ date: "YYYY-MM-DD", name, quantity, unit }]   // consumos reales
  kitchen_nutrition_v1 → { [nombreIngrediente]: { barcode, brand, kcal100g, protein100g,
                            carbs100g, sugars100g, fat100g, satFat100g, fiber100g, salt100g } }
```

Un plan semanal (`KX_RECIPES` en el código) es un objeto plano hardcodeado: `{ lunes: { desayuno: [{name, quantity, unit}], comida: [...], cena: [...] }, martes: {...}, ... }` — rellénalo tú mismo con tu propia dieta, la app no necesita ninguna tabla externa para esto.

## Sincronización entre dispositivos (opcional)

Aplica el patrón de Supabase de `prompt-01-dashboard.md`, `APP_KEY: 'kitchen'`, sincronizando `kitchen_storage`, `kitchen_log_v1` y `kitchen_nutrition_v1`.

## Notas de implementación

- Ninguna marca, cantidad ni ingrediente real de nadie debe quedar hardcodeada en el código publicado — usa solo el plan de ejemplo de 2 días de arriba.
- Open Food Facts pide identificar tu app con una cabecera `User-Agent` personalizada en sus propias normas, pero el navegador no permite establecer esa cabecera desde JS — es una limitación conocida de la plataforma, no un error tuyo; a la escala de uso personal de esta app no supone ningún problema (su límite público es de 15 peticiones/minuto por IP).
