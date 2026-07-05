**Predicción de ETA y atraso en rutas de última milla mediante minería de datos**

## **Propuesta de proyecto**

### **Tema del proyecto**

**El proyecto se enfocará en un problema de logística de última milla, específicamente en la predicción del tiempo estimado de llegada (ETA) y del atraso de una entrega dentro de un contexto de multi reparto.**

### **Problema a abordar**

**En operaciones de última milla, una de las principales dificultades es anticipar con precisión cuánto tardará una entrega en completarse. Las diferencias entre la ruta planificada y la ruta efectivamente realizada por el conductor pueden generar variaciones en los tiempos de llegada, afectando la planificación, el seguimiento operativo y el cumplimiento de ventanas horarias.**

### **Variable objetivo**

#### **Variable principal**

**La variable objetivo principal será una variable numérica asociada al tiempo estimado de llegada o a los minutos de atraso de una entrega.**

**Esto implica que el proyecto se abordará principalmente como un problema de regresión, ya que el modelo buscará predecir un valor continuo.**

#### **Variable descriptiva**

* **0 \= entrega a tiempo**  
* **1 \= entrega atrasada**

**Esta formulación corresponde a un problema de clasificación. No será el foco principal, pero podrá incorporarse como análisis complementario si resulta pertinente dentro del desarrollo del trabajo.**

### **Dataset seleccionado**

**El dataset seleccionado para el proyecto es “Last-mile delivery route deviations dataset: planned vs. actual routes”, publicado en Mendeley Data el 13 de diciembre de 2024\. Según su descripción oficial, este conjunto de datos registra rutas de última milla de una empresa logística en dos países e incluye la secuencia planificada de paradas, la secuencia real seguida por los conductores, además de identificación del conductor, información de llegada, ventanas de tiempo y distancias entre paradas. También se indica que los valores de tiempo están codificados como diferencias respecto del instante inicial de cada ruta.**

### **Justificación de la elección del dataset**

**Este dataset fue elegido porque representa de mejor manera un escenario de última milla real que otros conjuntos de datos más tradicionales o sintéticos. Su principal fortaleza es que permite estudiar la diferencia entre lo planificado y lo ejecutado en una ruta de reparto, lo que se relaciona directamente con el problema de predecir ETA o atraso. Además, la presencia de información de llegada y ventanas de tiempo lo convierte en una base adecuada para construir modelos predictivos más cercanos a una operación logística real.**

### **Objetivo general**

**Desarrollar un modelo de minería de datos capaz de predecir el tiempo estimado de llegada o los minutos de atraso en entregas de última milla, utilizando información operativa de rutas planificadas y rutas reales.**

### **Objetivos específicos**

* **Analizar y comprender la estructura del dataset seleccionado.**

* **Identificar las variables más relevantes para la predicción del ETA o atraso.**

* **Realizar el preprocesamiento y preparación de los datos para su uso en modelos predictivos.**

* **Entrenar y evaluar un modelo de regresión orientado a estimar tiempos de llegada o minutos de atraso.**

* **Explorar, de manera complementaria, una formulación de clasificación para distinguir entre entregas a tiempo y entregas atrasadas.**

### **Enfoque del trabajo**

**El desarrollo del proyecto se centrará principalmente en un enfoque de regresión, ya que este permite modelar de manera más precisa el tiempo de llegada o el atraso como valor numérico. Como extensión opcional, se podrá construir un segundo modelo de clasificación para comparar resultados y enriquecer el análisis.**

### **Síntesis**

**En conclusión, el proyecto se centrará en la predicción de ETA y atraso en logística de última milla, usando como base un dataset real que contiene rutas planificadas y reales, información de llegada y ventanas de tiempo. La variable principal será numérica y permitirá abordar el problema como regresión, dejando abierta la posibilidad de complementarlo con una formulación de clasificación.**

