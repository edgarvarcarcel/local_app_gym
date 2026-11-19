# 🏋️‍♂️ GymVault — Local-First Strength Telemetry & Analytics App

[![Architecture](https://img.shields.io/badge/Architecture-Local--First%20%7C%20Zero--Backend-10b981?style=flat-square)](#arquitectura-t%C3%A9cnica)
[![Stack](https://img.shields.io/badge/Stack-HTML5%20%7C%20ES6%2B%20%7C%20Tailwind%20CSS-06b6d4?style=flat-square)](#arquitectura-t%C3%A9cnica)
[![Storage](https://img.shields.io/badge/Storage-IndexedDB%20%2B%20LocalStorage%20%7C%20Schema%20v2-3b82f6?style=flat-square)](#modelo-de-datos-persistencia-y-compatibilidad)
[![Data Export](https://img.shields.io/badge/Data%20Export-CSV%20%7C%20JSON%20%7C%20ZIP%20%7C%20TXT-f59e0b?style=flat-square)](#portabilidad-y-an%C3%A1lisis-de-datos)

**GymVault** es una aplicación web *single-file* y *local-first* diseñada para la captura estructurada de entrenamientos de fuerza, seguimiento longitudinal de progresión y exportación directa de datos listos para análisis en **Python, R, SQL, Excel o Power BI**, sin servidores, suscripciones ni dependencias de compilación.

---

## 🎯 Visión de Producto y Analítica

En proyectos de analítica aplicada, el principal cuello de botella rara vez es el modelo: suele ser la **calidad y fricción en la captura del dato**. GymVault resuelve ese problema con un enfoque pragmático de producto e ingeniería:

- **Captura sin fricción en tiempo real:** registro móvil ergonómico de peso, repeticiones, `RIR` (*Repetitions in Reserve*), descansos resilientes a bloqueo de pantalla, reemplazo de ejercicios en vivo, notas de sesión y observaciones técnicas por ejercicio.
- **Soberanía, portabilidad y evolución segura del dato:** el usuario es dueño absoluto de su historial. Un pipeline centralizado de **migración incremental de esquemas (`v1 → v2`)** garantiza compatibilidad retroactiva al importar respaldos antiguos, y con un clic se puede exportar en `.zip`, `.csv`, `.txt`, portapapeles o `.html` autónomo.
- **Analítica integrada en cliente:** cálculo automático de volumen por sesión, récords personales (PRs) y estimación de fuerza máxima (**1RM** mediante la fórmula de Epley) visualizados en gráficos interactivos sobre HTML5 Canvas.

---

## ✨ Funcionalidades Principales

| Módulo | Capacidades |
| :--- | :--- |
| **Entrenar (Live Logger)** | Registro en vivo serie por serie con referencia automática de marca anterior, **cronómetro de descanso resistente al bloqueo de pantalla** (basado en timestamps absolutos + `visibilitychange`), **reemplazo de ejercicios en sesión (`🔄`)** con opción al finalizar de **mantener la rutina original o actualizarla**, alertas automáticas si un entreno queda abierto, vibración háptica y autoguardado local con guía interactiva. |
| **Rutinas** | Creación, edición y reordenamiento de rutinas con **buscador predictivo en vivo (sin acentos y multipalabra)** para agregar ejercicios sin recorrer listados largos, series objetivo, rango de repeticiones, descansos y notas técnicas. |
| **Parámetros y Catálogo** | Configuración global (`kg`/`lb`, descanso base, sonido, **Pantalla Activa / Wake Lock**, **Notificaciones del navegador** y umbral de **Aviso de Entreno Abierto**), panel de capacidades del dispositivo y catálogo maestro con observaciones personales y detección de duplicados al escribir. |
| **Progreso y Analítica** | Historial indefinido de sesiones, desglose de volumen/series y gráficos de progresión por ejercicio (`1RM` estimado, peso máximo y volumen). |
| **Datos `.ZIP` y Portabilidad** | Exportación/importación `.zip` (modos *Reemplazar* o *Fusionar* con migración automática de versiones), `.csv` analítico, resumen `.txt`, **copia directa al portapapeles (WhatsApp)** y generación de `.html` autónomo con semilla embebida. |

---

## 🏗️ Arquitectura Técnica

Todo el sistema vive en un único archivo autocontenido ([gymvault_rutinas_y_registro_de_gimnasio.html](gymvault_rutinas_y_registro_de_gimnasio.html)), aprovechando APIs web estándar con mejora progresiva y cero backend:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                    GymVault (Single-File Web App)                       │
├─────────────────────────────────────────────────────────────────────────┤
│ UI Layer        │ Tailwind CSS (CDN) • 5 Tabs • HTML5 Canvas            │
│ Logic Engine    │ Vanilla JS (ES6+) • Epley 1RM • Accent-Free Search    │
│ Mobile Web APIs │ Page Visibility • Wake Lock • Notifications • Vibrate │
│ Persistence     │ IndexedDB + LocalStorage + Storage Persist + Seed     │
│ Compatibility   │ Centralized Schema Migration Pipeline (v1 → v2)       │
│ Portability     │ JSZip (.zip) • .csv • .txt • Clipboard • Portable HTML│
└─────────────────────────────────────────────────────────────────────────┘
```

### Modelo de Datos, Persistencia y Compatibilidad
- **Doble respaldo en navegador + Memoria Persistente:** cada cambio dispara un autoguardado (debounce de `1s`) en `IndexedDB` (`GymVaultDB`) y `localStorage` (`GYMVAULT_STATE_V1`), solicitando `navigator.storage.persist()` para proteger el historial frente a limpiezas automáticas del dispositivo.
- **Cronómetros exactos en segundo plano:** tanto el reloj de sesión como el temporizador de descanso calculan el tiempo desde timestamps absolutos (`startTime`, `restTimerEndAt`) persistidos localmente y resincronizados al desbloquear el celular (`visibilitychange` / `focus` / `pageshow`), evitando el congelamiento típico de `setInterval` en móviles.
- **Compatibilidad garantizada entre versiones (`migrateState()`):** todo respaldo importado (`.zip` / `.json`) o estado previo pasa por migraciones aditivas incrementales (`v1 → v2`), preservando intacto el histórico de entrenamientos.
- **Semilla embebida (`#gymvault-embedded-seed`):** al exportar la versión portable en HTML, el estado completo se serializa dentro del documento para continuar entrenando en otro dispositivo.

---

## 📊 Portabilidad y Análisis de Datos

El archivo `historial_entrenamientos.csv` incluido en las exportaciones está estructurado en formato tabular limpio para integrarse directamente en flujos de **Ciencia de Datos / BI**:

```python
import pandas as pd

df = pd.read_csv("historial_entrenamientos.csv")

# Ejemplo: evolución de 1RM estimado y volumen semanal por ejercicio
df["Fecha"] = pd.to_datetime(df["Fecha"])
resumen = (
    df.groupby(["Ejercicio", pd.Grouper(key="Fecha", freq="W")])
      .agg(volumen_semanal=("Volumen", "sum"), max_1rm=("1RM_Estimado", "max"))
      .reset_index()
)
print(resumen.head())
```

---

## 🚀 Cómo Usarlo

1. **Uso 100% local (sin servidor):** descarga o clona este repositorio y abre `gymvault_rutinas_y_registro_de_gimnasio.html` directamente en Chrome, Safari, Edge o Firefox (PC o celular).
2. **Despliegue estático gratuito (recomendado para móvil):** habilitando **GitHub Pages** sobre este mismo repositorio (`https://`), el archivo único desbloquea además **Screen Wake Lock** (pantalla activa) y **Notificaciones nativas del sistema**, manteniendo el almacenamiento 100% local en el dispositivo del usuario.
3. Configura tus rutinas y observaciones, registra tus sesiones y exporta tus datos desde la pestaña **Datos .ZIP** cuando quieras respaldar, compartir por WhatsApp o analizar tu historial.

---

## 👨‍💻 Autor

**Edgar David Varcarcel Trujillo**  
*Científico de Datos*

- GitHub: [@edgarvarcarcel](https://github.com/edgarvarcarcel)
- Enfoque: Analítica Avanzada, Ciencia de Datos Aplicada, Ingeniería de Datos y Soluciones Analíticas de Alto Impacto.
