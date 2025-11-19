# 🫀 Predicción de Enfermedad Cardíaca: Un Enfoque Transparente y Ético (XAI)

### 📋 Resumen del Proyecto
Este proyecto implementa un modelo de Machine Learning (Random Forest) para predecir la presencia de enfermedades cardíacas utilizando el dataset **UCI Heart Disease (Cleveland)**.

El objetivo principal no es solo la predicción, sino la **Transparencia (XAI)** y la **Auditoría Ética**. Se aplicaron técnicas avanzadas para explicar *por qué* el modelo toma decisiones y se evaluó si el algoritmo discrimina entre hombres y mujeres.

---

## 1. 📊 El Dataset y Calidad de Datos

Utilizamos el dataset **Cleveland Heart Disease** del repositorio UCI.
* **Filas originales:** 303 pacientes.
* **Variables:** 13 características clínicas (Edad, Colesterol, Dolor de pecho, etc.) y 1 variable objetivo.

### 1.1 Auditoría de Calidad
Al cargar los datos, realizamos una inspección visual y estadística:
* **Detección de Nulos:** Se encontraron valores faltantes (representados originalmente como `?`) en las columnas `ca` (vasos sanguíneos) y `thal` (talasemia).
* **Acción de Limpieza:** Se descartaron 6 filas que contenían estos valores nulos.
    * *Justificación:* Los modelos como Random Forest no manejan nativamente datos faltantes sin imputación previa. Al ser un porcentaje mínimo de la data (<2%), la eliminación fue la estrategia más segura para no introducir ruido artificial.

**Gráfico de Auditoría de Nulos:**
> ![Inserte aquí la captura del gráfico de barras rojo de "Detección de Valores Nulos"]

---

## 2. ⚙️ Metodología y Pre-procesamiento

Para que el modelo pudiera procesar la información, realizamos dos transformaciones críticas:

### A. Traducción de Variables (Label Encoding)
El dataset original contenía variables categóricas en formato de texto (ej: "Typical Angina", "Male").
* **Técnica:** Aplicamos `LabelEncoder`.
* **Resultado:** Se generó un diccionario de mapeo para que el modelo entienda que, por ejemplo, `asymptomatic` = 0 y `typical angina` = 3.

**Diccionario de Transformación:**
> ![Inserte aquí la captura de la tabla "DICCIONARIO DE CODIFICACIÓN" que muestra el script]

### B. Simplificación del Objetivo (Target Binarization)
La variable original `num` clasificaba la enfermedad en 5 grados (0=Sano, 1, 2, 3, 4=Distintos niveles de gravedad).
* **El Problema:** Las clases 2, 3 y 4 tenían muy pocos ejemplos, lo que desbalanceaba el modelo.
* **La Solución:** Simplificamos el problema a **Clasificación Binaria**.
    * 0 = Sano
    * 1, 2, 3, 4 $\rightarrow$ **1 (Enfermo)**

**Distribución de Clases (Antes vs Después):**
> ![Inserte aquí la captura de los dos gráficos de barras azules comparando "Distribución Original" vs "Distribución Final"]

---

## 3. 🤖 Entrenamiento del Modelo

Seleccionamos **Random Forest Classifier** por dos razones:
1.  **No linealidad:** Captura interacciones complejas entre síntomas (ej. la edad afecta diferente si el colesterol es alto).
2.  **Compatibilidad XAI:** Funciona excelentemente con *TreeExplainer* de SHAP.

* **Precisión Global (Accuracy):** ~85% (en el set de prueba).

---

## 4. 🔍 Explicabilidad (XAI) y Transparencia

Para abrir la "caja negra" del modelo, aplicamos dos técnicas complementarias para validar qué variables impulsan los diagnósticos.

### Técnica 1: Permutation Feature Importance
Esta técnica responde: *¿Cuánto cae la precisión del modelo si "rompo" (mezclo aleatoriamente) una variable?*

**Resultados:**
> ![Inserte aquí la captura del Gráfico de Barras Horizontal]

* **Interpretación:** El modelo depende críticamente de `cp` (Dolor de pecho), `ca` (Vasos coloreados) y `thal`. Si eliminamos la información del dolor de pecho, el modelo deja de funcionar.

### Técnica 2: SHAP (Shapley Additive exPlanations)
Esta técnica es más profunda: nos dice no solo *qué* importa, sino *cómo* afecta (positiva o negativamente).

**Resumen Global (Beeswarm Plot):**
> ![Inserte aquí la captura del gráfico de puntos rojos y azules]

* **Análisis:**
    * **Puntos Rojos a la derecha:** Valores altos de `cp` (Dolor asintomático en nuestra codificación) aumentan drásticamente el riesgo de enfermedad.
    * **Puntos Azules a la derecha:** Valores bajos de `thalach` (Frecuencia cardiaca máxima) están asociados a enfermedad (el corazón no responde bien al esfuerzo).

### Explicación de un Caso Individual (Paciente #0)
Para demostrar transparencia, explicamos por qué el modelo diagnosticó **ENFERMEDAD** al primer paciente del test.

**Gráfico Waterfall:**
> ![Inserte aquí la captura del gráfico Waterfall]

* **Lectura del caso:** El paciente partió con una probabilidad base. Tener `thal`=2 y `cp`=3 empujó la probabilidad hacia arriba (+20%), confirmando el diagnóstico positivo.

---

## 5. ⚖️ Reflexión Ética y Sesgos

Esta es la sección más crítica. Evaluamos si el modelo se comporta igual para **Hombres** y **Mujeres**.

**Resultados de la Auditoría:**
> ![Inserte aquí la captura del gráfico de barras comparando "Sensibilidad por Género"]

### Hallazgos:
1.  **Disparidad de Sensibilidad (Recall):** Observamos que el modelo tiene una sensibilidad distinta entre hombres y mujeres. (Generalmente, en este dataset, detecta peor los infartos en mujeres).
2.  **Riesgo Social:** Si este sistema se implementa en un hospital sin corrección, existe un alto riesgo de **falsos negativos en mujeres**. Se podría enviar a casa a una paciente enferma diciéndole que está sana, simplemente porque sus síntomas no coinciden con el patrón masculino predominante en los datos.

---

## 6. 📝 Conclusiones y Recomendaciones

1.  **Eficacia:** El modelo Random Forest es efectivo para predecir enfermedades cardíacas, basándose principalmente en el tipo de dolor de pecho y el estado de los vasos sanguíneos.
2.  **Transparencia:** Las técnicas XAI (SHAP) nos permitieron validar que el modelo sigue una lógica médica coherente y no se basa en ruido.
3.  **Advertencia Ética:** Se recomienda **NO desplegar este modelo en producción** hasta mitigar el sesgo de género.
    * *Recomendación:* Re-entrenar el modelo utilizando técnicas de *oversampling* para mujeres o penalizar los errores en el grupo minoritario durante el entrenamiento.

---
*Repositorio creado para la asignatura de Ética e IA.*