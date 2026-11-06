# 🗂️ Aprendizaje Supervisado: Clasificación, Ensambles y Gestión de Riesgo | Supervised Learning: Classification, Ensembles & Risk Management

**ES:** En la industria, la clasificación no se trata simplemente de "acertar la clase". Se trata de la **gestión del riesgo**. Un error en un modelo de recomendación de películas es trivial; un error en un modelo de diagnóstico médico o de fraude bancario puede ser catastrófico. Un experto no optimiza el "Accuracy", optimiza el **Costo del Error**.

**EN:** In the industry, classification is not simply about "guessing the class". It is about **risk management**. An error in a movie recommendation model is trivial; an error in a medical diagnosis or bank fraud model can be catastrophic. An expert does not optimize "Accuracy"; they optimize the **Cost of Error**.

---

## 1. The Evolution of Classifiers: From Linear to Complex | Evolución de Clasificadores: De lo Lineal a lo Complejo

### A. Logistic Regression: The Probabilistic Baseline
No es una regresión, es un clasificador lineal. Utiliza la **Función Sigmoide** para mapear la salida a un rango $[0, 1]$.
- **Interpretación Experta:** El resultado es una probabilidad. La frontera de decisión es un hiperplano.
- **Uso Pro:** Es el modelo base (*baseline*). Si un modelo complejo (como una Red Neuronal) no supera significativamente a la Regresión Logística, el modelo complejo es innecesario.

### B. Decision Trees & The Entropy Problem
Los árboles dividen el espacio en hiper-rectángulos.
- **Criterios de División:** El algoritmo busca la división que maximice la **Ganancia de Información** (reduciendo la Entropía o la Impureza de Gini).
- **El Gran Problema:** Los árboles tienden al **Overfitting** masivo (alta varianza). Memorizan el ruido del dataset.

### C. Ensemble Learning: The Power of the Crowd | Aprendizaje por Ensambles
Para solucionar la inestabilidad de los árboles, usamos el **Ensemble Learning**:

1. **Bagging (Bootstrap Aggregating) $\rightarrow$ Random Forest:**
   - Entrena muchos árboles en paralelo sobre subconjuntos aleatorios de datos.
   - **Matemática:** Promedia los resultados. Esto **reduce la Varianza** sin aumentar el Sesgo.
2. **Boosting $\rightarrow$ XGBoost, LightGBM, CatBoost:**
   - Entrena árboles de forma secuencial. Cada nuevo árbol se enfoca en los **residuos (errores)** del anterior.
   - **Matemática:** Es un descenso de gradiente en el espacio de las funciones. **Reduce el Sesgo** y la Varianza. Es el estándar actual para datos tabulares.

---

## 2. Risk Management: Beyond Accuracy | Gestión de Riesgo: Más allá del Accuracy

El **Accuracy** es una métrica engañosa en clases desbalanceadas. Un experto utiliza la **Matriz de Confusión** para calcular el costo real.

### A. Precision vs Recall: El dilema del costo
- **Precision (Precisión):** $\frac{TP}{TP + FP}$. "Si el modelo dice que es fraude, ¿qué tan seguro está?". (Optimiza para evitar Falsos Positivos).
- **Recall (Sensibilidad):** $\frac{TP}{TP + FN}$. "¿Cuántos de todos los fraudes reales detectamos?". (Optimiza para evitar Falsos Negativos).

### B. Decision Thresholding (Ajuste del Umbral)
El modelo devuelve una probabilidad (ej. 0.6). Por defecto, el umbral es 0.5.
- **Escenario A (Baja Tolerancia al Riesgo):** En cáncer, preferimos un **Recall alto**. Bajamos el umbral a 0.2 $\rightarrow$ detectamos más enfermos, aunque aumenten los falsos positivos.
- **Escenario B (Alta Tolerancia al Riesgo):** En spam de correo, preferimos una **Precisión alta**. Subimos el umbral a 0.9 $\rightarrow$ solo movemos a spam lo que estamos seguros que es spam.

---

## 3. Model Robustness & Validation | Robustez y Validación del Modelo

Para asegurar que el modelo no colapse en producción, aplicamos técnicas de validación avanzada.

### A. Stratified K-Fold Cross Validation
En datasets desbalanceados (ej. 1% fraude), el K-Fold normal podría dejar un pliegue sin ningún caso de fraude. El **Stratified K-Fold** asegura que cada pliegue mantenga la misma proporción de clases que el dataset original.

### B. The ROC-AUC Curve
La curva ROC grafica la Tasa de Verdaderos Positivos vs la Tasa de Falsos Positivos.
- **AUC (Area Under the Curve):** Mide la capacidad del modelo para **separar las clases**. Un AUC de 0.9 significa que hay un 90% de probabilidad de que el modelo clasifique correctamente un ejemplo positivo aleatorio por encima de uno negativo.

---

## 💻 Professional Implementation: XGBoost with Threshold Tuning

```python
import numpy as np
from xgboost import XGBClassifier
from sklearn.metrics import precision_recall_curve, classification_report
from sklearn.model_selection import train_test_split

# 1. Entrenamiento de un modelo de alta performance
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y)
model = XGBClassifier(n_estimators=100, learning_rate=0.05, max_depth=6)
model.fit(X_train, y_train)

# 2. Obtener probabilidades en lugar de clases
probs = model.predict_proba(X_test)[:, 1]

# 3. AJUSTE DEL UMBRAL (Decision Threshold)
# Supongamos que el costo de un Falso Negativo es 10x el de un Falso Positivo
threshold = 0.3 # Bajamos el umbral para aumentar el Recall
custom_preds = (probs >= threshold).astype(int)

print(classification_report(y_test, custom_preds))
```

#machine-learning #classification #xgboost #risk-management #precision-recall #scikit-learn
