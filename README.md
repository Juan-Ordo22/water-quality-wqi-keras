# 1. Water Quality WQI Prediction with Keras

# Informe del Proyecto: Análisis y Predicción del Índice de Calidad del Agua mediante WQI y Keras

## 2. Descripción general

Este proyecto desarrolla un flujo completo de análisis de calidad del agua utilizando un conjunto de datos con mediciones fisicoquímicas y microbiológicas tomadas en diferentes estaciones de India. El objetivo principal es calcular un índice de calidad del agua, conocido como **WQI** (*Water Quality Index*), y posteriormente construir un modelo de regresión en **Keras** capaz de predecir dicho índice a partir de variables asociadas a la calidad del agua.

El trabajo parte de un cuaderno original que ya contenía procesamiento con PySpark, cálculo del WQI y una red neuronal inicial. Sin embargo, durante el desarrollo se identificaron varios puntos de mejora importantes, especialmente en el tratamiento de valores nulos, la interpretación de la escala WQI, la preparación de datos para el modelo y la evaluación del desempeño.

Como resultado, el proyecto fue reorganizado y mejorado para tener un flujo más sólido, interpretable y defendible desde el punto de vista metodológico.

---

## 3. Objetivo del proyecto

El objetivo general del proyecto es:

> Calcular y modelar el índice de calidad del agua WQI a partir de variables fisicoquímicas y microbiológicas, aplicando técnicas de limpieza de datos, análisis exploratorio, visualización geográfica y redes neuronales en Keras.

Los objetivos específicos son:

1. Cargar y explorar un conjunto de datos de calidad del agua.
2. Limpiar valores faltantes y convertir variables numéricas correctamente.
3. Calcular puntuaciones individuales de calidad para cada parámetro.
4. Construir un índice WQI mediante una suma ponderada.
5. Clasificar la calidad del agua a partir del valor del WQI.
6. Visualizar la información mediante gráficas y mapas.
7. Entrenar un modelo de regresión en Keras para predecir el WQI.
8. Comparar versiones del modelo y justificar las mejoras aplicadas.
9. Analizar el desempeño final del modelo mediante métricas de regresión.

---

## 4. Herramientas utilizadas

El proyecto utiliza principalmente:

- **Python**
- **PySpark**
- **Pandas**
- **NumPy**
- **Matplotlib**
- **Seaborn**
- **GeoPandas**
- **Scikit-learn**
- **TensorFlow / Keras**

PySpark se utiliza en las primeras etapas para la carga, limpieza y transformación del dataset. Pandas y NumPy se emplean para preparar los datos antes del modelado. Matplotlib y Seaborn se usan para las visualizaciones, GeoPandas para los mapas, y Keras para construir la red neuronal de regresión.

---

## 5. Dataset utilizado

El conjunto de datos contiene mediciones de calidad del agua en diferentes estaciones de India. Entre sus columnas principales se encuentran:

| Variable | Descripción |
|---|---|
| `STATION CODE` | Código de la estación de medición |
| `LOCATIONS` | Ubicación de la estación |
| `STATE` | Estado de India donde se tomó la muestra |
| `TEMP` | Temperatura del agua |
| `DO` | Oxígeno disuelto |
| `pH` | Nivel de acidez o basicidad |
| `CONDUCTIVITY` | Conductividad eléctrica |
| `BOD` | Demanda bioquímica de oxígeno |
| `NITRATE_N_NITRITE_N` | Concentración de nitratos y nitritos |
| `FECAL_COLIFORM` | Indicador de contaminación fecal |
| `TOTAL_COLIFORM` | Coliformes totales |

La variable objetivo `WQI` no venía originalmente en el dataset. Fue calculada dentro del cuaderno mediante reglas de puntuación y pesos definidos para cada parámetro.

---

## 6. Limpieza de datos

### 6.1 Problema identificado

En el cuaderno original se verificaban valores nulos, pero algunos valores faltantes estaban escritos como texto `"NA"`. Esto generaba un problema porque Spark no los interpretaba inicialmente como valores nulos reales.

