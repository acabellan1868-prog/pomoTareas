# Proyecto: Pomodoro & Tareas — Resumen

---

## Cambios — 05/03/2026 (v9)

### Integración con Supabase — Sincronización en la nube

**Persistencia en la nube**
- El estado completo de la aplicación (tareas, sesiones, estadísticas, nota, tarjetas Kanban) se sincroniza automáticamente con Supabase.
- Se mantiene `localStorage` como caché local: la app sigue funcionando aunque no haya conexión a internet.
- Al abrir la app en cualquier dispositivo, carga primero los datos locales (instantáneo) y después consulta Supabase para aplicar los datos más recientes.

**Sincronización con debounce**
- Cada llamada a `save()` persiste inmediatamente en `localStorage` y programa una sincronización remota con debounce de 4 segundos.
- Esto evita llamadas excesivas a la API en cambios rápidos sucesivos (escribir en la nota, marcar tareas, etc.).

**Indicador visual de sincronización**
- Nuevo elemento en la cabecera junto al toggle de tema que muestra el estado de la sincronización:
  - `☁️ Sincronizado` — estado neutro
  - `🔄 Guardando…` — sincronización en curso (con animación de rotación)
  - `☁️ Datos cargados` — datos remotos aplicados correctamente (verde, 3 segundos)
  - `⚠️ Error sync` — error de comunicación con Supabase (rojo)
  - `📴 Sin conexión` — sin acceso a internet

**Export/Import JSON mantenidos**
- Las funciones de exportación e importación manual de JSON se mantienen sin cambios, como copia de seguridad independiente de Supabase.

**Configuración técnica**
- CDN de Supabase JS v2 añadido en el `<head>`.
- Cliente inicializado con URL pública y clave `publishable` (segura para el navegador).
- Tabla utilizada: `estado_app` con columnas `id` (text, PK) y `datos` (jsonb).
- Fila única con `id = 'principal'` para uso personal.

---

## Cambios — 05/03/2026 (v8)

### Correcciones y mejoras de comportamiento

**Inicio automático del reloj al asignar tarea por arrastre (drag-to-assign)**
- Al soltar una tarea pendiente sobre la zona «Tarea activa para esta sesión», el temporizador se inicia automáticamente si no estaba en marcha.
- El comportamiento es idéntico al del desplegable: cambiar la tarea activa → el reloj arranca solo.
- Se muestra un toast secundario «▶ Reloj iniciado» confirmando la acción.

**Actualización en tiempo real de tareas tachadas en el Kanban**
- Al marcar o desmarcar una tarea como completada desde el gestor de tareas, el tablero Kanban se re-renderiza inmediatamente.
- Ya no es necesario refrescar la página ni mover la tarjeta para que el estado de la tarea hija se refleje en la tarjeta Kanban.
- Se añadió `renderKanban()` al final de la función `toggleTask`.

**Columnas Kanban plegables verticalmente**
- Cada columna del tablero Kanban tiene ahora un botón ▼ en su cabecera para colapsar o expandir su contenido.
- Al plegar una columna, se ocultan sus tarjetas y el botón «+ Nueva tarjeta» con una animación suave (`max-height` + `opacity`).
- La columna colapsada se estrecha a 120 px para ceder espacio a las columnas expandidas y evitar que la última columna quede cortada.
- El contador de tarjetas se muestra siempre, incluso en estado colapsado (con el elemento `kanban-col-count-colapsada`).
- El estado de colapso de cada columna se persiste en `localStorage` (clave `pomo_kanban_columnas_colapsadas`) y se restaura al recargar la página.

**Corrección de la última columna cortada (complementaria)**
- El estrechamiento de columnas colapsadas (flex: 0 0 120px) resuelve definitivamente el problema de la última columna cortada cuando hay tarjetas en varias fases simultáneamente, ya que libera espacio horizontal para las columnas activas.

---

## Cambios — 05/03/2026

### Mejoras en el tablero Kanban

