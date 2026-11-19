---
title: Completion and Translation Queue
type: tracker
tags: [vault, maintenance, todo]
---

# 🚧 Completion and Translation Queue

Status of the vault after the November 2026 reorganisation. Work through this list (or ask Claude to continue with the next block).

## ✅ Done in this reorganisation
- New English **Python track** (16 notes, beginner → advanced) → [[00 - Python Roadmap]]
- **NumPy (6) and pandas (8)** library notes completed with intermediate + advanced sections
- New **Polars**, **DuckDB** and **Python Libraries Map** notes
- New English **SQL track** (12 notes + practice DB + 60 tested exercises) → [[00 - SQL Roadmap]]
- **Databases hub**, PostgreSQL guide, database types → [[00 - Databases Hub]]
- **Git, Linux, Docker** notes completed; new Tools roadmap, dev environment, dbt, Airflow → [[00 - Tools Roadmap]]
- **English system**: 30+ notes filled or created (numbers, pronunciation key, sentence building, vocabulary, verbs, professional English, practice) → [[English Dashboard]]
- 61 notes had **unclosed code blocks** (cut-off text) — all fixed.
- Spanish duplicates moved to `99 - Archive/Spanish Originals`; backup of the original vault in `99 - Archive/_backup_before_reorganization_2026-11-18.tar.gz`.

## 1. Incomplete notes (only beginner section) — complete them
### scikit-learn (8)
- [ ] [[sklearn_api_design]] · [ ] [[sklearn_preprocessing]] · [ ] [[sklearn_linear_logistic_regression]] · [ ] [[sklearn_tree_ensemble_models]]
- [ ] [[sklearn_model_evaluation]] · [ ] [[sklearn_feature_selection]] · [ ] [[sklearn_clustering_unsupervised]] · [ ] [[sklearn_pipelines_deployment]]
### TensorFlow (8)
- [ ] [[tensorflow_tensors_autograd]] · [ ] [[tensorflow_sequential_api]] · [ ] [[tensorflow_functional_api]] · [ ] [[tensorflow_training_callbacks]]
- [ ] [[tensorflow_convolutional_neural_networks]] · [ ] [[tensorflow_recurrent_neural_networks]] · [ ] [[tensorflow_transformers_attention]] · [ ] [[tensorflow_saving_deployment]]
### matplotlib / seaborn (5)
- [ ] [[matplotlib_architecture]] · [ ] [[matplotlib_customization]] · [ ] [[seaborn_statistical_distribution]] · [ ] [[seaborn_categorical_relationships]] · [ ] [[seaborn_multivariate_grids]]
### Mathematics (14)
- [ ] [[LA_01_Vectors_Spaces_and_Norms]] · [ ] [[LA_02_Matrices_and_Linear_Transformations]] · [ ] [[LA_03_Eigenvalues_Eigenvectors_and_Diagonalization]] · [ ] [[LA_04_SVD_and_Principal_Component_Analysis]]
- [ ] [[OPT_01_Convex_Optimization_and_Lagrange_Multipliers]] · [ ] [[OPT_02_Gradient_Descent_Variants_and_Convergence]]
- [ ] [[STAT_01_Probability_Axioms_and_Bayes_Theorem]] · [ ] [[STAT_02_Random_Variables_and_Distributions]] · [ ] [[STAT_03_Maximum_Likelihood_Estimation]] · [ ] [[STAT_04_Hypothesis_Testing_and_AB_Testing]]
- [ ] [[TC_01_Partial_Derivatives_and_The_Gradient]] · [ ] [[TC_02_Jacobian_and_Hessian_Matrices]] · [ ] [[TC_03_Tensor_Algebra_and_Index_Notation]] · [ ] [[TC_04_Matrix_Calculus_for_Neural_Networks]]
### Orchestration (short)
- [ ] [[ORCH_02_Automated_Data_Pipelines_and_Database_Sync]] · [ ] [[ORCH_03_Pro_Error_Handling_and_Self_Hosted_Scaling]]

## 2. Spanish notes to translate into English (content is good, language is not)
| Area | Notes |
|---|---|
| Mathematics (Spanish Notes) | [[Álgebra Lineal I - Vectores]], [[Álgebra Lineal II - Matrices]], [[AL - Espacios Vectoriales]], [[AL - Transformaciones Lineales]], [[AL - Determinantes e Inversas]], [[AL - Eigenvalues y Eigenvectors]], [[AL - SVD]], [[Cálculo y Optimización]], [[CAL - Derivadas]], [[CAL - Derivadas Parciales y Gradiente]], [[CAL - Regla de la Cadena]], [[CAL - Descenzo del Gradiente]], [[CAL - Optimos Locales vs Globales]] |
| Python deep dives | [[Pandas_High_Performance]], [[Polars_High_Performance]], [[SciPy_Statsmodels_Professional]], [[Visualizacion_Interactiva]], [[Web_Scraping_Profesional]] |
| Data analysis | [[Fundamentos del Análisis de Datos]], [[Análisis Exploratorio de Datos]], [[Métricas y KPIs]] |
| Visualization & BI | [[Fundamentos de Visualización]], [[Power BI]], [[DAX_Maestro]], [[Power_Query_M]], [[BI_Optimization_Tools]] |
| Machine learning | [[Introducción al Machine Learning]], [[Regresión]], [[Clasificación]], [[Aprendizaje No Supervisado]] |
| AI & LLMs | [[Fundamentos de IA y Deep Learning]], [[NLP y LLMs]], [[Transformers y LLMs]], [[Prompt Engineering y RAG]] |
| Tools (Spanish deep dives) | [[Git y GitHub]], [[Linux y Terminal]], [[Docker]], [[Docker_for_DS]], [[APIs y Cloud]], [[FastAPI_ML_Deployment]] |
| Other | [[Guía de Proyectos]], [[Guía de Recursos]], [[Data_Storytelling]] |
| English grammar notes | Explanations are partly in Spanish — that is fine for learning; optionally add English summaries. |

## 3. Suggested next blocks
1. scikit-learn completion + English ML notes (merge the 4 Spanish ML notes into the sklearn path).
2. Statistics notes (STAT_01–04) completion — needed for A/B testing and ML.
3. Visualization: complete matplotlib/seaborn + translate Power BI/DAX.
4. Projects: an English "Portfolio Projects" note with 5 end-to-end projects using DataShop.

🏠 [[Home]]