Además, el cuaderno original eliminaba filas con valores nulos mediante una consulta SQL, lo que podía reducir innecesariamente el tamaño del dataset.

### 6.2 Decisión tomada

Se decidió reemplazar los valores `"NA"` por nulos reales y posteriormente convertir las columnas numéricas a tipo `FloatType`.

Luego, en lugar de eliminar filas con nulos, se aplicó **imputación por mediana**.

Esta decisión se justificó porque:

- El dataset tiene un tamaño limitado.
- Eliminar filas podía reducir la cantidad de información disponible.
- Variables como `CONDUCTIVITY` y `FECAL_COLIFORM` pueden tener valores extremos.
- La mediana es más robusta que el promedio frente a valores atípicos.

También se eliminó la columna `TOTAL_COLIFORM`, ya que no se utilizaba directamente en el cálculo del WQI y presentaba valores faltantes.

---

## 7. Análisis exploratorio y visualizaciones

Durante el análisis exploratorio se generaron diferentes gráficas para comprender el comportamiento de las variables.

### 7.1 Oxígeno disuelto y pH

Se graficaron `DO` y `pH` para observar su comportamiento a lo largo de las muestras.

El pH presentó un comportamiento relativamente estable, mientras que el oxígeno disuelto mostró mayor variabilidad. Ambas variables son importantes porque un pH dentro de rangos adecuados y un buen nivel de oxígeno disuelto suelen asociarse con mejores condiciones de calidad del agua.

### 7.2 BOD y nitratos/nitritos

Se compararon `BOD` y `NITRATE_N_NITRITE_N`.

El `BOD` permite identificar posible contaminación orgánica, mientras que los nitratos y nitritos pueden relacionarse con contaminación por fertilizantes, aguas residuales o exceso de nutrientes.

### 7.3 Conductividad y coliformes fecales

También se graficaron `CONDUCTIVITY` y `FECAL_COLIFORM`.

Estas variables mostraron alta dispersión y posibles valores extremos. Esta observación fue fundamental para decidir aplicar una transformación logarítmica durante el modelado.

---

## 8. Cálculo de puntuaciones individuales `qr`

Para calcular el WQI, cada variable original fue transformada en una puntuación individual de calidad. Estas puntuaciones se nombraron con el prefijo `qr`.

Las variables generadas fueron:

| Variable derivada | Parámetro evaluado |
|---|---|
| `qrPH` | pH |
| `qrDO` | Oxígeno disuelto |
| `qrCOND` | Conductividad |
| `qrBOD` | Demanda bioquímica de oxígeno |
| `qrNN` | Nitratos y nitritos |
| `qrFecal` | Coliformes fecales |

Cada puntuación asigna valores como 0, 40, 60, 80 o 100 según rangos de calidad definidos. Los valores altos indican mejores condiciones para el parámetro evaluado.

Estas variables `qr` permiten expresar todos los parámetros en una escala común de calidad.

---

## 9. Cálculo del índice WQI

Después de calcular las puntuaciones individuales, se aplicó una suma ponderada para obtener el índice WQI.

Los pesos utilizados fueron:

| Parámetro | Peso |
|---|---:|
| pH | 0.165 |
| Oxígeno disuelto | 0.281 |
| Conductividad | 0.234 |
| BOD | 0.009 |
| Nitratos/Nitritos | 0.028 |
| Coliformes fecales | 0.281 |

La fórmula general utilizada fue:

```text
WQI = qrPH*0.165 + qrDO*0.281 + qrCOND*0.234 + qrBOD*0.009 + qrNN*0.028 + qrFecal*0.281
```

En este proyecto, el WQI se interpreta como un índice donde los valores altos indican mejor calidad del agua, ya que las puntuaciones individuales también asignan valores altos a condiciones favorables.

---

## 10. Clasificación de calidad del agua

A partir del WQI se asignó una categoría de calidad a cada muestra.

