# 🗂️ Aprendizaje Supervisado: Clasificación y Diagnóstico de Errores

La clasificación es el problema de asignar una entrada a una categoría. En la industria, la clasificación no se trata de "acertar la clase", sino de **gestionar el riesgo del error**.

## 1. Algoritmos de Clasificación: De la Simplicidad a la Robustez

### A. Regresión Logística (El Baseline)
No es una regresión, es un clasificador lineal que utiliza la **Función Sigmoide** para mapear cualquier valor al rango $[0, 1]$.
- **Análisis Técnico**: El resultado es una probabilidad. Si $P(y=1) > 0.5$, clasificamos como 1.
- **Uso Pro**: Ideal para problemas donde necesitamos saber la probabilidad exacta del evento (Ej: Probabilidad de default crediticio).

### B. Árboles de Decisión y Random Forest
Los árboles dividen el espacio de datos en hiper-rectángulos usando la **Entropía** o la **Impureza de Gini**.
- **Random Forest (Ensemble Learning)**: Combina múltiples árboles entrenados en subconjuntos aleatorios de datos (Bagging). 
- **Ventaja**: Reduce drásticamente la varianza del modelo, evitando el sobreajuste.

### C. Gradient Boosting Machines (XGBoost / LightGBM)
El estado del arte para datos tabulares. A diferencia de Random Forest, los árboles se construyen de forma secuencial: cada árbol intenta corregir los errores del árbol anterior.

---

## 2. Evaluación Profesional: Más allá del Accuracy

En el mundo real, el **Accuracy** es una métrica engañosa. Si el 99% de las transacciones son legales, un modelo que diga siempre "Legal" tendrá un 99% de accuracy, pero será inútil porque no detecta ningún fraude.

### La Matriz de Confusión y sus Métricas:
1. **Precisión (Precision)**: $\frac{TP}{TP + FP}$. "De todo lo que marqué como fraude, ¿cuánto fue realmente fraude?". (Evita falsas alarmas).
2. **Sensibilidad (Recall)**: $\frac{TP}{TP + FN}$. "De todos los fraudes reales, ¿cuántos logré detectar?". (Evita dejar pasar el fraude).
3. **F1-Score**: La media armónica entre Precisión y Recall. Es la métrica estándar cuando hay clases desbalanceadas.

### Curvas de Decisión: ROC y AUC
- **Curva ROC**: Muestra la tasa de Verdaderos Positivos vs. Falsos Positivos para diferentes umbrales de decisión.
- **AUC (Area Under the Curve)**: Un valor de $1.0$ es un modelo perfecto; $0.5$ es equivalente a lanzar una moneda.

---

## 3. Implementación Técnica: Clasificación Robusta en Python

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, roc_auc_score, confusion_matrix
from sklearn.model_selection import cross_val_score

# 1. Entrenamiento con Validación Cruzada (K-Fold)
# Evita que el resultado dependa de una división azarosa de los datos
clf = RandomForestClassifier(n_estimators=100, max_depth=10, random_state=42)
cv_scores = cross_val_score(clf, X_train, y_train, cv=5)
print(f"Accuracy promedio: {cv_scores.mean():.2%}")

# 2. Predicción de Probabilidades (No solo la clase)
probs = clf.predict_proba(X_test)[:, 1]
auc = roc_auc_score(y_test, probs)
print(f"AUC-ROC Score: {auc:.2f}")

# 3. Análisis de Errores
print(confusion_matrix(y_test, clf.predict(X_test)))
```

---

## 🎯 Reto de Análisis de Riesgo
**Escenario**: Estás diseñando un modelo para detectar cáncer.
- Un **Falso Positivo** (decir que hay cáncer cuando no hay) causa estrés y una biopsia innecesaria.
- Un **Falso Negativo** (decir que no hay cáncer cuando sí hay) significa que el paciente no recibe tratamiento y puede morir.

**Pregunta**: ¿Cuál métrica debes optimizar? ¿Precisión o Recall?
**Respuesta**: **Recall**. En este caso, es infinitamente más costoso dejar pasar un enfermo que hacer una prueba extra a un sano. Debes minimizar los Falsos Negativos.

#machine-learning #classification #metrics #fraud-detection #scikit-learn
