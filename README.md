# Binary Mixture Analyzer

**[English](#english) · [Español](#español)**

---

## English

A browser tool to analyze the effects of **binary mixtures of contaminants on multiple response variables**. For each variable, time point and mixture it tests whether the combined effect departs from additivity and classifies the interaction as additive, synergistic or antagonistic, using both the classic criterion and the directional classification proposed by **Piggott et al. (2015)**.

It runs entirely in the browser: no installation, no server, and the data never leave your computer.

### Background

The tool reproduces and extends a procedure the author had previously applied by hand to factorial experiments with herbicide and insecticide mixtures (Lozano et al. 2018; Miranda et al. 2021; Lozano et al. 2024a, 2024b). It was built with the assistance of **Claude (Anthropic)** from those studies and spreadsheets. The original procedure only distinguished synergism and antagonism by magnitude; this version adds the directional framework of Piggott et al. (2015), which takes into account whether the single and combined effects are positive or negative.

### What it does

- Reads an Excel (.xlsx) or CSV file, or a table pasted from Excel, in long format.
- Analyzes many variables, mixtures (for example, low and high concentration) and time points at once.
- For each case reports the single effects, the additive expectation, the observed combined effect, the interaction with its 95% confidence interval and p-value, and Hedges' d of the interaction.
- Classifies each case with the five classes of Piggott et al. (2015) and flags mitigating synergism. The classic three-class label is shown alongside for comparison.
- Draws a forest-style figure with an additivity band, one marker per mixture and the class of each case. The figure can be exported as SVG or PNG and the table as CSV or copied directly into Excel.

### Interaction classes (Piggott et al. 2015)

Effects are differences from the control mean: `a = Ā − C̄`, `b = B̄ − C̄`, expected additive effect `E = a + b`, observed combined effect `O = AB̄ − C̄`, interaction `I = O − E`.

| Condition | Class |
|---|---|
| Interaction not significant | **AD** additive |
| I > 0 and O > max(a, b, 0) | **+S** positive synergism |
| I > 0 and O ≤ max(a, b, 0) | **−A** negative antagonism (a negative effect is dampened) |
| I < 0 and O < min(a, b, 0) | **−S** negative synergism |
| I < 0 and O ≥ min(a, b, 0) | **+A** positive antagonism (a positive effect is dampened) |

When both single effects act in the same direction and the combined effect reverses sign, the case is flagged as **mitigating synergism** (↺), following Piggott et al. (2015). The classic label compares magnitudes only (|O| > |E| synergism, |O| < |E| antagonism).

### Input format

One row per experimental unit, one column with the treatment code and one column per response variable. An optional column (for example, sampling time) can be used to split the analysis.

```
Unit,Treatment,Time,Chlorophyll,Abundance
C-1,C,7,19.0,504
A1-1,A1,7,17.2,470
B1-1,B1,7,18.1,481
M1-1,M1,7,15.3,452
...
```

In the *Design* panel you choose the control code and, for each mixture, which codes correspond to A alone, B alone and A + B. Codes that share a suffix (A1, B1, M1) are detected automatically. Comma or dot decimals are accepted. Each cell needs at least two replicates.

### How to use

1. Open `index.html` in any modern browser (or the online version on GitHub Pages).
2. Load your file or paste your table. The page opens with a simulated example data set (`example_data/example_binary_mixture.csv`) that contains all five classes.
3. Check the design (control, mixtures, variables) and the analysis options.
4. Read the summary, the figure and the table; export what you need.

### Methods

- **Additivity test (default):** interaction contrast of a 2 × 2 ANOVA, `SE(I) = s_p · √(1/n_C + 1/n_A + 1/n_B + 1/n_AB)`, df = N − 4. This is equivalent to the interaction F test.
- **Welch option:** each cell keeps its own variance, with Satterthwaite degrees of freedom. Useful when variances differ strongly between treatments.
- **Single effects** a and b are tested with the same error term. By default, a non-significant single effect counts as 0 when classifying (the "neutral" rows of Table 1 in Piggott et al. 2015).
- **Strict synergism (optional):** the 95% CI of the combined effect must not overlap the largest single effect or the control, as in the re-analysis by Piggott et al. (2015) of the Crain et al. (2008) data set.
- **Hedges' d of the interaction:** `d_I = I / (2 s_p) · J`, `J = 1 − 3/(4m − 1)`, m = N − 4, `v = ¼ [Σ 1/n_i + d_I² / (2N)]` (Gurevitch et al. 2000; Crain et al. 2008). It allows comparison across variables with different units or across experiments.
- **Figure axis (default):** interaction divided by the half-width of its 95% CI. The bar always spans ±1, so a case outside the grey band is non-additive and the figure agrees exactly with the test. Other axes: Hedges' d, % of the control mean and original units. The *Magnitude* orientation multiplies by the sign of E, so that synergism (by magnitude) falls to the right, as in the classic figures.

**Limitations.** The tool uses the additive null model on the original scale. For proportions such as mortality, or for multiplicative processes, a multiplicative model (independent action) may be more appropriate. Designs with blocks (for example, tanks or trays that each hold one replicate of every treatment) are not modeled by the page; in that case the block should be included in the analysis with statistical software. When many variables or time points are tested, consider correcting for multiple comparisons.

### Validation

`validation/validate.py` extracts the calculation code from `index.html`, runs it with Node.js on the example data and compares it with an independent fit of `y ~ A * B` in statsmodels. Interaction estimates and p-values match for all cases (`validation/validation_results.csv`). `validation/validate.R` fits the same model in R for an independent check.

### How to cite

Please cite the software (see `CITATION.cff`): "Lozano, V.L (2026) Binary Mixture Analyzer: additivity, synergism and antagonism of binary contaminant mixtures on multiple response variables" and the classification framework:

> Piggott, J.J., Townsend, C.R. & Matthaei, C.D. (2015). Reconceptualizing synergism and antagonism among multiple stressors. *Ecology and Evolution*, 5(7), 1538–1547. https://doi.org/10.1002/ece3.1465

### License

MIT (see `LICENSE`). `index.html` embeds SheetJS Community Edition 0.18.5 to read Excel files, distributed under the Apache License 2.0 (`THIRD_PARTY_LICENSE_SheetJS.txt`).

---

## Español

Una herramienta que funciona en el navegador para analizar los efectos de **mezclas binarias de contaminantes sobre múltiples variables de respuesta**. Para cada variable, tiempo y mezcla evalúa si el efecto combinado se aparta de la aditividad y clasifica la interacción como aditiva, sinérgica o antagónica, con el criterio clásico y con la clasificación direccional propuesta por **Piggott et al. (2015)**.

Funciona completamente en el navegador: no hay que instalar nada, no usa servidor y los datos no salen de tu computadora.

### Origen

La herramienta reproduce y amplía un procedimiento que la autora aplicaba de forma manual en experimentos factoriales con mezclas de herbicidas e insecticidas (Lozano et al. 2018; Miranda et al. 2021; Lozano et al. 2024a, 2024b). Fue armada con la ayuda de **Claude (Anthropic)** a partir de esos trabajos y planillas. El procedimiento original solo distinguía sinergismo y antagonismo por magnitud; esta versión incorpora el marco direccional de Piggott et al. (2015), que tiene en cuenta si los efectos individuales y el combinado son positivos o negativos.

### Qué hace

- Lee un archivo Excel (.xlsx) o CSV, o una tabla pegada desde Excel, en formato largo.
- Analiza muchas variables, mezclas (por ejemplo, concentración baja y alta) y tiempos a la vez.
- Para cada caso informa los efectos individuales, el esperado aditivo, el efecto combinado observado, la interacción con su intervalo de confianza del 95 % y su valor p, y el d de Hedges de la interacción.
- Clasifica cada caso con las cinco clases de Piggott et al. (2015) y señala el sinergismo mitigador. Al lado muestra la clasificación clásica de tres clases para comparar.
- Dibuja una figura tipo forest plot con la banda de aditividad, un símbolo por mezcla y la clase de cada caso. La figura se exporta en SVG o PNG y la tabla en CSV o se copia directamente a Excel.

### Clases de interacción (Piggott et al. 2015)

Los efectos se calculan como diferencias respecto de la media del control: `a = Ā − C̄`, `b = B̄ − C̄`, esperado aditivo `E = a + b`, efecto combinado observado `O = AB̄ − C̄`, interacción `I = O − E`.

| Condición | Clase |
|---|---|
| Interacción no significativa | **AD** aditivo |
| I > 0 y O > máx(a, b, 0) | **+S** sinergismo positivo |
| I > 0 y O ≤ máx(a, b, 0) | **−A** antagonismo negativo (se atenúa un efecto negativo) |
| I < 0 y O < mín(a, b, 0) | **−S** sinergismo negativo |
| I < 0 y O ≥ mín(a, b, 0) | **+A** antagonismo positivo (se atenúa un efecto positivo) |

Cuando los dos efectos individuales van en la misma dirección y el efecto combinado cambia de signo, el caso se marca como **sinergismo mitigador** (↺), siguiendo a Piggott et al. (2015). La clasificación clásica compara solo magnitudes (|O| > |E| sinergismo, |O| < |E| antagonismo).

### Formato de los datos

Una fila por unidad experimental, una columna con el código de tratamiento y una columna por variable de respuesta. Opcionalmente, una columna (por ejemplo, el tiempo de muestreo) para separar el análisis.

```
Unit,Treatment,Time,Chlorophyll,Abundance
C-1,C,7,19.0,504
A1-1,A1,7,17.2,470
B1-1,B1,7,18.1,481
M1-1,M1,7,15.3,452
...
```

En el panel *Design* se elige el código del control y, para cada mezcla, qué códigos corresponden a A sola, B sola y A + B. Los códigos que comparten sufijo (A1, B1, M1) se detectan automáticamente. Se aceptan decimales con coma o con punto. Cada celda necesita al menos dos réplicas.

### Cómo se usa

1. Abrir `index.html` en cualquier navegador actual (o la versión en línea en GitHub Pages).
2. Cargar el archivo o pegar la tabla. La página abre con un conjunto de datos simulado (`example_data/example_binary_mixture.csv`) que contiene las cinco clases.
3. Revisar el diseño (control, mezclas, variables) y las opciones de análisis.
4. Leer el resumen, la figura y la tabla, y exportar lo que se necesite.

### Métodos

- **Prueba de aditividad (por defecto):** contraste de interacción de un ANOVA 2 × 2, `EE(I) = s_p · √(1/n_C + 1/n_A + 1/n_B + 1/n_AB)`, gl = N − 4. Equivale a la prueba F de la interacción.
- **Opción Welch:** cada celda conserva su propia varianza, con grados de libertad de Satterthwaite. Útil cuando las varianzas difieren mucho entre tratamientos.
- **Efectos individuales:** a y b se prueban con el mismo término de error. Por defecto, un efecto individual no significativo cuenta como 0 al clasificar (filas "neutro" de la Tabla 1 de Piggott et al. 2015).
- **Sinergismo estricto (opcional):** el IC95 del efecto combinado no debe solaparse con el efecto individual mayor ni con el control, como en el reanálisis de Piggott et al. (2015) sobre los datos de Crain et al. (2008).
- **d de Hedges de la interacción:** `d_I = I / (2 s_p) · J`, `J = 1 − 3/(4m − 1)`, m = N − 4, `v = ¼ [Σ 1/n_i + d_I² / (2N)]` (Gurevitch et al. 2000; Crain et al. 2008). Permite comparar variables con unidades distintas o distintos experimentos.
- **Eje de la figura (por defecto):** la interacción dividida por el semiancho de su IC95. La barra siempre mide ±1, así que un caso fuera de la banda gris es no aditivo y la figura coincide exactamente con la prueba. Otros ejes: d de Hedges, % de la media del control y unidades originales. La orientación *Magnitude* multiplica por el signo de E, de modo que el sinergismo (por magnitud) queda a la derecha, como en las figuras clásicas.

**Limitaciones.** La herramienta usa el modelo nulo aditivo en la escala original. Para proporciones como la mortalidad, o para procesos multiplicativos, puede ser más adecuado un modelo multiplicativo (acción independiente). Los diseños con bloques (por ejemplo, tanques o bandejas que contienen una réplica de cada tratamiento) no se modelan en la página; en ese caso conviene incluir el bloque en el análisis con un programa estadístico. Cuando se prueban muchas variables o tiempos, conviene corregir por comparaciones múltiples.

### Validación

`validation/validate.py` extrae el código de cálculo de `index.html`, lo ejecuta con Node.js sobre los datos de ejemplo y lo compara con un ajuste independiente de `y ~ A * B` en statsmodels. Las estimaciones y los valores p de la interacción coinciden en todos los casos (`validation/validation_results.csv`). `validation/validate.R` ajusta el mismo modelo en R para una verificación independiente.

### Cómo citar

Citar el software (ver `CITATION.cff`): "Lozano, V.L (2026) Binary Mixture Analyzer: additivity, synergism and antagonism of binary contaminant mixtures on multiple response variables" y el marco de clasificación:

> Piggott, J.J., Townsend, C.R. & Matthaei, C.D. (2015). Reconceptualizing synergism and antagonism among multiple stressors. *Ecology and Evolution*, 5(7), 1538–1547. https://doi.org/10.1002/ece3.1465

### Licencia

MIT (ver `LICENSE`). `index.html` incluye SheetJS Community Edition 0.18.5 para leer archivos Excel, distribuido bajo la licencia Apache 2.0 (`THIRD_PARTY_LICENSE_SheetJS.txt`).

---

## References / Referencias

- Crain, C.M., Kroeker, K. & Halpern, B.S. (2008). Interactive and cumulative effects of multiple human stressors in marine systems. *Ecology Letters*, 11, 1304–1315.
- Gurevitch, J., Morrison, J.A. & Hedges, L.V. (2000). The interaction between competition and predation: a meta-analysis of field experiments. *The American Naturalist*, 155, 435–453.
- Lozano, V.L., Vinocur, A., Sabio y García, C.A., Allende, L., Cristos, D.S., Rojas, D., Wolansky, M. & Pizarro, H. (2018). Effects of glyphosate and 2,4-D mixture on freshwater phytoplankton and periphyton communities: a microcosms approach. *Ecotoxicology and Environmental Safety*, 148, 1010–1019.
- Lozano, V.L., Paolucci, E.M., Sabatini, S.E., Noya Abad, T., Muñoz, C., Liquin, F., Hollert, H. & Sylvester, F. (2024a). Assessing the impact of imidacloprid, glyphosate, and their mixtures on multiple biomarkers in *Corbicula largillierti*. *Science of the Total Environment*, 942, 173685.
- Lozano, V.L., Miranda, C.E., Vinocur, A.L., Sabio y García, C.A., Vera, M.S., González, C., Wolansky, M.J. & Pizarro, H.N. (2024b). Freshwater dissolved oxygen dynamics: changes due to glyphosate, 2,4-D, and their mixture, both under clear and turbid-organic conditions. *Hydrobiologia*. https://doi.org/10.1007/s10750-024-05594-3
- Miranda, C.E., Clauser, C.D., Lozano, V.L., Cataldo, D.H. & Pizarro, H.N. (2021). An invasive mussel is in trouble: how do glyphosate, 2,4-D and its mixture affect *Limnoperna fortunei*'s survival? *Aquatic Toxicology*, 239, 105957.
- Piggott, J.J., Townsend, C.R. & Matthaei, C.D. (2015). Reconceptualizing synergism and antagonism among multiple stressors. *Ecology and Evolution*, 5(7), 1538–1547. https://doi.org/10.1002/ece3.1465
