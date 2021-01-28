---
date: "2020-01-26"
tags:
- preparacion
title: Funciones
enableEmoki: true
---

# Presentación: funciones

En esta sección y la [siguiente](/preparacion_datos/) presentaremos brevemente los pasos previos al análisis. Comenzaremos ocupándonos del "esqueleto" :skull: de nuestro proyecto: el desarrollo de _funciones_. 

{{< note >}}
Las funciones son fragmentos de código que nos permiten replicar tareas de forma sencilla. Las que definimos para nuestro trabajo realizan transformaciones: reciben determinado input, y arrojan un resultado deseado
{{< /note >}}

Si bien algunas de éstas servirán al análisis que presentarmemos más adelante, elegimos comenzar por aquí ya que su principal aplicación es en el manejo y preparación de los datos. Como veremos en [el post correspondiente](/preparacion_datos/), nuestro proyecto ha hecho uso de bases de datos múltiples y dispersas. Su adecuado nos requería repetir una gran cantidad de operaciones, por lo que decidimos sistematizar las funciones que presentaremos a continuación... y, de allí en adelante, fuimos desarrollando algunas funciones adicionales. 

Comenzaremos presentando funciones para scrappear y formatear tablas con datos electorales, seguiremos con las funciones para trabajar con bases de datos de tuits. Un conjunto de estas está destinado a la descarga y limpieza de datos, otro al análisis del texto contenido en estos tuits. Finalmente, desarrollamos un par de funciones para hacer más simple el ploteo de gráficos. 

{{< warning >}}

Nuestras funciones constituyen una respuesta ad hoc a los desafíos implicados por este proyecto. De todos modos, hemos intentado que su diseño sea lo más general (y replicable) posible. 

{{< /warning >}}

A los fines de la claridad en la lectura, simplemente nombraremos y describiremos cada función. Incorporaremos los enlaces a los scripts correspondientes para quien quiera interiorizarse con su operatoria. 


## Funciones para scrappear y formatear tablas con datos electorales

