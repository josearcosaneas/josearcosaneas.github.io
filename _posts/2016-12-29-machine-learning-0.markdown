---
layout: post
title:  "Introducción al aprendizaje automático (Machine Learning) -    1 "
date:   2016-12-29 22:06:44 +0100
categories: Python R MachineLearningSupervised
---

Este post será una breve introducción al aprendizaje automático. No vamos a
entrar en conceptos matemáticos, a no ser que sea necesario, y tampoco a
describir en profundidad ninguno de los algoritmos que vamos a usar. Existe
mucha documentación en español, y en sobre todo en inglés, para él que quiera
entrar más a fondo en el tema.

Actualmente sea puesto muy de moda el aprendizaje automático, del inglés
**Machine Learning**, esta técnica  permite crear pratones de
características conocidas y también de características desconocidas a priori.

El *aprendizaje automático* es una rama de la inteligencia artificial que tiene
como objetivo desarrollar técnicas que permitan a las máquinas aprender. Se ve
el aprendizaje automático como un proceso de inducción del conocimiento.

El aprendizaje automático se divide en dos extensas áreas: **el aprendizaje
supervisado**, y el **aprendizaje no supervisado**.

El *aprendizaje automático supervisado* extrae conocimiento de las
características de datos clasificados previamente para crear un modelo, un
conjunto de patrones, para poder clasificar nuevos datos.
El *aprendizaje automático no supervisado* usa datos que no estan etiquetados
(clasificados), el objetivo de esta técnica es encontrar alguna forma de
organizarlos. Una técnica muy común en el aprendizaje automático no supervisado
es el **clustering**.

Para trabajar con técnicas de aprendizaje automático existen multitud de
lenguajes, de librerías y de programas que facilitan codificar una solución al
nuestro problema.
Como lenguajes de programación cabe destacar *Python* y *R*,
aunque también es muy común usar *C* (o *C++*) o *JAVA*.
Si usas Python te recomiendo visitar la página de [scikit-learn].
Existen multitud de técnicas para aplicar aprendizaje automático a problemas.
En este blog intentaré explicar, poco a poco, algunas de técnicas más comunes.

Quizás, el más sencillo y también el más usado sea: el aprendizaje automático
supervisado. Dedicaremos este post a conocer mejor esta técnica.

## Aprendizaje automático supervisado

Podemos ver el *aprendizaje automático supervisado* como una técnica para
extraer información de un conjunto de datos clasificados previamente, para
crear patrones con los que podamos predecir la clase a la que pertenecen
otros datos, o sea, **clasificarlos**.

Básicamente, podemos describir el aprendizaje automático supervisado con los
siguientes pasos:

1.  Preparar y dividir el conjunto de entrenamiento (*data set*) en dos
conjuntos; el *conjunto de entrenamiento* y *conjunto de test*. El conjunto de
entrenamiento serán el conjunto de datos previamente clasificados. El proceso
de preparar los datos puede incluir el proceso de normalización de los datos o
un proceso conocido como *validación cruzada*. Estos procesos pretender obtener
resultados más precisos y evitar el *sobreajuste*.

2.  Elegir un algoritmo de aprendizaje automático como puede ser; *K-nn*,
*SVM*, *árboles de decisión*, *Rochio*, ...
Una vez elegido el algoritmo que mejor se ajuste al problema, entrenaremos un
modelo con el algoritmos seleccionado y el conjunto de entrenamiento. Esto
generará un modelo, un conjunto de patrones, con el que predecir nuevos datos.

3.  Ahora con el modelo *clasificamos* o *predecimos* los valores del conjunto de
test y evaluamos las propiedades del modelo.


En este post veremos un ejemplo típico del aprendizaje automático supervisado.
Para el problema que vamos a resolver vamos a utilizar un algoritmo simple y
muy usado: **K-NN**.

### K-NN

El algoritmo K-NN tiene como idea básica  que un dato nuevo se clasificará
en la clase más frecuente a la que pertencen sus K vecinos más cercanos. Esta
idea es muy sencilla, también lo es su implementación, aunque en nuestro caso
aplicaremos el algoritmo a través de las implementaciones que ya existen en
lenguajes como Python o R.

En K-NN los ejemplos de entrenamiento son vectores de un espacio
multidimensional, cada ejemplo tiene *n* atributos y *m* clases. El espacio se
particiona en regiones en función de los valores de atributo y las clases
(etiquetas) de los ejemplos de entrenamiento.

Un nuevo dato será clasificado en una clase si está es la clase más frecuente
entre los **K** ejemplos de entrenamiento más cercanos. Para el cálculo de la
distancia es frecuente usar *la distancia euclidea* o *la distancia manhattan*.
Durante la fase de entrenamiento de este algoritmo se almacenan los vectores
característicos y las clases de los ejemplos de entremiento.
La clasificación consiste en evaluar un ejemplo como un vector de
características y calcular la distancia entre los vectores almacenados y el
nuevo vector y posteriormente se seleccionan los **K** mas cercados.
El nuevo ejemplo será clasificado en la clase a la que más vecinos pertenezcan.
Es un algoritmo sencillo que con el que se obtienen buenos resultados.

A continuación, veremos un ejemplo sencillo con dos lenguajes de programación
distintos. El conjunto de datos para el ejemplo podemos encontrarlo en internet
en la web del [UCI Machine Learning Repository].
En nuestro caso usaremos el conjunto de datos (*data set*) más usado [iris].
Aunque podríamos leerlo directamente, *R* y *Python* ya implementan funciones
que los leen fácilmente.


