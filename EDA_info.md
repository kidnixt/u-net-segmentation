
## 🔬 Análisis 1: Integridad y Diseño Estructural del Dataset

Este análisis confirma la alta calidad y la estructura uniforme del *dataset*, lo que simplifica el *pipeline* de preprocesamiento y permite el diseño directo de la arquitectura U-Net.

### 1.1 Resumen del Conteo y Formato de Datos

| Métrica | Valor | Conclusión e Impacto |
| :--- | :--- | :--- |
| **Total de Imágenes (Train)** | 2133 | Conjunto de datos de tamaño moderado. |
| **Total de Máscaras (Train)** | 2133 | **Integridad Perfecta.** Coincidencia $1:1$ de pares. |
| **Tamaño Fijo** | $800 \times 800$ | **CRÍTICO.** Elimina la necesidad de *Data Augmentation* de escalado o *padding* variable. La U-Net se diseña para esta dimensión exacta. |
| **Máscaras Vacías** | 0 ($0.00\%$) | **Alta Calidad.** Todos los *samples* de entrenamiento contienen el objeto de interés. |
| **Decodificación** | Exitosa (Canal Alfa / Grayscale) | Se confirma la accesibilidad y validez del *ground truth* binario. |

### 1.2 Implicaciones en el Diseño de la U-Net

El formato uniforme $800 \times 800$ permite una arquitectura U-Net con una **entrada fija**, asegurando que todas las operaciones de *pooling* y convolución sean consistentes. Esto contribuye a la **estabilidad del entrenamiento** desde la primera época.

| Decisión de Diseño | Justificación (Basada en $800 \times 800$) |
| :--- | :--- |
| **Arquitectura** | **U-Net de entrada $800 \times 800$**. El *Encoder* deberá ser lo suficientemente profundo para capturar el contexto global de la escena con un tamaño de *feature map* manejable. |
| **Pre-procesamiento** | **Solo Normalización y Augmentation**. No se requiere un paso de *resize* forzado, optimizando el tiempo de carga de datos. |

---


## 🎨 Análisis 2: Estadísticas de Color y Normalización

Este análisis cuantifica la distribución de intensidad de píxeles, revelando un balance de color y variabilidad significativos que deben ser compensados mediante la normalización.

### 2.1 Resultados Cuantitativos (Media y Desviación Estándar)

| Métrica | Canal Rojo (R) | Canal Verde (G) | Canal Azul (B) |
| :--- | :--- | :--- | :--- |
| **Media ($\mu$)** | **0.515** | **0.484** | **0.460** |
| **Desviación Estándar ($\sigma$)** | **0.314** | **0.303** | **0.303** |

### 2.2 Interpretación del Histograma de Intensidad

  * **Distribución Multimodal:** El histograma muestra claramente **picos en ambos extremos** (cerca de 0.0 y 1.0) para todos los canales, con un fuerte sesgo en los valores bajos (sombras) y altos (luces/fondos blancos). Esto indica una **alta variabilidad de iluminación y contraste** en el *dataset*.
  * **Balance de Color:** La media de los canales **R (0.515) y G (0.484)** es ligeramente **superior** a la media del canal **B (0.460)**, sugiriendo una ligera dominante de tonos cálidos o una mayor presencia de piel/fondos rojizos/verdosos.
  * **Contraste Elevado:** Los valores de la **Desviación Estándar ($\sigma \approx 0.30$)** son relativamente altos. Para datos normalizados en $[0, 1]$, un $\sigma$ de $0.3$ a $0.35$ indica una **distribución amplia** de intensidades (alto contraste y alta variabilidad de iluminación).

### 2.3 Justificación del Pre-procesamiento: Normalización Z-score

El objetivo del pre-procesamiento es transformar los datos de entrada para que la red neuronal pueda aprender de forma más eficiente.

