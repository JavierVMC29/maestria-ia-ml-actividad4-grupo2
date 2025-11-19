# 🫀 Predicción de Enfermedad Cardíaca: Un Enfoque Transparente y Ético (XAI)

### 📋 Resumen del Proyecto
Este proyecto implementa un modelo de Machine Learning (Random Forest) para predecir la presencia de enfermedades cardíacas utilizando el dataset **Heart Disease UCI** (combinado).

El objetivo central no es solo maximizar la precisión, sino garantizar la **Calidad del Dato**, la **Transparencia (XAI)** y realizar una **Auditoría Ética** rigurosa para detectar discriminación algorítmica antes de un hipotético despliegue médico.

---

## 1. 📊 Gestión de Calidad de Datos

Se utilizó el dataset combinado que incluye 4 bases de datos (Cleveland, Hungría, Suiza, Long Beach VA) con un total inicial de **920 registros**.

### 1.1 Auditoría y Limpieza Rigurosa
Durante la exploración de datos (EDA), detectamos un problema crítico:
* **El Problema:** Las bases de datos de Hungría, Suiza y Long Beach no registraron consistentemente variables vitales como `ca` (número de vasos mayores coloreados por fluoroscopia) y `thal` (tipo de talasemia). Estas columnas presentaban un **>60% de valores nulos**.
* **La Decisión:** Priorizamos la **integridad clínica** sobre el volumen de datos.
* **La Acción:** Se aplicó un filtrado estricto (`dropna`), descartando los registros incompletos.
    * *Resultado:* El dataset se redujo a **299 pacientes** (principalmente del estándar de oro de Cleveland).
    * *Justificación:* Imputar (inventar) datos complejos como una fluoroscopia o un defecto genético (talasemia) introduciría ruido inaceptable en un modelo de salud, alucinando explicaciones en la fase de XAI.

**Auditoría de Nulos (Antes de la limpieza):**
> ![Gráfico de barras de "Detección de Valores Nulos"](./img/valores-nulos.png)

---

## 2. ⚙️ Metodología y Pre-procesamiento

Para preparar los datos para el modelo Random Forest, aplicamos:

### A. Traducción de Variables (Label Encoding)
El dataset contenía variables categóricas en texto (ej: "Typical Angina", "Male"). Se codificaron numéricamente y se generó un diccionario para mantener la interpretabilidad.

**Diccionario de Mapeo:**
> ![Inserte aquí la captura de la tabla "DICCIONARIO DE CODIFICACIÓN"](./img/codificacion.png)

### B. Simplificación del Objetivo (Target Binarization)
La variable `num` original clasifica la enfermedad del 0 al 4.
* **Transformación:** Simplificamos el problema a **Clasificación Binaria**.
    * `0` = Sano
    * `1, 2, 3, 4` $\rightarrow$ **1 (Enfermo)**
* **Motivo:** Las clases severas (3 y 4) tenían muestras insuficientes, lo que hubiera impedido el aprendizaje del modelo.

**Distribución de Clases:**
> ![Gráficos comparativos de distribución](./img/distribucion-datos.png)

---

## 3. 🤖 Entrenamiento del Modelo

* **Algoritmo:** Random Forest Classifier.
* **Configuración:** 100 árboles, profundidad máxima de 5 (para evitar sobreajuste).
* **Métricas Globales:** El modelo alcanzó una precisión (Accuracy) del 85%, en el set de prueba depurado.

---

## 4. 🔍 Explicabilidad (XAI)

Para validar la lógica médica del algoritmo, utilizamos dos técnicas de Caja Blanca:

### Técnica 1: Permutation Feature Importance
Mide qué tanto cae el rendimiento del modelo si eliminamos la información de una variable.

**Variables Más Importantes:**
> ![Gráfico de Barras Horizontal](./img//permutation-importance.png)

* **Hallazgo:** El modelo depende fuertemente de `cp` (Dolor de pecho), `thal` (Talasemia) y `ca` (Vasos), lo cual coincide con la literatura cardiológica.

### Técnica 2: SHAP (Global y Local)
Analiza el impacto positivo o negativo de cada síntoma.

**Resumen Global (Beeswarm):**
> ![Gráfico de puntos de colores](./img/shap.png)

### Explicación de un Caso (Paciente #0)
Desglosamos la decisión del modelo para el primer paciente del set de prueba.

**Gráfico Waterfall:**
> ![Inserte aquí la captura del gráfico Waterfall](./img/xai-caso-individual.png)

* **Interpretación:** Podemos ver exactamente qué síntomas (barras rojas) empujaron al modelo a diagnosticar enfermedad y qué factores protectores (barras azules) intentaron mitigar el riesgo.

---

## 5. ⚖️ Auditoría Ética y Sesgos

Se evaluó el principio de **Justicia (Fairness)** comparando el rendimiento en **Hombres** vs **Mujeres**.

**Gráfico de Disparidad:**
> ![Inserte aquí la captura del gráfico de barras "Sensibilidad por Género"](./img/disparidad-rendimiento-por-genero.png)

### 🚨 Resultados Críticos:
Los datos revelaron un comportamiento inesperado en este experimento:
1.  **Sensibilidad en Mujeres (1.00):** El modelo detectó el **100%** de los casos de enfermedad en mujeres. No hubo falsos negativos.
2.  **Sensibilidad en Hombres (0.75):** El modelo falló al detectar la enfermedad en el **25%** de los hombres enfermos.
3.  **Conclusión del Sesgo:** Existe una brecha de rendimiento del 25% que penaliza a los hombres. En un entorno hospitalario, este modelo sería peligroso para los pacientes masculinos, ya que 1 de cada 4 podría ser enviado a casa erróneamente sin tratamiento.

---

## 6. 📝 Conclusiones y Recomendaciones

1.  **Calidad sobre Cantidad:** La decisión de descartar el 60% de la data fue correcta para garantizar que las explicaciones (SHAP) se basaran en datos clínicos reales y no imputados.
2.  **Transparencia:** Las herramientas XAI demostraron que el modelo "piensa" correctamente (usa las variables médicas adecuadas), pero eso no garantiza que sea justo.
3.  **Recomendación de No-Despliegue:** A pesar de la buena precisión global, **el modelo no debe pasar a producción**.
    * La disparidad de sensibilidad contra los hombres es éticamente inaceptable.
    * **Próximos pasos:** Se requiere recolectar más datos masculinos de alta calidad o aplicar técnicas de regularización para equilibrar la sensibilidad entre géneros antes de su uso clínico.

---