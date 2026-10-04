# Dataset Card

## Estado actual

**Aceptado**, tras auditar el archivo real descargado (ver sección "Calidad observada"). Este es el segundo candidato evaluado — el primero fue rechazado por contener texto sintético; ver "Historial de auditoría" al final.

## Identificación

- Nombre: LLM - Detect AI Generated Text Dataset
- Fuente original: Kaggle — https://www.kaggle.com/datasets/sunilthite/llm-detect-ai-generated-text-dataset
- Responsable: recopilado por Sunil Thite en Kaggle, a partir del concurso "LLM - Detect AI Generated Text" (ensayos humanos del corpus Persuade + ensayos generados por varios LLMs)
- Versión o fecha: descargado el 4 de octubre de 2026. **SHA-256 del archivo usado:** `f68515bedeaa2e7a55705430863aaaa3a9b59cffa1b84dbc28eb095f1cb3a75e` (fija exactamente qué versión del archivo se auditó; si Kaggle lo actualiza más adelante, el hash cambiaría).
- Licencia: Apache 2.0 — permite descargar, analizar y publicar resultados derivados.
- Idioma: inglés

## Propósito y variable objetivo

Clasificación binaria: distinguir un ensayo escrito por un estudiante humano de uno generado por un LLM. Target: `generated` (0 = humano, 1 = generado por IA). Característica de modelado: `text` (el ensayo completo).

## Diccionario de datos

| Columna | Tipo | Descripción | Valores o unidad |
|---|---|---|---|
| `text` | texto | Ensayo completo (persuasivo, carta a un senador, reflexión sobre tecnología, etc.) | string libre, 1–9,157 caracteres |
| **`generated`** (target) | binaria | 0 = escrito por un estudiante humano (corpus Persuade), 1 = generado por un LLM | {0, 1} |

## Procedimiento de obtención

1. Se descargó manualmente desde https://www.kaggle.com/datasets/sunilthite/llm-detect-ai-generated-text-dataset con sesión de Kaggle iniciada (botón "Download").
2. Se colocó en `data/raw/dataset.csv` (65,457,029 bytes).
3. Se verificó con `uv run python scripts/download_data.py` → confirma modo local y calcula el SHA-256 anterior.
4. `.env` configurado en la raíz del proyecto con:
   ```
   DATA_SOURCE=local
   DATASET_PATH=data/raw/dataset.csv
   DATASET_URL=
   TEXT_COLUMN=text
   TARGET_COLUMN=generated
   RANDOM_STATE=42
   ```
5. No se usó `kagglehub` ni la API de Kaggle (requeriría token de cuenta, lo que iría contra el principio de "sin credenciales en el código" del paso 10 de la guía).

## Calidad observada

Auditoría sobre el archivo real (29,145 filas × 2 columnas):

- **Nulos:** ninguno en `text` ni en `generated`.
- **Duplicados de texto:** 1,805 filas (6.2%) comparten texto exacto con alguna otra fila. Se revisó si algún texto duplicado tiene etiquetas distintas (lo cual sería una contradicción grave) — **no ocurre en ningún caso** (0 de 1,802 textos duplicados únicos). Aun así, antes de dividir en entrenamiento/prueba conviene eliminar duplicados para que el mismo ensayo no aparezca en ambos conjuntos.
- **Distribución de clases:** 60.1% humano (0) / 39.9% IA (1) — desbalanceado pero no extremo. El pipeline de `src/inf8239_u02/modeling.py` ya usa `class_weight="balanced"` en la regresión logística, lo cual es apropiado para este desbalance.
- **Longitud de texto:** media 2,236 caracteres, mediana 2,158, percentil 99 en 5,237, máximo 9,157.
- **Filas basura:** 3 filas con texto casi vacío (`"]"`, `"]\n\n[Email]\n\n[Phone Number]"`, `"]\n[Email]\n[Phone Number]"`) — restos de una plantilla de carta mal capturada durante la generación del ensayo por IA. Son despreciables en cantidad (3 de 29,145) pero deben filtrarse (por ejemplo, descartar textos de menos de ~20 caracteres) antes de entrenar.
- **Diferencia de longitud por clase:** los textos humanos promedian ~2,403 caracteres y los generados por IA ~1,984 (17% más cortos en promedio). No es un error de los datos, pero es una señal superficial real que un modelo de TF-IDF podría aprovechar parcialmente en vez de aprender estilo genuino — se documenta como riesgo abajo.

## Población cubierta y excluida