| Decisión de Pre-procesamiento | Justificación y Ventaja |
| :--- | :--- |
| **Normalización Z-score** | Se implementará la fórmula $\text{Imagen}_{\text{norm}} = (\text{Imagen} - \mu) / \sigma$. |
| **Impacto en el Modelo** | Esta técnica **centra los datos** alrededor de cero ($\mu$) y los **escala** a una desviación estándar unitaria ($\sigma$), previniendo que los valores de intensidad más grandes (ej., píxeles blancos) dominen la función de activación de las capas iniciales de la U-Net. Esto es **crítico** para la convergencia rápida y estable del optimizador. |
| **Valores a Usar** | **Media:** $[0.515, 0.484, 0.460]$ <br> **Desv. Est.:** $[0.314, 0.303, 0.303]$ |

-----


## 🧍‍♂️ Análisis 3: Desequilibrio de Píxeles y Función de Pérdida

Este análisis cuantifica la proporción de la **Clase Positiva (Persona)** y justifica la necesidad de utilizar una función de pérdida robusta contra el desequilibrio de clases (píxeles).

### 3.1 Resultados Cuantitativos del Desequilibrio

| Métrica | Valor | Conclusión e Impacto |
| :--- | :--- | :--- |
| **Píxeles Positivos ($\mu$ Promedio)** | **$39.90\%$** | La clase de interés ("Persona") no es tan minoritaria como se podría esperar en segmentación, pero sigue siendo la **clase minoritaria** frente al $60.10\%$ de "Fondo". |
| **Desviación Estándar** | $21.16\%$ | **Alta Variabilidad.** La segmentación debe ser efectiva tanto para personas grandes (hasta $100\%$) como para personas pequeñas (desde $1.04\%$). |
| **Rango de Área** | $1.04\%$ a $100.00\%$ | **Reto de Escala.** El modelo debe aprender a segmentar objetos que varían dramáticamente en tamaño (desde pequeños detalles hasta cuerpos completos). |

### 3.2 Interpretación del Histograma


* **Distribución Ancha:** El histograma es amplio, con una dispersión significativa de la media. Esto confirma la **alta variabilidad en la escala y pose** de las personas. La red debe ser robusta a la escala (lo cual el U-Net ya maneja gracias a su estructura piramidal).
* **Ausencia de Desequilibrio Extremo:** El promedio de casi $40\%$ es mucho más alto que el $1-5\%$ típico en segmentación médica. Esto atenúa el riesgo de que el modelo solo aprenda "Fondo", pero el riesgo sigue siendo significativo.

### 3.3 Justificación de la Función de Pérdida y Métrica

Dada la variabilidad en el área y el hecho de que el "Fondo" sigue siendo la clase mayoritaria ($60\%$), la función de pérdida debe priorizar la precisión en la clase minoritaria ("Persona").

| Decisión de Modelado | Justificación de los Resultados |
| :--- | :--- |
| **Función de Pérdida** | **Dice Loss o Focal Loss (Recomendado: Dice Loss).** La **Cross-Entropy (BCE)** tradicional penalizaría en exceso a los píxeles de fondo, resultando en un modelo que predice "Fondo" con demasiada facilidad. La Dice Loss se enfoca en la superposición (Intersection over Union, IoU), que es crucial para la segmentación y maneja mejor este nivel de desequilibrio. |
| **Métrica de Evaluación** | **Dice Coefficient (IoU).** Será la métrica principal para la validación y el *Test Set*. A diferencia de la precisión simple, el IoU refleja con precisión la calidad de la segmentación en la clase minoritaria, que es el objetivo del proyecto. |

---


## 👁️ Análisis 4: Visualización de Bordes y Arquitectura

La inspección visual de los pares Imagen-Máscara y su *Overlay* valida la calidad del *ground truth* y proporciona la justificación crítica para la selección de una arquitectura que priorice la **precisión espacial** y la **captura de detalles finos**.

### 4.1 Observaciones Clave del *Overlay*


* **Alineación Precisa:** Los *overlays* (Columna 3) muestran una alineación excelente entre la máscara y el contorno real de la persona. Esto confirma la **alta calidad del etiquetado**, minimizando el ruido en las etiquetas que podría confundir al modelo.
* **Contornos Complejos:** Las máscaras deben capturar bordes finos e irregulares, incluyendo:
    * **Contornos Orgánicos:** Cabello, manos, dedos (Ej. `1286.png`).
    * **Contornos de Ropa:** Pliegues del vestido o tela suelta (Ej. `1870.png`).
    * **Espacios Internos:** El espacio entre las piernas o brazos (Ej. `387.png`).