**Tarjetas en «Hecho» excluidas del selector de proyecto**
- Las tarjetas Kanban cuya columna es «Hecho» ya no aparecen en el selector de proyecto al crear una nueva tarea.
- Tampoco aparecen en el selector del formulario de edición inline, salvo que la tarea ya estuviera previamente vinculada a esa tarjeta (en cuyo caso se muestra con el sufijo ✅ para indicar que está finalizada).
- Esto evita vincular nuevas tareas a proyectos ya cerrados.

**Completado automático de tareas al mover a «Hecho»**
- Al arrastrar una tarjeta Kanban a la columna «Hecho», todas sus tareas hijas que estén pendientes se marcan automáticamente como completadas, registrando la fecha de completado.
- Se muestra un único toast informativo que indica la tarjeta movida y, si las hay, el número de tareas autoccompletadas: `Tarjeta → Hecho · N tarea(s) completada(s)`.

**Arrastre de tarjetas solo desde el título**
- El `draggable` se ha movido del `<div>` de la tarjeta completa al `<div>` del título (`kanban-tarjeta-titulo`).
- El cursor grab solo aparece al posicionarse sobre el título. El resto del contenido (meta, lista de tareas hijas, botones) mantiene el cursor por defecto y no interfiere con el arrastre.
- El título tiene un fondo sutil al pasar el ratón para indicar visualmente que es el handle de arrastre.

**Tooltip con título completo en tareas hijas**
- El elemento `<span>` del nombre de cada tarea hija dentro de la tarjeta Kanban ya incluía el atributo `title` con la descripción completa. Se ha verificado y mantenido correctamente para que el título completo sea accesible al pasar el cursor sobre el texto truncado.

**Corrección de la última columna cortada**
- El contenedor `.kanban-tablero` ahora tiene `padding-right: 16px` (antes `2px`), lo que garantiza que la última columna del tablero no quede visualmente cortada cuando hay tarjetas en tres o cuatro estados simultáneos.

---

## Cambios — 03/03/2026 (II)

### Burbuja Kanban siempre a ancho completo

**Enfoque simplificado via CSS puro**
- La burbuja Kanban ocupa **siempre** las dos columnas del layout, independientemente de en qué posición la haya colocado el usuario al reordenar.
- Se logra con dos cambios mínimos de CSS, sin lógica JavaScript adicional:
  - `.layout-col` pasa a `display: contents`, de modo que sus hijos participan directamente en el grid padre.
  - `#burbuja-kanban` tiene `grid-column: 1 / -1`, lo que le hace ocupar siempre las dos columnas del grid.
- No hay botones extra, no hay zonas especiales, no hay claves de localStorage adicionales.
- El drag & drop de burbujas sigue funcionando igual; el Kanban se puede mover arriba o abajo dentro de cualquier columna y siempre mantiene el ancho completo.

---

## Cambios — 03/03/2026

### Tablero Kanban integrado

**Nueva burbuja: Tablero Kanban**
- Panel arrastrable y colapsable como el resto de burbujas, ubicado por defecto en la columna derecha entre «Gestión de tareas» y «Nota».
- Cuatro columnas fijas con scroll horizontal: 💡 Idea·Análisis, 🔨 Desarrollo, 🔍 Revisión·Bloqueo, ✅ Hecho.
- Cada columna muestra un contador de tarjetas y un botón «+ Nueva tarjeta».
- Las tarjetas se arrastran entre columnas para cambiar su estado (drag & drop independiente del de burbujas y tareas).
- Al mover una tarjeta se muestra un toast confirmando el movimiento.

**Tarjetas Kanban**
- Campos: título, prioridad (alta/media/baja con badge de color), fecha límite opcional y columna.
- Modal centrado para crear y editar tarjetas. Se cierra con botón Cancelar, click en el overlay o tecla Escape.
- Botón de eliminación en la tarjeta: desvincula automáticamente todas las tareas hijas antes de borrar.
- Las tarjetas muestran el total de pomodoros acumulados sumando los de sus tareas hijas.

**Lista compacta de tareas hijas**
- Cada tarjeta Kanban muestra sus tareas vinculadas en formato compacto: check de completado, título truncado y contador de pomodoros.
- El check de completado de la tarea hija funciona directamente desde la tarjeta Kanban (llama a `toggleTask`).
- Botón «+ Tarea» en la tarjeta: expande la burbuja de tareas, preselecciona el proyecto y pone el foco en el campo de descripción.

