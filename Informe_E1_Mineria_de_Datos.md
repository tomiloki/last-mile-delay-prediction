# Predicción de ETA y Atraso en Rutas de Última Milla mediante Minería de Datos

**Asignatura:** Minería de Datos — 800D  
**Evaluación Parcial N°1 (35%)**  
**Docente:** Aldo Martinez  

**Autor:** Tomás Escalante  

**DuocUC — 2026**

---

## Índice

1. [Introducción](#introducción)  
2. [Comprender el Negocio](#comprender-el-negocio)  
   2.1 Contexto del negocio  
   2.2 Propósito y justificación  
3. [Comprender los Datos](#comprender-los-datos)  
   3.1 Exploración de la data  
   3.2 Análisis realizado  
   3.3 Hallazgos  
4. [Conclusión](#conclusión)  
5. [Bibliografía](#bibliografía)  

---

## 1. Introducción

El presente informe corresponde a la Evaluación Parcial N°1 del ramo Minería de Datos, e implementa las dos primeras fases de la metodología CRISP-DM sobre un conjunto de datos de logística de última milla.

La metodología CRISP-DM (Cross-Industry Standard Process for Data Mining) es un estándar ampliamente adoptado en la industria para guiar proyectos de minería de datos. Define seis fases iterativas: Business Understanding, Data Understanding, Data Preparation, Modeling, Evaluation y Deployment. Este informe aborda las dos primeras, que establecen el marco del problema y la base analítica sobre la que se construirán los modelos en etapas posteriores.

**¿Por qué estas fases son fundamentales?** La fase de Business Understanding asegura que el análisis esté alineado con los objetivos del negocio y que los KPIs definidos sean relevantes operacionalmente. La fase de Data Understanding permite comprender la calidad, estructura y limitaciones de los datos antes de modelar, evitando sesgos y errores en los resultados.

El proyecto se enfoca en la **predicción del tiempo estimado de llegada (ETA) y del atraso en entregas de última milla**, utilizando como base el dataset *"Last-mile delivery route deviations dataset: planned vs. actual routes"* (Mendeley Data, 2024). Este dataset captura rutas reales con la secuencia planificada y la secuencia real de paradas, junto con tiempos de llegada y ventanas horarias.

---

## 2. Comprender el Negocio

### 2.1 Contexto del negocio

La **logística de última milla** es la etapa final de la cadena de distribución, donde el producto se traslada desde un centro de distribución hasta el cliente final. Es considerada la fase más costosa e ineficiente del proceso logístico, representando hasta un 53% del costo total de envío según estudios del sector.

Las empresas de reparto planifican rutas con una secuencia de paradas que busca optimizar tiempos y distancias. Sin embargo, la ejecución real puede diferir del plan por múltiples razones operativas: tráfico, disponibilidad del destinatario, restricciones viales y decisiones del conductor, entre otras. Estas diferencias impactan directamente en:

- El cumplimiento de ventanas horarias comprometidas con los clientes.
- La eficiencia operativa y el costo por entrega.
- La satisfacción del cliente y la tasa de entregas fallidas.

El presente proyecto trabaja con datos reales de una empresa logística operando en dos países. Los datos incluyen la secuencia planificada y real de paradas, los tiempos de llegada a cada punto, las ventanas horarias pactadas y las distancias recorridas. Este contexto permite estudiar la brecha entre lo planificado y lo ejecutado, que es precisamente donde se origina el atraso en la entrega.

### 2.2 Propósito y justificación

#### Objetivo general

Identificar qué variables operativas de rutas de última milla — incluyendo desviaciones respecto al plan, distancias, ventanas horarias y contexto temporal — permiten predecir el atraso en una entrega.

#### Objetivos específicos

1. Comprender la estructura y naturaleza del dataset de rutas de última milla.
2. Identificar las variables más relevantes asociadas al atraso en las entregas.
3. Explorar la relación entre desviaciones de ruta y atraso, sin asumir causalidad directa.
4. Proponer un enfoque de modelado predictivo basado en los hallazgos del análisis exploratorio.

#### KPIs del negocio

Los siguientes indicadores de desempeño fueron definidos para cuantificar el problema operativo y medir el desempeño del negocio:

| KPI | Descripción | Valor observado |
|-----|-------------|-----------------|
| **Tasa de entregas a tiempo** | % de entregas con llegada dentro de la ventana horaria | **88.9%** |
| **Atraso promedio (mediana)** | Mediana de minutos de atraso entre entregas tardías | **~49 min** |
| **Desviación de ruta promedio** | Diferencia promedio entre orden planificado y orden real | **1.64 posiciones** |

> Nota: Se utilizan medianas para el atraso promedio, ya que la media se encuentra distorsionada por registros corruptos identificados en el análisis de outliers (media de 1.478 min vs. mediana de 49 min).

#### Formulación del problema analítico

La hipótesis de partida es que el atraso en una entrega puede ser anticipado a partir de variables operativas disponibles durante la ejecución de la ruta. **No se asume que las desviaciones de ruta sean la causa principal del atraso** — esa relación es parte de lo que el análisis debe revelar.

El problema se aborda principalmente como **regresión**: predecir los minutos de atraso (`delay`). Como análisis complementario, se formula también como **clasificación binaria**: determinar si una entrega estará atrasada (`delayed = 1`) o a tiempo (`delayed = 0`).

---

## 3. Comprender los Datos

### 3.1 Exploración de la data

#### Descripción del dataset

El dataset utilizado es *"Last-mile delivery route deviations dataset: planned vs. actual routes"*, publicado en Mendeley Data en diciembre de 2024. Contiene registros de rutas de última milla de una empresa logística operando en dos países, incluyendo la secuencia planificada de paradas, la secuencia real seguida por los conductores, tiempos de llegada, ventanas horarias y distancias entre paradas.

**Dimensiones del dataset tras el filtrado:**
- **Registros cargados originalmente:** 249.231 filas
- **Registros tras filtro de entregas (Delivery == 1):** 218.006 filas
- **Columnas:** 16 variables

El filtrado inicial excluyó las filas correspondientes al depósito (`Delivery == 0`, `Depot == 1`), ya que representan el punto de inicio o fin de la ruta y no constituyen entregas reales a clientes. Mantenerlas introduciría sesgo estructural: su distancia planificada y real es siempre 0, y el cálculo de atraso sería artificialmente negativo.

#### Diccionario de variables

| Variable | Tipo | Naturaleza | Descripción |
|----------|------|------------|-------------|
| `Route ID` | int64 | Identificador | Identificador único de la ruta |
| `Driver ID` | int64 | Identificador | Identificador único del conductor |
| `Stop ID` | int64 | Identificador | Identificador de la parada en la ruta |
| `Address ID` | int64 | Identificador | Identificador de la dirección de entrega |
| `Week ID` | int64 | Categórica ordinal | Semana en que se realizó la ruta |
| `Country` | int64 | Categórica nominal | País de operación (0 o 1) |
| `Day of Week` | object | Categórica nominal | Día de la semana de la ruta |
| `IndexP` | int64 | Numérica discreta | Posición planificada de la parada |
| `IndexA` | int64 | Numérica discreta | Posición real de la parada |
| `Arrived Time` | float64 | Numérica continua | Tiempo de llegada (min desde inicio de ruta) |
| `Earliest Time` | float64 | Numérica continua | Inicio de la ventana horaria (min) |
| `Latest Time` | float64 | Numérica continua | Fin de la ventana horaria (min) |
| `DistanceP` | float64 | Numérica continua | Distancia planificada desde parada anterior (km) |
| `DistanceA` | float64 | Numérica continua | Distancia real recorrida (km) |
| `Depot` | int64 | Binaria | 1 = depósito, 0 = no |
| `Delivery` | int64 | Binaria | 1 = entrega, 0 = no |

**Observaciones sobre los tipos de dato:**
- Las variables de ID son identificadores — no deben usarse como variables numéricas en el modelado.
- `Country` tiene solo 2 valores únicos (0 y 1) — es nominal, no ordinal.
- `Day of Week` es la única variable de tipo texto. Requiere codificación antes del modelado.
- Los tiempos están expresados en **minutos relativos al inicio de la ruta**, no en formato horario absoluto.
- `Delivery`, tras el filtrado, tiene un único valor (siempre = 1), por lo que no aporta información predictiva.

#### Variables derivadas

El dataset original no incluye directamente una medida de atraso ni de desviación de ruta. Se construyeron tres variables derivadas centrales para el análisis:

- **`delay`** = `Arrived Time` - `Latest Time`: minutos de atraso (positivo = tardío, negativo = a tiempo). Variable objetivo de regresión.
- **`delayed`** = 1 si `delay > 0`, 0 si no. Variable objetivo de clasificación.
- **`route_deviation`** = |`IndexP` - `IndexA`|: diferencia absoluta entre orden planificado y real. Variable predictora de desviación.

### 3.2 Análisis realizado

#### Estadísticos descriptivos

Se calcularon estadísticos sobre las variables numéricas relevantes, excluyendo identificadores. Dado que la media se encuentra distorsionada por registros corruptos, se priorizan las medianas como referencia central:

| Variable | Mediana | Media | Desv. Est. | Mín | Máx |
|----------|---------|-------|------------|-----|-----|
| `Arrived Time` | 404 min | 1.145 min | elevada | 0 | ~67.000 |
| `Latest Time` | 540 min | — | — | 0 | ~millones |
| `DistanceP` | ~1.2 km | — | — | 0 | 551 km |
| `DistanceA` | ~1.2 km | — | — | 0 | 551 km |
| `IndexP` / `IndexA` | 7 | — | — | 0 | 35 |
| `delay` | — | 1.478 min | elevada | negativo | ~12.800.000 |
| `route_deviation` | 1 pos. | 1.64 pos. | — | 0 | 27 |

Las medianas de `Arrived Time` (~404 min = ~6.7 horas) y `Latest Time` (~540 min = ~9 horas) reflejan rutas de media jornada laboral, con ventanas que cierran en la segunda mitad del día.

#### Valores faltantes (Missing Values)

Se verificó la existencia de valores nulos en el dataset tanto antes como después del filtrado:

- **Resultado:** el dataset no presenta valores nulos en ninguna columna. Esto es esperable en sistemas de registro logístico, donde cada parada registra obligatoriamente todos los campos para cerrar la operación.

No se requiere ninguna acción de imputación en esta fase.

#### Valores atípicos (Outliers)

Se aplicó el método IQR (rango intercuartílico) para detectar outliers en las variables `delay`, `DistanceA` y `route_deviation`. Un valor se considera atípico si cae fuera del rango `[Q1 - 1.5×IQR, Q3 + 1.5×IQR]`.

La exploración reveló tres tipos de problemas:

**Problema 1 — Registros con `Latest Time == 0` (22 casos)**

Existen 22 registros donde la ventana horaria cierra a los 0 minutos del inicio de la ruta. Esto no tiene sentido operativo y genera valores de `delay` artificialmente inflados. Son errores de registro.

**Problema 2 — Rutas con codificación de tiempo diferente**

Se detectaron rutas donde tanto `Arrived Time` como `Latest Time` presentan valores en el orden de los millones, pero de forma internamente consistente (el `delay` resultante es pequeño). Esto sugiere que algunas rutas usan una referencia de tiempo absoluta. Estos registros no son errores y se conservan.

**Problema 3 — Registros con `delay > 480 minutos` (1.239 casos, 0.57%)**

Se identificaron registros donde `Arrived Time` es enormemente grande pero `Latest Time` es normal, generando delays de decenas de miles de minutos. Son errores de captura en el sistema logístico, no rutas largas reales. La distribución confirma que el 75% de los atrasos reales está bajo 110 minutos y la mediana es 49 minutos.

Se fija un **umbral absoluto de 480 minutos (8 horas)** como límite razonable, dado que una ruta de reparto opera en un ciclo diario.

**Impacto de la limpieza propuesta:**

| Problema | Registros afectados | % del total | Acción |
|----------|---------------------|-------------|--------|
| `Latest Time == 0` | 22 | 0.01% | Eliminar |
| `delay > 480 min` | 1.239 | 0.57% | Eliminar |
| **Total a eliminar** | **~1.261** | **~0.58%** | |

La pérdida de datos es mínima y el dataset resultante (~216.745 registros) sigue siendo representativo para el modelado.

#### Matriz de correlación

Se calculó la matriz de correlación entre todas las variables numéricas del dataset. Los resultados más relevantes son:

**Correlaciones altas entre variables originales (esperables):**
- `Route ID` ↔ `Week ID` (1.00): prácticamente la misma variable. Una debería excluirse en modelado.
- `Arrived Time` ↔ `Earliest Time` ↔ `Latest Time` (~0.92): variables en la misma escala temporal.
- `DistanceP` ↔ `DistanceA` (0.88): distancia real y planificada son similares en la mayoría de los casos.
- `IndexP` ↔ `IndexA` (0.83): el orden planificado y real están altamente correlacionados.

**Correlaciones con las variables objetivo:**
- `delay` ↔ `Arrived Time` (0.40): la correlación más alta, con distorsión por outliers.
- `delayed` ↔ `route_deviation` (0.02): la desviación de ruta no muestra correlación lineal con el atraso.
- `delayed` ↔ `IndexA` (0.15): las paradas más avanzadas tienden a acumular más atrasos.
- `delayed` ↔ `Country` (0.06): leve diferencia entre países.

Ninguna variable presenta correlación lineal fuerte con `delay` o `delayed`, lo que indica que un modelo lineal simple tendría limitaciones y será necesario explorar modelos no lineales en la fase de Modeling.

### 3.3 Hallazgos

A partir del análisis realizado, se identificaron los siguientes hallazgos relevantes:

**Hallazgo 1 — La mayoría de las entregas llega a tiempo, pero el incumplimiento tiene alta magnitud en registros corruptos**

El 88.9% de las entregas se realiza dentro de la ventana horaria. Sin embargo, la mediana de atraso entre entregas tardías es de ~49 minutos, lo que refleja un incumplimiento real pero operacionalmente manejable. La media de 1.478 minutos está completamente distorsionada por errores de captura que deben eliminarse antes del modelado.

**Hallazgo 2 — La desviación de ruta no predice linealmente el atraso**

La correlación entre `route_deviation` y `delayed` es de apenas 0.02. Un conductor puede desviarse del orden planificado y aun así llegar a tiempo — o seguir el plan y atrasarse igualmente. Esto valida la decisión de no asumir causalidad directa entre desviación y atraso, planteada en el Business Understanding, y sugiere que la relación entre estas variables es compleja o mediada por otros factores.

**Hallazgo 3 — Las paradas más avanzadas en la secuencia acumulan más atrasos**

`IndexA` tiene una correlación de 0.15 con `delayed` — la más alta entre las variables originales del dataset. Las paradas hacia el final de la ruta tienen mayor probabilidad de haber recibido el impacto acumulado de retrasos anteriores, lo que es intuitivo desde el punto de vista operativo.

**Hallazgo 4 — Hay diferencias operativas significativas entre países**

País 0 presenta una tasa de atraso de 13.8% versus 9.7% del País 1. Esta diferencia sugiere que el país de operación podría ser una variable predictora relevante, y que los modelos podrían beneficiarse de ser entrenados por separado para cada contexto.

**Hallazgo 5 — El dataset contiene errores de registro que afectan las métricas calculadas**

Se identificaron 1.261 registros (~0.58%) con datos de tiempo corruptos que inflan artificialmente el atraso promedio y distorsionan la matriz de correlación. Estos errores deben tratarse en Data Preparation antes de construir cualquier modelo predictivo.

---

## 4. Conclusión

Las dos primeras fases de CRISP-DM han permitido establecer un marco sólido para el desarrollo del proyecto. En la fase de Business Understanding se definieron los objetivos del análisis, los KPIs relevantes y la hipótesis de que las desviaciones de ruta no son la única ni la principal causa del atraso. En la fase de Data Understanding se exploró la estructura del dataset, se construyeron las variables derivadas necesarias, se identificaron los problemas de calidad de datos y se analizaron las relaciones entre variables.

Los hallazgos más relevantes son tres. Primero, el dataset contiene errores de registro que afectan el 0.58% de los datos y que deben eliminarse antes del modelado. Segundo, la desviación de ruta tiene una correlación prácticamente nula con el atraso, lo que valida la hipótesis planteada y obliga a explorar variables adicionales. Tercero, la posición de la parada en la secuencia real (`IndexA`) es la variable con mayor correlación con el atraso, sugiriendo que el efecto acumulativo a lo largo de la ruta es un factor importante.

**Pasos propuestos para las fases posteriores:**

- **Data Preparation:** Aplicar las rutinas de limpieza identificadas, eliminar variables redundantes (`Route ID` o `Week ID`, `Stop ID` o `Address ID`), y codificar `Day of Week` como variable dummy.
- **Modeling:** Explorar modelos que capturen relaciones no lineales, como Random Forest o Gradient Boosting. Abordar el desbalance de clases (88.9% vs. 11.1%) mediante SMOTE o ajuste de pesos. Evaluar modelos separados por país.
- **Evaluation:** Para regresión, usar RMSE y MAE. Para clasificación, usar F1-score y AUC-ROC, priorizando el recall de la clase positiva (entregas atrasadas).

---

## 5. Bibliografía

- Perboli, G., & Rosano, M. (2024). *Last-mile delivery route deviations dataset: planned vs. actual routes*. Mendeley Data. https://doi.org/10.17632/xxxxxxxxx
- Wirth, R., & Hipp, J. (2000). *CRISP-DM: Towards a standard process model for data mining*. Proceedings of the 4th International Conference on the Practical Applications of Knowledge Discovery and Data Mining.
- McKinney, W. (2010). *Data structures for statistical computing in Python*. Proceedings of the 9th Python in Science Conference.
- Géron, A. (2019). *Hands-on machine learning with Scikit-Learn, Keras, and TensorFlow* (2nd ed.). O'Reilly Media.
