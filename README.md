# P1. Que tempo fixo?

## 1. Descripción del proyecto

Este proyecto se realiza para la asignatura de Sistemas de Big Data. El objetivo es construir un pipeline completo de datos meteorológicos diarios de la estación de A Coruña / Alvedro durante el período comprendido entre el 1 de enero de 2021 y el 31 de diciembre de 2025.

El proyecto incluye la descarga automática de los datos, la inspección del archivo original, la limpieza y validación de los datos, el análisis de los resultados, la creación de gráficas, la comparación del rendimiento de distintas operaciones y la carga de los datos en una base de datos PostgreSQL.

## 2. Estación meteorológica

- **Nombre:** A Coruña / Alvedro (aeropuerto)
- **Indicativo:** 08002
- **Latitud:** 43.3000
- **Longitud:** -8.3833
- **Altitud:** 97 m
- **Período analizado:** 01/01/2021 - 31/12/2025

## 3. Fuente de los datos

Los datos proceden originalmente de **Meteostat**. Para esta práctica se utiliza un archivo modificado con fines didácticos para incluir diferentes problemas de calidad que deben ser detectados y tratados durante el desarrollo del proyecto.

- **Fuente original:** Meteostat
- **Archivo:** `meteo_alvedro_2021_2025.csv`
- **URL de descarga:** `https://raw.githubusercontent.com/bigdatawirtz/datasets/main/datasets/meteo_alvedro_2021_2025.csv`
- **Fecha de descarga:** 29/10/2026
- **Condiciones de uso:** fichero modificado con fines didácticos a partir de datos cuya fuente original es Meteostat.

El archivo descargado se guarda sin modificar en `datos/raw/`. Después de realizar la limpieza, el resultado se guarda en `datos/clean/` como CSV con codificación UTF-8.

## 4. Estructura del proyecto

```text
p1-que-tempo-fixo/
├── README.md
├── requirements.txt
├── .gitignore
├── datos/
│   ├── raw/
│   └── clean/
├── notebooks/
│   └── 01_pipeline_meteorologico.ipynb
└── output/
```

- `datos/raw/`: contiene el archivo original descargado durante la ejecución.
- `datos/clean/`: contiene los datos después de la limpieza durante la ejecución.
- `notebooks/`: contiene el notebook con el código, explicaciones y resultados.
- `output/`: carpeta destinada a resultados que puedan exportarse.
- `requirements.txt`: contiene las bibliotecas de Python necesarias.
- `.gitignore`: indica los archivos que no deben incluirse en el repositorio.

Los archivos de datos no se incluyen en el repositorio. El notebook los genera de nuevo al ejecutar el proyecto desde cero.

## 5. Requisitos e instalación

Para ejecutar el proyecto se necesita:

- Python 3
- Docker
- Jupyter Notebook o Visual Studio Code con soporte para notebooks

Se recomienda utilizar un entorno virtual.

### Crear el entorno virtual

```bash
python3 -m venv .venv
```

### Activar el entorno virtual

En macOS/Linux:

```bash
source .venv/bin/activate
```

### Instalar las dependencias

```bash
python -m pip install -r requirements.txt
```

## 6. Ejecución del proyecto

1. Clonar o descargar el repositorio.
2. Crear y activar el entorno virtual.
3. Instalar las dependencias de `requirements.txt`.
4. Abrir `notebooks/01_pipeline_meteorologico.ipynb`.
5. Seleccionar como kernel el entorno virtual creado.
6. Ejecutar el notebook desde el principio.

El notebook descarga automáticamente el archivo original y lo guarda en `datos/raw/`. Después realiza la inspección, limpieza y análisis de los datos y guarda el resultado limpio en `datos/clean/`.

Para la parte de PostgreSQL se utiliza Docker. El contenedor puede crearse con:

```bash
docker run --name postgres-meteo \
  -e POSTGRES_USER=meteo \
  -e POSTGRES_PASSWORD=meteo123 \
  -e POSTGRES_DB=meteo_db \
  -p 5432:5432 \
  -d postgres:16
```

Si el contenedor ya existe pero está detenido, puede iniciarse con:

```bash
docker start postgres-meteo
```

Con PostgreSQL funcionando, se puede continuar la ejecución del notebook. Los datos limpios se cargan en PostgreSQL y se realizan las tres consultas SQL de la práctica.

