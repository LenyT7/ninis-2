# Interfaz Streamlit — Análisis de jóvenes NiNi en Pasto

## Descripción

Este proyecto adapta a una interfaz web con **Streamlit** el procesamiento de microdatos de la Gran Encuesta Integrada de Hogares (GEIH) que aparece en el archivo `codigo_pob_nini.py`.

El flujo de análisis contemplado en el código fuente es:

1. Leer archivos ZIP organizados por año.
2. Identificar dentro de cada ZIP los módulos de características generales, ocupados/fuerza de trabajo y, cuando exista, vivienda y hogares.
3. Filtrar la cabecera municipal de Pasto mediante `AREA = 52` y `CLASE = 1`.
4. Seleccionar jóvenes entre 14 y 28 años.
5. Construir el indicador NiNi: persona que no está ocupada y no asiste a una institución educativa, de acuerdo con las variables detectadas en los microdatos.
6. Construir variables complementarias: sexo, edad, edad al cuadrado, nivel educativo y factor de expansión.
7. Cuando el módulo de vivienda/hogares contiene las variables necesarias, calcular el ingreso per cápita del hogar y clasificarlo en quintiles.
8. Preparar una base consolidada y estimar un modelo Logit, efectos marginales promedio y diagnóstico de multicolinealidad mediante VIF.

> **Alcance del código de origen:** los años definidos actualmente son 2020, 2021, 2022, 2023 y 2024. No se debe presentar el análisis como si cubriera 2025 hasta incorporar y validar esos microdatos.

## Estructura esperada de los datos

El código original espera una carpeta de datos con subcarpetas por año y archivos ZIP dentro de cada una:

```text
datos_geih/
├── 2020/
│   ├── archivo_enero.zip
│   └── ...
├── 2021/
│   └── ...
├── 2022/
│   └── ...
├── 2023/
│   └── ...
└── 2024/
    └── ...
```

Cada ZIP debe contener los módulos GEIH correspondientes. El programa busca archivos cuyos nombres permitan reconocer los módulos de características generales, ocupados/fuerza de trabajo y vivienda/hogares. Los módulos y nombres de variables pueden cambiar entre versiones de la GEIH; por eso, la detección automática debe verificarse con los archivos reales.

## Instalación

Se recomienda utilizar Python 3.10 o superior y crear un entorno virtual.

### Windows

```bash
py -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Linux o macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Ejecución

Cuando el archivo de la interfaz se encuentre creado —por ejemplo, `app.py`—, ejecutar:

```bash
streamlit run app.py
```

La interfaz debería permitir seleccionar o cargar los ZIP, elegir los años que se procesarán, ejecutar el procesamiento y consultar/descargar los resultados. Estas funciones corresponden al diseño previsto de la interfaz; no están implementadas por el archivo `codigo_pob_nini.py` por sí solo.

## Dependencias

Las dependencias directas se encuentran en `requirements.txt`.

- `streamlit`: interfaz web.
- `pandas` y `numpy`: lectura, limpieza y transformación de datos.
- `statsmodels`: estimación Logit, efectos marginales y VIF.
- `pyreadstat`: lectura de archivos SPSS `.sav` si están incluidos en los ZIP.

`zipfile`, `glob`, `os`, `unicodedata` y `warnings` pertenecen a la biblioteca estándar de Python y no requieren instalación separada.

## Consideraciones metodológicas y de validación

Antes de interpretar o publicar resultados, se deben revisar estos puntos del código original:

- **Identificación geográfica:** el código asigna `AREA = 52` o `CLASE = 1` cuando esas columnas no existen. Ese comportamiento puede clasificar registros sin evidencia geográfica suficiente. En una versión de producción, es preferible detener el procesamiento y advertir que faltan las variables.
- **Definición de NiNi:** verificar los códigos válidos de la variable de asistencia educativa y del estado de ocupación para cada año y módulo. La regla implementada es `OCI != 1` y asistencia igual a `2`; no debe asumirse que los códigos conservan el mismo significado sin revisar los diccionarios de cada año.
- **Cruce de módulos:** validar que las llaves de identificación sean suficientes, no estén vacías y produzcan relaciones esperadas. Revisar duplicados y registros no emparejados.
- **Ingresos del hogar:** comprobar que la variable detectada represente efectivamente el ingreso total del hogar y que el número de personas corresponda al mismo hogar y periodo. La selección automática por fragmentos del nombre de variable requiere validación.
- **Quintiles:** el código calcula quintiles sobre el ranking de los registros consolidados. Para análisis poblacional, debe evaluarse si corresponde calcularlos con ponderación del factor de expansión y dentro del periodo pertinente.
- **Factor de expansión:** conservar y utilizar el factor de expansión de acuerdo con el diseño muestral de la GEIH. No reemplazar automáticamente valores faltantes por uno sin documentar y evaluar el efecto.
- **Modelo Logit:** revisar categorías de referencia, observaciones faltantes, separación, convergencia, errores estándar y diseño muestral. El VIF es un diagnóstico auxiliar y no demuestra por sí solo la validez del modelo.
- **Manejo de errores:** el código original omite excepciones durante el procesamiento de ZIP. La interfaz debería mostrar un registro de archivos procesados, omitidos y fallidos, con el motivo correspondiente.

## Archivos del proyecto

```text
proyecto/
├── app.py
├── codigo_pob_nini.py
├── requirements.txt
├── README.md
└── datos_geih/
    ├── 2020/
    ├── 2021/
    ├── 2022/
    ├── 2023/
    └── 2024/
```

La estructura es orientativa. Si la interfaz permite cargar archivos directamente, la carpeta `datos_geih/` puede no ser necesaria.

## Nota

Este README documenta el alcance que se observa en `codigo_pob_nini.py` y una propuesta de ejecución con Streamlit. No certifica que las reglas de identificación, los cruces o las estimaciones sean estadísticamente correctos: deben validarse con los diccionarios y la documentación metodológica de la GEIH de cada año.
