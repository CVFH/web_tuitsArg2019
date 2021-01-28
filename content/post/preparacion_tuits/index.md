---
date: "2021-01-26"
author: Carolina Franco
tags:
- preparacion
- datos
title: Preparacion de Bases de Datos de Tuits
weight: 4
---

El objetivo de este post es exponer los pasos realizados para agregar y formatear la :meat_on_bone: carne de nuestro proyecto: información relativa a los tuits emitidos por los candidatos a las elecciones de 2019 en la Argentina.

De manera similar al [post anterior](../preparacion_electorales/), aquí referimos y explicamos brevemente el script ["preparacion_datos_tuits.R"](https://github.com/CVFH/Tuits_arg_2019/blob/master/preparacion_datos_tuits.R), que define la función `traerDatosTuits()`[^1].

## La previa

De nuevo, comenzamos por la apertura de algunos paquetes..

{{< code numbered="true" >}}
#####
# APERTURA DE LIBRERIAS
#####

[[[# paquetes]]]

library(tidyverse)
library(janitor)
library(readxl)

[[[# propias]]]

source("Modules/tuitsCandidatos.R", encoding = "UTF-8")
{{< /code >}}

1. "oficiales"
2. y ["propios"](https://github.com/CVFH/Tuits_arg_2019/blob/master/Modules/tuitsCandidatos.R)

## traerDatosTuits(). Una función para llamar a datos de bases de Twitter.

Luego, definimos la función de nuestro interés. De manera similar a `traerDatosElectorales()`, esta recibe una serie de opciones alternativas: "gob", para traer los datos de las bases de los candidatos a gobernador que obtuvieron el primer y el segundo puesto en las provincias seleccionadas; "presid", para traer los tuits emitidos por los candidatos a la presidencia; "tot" para el conjunto de estos datos. Incluimos además una opción "base" que invoca a la meta-data de la muestra elegida. 

{{< note >}}
La función llama y formatea a bases de datos previamente almacenadas, que obtuvimos en su momento con `get_timeline()` de `rtweet`.
{{< /note >}}

Presentamos el código, y abajo incluímos una breve explicación de los pasos más relevantes.

{{< code numbered="true" >}}
#####

#IMPORTACION DE DATOS #####

traerDatosTuits <- function(tipo_dato){
    
   # función que trae los datos necesarios.
   # opciones: candidatos a presidente, a gobernador, todos juntos, datos   de base
   # "presid", "gob", "tot", "base", respectivamente
    
    if(tipo_dato == "base") {
   
    # ids
  
    datos_base <- read_xlsx("Data/datos_base.xlsx")
    devolver_data <- datos_base
   }
  
    else if (tipo_dato == "gob") {
      
    [[[# QUE DESDOBLARON:]]] ELECCIONES GRALES 16/JUNIO. SOLO SANTA FE   PASO   28/ABRIL
    
    [[[fecha_elecciones_desdoblada]]] <-as.Date("2019-06-16")
    fecha_campaña_desdoblada <-as.Date("2019-04-28")
    
    [[[formosa1]]] <- "Data/insfran_gildo.csv"
    formosa2 <- "Data/adrianbogadoOK.csv"
    sluis1 <-"Data/alberto_rsaa.csv"
    sluis2 <-"Data/claudiojpoggi.csv"
    sfe1 <- "Data/omarperotti.csv"
    sfe2 <-"Data/AntonioBonfatti.csv"
    tfuego1 <-"Data/gustavomelella.csv"
    tfuego2 <-"Data/RosanaBertone.csv"
    
    [[[desdobladas_filenames]]] <- c(formosa1, formosa2, sluis1, sluis2,   sfe1, sfe2, tfuego1, tfuego2)
    
    desdobladas_df <- desdobladas_filenames %>% 
      [[[map_dfr]]]([[[read.csv]]], encoding = "UTF-8", stringsAsFactors =   FALSE) %>% 
      [[[determinarTuitsCampaña]]](fecha_campaña_desdoblada,   fecha_elecciones_desdoblada)
    
    
    # SIMULTANEAS: QUE CELEBRARON ELECCIONES GRALES 27/OCTUBRE, PASO   11/AGOSTO
    
    fecha_paso <- as.Date("2019-08-11")
    fecha_grales <- as.Date("2019-10-27")
    
    baires1 <- "Data/Kicillofok.csv"
    baires2 <- "Data/mariuvidal.csv"
    caba1 <- "Data/horaciorlarreta.csv"
    caba2 <- "Data/MatiasLammens.csv"
    catamarca1 <- "Data/RaulJalil_ok.csv"
    lrioja1 <- "Data/QuintelaRicardo.csv"
    lrioja2 <- "Data/JulioMartinezLR.csv"
    
    simultaneas_filenames <- c(baires1, baires2, caba1, caba2, catamarca1,   lrioja1, lrioja2)
    
    simultaneas_df <- simultaneas_filenames %>% 
      map_dfr(read.csv, encoding = "UTF-8" ) %>% 
      determinarTuitsCampaña(fecha_paso, fecha_grales)
    
    [[[joined_gobernadores <- rbind(simultaneas_df, desdobladas_df)]]]
    
    [[[devolver_data <- joined_gobernadores]]]
   
    }
    
    else if (tipo_dato == "presid") {
      
    # PRESIDENCIALES 
      
      # fechas
      
      fecha_paso <- as.Date("2019-08-11")
      fecha_grales <- as.Date("2019-10-27")
      
    # aquí tenemos un archivo -xlsx y los demás .csv. 
    # resultó más sencillo descargarlos independientemente y luego unirlos
    
    presid1 <- read_xlsx("Data/alferdez.xlsx")
    presid2 <- read.csv("Data/mauriciomacri.csv", encoding = "UTF-8",   stringsAsFactors = FALSE)
    presid3 <- read.csv("Data/RLavagna.csv", encoding = "UTF-8",   stringsAsFactors = FALSE)
    presid4 <- read.csv("Data/NicolasdelCano.csv", encoding = "UTF-8",   stringsAsFactors = FALSE)
    presid5 <- read.csv("Data/juanjomalvinas.csv", encoding = "UTF-8",   stringsAsFactors = FALSE)
    presid6 <- read.csv("Data/jlespert.csv", encoding = "UTF-8",   stringsAsFactors = FALSE)
    
    joined_presid <- rbind(presid1, presid2, presid3, presid4, presid5,   presid6)
    
    joined_presid <- joined_presid %>% 
      determinarTuitsCampaña(fecha_paso, fecha_grales)
    
    devolver_data <- joined_presid
  
  }
  
    [[[else if (tipo_dato=="tot")]]] {
      
    # trae bases separadas
       joined_presid <- traerDatosTuits("presid")
       joined_gobernadores <- traerDatosTuits("gob")
       
    # unir las dos de manera prolija
    
    mismatched_cols <- compare_df_cols(joined_presid, 
                                       joined_gobernadores, 
                                       return =  "mismatch",
                                       bind_method = "rbind")
    
    joined_presid <- joined_presid %>% 
      mutate(created_at = as.factor(created_at),
             created_at_user = as.factor(created_at_user))
    joined_gobernadores <- joined_gobernadores %>% 
      mutate( text = as.character(text),
              tweet_id = as.numeric(tweet_id))
        
    joined_candidatos <- rbind(joined_gobernadores,
                               joined_presid) 
    
    devolver_data <- joined_candidatos
  
    }
    
    else {
      devolver_data <- "datos no disponibles"
      #pendiente: hacer un raise warning
    }
   
    return(devolver_data)

}
{{< /code >}}

1. Verá nuestro lector que tratamos separadamente a las provincias que realizaron elecciones simultáneas y desdobladas. Esto es porque nos interesaba identificar los tuits emitidos durante la campaña[^2]
2. Esto implicó proveer dos parámetros: la fecha de inicio y de fin de la campaña
3. Para cada distrito, definimos la ruta del archivo correspondiente[^3]
4. Agregamos los nombres de los archivos en un vector...
5. ...a los fines de trabajar simultáneamente sobre todos ellos, valiéndonos de la función `map()` de `purrr`, para
6. leer los .csv
7. identificar los tuits emitidos durante la campaña
8. Cuando correspondía, luego, agregamos las bases (por ejemplo, de provincias que hicieron elecciones desdobladas y en simultáneo).
9. Finalmente, "devolvimos" los datos de interés
10.  _Note el lector que, nuevamente, para la opción que pide la totalidad de los tuits emitidos por los candidatos, utilizamos un esquema recursivo._

:tada: :tada: :tada: listos para su análisis!!! :tada: :tada: :tada:

[^1]: Que, a su vez, se vale de [funciones definidas anteriormente](../preparacion_funciones/)
[^2]: Con la función `determinarTuitsCampaña()` que presentamos en [este post](../preparacion_funciones)
[^3]: Pueden acceder a los archivos utilizados [desde nuestro repositorio de Git](https://github.com/CVFH/Tuits_arg_2019/tree/master/Data)