
## 🧱 **Fases clave del obligatorio (con enfoque profesional)**

### 🧩 1. **EDA (Exploratory Data Analysis) — obligatorio**

Ya lo hablamos, pero acá va el desglose extendido con ideas “plus”:

* 📂 **Verificación de integridad**: contar imágenes, tamaños, correspondencia `image-mask`.
* 📊 **Distribuciones estadísticas**: proporción de píxeles positivos, tamaño relativo de la persona, histogramas de intensidad, canales RGB.
* 🧠 **Variabilidad visual**: comparar ejemplos con fondos simples vs complejos, diferentes poses, iluminaciones, escalas.
* 🎨 **Visualización avanzada**: overlays (`alpha blending`) entre imagen y máscara, grid de ejemplos, mapas de calor del promedio de máscaras.
* 🔁 **Propuesta de data augmentation** basada en tus observaciones (p. ej., si hay fondos homogéneos, aplicar cambios de brillo; si hay cuerpos en distintas posiciones, usar rotaciones).
* 🧾 **Conclusión EDA:** resumen de hallazgos y justificación de preprocessing y augmentations.

---

### ⚙️ 2. **Preprocesamiento y preparación del dataset**

Esta parte debería estar bien fundamentada a partir del EDA:

* Normalización (rango de píxeles, mean/std).
* Creación de `Dataset` y `DataLoader` de PyTorch.
* Aumento de datos: flips, rotaciones, crops, elastic deformation, cambios de color.
* Partición `train/val/test` (o validación cruzada).
* Visualización post-augmentation (para demostrar efectividad).

💡 *Plus opcional:* si el dataset es chico, generar una métrica de “diversidad” visual (por ejemplo, usando embeddings CLIP o PCA para estimar variabilidad entre imágenes).

---

### 🧱 3. **Implementación del modelo**

Aun sin código, ya deberías planificar lo siguiente:

* Arquitectura base: U-Net clásica con `Conv → ReLU → BN → MaxPool` en encoder y `ConvTranspose → ReLU → BN` en decoder.
* Parámetros ajustables: número de filtros iniciales (16, 32, 64), profundidad, dropout, activación (ReLU/LeakyReLU).
* Posibles mejoras:

  * **Residual U-Net** (skip connections dentro de bloques).
  * **Attention U-Net** (para focalizar regiones relevantes).
  * **U-Net++** (para combinar features intermedias).
  * **Pretrained encoder** (ResNet/DenseNet como backbone).

💡 *Plus:* justificar las elecciones con base en papers (por ejemplo: “se incorporó BatchNorm como en Ronneberger et al., 2015 para acelerar la convergencia y estabilizar el gradiente”).

---

### 🧮 4. **Función de pérdida y métrica**

* Base: `DiceLoss`, `BCELoss`, o `BCE + Dice`.
* Justificar por qué: (desbalance de clases, Dice más sensible al solapamiento).
* Calcular métricas adicionales: IoU (Jaccard), Precision, Recall.
* Monitorear *Dice* en validación cada epoch.

💡 *Plus:* hacer un pequeño experimento comparando distintas losses y mostrar cómo cambian las curvas de entrenamiento.

---

### 🚀 5. **Entrenamiento del modelo**

* Definir optimizador (`Adam` recomendado) y scheduler (ReduceLROnPlateau o CosineAnnealing).
* Incluir regularización (dropout, weight decay, early stopping).
* Mostrar evolución de:

  * Training vs validation loss.
  * Dice Coefficient por epoch.
* Guardar el mejor modelo (`torch.save`) según validación.

💡 *Plus:* usar **Weights & Biases (W&B)** para registrar los experimentos — no obligatorio, pero suma puntos y te permite justificar con visualizaciones automáticas.

---

### 📈 6. **Evaluación exhaustiva**

* Métricas globales: Dice, IoU, Precision, Recall, Accuracy.
* Ejemplos visuales: comparar predicciones correctas e incorrectas.
* Mapa de calor de errores (diferencias entre máscara real y predicha).
* Análisis de errores:

  * Casos en que el modelo falla (personas pequeñas, fondos complejos, sombras).
  * Análisis cuantitativo: correlación entre % de píxeles positivos y error.

💡 *Plus:* visualización tipo “confusion map” (TP, FP, FN en colores distintos sobre la imagen).

---

### 🧾 7. **Postprocesamiento**

* Aplicar umbral óptimo (`threshold tuning`) para maximizar Dice en validación.
* Morfología (dilate/erode) para limpiar bordes.
* *Connected components* para eliminar regiones pequeñas de ruido.
* Codificación RLE final (como te pide Kaggle).

💡 *Plus:* explorar *Test-Time Augmentation (TTA)* — promedio de predicciones tras flips/rotaciones.

---

### 🧰 8. **Predicciones y competencia Kaggle**

* Generar CSV final: `id, encoded_pixels`.
* Verificar que el CSV tenga el formato correcto.
* Subir al Kaggle privado y registrar tu score.
* Documentar resultado y análisis comparativo (e.g., “nuestro modelo alcanzó Dice = 0.82 en Kaggle, superando el umbral de 0.75”).

💡 *Plus:* si hacés varias submissions, mostrar una tabla resumen con configuración y score de cada una.

---

### 📘 9. **Informe / Notebook final bien documentado**

* Markdown con narrativa clara y justificaciones.
* Explicación de cada decisión (técnica y empírica).
* Conclusiones generales y futuras mejoras.

💡 *Plus:* cerrar con un breve apartado tipo “Discusión de resultados” o “Trabajo futuro”, donde propongas cómo mejorarías el modelo (por ejemplo, usar Vision Transformers o Data Augmentation adversarial).

---

## 🚦 En resumen

Si querés **ir por un trabajo sólido y destacarte**, deberías tener:

| Etapa                                                             | Prioridad      | Puntos que aporta           |
| ----------------------------------------------------------------- | -------------- | --------------------------- |
| EDA detallado                                                     | 🔥 Alta        | 5 + fortalece justificación |
| Modelo bien implementado                                          | 🔥 Alta        | 20                          |
| Entrenamiento sólido con curvas y métricas                        | 🔥 Alta        | 10                          |
| Evaluación visual + análisis de errores                           | 🔥 Alta        | 10                          |
| Kaggle + documentación clara                                      | 🔥 Alta        | 5                           |
| Experimentos extra (loss, augmentations, mejoras arquitectónicas) | ⭐ Extra credit | Mejora global de nota       |

---
