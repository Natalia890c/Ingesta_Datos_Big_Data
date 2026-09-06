# Proyecto de Ingesta de Datos Big Data

## Descripción

Este proyecto implementa la etapa de **ingestión de datos** de un proyecto de Big Data, utilizando Python, una fuente de datos accesible mediante API, SQLite como sistema de almacenamiento y GitHub Actions para automatizar la ejecución del proceso.

La fuente utilizada corresponde al dataset **Online Retail**, obtenido mediante la API pública de Kaggle.

El proceso permite:

1. Obtener los datos desde la API.
2. Almacenar el archivo fuente en formato CSV.
3. Cargar los datos en una base de datos SQLite.
4. Realizar controles de calidad.
5. Aplicar reglas de validación y limpieza.
6. Transformar los datos para su utilización analítica.
7. Construir un modelo dimensional tipo estrella.
8. Generar evidencias de auditoría.
9. Generar una muestra de los datos mediante Pandas.
10. Automatizar todo el proceso mediante GitHub Actions.



## Fuente de datos

**Dataset:** Online Retail

**Fuente:** Kaggle

**API utilizada:**

```text
https://www.kaggle.com/api/v1/datasets/download/vijayuv/onlineretail
```

La descarga se realiza mediante una solicitud HTTP utilizando la biblioteca `requests`.

El dataset original contiene:

* **541.909 registros**
* **8 columnas**

Columnas:

```text
InvoiceNo
StockCode
Description
Quantity
InvoiceDate
UnitPrice
CustomerID
Country
```

La regla principal utilizada para identificar facturas canceladas es que el campo `InvoiceNo` comienza con la letra `C`.



## Tecnologías utilizadas

* Python 3.12
* Pandas
* Requests
* SQLite
* Git
* GitHub
* GitHub Actions

SQLite no requiere una dependencia externa porque forma parte de la biblioteca estándar de Python.



## Arquitectura del proceso

El flujo implementado es:

```text
API de Kaggle
     │
     ▼
OnlineRetail.csv
     │
     ▼
SQLite - stg_online_retail
     │
     ▼
Control de calidad
     │
     ▼
Validación de reglas
     │
     ▼
ventas_limpias
     │
     ├──────────────┐
     ▼              ▼
Dimensiones     FactVentas
     │              │
     └───────┬──────┘
             ▼
    Validación final
             │
             ▼
       Evidencias
             │
             ▼
      GitHub Actions
```



## Modelo de datos

El proyecto utiliza un modelo dimensional tipo estrella.

### Dimensiones

#### DimFecha

Contiene la información correspondiente a las fechas de las ventas.

Registros generados:

```text
305
```

Rango:

```text
2010-12-01 a 2011-12-09
```

#### DimCliente

Contiene los clientes identificados mediante `CustomerID`.

Registros:

```text
4.338
```

#### DimProducto

Contiene los productos identificados mediante `StockCode`.

Registros:

```text
3.922
```

### Tabla de hechos

#### FactVentas

Contiene el detalle de las ventas válidas.

Registros:

```text
530.104
```

La granularidad de la tabla corresponde a una línea de producto dentro de una transacción.

El campo `TotalVenta` se calcula mediante:

```text
TotalVenta = Quantity × UnitPrice
```



## Reglas de limpieza

Antes de construir `FactVentas` se aplican las siguientes reglas:

* La factura no debe estar cancelada.
* `Quantity` debe ser mayor que cero.
* `UnitPrice` debe ser mayor que cero.

Los registros que no cumplen estas reglas son excluidos de las ventas válidas.

Resultados:

```text
Registros originales:          541.909
Registros válidos:             530.104
Registros excluidos:            11.805
```

Los registros sin `CustomerID` no son descartados, ya que la ausencia de cliente no impide considerar válida una venta.


## Resultados de calidad

La auditoría inicial identificó:

```text
Registros totales:              541.909
CustomerID nulos:               135.080
Description nulas:                1.454
Cantidad no positiva:            10.624
Precio no positivo:               2.517
Facturas canceladas:              9.288
Grupos duplicados:                4.879
```

Estas categorías pueden presentar intersecciones; por lo tanto, sus valores no deben sumarse para calcular el total de registros inválidos.



## Validación de la carga

La comparación entre la fuente y SQLite produjo:

```text
Registros API:                  541.909
Registros SQLite:               541.909
Diferencia:                           0
```

Resultado:

```text
APROBADO
```

La tabla de hechos también fue comparada con los registros válidos:

```text
Registros válidos:              530.104
Registros FactVentas:           530.104
Diferencia:                           0
```

Resultado:

```text
APROBADO
```



## Validación final del modelo

La validación final produjo:

```text
stg_online_retail:               541.909
ventas_limpias:                  530.104
DimFecha:                            305
DimCliente:                       4.338
DimProducto:                      3.922
FactVentas:                      530.104
```

Integridad referencial:

```text
DateID inválido:                     0
ProductoID inválido:                 0
ClienteID inválido:                  0
```

Reglas de calidad después de la transformación:

```text
Cantidades <= 0:                    0
Precios <= 0:                        0
Facturas canceladas:                0
TotalVenta incorrecto:              0
```

