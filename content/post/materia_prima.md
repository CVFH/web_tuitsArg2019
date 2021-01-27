---
date: "2020-01-26"
tags:
- preparacion
title: Preparación
enableEmoki: true
---

Testing out GitHub issue https://github.com/zwbetz-gh/cupper-hugo-theme/issues/36 -- Multiple expandable shortcodes do not work if they have the same inner text.



{{< expandable label="Funciones" level="2" >}}
Same inner text.
```
code chunk
```
{{< /expandable >}}


{{<code numbered="true">}}
```r
# tuitsCandidatos

# modulo para trabajar con bases de datos de tuits 
# emitidos por políticos


#apertura de liberarias

require(stopwords)
require(tidyverse)

# FUNCIONES 

# de base

[[[determinarTuitsCampaña]]] <- function(df_tuits, fecha_inicio_campaña, fecha_elecciones){
  
  #recibe un dataframe con tuits. se espera que haya una columna
  #"created_at" que incluya la fecha de emisión del tuit
  #además recibe dos parámetros tipo Date: fecha de inicio de la campaña
  #y fecha de las elecciones
  #determina si un tuit fue emitido en campaña o no.
  #crea una columna "Campaña" donde 1 equivale a que el tuit fue emitido durante la campaña
  # 0 en caso contrario
  
  df_tuits_clasif_campaña <- df_tuits %>%  
    mutate( Campaña  = 
              ifelse(as.Date(created_at) < fecha_elecciones & as.Date(created_at) > fecha_inicio_campaña, 
                     1,
                     0) )
  return(df_tuits_clasif_campaña)
}

seleccionarTextoTuits <- function(df_tuits, colums = c("text", "screen_name", "tweet_id", "created_at", "rts", "fav_count")){
  
  ### recibe un dataframe con tuits emitidos
  ## se espera que contenga variable "text" con el texto de los tuits emitidos
  #### la devuelve transformada en caracteres
  ## junto a todas las variables que se hayan indicado en "colums". 
  ## por default, estas son: text, screen_name, tweet_id, created_at, rts, fav_count
  seleccion_text <- df_tuits %>% 
    select(colums) %>% 
    mutate(text = as.character(text)) 
  
  return(seleccion_text)
}

transformarEnlacesTwitter <- function(df_tuits){
  
  #funcion auxiliar que detecta y transforma enlaces de tuiter
  #a fines de descartarlos o eventualmente trabajarlos en analisis posteriores
  #recibe un df de tuits que se espera estén contenidos en una variable "text"
  #devuelve un df en el que los enlaces de tuits han sido identificados como "enlacetuit" en el texto
  #reemplazando al estándar "https://t.co/"
  
  df_tuits_enlaces_resaltados <- df_tuits %>% 
    mutate(text = str_replace_all(text,"https://t.co/", "enlacetuit")) %>% 
    mutate(text = str_replace_all(text,"#", "hashtag")) %>% 
    mutate(text = str_replace_all(text,"@", "mention"))
  
  return(df_tuits_enlaces_resaltados)
}


# funciones agregadas 

tokenizarTextoTuits <- function(df_tuits, filtrar_campaña = TRUE, deshacerse_RT = TRUE, tipo_token = "words", grams = 2){
  
  ## recibe un df con tuits
  # se espera que el texto de los mismos esté contenido en una variable "text"
  ## devuelve su texto tokenizado
  ## optativo: no filtrar tuits de campaña (por variable campaña)
  
  if (isTRUE(filtrar_campaña)) { df_tuits <- df_tuits %>% subset( Campaña ==1 ) }
  if (isTRUE(deshacerse_RT)) { df_tuits <- df_tuits %>% subset( !str_detect(text, "^RT") )}
  
  seleccion_text <- df_tuits %>%  
    seleccionarTextoTuits() 
  
  if (tipo_token == "words") {
  seleccion_id_enlaces <- seleccion_text %>% 
    transformarEnlacesTwitter()
  
  seleccion_tokenizada <- seleccion_id_enlaces %>% 
    unnest_tokens(tokens, text) 
  }
  
  else if (tipo_token == "ngrams") {
    seleccion_id_enlaces <- seleccion_text %>% 
      transformarEnlacesTwitter() 
    
    seleccion_tokenizada <- seleccion_id_enlaces %>% 
      unnest_tokens(tokens, text, token = "ngrams", n = grams) 
    }
  
  else { 
    seleccion_tokenizada <- seleccion_text %>% 
      unnest_tokens(tokens, text, token = tipo_token) 
    } 
  
  return(seleccion_tokenizada)
}

limpiarTokens <- function(seleccion_tokenizada, bigramas = FALSE, lista_estandar= TRUE, largo = FALSE, palabras_web = FALSE, mentions = FALSE, hashtags = FALSE){
  
  #recibe una lista de tokens, en una variable tokens 
  # retira aquellos que consideramos innecesarios para el analisis:
  # en principio la lista de palabras que provee la funcion get_stopwords del paquete stopwords
  # optativo: borrar las palabras cortas (menos de tres caracteres)
  # tambien optativo: borrar enlaces de tuiter 
  # finalmente se ofrece la opción de limpiar bigramas (solo palabras stopwords)
  
  palabras_a_borrar <- stopwords("spanish")  # de tm. funciona mejor
  # palabras_a_borrar <- get_stopwords("es") %>%
  #   rename(tokens = "word")
  
  tokens_limpios <- seleccion_tokenizada
  
  if (isTRUE(bigramas)) {
    
    lista_estandar <- FALSE 
    
    bigrams_separated <- tokens_limpios %>%
      separate(tokens, c("word1", "word2"), sep = " ")
    
    bigrams_filtered <- bigrams_separated %>%
      filter(!word1 %in% palabras_a_borrar) %>%
      filter(!word2 %in% palabras_a_borrar)
    
    tokens_limpios <- bigrams_filtered %>%
      unite(tokens, word1, word2, sep = " ")
  }
  
  if (isTRUE(lista_estandar)){
    tokens_limpios <- tokens_limpios %>%
      filter(!(tokens %in% palabras_a_borrar))
  }
  
  if (isTRUE(largo)) {
    tokens_limpios <- tokens_limpios %>% 
    subset(str_length(tokens) > 3 & !(tokens == "no"))
  }
  
  if (isTRUE(palabras_web)) {
    tokens_limpios <- tokens_limpios %>% 
    subset(!str_detect(tokens, "(http)|(t.co)|(enlacetuit)"))  
  }
  if (isTRUE(mentions)) {
    tokens_limpios <- tokens_limpios %>% 
      subset(!str_detect(tokens, "(mention)|(@)"))  
  }
  if (isTRUE(hashtags)) {
    tokens_limpios <- tokens_limpios %>% 
      subset(!str_detect(tokens, "(hashtag)|(#)"))  
  }
  return(tokens_limpios)
  
}

l
```
{{</code>}}

