# Declaración de uso de IA — U02.E03

## Herramienta utilizada

Claude (Anthropic), en modalidad Cowork, con la sesión vinculada a mi computadora para poder
ejecutar comandos de PowerShell, leer archivos reales del proyecto y verificar resultados — no se
trabajó solo a partir de descripciones, sino ejecutando el código real en mi máquina y revisando su
salida.

## Tareas en las que se usó como apoyo

- Guiar la búsqueda y evaluación de datasets candidatos en Kaggle (verificación de licencia, tamaño
  y columnas vía la API pública de metadatos de Kaggle).
- Interpretar la salida de `scripts/audit_data.py` y `scripts/train_text.py`.
- Diagnosticar errores de entorno (`uv`, rutas de `.env`, archivos mal ubicados).
- Leer y categorizar los 74 errores del conjunto de prueba (`reports/error_analysis.csv`).
- Redactar un borrador del cierre interpretativo del README y del análisis de errores, que luego
  revisé y ajusté.
- Dar instrucciones de Git para los commits de cada etapa.

## Partes verificadas con datos reales (no generadas ni asumidas)

- El SHA-256, la forma del dataset (filas/columnas), los nulos, los duplicados y la distribución de
  clases provienen de ejecutar `audit_data.py` sobre el archivo real descargado — no fueron
  inventados.
- Las métricas de los tres modelos (F1 macro, matriz de confusión) provienen de ejecutar
  `train_text.py` en mi máquina y pegar la salida real de la terminal.
- Los fragmentos de texto citados en `docs/analisis_errores.md` fueron leídos directamente de
  `reports/error_analysis.csv`, generado por mi propio modelo entrenado.
- Las tres pruebas manuales en la app de Streamlit (texto en español, texto casual fuera de dominio,
  texto corto formal) las escribí y ejecuté yo mismo; el resultado reportado es el que obtuve en mi
  pantalla.

## Correcciones realizadas a partir de la revisión con IA

- Descarté el primer dataset candidato ("AI vs Human Content Detection") después de auditar el
  archivo real descargado: el texto resultó ser sintético (tipo generado por la librería Faker), sin
  relación real con el propósito del dataset. La ficha de Kaggle no mostraba este problema — solo se
  detectó al revisar el contenido real.
- Corregí dos archivos que había colocado en el lugar equivocado (`.env` y un script de descarga
  dentro de `data/raw/`, que `config.py` nunca lee desde ahí).
- Corregí el valor de `TEXT_COLUMN` en `.env`, que no coincidía con el nombre real de la columna del
  segundo dataset (el valor por defecto de la plantilla era `text_content`/`text`/`label` según el
  caso, pero cada dataset tiene su propio nombre de columna real).

## Responsabilidad

Revisé y puedo explicar cada fragmento de código, cada cifra y cada conclusión presentada en esta
entrega. Los datos, el código, las referencias (licencias, SHA-256) y las conclusiones del cierre
interpretativo son de mi responsabilidad.
