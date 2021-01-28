---
title: preparacion_electorales
author: ~
date: '2021-01-27'
slug: preparacion-datos
categories: []
tags:
  [preparacion]
---

En este post comentamos los pasos seguidos para scrappear la web y obtener tablas con los resultados electorales para los distritos de nuestro interés.
En otras palabras, mostramos y explicamos brevemente el script de ["preparacion_datos_electorales.R"](https://github.com/CVFH/Tuits_arg_2019/blob/master/preparacion_datos_electorales.R), en el que desarrollamos la función `traerDatosElectorales()`.

## Intro

Como dijimos, queríamos información sobre los resultados de las elecciones de 2019 para 8 provincias argentinas y el nivel nacional. Hallamos la información de nuestro interés en Wikipedia y en el sitio oficial de la Dirección Electoral Nacional. 

Para leer y formatear esta información, implementamos ciertas [funciones que introdujimos en un post anterior](../preparacion_funciones), a las que asignamos los parámetros correspondientes.

## Los pasos preliminares

Como en toda tarea en `r`, nuestro primer paso consiste en la activación de algunos paquetes útiles. 

En este caso, además de los 
1. "oficiales", incluimos un 
2. [script propio](https://github.com/CVFH/Tuits_arg_2019/blob/master/Modules/tablasElectorales.R):

```
{{< code numbered="true" >}}
# Apertura de librerías.

[[[# Oficiales]]]
library(tidyverse)
library(rvest) # extraer datos de html
library(readxl) # extraer datos de excel

[[[# Propio]]]
source("https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Modules/tablasElectorales.R")
{{< /code >}}
```
