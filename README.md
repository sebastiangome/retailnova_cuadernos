# RetailNova — Predicción de abandono de clientes

SCY1102 Gestión de Proyectos de Datos · Sección 001D · DuocUC
Equipo: Luis Hurtubia, Rafael Pacheco, Sebastián Curi, Sebastián Gómez.

Caso 1: RetailNova pierde ~24 % de sus clientes frecuentes. El proyecto está organizado en **cuadernos Jupyter numerados** (ya ejecutados, con sus resultados y gráficos), apoyados en módulos de `src/` con pruebas automáticas. No hay dashboard HTML: el panel de resultados está dentro del cuaderno 06.

> Datos 100 % sintéticos. Las probabilidades no son riesgo real calibrado.

## Cuadernos (ejecutar en orden desde la carpeta `notebooks/`)

| Cuaderno | Contenido |
|---|---|
| `01_generacion_y_exploracion` | Base sintética de 3 años (185.000), muestra de 12 meses y EDA con 5 gráficos justificados |
| `02_preparacion_de_datos` | Nulos, coherencia temporal, imputación, validaciones formales, RFM y fuga de datos |
| `03_modelado` | Validación cruzada de 5 folds, elección de modelo, umbral OOF |
| `04_evaluacion_y_equidad` | Evaluación única en prueba y auditoría regional RNF6 |
| `05_explicabilidad_shap` | SHAP global y ficha de un cliente |
| `06_scoring_y_panel` | Scoring, niveles de riesgo, KPIs, gráficos, top 20, resultados y seguimiento del proyecto |

## Uso

```bash
pip install -r requirements.txt
jupyter lab                          # abrir y ejecutar notebooks/01 … 06 en orden
# alternativa por consola (sin cuadernos):
python main.py --proceso completo
python -m pytest
```

Los CSV y el modelo no se versionan: se regeneran al ejecutar los cuadernos (semilla 42).

## Decisiones metodológicas

- **Fuga de datos:** `recencia_dias` define el abandono (> 180 días), por lo que no entra como predictora (hay una guarda en el código y una prueba).
- **Validación cruzada** de 5 folds estratificados solo sobre entrenamiento; si la diferencia de AUC es < 0,01 se prefiere Regresión Logística.
- **Umbral** por predicciones out-of-fold: el más alto con recall ≥ 0,75. El test se evalúa una sola vez.
- **RNF6:** brecha de F1 por región vs. F1 general < 10 %.
- **Riesgo:** percentiles 66 y 90 de la probabilidad → Bajo / Medio / Alto.

## Resultados (semilla 42)

| Métrica | Valor |
|---|---|
| Clientes (12 meses) | 51.346 · abandono 24,0 % |
| AUC CV: Random Forest / Reg. Logística | 0,909 / 0,908 → se elige Reg. Logística |
| Test: ROC-AUC · PR-AUC | 0,907 · 0,745 |
| Test: recall · precisión · F1 (umbral 0,64) | 0,740 · 0,646 · 0,690 |
| Brecha regional (Biobío / RM / Valparaíso) | 2,6 % / 0,1 % / 1,4 % — cumple |
| Riesgo Alto / Medio / Bajo | 5.135 / 12.323 / 33.888 |

## Estructura

```
notebooks/         cuadernos 01–06 (ejecutados)
src/               módulos usados por los cuadernos
main.py            alternativa por consola
seguimiento/       tareas.json (tareas y desviaciones)
tests/             pruebas pytest
.github/workflows/ pruebas automáticas
```

## Limitaciones

- Datos sintéticos: el AUC alto refleja las reglas del generador, no el comportamiento real.
- Probabilidades no calibradas; el nivel de riesgo es relativo (percentiles).
- Con datos reales hay que volver a auditar la equidad regional.