La primera parte de nuestro análisis, la _["exploración de la popularidad"](/explorando_popularidad/)_ requería el manejo de resultados electorales de las elecciones argentinas de 2019 en las provincias, y a nivel nacional. A la fecha de ejecución de nuestro proyecto, el modo más accesible y sistemático de obtener estos datos era a través de wikipedia. Por eso, desarrollamos una serie de funciones para scrappear y formatear la información contenida en las páginas para cada distrito. Luego pudimos aplicarlas a la extracción del mismo tipo de datos (resultados electorales) de la [página oficial del goberno argentino, la  Dirección Nacional Electoral](https://www.argentina.gob.ar/interior/dine/resultados-y-estadisticas/elecciones-2019).

Procedimos en dos pasos: primero desarrollamos ciertas funciones "de base", que luego agregamos en funciones más complejas, para hacer la extracción y manejo de estos datos más automática. 

Nuestras funciones se apoyan en los paquetes core de  _[tidyverse](https://www.tidyverse.org/)_ y en particular en _[rvest](https://rvest.tidyverse.org/)_, que sirve para scrappear datos de páginas web. 

#### Funciones de base

Para leer datos de la web:

{{< ticks >}}
* `arbolTablas`: recibe un url, lee el código html y se queda con los nodos "tabla"
* `extraerTabla`: recibe un _data frame_ (df) con el árbol de tablas de una página web y un parámetro numérico que indica la tabla con la que deseamos quedarnos.
{{< /ticks >}}

Para formatear las tablas extraídas en el punto anterior:

_Se trata de funciones que emprolijan las particularidades del set de datos con el que trabajamos consideramos; a saber, tablas con datos de resultados electorales para las elecciones argentinas de 2019, scrappeadas desde wikipedia. Cada una de ellas recibe el resultado de aplicar la función anterior. Consideramos que los nombres son suficientemente informativos._

{{< ticks >}}

* `borrarPrimeraFila`
* `reducirAnchoTabla`: borra columnas inutilizadas.
* `renombrarTabla`: renombra variables de modo que sean compatibles entre tablas.
* `limpiezaTabla`: emprilija atributos de las variables de las tablas. Formatea los números indicativos de la cantidad de votos y el procentaje obtenido por cada candidato.
* `agregarVotosGobernador`: en algunos casos el sistema electoral de los distritos argentinos implica que los votos obtenidos por un candidato son el resultado agregado de múltiples listas. Esta función hace el cálculo y se queda con el resultado por candidato, en lugar de por lista.
* `reducirLargoTabla`: se deshace del conteo de Votos Nulos, en Blanco, y etc., que no sirven a nuestros fines; en otras palabras, retiene solamente los votos obtenidos por candidato. 
* `agregarColumnas`: agrega datos de futuro interés: calcula y añade un ranking con el puesto obtenido por cada candidato en la respectiva elección y, optativamente, el nombre del distrito en cuestión. 

 
{{< /ticks >}}

#### Funciones "agregadas"

Sobre la base de las anteriores, armamos dos funciones para la extracción y sistematización de datos electorales desde páginas web:

{{< ticks >}}
* `extraer_datos_wiki`: agrega las primeras funciones, para leer datos de la web y obtener una tabla con resultados electorales. 
* `procesar_datos_wiki`: agrega las funciones de formateo y limpieza de los datos, de modo de quedarnos con una tabla "tidy" y adecuada a nuestros fines.
{{< /ticks >}}

{{< expandable label="¿Querés saber cómo funcionan? "  level="2"  >}}
Para conocer el detalle de estas funciones, dejamos aquí el  [:arrow_right: script](https://github.com/CVFH/Tuits_arg_2019/blob/master/Modules/tablasElectorales.R) correspondiente.
{{< /expandable >}}

## Funciones para trabajar con bases de datos de tuits

Las funciones a continuación fueron centrales para la manipulación de los datos que nos conciernen: bases de datos de tuits, emitidas por candidatos a cargos políticos.

De nuevo, su gramática se sustenta en el paquete tidyverse, _[tidyverse](https://www.tidyverse.org/)_. Por su parte, para el análisis de texto hemos sacado provecho de _[tidytext](https://www.tidyverse.org/)_.[^1]

De nuevo, también, su desarrollo y presentación va de lo más simple a lo más complejo: introduciremos algunas funciones que llamamos "de base", que constituyen los bloques para las funciones más complejas.

#### Funciones de base

Para leer datos de la web:

{{< ticks >}}
* `determinarTuitsCampaña`: recibe un df con tuits (del que se espera que haya una columna que consigne la fecha de su emisión) y dos parámetros: la fecha de inicio y la de fin de la campaña. Crea una variable que indica si el tuit fue emitido o no durante la campaña.
* `seleccionarTextoTuits`: recibe un df con tuits y, opcionalmente, un parámetro para seleccionar columnas. Formatea la columna que contiene el texto del tuit a tipo _character_ y acota el data frame a las columnas seleccionads.
* `transformarEnlacesTwitter`: recibe un df con tuits y transforma ciertos caracteres del texto de los tuits, a los fines de identificarlos facilmente a la hora de analizar el texto. En concreto, busca los enlaces, los hashtags ("#") y las menciones (@).

{{< /ticks >}}

#### Funciones "agregadas"

{{< ticks >}}
* `tokenizarTextoTuits`: recibe un df con tuits y devuelve su texto **"tokenizado"** (esto es, transformado de manera tidy en unidades mínimas constituyentes, a los fines de su análisis). De manera optativa, filtra los tuits emitidos durante una campaña electoral, y se deshace de los tuits que sean RTs, es decir, que no hayan sido redactados por el emisor de interés (en nuestro caso, los candidatos). La opción por defecto es tokenizar "palabras", pero alternativamente se puede optar por la opción "ngramas" o "tweets" del paquete _tidytext_.
* `limpiarTokens`: recibe un df con tuits "tokenizados" y ofrece distintas opciones para "limpiarlos": extraer ciertas palabras muy utilizadas e "insignificantes" (para lo que nos valemos del paquete [stopwords](https://www.rdocumentation.org/packages/stopwords)), y deshacernos de palabras demasiado cortas, enlaces, mentios y/o hasthags.

{{< /ticks >}}

{{< expandable label="¿Cómo fueron hechas? "  level="2"  >}}
Para conocer el detalle de estas funciones :mag:, seguir  [:arrow_right: este enlace](https://github.com/CVFH/Tuits_arg_2019/blob/master/Modules/tuitsCandidatos.R).
{{< /expandable >}}


## Otras funciones

Adicionalmente hemos simplificado algunas tareas con el desarrollo de funciones. Destacamos en particular dos que alivianan el ploteo de gráficos (y estandarizan su formato): `plotPoint` y `plotPointText` y `formatPlot`. 
Las tres se sustentan en la gramática de [ggplot](https://ggplot2.tidyverse.org/). 
Pueden [explorar :mag: el código aquí](https://github.com/CVFH/Tuits_arg_2019/blob/master/Modules/funcionesGraficos.R).
Finalmente, hemos optado por "envolver" las operaciones para preparar nuestros datos en dos funciones respectivas: `traerDatosElectorales` y `traerDatosTuits`, como explicaremos a partir de la [próxima sección](/preparacion_datos). 


[^1]: Hemos leido y recomendamos [este libro](https://www.tidytextmining.com/)