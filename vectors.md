# Vettori

## Introduzione

Finora questo libro si è concentrato sulle tibble e sui pacchetti che lavorano con esse. Ma quando inizierete a scrivere le vostre funzioni e a scavare più a fondo in R, avrete bisogno di imparare i vettori, gli oggetti che sono alla base delle tibble. Se avete imparato R in modo più tradizionale, probabilmente avete già familiarità con i vettori, dato che la maggior parte delle risorse di R inizia con i vettori e si fa strada fino alle tibble. Penso che sia meglio iniziare con le tibble perché sono immediatamente utili, e poi lavorare fino ai componenti sottostanti.

I vettori sono particolarmente importanti perché la maggior parte delle funzioni che scriverete lavoreranno con i vettori. È possibile scrivere funzioni che lavorano con le tibbie (come ggplot2, dplyr e tidyr), ma gli strumenti necessari per scrivere tali funzioni sono attualmente idiosincratici e immaturi. Sto lavorando ad un approccio migliore, <https://github.com/hadley/lazyeval>, ma non sarà pronto in tempo per la pubblicazione del libro. Anche quando sarà completo, avrete ancora bisogno di capire i vettori, renderà solo più facile scrivere un livello user-friendly sopra.

### Prerequisiti

Il focus di questo capitolo è sulle strutture dati di base di R, quindi non è essenziale caricare alcun pacchetto. Tuttavia, useremo una manciata di funzioni del pacchetto __purrr__ per evitare alcune incongruenze in R base.


```r
library(tidyverse)
```

## Nozioni di base sui vettori

Ci sono due tipi di vettori:

1. 1. Vettori __atomic__, di cui esistono sei tipi:
  __logical__, __integer__, __double__, __character__, __complex__, e 
  __raw__. I vettori interi e doppi sono conosciuti collettivamente come
  vettori __numerici__. 

1. __Liste__, che a volte sono chiamate vettori ricorsivi perché le liste possono contenere altre liste. 

