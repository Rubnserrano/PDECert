
En este laboratorio aprenderá a:
- Crear un trabajo Dataflow a partir de una plantilla
- Transmitir una pipeline de datos a BigQuery
- Monitorizar una pipeline de Dataflow en BigQuery
- Analizar resultados con SQL
- Visualizar métricas clave en Looker Studio
### 1) Crear un Dataset de BigQuery
Creamos un dataset que se llame taxirides, podemos hacerlo desde la CloudShell:

```bash
bq --location=us-west1 mk taxirides

```

y creamos una tabla dentro de este dataset

```bash
bq --location=us-west1 mk \
--time_partitioning_field timestamp \
--schema \
ride_id:string,point_idx:integer,latitude:float, longitude:float, \
timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string, \
passenger_count:integer -t
taxirides.realtime
```

O también lo podemos crear mediante la UI de BigQuery.
Tendremos que crear el dataset y la tabla como ya sabemos y editar el schema como texto de la siguiente forma:

```
ride_id:string,
point_idx:integer,
latitude:float,
longitude:float,
timestamp:timestamp,
meter_reading:float,
meter_increment:float,
ride_status:string,
passenger_count:integer
```

En Partition and cluster settings seleccionamos timestamp y creamos la tabla.

### 2) Copiar archivos requeridos del laboratorio
En esta sección moveremos algunos archivos requeridos a nuestro Proyecto.
Ejecutamos lo siguiente en la CloudShell:

```bash
gcloud storage cp gs://cloud-training/bdml/taxisrcdata/schema.json  gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/schema.json
gcloud storage cp gs://cloud-training/bdml/taxisrcdata/transform.js  gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/transform.js
gcloud storage cp gs://cloud-training/bdml/taxisrcdata/rt_taxidata.csv  gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/rt_taxidata.csv
```

Básicamente lo que estamos haciendo es copiar desde un bucket público a nuestro bucket los archivos schema.json, transform.js y rt_taxidata.csv

### 3) Configurar una pipeline de Dataflow
En esta sección setearemos la pipeline de streaming para leer archivos de nuestro Cloud Storage y escribirlos en BigQuery.

En la UI de Dataflow clickamos en _Create Job From Template_ y escribimos **streaming-taxi-pipeline** como nombre del Job. También seleccionamos us-west1 como región. En plantilla seleccionamos _Cloud Storage Text to BigQuery (Stream)_

En la opción _The GCS location of the text you'd like to process_ elegimos el path del csv:

```
gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/rt_taxidata.csv
```

En el schema copiamos la ruta del schema.json y en la tabla de output la ruta de la tabla de BQ:

```
qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/schema.json
qwiklabs-gcp-02-a7a3370660a4:taxirides.realtime
```

Además también debemos especificar las transformaciones que queremos realizar, que seran el archivo js que nos faltaba por meter. También le asignamos un nombre a la UDF JavaScript function name: transform

```
gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/transform.js
```

El archivo transform.js contiene lo siguiente:

```javascript
function transform(line) {
var values = line.split(',');
var custObj = new Object();
custObj.ride_id = values[0];
custObj.point_idx = values[1];
custObj.latitude = values[2];
custObj.longitude = values[3];
custObj.timestamp = values[4];
custObj.meter_reading = values[5];
custObj.meter_increment = values[6];
custObj.ride_status = values[7];
custObj.passenger_count = values[8];
var outJson = JSON.stringify(custObj);
return outJson;
}
```

Seleccionamos el directorio principal para los logs de BigQuery como la carpeta tmp de nuestro bucket.
Por último seleccionamos estas opciones en la ventana de parámetros opcionales:
- Max Workers = 2
- Number of Workers = 1
- Deseleccionar _Use default machine type_
- En _General purpose_ seleccionar la máquina e2-medium
- Lanzar el job

El comando de CloudShell para ejecutar todo esto rápidamente es el siguiente:

```bash
gcloud dataflow flex-template run streaming-taxi-pipeline \
--template-file-gcs-location gs://dataflow-templates-us-west1/latest/flex/Stream_GCS_Text_to_BigQuery_Flex --region us-west1 \
--num-workers 1 --additional-user-labels "" --parameters \
inputFilePattern=gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/rt_taxidata.csv,JSONPath=gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/schema.json,outputTable=qwiklabs-gcp-02-a7a3370660a4:taxirides.realtime,javascriptTextTransformGcsPath=gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp/transform.js,javascriptTextTransformFunctionName=transform,bigQueryLoadingTemporaryDirectory=gs://qwiklabs-gcp-02-a7a3370660a4-bucket/tmp,useStorageWriteApi=false,numStorageWriteApiStreams=0,javascriptTextTransformReloadIntervalMinutes=0,maxNumWorkers=2,workerMachineType=e2-medium
```


### 4) Analizar los datos de los taxis con BigQuery

Simplemente hacemos que nos muestre las 10 primeras filas para comprobar que está todo bien.

```
SELECT * FROM taxirides.realtime
LIMIT 10
```

### 5) Realizar agregaciones para el reporte

```sql
WITH streaming_data AS (
SELECT
  timestamp,
  TIMESTAMP_TRUNC(timestamp, HOUR, 'UTC') AS hour,
  TIMESTAMP_TRUNC(timestamp, MINUTE, 'UTC') AS minute,
  TIMESTAMP_TRUNC(timestamp, SECOND, 'UTC') AS second,
  ride_id,
  latitude,
  longitude,
  meter_reading,
  ride_status,
  passenger_count
FROM
  taxirides.realtime
ORDER BY timestamp DESC
LIMIT 1000
)

# calculate aggregations on stream for reporting:
SELECT
 ROW_NUMBER() OVER() AS dashboard_sort,
 minute,
 COUNT(DISTINCT ride_id) AS total_rides,
 SUM(meter_reading) AS total_revenue,
 SUM(passenger_count) AS total_passengers
FROM streaming_data
GROUP BY minute, timestamp
```

**Este código SQL analiza datos de viajes de taxi en tiempo real. Primero, selecciona los últimos 1000 registros de la tabla de datos de viajes ordenados por tiempo. Luego, calcula el número total de viajes, ingresos totales y pasajeros totales para cada minuto en base a esos datos.**

El código SQL incluye una función de ventana `ROW_NUMBER() OVER()` que genera un número de fila secuencial para cada fila resultante. Esto significa que cada fila tendrá un número de fila único, incluso si las filas comparten el mismo minuto. Por lo tanto, incluso si varias filas tienen el mismo minuto, cada una tendrá un número de fila diferente debido a la función de ventana utilizada en la consulta.

Datos pre-sentencia SQL

<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/PDELabs/imgs/lab01-presql.png?raw=true" /> 

Datos post-sentencia SQL
<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/PDELabs/imgs/lab01-postsql.png?raw=true" /> 

### 6) Cancelar el job de Dataflow

### 7) Crear un dashboard en tiempo real

### 8) Crear un dashboard de series temporales