## 7. Limpieza y tratamiento de los datos

Durante la inspección del archivo se encontraron distintos problemas de calidad. Las principales decisiones tomadas fueron las siguientes:

| Problema detectado | Cómo se detectó | Decisión tomada |
|---|---|---|
| El archivo utiliza codificación `cp1252` | Inspección de los bytes y de las primeras líneas antes de utilizar pandas | Leer el archivo indicando `encoding="cp1252"` |
| El archivo contiene metadatos antes de los datos | Inspección de las primeras líneas del archivo | Omitir las primeras 6 líneas al cargar el CSV |
| El archivo utiliza `;` como separador, coma decimal y punto de miles | Inspección del contenido original | Configurar estos parámetros al leer el archivo con `read_csv` |
| Las fechas aparecen en dos formatos diferentes | Revisión de los valores de la columna `Data` | Convertir explícitamente los formatos `YYYY-MM-DD` y `DD/MM/YYYY` a `datetime` |
| Aparecen 121 valores `Ip` en precipitación | Revisión de los valores no numéricos de la columna | Interpretarlos como precipitación inapreciable y convertirlos a 0 mm |
| Aparecen valores `-9999` en las columnas de temperatura | Revisión de valores fuera de un rango razonable y representación gráfica previa a la limpieza | Sustituirlos por `NaN`, ya que no representan temperaturas reales |
| Aparecen dos valores `-9999` en precipitación | Comprobación de precipitaciones negativas | Sustituirlos por `NaN`, ya que se trata de un código de valor ausente |
| Existen valores nulos en algunas variables | Recuento de valores ausentes de cada columna | Mantenerlos como `NaN` para no inventar datos meteorológicos que no están disponibles |
| Hay 13 fechas duplicadas: 8 con datos idénticos y 5 con diferencias | Búsqueda de fechas duplicadas y comparación de sus variables meteorológicas | Agrupar los registros de una misma fecha y calcular la media de las variables numéricas |
| Faltan 27 días del período analizado | Comparación de las fechas disponibles con el calendario completo entre 2021 y 2025 | Mantener los días ausentes sin rellenarlos, ya que no se conocen sus valores reales |
| Una temperatura media no está entre la mínima y la máxima | Comprobación de la relación entre las tres variables de temperatura | Sustituir esa temperatura media por `NaN` |
| Posibles valores incorrectos de humedad, viento y presión | Comprobación de humedad fuera de 0-100 %, viento negativo y presión fuera de 900-1100 hPa | No se encontraron valores fuera de estos rangos, por lo que no fue necesario modificarlos |

Después de la limpieza se realiza una validación final. El conjunto resultante contiene 1799 registros, no quedan fechas duplicadas, precipitaciones negativas ni inconsistencias entre las temperaturas. Se mantienen 27 días ausentes y algunos valores `NaN` porque no se dispone de información fiable para completarlos.

El conjunto limpio se guarda en:

`datos/clean/meteo_alvedro_2021_2025_clean.csv`

con codificación UTF-8.

## 8. Análisis realizado

Con los datos limpios se responden las siguientes preguntas:

1. Temperatura media de cada año y posible tendencia.
2. Mes más lluvioso y mes más seco de media.
3. Número de días de lluvia de cada mes, considerando día lluvioso aquel con al menos 1 mm de precipitación.
4. Día más caliente y más frío de cada año.

También se crean tres gráficas para facilitar la interpretación de los datos y se compara el rendimiento del cálculo de la amplitud térmica mediante un bucle y mediante una operación vectorizada.

Finalmente, los datos limpios se cargan en PostgreSQL y se realizan tres consultas SQL, comprobando sus resultados con los obtenidos previamente mediante pandas.

## 9. Uso de inteligencia artificial

Durante la realización de la práctica se ha utilizado **ChatGPT** como herramienta de apoyo.

Se ha utilizado principalmente para:

- resolver dudas sobre Python, pandas, Jupyter, PostgreSQL y Docker;
- ayudar a interpretar errores encontrados durante el desarrollo;
- revisar la organización y documentación del proyecto;
- revisar partes del código y proponer formas de comprobar los resultados.

El código y los resultados obtenidos se han revisado durante el desarrollo de la práctica y se ha comprobado la ejecución completa del notebook.
