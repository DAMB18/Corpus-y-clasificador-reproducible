# INF-8239 · Unidad 02 · Proyecto NLP

Autor académico: Edwin Ramón José Nolasco

Proyecto base para LAB04–LAB06. No sustituya la comprensión por ejecución mecánica.

## Inicio rápido

```bash
uv python install 3.12
uv sync
uv run pytest -q
uv run python scripts/audit_data.py
```

Copie `.env.example` como `.env` y configure el dataset aprobado.

## Dataset

Complete `docs/DATASET_CARD.md`, el diccionario, la licencia y el procedimiento de obtención.
El archivo `data/sample/demo_text.csv` solamente comprueba la arquitectura.

**Dataset aprobado (LAB04):** "LLM - Detect AI Generated Text Dataset" (Kaggle, Apache 2.0, 29,145
filas, columnas `text`/`generated`). Ver `docs/DATASET_CARD.md` para la auditoría completa, incluido
el historial de un primer candidato rechazado por contener texto sintético.

## Entrenamiento

```bash
uv run python scripts/train_text.py
uv run streamlit run app/streamlit_app.py
```

**Resultados (LAB05):** dummy F1 macro=0.371, Naive Bayes=0.959, regresión logística=0.989
(modelo seleccionado). Ver `reports/text_metrics.csv`, `reports/confusion_text.png` y
`docs/analisis_errores.md` para el análisis completo de los 74 errores del conjunto de prueba.

## Notebook resumen

`notebooks/resumen_resultados.ipynb` reutiliza el código de `src/inf8239_u02` y los artefactos de
`reports/` y `models/` para mostrar evidencia visual (auditoría, métricas, matriz de confusión,
errores categorizados) sin duplicar la lógica de entrenamiento. Para ejecutarlo:

```bash
uv add --group dev jupyter
uv run jupyter nbconvert --to notebook --execute --inplace notebooks/resumen_resultados.ipynb
```

## Interpretación

Toda conclusión debe separar observación, evidencia, interpretación y decisión.

### Cierre interpretativo (LAB05)

**Resultado principal:** se entrenaron tres clasificadores de texto (baseline aleatorio, Naive Bayes
complementario y regresión logística) sobre 27,340 ensayos del corpus "LLM - Detect AI Generated
Text" (tras eliminar 1,805 duplicados), usando TF-IDF como representación. Los dos modelos
entrenados superaron ampliamente el baseline (F1 macro=0.371): Naive Bayes alcanzó 0.959 y la
regresión logística 0.989.

**Modelo seleccionado y evidencia:** se seleccionó la regresión logística por su F1 macro superior y
su balance entre clases (recall de 0.99 para texto humano y 0.98 para texto generado por IA). La
matriz de confusión muestra 4,010 aciertos en la clase humana y 2,751 en la clase IA, frente a solo
21 falsos positivos y 53 falsos negativos sobre 6,835 casos de prueba (98.9% de exactitud).

**Clase con mayor dificultad:** en términos absolutos, la clase "generado por IA" concentra más
errores (53 falsos negativos frente a 21 falsos positivos de la clase humana), aunque la diferencia
relativa de recall es pequeña (0.98 vs 0.99).

**Tipo de error más frecuente:** se categorizaron los 74 errores del conjunto de prueba (ver
`docs/analisis_errores.md`). El 43% (32 casos) son ensayos de IA bien escritos, en registro formal,
sin ninguna marca superficial que los delate. Un 24% (18 casos) son ensayos humanos inusualmente
bien estructurados que el modelo confundió con IA — aparentemente aprendió "estructura formal = IA"
como atajo. Un 23% (17 casos) son ensayos de IA escritos deliberadamente en registro informal o con
errores ortográficos insertados a propósito, un patrón consistente con ejemplos adversariales
documentados en la literatura sobre contaminación en este tipo de benchmarks.

**Impacto en el contexto:** se probó el modelo con tres casos fuera del comportamiento esperado: un
texto en español de autoría propia, una oración casual en inglés fuera del dominio del corpus, y una
frase formal y corta imitando el estilo de un ensayo. Las tres veces el modelo predijo "generado por
IA" con la misma confianza aparente que para un caso real del corpus, sin ninguna señal de
incertidumbre. En un contexto real de detección de plagio o uso indebido de IA, esto significa que un
texto corto, fuera de dominio, o simplemente bien escrito por un humano, podría generar una
acusación falsa.

**Limitación del dataset:** el corpus es exclusivamente en inglés, con un desbalance de clases de
60/40, un 6.2% de duplicados (corregido antes de entrenar) y una diferencia sistemática de longitud
entre clases (los textos humanos son ~17% más largos en promedio) que probablemente el modelo
aprovecha como señal además del contenido real. El género textual se limita a ensayos persuasivos de
estilo escolar de una época específica de modelos de lenguaje (2023).

**Decisión antes del despliegue:** no se desplegaría este modelo para tomar decisiones automáticas
sobre estudiantes reales. Antes de cualquier uso con consecuencias, se agregaría un mecanismo de
detección de fuera-de-dominio (idioma, longitud mínima, similitud con el vocabulario de
entrenamiento) que se niegue a predecir en vez de forzar una respuesta, manteniendo siempre revisión
humana antes de cualquier acción disciplinaria.

## Uso de IA

Ver `docs/declaracion_uso_ia.md` para la declaración completa de herramientas de IA utilizadas,
partes verificadas con datos reales y correcciones realizadas.