Valor total de ventas:

```text
10.666.684,54
```

---

## Evidencias generadas

El proyecto genera diferentes archivos de evidencia.

### Base de datos

```text
data/retail.db
```

Contiene las tablas de staging, transformación, dimensiones y hechos.

### Muestra con Pandas

```text
data/muestra_ingestion.csv
```

La muestra contiene:

```text
100 registros
8 columnas
```

Fue generada utilizando Pandas a partir de `FactVentas`.

### Auditoría de extracción

```text
data/auditoria_extraccion.json
```

Registra información sobre la fuente y los registros obtenidos.

### Auditoría de carga

```text
data/auditoria_carga.json
```

Compara los registros del CSV con los registros almacenados en SQLite.

### Auditoría de calidad

```text
data/auditoria_calidad.json
```

Registra los resultados de los controles de calidad.

### Auditoría final

```text
data/auditoria_final.json
```

Contiene la validación final del modelo analítico.

### Auditoría TXT

```text
src/static/auditoria/ingestion.txt
```

Contiene la comparación entre los registros extraídos y almacenados:

```text
API:       541.909
SQLite:    541.909
Diferencia:     0

Válidos:   530.104
FactVentas:530.104
Diferencia:     0

Estado: APROBADO
```

---

## Estructura del proyecto

```text
Ingesta_Datos_Big_Data/
│
├── .github/
│   └── workflows/
│       └── bigdata.yml
│
├── data/
│   ├── OnlineRetail.csv
│   ├── retail.db
│   ├── muestra_ingestion.csv
│   ├── auditoria_extraccion.json
│   ├── auditoria_carga.json
│   ├── auditoria_calidad.json
│   └── auditoria_final.json
│
├── src/
│   ├── ingestion.py
│   │
│   ├── static/
│   │   └── auditoria/
│   │       └── ingestion.txt
│   │
│   └── db/
│       ├── database.py
│       ├── quality.py
│       ├── validation.py
│       ├── transform.py
│       ├── dimensions.py
│       ├── fact.py
│       ├── validation_final.py
│       ├── indexes.py
│       ├── muestra.py
│       └── auditoria_txt.py
│
├── .gitignore
├── README.md
├── requirements.txt
└── setup.py
```



## Instalación

### 1. Clonar el repositorio

```bash
git clone URL_DEL_REPOSITORIO
```

Ingresar al proyecto:

```bash
cd Ingesta_Datos_Big_Data
```

### 2. Crear entorno virtual

En Windows:

```powershell
python -m venv .venv
```

Activar el entorno:

```powershell
.venv\Scripts\activate
```

### 3. Instalar dependencias

```powershell
pip install -r requirements.txt
```

Las dependencias principales son:

```text
pandas
requests
```



## Ejecución local

El proceso puede ejecutarse mediante los siguientes scripts:

```powershell
python src/ingestion.py
python src/db/database.py
python src/db/quality.py
python src/db/validation.py
python src/db/transform.py
python src/db/dimensions.py
python src/db/fact.py
python src/db/validation_final.py
python src/db/indexes.py
python src/db/muestra.py
python src/db/auditoria_txt.py
```

Al finalizar se generan la base SQLite, la muestra y las auditorías correspondientes.



## Automatización con GitHub Actions

El proyecto utiliza GitHub Actions para automatizar el proceso de ingestión.

El workflow se encuentra en:

```text
.github/workflows/bigdata.yml
```

El workflow se ejecuta automáticamente cuando se realiza un `push` a la rama `main`.

También puede ejecutarse manualmente mediante `workflow_dispatch`.

El flujo automatizado realiza:

```text
Instalación de dependencias
        ↓
Extracción desde API
        ↓
Carga en SQLite
        ↓
Control de calidad
        ↓
Validación
        ↓
Transformación
        ↓
Creación de dimensiones
        ↓
Creación de FactVentas
        ↓
Validación final
        ↓
Creación de índices
        ↓
Generación de muestra
        ↓
Generación de auditoría
        ↓
Verificación de evidencias
        ↓
Publicación de artifacts
```

### Verificación de la ejecución

Para comprobar una ejecución:

1. Ingresar al repositorio en GitHub.
2. Seleccionar la pestaña **Actions**.
3. Seleccionar el workflow **Big Data - Ingesta y Evidencias**.
4. Abrir la ejecución correspondiente.
5. Revisar que todos los pasos finalicen correctamente.
6. Consultar los artifacts generados al finalizar el workflow.

Entre los artifacts se encuentran:

```text
retail.db
muestra_ingestion.csv
ingestion.txt
auditorias JSON
```



## Resultado

El proceso de ingestión fue implementado y validado utilizando una fuente accesible mediante API, almacenamiento en SQLite, procesamiento con Python y Pandas, generación de evidencias y automatización mediante GitHub Actions.

La auditoría final confirma que:

```text
Registros extraídos:             541.909
Registros almacenados:            541.909
Diferencia:                            0

Registros válidos:               530.104
Registros en FactVentas:         530.104
Diferencia:                            0

Estado final:                    APROBADO
```