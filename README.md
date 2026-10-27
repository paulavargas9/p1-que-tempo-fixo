Que tempo fixo?
1. Descripción del proyecto

Este proyecto se realiza para la asignatura de Sistemas de Big Data. El objetivo es construir un pipeline completo de datos meteorológicos diarios de la estación de A Coruña / Alvedro durante el período comprendido entre el 1 de enero de 2021 y el 31 de diciembre de 2025.

El proyecto incluye la descarga automatizada de los datos, la inspección del archivo original, la limpieza y validación de los datos, el análisis estadístico, la creación de gráficas, la comparación del rendimiento de distintas operaciones y la carga de los datos en una base de datos PostgreSQL.

2. Estación meteorológica
Nombre: A Coruña / Alvedro.
Indicativo: 08002.
Coordenadas: pendientes de verificar en una fuente oficial.
Período analizado: 2021-2025.
3. Fuente de los datos
Fuente indicada en el enunciado: Meteostat.
Archivo: meteo_alvedro_2021_2025.csv.
URL del archivo: https://github.com/bigdatawirtz/datasets/blob/main/datasets/meteo_alvedro_2021_2025.csv
Fecha de descarga: pendiente de registrar.
Condiciones de uso: pendientes de comprobar según la licencia y los términos aplicables a la fuente de los datos.

El archivo original se conservará sin modificar en datos/raw/. Los datos procesados se guardarán en datos/clean/ en formato CSV y codificación UTF-8.

4. Estructura del proyecto
text
que-tempo-fixo/
├── README.md
├── requirements.txt
├── .gitignore
├── datos/
│   ├── raw/
│   └── clean/
├── notebooks/
└── output/

datos/raw/: contiene el archivo original descargado.
datos/clean/: contiene los datos después de la limpieza.
notebooks/: contiene el notebook con el código y los resultados del proyecto.
output/: almacena las gráficas y otros resultados exportados.
requirements.txt: especifica las bibliotecas de Python necesarias.
.gitignore: indica qué archivos deben excluirse del repositorio.
5. Requisitos e instalación

Se necesita Python y, para la parte de base de datos, Docker.

Las bibliotecas necesarias están indicadas en requirements.txt.

Para crear un entorno virtual de Python:

bash
python -m venv .venv


Para instalar las dependencias:

bash
python -m pip install -r requirements.txt

6. Ejecución del proyecto
Abrir el proyecto en Visual Studio Code.
Activar el entorno virtual de Python.
Instalar las dependencias indicadas en requirements.txt.
Ejecutar el notebook de principio a fin.
Descargar los datos automáticamente desde la URL configurada.
Inspeccionar, limpiar y validar los datos.
Guardar el archivo limpio y generar las gráficas.
Iniciar PostgreSQL mediante Docker y cargar los datos.
Ejecutar las consultas SQL y comprobar sus resultados frente a los obtenidos con pandas.

El proyecto debe poder ejecutarse desde cero con las