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

<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/imgs/storageporcasodeuso.jpg?raw=true" /> 