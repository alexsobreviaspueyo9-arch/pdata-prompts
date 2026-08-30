pdata es una app personal ("Personal Data") construida con Claude Code para centralizar y trackear tu vida en un solo sitio: entrenamiento, nutrición, sueño, gastos, lecturas y más — con una estética oscura y minimalista, y todo tuyo (tus datos, tu propia infraestructura, sin depender de servicios de terceros que se queden con tu información).

Este repo contiene los prompts exactos que usé para construir cada sección con Claude Code. La idea es que cualquiera pueda coger estos prompts, adaptarlos a su vida, y construirse su propia versión de pdata — gratis, sin necesidad de saber programar.

Qué incluye
Sección	Qué hace
🏠 Dashboard	Vista general con accesos rápidos, entreno del día, streaks y progreso semanal
🏋️ Gym	Rutinas por día de la semana, seguimiento de sets/reps/peso, sincronización con Polar Flow
🍳 Kitchen	Inventario de la despensa y planificación de comidas semanales
📅 Meal Prep	Timeline guiada de sesiones de meal prep con temporizadores por tarea
❤️ Health	Métricas generales de salud y bienestar
😴 Sleep	Calidad del sueño, fases y tendencias, con datos de Polar Flow
📊 Analysis	Análisis y cruces de datos entre las distintas secciones
📚 Books	Seguimiento de lecturas
💳 Expenses	Gastos registrados automáticamente desde Apple Pay vía un Atajo de iOS + Supabase
Cómo usarlo
Requisitos: tener Claude Code instalado, y una cuenta gratuita en Supabase si quieres usar las secciones que guardan datos (Expenses, y otras que lo necesiten).
Ve a la carpeta prompts/ y sigue el orden recomendado en su README: primero la base de la app (sidebar + landing), luego cada sección.
Copia el contenido de cada prompt y pégaselo a Claude Code, uno por uno. Cada prompt está pensado para funcionar de forma independiente y te indica si necesitas algo previo (ej. tu propia URL/key de Supabase).
Sustituye los datos de ejemplo (rutinas, plan de comidas, categorías de gasto...) por los tuyos — estos prompts están escritos de forma genérica a propósito, para que los adaptes a tu vida, no a la mía.
Por qué esto es gratis

Empecé este proyecto para mí, y quiero que le sirva a más gente. Si te resulta útil:

Dale una ⭐ al repo, ayuda a que más gente lo encuentre.
Comparte tu propia versión (screenshot, vídeo, lo que sea) — me encantaría verla.
Si te atascas montando algo, abre un Issue y lo miramos entre todos.
Vídeos

Estoy grabando una serie de vídeos explicando cómo construí cada sección paso a paso. [Enlace al canal / playlist — añadir aquí]

Licencia

Este proyecto está bajo licencia MIT — úsalo, modifícalo y compártelo libremente.

Aviso

Los prompts incluyen referencias de estilo visual (paletas de color, tipo de gráficos) inspiradas en varios diseños que encontré en Dribbble/Behance y otras apps, pero no incluyen ningún asset, logo ni contenido con derechos de terceros — son descripciones de estilo para que Claude Code las interprete, no copias de ningún diseño concreto.
