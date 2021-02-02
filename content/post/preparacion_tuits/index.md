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
   
   
   # definimos previamente un vector con las columnas que retendremos de los dataframes de tuits (ver punto 10)
  
  select_columnas <- c("created_at", 
                       "text", 
                       "rts", "fav_count", 
                       "tweet_id", 
                       "screen_name", "user_id", "description", 
                       "location", 
                       "mention_screen_names", 
                       "in_reply_to_screen_name",
                       "Campaña")
                       
    if(tipo_dato == "base") {
   
    # ids
    # descargamos directamente desde Git
    
    datos_base <- read.csv("https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Data/datos_base.csv", encoding = "UTF-8", stringsAsFactors = FALSE) 
    
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
    
    [[[joined_gobernadores <- rbind(simultaneas_df, desdobladas_df)]]] %>% 
    [[[select(select_columnas)]]]
    
    [[[devolver_data <- joined_gobernadores]]]
   
    }
    
      else if (tipo_dato == "presid") {
    
  # PRESIDENCIALES 
  
  # mismos pasos
    
    # fechas
    
    fecha_paso <- as.Date("2019-08-11")
    fecha_grales <- as.Date("2019-10-27")
    
  # enlaces
    
  presid1 <- "https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Data/alferdez.csv"
  presid2 <- "https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Data/mauriciomacri.csv"
  presid3 <- "https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Data/RLavagna.csv"
  presid4 <- "https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Data/NicolasdelCano.csv"
  presid5 <- "https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Data/juanjomalvinas.csv"
  presid6 <- "https://raw.githubusercontent.com/CVFH/Tuits_arg_2019/master/Data/jlespert.csv"
  
  presid_filenames <- c(presid2, presid3, presid4, presid5, presid6)
  
  joined_presid <- presid_filenames %>% 
    map_dfr(read.csv, encoding = "UTF-8", stringsAsFactors = FALSE ) %>% 
    determinarTuitsCampaña(fecha_paso, fecha_grales) %>% 
    select(select_columnas)

  # en este caso traemos por separado la base de alferdez por una diferencia en el .csv, y después las unimos
  
  presid1 <- read.csv(presid1, encoding = "UTF-8", stringsAsFactors = FALSE) %>% 
    determinarTuitsCampaña(fecha_paso, fecha_grales) %>% 
    select(select_columnas)
  
  # unimos la base con los tuits de @alferdez y el resto
  
  joined_presid <- joined_presid %>% rbind(presid1)
  
  devolver_data <- joined_presid

}
  
    [[[else if (tipo_dato=="tot")]]] {
      
     # traemos bases separadas
    
     joined_presid <- traerDatosTuits("presid")
     joined_gobernadores <- traerDatosTuits("gob")
     
     # las unimos
      
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
6. leer los .csv y para 
7. identificar los tuits emitidos durante la campaña
8. Cuando correspondía, luego, agregamos las bases (por ejemplo, de provincias que hicieron elecciones desdobladas y en simultáneo).
9. Nos quedamos con las columnas que nos interesan, definidas previamente
10. Finalmente, "devolvimos" los datos de interés
11.  _Note el lector que, nuevamente, para la opción que pide la totalidad de los tuits emitidos por los candidatos, utilizamos un esquema recursivo._

:tada: :tada: :tada: listos para su análisis!!! :tada: :tada: :tada:

Te invitamos a comenzar la lectura de nuestros resultados desde [este post](../explorando_popularidad/). También podés [volver al inicio :house:](/).

[^1]: Que, a su vez, se vale de [funciones definidas anteriormente](../preparacion_funciones/)
[^2]: Con la función `determinarTuitsCampaña()` que presentamos en [este post](../preparacion_funciones)
[^3]: Pueden acceder a los archivos utilizados [desde nuestro repositorio de Git](https://github.com/CVFH/Tuits_arg_2019/tree/master/Data)