La clasificación utilizada fue:

| Rango WQI | Categoría |
|---|---|
| 75 - 100 | Excelente |
| 50 - 75 | Buena |
| 25 - 50 | Baja |
| 0 - 25 | Muy_Baja |

Esta clasificación fue corregida respecto a la lógica inicial, ya que en el cuaderno original la interpretación del WQI podía quedar invertida. Dado que el WQI calculado aumenta cuando las condiciones son mejores, lo correcto es considerar que valores altos representan mejor calidad.

La distribución obtenida fue:

| Categoría | Cantidad |
|---|---:|
| Buena | 321 |
| Baja | 125 |
| Excelente | 74 |
| Muy_Baja | 14 |

Esto muestra que la mayoría de registros se concentran en la categoría `Buena`, mientras que `Muy_Baja` tiene muy pocos datos.

---

## 11. Visualización geográfica

El cuaderno también incluye una etapa de análisis geográfico utilizando un shapefile de los estados de India.

### 11.1 Mapa base de India

Primero se graficó el mapa base de India y se superpusieron etiquetas con los nombres de los estados. Esto permite verificar visualmente que las geometrías fueron cargadas correctamente y que los nombres de los estados están disponibles para realizar la unión con el dataset.

### 11.2 Mapa del índice WQI

Para representar el WQI geográficamente, se calculó el promedio del WQI por estado. Esta decisión mejora el análisis, ya que evita que un estado quede representado solamente por una muestra individual.

Luego se unió la información promedio de WQI con el mapa mediante la columna `STATE`.

Los estados sin datos disponibles se muestran en color gris.

### 11.3 Histograma de WQI por estado

Además del mapa, se generó un histograma horizontal del WQI promedio por estado. Esta gráfica permite comparar de manera directa los valores promedio entre estados.

Mientras el mapa ofrece una lectura espacial, el histograma permite una comparación cuantitativa más clara.

---

## 12. Preparación para Machine Learning

Para entrenar el modelo, se utilizó como base el DataFrame `df06`, que contiene:

- Variables originales.
- Variables derivadas `qr`.
- Pesos de cada parámetro.
- Índice `WQI`.
- Categoría `CALIDAD`.
- Estados normalizados para visualización geográfica.

Los datos se convirtieron a Pandas para ser usados con Scikit-learn y Keras.

Se verificó nuevamente la ausencia de valores nulos antes del entrenamiento.

---

## 13. Decisión de cambiar el modelo original

### 13.1 Modelo original

El cuaderno original incluía una red neuronal con una arquitectura mucho más grande, compuesta por varias capas densas de 350 neuronas. Esta red tenía una cantidad muy alta de parámetros en comparación con el tamaño del dataset.

Además, el modelo original utilizaba como entrada principalmente las variables `qr` para predecir el WQI. Esto generaba un resultado aparentemente muy bueno, pero metodológicamente era limitado, porque el WQI se calcula directamente a partir de esas mismas variables.

Es decir, la red neuronal estaba aprendiendo una fórmula casi directa.

### 13.2 Problemas del modelo original

Se identificaron los siguientes problemas:

1. La arquitectura era demasiado grande para el tamaño del dataset.
2. No se estaba evaluando de forma adecuada sobre datos de prueba.
3. No había una validación clara durante el entrenamiento.
4. No se aplicaba `EarlyStopping`.
5. No se justificaba suficientemente la relación entre variables de entrada y WQI.
6. No se trataban correctamente valores extremos en variables como `CONDUCTIVITY` y `FECAL_COLIFORM`.

### 13.3 Nuevo enfoque

Se decidió construir un modelo con un enfoque distinto:

1. Primero se entrenó un modelo con variables originales.
2. Luego se aplicó transformación logarítmica a variables con alta dispersión.
3. Finalmente se incorporaron variables `qr` .

Esta estrategia permite mostrar una evolución clara del modelo y justificar cada mejora.

---

## 14. Transformación logarítmica

