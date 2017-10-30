---
layout: post
title:  "Introducción al procesamiento del lenguaje natural-1 "
date:   2017-1-2 22:06:44 +0100
categories: Python R procesamiento lenguaje
---

A lo largo de este post estudiaremos algunas técnicas de **procesamiento
del lenguaje natural**, por sus siglas **PLN**.
El *procesamiento del lenguaje natural* combina la ciencias de las computación
con la linguística. El objetivo de estas técnicas es el de hacer posible la
comprensión y el procesamiento por parte del ordenador.

Los campos principales en los que se aplican este tipo de técnicas es la
traducción automática, clasificadores de textos, los sistemas de diálogo,
el análisis de opiniones...

El procedimiento que vamos a usar como ejemplo incluye una serie de pasos.
En primer lugar, procederemos a tokenizar el texto.
Este proceso de tokenización consiste en distinguir los tokens,
en nuetro caso están representados por las palabras o signos de puntuación,
especiales,...
El segundo paso consiste en la eliminación de palabras vacías, del inglés stopwords.
Las palabras vacías son aquellas palabras que no aportan ningún significado al texto,
por ejemplo, los artículos, las preposiciones, ...

Antes de calcular la frecuencia de las palabras o usarlar en cualquier algoritmo
de clasificación suele realizarse un tercer paso, **stemming**.
El *stemming* consiste en extreaer la raíz de una palabra. Este proceso se realiza
porque la raíz de una palabra puede aparecer más veces en un texto.
Por último, calcularemos las palabras más frecuentes y su frecuencia.

Este procedimiento es muy sencillo de realizar en Python con ayuda de la librería
**NLTK**.
Esta librería probe de todo lo necesario para procesar textos.

Para instalar NLTK podemos usar PIP escribiendo: `sudo pip install -U nltk`.
Una vez hecho, esto escribimos en terminal `python`, esto nos
abrirá una consola de Python y escribimos lo siguiente:

{% highlight shell %}
import nltk
nltk.download()
{% endhighlight %}

Ahora nos aparecerá un asistente, se recomienda instalar **all-corpora**.
Y manos a la obra. Importamos todo los necesario.
{% highlight Python %}
# -*- coding: utf-8 -*-
from nltk import wordpunct_tokenize
from nltk.corpus import stopwords
import nltk
from nltk.stem.porter import PorterStemmer
from nltk.stem.Snowball import SnowballStemmer

{% endhighlight %}

En primer lugar extraeremos los tokens que forman el texto.
Usaremos la función **wordpunct_tokenize()** que tiene la libreria NLTK.
Esta función, a diferencia de la función **word_tokenize()**, almacena
los signos de puntuación como tokens diferentes.

Podríamos proceder de la siguiente manera:
{% highlight python %}
tokens = [word for word in wordpunct_tokenize(text)]
{% endhighlight %}

A continuación, extraeré del módulo stopwords de NLTK los stopwords
del lenguaje español.
Añadiré a la lista los signos de puntuación.
{% highlight python %}
stopw = [w.encode('utf-8') for w in stopwords.words('spanish')]
punc_lit = [u'.', u'[', ']', u',', u';', u'', u')', u'),', u' ', u'(']
stopw.extend(punc_lit)
{% endhighlight %}

Eliminaré el conjunto de palabras vacías de la lista de tokens
que compenen el texto.
{% highlight python %}
words = [token.decode('utf-8', 'ignore')
         for token in tokens if token not in stopw]
{% endhighlight %}

Con el conjunto restante realizamos el stemming.
En este caso usamos el algorimto de Porter. Básicamente, el algoritmo
de Porter es un algoritmo de derivación de palabras y consiste en elimar las
términaciones morfológicas e inflexionales comunes de las palabras
en inglés, aunque es posible usarlo también en español. Es un proceso
de normalización de términos muy usado en la clasificación automática
de textos o en sistemas de recuperación de información.
Para más información sobre este algoritmo visitar la página oficial del autor del
[algoritmo de Porter].


{% highlight python %}
porter_stemmer = PorterStemmer()
stemmers = [porter_stemmer.stem(word) for word in words]
final = [stem for stem in stemmers if stem.isalpha() and len(stem) > 1]
{% endhighlight %}

Ahora podemos calcular las palabras más frecuentes y su frecuencia.

{% highlight python %}
fdist = nltk.FreqDist(final)
print fdist.most_common(10)
{% endhighlight %}

Y el resultado mostrado por pantalla es el siguiente:
{% highlight shell %}
[(u'caballero', 15), (u'hab', 15), (u'as', 15), (u'nombr', 14), (u'ten', 8),
(u'tan', 7), (u'se', 7), (u'bien', 7), (u'an', 7), (u'raz\xf3n', 6)]
{% endhighlight %}

Ahora vamos a realizar el paso del stemming con el algoritmo de Porter2, este algorimto
es mejor si se usan textos en español. Añade una serie de reglas que dependen del
lenguaje que se utilice. Si quieres saber más sobre el algoritmo de Porter2 (o Snowball)
visitar la pagina ver oficial de [Snowball].
Realizaré de nuevo los pasos


{% highlight python %}
sSnowball_stemmer = SnowballStemmer('spanish')
stemmers2 = [Snowball_stemmer.stem(word) for word in words]
final2 = [stem for stem in stemmers2 if stem.isalpha() and len(stem) > 1]
{% endhighlight %}

Calculamos las palabras más frecuentes y su frecuencia.

{% highlight python %}
fdist2 = nltk.FreqDist(final2)
print fdist2.most_common(10)
{% endhighlight %}

El resultado en este caso es el siguiente:
{% highlight shell %}
[(u'hab', 16), (u'caballer', 15), (u'as', 15), (u'nombr', 14), (u'ten', 10),
(u'parec', 10), (u'llam', 9), (u'razon', 8), (u'hac', 8), (u'aquell', 8)]

{% endhighlight %}

El código completo sería el siguiente:

{% highlight python %}
languages = stopwords.fileids()
print languages
tokens_lower = [word for word in wordpunct_tokenize(text)]
print tokens_lower
stopw = [w.encode('utf-8') for w in stopwords.words('spanish')]
punc_lit = [u'.', u'[', ']', u',', u';', u'', u')', u'),', u' ', u'(']
stopw.extend(punc_lit)
words = [token.decode('utf-8', 'ignore')
         for token in tokens_lower if token not in stopw]

porter_stemmer = PorterStemmer()
print words
stemmers = [porter_stemmer.stem(word) for word in words]
final = [stem for stem in stemmers if stem.isalpha() and len(stem) > 1]
print final
snowball_stemmer = SnowballStemmer('spanish')
stemmers2 = [Snowball_stemmer.stem(word) for word in words]
final2 = [stem for stem in stemmers2 if stem.isalpha() and len(stem) > 1]
fdist = nltk.FreqDist(final)
print fdist.most_common(10)
fdist2 = nltk.FreqDist(final2)
print fdist2.most_common(10)
{% endhighlight %}

Un saludo.



## Referencias:
[https://tartarus.org/martin/PorterStemmer](https://tartarus.org/martin/PorterStemmer)

[http://Snowballstem.org/](http://Snowballstem.org/)

[http://scikit-learn.org/](http://scikit-learn.org/)



[algoritmo de Porter]: https://tartarus.org/martin/PorterStemmer
[Snowball]: http://Snowballstem.org/
[scikit-learn]: http://scikit-learn.org/