La differenza principale tra vettori atomici e liste è che i vettori atomici sono __omogenei__, mentre le liste possono essere __eterogenee__. C'è un altro oggetto correlato: `NULL`. `NULL` è spesso usato per rappresentare l'assenza di un vettore (al contrario di `NA` che è usato per rappresentare l'assenza di un valore in un vettore). `NULL` si comporta tipicamente come un vettore di lunghezza 0. La figura \@ref(fig:datatypes) riassume le interrelazioni. 

<div class="figure" style="text-align: center">
<img src="diagrams/data-structures-overview.png" alt="The hierarchy of R's vector types" width="50%" />
<p class="caption">(\#fig:datatypes)The hierarchy of R's vector types</p>
</div>

Ogni vettore ha due proprietà chiave:

1.  Il suo __tipo__, che potete determinare con `typeof()`.

    
    ```r
    typeof(letters)
    #> [1] "character"
    typeof(1:10)
    #> [1] "integer"
    ```

1. La sua __length__, che potete determinare con `length()`.

    
    ```r
    x <- list("a", "b", 1:10)
    length(x)
    #> [1] 3
    ```

I vettori possono anche contenere metadati aggiuntivi arbitrari sotto forma di attributi. Questi attributi sono usati per creare __vettori aumentati__ che si basano su comportamenti aggiuntivi. Ci sono tre tipi importanti di vettori aumentati:

* I fattori sono costruiti sopra i vettori interi.
* Date e date-ora sono costruiti sopra vettori numerici.
* I data frame e le tibble sono costruiti sopra le liste.

Questo capitolo vi introdurrà a questi importanti vettori dal più semplice al più complicato. Inizierete con i vettori atomici, poi arriverete alle liste, e finirete con i vettori aumentati.

## Tipi importanti di vettori atomici

I quattro tipi più importanti di vettore atomico sono logico, intero, doppio e carattere. Raw e complex sono usati raramente durante un'analisi dei dati, quindi non li discuterò qui.

### Logico

I vettori logici sono il tipo più semplice di vettore atomico perché possono assumere solo tre possibili valori: `FALSE`, `TRUE` e `NA`. I vettori logici sono solitamente costruiti con operatori di confronto, come descritto in [comparatori]. Puoi anche crearli a mano con `c()`:


```r
1:10 %% 3 == 0
#>  [1] FALSE FALSE  TRUE FALSE FALSE  TRUE FALSE FALSE  TRUE FALSE

c(TRUE, TRUE, FALSE, NA)
#> [1]  TRUE  TRUE FALSE    NA
```

### Numerico

I vettori interi e doppi sono conosciuti collettivamente come vettori numerici. In R, i numeri sono doppi per default. Per fare un intero, metti una `L` dopo il numero:


```r
typeof(1)
#> [1] "double"
typeof(1L)
#> [1] "integer"
1.5L
#> [1] 1.5
```

La distinzione tra interi e double non è solitamente importante, ma ci sono due importanti differenze di cui dovreste essere consapevoli:

1.  I double sono approssimazioni. I double rappresentano numeri in virgola mobile che non possono essere sempre rappresentati con precisione con una quantità fissa di memoria. Questo significa che dovreste considerare tutti i double come approssimazioni. Per esempio, cos'è il quadrato della radice quadrata di due?

    
    ```r
    x <- sqrt(2) ^ 2
    x
    #> [1] 2
    x - 2
    #> [1] 4.440892e-16
    ```

    Questo comportamento è comune quando si lavora con i numeri in virgola mobile: la maggior parte
    calcoli includono qualche errore di approssimazione. Invece di confrontare i numeri in virgola mobile
    in virgola mobile usando `==`, dovreste usare `dplyr::near()` che permette 
    qualche tolleranza numerica.

1.  Gli interi hanno un valore speciale: `NA`, mentre i double ne hanno quattro:
    `NA`, `NaN`, `Inf` e `-Inf`. Tutti e tre i valori speciali `NaN`, `Inf` e `-Inf` possono presentarsi durante la divisione:
   
    
    ```r
    c(-1, 0, 1) / 0
    #> [1] -Inf  NaN  Inf
    ```

    Evita di usare `==` per controllare questi altri valori speciali. Usate invece le 
    funzioni di aiuto `is.finite()`, `is.infinite()`, e `is.nan()`:
    
    |                  |  0  | Inf | NA  | NaN |
    |------------------|-----|-----|-----|-----|
    | `is.finite()`    |  x  |     |     |     |
    | `is.infinite()`  |     |  x  |     |     |
    | `is.na()`        |     |     |  x  |  x  |
    | `is.nan()`       |     |     |     |  x  |

### Carattere

I vettori di caratteri sono il tipo più complesso di vettore atomico, perché ogni elemento di un vettore di caratteri è una stringa, e una stringa può contenere una quantità arbitraria di dati.

Hai già imparato molto su come lavorare con le stringhe in [stringhe]. Qui volevo menzionare una caratteristica importante dell'implementazione delle stringhe sottostanti: R usa un pool globale di stringhe. Questo significa che ogni stringa unica è immagazzinata in memoria solo una volta, e ogni uso della stringa punta a quella rappresentazione. Questo riduce la quantità di memoria necessaria alle stringhe duplicate. Potete vedere questo comportamento in pratica con `pryr::object_size()`:


```r
x <- "This is a reasonably long string."
pryr::object_size(x)
#> 152 B

y <- rep(x, 1000)
pryr::object_size(y)
#> 8.14 kB
```

`y` non occupa 1000 volte più memoria di `x`, perché ogni elemento di `y` è solo un puntatore a quella stessa stringa. Un puntatore è di 8 byte, quindi 1000 puntatori a una stringa di 152 B sono 8 * 1000 + 152 = 8.14 kB.

### Valori mancanti

Nota che ogni tipo di vettore atomico ha il suo valore mancante:


```r
NA            # logical
#> [1] NA
NA_integer_   # integer
#> [1] NA
NA_real_      # double
#> [1] NA
NA_character_ # character
#> [1] NA
```

Normalmente non avete bisogno di conoscere questi diversi tipi perché potete sempre usare `NA` e sarà convertito nel tipo corretto usando le regole di coercizione implicite descritte in seguito. Tuttavia, ci sono alcune funzioni che sono rigide riguardo ai loro input, quindi è utile avere questa conoscenza in tasca in modo da poter essere specifici quando necessario.

### Esercizi

1.  Descrivete la differenza tra `is.finite(x)` e `!is.infinite(x)`.

1.  Leggete il codice sorgente di `dplyr::near()` (Suggerimento: per vedere il codice sorgente,
    eliminate il `()`). Come funziona? 

1.  Un vettore logico può assumere 3 possibili valori. Quanti valori possibili
    valori possibili può assumere un vettore intero? Quanti valori possibili può assumere
    prendere un double? Usa Google per fare qualche ricerca.

1.  Inventa almeno quattro funzioni che ti permettono di convertire un doppio in un
    intero. In cosa differiscono? Sii preciso.
    
1.  Quali funzioni del pacchetto readr ti permettono di trasformare una stringa
    in un vettore logico, intero e double?

## Usare i vettori atomici

Ora che hai capito i diversi tipi di vettore atomico, è utile rivedere alcuni degli strumenti importanti per lavorare con essi. Questi includono:

1.  Come convertire da un tipo all'altro e quando ciò avviene
    automaticamente.

1.  Come capire se un oggetto è un tipo specifico di vettore.

1.  Cosa succede quando si lavora con vettori di diversa lunghezza.

1.  Come nominare gli elementi di un vettore.

1.  Come estrarre gli elementi di interesse.

### Coercizione

Ci sono due modi per convertire, o coercere, un tipo di vettore in un altro:

1.  La coercizione esplicita avviene quando si chiama una funzione come `as.logical()`,
    `as.integer()`, `as.double()`, o `as.character()`. Ogni volta che vi trovate
    te di usare la coercizione esplicita, dovresti sempre controllare se è possibile
    fare la correzione a monte, in modo che il vettore non abbia mai avuto il tipo sbagliato in 
    all'inizio. Per esempio, potrebbe essere necessario modificare la specifica di readr 
    `col_types`.

1.  La coercizione implicita avviene quando si usa un vettore in un contesto specifico
    che si aspetta un certo tipo di vettore. Per esempio, quando usate un vettore logico
    logico con una funzione di riepilogo numerico, o quando si usa un vettore doppio
    dove ci si aspetta un vettore intero.
    
Poiché la coercizione esplicita è usata relativamente raramente, ed è in gran parte facile da capire, mi concentrerò sulla coercizione implicita qui. 

Avete già visto il tipo più importante di coercizione implicita: usare un vettore logico in un contesto numerico. In questo caso `TRUE` è convertito in `1` e `FALSE` convertito in `0`. Ciò significa che la somma di un vettore logico è il numero di veri, e la media di un vettore logico è la proporzione di veri:


```r
x <- sample(20, 100, replace = TRUE)
y <- x > 10
sum(y)  # quanti sono maggiori di 10?
#> [1] 38
mean(y) # quale proporzione è maggiore di 10?
#> [1] 0.38
```

Potreste vedere del codice (tipicamente più vecchio) che si basa sulla coercizione implicita nella direzione opposta, da intero a logico:


```r
if (length(x)) {
  # fa qualcosa
}
```

In questo caso, 0 è convertito in `FALSE` e tutto il resto è convertito in `TRUE`. Penso che questo renda più difficile capire il tuo codice, e non lo consiglio. Sii invece esplicito: `lunghezza(x) > 0`.

È anche importante capire cosa succede quando provate a creare un vettore contenente più tipi con `c()`: il tipo più complesso vince sempre.


```r
typeof(c(TRUE, 1L))
#> [1] "integer"
typeof(c(1L, 1.5))
#> [1] "double"
typeof(c(1.5, "a"))
#> [1] "character"
```

Un vettore atomico non può avere un mix di tipi diversi perché il tipo è una proprietà del vettore completo, non dei singoli elementi. Se hai bisogno di mescolare più tipi nello stesso vettore, dovresti usare una lista, che imparerai a conoscere tra poco.

### Funzioni di test

A volte vuoi fare cose diverse in base al tipo di vettore. Un'opzione è usare `typeof()`. Un'altra è usare una funzione di test che restituisca un `TRUE` o un `FALSE`. Base R fornisce molte funzioni come `is.vector()` e `is.atomic()`, ma spesso restituiscono risultati sorprendenti. Invece, è più sicuro usare le funzioni `is_*` fornite da purrr, che sono riassunte nella tabella qui sotto.

|                  | lgl | int | dbl | chr | list |
|------------------|-----|-----|-----|-----|------|
| `is_logical()`   |  x  |     |     |     |      |
| `is_integer()`   |     |  x  |     |     |      |
| `is_double()`    |     |     |  x  |     |      |
| `is_numeric()`   |     |  x  |  x  |     |      |
| `is_character()` |     |     |     |  x  |      |
| `is_atomic()`    |  x  |  x  |  x  |  x  |      |
| `is_list()`      |     |     |     |     |  x   |
| `is_vector()`    |  x  |  x  |  x  |  x  |  x   |

### Scalari e regole di riciclaggio

Oltre a costringere implicitamente i tipi di vettori ad essere compatibili, R costringerà implicitamente anche la lunghezza dei vettori. Questo è chiamato riciclo dei vettori, perché il vettore più corto viene ripetuto, o riciclato, alla stessa lunghezza del vettore più lungo.

Questo è generalmente più utile quando si mescolano vettori e "scalari". Ho messo gli scalari tra virgolette perché R in realtà non ha scalari: invece, un singolo numero è un vettore di lunghezza 1. Poiché non ci sono scalari, la maggior parte delle funzioni built-in sono __vettorizzate__, il che significa che opereranno su un vettore di numeri. Ecco perché, per esempio, questo codice funziona:


```r
sample(10) + 100
#>  [1] 107 104 103 109 102 101 106 110 105 108
runif(10) > 0.5
#>  [1] FALSE  TRUE FALSE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
```

In R, le operazioni matematiche di base lavorano con i vettori. Ciò significa che non dovreste mai aver bisogno di eseguire un'iterazione esplicita quando eseguite semplici calcoli matematici.

È intuitivo ciò che dovrebbe accadere se si aggiungono due vettori della stessa lunghezza, o un vettore e uno "scalare", ma cosa succede se si aggiungono due vettori di lunghezza diversa?


```r
1:10 + 1:2
#>  [1]  2  4  4  6  6  8  8 10 10 12
```

Qui, R espanderà il vettore più breve alla stessa lunghezza del più lungo, il cosiddetto riciclo. Questo è silenzioso tranne quando la lunghezza del più lungo non è un multiplo intero della lunghezza del più corto:


```r
1:10 + 1:3
```

Mentre il riciclo vettoriale può essere usato per creare codice molto succinto e intelligente, può anche nascondere silenziosamente dei problemi. Per questo motivo, le funzioni vettoriali di tidyverse daranno errore quando si ricicla qualcosa che non sia uno scalare. Se volete riciclare, dovrete farlo voi stessi con `rep()`:


```r
tibble(x = 1:4, y = 1:2)
#> Error:
#> ! Tibble columns must have compatible sizes.
#> • Size 4: Existing data.
#> • Size 2: Column `y`.
#> ℹ Only values of size one are recycled.

tibble(x = 1:4, y = rep(1:2, 2))
#> # A tibble: 4 × 2
#>       x     y
#>   <int> <int>
#> 1     1     1
#> 2     2     2
#> 3     3     1
#> 4     4     2

tibble(x = 1:4, y = rep(1:2, each = 2))
#> # A tibble: 4 × 2
#>       x     y
#>   <int> <int>
#> 1     1     1
#> 2     2     1
#> 3     3     2
#> 4     4     2
```

### Denominazione dei vettori

Tutti i tipi di vettori possono essere nominati. Puoi nominarli durante la creazione con `c()`:


```r
c(x = 1, y = 2, z = 4)
#> x y z 
#> 1 2 4
```

O dopo averlo fatto, con `purrr::set_names()`:


```r
set_names(1:3, c("a", "b", "c"))
#> a b c 
#> 1 2 3
```

I vettori nominati sono molto utili per il subsetting, descritto di seguito.

### Sottoinsiemi {#vector-subsetting}

Finora abbiamo usato `dplyr::filter()` per filtrare le righe in una tibla. Il `filtro()` funziona solo con le tibbie, quindi avremo bisogno di un nuovo strumento per i vettori: `[`. `[` è la funzione di sottoinsieme, e si chiama come `x[a]`. Ci sono quattro tipi di cose con cui è possibile subsettare un vettore:

1.  Un vettore numerico contenente solo numeri interi. I numeri interi devono essere o tutti 
    positivi, tutti negativi o zero.
    
    Il sottoinsieme con i numeri interi positivi mantiene gli elementi in quelle posizioni:
    
    
    ```r
    x <- c("one", "two", "three", "four", "five")
    x[c(3, 2, 5)]
    #> [1] "three" "two"   "five"
    ```
    
    Ripetendo una posizione, si può effettivamente fare un output più lungo dell'input:
    
    
    ```r
    x[c(1, 1, 5, 5, 5, 2)]
    #> [1] "one"  "one"  "five" "five" "five" "two"
    ```
    
    I valori negativi fanno cadere gli elementi nelle posizioni specificate:
    
    
    ```r
    x[c(-1, -3, -5)]
    #> [1] "two"  "four"
    ```
    
    È un errore mischiare valori positivi e negativi:
    
    
    ```r
    x[c(1, -1)]
    #> Error in x[c(1, -1)]: only 0's may be mixed with negative subscripts
    ```

    Il messaggio di errore menziona il subsetting con zero, che non restituisce alcun valore:
    
    
    ```r
    x[0]
    #> character(0)
    ```
    
    Questo non è utile molto spesso, ma può essere utile se volete creare 
    strutture di dati insolite con cui testare le vostre funzioni.
  
1.  Il sottoinsieme con un vettore logico mantiene tutti i valori corrispondenti ad un
    valore `TRUE`. Questo è più spesso utile in combinazione con le 
    funzioni di confronto.
    
    
    ```r
    x <- c(10, 3, NA, 5, 8, 1, NA)
    
    # Tutti i valori non mancanti di x
    x[!is.na(x)]
    #> [1] 10  3  5  8  1
    
    # Tutti i valori pari (o mancanti!) di x
    x[x %% 2 == 0]
    #> [1] 10 NA  8 NA
    ```

1.  Se avete un vettore di nome, potete sottoinvestirlo con un vettore di caratteri:
    
    
    ```r
    x <- c(abc = 1, def = 2, xyz = 5)
    x[c("xyz", "def")]
    #> xyz def 
    #>   5   2
    ```
    
    Come con gli interi positivi, potete anche usare un vettore di caratteri per 
    duplicare singole voci.

1.  Il tipo più semplice di sottoinsieme è il nulla, `x[]`, che restituisce il 
    completo `x`. Questo non è utile per il sottoinsieme di vettori, ma è utile
    quando si sottopongono matrici (e altre strutture ad alta dimensione) perché
    permette di selezionare tutte le righe o tutte le colonne, lasciando
    indice vuoto. Per esempio, se `x` è 2d, `x[1, ]` seleziona la prima riga e 
    tutte le colonne, e `x[, -1]` seleziona tutte le righe e tutte le colonne tranne
    la prima.
    
Per saperne di più sulle applicazioni del subsetting, leggete il capitolo "Subsetting" di _Advanced R_: <http://adv-r.had.co.nz/Subsetting.html#applications>.

C'è un'importante variazione di `[` chiamata `[[`. `[[` estrae sempre e solo un singolo elemento, e abbandona sempre i nomi. È una buona idea usarla ogni volta che volete rendere chiaro che state estraendo un singolo elemento, come in un ciclo for. La distinzione tra `[` e `[[` è più importante per le liste, come vedremo tra poco.

### Esercizi

1.  Cosa ti dice il `mean(is.na(x))` di un vettore `x`? 2. Che dire di
    `sum(!is.finite(x))`?

1.  Leggete attentamente la documentazione di `is.vector()`. Cosa verifica effettivamente
    verifica? Perché `is.atomic()` non concorda con la definizione di 
    vettori atomici di cui sopra?
    
1.  Confronta e contrasta `setNames()` con `purrr::set_names()`.

1.  Creare funzioni che prendono un vettore come input e restituiscono:
    
    1. L'ultimo valore.  Dovreste usare `[` o `[[`?

    1. Gli elementi nelle posizioni pari.
    
    1. Ogni elemento tranne l'ultimo valore.
    
    1. Solo i numeri pari (e nessun valore mancante).

1.  Perché `x[-che(x > 0)]`non è lo stesso di `x[x <= 0]`? 

1.  Cosa succede quando si sottoinveste con un numero intero positivo più grande
    della lunghezza del vettore? Cosa succede quando si effettua un sottoinsieme con un 
    nome che non esiste?

## Vettori ricorsivi (liste) {#liste}

Le liste sono un passo avanti nella complessità rispetto ai vettori atomici, perché le liste possono contenere altre liste. Questo le rende adatte a rappresentare strutture gerarchiche o ad albero. Si crea una lista con `list()`:


```r
x <- list(1, 2, 3)
x
#> [[1]]
#> [1] 1
#> 
#> [[2]]
#> [1] 2
#> 
#> [[3]]
#> [1] 3
```

Uno strumento molto utile per lavorare con le liste è `str()` perché si concentra sulla **struttura**, non sul contenuto.


```r
str(x)
#> List of 3
#>  $ : num 1
#>  $ : num 2
#>  $ : num 3

x_named <- list(a = 1, b = 2, c = 3)
str(x_named)
#> List of 3
#>  $ a: num 1
#>  $ b: num 2
#>  $ c: num 3
```

A differenza dei vettori atomici, `list()` può contenere un mix di oggetti:


```r
y <- list("a", 1L, 1.5, TRUE)
str(y)
#> List of 4
#>  $ : chr "a"
#>  $ : int 1
#>  $ : num 1.5
#>  $ : logi TRUE
```

Le liste possono anche contenere altre liste!


```r
z <- list(list(1, 2), list(3, 4))
str(z)
#> List of 2
#>  $ :List of 2
#>   ..$ : num 1
#>   ..$ : num 2
#>  $ :List of 2
#>   ..$ : num 3
#>   ..$ : num 4
```

### Visualizzazione delle liste

Per spiegare le funzioni di manipolazione delle liste più complicate, è utile avere una rappresentazione visiva delle liste. Per esempio, prendete queste tre liste:


```r
x1 <- list(c(1, 2), c(3, 4))
x2 <- list(list(1, 2), list(3, 4))
x3 <- list(1, list(2, list(3)))
```

Li disegnerò come segue:

<img src="diagrams/lists-structure.png" width="75%" style="display: block; margin: auto;" />

Ci sono tre principi:

1.  Le liste hanno angoli arrotondati. I vettori atomici hanno angoli quadrati.
  
1.  I figli sono disegnati all'interno del loro genitore e hanno uno sfondo
    sfondo per rendere più facile vedere la gerarchia.
  
1.  L'orientamento dei figli (cioè righe o colonne) non è importante, 
    quindi sceglierò un orientamento di riga o di colonna per risparmiare spazio o per illustrare 
    una proprietà importante nell'esempio.

### Sottoinsieme

Ci sono tre modi per suddividere una lista, che illustrerò con una lista chiamata `a`:


```r
a <- list(a = 1:3, b = "a string", c = pi, d = list(-1, -5))
```

* `[` estrae una sotto-lista. Il risultato sarà sempre una lista.

    
    ```r
    str(a[1:2])
    #> List of 2
    #>  $ a: int [1:3] 1 2 3
    #>  $ b: chr "a string"
    str(a[4])
    #> List of 1
    #>  $ d:List of 2
    #>   ..$ : num -1
    #>   ..$ : num -5
    ```
    
    Come con i vettori, potete sottoporre a subset un vettore logico, intero o di caratteri.
    
* `[[` estrae un singolo componente da una lista. Rimuove un livello di gerarchia dalla lista.

    
    ```r
    str(a[[1]])
    #>  int [1:3] 1 2 3
    str(a[[4]])
    #> List of 2
    #>  $ : num -1
    #>  $ : num -5
    ```

* `$` è un'abbreviazione per estrarre elementi nominati di una lista. Funziona in modo simile a `[[` eccetto che non c'è bisogno di usare le virgolette.
    
    
    ```r
    a$a
    #> [1] 1 2 3
    a[["a"]]
    #> [1] 1 2 3
    ```

La distinzione tra `[` e `[[` è davvero importante per le liste, perché `[[` si addentra nella lista mentre `[` restituisce una nuova lista più piccola. Confrontate il codice e l'output di cui sopra con la rappresentazione visiva nella figura \@ref(fig:lists-subsetting).

<div class="figure" style="text-align: center">
<img src="diagrams/lists-subsetting.png" alt="Subsetting a list, visually." width="75%" />
<p class="caption">(\#fig:lists-subsetting)Subsetting a list, visually.</p>
</div>

### Liste di condimenti

La differenza tra `[` e `[[` è molto importante, ma è facile confondersi. Per aiutarti a ricordare, lascia che ti mostri un insolito saliera per il pepe.

<img src="images/pepper.jpg" width="25%" style="display: block; margin: auto;" />

Se questa saliera di pepe è la vostra lista `x`, allora, `x[1]` è una saliera di pepe contenente un singolo pacchetto di pepe:

<img src="images/pepper-1.jpg" width="25%" style="display: block; margin: auto;" />

`x[2]` avrebbe lo stesso aspetto, ma conterrebbe il secondo pacchetto. `x[1:2]` sarebbe una saliera per il pepe contenente due pacchetti di pepe.

`x[[1]]` è:

<img src="images/pepper-2.jpg" width="25%" style="display: block; margin: auto;" />

Se voleste ottenere il contenuto del pacchetto pepper, avreste bisogno di `x[[1]][[1]]`:

<img src="images/pepper-3.jpg" width="25%" style="display: block; margin: auto;" />

### Esercizi

1.  Disegna le seguenti liste come insiemi annidati:

    1.  `lista(a, b, lista(c, d), lista(e, f))`
    1.  `lista(lista(lista(lista(lista(lista(a))))))`)

1.  Cosa succede se si sottoinveste un tibble come se si stesse sottoponendo una lista?
    Quali sono le differenze chiave tra una lista e una tibla?

## Attributi

Ogni vettore può contenere metadati aggiuntivi arbitrari attraverso i suoi __attributes__. Puoi pensare agli attributi come a una lista di vettori denominata che può essere allegata a qualsiasi oggetto. 
Potete ottenere e impostare i valori dei singoli attributi con `attr()` o vederli tutti insieme con `attributes()`.


```r
x <- 1:10
attr(x, "greeting")
#> NULL
attr(x, "greeting") <- "Hi!"
attr(x, "farewell") <- "Bye!"
attributes(x)
#> $greeting
#> [1] "Hi!"
#> 
#> $farewell
#> [1] "Bye!"
```

Ci sono tre attributi molto importanti che vengono utilizzati per implementare parti fondamentali di R:

1. __Names__ sono usati per nominare gli elementi di un vettore. 1. 2. __Dimensions__ (dims, in breve) fanno sì che un vettore si comporti come una matrice o un array. 1. __Class__ è usato per implementare il sistema orientato agli oggetti S3.

Avete visto i nomi sopra, e non parleremo di dimensioni perché non usiamo matrici in questo libro. Resta da descrivere la classe, che controlla il funzionamento delle __funzioni generiche__. Le funzioni generiche sono la chiave per la programmazione orientata agli oggetti in R, perché fanno sì che le funzioni si comportino diversamente per diverse classi di input. Una discussione dettagliata della programmazione orientata agli oggetti va oltre lo scopo di questo libro, ma potete leggerne di più in _Advanced R_ a <http://adv-r.had.co.nz/OO-essentials.html#s3>.

Ecco come appare una tipica funzione generica:


```r
as.Date
#> function (x, ...) 
#> UseMethod("as.Date")
#> <bytecode: 0x55a5b039ed00>
#> <environment: namespace:base>
```

La chiamata a "UseMethod" significa che questa è una funzione generica, e chiamerà uno specifico __method__, una funzione, basata sulla classe del primo argomento. (Tutti i metodi sono funzioni; non tutte le funzioni sono metodi). Potete elencare tutti i metodi di un generico con `metodi()`:


```r
methods("as.Date")
#> [1] as.Date.character   as.Date.default     as.Date.factor     
#> [4] as.Date.numeric     as.Date.POSIXct     as.Date.POSIXlt    
#> [7] as.Date.vctrs_sclr* as.Date.vctrs_vctr*
#> see '?methods' for accessing help and source code
```

Per esempio, se `x` è un vettore di caratteri, `as.Date()` chiamerà `as.Date.character()`; se è un fattore, chiamerà `as.Date.factor()`.

Potete vedere l'implementazione specifica di un metodo con `getS3method()`:


```r
getS3method("as.Date", "default")
#> function (x, ...) 
#> {
#>     if (inherits(x, "Date")) 
#>         x
#>     else if (is.null(x)) 
#>         .Date(numeric())
#>     else if (is.logical(x) && all(is.na(x))) 
#>         .Date(as.numeric(x))
#>     else stop(gettextf("do not know how to convert '%s' to class %s", 
#>         deparse1(substitute(x)), dQuote("Date")), domain = NA)
#> }
#> <bytecode: 0x55a5adecfed0>
#> <environment: namespace:base>
getS3method("as.Date", "numeric")
#> function (x, origin, ...) 
#> {
#>     if (missing(origin)) {
#>         if (!length(x)) 
#>             return(.Date(numeric()))
#>         if (!any(is.finite(x))) 
#>             return(.Date(x))
#>         stop("'origin' must be supplied")
#>     }
#>     as.Date(origin, ...) + x
#> }
#> <bytecode: 0x55a5aded6140>
#> <environment: namespace:base>
```

Il più importante generico S3 è `print()`: controlla come l'oggetto viene stampato quando si digita il suo nome sulla console. Altri generici importanti sono le funzioni di subsetting `[`, `[[`, e `$`.

## Vettori incrementati

I vettori atomici e le liste sono i mattoni per altri importanti tipi di vettori come i fattori e le date. Li chiamo __vettori aumentati__, perché sono vettori con ulteriori __attributi__, inclusa la classe. Poiché i vettori aumentati hanno una classe, si comportano diversamente dal vettore atomico su cui sono costruiti. In questo libro, facciamo uso di quattro importanti vettori aumentati:

* Fattori 
* Date 
* Data-ora 
* Tibble

Questi sono descritti di seguito.

### Fattori

I fattori sono progettati per rappresentare dati categorici che possono assumere un insieme fisso di possibili valori. I fattori sono costruiti sopra gli interi e hanno un attributo levels:


```r
x <- factor(c("ab", "cd", "ab"), levels = c("ab", "cd", "ef"))
typeof(x)
#> [1] "integer"
attributes(x)
#> $levels
#> [1] "ab" "cd" "ef"
#> 
#> $class
#> [1] "factor"
```

### Date e data-ora

Le date in R sono vettori numerici che rappresentano il numero di giorni dal 1° gennaio 1970.


```r
x <- as.Date("1971-01-01")
unclass(x)
#> [1] 365

typeof(x)
#> [1] "double"
attributes(x)
#> $class
#> [1] "Date"
```

I data-ora sono vettori numerici con classe `POSIXct` che rappresentano il numero di secondi dal 1 gennaio 1970. (Nel caso ve lo steste chiedendo, "POSIXct" sta per "Portable Operating System Interface", ora del calendario).

```r
x <- lubridate::ymd_hm("1970-01-01 01:00")
unclass(x)
#> [1] 3600
#> attr(,"tzone")
#> [1] "UTC"

typeof(x)
#> [1] "double"
attributes(x)
#> $class
#> [1] "POSIXct" "POSIXt" 
#> 
#> $tzone
#> [1] "UTC"
```

L'attributo `tzone` è opzionale. Controlla come viene stampata l'ora, non a quale ora assoluta si riferisce.


```r
attr(x, "tzone") <- "US/Pacific"
x
#> [1] "1969-12-31 17:00:00 PST"

attr(x, "tzone") <- "US/Eastern"
x
#> [1] "1969-12-31 20:00:00 EST"
```

C'è un altro tipo di date-times chiamato POSIXlt. Questi sono costruiti sopra le liste con nome:


```r
y <- as.POSIXlt(x)
typeof(y)
#> [1] "list"
attributes(y)
#> $names
#>  [1] "sec"    "min"    "hour"   "mday"   "mon"    "year"   "wday"   "yday"  
#>  [9] "isdst"  "zone"   "gmtoff"
#> 
#> $class
#> [1] "POSIXlt" "POSIXt" 
#> 
#> $tzone
#> [1] "US/Eastern" "EST"        "EDT"
```

I POSIXlts sono rari all'interno del tidyverse. Spuntano fuori in R di base, perché sono necessari per estrarre componenti specifici di una data, come l'anno o il mese. Dato che lubridate fornisce degli helper per fare questo, non ne avete bisogno. I POSIXct sono sempre più facili da lavorare, quindi se scoprite di avere un POSIXlt, dovreste sempre convertirlo in un normale data time `lubridate::as_date_time()`.

### Tibble

Le tibble sono liste aumentate: hanno classe "tbl_df" + "tbl" + "data.frame", e gli attributi `names` (colonna) e `row.names`:


```r
tb <- tibble::tibble(x = 1:5, y = 5:1)
typeof(tb)
#> [1] "list"
attributes(tb)
#> $class
#> [1] "tbl_df"     "tbl"        "data.frame"
#> 
#> $row.names
#> [1] 1 2 3 4 5
#> 
#> $names
#> [1] "x" "y"
```

La differenza tra una tibble e una lista è che tutti gli elementi di un data frame devono essere vettori della stessa lunghezza. Tutte le funzioni che lavorano con le tibble impongono questo vincolo.

I data.frame tradizionali hanno una struttura molto simile:


```r
df <- data.frame(x = 1:5, y = 5:1)
typeof(df)
#> [1] "list"
attributes(df)
#> $names
#> [1] "x" "y"
#> 
#> $class
#> [1] "data.frame"
#> 
#> $row.names
#> [1] 1 2 3 4 5
```

La differenza principale è la classe. La classe di tibble include "data.frame", il che significa che le tibble ereditano il comportamento dei normali data frame per default.

### Esercizi

1.  Cosa restituisce `hms::hms(3600)`? Come si stampa? Quale primitiva
    è costruito sopra il vettore aumentato? Quali attributi usa 
    usa?
    
1.  Prova a fare un tibbo che abbia colonne di lunghezza diversa. Cosa
    succede?

1.  In base alla definizione di cui sopra, va bene avere una lista come
    colonna di una tibla?
