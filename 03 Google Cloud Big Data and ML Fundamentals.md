
# Big Data and ML on Google Cloud
## Google Cloud Infrastructure
Está basada en 5 locations: **Norte América, Sudamérica, Europa, Asia y Australia**.
Tener múltiples localizaciones de servicios es importante para elegir donde alojar nuestras aplicaciones ya que esto afectará a la disponibilidad, durabilidad y latencia.
Cada una de estas _locations_ se dividen a su vez en _regiones_ y estas también se dividen en _zonas_.
Algunos de los servicios que ofrece GCP soportan alojar los recursos en multi-regiones, como por ejemplo las replicaciones de Cloud Spanner.

## Compute
Google ofrece un montón de servicios de computación.
- Compute Engine: es un IaaS que provee virtualmente computación, almacenamiento y recursos de redes de forma similar a un data center físico.
- Google Kubernetes Engine: corre aplicaciones contanerizadas en un entorno cloud, en lugar de en una máquina virtual individual. (Osea orientado a microservicios)
- App Engine: PaaS totalmente administrado para crear aplicaciones webs y backends de dispositivos móviles escalables. Solo tienes que subir el código y Google administra la disponibilidad de la app.
- Cloud Functions: FaaS (functions as a service) Como AWS Lambda.
- Cloud Run: plataforma de procesamiento administrada que te permite ejecutar contenedores directamente sobre la infraestructura escalable de Google.

## Storage
En la computación en la nube, las limitaciones del procesamiento no están atadas al almacenamiento de los discos.  Podrías correr una base de datos en una instancia de Compute Engine (que esta si tendría almacenamiento limitado a la cantidad de gb de los discos), o por otro lado usar servicios totalmente gestionados de almacenamiento y bases de datos. Algunos de estos servicios (que dependen del caso de uso) son: Cloud Storage, Cloud BigTable, CloudSQL, Cloud Spanner, Firestore y BigQuery.

Podemos diferenciar dos tipos de datos. Los estructurados: que estan altamente organizados y formateados de tal manera que se pueden buscar fácilmente en bbdd relacionales. Los no estructurados no tienen formato ni organización predefinidos.

Cloud Storage es un servicio gestionado por Google para guardar datos no estructurados. A diferencia de otras arquitecturas, designa los datos como unidades distintas, agrupadas con metadatos y un identificador único. Estas unidades se denominan objetos y se guardan en contenedores llamados 'buckets', que estan asociados con un proyecto y estos proyectos pueden o no pertenecer a una organización que los agrupa.

Cloud Storage tiene 4 clases primarias:
- Standard Storage: datos que accedes de forma frecuente. También es util para guardar datos durante un periodo corto de tiempo.
- Nearline Storage: datos a los que accedes de forma poco frecuente, 1 vez al mes o menos. 
- Coldline Storage: parecido a Nearline Storage, 1 vez cada 3 meses
- Archive Storage: una vez al año. (duración minima de 365 días)
Conforme menos debamos acceder a esos datos, más barato sera este tipo de almacenamiento.

De forma alternativa, si tenemos datos estructurados el uso de los diferentes servicios vendría dado por el siguiente diagrama:

<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/imgs/storageporcasodeuso.png?raw=true" /> 





# Data Engineering for Streaming Data

## Message Orientated Architectures

Hoy en día los datos pueden ser enviados desde varios dispositivos y de diferentes formas. Además, estos datos pueden venir muy rápido y con gran volumen por lo que debemos asegurarnos de que los servicios sean seguros, confiables y funcionen como esperamos.
Google ofrece un servicio para manejar arquitecturas distribuidas orientadas a mensajes a gran escala, **Pub/Sub**.
Es un mensaje de mensajería distribuida que puede recibir mensajes de varios dispositivos como eventos de gaming, dispositivos IoT y flujos de aplicaciones. 

Una arquitectura end-to-end de streaming podría verse algo así:

<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/imgs/arqpubsub.png?raw=true" /> 