**Vinculación tarea ↔ Kanban (las dos direcciones)**
- Al crear una tarea: selector opcional «📋 Sin proyecto Kanban» en el formulario. Si se elige un proyecto, la tarea queda vinculada.
- Al editar una tarea (inline): el formulario de edición incluye también el selector de proyecto Kanban.
- Desde la tarjeta Kanban: botón «+ Tarea» que preselecciona el proyecto en el formulario de nueva tarea.
- Las tareas sin vinculación siguen funcionando exactamente igual que antes.

**Badge Kanban en la lista de tareas**
- Las tareas vinculadas a un proyecto muestran un badge compacto (📋 nombre del proyecto) junto a la etiqueta y duración.

**Modelo de datos**
- Nuevo array `tarjetasKanban` en el estado (compatible con datos existentes, no rompe nada).
- Nuevo campo `idKanban` en las tareas: `null` si son independientes, o el ID de la tarjeta padre.
- Persiste en `localStorage` junto al resto del estado (clave `pomo_state`).
- El layout por defecto actualizado incluye la burbuja `kanban` en la columna derecha.

---

## Cambios — 27/02/2026

### Nuevas funcionalidades implementadas en esta sesión

**Arrastrar tarea al temporizador (drag-to-assign)**
- Cada tarea pendiente muestra un handle de arrastre (⠿) naranja en su zona de acciones.
- La zona «Tarea activa para esta sesión» actúa como drop target: se ilumina con borde naranja discontinuo al arrastrar encima.
- Al soltar, la tarea queda seleccionada en el desplegable y se resalta en naranja en la lista.
- Las tareas completadas no permiten arrastre.
- Se muestra un toast de confirmación al asignar.
- El arrastre de tareas y el reordenamiento de burbujas son independientes: se distinguen por el tipo de dato en `dataTransfer` (`pomo-tarea-id` vs `text/plain`).

**Colapso de burbujas**
- Todas las burbujas (Temporizador, Estadísticas, Gestión de tareas, Nota y Datos) tienen un botón ▼ en su cabecera para colapsar/expandir el contenido.
- La animación usa `max-height` y `opacity` con transición CSS suave (0.3s).
- El estado de colapso de cada burbuja se persiste en `localStorage` (clave `pomo_burbujas_colapsadas`) y se restaura al recargar.

---

## ¿Qué es?

Aplicación web de gestión de tareas combinada con temporizador Pomodoro. Un único archivo HTML autocontenido (`pomodoro-tareas-v4.html`), sin dependencias externas ni servidor, que funciona directamente en el navegador.

---

## Funcionalidades implementadas (estado actual — v4)

### Gestión de tareas
- Crear tareas con descripción, etiqueta/categoría, fecha límite opcional y duración estimada.
- Fecha de creación automática.
- Edición inline de tareas: descripción, etiqueta, duración estimada y fecha límite.
- Marcar tareas como completadas (con fecha de completado registrada).
- Al completar la tarea activa en el temporizador: el selector se limpia y el reloj se pausa automáticamente.
- Eliminar tareas.
- Filtros por estado (todas / pendientes / completadas), por etiqueta y por duración estimada.
- Etiquetas disponibles: 💼 Trabajo, 🏠 Personal, 📚 Estudio, ⚙️ Proyecto, 📌 Otro.
- Duración estimada: ⚡ Rápida, 🕐 Media, 🗓 Larga — con badge de color diferenciado en cada tarea.
- Contador de pomodoros completados por tarea (🍅 ×N).
- Registro de IDs de intervalos asociados a cada tarea (visible en tooltip al pasar sobre el contador).
- Arrastrar una tarea pendiente sobre la zona «Tarea activa» del temporizador para asignarla directamente.