* **Variabilidad de Pose y Escala:** Las imágenes capturan poses muy variadas (sentado, agachado, de pie, vista lateral/trasera), lo que demanda que el modelo aprenda características invariantes a la pose.

### 4.2 Justificación de la Arquitectura U-Net

El reto principal de este *dataset* es la **precisión de la segmentación a nivel de píxel** en los bordes orgánicos y complejos.

| Decisión de Modelado | Justificación de los Resultados |
| :--- | :--- |
| **Arquitectura Principal** | **U-Net.** El modelo de segmentación *debe* ser una U-Net (o variante como FPN/LinkNet) debido a la **necesidad crítica de capturar el detalle del borde**. |
| **Importancia de las *Skip Connections*** | **Esencial.** El *Encoder* (que reduce la resolución para capturar el **contexto** global) perdería los detalles finos. Las *skip connections* transfieren la **información de alta resolución** (los bordes) directamente desde el *Encoder* al *Decoder*. Esto asegura que la reconstrucción final de la máscara sea nítida y precisa, en lugar de borrosa o suave. |
| **Generalización** | El modelo debe ser capaz de generalizar los bordes en condiciones variadas de fondo, iluminación y color (lo cual será reforzado por el *Data Augmentation*). |


---

## 🎨 Análisis 5: Brillo, Contraste y Saturación (HSV)

Este análisis cuantifica la variabilidad del color y la iluminación, demostrando la necesidad de aplicar **Data Augmentation** para hacer que la U-Net sea robusta a las condiciones de iluminación del mundo real.

### 5.1 Resultados Cuantitativos del Análisis HSV

| Métrica | Valor Promedio ($\mu$) | Conclusión |
| :--- | :--- | :--- |
| **Brillo** (Canal V) | **$140.36$** (de 255) | **Iluminación Neutra a Brillante.** La media está ligeramente por encima del punto medio ($128$), indicando que el *dataset* tiende a estar bien o sobre expuesto. |
| **Contraste** ($\sigma$ de V) | **$60.39$** (de 255) | **Contraste Moderado.** La distribución es claramente normal (forma de campana), lo que significa que el contraste es consistente, pero no extremo. |
| **Saturación** (Canal S) | **$67.47$** (de 255) | **Saturación Baja.** La media está bien por debajo del punto medio, y el histograma está sesgado hacia valores bajos (cercanos a 0). Esto sugiere que la mayoría de las imágenes tienen **colores apagados o grisáceos** (ambientes interiores o condiciones de luz difusa). |

### 5.2 Justificación de la Data Augmentation de Color

El objetivo es evitar que el modelo se sobreajuste a la iluminación y saturación promedio del *dataset* (es decir, el modelo no debe fallar solo porque la foto es más brillante o los colores son más vivos).

| Componente HSV | Decisión de Augmentation | Parámetros Sugeridos (Ej. en rango 0.0 a 1.0) |
| :--- | :--- | :--- |
| **Brillo (Valor)** | **Jitter Simétrico.** Dado que la media es $140.36$, la variación debe ser en ambos sentidos para simular días oscuros y días muy soleados. | **(0.7, 1.3):** Multiplicador de $0.7$ (oscuro) a $1.3$ (brillante) sobre la imagen normalizada. |
| **Contraste ($\sigma$ de V)** | **Jitter Moderado.** La distribución es centrada. Se debe aplicar variación para simular fotos "planas" y fotos con luces y sombras duras. | **(0.75, 1.25):** Variación de $\pm 25\%$ en contraste. |
| **Saturación (Canal S)** | **Jitter Agresivo.** La baja media ($67.47$) y el sesgo hacia 0 obligan a **introducir colores más vivos** de forma artificial. | **(0.5, 1.5):** Variación desde $50\%$ menos (casi blanco y negro) hasta $50\%$ más (colores muy vivos). Esto es **crítico** para la generalización. |