Pub/Sub te permite crear sistemas de productores y consumidores de eventos, llamados **publicadores** y **suscriptores**. Los publicadores se comunican con los suscriptores de forma asíncrona mediante la transmisión de eventos, en lugar de llamadas de procedimiento remoto (RPC) síncronas.

Los publicadores envían eventos al servicio de Pub/Sub sin tener en cuenta cómo o cuándo se deben procesar. Luego, Pub/Sub entrega eventos a todos los servicios que reaccionan a ellos

## Designing streaming pipelines with Apache Beam

Después de que los mensajes sean capturados de una fuente de streaming necesitas enviar los datos a un data warehouse para el análisis. Aquí es donde entra Dataflow.
Dataflow crea una pipeline para procesamiento en batch y en streaming. Procesar en este caso se refiere a los pasos de extraer, transformar y cargar los datos.  
Apache Beam modelo de programación open source para definir y ejecutar pipelines de procesamiento de datos.
Es unificado, ya que usa un único modelo de programación para batch y streaming.
Es portátil, ya que puede funcionar en múltiples entornos de ejecución como Dataflow o Apache Spark.
Es extensible, ya que te permite escribir y compartir tus propias librerías de conectores y transformaciones.
Además, Apache Beam también ofrece plantillas de pipelines que se pueden dividir en streaming, batch y utility.

## Implementing streaming pipelines on Cloud Dataflow.

Apache Beam se usa para crear pipelines, pero el siguiente paso es identificar que dispositivo de ejecución las implementará. Esto nos lleva a Dataflow, un servicio totalmente autogestionado por Google para ejecutar pipelines de Beam en GCP. No necesitas gestionar los recursos  (serverless) ya que dataflow posee autoescalado (no-ops) para cumplir los requerimientos de las pipelines.

Cuando Dataflow recibe un job, empieza optimizando el grafo de ejecución de la pipeline para eliminar ineficiencias. Luego planifica el trabajo distribuido a nuevos workers y escala según necesario. Después se "sana" de cualquier fallo de los workers y automáticamente rebalancea las cargas para que los workers trabajen de la forma más eficiente posible. Y por último, saca los datos como output, por ejemplo a BigQuery.


# Big Data with BigQuery

BigQuery es un datawarehouse totalmente gestionado , osea que no necesitamos encargarnos de la infraestructura (serverless), la seguridad o el despliegue del mismo. Además de ser un datawarehouse nos proporciona un entorno de análisis mediante consultas SQL. Tiene un modelo de pago flexible, podemos pagar por lo que usamos o contratar cuotas de uso. Este servicio también gestiona la encriptación de los datos.

### Storage and analytics

Podemos usar datos internos que ya estén en BigQuery o hacer el traslado desde otros servicios de GCP e incluso desde otros proveedores de cloud.
Una vez que los datos están en BQ, automáticamente se replican, se hacen back-ups y se autoescala según necesidades. También podemos hacer queries de datos que se encuentren en servicios externos de GCP como CloudStorage, y bbdd como Cloud Spanner o Cloud SQL sin necesidad de ingestarlos antes.
BigQuery permite realizar análisis ad hoc, análisis geoespaciales, crear modelos de ML y crear dashboards de BI, entre otros.

### BigQuery ML
Como hemos adelantado anteriormente, BigQuery también soporta la creación de modelos de ML supervisados y no supervisados.
El primer paso sería definir el modelo con una sentencia de este tipo:

```sql
CREATE MODEL {nombre}
OPTIONS 
	(model_type = '{modelo}', labels = ['{etiquetas}']) AS
WITH {dataset/tabla} AS (
SELECT ...{consulta}
)
```

y para predecir usaríamos la sentencia _ml.PREDICT_, por ejemplo:

```sql
SELECT 
predicted_num_trips, num_trips, trip_data,
FROM
ml.PREDICT(MODEL `{modelname}`, (WITH bike_data AS
	SELECT ... {consulta})
```

Adicionalmente también podemos evaluar el modelo usando la orden ML.EVALUATE tal que así:

```sql
SELECT
	roc_auc, 
	accuracy,
	precision,
	recall,
FROM
	ML.EVALUATE(MODEL `{nombre_modelo}`)
```

También tiene opciones de MLOps, como importar modelos de tensorflow, exportar modelos o modificar los hiperparámetros del mismo (esto podemos hacerlo manualmente o dejar que lo haga automáticamente.
Algunas de las opciones de tratamiento de datos previas a la creación del modelo que nos ofrece BQ de forma automática es one-hot encoding de vars categóricas o splitting en train y test.

### Fases de un proyecto de BQ ML
- ETL into BQ
- Seleccionar y preprocesar las features
- Crear el modelo en BQ
- Evaluar la performance del mismo
- Realizar predicciones con el modelo

### Comandos clave de BQ ML

<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/imgs/bqmlcommands.png?raw=true" /> 
# Machine Learning Options on Google Cloud

### Options to build ML Models
- BigQuery ML. Mediante consultas SQL
- API's ya construidas. Usar modelos ya creados por google para tareas específicas.
- AutoML. Solución no-code para construir los modelos en VertexAI
- Custom Training. Crear un entorno de ML y tener el control absoluto

<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/imgs/gcpmloptions.png?raw=true" /> 

### Pre-builts API's
Si no tenemos suficientes datos para crear un modelo aceptable, podemos hacer uso de estas API's ya construidas entrenadas con muchísimos datos por Google que podemos fine-tunear para usar en nuestro caso de uso. Algunos ejemplos son los siguientes:
- Speech-to-text API: para convertir audio a texto
- Cloud Natural Language API: reconoce partes de un texto y análisis de sentiminetos
- Cloud Translation API: convierte texto de un idioma a otro
- Text-to-speech API: convierte texto a audio
- Vision API: reconocimiento de contenido en imágenes
- Video Intelligence API: reconoce movimientos y acciones en vídeos


### AutoML
El objetivo de autoML es ahorrar trabajo manual y repetitivo a los data scientist como tunear hiperparámetros y comparar diferentes modelos. Para esto, dos tecnologías son vitales.
La primera es transfer learning, una técnica de ML que se aprovecha del conocimiento adquirido por un modelo entrenado en una tarea para mejorar el rendimiento de otra tarea relacionada. En lugar de entrenar un modelo desde cero, se utiliza un modelo preentrenado y se ajusta a la nueva tarea específica.
La segunda es _Neural Architecture Search_ que hace un ensamblado de modelos de ML y elige la mejor opción.
AutoML es una herramienta no-code, y soporta 4 tipos de datos: imágenes, tabulares, texto y vídeos


### Custom Training
Podemos programar nuestro propio modelo de ML con _Vertex AI Workbench_, se trata de un entorno de desarrollo para ciencia de datos desde la exploración hasta el entrenamiento y el despliegue.
Antes de empezar a programar, necesitamos determinar que tipo de entorno queremos para el entrenamiento de nuestro modelo. Tenemos dos opciones: un contenedor pre-construido o uno personalizado.
Si nuestro entrenamiento necesita de una plataforma como TensorFlow, Pytorch, Scikit-learn, XGBoost o Python, un contener pre-construido seguramente sea la mejor opción.


### VertexAI
Existen un montón de retos alrededor de llevar modelos de ML a producción, retos que requieren de escalabilidad, montorización, CI/CD y despligue. Más tarde, también surgen retos en la facilidad de uso del mismo. Algunas herramientas del mercado requieren habilidades avanzadas de programación, que pueden desviar a los data scientist de la configuración del modelo.  Y sin un flujo de trabajo unificado, los data scientist pueden tener dificultades encontrando las herramientas que necesitan.
La solución de Google a muchos de estos retos de producción y de uso es VertexAI, una plataforma unificada que brinda todos los componentes del ecosistema y flujo de trabajo del ML juntos. Pero, ¿qué significa exactamente plataforma unificada?
En este caso, significa tener una única experiencia digital para crear, desplegar, manejar modelos en el tiempo y escalarlos.
