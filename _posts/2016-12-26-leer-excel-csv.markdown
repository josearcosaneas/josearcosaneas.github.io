---
layout: post
title:  "Leer ficheros CSV y XLS con Python"
date:   2016-12-26 13:56:44 +0100
categories: Python XLS CSV
---
En este post vamos a manejar ficheros **csv** y **xls**.
Este tipo de ficheros son muy usados actualmente para almacenar información.
A continuación, aprenderemos ha leer y escribir archivos de este tipo con Python de una forma
sencilla.

## Archivos CSV
Vamos a empezar por los archivos con extensión CSV del inglés *comma-separated values*.
Estos archivos son un clásico a la hora de representar datos en forma de tabla.

En primer lugar vamos a hacerlo sin usar ningún módulo adicional,
esta es la forma más tediosa cuando queremos leer muchos datos.

{% highlight python %}
with open('fh2.csv') as fh:
    for line in fh:
        print str(line.split(',')[0]), str(line.split(',')[1])
fh.close()
{% endhighlight %}
El código anterior lee el archivo `fh2.csv` línea a línea y lo muestra por
pantalla.
Y la salida del programa anterior es la siguiente:
{% highlight shell %}
Numeros Letras
1 a
2 b
3 c
4 d
5 f
6 g
7 h
8 I
9 j
{% endhighlight %}

La forma más habitual de hacer este tipo de lecturas con el módulo *csv*.

Es muy sencillo:
{% highlight python %}
import csv

f = open('fh2.csv')
lines = csv.reader(f)
for line in lines:
    print str(line[0]), str(line[1])
{% endhighlight %}
Este código realiza la misma tarea que el anterior, la salida vuelve a ser:
{% highlight shell %}
Numeros Letras
1 a
2 b
3 c
4 d
5 f
6 g
7 h
8 I
9 j
{% endhighlight %}


Es posible cambiar el tipo de separador (o delimitador) a *;* ,
de la siguiente forma:
{% highlight python %}
f = open('fh.csv')
lns = csv.reader(f,delimiter=';')
{% endhighlight %}

Para facilitar la entrada y salida de datos podemos usar los *dialect*. Los dialect
son clases predefinidas la más habital es *excel*.
Para hacer uso de ella solo tenemos que hacer lo siguiente:
{% highlight python %}
f = open('fh.csv')
lines = csv.reader(f, dialect="excel")
print csv.list_dialects()
{% endhighlight %}
Por útlimo vamos a crear un nuevo archivo y a escribir en él.

Lo haremos de la siguiente forma:
{% highlight python %}
f = open('fh.csv', 'w')
obj = csv.writer(f, delimiter=',', quotechar='|', quoting=csv.QUOTE_MINIMAL)
obj.writerow(['10', 'A'])
f.close()
{% endhighlight %}
Cómo podemos ver se ha creado un archivo *fh.csv* y se ha escrito una línea que contiene en la primera columna un
`10` y en la segunda un `A`.

## Archivos XLS

Si queremos leer archivos de tipo **xls**, que es el formato de Excel (junto s *xlsx*) podemos instalar las libreriás *xlrd* y
*xlwt*.

Para instalar los módulos que vamos a usar:

{% highlight shell %}
$ sudo pip install xlrd
{% endhighlight %}

En este ejemplo crearemos el archivo y después lo leeremos.

Creamos nuestro archivo de datos:
{% highlight python %}
import xlwt

list1 = [1, 2, 3, 4, 5]
list2 = ['a', 'b', 'c', 'd', 'e']
wb = xlwt.Workbook()
ws = wb.add_sheet('Primera hoja')
ws.write(0, 0, 'Columna A')
ws.write(0, 1, 'Columna B')
i = 1
for x, y in zip(list1, list2):
    print i
    # con zip recorro las 2
    # listas a la vez
    ws.write(i, 0, x)
    ws.write(i, 1, y)
    i += 1
    wb.save('datos.xls')
{% endhighlight %}

El código anterior creará un archivo con el nombre *datos.xls*. Este archivo
tendrá dos columnas con los datos de las listas que se definen en el código.

Ahora procedemos a leer el archivo. Esto es muy sencillo con la librería
*xlwr*.
Primero importamos y leemos el arhivo.
{% highlight python %}
import xlrd

book = xlrd.open_workbook('datos.xls')
sh = book.sheet_by_index(0) # Indicamos la primera hoja
{% endhighlight %}
Ahora podemos llamar ha funciones como las siguientes:
{% highlight python %}
 sh.nrows # número de filas
 sh.ncols # número de columnas
 sh.cell_value(rowx=1, colx=1) # Seleccionamos una celda
 sh.col_values(0) # Seleccionamos la primera columna
 sh.col_values(1) # Seleccionamos la segunda columna
 sh.row_values(0) # Seleccionamos la primera fila
{% endhighlight %}
El resultado de la ejecución de las ordenes anteriores sobre el archivo *datos.xls*,
creado anteriormente, es el siguiente:
{% highlight shell %}
6
2
a
[u'Columna A', 1.0, 2.0, 3.0, 4.0, 5.0]
[u'Columna B', u'a', u'b', u'c', u'd', u'e']
[u'Columna A', u'Columna B']

{% endhighlight %}
Para más documentación visitar la página oficial de [CSV] y [xlwr/xlrd]


[CSV]: https://docs.python.org/2/library/csv.html
[xlwr/xlrd]:https://pypi.python.org/pypi/xlwt