---

¡Estos resultados son sorprendentes e importantes! El **Paso 6: Calidad de Máscaras** revela que casi la totalidad del *dataset* está marcado como potencialmente ruidoso ($2115$ de $2133$). Sin embargo, la clave está en el **tipo de ruido**.

Aquí tienes el análisis formal y la justificación para tu informe en formato Markdown:

---

## 🧼 Análisis 6: Calidad de Máscaras y Estrategia de Post-Procesamiento

El análisis de calidad detectó ruido en la abrumadora mayoría de las máscaras, lo que requiere una decisión informada sobre la estrategia de post-procesamiento del modelo.

### 6.1 Resultados Cuantitativos del Análisis de Ruido

| Métrica | Valor | Conclusión e Implicación |
| :--- | :--- | :--- |
| **Total de Máscaras** | 2133 | |
| **Máscaras Ruidosas Detectadas** | **2115** ($99.15\%$) | **ALERTA.** Casi todas las máscaras contienen ruido según los criterios de detección (agujeros O componentes > 1). |
| **Componentes Separados** (Ejemplo) | **1** (Caso Duda: `990.png`) | La gran mayoría de las máscaras tienen **1 componente principal** (la persona), lo que es correcto. El ruido es **principalmente por agujeros**. |
| **Agujeros Detectados** (Ejemplo) | **True** (Caso Duda: `990.png`) | La operación de cierre detectó que casi todas las máscaras contienen **pequeños agujeros internos** o interrupciones. |

### 6.2 Interpretación de la Detección de Ruido

La alta tasa de detección de "ruido" no significa necesariamente que las etiquetas estén rotas, sino que **los contornos de la persona son altamente orgánicos e irregulares**. Los agujeros detectados son probablemente:

1.  Espacios entre los dedos, brazos, o el cabello.
2.  Agujeros intencionales de etiquetado (ej., un anillo o un *piercing* en la máscara).

Dado que el $99.15\%$ de las máscaras tienen este "ruido", esto debe considerarse una **característica intrínseca** de la clase "Persona" y no un error a limpiar previamente.

### 6.3 Justificación de la Estrategia de Post-Procesamiento

La detección masiva de agujeros implica un desafío: el modelo debe aprender a llenar o ignorar estos pequeños vacíos.

| Decisión de Modelado | Justificación de los Resultados |
| :--- | :--- |
| **Post-Procesamiento** | **Obligatorio pero Controlado.** Dado que la tasa de detección es $99.15\%$ (mucho mayor al umbral del $2\%$), el post-procesamiento es esencial, pero su objetivo debe ser solo **suavizar** la predicción. |
| **Técnica Recomendada** | Aplicar una **operación morfológica de CIERRE** (`cv2.MORPH_CLOSE` con un *kernel* pequeño, ej. $3 \times 3$ o $5 \times 5$) a la **predicción binaria final** del modelo. |
| **Impacto en el Modelo** | Esto sirve como una capa de corrección final para **rellenar los pequeños fallos de segmentación** (agujeros) y **eliminar el ruido aislado** (*salt and pepper noise*) que la U-Net podría no haber manejado perfectamente en los bordes. |

---


## 📉 Análisis 7: Espectro de Magnitud Promedio (FFT)

El análisis espectral de la transformada de Fourier revela dónde se concentra la "energía" de las imágenes, lo que nos permite inferir la complejidad de las texturas y los detalles finos.

### 7.1 Interpretación del Espectro


* **Concentración Central Extrema:** La energía está casi en su totalidad concentrada en el **punto central** del espectro. Este centro representa las **bajas frecuencias** (formas grandes, áreas de color uniforme, fondos suaves, etc.).
* **Decaimiento Rápido:** La intensidad de la frecuencia decae muy rápidamente a medida que nos alejamos del centro hacia los bordes. Los **bordes** del espectro representan las **altas frecuencias** (detalles finos, texturas, ruido, bordes agudos).
* **Conclusión:** El *dataset* está dominado por **bajas frecuencias**. Las imágenes, en promedio, **carecen de texturas finas complejas** o tienen fondos muy suaves.

