<div align="center">

# 🧬 Proyecto Final de Bioinformática 2026
## Glioma Grading: clasificación LGG vs. GBM

**Predicción del grado tumoral combinando aprendizaje no supervisado y supervisado sobre cohortes TCGA**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3+-F7931E?logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-FF6F00?logo=tensorflow&logoColor=white)](https://www.tensorflow.org/)
[![UCI](https://img.shields.io/badge/UCI-Dataset%20759-orange)](https://archive.ics.uci.edu/dataset/759/glioma+grading+clinical+and+mutation+features+dataset)

</div>

---

## 📄 Paper cabecera

> Tasci E., Zhuge Y., Kaur H., Camphausen K., Krauze A. V. (2022). **Hierarchical Voting-Based Feature Selection and Ensemble Learning Model Scheme for Glioma Grading with Clinical and Molecular Characteristics**. *Int. J. Mol. Sci.*, 23(22), 14155.
> DOI: [10.3390/ijms232214155](https://doi.org/10.3390/ijms232214155)
> National Cancer Institute (NCI/NIH), EE. UU.

---

## 📊 Dataset

| Campo | Detalle |
|---|---|
| **Nombre** | Glioma Grading Clinical and Mutation Features |
| **Fuente** | [UCI ML Repository, ID 759](https://archive.ics.uci.edu/dataset/759/glioma+grading+clinical+and+mutation+features+dataset) |
| **Origen** | TCGA-LGG + TCGA-GBM (The Cancer Genome Atlas) |
| **Instancias** | 839 pacientes |
| **Features** | 23 originales (20 genes binarios + 3 clínicas) → 25 tras one-hot encoding de `Race` |
| **Target** | `Grade` (0 = LGG, 1 = GBM) |
| **Balance** | 487 LGG (58.05 %) vs. 352 GBM (41.95 %) |
| **Valores faltantes** | 0 (dataset preprocesado) |

---

## 📁 Estructura del repositorio
.  \
├── 📓 Brizuela_Proyecto_Final_Bioinformatica_2026.ipynb  \
├── 📄 dataset.csv  \
├── 📄 paper.pdf  \
├── 📄 README.md  \
└── 📄 GITHUB_URL.txt  \

---

## 🛠 Librerías utilizadas

| Categoría | Librerías |
|---|---|
| Datos | `pandas`, `numpy`, `ucimlrepo` |
| Visualización | `matplotlib`, `seaborn` |
| Modelado | `scikit-learn`, `scipy`, `tensorflow.keras` |

---

## 🚀 Cómo ejecutar

1. Abrir el notebook en [Google Colab](https://colab.research.google.com/).
2. `Runtime → Run all`.
3. El dataset se descarga automáticamente con `fetch_ucirepo(id=759)`. No se requieren archivos locales.

---

## 🔬 Resumen de resultados

### Análisis exploratorio

- **IDH1**: mutado en **78.2 %** de los LGG y solo **6.5 %** de los GBM (Δ = 71.7 pp). El gen más informativo del dataset.
- **Edad**: LGG = 43.87 ± 13.26 años vs. GBM = 60.70 ± 13.43 años. Diferencia de ~17 años.
- **PTEN**: patrón invertido, 33.0 % en GBM vs. 5.1 % en LGG.

### Aprendizaje no supervisado

**PCA** sobre las 25 features estandarizadas:

| Componente | Varianza explicada | Acumulada |
|---|---|---|
| PC1 | 22.31 % | 22.31 % |
| PC2 | 12.32 % | 34.63 % |
| — | 16 PCs para ≥ 90 % | 20 PCs para ≥ 95 % |

Top loadings PC1: **IDH1** (+0.598), **TP53** (+0.512), **ATRX** (+0.483).

**Clustering (ARI vs Grade):**

| Método | ARI |
|---|---|
| 🥇 K-Means (k=2) | **0.496** |
| Jerárquico Ward | 0.403 |
| Jerárquico Average | 0.002 |
| Jerárquico Complete | 0.001 |

K-Means reconstruye buena parte de la división clínica sin usar etiquetas: 326/352 GBM caen en un cluster, 389/487 LGG en el otro.

### Aprendizaje supervisado

Partición estratificada 64 / 16 / 20 (`random_state=42`). `StandardScaler` ajustado **solo sobre train**.

**Resultados en test:**

| Modelo | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| 🥇 **Árbol de Decisión** | **0.8690** | 0.7857 | **0.9429** | **0.8571** | 0.8790 |
| SVM (RBF) | 0.8571 | 0.7805 | 0.9143 | 0.8421 | 0.8939 |
| Red Neuronal (MLP) | 0.8393 | 0.7867 | 0.8429 | 0.8138 | 🥇 **0.9200** |
| Random Forest | 0.8155 | 0.7746 | 0.7857 | 0.7801 | 0.9087 |

**Validación cruzada (5-fold sobre train, F1):**

| Modelo | F1 (5-fold) |
|---|---|
| Árbol de Decisión | **0.8439 ± 0.0273** |
| SVM (RBF) | 0.8165 ± 0.0516 |
| Random Forest | 0.8132 ± 0.0447 |

El test del árbol (F1 = 0.857) cae dentro de 1 desvío estándar del promedio CV. **El rendimiento es estable y no producto de un split favorable.**

---

## 🧠 Genes más discriminantes

| Gen | Patrón | Frecuencia LGG | Frecuencia GBM | Importancia |
|---|---|---|---|---|
| **IDH1** | 🔵 LGG | 78.2 % | 6.5 % | 0.914 (árbol) |
| **Age_at_diagnosis** | 🟠 GBM | — | — | 0.311 (RF) |
| **ATRX** | 🔵 LGG | 37.6 % | 9.7 % | 0.048 (RF) |
| **PTEN** | 🟠 GBM | 5.1 % | 33.0 % | 0.047 (RF) |
| **CIC** | 🔵 LGG | 22.0 % | 1.1 % | 0.049 (RF) |

Estos cinco concentran la mayor parte de la señal. IDH1 + edad ya separan la mayoría de los casos, como demuestra el árbol de profundidad 2.

---

## ⚠️ Limitaciones

- **Tamaño y origen**: 839 pacientes de TCGA, 91 % población blanca. La generalización requiere validación externa.
- **Codificación binaria**: los genes están como mutado / no mutado. Se pierde el tipo específico de mutación (ej. IDH1 R132H).
- **Sin datos de expresión**: solo se usan mutaciones, no niveles de ARN ni metilación.
- **Desbalance moderado**: 58 % LGG / 42 % GBM, manejado con `stratify` pero no eliminado.
- **Comparación con Tasci et al. (2022)**: Ellos reportan 87.6% de accuracy y 0.858 de F1 en TCGA usando un soft-voting ensemble de 3 modelos (SVM + RF + AdaBoost) con selección jerárquica de features. Nuestro árbol de decisión de profundidad 2 alcanza 86.9% de accuracy y 0.857 de F1 en TCGA, una diferencia de 0.7 pp en accuracy y 0.001 en F1. Esto sugiere que la señal molecular de IDH1 e IDH2 captura la mayor parte del poder predictivo del dataset, sin necesidad de ensembles más complejos."

---

## 👤 Autor

**Brizuela Luciano** — Bioinformática 2026
