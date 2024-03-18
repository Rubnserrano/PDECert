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