El problema es sencillo. Tenemos una serie de datos con cuatro características
y clasificados en tres clases. El objetivo es crear un clasificador capaz de
recibir nuevos datos y clasificarlo en la clase que les corresponda en función
de los valores de sus características. Para eso usaremos el algoritmo explicado
ateriormente.

### Código K-NN con R

**R** es un lenguaje de programación sencillo y que implementa muchas funciones
que facilitan la tarea de programación.
Junto a su IDE, **RStudio**, es una herramienta muy importante en el análisis de
datos en general.
El código que se muestra a continuación realiza el procedimiento explicado
anteriormente.
En primer lugar, cargaremos la función K-NN y el conjunto de datos de iris.
Posteriormente dividiremos, de forma aleatoria, el conjunto en dos; el conjunto
de entrenamiento y el conjunto de test. Una vez hecho esto predecimos el
resultado que se obtiene al aplicar el conjunto de test y el conjunto de
entrenamiento a un algorimto K-NN variando el numero de vecimos desde 1 a 50.
Calcularemos la tasa de error los resultados obtenidos y la almacenaremos para
posteriormente dibujar la gráfica.

{% highlight r %}
library(class)
set.seed(4948493) #Definimos una semilla

ir_sample <- sample(1:nrow(iris),size=nrow(iris)*.7)
ir_train <- iris[ir_sample,] # Seleccionamos el 70% para entrenamiento
ir_test <- iris[-ir_sample,] # Seleccionamos el 30% para test
iris_error <- numeric() # variable numérica

# Applicamos K-NN con k desde 1 hasta 50
for(i in 1:50){
  predict <- K-NN(ir_train[,-5],ir_test[,-5],ir_train$Species,k=i)
  iris_error <- c(iris_error, 1 - mean(predict==ir_test$Species))
}

# Dibujamos la tasa de error
plot(iris_error,type="l",ylab="Tasa de error",
    xlab="K",main="Tasa de error en función de K ")
{% endhighlight %}
El código devolverá la siguiente gráfica.

![Jekyll](/assets/images/tasa-error-R-1.png)


Como podemos ver la codificación en R es muy sencilla. Con
`predict==ir_test$Species` devolvemos un vector de `True` o `False`, que podemos
ver como *0's* y *1's*. Y con `mean(predict==ir_test$Species)` obtenemos la
media, que no es más que el porcentaje de `True` que hay en el vector, o el porcentaje de
acierto, la **Precisión**. Y por tanto, `1 - mean(predict==ir_test$Species)` nos devolverá
el porcentaje de `False`.
Por último con la función  `c(iris_error, mean())` almacenamos el resultado de
`mean()` en `iris_error`. Posteriormente dibujamos la gráfica con la función *plot*.


### Código K-NN con Python

El siguiente código en Python realiza la misma función que el anterior en R.
A diferencia de R aquí haremos uso de la librería **Sklearn**.
No volveré a explicar lo mismo, en lugar de eso, comentaré el código para
explicar las partes que puedan resultar más complejas.

{% highlight python %}
# -*- coding: utf-8 -*-
import numpy as np
import pylab as pl
from sklearn import datasets
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import precision_score

training, aux, numero_vecinos = 0.3, 0, 50
np.random.seed(4948493)
iris = datasets.load_iris()
iris_X, iris_y = iris.data, iris.target
# De forma aleatorio escogemos un conjunto para test y otro para training
lt = np.round(training * len(iris_X))
indices = np.random.permutation(len(iris_X))
iris_X_train, iris_y_train = iris_X[indices[:-lt]], iris_y[indices[:-lt]]
iris_X_test, iris_y_test = iris_X[indices[-lt:]], iris_y[indices[-lt:]]
# Las clases aqui vienen etiquetadas con 0, 1, 2
clase1_test, clase2_test, clase3_test, mediaclases = [], [], [], []

for i in range(1, numero_vecinos):
    neigh = KNeighborsClassifier(i, weights='uniform')
    y_ = neigh.fit(iris_X_train, iris_y_train).predict(iris_X_test)
    yy_ = neigh.fit(iris_X_train, iris_y_train).predict(iris_X_train)
    a = 1 - precision_score(iris_y_test, y_, average=None)
    clase1_test.append(a[0])
    clase2_test.append(a[1])
    clase3_test.append(a[2])
    aux = float(a[0] + a[1] + a[2]) / (3)
    mediaclases.append(aux)

pl.plot(range(len(range(
        1, numero_vecinos))), mediaclases, linewidth=1.5, label="mediaclases")
pl.legend(loc="best")
pl.xlim(0, numero_vecinos)
pl.ylim(0, .15)
nombre = "Tasa de error :1-" + str(numero_vecinos)
pl.title(nombre)
pl.show()
{% endhighlight %}

![Jekyll](/assets/images/tasa-error-Python-1.png)


El resultado no es el mismo que con el código *R* porque el conjunto de
entranamiento y de test varían.
La técnica que se mencionó anteriormente, *validación cruzada*, itera varías
veces la solución del problema variando en cada iteracion el conjunto de
test y de training con lo que se obtiene valores diferentes en cuanto a preción
y error medio.
La media de las distintas preciones y la media de los errores será considerado
un valor representativo del clasificador.
La validación cruzada es una técnica muy usada ya que hace que la solución de
un clasificador no sea tan dependiente del conjunto de entrenamiento y de test
que se elejan.

Si forzamos a que los datos de training y de test sean los mismos en ambos programas
obtendremos gráficas similares.

![Jekyll](/assets/images/tasa-error-Python-2.png)



[scikit-learn]: http://scikit-learn.org/
[iris]: https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data
[UCI Machine Learning Repository]: https://archive.ics.uci.edu/ml/index.html
