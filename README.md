# 🚚 Predicción de Atrasos en Rutas de Última Milla

Sistema de *machine learning* que predice, **mientras el conductor está en ruta**, si la próxima parada llegará tarde y cuántos minutos de atraso se esperan — para que la torre de control pueda intervenir **antes** de incumplir la ventana comprometida con el cliente.

Proyecto de Minería de Datos (DUOC UC) desarrollado con la metodología **CRISP-DM** de punta a punta, sobre un dataset real de **249.231 entregas**.

![Python](https://img.shields.io/badge/Python-3.11-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.8-orange)
![XGBoost](https://img.shields.io/badge/XGBoost-3.2-green)
![Metodología](https://img.shields.io/badge/CRISP--DM-completo-purple)

---

## 🎯 El problema

En logística de última milla, **1 de cada 10 entregas llega tarde**, y ese 10% concentra el costo: reintentos de distribución, llamadas de clientes fuera de ventana e incumplimiento de SLAs. El modelo operativo tradicional **reacciona** al atraso cuando ya ocurrió.

Este proyecto construye un motor de decisión en **tiempo real** que produce dos salidas accionables en cada escaneo de parada:

1. **Clasificación binaria** — ¿esta parada va a llegar tarde? (alerta de riesgo)
2. **Regresión** — ¿cuántos minutos de atraso se esperan? (severidad para priorizar)

---

## 📊 Resultados clave

### Clasificación (¿llega tarde?)

| Modelo | F1 (clase atrasada) | AUC-ROC | Accuracy |
| :--- | :---: | :---: | :---: |
| **Random Forest** ⭐ | **0.592** | 0.922 | 0.913 |
| HistGradientBoosting | 0.545 | 0.923 | 0.877 |
| XGBoost | 0.529 | **0.924** | 0.866 |
| Regresión Logística (baseline E2) | 0.335 | 0.780 | — |

> Partiendo de un F1 de **0.335** (baseline), la ingeniería de variables llevó el modelo a **0.592** — casi el doble. El cuello de botella no era el algoritmo: **eran los datos de entrada.**

### Regresión (¿cuántos minutos?)

| Modelo | R² | RMSE (min) | Predicción en atrasadas |
| :--- | :---: | :---: | :---: |
| OLS (baseline E2) | 0.605 | 96.3 | −90.3 (signo invertido ❌) |
| **Random Forest** ⭐ | **0.819** | **65.2** | +5.1 (cola correcta ✅) |
| HistGradientBoosting | 0.813 | 66.3 | +9.5 (mejor cola) |

---

## 🧠 Decisiones técnicas destacadas

Lo que separa este proyecto de un notebook cualquiera:

- **🛡️ Prevención de *data leakage*** — Todas las variables secuenciales (historial del conductor, tasa acumulada de la ruta) se construyen con `.shift(1)` + ventanas de expansión, para que el resultado de la entrega actual **nunca** contamine su propio predictor.
- **⏱️ Partición temporal, no aleatoria** — Split por `Week ID` (train < semana 22, test ≥ 22). Un split aleatorio habría mezclado pasado y futuro, inflando la evaluación de forma poco realista.
- **💰 Umbral como matriz de costos, no como F1** — El umbral de producción (**0.408**) se elige porque un falso negativo (atraso no detectado) cuesta más que un falso positivo. Sube el *recall* a **75%** sacrificando precisión de forma deliberada.
- **📐 GMM vs. KMeans para focalizar recursos** — GMM aísla el **14% de mayor riesgo (32.2% de atrasos)** frente al 39% difuso de KMeans. Traducción operativa: la torre de control concentra su capacidad donde realmente falla.
- **✅ Validación cruzada temporal** (`TimeSeriesSplit`, 5 folds) que confirma que los resultados no dependen de una partición afortunada.

---

## 🗂️ Metodología CRISP-DM

1. **Business Understanding** — KPIs de negocio, formulación del problema analítico.
2. **Data Understanding** — EDA sobre 249.231 filas, detección de errores de captura (atrasos de 12.8 millones de minutos 🤯).
3. **Data Preparation** — Limpieza en 2 iteraciones con criterio operacional (no IQR ciego), ingeniería de 7 variables de estado de ruta.
4. **Modeling** — 6 clasificadores + 6 regresores + clustering (KMeans, GMM, PCA).
5. **Evaluation** — F1, AUC, RMSE, R², análisis de la cola de predicción, CV temporal.
6. **Storytelling** — Arquitectura del pipeline de alertas en producción y plan de acción.

---

## 📁 Estructura del repositorio

```
.
├── Mineria_de_datos_E2.ipynb          # Notebook principal (análisis completo)
├── informe_final_mineria_completo.md  # Informe técnico final (Markdown)
├── Informe_Final_Mineria.pdf          # Informe final (PDF)
├── Informe_Final_Mineria.docx         # Informe final (Word)
├── Informe_E1_Mineria_de_Datos.md     # Informe de la Entrega 1
├── routes_performance.xlsx            # Dataset de entrada
├── img/                               # Figuras generadas (11 gráficos)
├── outputs/                           # Presentaciones (PPTX / PDF)
├── docs/                              # Briefs de la evaluación
├── requirements.txt
└── README.md
```

---

## ▶️ Cómo ejecutarlo

```bash
# 1. Clonar
git clone <url-del-repo>
cd "Minería de Datos"

# 2. Crear entorno e instalar dependencias
python -m venv .venv
# Windows:  .venv\Scripts\activate   |   Linux/Mac:  source .venv/bin/activate
pip install -r requirements.txt

# 3. Abrir el notebook
jupyter lab Mineria_de_datos_E2.ipynb
```

El notebook lee `routes_performance.xlsx` desde la raíz del proyecto.

---

## 📦 Dataset

*"Last-mile delivery route deviations dataset: planned vs. actual routes"* — 249.231 registros, 16 columnas (rutas planificadas vs. reales de entrega). El dataset es de un tercero y se rige por su licencia de origen.

---

## 👤 Autor

**Tomás Escalante** — Minería de Datos, DUOC UC.

## 📄 Licencia

Código y análisis bajo licencia [MIT](LICENSE). El dataset mantiene su licencia de origen.
