# 🍅 Pomo/Tareas

Aplicación de productividad tipo Pomodoro con gestión de tareas, todo en un único archivo HTML autocontenido. Sin dependencias externas, sin servidor, sin instalación.

---

## 🚀 Abrir la aplicación

👉 [Abrir Pomodoro & Tareas](https://acabellan1868-prog.github.io/pomoTareas/pomodoro-tareas-v11.html)

## ✨ Características

### ⏱ Temporizador Pomodoro
- Ciclos configurables de **trabajo / descanso corto / descanso largo**
- Configuración por defecto: 20 min trabajo · 5 min descanso · 15 min descanso largo (cada 3 pomodoros)
- Barra de progreso visual + indicadores de ciclo (dots)
- Sonido al finalizar cada fase (vía Web Audio API)
- Notificaciones del sistema (requiere permiso)
- Botones: Iniciar / Pausar / Continuar / Resetear / Saltar fase

### 📋 Gestión de Tareas
- Añadir tareas con:
  - **Descripción** libre
  - **Etiqueta**: Trabajo 💼 · Personal 🏠 · Estudio 📚 · Proyecto ⚙️ · Otro 📌
  - **Fecha límite** (opcional)
  - **Duración estimada**: Rápida ⚡ · Media 🕐 · Larga 🗓
- Marcar como completada / pendiente
- Edición inline de cualquier campo
- Eliminación individual
- Contador de pomodoros por tarea con registro de IDs de intervalo (`ddmmaaaa[N]`)

### 🔗 Asignación de tarea al temporizador
- **Arrastrar** cualquier tarea pendiente y soltarla sobre el selector del temporizador
- Al completar una tarea activa, el reloj se pausa automáticamente
- Al asignar una nueva tarea, el reloj se reanuda automáticamente

### 🔍 Filtros
- Por estado: Todas · Pendientes · Completadas
- Por etiqueta: 💼 🏠 📚 ⚙️ 📌
- Por duración: ⚡ Rápida · 🕐 Media · 🗓 Larga

### 📊 Estadísticas
- Pomodoros completados hoy y en total
- Tareas completadas
- Horas de enfoque acumuladas
- Historial de las últimas 20 sesiones con ID de intervalo y duración

### 📝 Nota con soporte Markdown
- Área de texto libre con guardado automático (debounce 600ms)
- **Modo edición** y **modo previsualización** con renderizado Markdown completo
- Soporte de: encabezados, listas, código inline y bloques, blockquotes, tablas, negrita, cursiva, enlaces, separadores
- Altura ajustable mediante slider (80–600 px), persistida entre sesiones

### 🧩 Interfaz de paneles arrastrables
- Cada panel (Temporizador, Estadísticas, Tareas, Nota, Datos) es una **burbuja** que se puede:
  - **Arrastrar** entre columnas y reordenar (mediante el handle ⠿)
  - **Colapsar / expandir** individualmente
- Layout y estado de colapso persisten en `localStorage`

### 🌓 Tema claro / oscuro
- Toggle en la cabecera
- Preferencia guardada en `localStorage`

### 💾 Exportación e Importación
- Exporta todos los datos (tareas, sesiones, estadísticas, nota) como archivo **JSON**
- Importa un JSON previamente exportado para restaurar el estado completo

---

## 🚀 Uso

1. Descarga el archivo `pomodoro-tareas-v5.html`
2. Ábrelo en cualquier navegador moderno (Chrome, Firefox, Edge, Safari)
3. No requiere servidor ni conexión a internet (excepto la carga inicial de las fuentes Google Fonts y la librería `marked.js` desde CDN)

---

## 🗂 Estructura de datos (localStorage)

| Clave | Contenido |
|---|---|
| `pomo_state` | Estado completo: tareas, sesiones, estadísticas, nota, contadores de intervalo |
| `pomo_theme` | Tema activo: `'light'` o `'dark'` |
| `pomo_layout_burbujas` | Posición y orden de cada burbuja por columna |
| `pomo_burbujas_colapsadas` | Estado de colapso de cada burbuja |
| `pomo_nota_altura` | Altura del textarea de nota en píxeles |

---

## ⚙️ Configuración del temporizador

Los intervalos son editables directamente en la sección colapsable **"Configuración"** del panel del temporizador:

| Parámetro | Por defecto | Rango |
|---|---|---|
| Trabajo | 20 min | 1–120 min |
| Descanso corto | 5 min | 1–60 min |
| Descanso largo | 15 min | 1–60 min |

El descanso largo se activa cada **3 pomodoros** completados.

---

## 🔖 Identificadores de intervalo

Cada pomodoro completado genera un ID único con el formato `ddmmaaaa[N]`, donde `N` es el número secuencial del día. Ejemplo: `03032026[2]` = segundo pomodoro del 3 de marzo de 2026. Este ID queda registrado en la sesión y en la tarea asociada, visible al pasar el cursor sobre el contador 🍅 de la tarea.

---

## 🛠 Tecnologías

- HTML5 + CSS3 + JavaScript vanilla (sin frameworks)
- [marked.js](https://marked.js.org/) v9.1.6 — renderizado Markdown
- [Space Mono](https://fonts.google.com/specimen/Space+Mono) + [Sora](https://fonts.google.com/specimen/Sora) — tipografías
- Web Audio API — sonidos de notificación
- Notifications API — alertas del sistema
- localStorage — persistencia de datos

---

## 📁 Archivos

```
pomodoro-tareas-v5.html   ← Aplicación completa (único archivo)
README.md                 ← Este documento
```

---

## 📄 Licencia

Uso personal y libre. Sin restricciones.