- Solo inglés; ensayos de estilo escolar/persuasivo (cartas a senadores, ensayos sobre tecnología, autos, exploración espacial, etc. — temas típicos del corpus Persuade de evaluación de escritura estudiantil en EE. UU.).
- No cubre otros géneros textuales (noticias, redes sociales, código, etc.) — un modelo entrenado aquí no necesariamente generaliza a esos dominios.
- No se declara qué modelo(s) de LLM generaron los textos "IA" específicamente en esta re-publicación del dataset.

## Riesgos, sesgos y usos prohibidos

- **Señal superficial de longitud:** los ensayos humanos son en promedio 17% más largos que los generados por IA (ver "Calidad observada"). Un clasificador podría apoyarse parcialmente en esto en vez de en diferencias de estilo más profundas — al interpretar resultados, conviene revisar si el modelo generaliza a textos de longitud similar entre clases.
- **Duplicados:** 6.2% de filas duplicadas — deben eliminarse antes de partir train/test para evitar fuga entre conjuntos (el mismo ensayo visto en ambos).
- **Filas basura:** 3 filas con texto casi vacío por un artefacto de recolección — deben filtrarse.
- **Desbalance de clases:** 60/40 — ya mitigado en el pipeline con `class_weight="balanced"`, pero vale mencionarlo al reportar métricas (preferir F1 o recall por clase sobre accuracy simple).
- **Riesgo documentado en la literatura académica** para esta familia de datasets (derivados del concurso de Kaggle "LLM - Detect AI Generated Text"): trabajos recientes señalan posible contaminación o artefactos de formato en versiones tempranas de este dataset de concurso. No se encontró evidencia de ese problema específico en esta auditoría (no hay columnas de metadatos de formato ni IDs de fuente en esta versión de 2 columnas), pero se documenta como antecedente a tener en cuenta si se reportan resultados inusualmente altos.
- **Uso previsto:** solo educacional, no comercializar. No usar para decisiones automatizadas de alto impacto (p. ej. acusar a un estudiante real de usar IA) sin revisión humana — es un dataset de entrenamiento académico, no un detector certificado.
- No se detectó información personal identificable real en el muestreo revisado (los "[Email]\n[Phone Number]" encontrados son placeholders de plantilla, no datos reales).

## Ejemplos anonimizados (muestra truncada a ~150 caracteres)

- `generated=0` (humano): "Dear Senator, I feel the need to eliminate the Electoral College process of voting and just have a regular election where the President with the most..."
- `generated=0` (humano): "The influening somepeople ued or beliefe that are for make a money or make more friends for have good relationship with people are rich and have many..." *(nota: errores ortográficos genuinos de escritura estudiantil — un indicio de autenticidad humana)*
- `generated=1` (IA): "Some people believe that modern technology has made life more convenient. Others believe that life was better when technology was simpler. I believe t..."
- `generated=1` (IA): "Dear Senator, I am writing to you today to express my strong support for changing our presidential election system to a popular vote. As you may kno..."
- `generated=1` (IA): " As math students, it is essential to understand the basics of math and practice solving problems. If you ever feel stuck, do not hesitate to seek hel..."

Nota: aun en ensayos con el mismo tema (voto popular, carta al Senador), se nota la diferencia de registro — el texto humano es más informal/con errores, el de IA más pulido y estructurado. Esto respalda que hay señal de estilo real detrás de la etiqueta, no solo longitud.

---

## Historial de auditoría

### Candidato rechazado: "AI vs Human Content Detection 1000+ record in 2025"

- **Licencia y ficha de Kaggle:** correctas (Apache 2.0, 1,367 filas, columnas bien documentadas).
- **Resultado de la auditoría con el archivo real** (`ai_human_content_detection_dataset.csv`, 1,367 filas × 17 columnas, sin duplicados en `text_content`, clases balanceadas 50.0%/49.9%):
  - El contenido de `text_content` **no es texto real**: es texto sintético tipo el generado por la librería `Faker` de Python (vocabulario fijo sin coherencia gramatical — ejemplo real del archivo: *"Open no cold. Field challenge candidate few executive relationship. Always nor artist network various road knowledge..."*).
  - Esto ocurre por igual en `label=0` y `label=1` — no hay ninguna diferencia real de estilo entre las clases.
  - Ni siquiera las 16 columnas de métricas ya calculadas (legibilidad, burstiness, etc.) distinguen las clases: `flesch_reading_ease` promedio es 52.18 para ambas clases; `burstiness` es 0.44 vs 0.41.
- **Conclusión:** el dataset falla el criterio de calidad/pertinencia (sección 2 y 4 de la guía) a pesar de cumplir licencia y estructura. Se documenta aquí como evidencia del proceso de auditoría — detectar y rechazar un dataset defectuoso después de revisar los datos reales, no solo la ficha, es el objetivo de este laboratorio.