### 7.2 Justificación de Data Augmentation (Textura)

La U-Net segmentará bien las formas grandes, pero podría **sobreajustarse a la suavidad** de las imágenes de entrenamiento y **fallar al encontrar la persona en entornos ruidosos o texturizados** (ej., fotos con grano o ambientes urbanos complejos).

| Decisión de Augmentation | Justificación de los Resultados |
| :--- | :--- |
| **Adición de Ruido** | **Ruido Gaussiano o Ruido de Sal y Pimienta.** Se debe inyectar ruido artificialmente en los datos de entrenamiento. |
| **Impacto en el Modelo** | Esto fuerza al modelo a aprender que la segmentación (el objeto "Persona") es válida **incluso en presencia de variaciones de alta frecuencia** (ruido). El modelo se vuelve **robusto a la textura** y es menos probable que confunda el ruido de alta frecuencia del entorno real con un fallo en la segmentación del borde. |
| **Augmentation Geométrica** | **Rotaciones y Shearing.** La línea cruzada (artefacto de las imágenes rectangulares) es visible, justificando también la necesidad de rotar para hacer el modelo invariante a la orientación. |


---

## 📊 Análisis 8: Ruido/Textura Local 

Este análisis cuantifica la granularidad o textura fina de las imágenes midiendo la desviación estándar promedio en parches locales. Confirma numéricamente las conclusiones extraídas del análisis FFT.

### 8.1 Resultados Cuantitativos del Análisis de Textura

| Métrica | Valor | Conclusión e Implicación |
| :--- | :--- | :--- |
| **Media de $\sigma_n$ (Textura Local)** | **$10.82$** (de 255) | **Valor Muy Bajo.** Una media de 10.82 en una escala de 0-255 indica que las imágenes son, en promedio, **extremadamente suaves** y carecen de textura fina o ruido de sensor. |

### 8.2 Interpretación del Histograma


* **Pico en Valores Bajos:** El histograma muestra una fuerte concentración de imágenes con un $\sigma_n$ entre 5 y 15. Esto confirma que la gran mayoría del *dataset* está compuesto por imágenes "limpias", con fondos desenfocados (*bokeh*) o superficies de piel/ropa suaves.
* **Cola Derecha (Skew):** Aunque la media es baja, la cola se extiende hasta $\approx 40$. Esto indica que existe un subconjunto minoritario de imágenes que sí contienen textura o ruido, pero no son la norma.

### 8.3 Justificación de Data Augmentation (Ruido)

Este análisis, combinado con el FFT (Paso 7), presenta una justificación sólida para la **Augmentation de Ruido**.

| Decisión de Augmentation | Justificación de los Resultados |
| :--- | :--- |
| **Añadir Ruido (Ej. `GaussianNoise`)** | **CRÍTICO.** El valor medio de $\sigma_n=10.82$ es muy bajo, lo que refuerza que el modelo se sobreajustará a imágenes "limpias". El modelo podría confundir el grano de una foto real o la textura de una tela con un borde, resultando en una segmentación ruidosa. |
| **Impacto en el Modelo** | Al añadir ruido artificial (ej. `GaussianNoise`) durante el entrenamiento, **forzamos a la U-Net a aprender la diferencia entre la "textura" (ruido) y la "forma" (bordes reales)**. Esto es vital para la generalización y la robustez del modelo en el *Test Set*. |

---

## 📐 Análisis 9: Compacidad y Complejidad del Borde

Este análisis mide la complejidad geométrica de las máscaras. La **compacidad** (calculada como $\frac{(\text{Perímetro})^2}{4\pi \times \text{Área}}$) cuantifica cuán irregular es un contorno. Un círculo perfecto tiene una compacidad de $1.0$; valores más altos indican formas más complejas.

### 9.1 Resultados Cuantitativos de Compacidad