### Temporizador Pomodoro
- Display digital `MM:SS` con tipografía monospace.
- Tres fases: Trabajo / Descanso corto / Descanso largo.
- Configuración por defecto: **20 min trabajo / 5 min descanso / 15 min descanso largo cada 3 pomodoros**.
- Configuración editable desde la interfaz (sección colapsable dentro del temporizador); persiste en `localStorage`.
- Ciclo visual de 3 puntos indicando progreso del ciclo (puntos naranja: completados; pulsante: en curso).
- Barra de progreso del tiempo restante (color cambia según la fase: naranja = trabajo, verde = descanso corto, azul = descanso largo).
- Controles: Iniciar / Pausar / Continuar / Reiniciar / Saltar fase.
- Sonido al terminar cada fase (generado con Web Audio API, sin archivos externos).
- Notificaciones del navegador al terminar cada fase (solicita permiso al iniciar).
- Asociar cada sesión Pomodoro a una tarea de la lista mediante selector desplegable (sección colapsable) o arrastrando la tarea.
- Al cambiar la tarea activa en el selector, el reloj se reanuda automáticamente si estaba pausado por completar la tarea anterior.

### Identificadores de intervalo Pomodoro
- Cada pomodoro completado genera un ID único con formato `ddmmaaaa[N]` (ej: `27022026[3]` → tercer pomodoro del 27/02/2026).
- El ID se registra tanto en la sesión como en la lista de intervalos de la tarea asociada.
- Visible en el historial de sesiones y en el tooltip del contador de pomodoros de cada tarea.

### Estadísticas
- Pomodoros completados hoy / total acumulado (mostrado en cabecera y panel de estadísticas).
- Tareas completadas.
- Horas de foco acumuladas (calculadas a partir de la duración de las sesiones de trabajo).
- Historial de las últimas 20 sesiones: tarea, duración, ID de intervalo y fecha.

### Panel de Nota
- Área de texto libre con soporte **Markdown** completo.
- Dos modos: ✏️ Editar (textarea) y 👁 Preview (renderizado con `marked.js` v9.1.6).
- Guardado automático con debounce de 600 ms; indicador visual «✓ Guardado».
- Altura del textarea ajustable mediante slider (80px – 600px, paso 20px); altura persistida en `localStorage`.
- Fondo diferenciado del resto de paneles para distinción visual.

### Burbujas arrastrables y colapsables
- Los cinco paneles (Temporizador, Estadísticas, Gestión de tareas, Nota y Datos) son «burbujas» independientes.
- **Reordenamiento**: cada burbuja tiene un handle ⠿ para arrastrarla y recolocarla dentro de su columna o moverla a la otra columna.
- **Colapso**: botón ▼ en la cabecera para plegar/desplegar el contenido con animación suave. El estado se persiste en `localStorage`.
- El layout (orden y columna de cada burbuja) se persiste en `localStorage` (clave `pomo_layout_burbujas`).
- El estado de colapso se persiste en `localStorage` (clave `pomo_burbujas_colapsadas`).
- Dentro del temporizador, las subsecciones «Tarea asociada» y «Configuración» son también colapsables de forma independiente.

### Persistencia y datos
- Almacenamiento automático en **localStorage** del navegador (clave `pomo_state`).
- Exportar datos a JSON (backup manual): tareas, sesiones, estadísticas y nota.
- Importar datos desde JSON (restaurar backup).

### UI / UX
- **Tema oscuro/claro** con toggle en la cabecera; preferencia guardada en `localStorage`.
- Diseño responsive (escritorio / tablet / móvil) con layout de dos columnas en escritorio.
- Toast de notificación visual al completar fases, asignar tareas, importar datos, etc.
- Cabecera siempre visible con contador de pomodoros del día, total acumulado y tareas completadas.

---

## Stack técnico

| Elemento        | Detalle                                                         |
|-----------------|-----------------------------------------------------------------|
| Lenguaje        | HTML + CSS + JavaScript vanilla                                 |
| Fuentes         | Space Mono (monospace), Sora (sans-serif) — Google Fonts        |
| Markdown        | marked.js v9.1.6 (CDN: cdnjs.cloudflare.com)                   |
| Sonido          | Web Audio API (sin ficheros externos)                           |
| Notificaciones  | Notifications API del navegador                                 |
| Persistencia    | localStorage                                                    |
| Distribución    | Fichero único `.html` autocontenido                             |

---

## Claves de localStorage utilizadas