Durante el análisis se observó que `CONDUCTIVITY` y `FECAL_COLIFORM` tenían valores muy dispersos. Por esta razón se aplicó:

```text
log1p(x) = log(1 + x)
```

Esta transformación reduce el impacto de valores extremos sin perder el orden de los datos. Además, es adecuada cuando existen valores iguales a cero.

Se crearon dos nuevas variables:

- `CONDUCTIVITY_LOG`
- `FECAL_COLIFORM_LOG`

Estas variables sustituyeron a las originales en el modelo base mejorado.

---

## 15. Modelo de regresión en Keras

El modelo final es una red neuronal de regresión. Su objetivo es predecir el valor numérico del índice WQI.

### 15.1 Variables de entrada del modelo final

El modelo final utilizó:

- `pH`
- `DO`
- `CONDUCTIVITY_LOG`
- `BOD`
- `NITRATE_N_NITRITE_N`
- `FECAL_COLIFORM_LOG`
- `qrPH`
- `qrDO`
- `qrCOND`
- `qrBOD`
- `qrNN`
- `qrFecal`

La variable objetivo fue:

- `WQI`

### 15.2 Justificación de las variables `qr`

Estas variables representan una transformación experta de los parámetros originales, ya que expresan cada medición en una escala común de calidad.

Es importante aclarar que las variables `qr` están directamente relacionadas con el cálculo del WQI. Por lo tanto, el modelo final no debe interpretarse como un modelo que descubre completamente la calidad del agua desde cero, sino como un modelo guiado por características derivadas.

Esto es válido dentro del objetivo del proyecto, porque se busca predecir el WQI calculado a partir de los criterios definidos en el cuaderno.

---

## 16. Arquitectura del modelo final

El modelo final utiliza capas densas con activación ReLU y capas Dropout para reducir el riesgo de sobreajuste.

La salida del modelo es una sola neurona con activación lineal, adecuada para regresión.

Una arquitectura representativa es:

```text
Dense(128, activation='relu')
Dropout(0.15)
Dense(64, activation='relu')
Dropout(0.15)
Dense(32, activation='relu')
Dense(1, activation='linear')
```

Se utilizó:

- Optimizador: Adam.
- Función de pérdida: MSE.
- Métrica adicional: MAE.
- Validación interna.
- EarlyStopping.
- Escalado robusto de variables.

---

## 17. Comparación de versiones del modelo

Durante el desarrollo se evaluaron varias versiones.

| Versión | Características | MAE | RMSE | R² |
|---|---|---:|---:|---:|
| Modelo inicial mejorado | Variables originales + `log1p` | 5.62 | 6.87 | 0.755 |
| Modelo final | Variables originales + `log1p` + variables `qr` | 1.34 | 2.20 | 0.975 |

La comparación muestra que:

1. La transformación `log1p` mejoró el desempeño al reducir el impacto de valores extremos.
2. La incorporación de variables `qr` mejoró significativamente el aprendizaje porque estas variables resumen cómo cada parámetro contribuye al WQI.
3. El modelo final tiene una capacidad predictiva alta para estimar el WQI.

---

## 18. Resultados finales del modelo

Los resultados finales fueron:

| Métrica | Valor |
|---|---:|
| MSE | 4.83 |
| RMSE | 2.20 |
| MAE | 1.34 |
| R² | 0.975 |

### 18.1 Interpretación del MAE

El MAE fue aproximadamente 1.34. Esto significa que el modelo se equivoca en promedio alrededor de 1.34 puntos de WQI.

Dado que el WQI se expresa en una escala aproximada de 0 a 100, este error es bajo.

### 18.2 Interpretación del RMSE

El RMSE fue aproximadamente 2.20. Esta métrica penaliza más los errores grandes, por lo que un valor bajo indica que el modelo no solo comete errores pequeños en promedio, sino que también reduce errores extremos.

### 18.3 Interpretación del R²

El R² fue aproximadamente 0.975. Esto significa que el modelo explica cerca del 97.5% de la variabilidad del WQI.

