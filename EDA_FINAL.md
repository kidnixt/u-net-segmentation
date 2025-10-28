
## 📋 Resumen del Análisis Exploratorio de Datos (EDA)

Basado en el análisis de 10 pasos del *dataset* de 2133 imágenes de $800 \times 800$, hemos identificado las características clave que definirán la arquitectura, el entrenamiento y las métricas de evaluación del modelo de segmentación U-Net.

### Hallazgos Clave

1.  **Formato (Integridad):** El *dataset* es de alta calidad, con pares $1:1$ y un formato fijo de $800 \times 800$, sin necesidad de *resize* (Paso 1).
2.  **Forma (Complejidad):** Las máscaras son geométricamente **complejas** (Compacidad $\mu=2.83$, Paso 9) y contienen **ruido intrínseco** (99.15% tienen "agujeros", Paso 6).
3.  **Clase (Desequilibrio):** La clase "Persona" es la **minoritaria** ($\mu=39.90%$, Paso 3), lo que justifica una función de pérdida especializada.
4.  **Posición (Traslación):** Los sujetos están **descentrados** ($\mu=119.09$ px del centro, Paso 10), requiriendo invariancia a la posición.
5.  **Calidad de Imagen (Textura):** Las imágenes son **excesivamente "limpias"** y carecen de textura/ruido (FFT de baja frecuencia, $\sigma_n=10.82$, Pasos 7 y 8).
6.  **Calidad de Imagen (Color):** Las imágenes tienen una **baja saturación** ($\mu=67.47$, Paso 5), creando un sesgo hacia colores apagados.

-----

## 🛠️ Plan de Acción: Decisiones de Diseño Justificadas por el EDA

A continuación, se detallan las acciones de implementación (arquitectura, pérdida, *augmentation* y post-procesamiento) justificadas por los hallazgos del EDA.

### 1\. Arquitectura del Modelo y Función de Pérdida

| Decisión | Justificación del EDA |
| :--- | :--- |
| **Arquitectura: U-Net** | **Paso 4 (Visual) y Paso 9 (Compacidad $\mu=2.83$):** La alta complejidad de los bordes (cabello, dedos) requiere las **Skip Connections** de la U-Net para transferir información de alta resolución y lograr una segmentación precisa. |
| **Función de Pérdida: Dice Loss** | **Paso 3 (Desequilibrio $\mu=39.90\%$):** Dado que la clase "Persona" es minoritaria (vs. $60\%$ de fondo), la Dice Loss es necesaria para optimizar la métrica de superposición (IoU) y evitar que el modelo se sobreajuste a la clase mayoritaria (fondo). |
| **Métrica Principal: Dice Coefficient** | **Paso 3:** Es la métrica de evaluación principal, ya que mide el rendimiento real en la clase minoritaria, que es el objetivo del proyecto. |

### 2\. Pipeline de Pre-procesamiento

| Decisión | Justificación del EDA |
| :--- | :--- |
| **Carga de Máscaras** | Usar la función `load_mask_robust` (Canal Alfa/Grayscale) para decodificar las máscaras binarias (Paso 1). |
| **Normalización: Z-Score** | **Paso 2 (Estadísticas de Color):** Normalizar las imágenes de entrada usando la media y desviación estándar calculadas para estabilizar y acelerar la convergencia del entrenamiento. |
| **Valores de Normalización** | **Media ($\mu$):** `[0.515, 0.484, 0.460]` <br> **Desv. Est. ($\sigma$):** `[0.314, 0.303, 0.303]` |

### 3\. Pipeline de Data Augmentation

Este es el componente más crítico identificado en el EDA para asegurar la generalización del modelo.

#### 3.1 Augmentation Geométrica

| Transformación | Justificación del EDA |
| :--- | :--- |
| **Horizontal Flip** | Estándar para duplicar el *dataset* y lograr invariancia lateral. |
| **Traslación/Shifts Aleatorios** (o `RandomResizedCrop`) | **Paso 10 (Centrado $\mu=119.09$ px):** El *dataset* está descentrado. Esta *augmentation* es **CRÍTICA** para forzar al modelo a ser invariante a la posición del sujeto. |
| **Elastic Deformation** (o `GridDistortion`) | **Paso 9 (Compacidad $\mu=2.83$):** Necesario para simular la variabilidad de las formas orgánicas y los contornos complejos, haciendo el modelo más robusto a las poses difíciles. |
| **Rotaciones Ligeras** ($\pm 15^\circ$) | **Paso 4 (Poses):** Para simular la variabilidad de las posturas corporales. |

#### 3.2 Augmentation de Color y Textura

| Transformación | Justificación del EDA |
| :--- | :--- |
| **Color Jitter (Agresivo)** | **Paso 5 (HSV $\mu$-Sat=67.47):** El *dataset* tiene colores apagados. Se debe aplicar un *jitter* fuerte (ej. `saturation=(0.5, 1.5)`) para forzar al modelo a reconocer al sujeto independientemente de la intensidad del color. |
| **Adición de Ruido (Ej. `GaussianNoise`)** | **Paso 7 (FFT) y Paso 8 ($\sigma_n=10.82$):** El *dataset* es demasiado "limpio" (baja textura). Esta *augmentation* es **CRÍTICA** para evitar el sobreajuste a la suavidad y hacer que el modelo sea robusto al ruido del mundo real. |

### 4\. Post-procesamiento

| Decisión | Justificación del EDA |
| :--- | :--- |
| **Cierre Morfológico (`MORPH_CLOSE`)** | **Paso 6 (Calidad de Máscaras):** El $99.15\%$ de las máscaras tienen "agujeros" (detalles finos). Aplicar un cierre morfológico a la **predicción final** del modelo ayudará a rellenar pequeños fallos de segmentación y suavizar el resultado. |