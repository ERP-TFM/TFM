# Gestión inteligente de la demanda energética: predicción, flexibilidad y optimización en entornos urbanos

Trabajo Fin de Máster del **Máster Universitario en Big Data y Ciencia de Datos** de la Universidad Internacional de Valencia (VIU). Este repositorio reúne los seis notebooks que documentan el desarrollo experimental: desde el análisis de consumos eléctricos reales hasta la predicción *day-ahead* y la simulación de estrategias de gestión de la demanda (*Demand Side Management*, DSM).

## Resumen

A partir del conjunto de datos abierto de **GoiEner**, el proyecto:

1. Analiza sus metadatos y selecciona puntos de suministro residenciales de Bilbao (CNAE 9820) con histórico suficiente.
2. Reconstruye las series horarias y completa las observaciones ausentes mediante una estrategia de imputación causal.
3. Compara enfoques de predicción de demanda **individual** a 24 horas: un modelo global multihorizonte y 24 modelos directos especializados.
4. Construye una cohorte fija de **923 consumidores** y predice directamente su demanda agregada mediante 24 modelos.
5. Utiliza esas previsiones para planificar, mediante programación lineal, desplazamiento de cargas, almacenamiento y respuesta de la demanda, tanto por separado como de forma combinada.

La optimización se calcula sobre la **demanda prevista**. La demanda observada se reserva para evaluar retrospectivamente las decisiones tomadas, evitando usarla como entrada del optimizador.

## Notebooks y orden de ejecución

Ejecutar en el orden indicado, ya que cada etapa utiliza datos o artefactos generados por las anteriores:

| Nº | Notebook | Contenido principal |
|---|---|---|
| 1 | `1_EDA_metadata.ipynb` | Exploración geográfica y por actividad; selección de los usuarios y generación de `metadata_filtered.csv`. |
| 2 | `2_Raw_data_preparation.ipynb` | Lectura de series crudas, reconstrucción de la rejilla horaria, imputación causal e informes de calidad. |
| 3 | `3_Demand_forecast_direct_24h.ipynb` | Primer experimento: predicción individual con un modelo global multihorizonte, referencias estacionales y evaluación temporal. |
| 4 | `4_Demand_forecast_24_models.ipynb` | Segundo experimento: predicción individual mediante 24 modelos directos, y evaluación de la suma de sus previsiones. |
| 5 | `5_Aggregated_demand_forecast_24_models.ipynb` | Cohorte estable, predicción directa de demanda agregada, selección de modelos y evaluación final. |
| 6 | `6_DSM_aggregated_demand.ipynb` | Optimización DSM sobre las previsiones agregadas, escenarios de flexibilidad, sensibilidad y referencia de *perfect foresight*. |

## Datos y estructura de trabajo

**Los datos originales y los artefactos no están incluidos en este repositorio.** Para reproducir los experimentos es necesario obtener los datos de GoiEner y generar los archivos intermedios.

Fuente utilizada: [GoiEner smart meters data (Zenodo, DOI: 10.5281/zenodo.7362094)](https://doi.org/10.5281/zenodo.7362094). Se utilizan **`metadata.csv`** y las series sin imputación del archivo **`raw.tzst`** de ese conjunto de datos; no deben sustituirse por los archivos ya imputados ni por versiones posteriores del dataset sin adaptar y revisar el procesamiento.

Organización esperada **en el directorio de trabajo desde el que se ejecutan los notebooks**:

```text
TFM/
├── 1_EDA_metadata.ipynb
├── 2_Raw_data_preparation.ipynb
├── 3_Demand_forecast_direct_24h.ipynb
├── 4_Demand_forecast_24_models.ipynb
├── 5_Aggregated_demand_forecast_24_models.ipynb
├── 6_DSM_aggregated_demand.ipynb
├── metadata.csv                         # Descargar de GoiEner
├── metadata_filtered.csv                # Generado por el notebook 1
├── raw/
│   └── raw_pub/
│       ├── <identificador_usuario_1>.csv
│       └── <identificador_usuario_2>.csv
├── processed/                           # Generado por los notebooks
├── artifacts_24h/                       # Resultados y modelo del notebook 3
├── artifacts_24_models_24h/             # Resultados y modelos del notebook 4
├── artifacts_aggregate_24h/             # Resultados y modelos del notebook 5
└── artifacts_dsm/                       # Resultados del notebook 6
```

Al descomprimir `raw.tzst`, organiza los CSV de consumo como `raw/raw_pub/<identificador_usuario>.csv`, de acuerdo con las rutas utilizadas en el notebook 2. Las series crudas se leen como **CSV de dos columnas sin cabecera**, correspondientes a fecha/hora y consumo.

### Principales archivos intermedios

Los nombres siguientes corresponden a los archivos esperados por las etapas posteriores:

| Archivo generado | Etapa que lo produce / utiliza |
|---|---|
| `metadata_filtered.csv` | Selección de usuarios del notebook 1; entrada del notebook 2. |
| `processed/consumption_all_users.parquet` | Series preparadas en el notebook 2; entrada de los experimentos individuales. |
| `processed/quality_report.parquet` y `processed/metadata_filtered.parquet` | Calidad y metadatos procesados; utilizados en los notebooks de predicción. |
| `processed/consumption_users_le10pct_imputed.parquet` | Subconjunto tras excluir usuarios con más de un 10 % de imputación; utilizado en la fase agregada. |
| `processed/bilbao_weather_hourly.parquet` | Caché de meteorología histórica de Bilbao; se genera o reutiliza durante el modelado. |
| `processed/bilbao_ce_aggregate_features.parquet` | Serie agregada y variables explicativas del notebook 5; entrada del notebook 6. |
| `artifacts_aggregate_24h/final_config.json`, `final_features.pkl` y `aggregate_h01.pkl` … `aggregate_h24.pkl` | Configuración y modelos finales del notebook 5; **necesarios para ejecutar el notebook 6**. |

Los notebooks crean otros CSV, Parquet y modelos en sus directorios de artefactos; su contenido se detalla en las propias celdas de guardado.

## Entorno y ejecución

Se recomienda un entorno virtual de Python y Jupyter. Las bibliotecas importadas en los notebooks incluyen:

```bash
python -m venv .venv
# Linux/macOS:
source .venv/bin/activate
# Windows (PowerShell):
# .venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
python -m pip install jupyterlab numpy pandas pyarrow matplotlib scipy scikit-learn lightgbm xgboost catboost optuna joblib requests holidays tqdm
jupyter lab
```

Este comando instala las dependencias utilizadas.

Para la descarga de meteorología histórica mediante **Open-Meteo** se necesita conexión a Internet.

## Fuente y atribución

**Datos:** Quesada Granja, C., Borges Hernández, C. E., Astigarraga, L., y Merveille, C. *GoiEner smart meters data*. Zenodo. https://doi.org/10.5281/zenodo.7362094

**Publicación asociada:** Quesada, C., Astigarraga, L., Merveille, C., & Borges, C. E. (2024). An electricity smart meter dataset of Spanish households: Insights into consumption patterns. *Scientific Data, 11*, 59. https://doi.org/10.1038/s41597-023-02846-0

**Meteorología:** [Open-Meteo](https://open-meteo.com/) — [Open-Meteo.com Weather API (Zenodo)](https://doi.org/10.5281/zenodo.7970649).

Este repositorio documenta los experimentos del TFM. La licencia de los datos originales es la indicada en su repositorio de origen; la publicación de estos notebooks no implica que se redistribuyan los datos ni que se conceda una licencia diferente sobre ellos.