Este resultado indica una capacidad predictiva muy alta dentro del conjunto de prueba.

---

## 19. Análisis de las gráficas del modelo

### 19.1 Curva de pérdida

La curva de pérdida muestra la evolución del error de entrenamiento y validación. Una reducción progresiva del error indica que el modelo aprendió durante el entrenamiento.

La cercanía entre la curva de entrenamiento y validación sugiere que no hay evidencia fuerte de sobreajuste.

### 19.2 WQI real vs WQI predicho

Esta gráfica compara los valores reales del WQI con los valores predichos por el modelo. La línea diagonal representa el comportamiento ideal, donde el valor real y el predicho son iguales.

En el modelo final, la mayoría de puntos se ubica cerca de la línea diagonal, lo cual es coherente con el alto valor de R² y el bajo MAE.

### 19.3 Distribución de errores

La distribución de errores permite observar si el modelo tiende a sobrestimar o subestimar el WQI.

Una distribución concentrada alrededor de cero indica que el modelo comete errores pequeños y no presenta un sesgo fuerte.

### 19.4 Comparación de valores reales y predichos

La comparación de algunas muestras del conjunto de prueba permite observar de forma directa qué tan cercanos son los valores estimados por el modelo frente a los valores reales.

---

## 20. Discusión metodológica

El resultado final es fuerte, pero debe interpretarse correctamente.

El modelo final alcanza un desempeño alto porque combina:

1. Variables originales.
2. Variables transformadas con `log1p`.
3. Variables derivadas `qr`.

Las variables `qr` contienen información directamente relacionada con el cálculo del WQI. Por lo tanto, el modelo se beneficia de una representación experta del problema.

Esto no representa un error metodológico si se declara claramente, ya que el objetivo no es descubrir una etiqueta externa independiente, sino predecir el índice WQI construido a partir de reglas definidas.

En otras palabras, el modelo final es un modelo de regresión guiado.

---

## 21. Conclusiones

Este proyecto permitió construir un flujo completo para analizar, calcular y predecir la calidad del agua mediante el índice WQI.

Las principales conclusiones son:

1. El tratamiento correcto de valores `"NA"` fue fundamental para evitar errores en el procesamiento.
2. La imputación por mediana permitió conservar registros sin verse afectada por valores extremos.
3. El cálculo de puntuaciones `qr` permitió transformar variables de diferentes escalas en una medida común de calidad.
4. La corrección de la interpretación del WQI fue necesaria para clasificar correctamente la calidad del agua.
5. Las visualizaciones ayudaron a identificar dispersión, valores extremos y distribución de categorías.
6. La transformación `log1p` mejoró el desempeño del modelo al reducir el impacto de variables altamente dispersas.
7. La incorporación de variables `qr` produjo una mejora significativa en la capacidad predictiva.
8. El modelo final alcanzó un MAE de 1.34 y un R² de 0.975, indicando una alta precisión en la predicción del WQI.

---

## 22. Cómo ejecutar el proyecto

1. Clonar el repositorio.

```bash
git clone <https://github.com/Juan-Ordo22/water-quality-wqi-keras.git>
cd water-quality-wqi-keras
```

2. Instalar dependencias.

```bash
pip install -r requirements.txt
```

3. Abrir el notebook.

```bash
jupyter notebook
```

4. Ejecutar el cuaderno en orden desde la primera celda.

---
## 23. Bibliografía y fuentes consultadas

Las siguientes fuentes fueron consultadas como apoyo conceptual para la construcción, mejora y análisis del modelo de Machine Learning aplicado al índice de calidad del agua `WQI`.

### Métricas de evaluación de modelos

Acaddemia. (s.f.). *Métricas de Machine Learning: guía para evaluación de modelos*. Acaddemia.  
Fuente utilizada para apoyar la explicación de métricas de evaluación como `MAE`, `MSE`, `RMSE` y `R²`, empleadas para analizar el desempeño del modelo de regresión.

