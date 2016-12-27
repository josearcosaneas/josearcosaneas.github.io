---
layout: post
title:  "Serialización y persistencia de objetos en Python"
date:   2016-12-26 13:56:44 +0100
categories: Python Serialización Persistencia
---
A la hora de realizar trabajos de gran tamaño en los que sea necesario almacenar la información para recuperarla para su posterior uso es muy util utilizar algunar de las siguientes librerías:
Pickle y Shelve.
La `persistencia` es la acción de conservar la información un objeto de ofrma permanente, pero también de recuperarla.
Para esto existe algo conocido como `serialización` de objetos. La serialización de un objeto consiste en generar una secuencia de bytes para su almacenamiento. Después mediante la `deserialización`, el estado original del objeto se puede reconstruir.

Esto es posible hacerlo en python con las librerías `Pickle` y `CPyckle`. La diferencia entre estas dos librerias es secillamente que la segunda esta escrita en *C*.

{% highlight python %}
import cPickle as pickle # or import pickle

d = {'var1':'desperados','var2':'coronita'}
# Abrimos el archivo donde almacenaremos el
# resultado de la serialización
with open('file.data', 'w') as fh:
    # Almacenamos el diccionario en el fichero con dump()
    pickle.dump(d, fh)
    fh.close()
# Con load leemos el contenido almacenado en el
# archivo donde almacenamos la información.
with pickle.load(open('file.data', 'r')) as spd:
    print spd
{% endhighlight %}

Otra forma de hacer persistente un objeto es con la librería Shelve de Python. Esta librería trabaja sobre pickle y permite almacenar objetos como un diccionario.
Es muy útil cuando queremos guardar muchos objetos y posteriormete acceder solo a algunos de ellos.

{% highlight python %}
import shelve

d = {'var1':'desperados'}
d2 = {'var2':'coronita'}
# Escribimos los datos asociandoles una llave
with shelve.open('file.data', 'c') as shelf:
    shelf["uno"] = d
    shelf["dos"] = d1
# Leemos los datos y los mostramos
with shelve.open('file.data', 'r') as shelf:
    for key in shelf.keys():
        print(repr(shelf[key])))
{% endhighlight %}

Para más documentación visitar la página oficial de [Shelve] y [Pickle]

[Shelve]: https://docs.python.org/2/library/shelve.html
[Pickle]: https://docs.python.org/2/library/pickle.html
