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
En la computación en la nube, las limitaciones del procesamiento no están atadas al almacenamiento. 