# Trabajo Práctico N.º 3 — Detección del logotipo

**Universidad de Buenos Aires — FIUBA — LSE**  
**Asignatura:** Visión por Computadora  
**Cohorte:** 25 — 3.er bimestre de 2026

## Integrantes

- a2413 — César Hernán Ruggeri
- a2512 — Armando Tomás Civini
- a2521 — Andrea Tatiana Duran
- a2525 — Pablo David Gorosito
- a2542 — Federico Tombesi

## Descripción

Este trabajo desarrolla y valida una solución para detectar el logotipo de Coca-Cola en las imágenes provistas para el TP3. La propuesta contempla cambios de tamaño, perspectiva, contraste y apariencia, además de escenas con varias apariciones simultáneas del logo.

La resolución combina dos estrategias complementarias:

- características locales SIFT, matching de descriptores y validación geométrica mediante RANSAC para imágenes con una aparición del logo;
- template matching multiescala, comparación por intensidades y bordes, máximos locales y Non-Maximum Suppression (NMS) para escenas con múltiples apariciones.

## Consignas abordadas

1. Obtener una detección del logotipo en cada imagen individual, sin falsos positivos.
2. Plantear y validar un algoritmo de detección múltiple sobre `coca_multi.png`, utilizando el mismo template.
3. Generalizar el procedimiento al conjunto completo y mostrar cada detección mediante un bounding box y un nivel de confianza.

## Metodología

### Ítem 1 — Detección de una instancia

El detector de una instancia aplica el siguiente procedimiento:

1. conversión de la imagen a escala de grises y mejora local del contraste mediante CLAHE;
2. construcción de variantes directa e invertida del template;
3. extracción de puntos y descriptores SIFT;
4. correspondencia de descriptores mediante FLANN y BFMatcher;
5. filtrado con el ratio test de Lowe;
6. estimación de una homografía con RANSAC;
7. validación del polígono proyectado según inliers, convexidad, área y ubicación;
8. selección de la detección con mayor consistencia geométrica.

Este enfoque permite absorber variaciones de escala, perspectiva y apariencia que el template matching directo no resuelve de manera estable.

### Ítem 2 — Detección de múltiples instancias

Para `coca_multi.png` se utiliza template matching multiescala. En cada escala se combinan:

- similitud por intensidades;
- similitud sobre bordes Canny;
- detección de máximos locales;
- control de densidad de bordes;
- NMS basado en Intersection over Union (IoU), para eliminar detecciones duplicadas.

Cada resultado informa el bounding box, el score combinado, la escala y la densidad de bordes.

### Ítem 3 — Detector general híbrido

El detector general ejecuta primero la búsqueda multiescala y analiza la cantidad de candidatos, su confianza mediana y su escala mediana. Si el conjunto es compatible con una escena de múltiples apariciones, conserva esas detecciones. En caso contrario, utiliza como respaldo el detector SIFT y RANSAC de una instancia.

La regla se aplica automáticamente a todas las imágenes y no depende del nombre del archivo.

## Resultados

Las salidas guardadas en la notebook muestran los siguientes resultados:

| Evaluación | Resultado | Método seleccionado |
| --- | ---: | --- |
| Imágenes individuales del ítem 1 | 6 de 6 con una detección | SIFT + RANSAC |
| `coca_multi.png` | 16 detecciones | Template matching multiescala |
| Conjunto completo del ítem 3 | 7 de 7 imágenes con detección | Detector híbrido |
| Total del ítem 3 | 22 detecciones | 16 múltiples + 6 individuales |

La inspección visual de los resultados permite comprobar que los bounding boxes se ubican sobre las apariciones del logotipo y que no se observan falsos positivos en las detecciones finales presentadas.

## Interpretación de la confianza

La confianza informada depende del método:

- en SIFT y RANSAC representa una medida de consistencia geométrica basada en la cantidad y proporción de inliers;
- en template matching multiescala corresponde al score combinado de similitud por intensidades y bordes.

Estos valores permiten describir y ordenar resultados dentro de cada método. No representan probabilidades estadísticas y no deben compararse directamente entre métodos.

## Organización de la notebook

La notebook está estructurada en las siguientes secciones:

1. preparación del entorno y carga de datos;
2. construcción del detector robusto;
3. funciones para detección, validación y visualización;
4. resolución y evaluación del ítem 1;
5. resolución, diagnóstico y evaluación del ítem 2;
6. generalización híbrida del ítem 3;
7. resumen y conclusiones.

Además de las visualizaciones, se generan tablas con las métricas y los datos de cada detección para facilitar la revisión de los resultados.

## Requisitos

- Python 3.9 o superior
- OpenCV
- NumPy
- Matplotlib
- pandas
- Git, únicamente si el repositorio de la materia todavía no fue descargado

## Ejecución

1. Abrir `TP3_versión beta.ipynb` en Jupyter Notebook, JupyterLab o Google Colab.
2. Ejecutar las celdas en orden desde el inicio.
3. En la primera ejecución, la notebook descarga automáticamente el repositorio de la materia si no encuentra la carpeta `vision_computadora_I`.
4. Verificar las figuras y tablas generadas para los tres ítems.

La notebook valida la existencia del template y de las imágenes antes de comenzar el procesamiento, por lo que informa de manera explícita cualquier archivo faltante.

## Conclusión

Los experimentos muestran que no existe un único método igualmente adecuado para todas las imágenes del conjunto. SIFT y RANSAC ofrecen mayor robustez frente a cambios de escala, perspectiva y apariencia, mientras que el template matching multiescala resulta más apropiado para localizar numerosas repeticiones similares dentro de una misma escena.

La integración de ambos enfoques permite responder las tres consignas con una regla general, resultados visualizables y métricas que hacen trazable cada detección.
