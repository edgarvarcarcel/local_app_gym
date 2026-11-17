# 🏋️‍♂️ GymVault — Local-First Strength Telemetry & Analytics App

[![Architecture](https://img.shields.io/badge/Architecture-Local--First%20%7C%20Zero--Backend-10b981?style=flat-square)](#arquitectura-t%C3%A9cnica)
[![Stack](https://img.shields.io/badge/Stack-HTML5%20%7C%20ES6%2B%20%7C%20Tailwind%20CSS-06b6d4?style=flat-square)](#arquitectura-t%C3%A9cnica)
[![Storage](https://img.shields.io/badge/Storage-IndexedDB%20%2B%20LocalStorage-3b82f6?style=flat-square)](#modelo-de-datos-y-persistencia)
[![Data Export](https://img.shields.io/badge/Data%20Export-CSV%20%7C%20JSON%20%7C%20ZIP%20%7C%20TXT-f59e0b?style=flat-square)](#portabilidad-y-an%C3%A1lisis-de-datos)

**GymVault** es una aplicación web *single-file* y *local-first* diseñada para la captura estructurada de entrenamientos de fuerza, seguimiento longitudinal de progresión y exportación directa de datos listos para análisis en **Python, R, SQL, Excel o Power BI**, sin servidores, suscripciones ni dependencias de compilación.

---

## 🎯 Visión de Producto y Analítica

En proyectos de analítica aplicada, el principal cuello de botella rara vez es el modelo: suele ser la **calidad y fricción en la captura del dato**. GymVault resuelve ese problema con un enfoque pragmático:

- **Captura sin fricción en tiempo real:** registro móvil ergonómico de peso, repeticiones, `RIR` (*Repetitions in Reserve*), descansos, notas de sesión y observaciones técnicas por ejercicio.
- **Soberanía y portabilidad del dato:** el usuario es dueño absoluto de su historial. Con un clic puede empaquetar toda su base de datos en un `.zip`, generar un `.csv` tabular para análisis estadístico, un `.txt` legible o un `.html` portable con los datos embebidos.
- **Analítica integrada en cliente:** cálculo automático de volumen por sesión, récords personales (PRs) y estimación de fuerza máxima (**1RM** mediante la fórmula de Epley) visualizados en gráficos interactivos sobre HTML5 Canvas.

---

## ✨ Funcionalidades Principales

| Módulo | Capacidades |
| :--- | :--- |
| **Entrenar (Live Logger)** | Registro en vivo serie por serie, referencia automática de la marca anterior, temporizador de descanso con alertas de audio sintetizadas (Web Audio API), notas de sesión y **autoguardado local** con guía interactiva de persistencia. |
| **Rutinas** | Creación, edición y reordenamiento de rutinas personalizadas con series objetivo, rango de repeticiones, tiempos de descanso y notas técnicas. |
| **Parámetros y Catálogo** | Configuración global (`kg`/`lb`, descanso base, sonido) y catálogo maestro de ejercicios con **observaciones personales** editables en cualquier momento. |
| **Progreso y Analítica** | Historial indefinido de sesiones, desglose de volumen/series y gráficos de progresión por ejercicio (`1RM` estimado, peso máximo y volumen). |
| **Datos `.ZIP` y Portabilidad** | Exportación/importación `.zip` (con modo *Reemplazar* o *Fusionar*), exportación `.csv`, resumen `.txt` y generación de `.html` autónomo con semilla embebida. |

---

## 🏗️ Arquitectura Técnica

Todo el sistema vive en un único archivo autocontenido ([gymvault_rutinas_y_registro_de_gimnasio.html](gymvault_rutinas_y_registro_de_gimnasio.html)), lo que permite abrirlo localmente en cualquier navegador de escritorio o móvil y compartirlo como un artefacto portable:

```text
┌──────────────────────────────────────────────────────────────────┐
│                 GymVault (Single-File Web App)                   │
├──────────────────────────────────────────────────────────────────┤
│  UI Layer      │ Tailwind CSS (CDN) • 5 Tabs • HTML5 Canvas      │
│  Logic Engine  │ Vanilla JS (ES6+) • Epley 1RM • Web Audio API   │
│  Persistence   │ Dual Engine: IndexedDB + LocalStorage + Seed    │
│  Portability   │ JSZip (.zip) • .csv • .txt • Standalone .html   │
└──────────────────────────────────────────────────────────────────┘
```

### Modelo de Datos y Persistencia
- **Doble respaldo en navegador:** cada cambio dispara un autoguardado (debounce de `1s`) en `IndexedDB` (`GymVaultDB`) y `localStorage` (`GYMVAULT_STATE_V1`), además de permitir guardado manual explícito y finalización de sesión.
- **Semilla embebida (`#gymvault-embedded-seed`):** al exportar la versión portable en HTML, el estado completo (`settings`, `exercises`, `routines`, `history`) se serializa dentro del mismo documento para continuar entrenando en otro dispositivo.

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

1. Descarga o clona este repositorio.
2. Abre `gymvault_rutinas_y_registro_de_gimnasio.html` directamente en Chrome, Safari, Edge o Firefox (PC o celular).
3. Configura tus rutinas y observaciones por ejercicio, registra tus sesiones y exporta tus datos desde la pestaña **Datos .ZIP** cuando quieras respaldar o analizar tu historial.

---

## 👨‍💻 Autor

**Edgar David Varcarcel Trujillo**  
*Científico de Datos*

- GitHub: [@edgarvarcarcel](https://github.com/edgarvarcarcel)
- Enfoque: Analítica Avanzada, Ciencia de Datos Aplicada, Ingeniería de Datos y Soluciones Analíticas de Alto Impacto.