| Clave                       | Contenido                                              |
|-----------------------------|--------------------------------------------------------|
| `pomo_state`                | Estado completo: tareas, sesiones, estadísticas, nota  |
| `pomo_theme`                | Tema activo: `'light'` o `'dark'`                      |
| `pomo_layout_burbujas`      | Orden y columna de cada burbuja                        |
| `pomo_kanban_columnas_colapsadas` | Estado colapsado/expandido de cada columna Kanban     |
| `pomo_burbujas_colapsadas`  | Estado colapsado/expandido de cada burbuja             |
| `pomo_nota_altura`          | Altura en px del textarea de nota                      |

---

## Configuración Pomodoro por defecto

```
Trabajo:         20 minutos
Descanso corto:   5 minutos
Descanso largo:  15 minutos
Ciclo:           cada 3 pomodoros → descanso largo
```

---

## Pendiente / Próximos pasos

### Docker
Desplegar la app como contenedor para:
- Servirla siempre disponible en la red local (sin depender de abrir el fichero).
- Permitir incrustarla en **Dashy** como página dedicada con iframe.

Ficheros necesarios (pendiente de generar):
- `Dockerfile` — imagen `nginx:alpine` sirviendo el HTML.
- `docker-compose.yml` — puerto configurable, `restart: unless-stopped`.
- `nginx.conf` — con header `X-Frame-Options: SAMEORIGIN` para permitir iframe desde Dashy.

### Dashy
- Integrar como **sección/página dedicada** usando widget iframe.
- Configuración vía UI de Dashy (sin edición manual del `config.yml`).
- Acceso solo desde red local.

### Backend opcional (ideas futuras)
Si se quiere persistencia independiente del navegador:
- Mini backend Node.js/Express o Python/Flask con `GET /data` y `POST /data`.
- Datos guardados en fichero JSON en volumen Docker.
- Accesible desde cualquier dispositivo de la red (o vía Tailscale).

---

## Notas sobre localStorage

- Persiste aunque se cierre el navegador o se reinicie el PC.
- Vinculado a `protocolo + dominio + puerto` — cada navegador tiene su propio almacén separado.
- Se borra si el usuario limpia datos de navegación del navegador.
- No se sincroniza entre dispositivos.
- Límite ~5 MB (más que suficiente para esta app).
- **Mitigación**: exportar JSON periódicamente mediante el panel «Datos».

---

## Cambios — 06/03/2026

### Buscador de tareas

**Campo de búsqueda en tiempo real**
- Nuevo input «🔍 Buscar tareas…» situado entre el formulario de nueva tarea y los filtros de estado/etiqueta/duración.
- Filtra las tareas en tiempo real mientras se escribe, buscando coincidencias parciales (case-insensitive) en la descripción de la tarea.
- Compatible con los filtros existentes: la búsqueda se combina con el filtro de estado, etiqueta y duración activos.
- Al borrar el texto de búsqueda se muestran de nuevo todas las tareas según los filtros activos.

**Implementación técnica**
- CSS: nuevo estilo `.search-box` con icono de lupa posicionado absolutamente a la izquierda del input.
- HTML: `<input type="text" id="task-search">` dentro de la burbuja de tareas, antes del div `.filters`.
- JS: nueva variable global `activeSearchQuery`, nueva función `onSearchChange()` y condición `searchOk` añadida a `filteredTasks()`.

### Diseño responsive / móvil

**Media queries para pantallas < 768px**
- Layout principal: cambia de grid de 2 columnas (`380px 1fr`) a 1 columna (`1fr`).
- Burbuja Kanban: pasa de `grid-column: 1 / -1` a `grid-column: 1` (columna única).
- Header: se apila verticalmente (logo arriba, stats y controles debajo centrados).
- Stats: grid de 4 columnas reducido a 2 columnas.
- Columnas Kanban: estrechadas a 180px (colapsadas a 100px), manteniendo scroll horizontal.
- Timer: tamaño de reloj reducido a `3.5rem`.

**Media queries para pantallas < 480px**
- Formulario de tareas: campos apilados verticalmente (`flex-wrap: wrap`).
- Header stats: tamaño reducido a `0.65rem`.
- Timer: tamaño de reloj reducido a `3rem`, padding compacto.
- Modal Kanban: padding reducido y ancho completo.
