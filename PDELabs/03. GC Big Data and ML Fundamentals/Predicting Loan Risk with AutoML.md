En este laboratorio aprenderemos a:
- Subir un dataset a VertexAI
- Entrenar un modelo de ML con AutoML
- Evaluar la performance del modelo
- Desplegar el modelo en un endpoint
- Obtener predicciones

# 1) Preparar el conjunto de entrenamiento

En VertexAI vamos a la opción Datasets y crearemos uno llamdo LoanRisk. Seleccionaremos datos tabulares y como objetivo Regression/classification
A continuación subiremos los datos, en este caso seleccionamos subir csv desde Cloud Storage y en el path seleccionamos

```
spls/cbl455/loan_risk.csv
```

En este paso podemos generar estadísticas automáticamente de nuestro dataset como el % de missing values por columna o el número de valores distintos por columna.

# 2) Entrenamiento del modelo
Seleccionaremos Train new model y después Other, en objetivo seleccionamos Classification ya que queremos saber si un individuo devuelva un préstamo (0) o si no lo hará (1). Seleccionamos siguiente.
En columna target marcamos Default y siguiente. Ahora nos pedirá el número de horas máxima de entrenamiento y marcamos 1h y dejamos el early-stopping seleccionado. Empezamos el entrenamiento.
Como esto tardará bastante, descargaremos el modelo entrenado para los siguientes pasos.

# 3) Demonstración de la evaluación del modelo
Podemos visualizar las métricas de nuestro modelo entrenado en Model Registry -> Nuestro modelo -> Evaluate

<img src=  "https://github.com/Rubnserrano/PDECert/blob/main/PDELabs/imgs/lab03_metrics.png?raw=true" /> 

También podremos ver la matriz de confusión y la importancia de cada característica.

# 4) Despliegue del modelo
Para desplegar nuestro modelo en un endpoint seleccionaremos: Deploy & test -> Deploy to Endpoint.
Al endpoint lo llamamos LoanRisk y le damos a continuar.
En ajustes del modelo seleccionamos:
- Traffic-splitting options como está
- Tipo de máquina e2-standard-8
- En Explanaibility options -> Feautre attribution
En la siguiente página nos vamos a Model monitoring y seleccionamos:
- Model objectives -> Training data source -> VertexAI dataset
Seleccionamos nuestro dataset y en target column -> Default.
Dejamos las demás opciones como están y le damos al botón de despliegue.
Ya tendremos listo nuestro modelo para realizar predicciones desde un endpoint.

# 5) SML Bearer Token
Para permitir a la pipeline autenticarse y ser autorizada para llamar al endpoint para obtener las predicciones necesitaremos de un Bearer Token.
Nos vamos a la url: https://gsp-auth-kjyo252taq-uc.a.run.app/ donde iniciamos con nuestras credenciales del lab y copiamos el código que se nos genera.

# 6) Obtener las predicciones

Para usar el modelo entrenado necesitaremos crear algunas variables de entorno:

```bash
export AUTH_TOKEN=bearertoken
```

Descargamos y extraemos las assets del lab:

```bash
gcloud storage cp gs://spls/cbl455/cbl455.tar.gz .
tar -xvf cbl455.tar.gz
```

Creamos una variable de entorno llamada endpoint y otra llamada INPUT_DATA_FILE

```bash
export ENDPOINT="https://sml-api-vertex-kjyo252taq-uc.a.run.app/vertex/predict/tabular_classification"
export INPUT_DATA_FILE="INPUT-JSON" 
```

Este input_json tendrá datos de un individuo al que querremos realizar la predicción.
Para hacer la petición hacemos:

```bash
./smlproxy tabular \
  -a $AUTH_TOKEN \
  -e $ENDPOINT \
  -d $INPUT_DATA_FILE
```

y el resultado deberá ser:

```
SML Tabular HTTP Response:
  2022/01/10 15:04:45 {"model_class":"0","model_score":0.9999981}
```