---
layout: post
title:  "Leer ficheros CSV y EXCEL con Python"
date:   2016-12-26 13:56:44 +0100
categories: Python Excel CSV
---
En este post vamos a ver como leer ficheros csv y excel.
Este tipo de ficheros son muy usados actualmente para almacenar información.

En primer lugar podemos hacerlo sin usar ningún modulo adicional, aunque claro esta es la forma más tediosa cuando queremos leer muchos datos.

{% highlight python %}
with open('fh2.csv') as fh:
    for line in fh:
        print str(line.split(',')[0]), str(line.split(',')[1])
fh.close()
{% endhighlight %}
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

La forma más habitual de hacer este tipo de lecturas con el módulo *csv*

{% highlight python %}
import csv

f = open('fh2.csv')
lines = csv.reader(f)
for line in lines:
    print str(line[0]), str(line[1])
{% endhighlight %}
La salida de este programa vuelve a ser:
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


Es posible cambiar el tipo de separa por ejemplo a *\t* o por ejemplo a *;*
de la siguiente forma:
{% highlight python %}
#Cambiando separador (delimitador):
f = open('fh.csv')
lns = csv.reader(f,delimiter=';')
{% endhighlight %}

Para facilitar la entrada y salida de datos podemos usar los dialect. Los diales
son clases predefinidas la más habital es *excel*.
Para hacer uso de ella solo tenemos que hacer los siguiente:
{% highlight python %}
f = open('fh.csv')
lines = csv.reader(f, dialect="excel")
print csv.list_dialects()
{% endhighlight %}
Por útlimo vamos a crear un nuevo archivo y a escribir en él. LO haremos de la
siguiente forma:
{% highlight python %}
f = open('fh.csv', 'w')
obj = csv.writer(f, delimiter=',', quotechar='|', quoting=csv.QUOTE_MINIMAL)
obj.writerow(['10', 'A'])
f.close()
{% endhighlight %}
Cómo podemos ver se ha escrito una linea que contiene en la primera columna un
10 y en la segunda un A.

Si queremos leer archivos de tipo xls podemos instalar las libreriás xlrd y
xlwt.
Su uso es muy sencillo.

{% highlight python %}
import xlrd

# diccionario vacio
data = {}
# Leemos el archivo
book = xlrd.open_workbook('datos.xls')
sh = book.sheet_by_index(0)
for i in range(1,sh.nrows):
    data[sh.cell_value(rowx=i, colx=1)] = \
    sh.cell_value(rowx=i, colx=2)

# Creando archivos
import xlwt

list1 = [1, 2, 3, 4, 5]
list2 = [a, b, c, d, f]
wb = xlwt.Workbook()
hoja = wb.add_sheet('Primera hoja')
hoja.write(1, 2, 'Columna A')
hoja.write(a, b, 'Columna B')
i = 1
# recorro las 2 listas a la vez
for x,y in zip(list1,list2):
    hoja.write(i,0,x)
    hoja.write(i,1,y)
    i += 1
wb.save('mynewfile.xls')
{% endhighlight %}

Otro módulo similar es openpyxl.

{% highlight python %}
#suponiendo que el archivo esta en el mismo directorio del script
import openpyxl
doc = openpyxl.load_workbook('address.xls')
# Leemos las hojas
doc.get_sheet_names()
# Elegimos una por nombre
hoja = doc.get_sheet_by_name('Sheet1')
hoja.title
# Leemos las celdas
hoja.rows
for filas in hoja.rows:
    print filas

print hoja['A1'].value
print type(hoja['F2'].value)

hoja.cell(row=1,column=1).value

for fila in hoja.rows:
    for columna in fila:
        print columna.value,
    print ""

hoja.get_highest_row()

hoja.get_highest_column()
{% endhighlight %}

Para más documentación visitar la página oficial de [Shelve] y [Pickle]

[Shelve]: https://docs.python.org/2/library/shelve.html
[Pickle]: https://docs.python.org/2/library/pickle.html