> Nota: esta fuente fue usada como referencia conceptual general sobre métricas de evaluación. Se recomienda verificar manualmente el contenido antes de la entrega final.

### Regresión con redes neuronales

Verma, N. (2023, noviembre 7). *Neural Network Regression Implementation and Visualization in Python*. Medium.  
Fuente utilizada para fundamentar la implementación de una red neuronal aplicada a problemas de regresión. Esta referencia fue útil para comprender que, en una regresión con redes neuronales, la salida del modelo corresponde a un valor numérico continuo, como ocurre en este proyecto con la predicción del índice `WQI`.

También sirvió como apoyo para justificar el uso de una capa de salida lineal, la división entre entrenamiento y prueba, el uso de funciones de pérdida como `MSE` y la evaluación mediante métricas como `MAE`, `RMSE` y `R²`.

### Optimizador Adam

DataCamp. (2024, agosto 30). *Tutorial del Optimizador Adam: intuición e implementación en Python*. DataCamp.  
Fuente utilizada para justificar el uso del optimizador `Adam` en el entrenamiento de la red neuronal. Adam es un algoritmo de optimización ampliamente usado en aprendizaje profundo, ya que adapta la tasa de aprendizaje de cada parámetro y combina ideas de técnicas como Momentum y RMSprop.

En este proyecto se utilizó `Adam` porque permite un entrenamiento eficiente y estable de la red neuronal de regresión.

### Regresión lineal y fundamentos de modelos de regresión

Google for Developers. (s.f.). *Regresión lineal*. Machine Learning Crash Course.  
Fuente utilizada como apoyo para comprender los fundamentos de los modelos de regresión, en los cuales el objetivo es predecir un valor numérico continuo. Aunque el modelo final del proyecto utiliza una red neuronal, la idea base sigue siendo un problema de regresión, ya que se busca estimar el valor del índice `WQI`.

### Función de activación ReLU

Krishnamurthy, B. (2024, febrero 26). *An Introduction to the ReLU Activation Function*. Built In.  
Fuente utilizada para justificar el uso de la función de activación `ReLU` en las capas ocultas del modelo. ReLU introduce no linealidad en la red neuronal, permitiendo que el modelo aprenda relaciones más complejas entre las variables de entrada y el índice `WQI`.

En el modelo desarrollado, `ReLU` se utilizó en las capas densas ocultas, mientras que la capa de salida utilizó activación lineal por tratarse de un problema de regresión.

### Introducción práctica a redes neuronales artificiales

Curo De La Cruz, W. (2024, octubre 5). *Crea tu primera Red Neuronal Artificial*. Apuntes de Walther Curo.  
Fuente utilizada como apoyo práctico para comprender la estructura general de una red neuronal artificial y el flujo básico de trabajo en un cuaderno de Python: preparación de datos, definición del modelo, entrenamiento y análisis de resultados.

Esta fuente fue útil como apoyo general para documentar la construcción del modelo en Keras.

### Redes neuronales

Google for Developers. (s.f.). *Redes neuronales*. Machine Learning Crash Course.  
Fuente utilizada para reforzar los conceptos generales sobre redes neuronales dentro del aprendizaje automático. Esta referencia apoya la explicación de cómo una red neuronal utiliza capas, neuronas y funciones de activación para aprender relaciones entre variables de entrada y una salida esperada.

En este proyecto, estos conceptos se aplican al entrenamiento de una red neuronal para predecir el índice `WQI`.

### Transformación logarítmica e ingeniería de características

APXML. (s.f.). *Log Transformation for Skewed Data*. APXML.  
Fuente utilizada para justificar la aplicación de la transformación logarítmica `log1p` sobre las variables `CONDUCTIVITY` y `FECAL_COLIFORM`.

Esta transformación fue aplicada porque dichas variables presentaban alta dispersión y posibles valores extremos. El uso de `log1p` permitió reducir el impacto de valores muy grandes y mejorar el desempeño del modelo de regresión.