{{< expandable label="Preparando Datos" level="2" >}}
```r
# tablasElectorales

## modulo con funciones
##para scrappear tablas web
## y extraer datos de resultados electorales
## las funciones globales están inspiradas en el reporte de datos provinciales de wikipedia
# pero las funciones de base pueden ser usadas en otros casos

#apertura de liberarias
##### 

require(tidyverse)
require(rvest) # extraer datos de html

# FUNCIONES

#PARA SCRAPPEAR TABLAS ELECTORALES


#de base

arbolTablas <- function(url){
  df_url_tables <-read_html(url) %>% 
    html_nodes("table")
  return(df_url_tables)
}

extraerTabla <- function(df_url_tables, nodo){
  
  ##extrae la tabla de interés
  ##y hace los primeros pasos de limpieza
  
  extracto_tabla <- html_table(df_url_tables[[nodo]], 
                               fill=TRUE, 
                               header=TRUE)  
  return(extracto_tabla)
}

borrarPrimeraFila <- function(extracto_tabla){
  
  extracto_tabla <- extracto_tabla[-c(1, nrow(extracto_tabla)), ]
  return(extracto_tabla)
}

reducirAnchoTabla <- function(extracto_tabla){
  
  ## función destinada a la limpieza de algunos atributos
  ## de las tablas obtenidas de wikipedia con datos sobre la elección a gobernador
  #en este caso: borramos columnas innecesarias
  
  columnas_a_borrar <- c(3) # en todos los casos la columna tres es innecesaria
  columna_a_borrar <- 0   # vamos a buscar otras columnas a borrar
  # en particular, 
  # hay algunas tablas que tienen columnas con nombres vacios, pero no todas. 
  # las limpiamos      
  for (i in colnames(extracto_tabla)) {      
    columna_a_borrar <- columna_a_borrar + 1 
    if (is.na(i)) { columnas_a_borrar <- append(columnas_a_borrar, columna_a_borrar)} 
  }
  
  extracto_tabla_corta <-  extracto_tabla[, -(columnas_a_borrar) ]
  return(extracto_tabla_corta)
}

omitNaTabla <- function(extracto_tabla_corta) {
  ## función destinada a la limpieza de algunos atributos
  ## de las tablas obtenidas de wikipedia con datos sobre la elección a gobernador
  #en este caso: omitimos filas con valores vacios 
  extracto_tabla_sin_nas <- na.omit(extracto_tabla_corta)
  return(extracto_tabla_sin_nas)
}

renombrarTabla <- function(extracto_tabla_sin_nas){
  
  ## función destinada a la limpieza de algunos atributos
  ## de las tablas obtenidas de wikipedia con datos sobre la elección a gobernador
  #en este caso: cambiamos nombres de las columnas
  extracto_tabla_renombrada <- extracto_tabla_sin_nas %>% 
    dplyr::rename(
      Candidato = 1,
      Vicecandidato = 2,
      Porcentaje = "%") 
  return(extracto_tabla_renombrada)
}

limpiezaTabla <- function(extracto_tabla_renombrada){
  
  ## función destinada a la limpieza de algunos atributos
  ## de las tablas obtenidas de wikipedia con datos sobre la elección a gobernador
  #en este caso:  formateamos variables de interés: votos y porcentaje
  
  extracto_tabla_limpia <- extracto_tabla_renombrada %>%  
    dplyr::mutate(Porcentaje = str_replace(Porcentaje,"\\%", "")) %>% 
    dplyr::mutate(Porcentaje = str_trim(
      str_replace(Porcentaje,"\\,", ".")
    ) ) %>% 
    dplyr::mutate(Porcentaje = as.numeric(Porcentaje)) %>% 
    dplyr::mutate(Votos = str_trim(
      str_replace_all(Votos,"\\.", "")
    ) ) %>% 
    dplyr::mutate(Votos = as.numeric(Votos))
  
  
  return(extracto_tabla_limpia)
  
}

agregarVotosGobernador <- function(tabla_limpia){ 
  #calcula los votos agregados de la fórmula
  #para tablas que reportan de manera separada cada lista
  
  votos_afirmativos <- sum(tabla_limpia$Votos)
  
  tabla_summarized <- tabla_limpia %>% 
    group_by(Candidato) %>% 
    dplyr::summarise(Votos = sum(Votos),
              Porcentaje = sum(Votos)/votos_afirmativos*100)
  
  return(tabla_summarized)
}

reducirLargoTabla <- function(tabla_limpia_agregada){
  
  #se queda solamente 
  # con los votos obtenidos por cada candidato
  
  tabla_reducida <- tabla_limpia_agregada %>% 
    subset(!str_detect(
      tabla_limpia_agregada$Candidato, "(Vot)|(Elect)|(Particip)|(Tot)|(Absten)")) 
  
  return(tabla_reducida)
}

agregarColumnas <- function(tabla_reducida, nombre_distrito = ""){
  
  #recibe una tabla limpia. la transforma: 
  #agrega ranking y nombre de distrito, si es que se provee uno (chr)
  
  #ordena por porcentaje y agrega ranking
  
  tabla_reducida <- tabla_reducida %>%arrange(desc(Porcentaje)) 
  tabla_reducida$Ranking <- 1:nrow(tabla_reducida)
  
  # si se ingresó, agrega nombre de distrito
  
  if (nchar(nombre_distrito) > 0) {tabla_reducida$Distrito <- nombre_distrito}
  
  return(tabla_reducida)
}

#funciones agregadas

extraer_datos_wiki <- function(url, nodo){
  #recibe un url de wikipedia y un nodo
  #devuelve la tabla del nodo seleccionado en crudo
  
  arbol_tablas <- arbolTablas(url) 
  tabla_cruda <- extraerTabla(arbol_tablas, nodo)
  return(tabla_cruda)
}

procesar_datos_wiki <- function(tabla_cruda, nombre_distrito = "") {
  
  #función que recibe una tabla cruda
  #con datos de resultados electorales 
  #extraidos de wikipedia
  # y devuelve una tabla limpia y reducida con los datos que necesitamos para trabajar
  # en este caso consisten en solamente las filas con los nombres de los candidatos
  #además, si se desea,
  #agrega una columna con el nombre del distrito ( ingresar una cadena, tipo chr )
  
  tabla_s_primera_fila <- borrarPrimeraFila(tabla_cruda)
  tabla_corta <- reducirAnchoTabla(tabla_s_primera_fila)
  tabla_sin_nas <- omitNaTabla(tabla_corta)
  tabla_renombrada <- renombrarTabla(tabla_sin_nas)
  tabla_limpia <- limpiezaTabla(tabla_renombrada)
  tabla_reducida <- reducirLargoTabla(tabla_limpia)
  tabla_lista <- agregarColumnas(tabla_reducida, nombre_distrito)
  
  return(tabla_lista)
}
```
{{< /expandable >}}