| Métrica | Valor | Conclusión e Implicación |
| :--- | :--- | :--- |
| **Compacidad Media ($\mu$)** | **$2.83$** | **Valor Significativamente Alto.** Una media de 2.83 (muy lejos del 1.0 de un círculo o 1.27 de un cuadrado) confirma que las máscaras son **geométricamente complejas**. |

### 9.2 Interpretación del Histograma


* **Pico Desplazado:** El pico de la distribución se encuentra alrededor de $2.0-2.5$, no de $1.0$. Esto indica que la "norma" en este *dataset* no son formas simples, sino contornos irregulares.
* **Fuerte Sesgo a la Derecha (Cola Larga):** El histograma tiene una cola larga que se extiende hasta valores $> 16.0$. Esto es causado por poses que maximizan el perímetro en relación con el área (ej. brazos extendidos, piernas separadas, cabello detallado), representando los casos de segmentación más difíciles.

### 9.3 Justificación de la Arquitectura U-Net

Este análisis refuerza la decisión de usar U-Net, tomada en el Paso 4.

| Decisión de Modelado | Justificación de los Resultados |
| :--- | :--- |
| **Arquitectura (U-Net)** | La alta media de compacidad ($2.83$) prueba cuantitativamente que el modelo debe ser capaz de **reconstruir formas irregulares**, no solo "manchas" (blobs). La U-Net es ideal para esto. |
| **Importancia de las *Skip Connections*** | **CRÍTICO.** La alta compacidad se debe a los **detalles de alta frecuencia** (bordes finos). Las *skip connections* son el mecanismo que permite a la U-Net transferir esta información espacial precisa desde el *encoder* al *decoder*, permitiendo la reconstrucción de bordes complejos. |
| **Data Augmentation (Geométrica)** | Para robustecer el modelo contra las formas más extremas (la cola derecha), se justifica el uso de **Elastic Deformation** o **Grid Distortion**, ya que simulan variaciones orgánicas en los contornos. |

---


## 🗺️ Análisis 10: Análisis de Posición (Distancia al Centro)

Este análisis mide el **"sesgo de centrado"** del *dataset* calculando la distancia promedio (en píxeles) desde el centroide de la persona hasta el centro exacto de la imagen ($400, 400$).

### 10.1 Resultados Cuantitativos del Análisis de Centrado

| Métrica | Valor | Conclusión e Implicación |
| :--- | :--- | :--- |
| **Distancia Media ($\mu$)** | **$119.09$ px** | **Sesgo Significativo.** El sujeto no está centrado. En promedio, el centro de la persona está a $\approx 119$ píxeles del centro de la imagen. |
| **Desviación Estándar** | $71.81$ px | **Alta Variabilidad.** La posición del sujeto varía mucho de una imagen a otra. |
| **Distancia Máxima** | $446.90$ px | **Casos Extremos.** Existen sujetos cuyo centroide está a $\approx 447$ píxeles del centro (muy cerca de las esquinas o bordes). |

### 10.2 Interpretación del Histograma


* **Pico Desplazado:** El histograma no está centrado en 0. Muestra un pico claro alrededor de $80-120$ píxeles, confirmando que la mayoría de los sujetos están notablemente descentrados.
* **Cola Larga:** La cola se extiende hasta $\approx 450$ píxeles, lo que indica que el modelo debe ser capaz de segmentar personas que están casi completamente en el borde de la imagen.

### 10.3 Justificación de la Data Augmentation Geométrica

Estos resultados justifican la necesidad de hacer el modelo **invariante a la traslación (posición)**. El modelo no puede asumir que la persona estará siempre en el centro.

| Decisión de Augmentation | Justificación de los Resultados |
| :--- | :--- |
| **Traslación Aleatoria** | **CRÍTICO.** Se deben aplicar **`Random Shifts`** (traslaciones) o **`RandomResizedCrop`** (que maneja tanto escala como traslación). |
| **Impacto en el Modelo** | Esta augmentation simula sujetos en posiciones aún más variadas, forzando a la U-Net a aprender las **características de la "persona"** en lugar de aprender la **"posición de la persona"**. Esto es esencial para la generalización en el *Test Set*. |

