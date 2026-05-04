# Sistema Predictivo de Desempeño Académico y Permanencia Estudiantil – UNAH
 
> **Proyecto de investigación aplicada** | Universidad Nacional Autónoma de Honduras (UNAH)  
> **Período analizado:** II PAC 2025 (junio–julio 2025)  
> **Autor:** Christian Alexis Manzanares Cruz · [ORCID 0009-0004-7419-0449](https://orcid.org/0009-0004-7419-0449)  
> **Afiliación:** Doctorado en Ciencias del Desarrollo Humano, UNAH  
> **Correo institucional:** christian.manzanares@unah.edu.hn
 
---
 
## Descripción general
 
Este repositorio contiene el sistema analítico integral desarrollado para modelar, explicar y predecir el **desempeño académico** y la **permanencia estudiantil** en la UNAH. El proyecto integra modelos de respuesta ordinal, algoritmos de aprendizaje automático, segmentación no supervisada y un simulador predictivo interactivo, con enfoque en interpretabilidad y aplicabilidad en políticas universitarias.
 
El análisis incorpora dimensiones multidisciplinares: académica, demográfica, socioeconómica, **seguridad alimentaria y nutricional (SAN)**, salud física y mental, recursos tecnológicos y migración.
 
---
 
## Tabla de contenidos
 
- [Objetivo](#objetivo)
- [Datos](#datos)
- [Metodología](#metodología)
- [Modelos y métricas](#modelos-y-métricas)
- [Variables predictoras](#variables-predictoras)
- [Segmentación de estudiantes](#segmentación-de-estudiantes)
- [Entregables](#entregables)
- [Requisitos técnicos](#requisitos-técnicos)
- [Uso](#uso)
- [Hallazgos clave](#hallazgos-clave)
- [Citación](#citación)
- [Licencia](#licencia)
---
 
## Objetivo
 
Desarrollar un sistema analítico que permita:
 
1. **Explicar** el desempeño estudiantil a través de variables académicas, socioeconómicas, de salud y SAN.
2. **Predecir** niveles ordinales de motivación, probabilidad de graduación y riesgo de abandono.
3. **Identificar** estudiantes en riesgo de manera temprana.
4. **Simular** escenarios individuales mediante un simulador predictivo interactivo.
5. **Segmentar** la población estudiantil en perfiles accionables para intervención institucional.
6. **Apoyar** la toma de decisiones basadas en evidencia en bienestar universitario.
---
 
## Datos
 
| Característica | Detalle |
|---|---|
| Fuente | Encuesta institucional UNAH – II PAC 2025 |
| Formato original | SPSS (.sav) con etiquetas de valor |
| Registros totales | 40,994 estudiantes × 309 variables |
| Filtros aplicados | `Estado = "SÍ II PAC"` · `Estado_duplicado = "ORIGINAL"` |
| Muestra analizada | **32,096 estudiantes** (−21.7 % tras filtrado) |
| Dataset limpio final | 32,096 filas × 204 columnas (sin valores faltantes) |
 
### Dimensiones del dataset
 
| Dimensión | Variables | Prefijo SPSS |
|---|---|---|
| Salud física | 72 | `d_ds_` · `v_d_ds_` |
| Migración | 39 | `d_m_` · `v_d_m_` |
| Demográfica | 22 | `b_dsd_` · `v_b_dsd_` |
| Educación previa | 18 | `d_de_` · `v_d_de_` |
| Seguridad Alimentaria (SAN) | 16 | `d_san_` · `v_d_san_` |
| Académica | 13 | `a_dg_` · `v_a_dg_` |
| Socioeconómica | 11 | `c_dse_` · `v_c_dse_` |
| Recursos tecnológicos | 5 | `d_rec_` |
| Salud mental | 2 | (depresión, ansiedad) |
| Índice SAN compuesto | 2 | `v_IDX_SAN_*` |
| Variables objetivo (target) | 3 | — |
 
> **Nota:** El dataset institucional no se redistribuye en este repositorio. Los notebooks están diseñados para ejecutarse con acceso al archivo `00.data.sav` provisto por la UNAH.
 
---
 
## Metodología
 
### Preparación de datos
 
- Filtrado obligatorio por estado de matrícula y registros originales.
- Eliminación de variables con >40 % de valores faltantes, dummies hiper-granulares (municipio, campus) y variables sensibles o redundantes.
- **Imputación múltiple** mediante `IterativeImputer` con `ExtraTreesRegressor` (5 iteraciones, equivalente a MICE).
- Construcción del **Índice SAN compuesto** (`v_IDX_SAN_inseguridad`, rango 0–6) y categorías: Segura, Leve, Moderada, Severa.
- Recodificación de targets:
  - `y_motivacion`: 4 niveles ordinales (Baja colapsada 1+2, Media, Alta, Muy alta).
  - `y_graduacion`: 3 niveles ordinales (No/pesimista, No sé, Sí/optimista).
  - `y_abandono`: binaria (0 = Sin riesgo, 1 = En riesgo).
### Selección de variables
 
Proceso en tres etapas:
1. **Tamizaje univariado:** Cramér's V (top-30 por asociación con cada target).
2. **Control de multicolinealidad:** VIF < 5 (VIF máximo final = 3.91).
3. **Selección multivariada:** Lasso + Mutual Information → 15 variables finales.
### Estrategia de modelado
 
| Tipo | Modelos |
|---|---|
| Ordinales principales | Regresión Logística Ordinal (Ordered Logit) · Ordered Probit |
| Comparativos / ML | Random Forest · Gradient Boosting (HistGradientBoosting) · XGBoost |
| Importancia de variables | SHAP values (modelos de árbol) · Odds Ratios con IC 95 % |
| Simulador | Ensemble 50 % · Logit + 50 % · Árbol destilado |
 
### Segmentación
 
- Algoritmo: K-Means (k = 4).
- Validación: método del codo + coeficiente Silhouette.
---
 
## Modelos y métricas
 
### Modelos ordinales / binarios en conjunto de prueba (n = 6,420)
 
| Target | Mejor modelo | Accuracy | Acc ±1 | Kappa ponderado | MAE ordinal | AUC |
|---|---|---|---|---|---|---|
| Motivación | Ordered Probit | 60.1 % | 88.3 % | 0.053 | 0.54 | — |
| Probabilidad graduación | Ordered Logit ⭐ | 49.1 % | 86.2 % | 0.394 | 0.65 | — |
| Abandono estudiantil | Probit binario ⭐ | 74.2 % | — | 0.197 | — | 0.731 |
 
### Modelos ML comparativos (mejores por target)
 
| Target | Modelo | Accuracy |
|---|---|---|
| Motivación | Random Forest | 60 % |
| Probabilidad graduación | XGBoost | 53 % · Acc±1 = 89 % |
| Abandono estudiantil | XGBoost | AUC = 0.74 · Brier = 0.174 |
 
> El supuesto de proporcionalidad de odds (test de Brant aproximado) **se cumple** en ambos modelos ordinales (0/15 variables con violación a p < 0.05).
 
### División de datos
 
- **Entrenamiento:** 25,676 registros (80 %).
- **Prueba:** 6,420 registros (20 %).
- Estratificación por distribución de la variable `y_motivacion`.
---
 
## Variables predictoras
 
Las 15 variables finales seleccionadas (ordenadas por importancia consolidada SHAP + Odds Ratio):
 
| N.° | Variable | Dimensión |
|---|---|---|
| 1 | `a_dg_anios_estudio` | Académica |
| 2 | `a_dg_porcentaje_Asistencia` | Académica |
| 3 | `a_dg_catidad_clasesAprobadas` | Académica |
| 4 | `d_m_expectativas_migracion` | Migración |
| 5 | `c_dse_desea_trabajar` | Socioeconómica |
| 6 | `v_b_dsd_edad` | Demográfica |
| 7 | `d_rec_calidad_conexion_internet_unah` | Recursos TEC |
| 8 | `d_rec_calidad_conexion_internet_casa` | Recursos TEC |
| 9 | `a_dg_indice_academico` | Académica |
| 10 | `d_san_reduccion_cantidad_comidas` | SAN |
| 11 | `a_dg_indice_global` | Académica |
| 12 | `d_de_anio_graduacion` | Educación previa |
| 13 | `c_dse_financiamiento_estudios` | Socioeconómica |
| 14 | `v_d_m_razones_emigrar_bajos_ingre` | Migración |
| 15 | `v_d_ds_tipos_de_enfermedades_depresion` | Salud mental |
 
---
 
## Segmentación de estudiantes
 
K-Means con k = 4 perfiles accionables:
 
| Clúster | Nombre | Proporción | Descripción |
|---|---|---|---|
| C0 | Alto Riesgo | 1.6 % | Condiciones críticas en SAN, salud mental, asistencia y financiamiento |
| C2 | Riesgo Socioeconómico | 34.6 % | Vulnerabilidad económica y migratoria con desempeño medio-bajo |
| C3 | Estable | 54.8 % | Perfil mayoritario con condiciones aceptables y riesgo moderado |
| C1 | Alto Desempeño | 9.0 % | Indicadores sólidos en todas las dimensiones, bajo riesgo de abandono |
 
---
 
## Entregables
 
| Archivo | Descripción |
|---|---|
| `modeloBienestarUNAH_didactico.ipynb` | Notebook con narrativa paso a paso, visualizaciones matplotlib y explicaciones metodológicas |
| `modeloBienestarUNAH_tecnico.ipynb` | Versión compacta orientada a revisión técnica y reproducibilidad |
| `modeloBienestarUNAH_ejecutable.ipynb` | Pipeline end-to-end ejecutable desde `00.data.sav` |
| `dashboard_modelo_bienestar.html` | Dashboard interactivo (8 secciones, simulador ensemble, filtros dinámicos) |
| `eda_bienestar_unah.html` | Análisis exploratorio dirigido por variables objetivo |
| `ReporteTecnico_BienestarUNAH.docx` | Reporte técnico narrativo con OR, IC 95 %, clusters y recomendaciones |
| `ArticuloIIS_BienestarUNAH_Manzanares_V2.docx` | Artículo científico para la Revista del IIS-UNAH (ISSN 2411-7358) |
| `diccionario_final.csv` | Diccionario completo de las 309 variables estructurado por dimensión |
 
---
 
## Requisitos técnicos
 
### Python (≥ 3.10)
 
```
pandas
numpy
scipy
pyreadstat          # lectura de archivos .sav
scikit-learn        # modelos ML, clustering, imputación
statsmodels         # modelos ordinales (Logit, Probit)
xgboost
shap
matplotlib
seaborn
plotly
joblib
```
 
Instalación rápida:
 
```bash
pip install pandas numpy scipy pyreadstat scikit-learn statsmodels xgboost shap matplotlib seaborn plotly joblib
```
 
### Dashboard
 
El archivo `dashboard_modelo_bienestar.html` es **autocontenido** (no requiere servidor ni dependencias externas). Ábralo directamente en cualquier navegador moderno (Chrome, Firefox, Edge).
 
---
 
## Uso
 
### 1. Ejecutar el pipeline completo
 
```bash
jupyter notebook modeloBienestarUNAH_ejecutable.ipynb
```
 
Asegúrese de que `00.data.sav` se encuentre en el mismo directorio.
 
### 2. Explorar el análisis paso a paso
 
```bash
jupyter notebook modeloBienestarUNAH_didactico.ipynb
```
 
### 3. Consultar el dashboard interactivo
 
Abra `dashboard_modelo_bienestar.html` en su navegador. El simulador permite ingresar valores individuales de las 15 variables y obtener:
 
- Distribución de probabilidades por categoría ordinal.
- Categoría más probable para cada variable objetivo.
- Nivel de riesgo interpretado en lenguaje natural.
---
 
## Hallazgos clave
 
- El **índice SAN compuesto** aparece consistentemente entre los 10 predictores más importantes de los tres targets, validando la integración de la dimensión alimentaria en modelos de permanencia estudiantil.
- Las **expectativas migratorias** funcionan como proxy de malestar socioeconómico y se asocian significativamente con motivación, graduación y abandono.
- La **depresión** (diagnóstico autoreportado) es el predictor de salud mental con mayor peso en los modelos, especialmente para el abandono.
- La **asistencia a clases** y los **años de estudio** son los predictores académicos con mayor poder discriminante.
- El **Clúster C0 (Alto Riesgo)**, aunque minoritario (1.6 %), concentra los perfiles más críticos en SAN severa, salud mental deteriorada y alta probabilidad de abandono, lo que lo convierte en la prioridad de intervención institucional.
---
 
## Citación
 
Si utiliza este trabajo en publicaciones académicas o institucionales, cite de la siguiente manera:
 
**APA 7.ª edición:**
 
> Manzanares Cruz, C. A. (2026). *Sistema predictivo de desempeño académico y permanencia estudiantil – UNAH: Modelos ordinales, segmentación y simulador interactivo* [Software y datos de investigación]. Universidad Nacional Autónoma de Honduras. https://camc30393.github.io/bienestar_simulador_unah/#san 
 
**BibTeX:**
 
```bibtex
@misc{manzanares2026bienestar,
  author       = {Manzanares Cruz, Christian Alexis},
  title        = {Sistema Predictivo de Desempeño Académico y Permanencia Estudiantil – UNAH},
  year         = {2026},
  institution  = {Universidad Nacional Autónoma de Honduras},
  note         = {Software y datos de investigación},
  url          = {https://camc30393.github.io/bienestar_simulador_unah/#san},
  orcid        = {0009-0004-7419-0449}
}
```
 
---
 
## Declaración sobre uso de inteligencia artificial
 
En el desarrollo de los artefactos analíticos, notebooks y dashboard de este proyecto se utilizó asistencia de inteligencia artificial generativa (Deepseek V3) como herramienta de apoyo en la redacción de código, revisión metodológica y estructuración de resultados. La interpretación analítica, las decisiones metodológicas y la validación de hallazgos son responsabilidad exclusiva del autor.
 
---
 
## Licencia
 
Este proyecto está licenciado bajo la [Licencia MIT](LICENSE).
 
Los datos institucionales subyacentes son propiedad de la Universidad Nacional Autónoma de Honduras (UNAH) y no pueden redistribuirse sin autorización institucional expresa.
 
---
 
<div align="center">
**Christian Alexis Manzanares Cruz**  
Doctorado en Ciencias del Desarrollo Humano · UNAH  
[ORCID: 0009-0004-7419-0449](https://orcid.org/0009-0004-7419-0449) · christian.manzanares@unah.edu.hn
 
</div>
