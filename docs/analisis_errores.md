# Análisis de errores — LAB05 (regresión logística)

Categorización de los **74 errores** del conjunto de prueba (6,835 casos, F1 macro=0.989).
El conteo mínimo pedido por la guía es 20; se categorizó el universo completo de errores para no
sesgar la interpretación con una muestra parcial.

## Matriz de confusión

| | Predicho: humano (0) | Predicho: IA (1) |
|---|---|---|
| **Real: humano (0)** | 4,010 | 21 (falso positivo) |
| **Real: IA (1)** | 53 (falso negativo) | 2,751 |

Los falsos negativos (IA clasificada como humana) son más numerosos en términos absolutos, aunque
la diferencia de recall entre clases es pequeña (0.98 vs 0.99).

## Categorías encontradas

| Categoría | Casos | Dirección |
|---|---|---|
| Ensayo de IA en registro formal/competente, sin marca aparente (caso difícil, cerca del límite de decisión) | 32 | IA → humano |
| Texto humano muy estructurado/formal (parece IA) | 18 | humano → IA |
| Registro informal/jerga — la IA imita habla casual ("Hey there!", "like,", "gonna", "stuff") | 14 | IA → humano |
| Errores ortográficos insertados artificialmente ("writting", "choise", "ushally") | 4 | IA → humano |
| Registro informal + errores insertados a la vez | 3 | IA → humano |
| Estructura formal con cita célebre ("Winston Churchill once said...") | 3 | humano → IA |

## Interpretación

**1. Un 40% de los falsos negativos (21 de 53) son ensayos de IA deliberadamente "disfrazados" de
texto humano** — jerga, muletillas y, en varios casos, errores ortográficos insertados a propósito
("When peoples ask for advices... ushally... choise"). No es ruido: coincide con el tipo de ejemplo
adversarial que señala la literatura académica sobre contaminación en esta familia de datasets
(ver `docs/DATASET_CARD.md`, sección de riesgos). Es una hipótesis verificable — el modelo depende
del registro/formalidad como señal, y cuando la IA imita informalidad, la señal se debilita.

**2. El 60% restante de los falsos negativos (32 de 53) son ensayos de IA bien escritos, sin ninguna
marca superficial obvia** — casos genuinamente cerca del límite de decisión del modelo, no un patrón
sistemático identificable.

**3. Los falsos positivos (humano → IA) tienden a ser ensayos humanos inusualmente bien
estructurados**, varios abriendo con una cita célebre ("Winston Churchill once said...", "The famous
artist Michelangelo once said..."), un recurso enseñado en la escuela que también es un patrón común
de apertura en texto generado por LLM. El modelo parece haber aprendido "estructura formal con cita =
IA", lo cual penaliza a quienes escriben bien.

## Verificación manual adicional (app Streamlit)

Se probaron tres casos fuera del comportamiento esperado directamente en `app/streamlit_app.py`:

| Texto | Predicción |
|---|---|
| "Excelente servicio" (español) | Generado por IA |
| "Today, we are waiting for the news from London, while we are waiting, we must go for a pizza." (inglés, fuera del género del corpus) | Generado por IA |
| "I believe that students should not be allowed to use cell phones during class because it distracts them from learning and reduces their attention span." (inglés, formal, corto — escrito por el estudiante) | Generado por IA |

Los tres casos — incluyendo un texto humano real (el tercero, de autoría del estudiante) —
fueron clasificados como "generado por IA" con la misma confianza aparente que un caso real del
corpus. Esto confirma que el modelo nunca se abstiene: fuerza una respuesta entre las dos clases
incluso cuando el texto está completamente fuera del idioma, género o longitud del corpus de
entrenamiento. Es la evidencia más directa de por qué este modelo no debe usarse para decisiones
automáticas sin revisión humana (ver `docs/DATASET_CARD.md` y el cierre interpretativo en
`README.md`).
