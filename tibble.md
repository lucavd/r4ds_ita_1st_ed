# Tibble

## Introduzione

In questo libro lavoriamo con i "tibble" invece dei tradizionali `data.frame` di R. I tibble _sono_ data frame, ma modificano alcuni vecchi comportamenti per renderci la vita un po' più facile. R è un vecchio linguaggio, e alcune cose che erano utili 10 o 20 anni fa ora intralciano. È difficile cambiare la base di R senza rompere il codice esistente, così la maggior parte dell'innovazione avviene nei pacchetti. Qui descriveremo il pacchetto __tibble__, che fornisce frame di dati opinionati che rendono il lavoro nel tidyverse un po' più facile. Nella maggior parte dei punti, userò il termine tibble e data frame in modo intercambiabile; quando voglio attirare l'attenzione sui data frame integrati in R, li chiamerò `data.frame`.

Se questo capitolo vi lascia la voglia di imparare di più sulle tibbie, potreste godervi la `vignette("tibble")`.

### Prerequisiti

In questo capitolo esploreremo il pacchetto __tibble__, parte del core tidyverse.


```r
library(tidyverse)
```

## Creare una tibble

Quasi tutte le funzioni che userete in questo libro producono tibble, poiché le tibble sono una delle caratteristiche unificanti del tidyverse. La maggior parte degli altri pacchetti di R usa normali data frame, quindi potreste voler forzare un data frame in una tibble. Potete farlo con `as_tibble()`:


```r
as_tibble(iris)
#> # A tibble: 150 × 5
#>   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
#>          <dbl>       <dbl>        <dbl>       <dbl> <fct>  
#> 1          5.1         3.5          1.4         0.2 setosa 
#> 2          4.9         3            1.4         0.2 setosa 
#> 3          4.7         3.2          1.3         0.2 setosa 
#> 4          4.6         3.1          1.5         0.2 setosa 
#> 5          5           3.6          1.4         0.2 setosa 
#> 6          5.4         3.9          1.7         0.4 setosa 
#> # … with 144 more rows
```

Potete creare un nuovo tibble da vettori individuali con `tibble()`. La funzione `tibble()` ricicla automaticamente gli input di lunghezza 1, e vi permette di fare riferimento alle variabili che avete appena creato, come mostrato qui sotto.


```r
tibble(
  x = 1:5, 
  y = 1, 
  z = x ^ 2 + y
)
#> # A tibble: 5 × 3
#>       x     y     z
#>   <int> <dbl> <dbl>
#> 1     1     1     2
#> 2     2     1     5
#> 3     3     1    10
#> 4     4     1    17
#> 5     5     1    26
```

Se avete già familiarità con `data.frame()`, notate che `tibble()` fa molto meno: non cambia mai il tipo di input (ad esempio non converte mai le stringhe in fattori!), non cambia mai i nomi delle variabili, e non crea mai nomi di righe.

È possibile che un tibble abbia nomi di colonne che non sono nomi di variabili R validi, anche detti nomi __non-sintattici__. Per esempio, potrebbero non iniziare con una lettera, o potrebbero contenere caratteri insoliti come uno spazio. Per fare riferimento a queste variabili, avete bisogno di circondarle con i backtick, `` ` ``:


```r
tb <- tibble(
  `:)` = "smile", 
  ` ` = "space",
  `2000` = "number"
)
tb
#> # A tibble: 1 × 3
#>   `:)`  ` `   `2000`
#>   <chr> <chr> <chr> 
#> 1 smile space number
```

Avrete anche bisogno dei backtick quando lavorerete con queste variabili in altri pacchetti, come ggplot2, dplyr e tidyr.

Un altro modo per creare un tibble è con `tribble()`, abbreviazione di **tr**ansposed tibble.  `tribble()` è personalizzato per l'inserimento dei dati nel codice: le intestazioni delle colonne sono definite da formule (cioè iniziano con `~`), e le voci sono separate da virgole. Questo rende possibile la disposizione di piccole quantità di dati in forma facilmente leggibile.


```r
tribble(
  ~x, ~y, ~z,
  #--|--|----
  "a", 2, 3.6,
  "b", 1, 8.5
)
#> # A tibble: 2 × 3
#>   x         y     z
#>   <chr> <dbl> <dbl>
#> 1 a         2   3.6
#> 2 b         1   8.5
```

Spesso aggiungo un commento (la linea che inizia con `#`), per rendere davvero chiaro dove si trova l'intestazione.

## Tibble vs. data.frame

Ci sono due differenze principali nell'uso di un tibble rispetto a un classico `data.frame`: stampa e sottoinsieme.

### Stampa

Le tibble hanno un metodo di stampa raffinato che mostra solo le prime 10 righe e tutte le colonne che si adattano allo schermo. Questo rende molto più facile lavorare con grandi quantità di dati. Oltre al suo nome, ogni colonna riporta il suo tipo, una bella caratteristica presa in prestito da `str()`:


```r
tibble(
  a = lubridate::now() + runif(1e3) * 86400,
  b = lubridate::today() + runif(1e3) * 30,
  c = 1:1e3,
  d = runif(1e3),
  e = sample(letters, 1e3, replace = TRUE)
)
#> # A tibble: 1,000 × 5
#>   a                   b              c     d e    
#>   <dttm>              <date>     <int> <dbl> <chr>
#> 1 2022-06-21 23:28:11 2022-06-28     1 0.368 n    
#> 2 2022-06-22 17:33:21 2022-07-03     2 0.612 l    
#> 3 2022-06-22 11:57:00 2022-07-13     3 0.415 p    
#> 4 2022-06-22 01:18:17 2022-07-12     4 0.212 m    
#> 5 2022-06-21 21:42:34 2022-07-09     5 0.733 i    
#> 6 2022-06-22 08:43:31 2022-07-05     6 0.460 n    
#> # … with 994 more rows
```

Le tibble sono progettate in modo da non sovraccaricare accidentalmente la vostra console quando stampate grandi frame di dati. Ma a volte avete bisogno di più output rispetto alla visualizzazione di default. Ci sono alcune opzioni che possono aiutare.

Per prima cosa, potete esplicitamente `print()` il data frame e controllare il numero di righe (`n`) e la `larghezza` della visualizzazione. La `larghezza = Inf` visualizzerà tutte le colonne:


```r
nycflights13::flights %>% 
  print(n = 10, width = Inf)
```

Potete anche controllare il comportamento di stampa predefinito impostando le opzioni:

* `options(tibble.print_max = n, tibble.print_min = m)`: se più di `n`
  righe, stampa solo righe `m`. Usa `options(tibble.print_min = Inf)` per mostrare sempre
  mostrare sempre tutte le righe.

* Usa `options(tibble.width = Inf)` per stampare sempre tutte le colonne, indipendentemente
  della larghezza dello schermo.

Puoi vedere una lista completa di opzioni guardando l'aiuto del pacchetto con `package?tibble`.

Un'ultima opzione è quella di usare il visualizzatore di dati integrato in RStudio per ottenere una vista a scorrimento dell'intero set di dati. Questo è spesso utile alla fine di una lunga catena di manipolazioni.


```r
nycflights13::flights %>% 
  View()
```

### Subsetting

Finora tutti gli strumenti che avete imparato hanno lavorato con data frame completi. Se volete estrarre una singola variabile, avete bisogno di alcuni nuovi strumenti, `$` e `[[`. `[[` può estrarre per nome o per posizione; `$` estrae solo per nome, ma è un po' meno digitabile.


```r
df <- tibble(
  x = runif(5),
  y = rnorm(5)
)

# Extract by name
df$x
#> [1] 0.73296674 0.23436542 0.66035540 0.03285612 0.46049161
df[["x"]]
#> [1] 0.73296674 0.23436542 0.66035540 0.03285612 0.46049161

# Extract by position
df[[1]]
#> [1] 0.73296674 0.23436542 0.66035540 0.03285612 0.46049161
```

Per usarli in una pipe, dovrete usare il segnaposto speciale `.`:


```r
df %>% .$x
#> [1] 0.73296674 0.23436542 0.66035540 0.03285612 0.46049161
df %>% .[["x"]]
#> [1] 0.73296674 0.23436542 0.66035540 0.03285612 0.46049161
```

Rispetto a un `data.frame`, le tibble sono più rigide: non fanno mai corrispondenze parziali e generano un avviso se la colonna a cui si sta cercando di accedere non esiste.

## Interagire con il vecchio codice

Alcune vecchie funzioni non funzionano con le tibble. Se incontrate una di queste funzioni, usate `as.data.frame()` per trasformare un tibble in un `data.frame`:


```r
class(as.data.frame(tb))
#> [1] "data.frame"
```

La ragione principale per cui alcune vecchie funzioni non funzionano con tibble è la funzione `[`.  Non usiamo molto la funzione `[` in questo libro perché `dplyr::filter()` e `dplyr::select()` vi permettono di risolvere gli stessi problemi con un codice più chiaro (ma ne imparerete qualcosa in [sottoinsiemi di vettori](#vector-subsetting)). Con i data frame di base di R, `[` a volte restituisce un data frame e a volte un vettore. Con le tibble, `[` restituisce sempre un'altra tibble.

## Esercizi

1.  Come potete dire se un oggetto è una tibla? (Suggerimento: provate a stampare `mtcars`,
    che è un normale data frame). 

1.  Confronta e contrasta le seguenti operazioni su un `data.frame` e 
    tibble equivalente. Cosa c'è di diverso? Perché i comportamenti predefiniti dei data frame
    vi causano frustrazione?
    
    
    ```r
    df <- data.frame(abc = 1, xyz = "a")
    df$x
    df[, "xyz"]
    df[, c("abc", "xyz")]
    ```

1.  Se avete il nome di una variabile memorizzata in un oggetto, ad esempio `var <- "mpg"`,
    come potete estrarre la variabile di riferimento da una tibla?

1.  Esercitati a fare riferimento a nomi non sintattici nel seguente data frame:

    1.  Estrarre la variabile chiamata `1`.

    1.  Tracciare uno scatterplot di `1` contro `2`.

    1.  Creare una nuova colonna chiamata `3` che è `2` divisa per `1`.
        
    1.  Rinominare le colonne in `one`, `two` e `three`. 
    
    
    ```r
    annoying <- tibble(
      `1` = 1:10,
      `2` = `1` * 2 + rnorm(length(`1`))
    )
    ```

1.  Cosa fa `tibble::enframe()`? Quando potreste usarlo?

1.  Quale opzione controlla quanti nomi di colonna aggiuntivi vengono stampati
    a piè di pagina di una tibble?
