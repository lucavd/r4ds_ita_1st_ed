# Funzioni

## Introduzione 

Uno dei modi migliori per migliorare il proprio raggio d'azione come scienziato dei dati è scrivere funzioni. Le funzioni permettono di automatizzare compiti comuni in un modo più potente e generale del copia-e-incolla. Scrivere una funzione ha tre grandi vantaggi rispetto al copia-e-incolla:

1.  Potete dare ad una funzione un nome evocativo che rende il vostro codice più facile da 
    capire.

1.  Quando i requisiti cambiano, hai solo bisogno di aggiornare il codice in un posto, invece di
    invece che in molti.

1.  Si elimina la possibilità di fare errori accidentali quando si copia e 
    incollare (cioè aggiornare un nome di variabile in un posto, ma non in un altro).

Scrivere buone funzioni è un viaggio lungo una vita. Anche dopo aver usato R per molti anni, imparo ancora nuove tecniche e modi migliori di affrontare vecchi problemi. L'obiettivo di questo capitolo non è quello di insegnarvi ogni dettaglio esoterico delle funzioni, ma di farvi iniziare con alcuni consigli pragmatici che potete applicare immediatamente.

Oltre a consigli pratici per scrivere funzioni, questo capitolo vi dà anche alcuni suggerimenti su come stilizzare il vostro codice. Un buono stile del codice è come una corretta punteggiatura. Si può gestire anche senza, ma di sicuro rende le cose più facili da leggere! Come per gli stili di punteggiatura, ci sono molte possibili variazioni. Qui presentiamo lo stile che usiamo nel nostro codice, ma la cosa più importante è essere coerenti.

### Prerequisiti

L'obiettivo di questo capitolo è scrivere funzioni in R base, quindi non avrete bisogno di altri pacchetti.

## Quando dovreste scrivere una funzione?

Dovreste considerare di scrivere una funzione ogni volta che avete copiato e incollato un blocco di codice più di due volte (cioè ora avete tre copie dello stesso codice). Per esempio, date un'occhiata a questo codice. Cosa fa?


```r
df <- tibble::tibble(
  a = rnorm(10),
  b = rnorm(10),
  c = rnorm(10),
  d = rnorm(10)
)

df$a <- (df$a - min(df$a, na.rm = TRUE)) / 
  (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$b <- (df$b - min(df$b, na.rm = TRUE)) / 
  (max(df$b, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$c <- (df$c - min(df$c, na.rm = TRUE)) / 
  (max(df$c, na.rm = TRUE) - min(df$c, na.rm = TRUE))
df$d <- (df$d - min(df$d, na.rm = TRUE)) / 
  (max(df$d, na.rm = TRUE) - min(df$d, na.rm = TRUE))
```

Potreste essere in grado di capire che questo ridimensiona ogni colonna per avere un intervallo da 0 a 1. Ma avete notato l'errore? Ho fatto un errore quando ho copiato e incollato il codice per `df$b`: Ho dimenticato di cambiare una `a` con una `b`. Estrarre il codice ripetuto in una funzione è una buona idea perché ti impedisce di fare questo tipo di errore.

Per scrivere una funzione devi prima analizzare il codice. Quanti input ha?


```r
(df$a - min(df$a, na.rm = TRUE)) /
  (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
```

Questo codice ha solo un input: `df$a`. (Se siete sorpresi che `TRUE` non sia un input, potete esplorare il perché nell'esercizio seguente). Per rendere gli input più chiari, è una buona idea riscrivere il codice usando variabili temporanee con nomi generici. Qui questo codice richiede solo un singolo vettore numerico, quindi lo chiamerò `x`:


```r
x <- df$a
(x - min(x, na.rm = TRUE)) / (max(x, na.rm = TRUE) - min(x, na.rm = TRUE))
#>  [1] 0.2892677 0.7509271 0.0000000 0.6781686 0.8530656 1.0000000 0.1716402
#>  [8] 0.6107464 0.6116181 0.6008793
```

C'è qualche duplicazione in questo codice. Stiamo calcolando l'intervallo dei dati tre volte, quindi ha senso farlo in un solo passo:

```r
rng <- range(x, na.rm = TRUE)
(x - rng[1]) / (rng[2] - rng[1])
#>  [1] 0.2892677 0.7509271 0.0000000 0.6781686 0.8530656 1.0000000 0.1716402
#>  [8] 0.6107464 0.6116181 0.6008793
```

Tirare fuori i calcoli intermedi in variabili con nome è una buona pratica perché rende più chiaro cosa sta facendo il codice. Ora che ho semplificato il codice e controllato che funzioni ancora, posso trasformarlo in una funzione:


```r
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE)
  (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(c(0, 5, 10))
#> [1] 0.0 0.5 1.0
```

Ci sono tre passi chiave per creare una nuova funzione:

1.  Dovete scegliere un __nome__ per la funzione. Qui ho usato `rescale01`. 
    perché questa funzione ridimensiona un vettore per farlo stare tra 0 e 1.

1.  Si elencano gli input, o __argomenti__, alla funzione all'interno di `function`.
    Qui abbiamo solo un argomento. Se ne avessimo di più, la chiamata sarebbe come
    `funzione(x, y, z)`.

1.  Mettete il codice che avete sviluppato in __corpo__ della funzione, un blocco 
    `{` blocco che segue immediatamente la `funzione(...)`.

Notate il processo generale: Ho creato la funzione solo dopo aver capito come farla funzionare con un semplice input. È più facile iniziare con del codice funzionante e trasformarlo in una funzione; è più difficile creare una funzione e poi cercare di farla funzionare.

A questo punto è una buona idea controllare la vostra funzione con alcuni input diversi:


```r
rescale01(c(-10, 0, 10))
#> [1] 0.0 0.5 1.0
rescale01(c(1, 2, 3, NA, 5))
#> [1] 0.00 0.25 0.50   NA 1.00
```

Man mano che scrivete sempre più funzioni, alla fine vorrete convertire questi test informali e interattivi in test formali e automatizzati. Questo processo è chiamato test delle unità. Sfortunatamente, va oltre lo scopo di questo libro, ma potete impararlo in <http://r-pkgs.had.co.nz/tests.html>.

Possiamo semplificare l'esempio originale ora che abbiamo una funzione:


```r
df$a <- rescale01(df$a)
df$b <- rescale01(df$b)
df$c <- rescale01(df$c)
df$d <- rescale01(df$d)
```

Rispetto all'originale, questo codice è più facile da capire e abbiamo eliminato una classe di errori di copia e incolla. C'è ancora un bel po' di duplicazione poiché stiamo facendo la stessa cosa a più colonne. Impareremo come eliminare questa duplicazione in [iterazione], una volta che avrete imparato di più sulle strutture dati di R in [vettori].

Un altro vantaggio delle funzioni è che se i nostri requisiti cambiano, dobbiamo fare il cambiamento solo in un posto. Per esempio, potremmo scoprire che alcune delle nostre variabili includono valori infiniti, e `rescale01()` fallisce:


```r
x <- c(1:10, Inf)
rescale01(x)
#>  [1]   0   0   0   0   0   0   0   0   0   0 NaN
```

Poiché abbiamo estratto il codice in una funzione, abbiamo solo bisogno di fare la correzione in un posto:


```r
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE, finite = TRUE)
  (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(x)
#>  [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556 0.6666667
#>  [8] 0.7777778 0.8888889 1.0000000       Inf
```

Questa è una parte importante del principio "non ripetersi" (o DRY, 'Do not Repeat Yourself'). Più ripetizioni avete nel vostro codice, più posti dovete ricordarvi di aggiornare quando le cose cambiano (e cambiano sempre!), e più è probabile che creiate bug nel tempo.

### Esercizi

1.  Perché `TRUE` non è un parametro di `rescale01()`? 2. Cosa accadrebbe se `x` contenesse un singolo valore mancante e `na.rm` fosse `FALSE`?

1.  Nella seconda variante di `rescale01()`, i valori infiniti sono lasciati invariati. Riscrivi `rescale01()` in modo che `-Inf` sia mappato a 0, e `Inf` sia mappato a 1.

1.  Esercitatevi a trasformare i seguenti frammenti di codice in funzioni. Pensate a cosa fa ogni funzione. Come la chiameresti? Di quanti argomenti ha bisogno? Puoi riscriverla per essere più espressiva o meno duplicata?

    
    ```r
    mean(is.na(x))
    
    x / sum(x, na.rm = TRUE)
    
    sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
    ```

1. scrivi le tue funzioni per calcolare la varianza e l'asimmetria di un vettore numerico.
    La varianza è definita come
    $$
    \mathrm{Var}(x) = \frac{1}{n - 1} \sum_{i=1}^n (x_i - \bar{x}) ^2 \testo{,}
    $$
    dove $bar{x} = (\sum_i^n x_i) / n$ è la media del campione.
    L'asimmetria è definita come
    $$
    \mathrm{Skew}(x) = \frac{frac{1}{n-2}\sinistra(\sum_{i=1}^n(x_i - \bar x)^3\destra)}{mathrm{Var}(x)^{3/2} \testo{.}
    $$

1.  Scrivere `both_na()`, una funzione che prende due vettori della stessa lunghezza 
    e restituisce il numero di posizioni che hanno un `NA` in entrambi i vettori.

1.  Cosa fanno le seguenti funzioni? Perché sono utili anche se sono
    sono così corte?
    
    
    ```r
    is_directory <- function(x) file.info(x)$isdir
    is_readable <- function(x) file.access(x, 4) == 0
    ```

1.  Leggi il [testo completo](https://en.wikipedia.org/wiki/Little_Bunny_Foo_Foo) 
    di "Little Bunny Foo Foo". Ci sono molti doppioni in questa canzone. 
    Estendete l'esempio iniziale di piping per ricreare la canzone completa, e usate 
    funzioni per ridurre la duplicazione.

## Le funzioni sono per gli umani e per i computer

È importante ricordare che le funzioni non sono solo per il computer, ma anche per gli esseri umani. A R non interessa come viene chiamata la vostra funzione, o quali commenti contiene, ma questi sono importanti per i lettori umani. Questa sezione discute alcune cose che dovreste tenere a mente quando scrivete funzioni che gli umani possano capire.

Il nome di una funzione è importante. Idealmente, il nome della vostra funzione sarà breve, ma evocherà chiaramente ciò che la funzione fa. Questo è difficile! Ma è meglio essere chiari che brevi, poiché il completamento automatico di RStudio rende facile digitare nomi lunghi.

In generale, i nomi delle funzioni dovrebbero essere verbi, e gli argomenti dovrebbero essere sostantivi. Ci sono alcune eccezioni: i sostantivi vanno bene se la funzione calcola un sostantivo molto noto (ad esempio `mean()` è meglio di `compute_mean()`), o accede a qualche proprietà di un oggetto (ad esempio `coef()` è meglio di `get_coefficients()`). Un buon segno che un sostantivo potrebbe essere una scelta migliore è se state usando un verbo molto ampio come "get", "compute", "calculate", o "determine". Usate il vostro miglior giudizio e non abbiate paura di rinominare una funzione se trovate un nome migliore in seguito.


```r
# Troppo corto
f()

# Non è un verbo, o descrittivo
my_awesome_function()

# Lungo, ma chiaro
impute_missing()
collapse_years()
```

Se il nome della vostra funzione è composto da più parole, vi consiglio di usare "snake\_case", dove ogni parola minuscola è separata da un underscore. camelCase è un'alternativa popolare. Non importa quale scegliete, l'importante è essere coerenti: scegliete uno o l'altro e mantenetelo. R stesso non è molto coerente, ma non puoi farci niente. Assicuratevi di non cadere nella stessa trappola rendendo il vostro codice il più coerente possibile.


```r
# Non farlo mai!
col_mins <- function(x, y) {}
rowMaxes <- function(y, x) {}
```

Se avete una famiglia di funzioni che fanno cose simili, assicuratevi che abbiano nomi e argomenti coerenti. Usate un prefisso comune per indicare che sono collegate. Questo è meglio di un suffisso comune perché il completamento automatico vi permette di digitare il prefisso e vedere tutti i membri della famiglia.


```r
# Buono
input_select()
input_checkbox()
input_text()

# Non così buono
select_input()
checkbox_input()
text_input()
```

Un buon esempio di questo design è il pacchetto stringr: se non ricordate esattamente quale funzione vi serve, potete digitare `str_` e rinfrescarvi la memoria.

Dove possibile, evitate di sovrascrivere funzioni e variabili esistenti. E' impossibile farlo in generale perché tanti buoni nomi sono già presi da altri pacchetti, ma evitare i nomi più comuni da R base eviterà la confusione.


```r
# Non fatelo!
T <- FALSE
c <- 10
mean <- function(x) sum(x)
```

Usate i commenti, linee che iniziano con `#`, per spiegare il "perché" del vostro codice. Generalmente dovreste evitare commenti che spiegano il "cosa" o il "come". Se non riesci a capire cosa fa il codice leggendolo, dovresti pensare a come riscriverlo per essere più chiaro. Avete bisogno di aggiungere alcune variabili intermedie con nomi utili? Avete bisogno di separare un sottocomponente di una grande funzione in modo da poterle dare un nome? Tuttavia, il vostro codice non può mai catturare il ragionamento dietro le vostre decisioni: perché avete scelto questo approccio invece di un'alternativa? Cos'altro avete provato che non ha funzionato? È una grande idea catturare questo tipo di pensiero in un commento.

Un altro uso importante dei commenti è quello di spezzare il tuo file in pezzi facilmente leggibili. Usate lunghe linee di `-` e `=` per rendere facile individuare le interruzioni.


```r
# Load data --------------------------------------

# Plot data --------------------------------------
```

RStudio fornisce una scorciatoia da tastiera per creare queste intestazioni (Cmd/Ctrl + Shift + R), e le mostrerà nel drop-down di navigazione del codice in basso a sinistra dell'editor:

<img src="screenshots/rstudio-nav.png" width="125" style="display: block; margin: auto;" />

### Esercizi

1.  Leggete il codice sorgente di ciascuna delle tre funzioni seguenti, fate un puzzle di ciò che fanno, e poi fate un brainstorming di nomi migliori.
    
    
    ```r
    f1 <- function(string, prefix) {
      substr(string, 1, nchar(prefix)) == prefix
    }
    f2 <- function(x) {
      if (length(x) <= 1) return(NULL)
      x[-length(x)]
    }
    f3 <- function(x, y) {
      rep(y, length.out = length(x))
    }
    ```
    
1.  Prendi una funzione che hai scritto recentemente e spendi 5 minuti 
    per trovare un nome migliore per essa e per i suoi argomenti.

1.  Confrontate e contrastate `rnorm()` e `MASS::mvrnorm()`. Come potreste renderli
    più coerenti? 
    
1.  Spiegate perché `norm_r()`, `norm_d()` ecc. sarebbero meglio di
    `rnorm()`, `dnorm()`. 2. Inventa un caso per il contrario.

## Esecuzione condizionale

Un'istruzione `if` ti permette di eseguire del codice in modo condizionale. Si presenta così:


```r
if (condition) {
  # codice eseguito quando la condizione è VERA
} else {
  # codice eseguito quando la condizione è FALSA
}
```

Per avere un aiuto su `if` dovete circondarlo di backtick: `` ?`if` ``. L'aiuto non è particolarmente utile se non sei già un programmatore esperto, ma almeno sai come arrivarci!

Ecco una semplice funzione che usa una dichiarazione `if`. L'obiettivo di questa funzione è di restituire un vettore logico che descriva se ogni elemento di un vettore è nominato o meno.


```r
has_name <- function(x) {
  nms <- names(x)
  if (is.null(nms)) {
    rep(FALSE, length(x))
  } else {
    !is.na(nms) & nms != ""
  }
}
```

Questa funzione sfrutta la regola di ritorno standard: una funzione restituisce l'ultimo valore che ha calcolato. Qui si tratta di uno dei due rami dell'istruzione `if`.

### Condizioni

La `condizione` deve valutare o `TRUE` o `FALSE`. Se è un vettore, avrai un messaggio di avvertimento; se è un `NA`, avrai un errore. Fai attenzione a questi messaggi nel tuo codice:


```r
if (c(TRUE, FALSE)) {}

if (NA) {}
```

Potete usare `||` (o) e `&&` (e) per combinare più espressioni logiche. Questi operatori sono "a corto circuito": non appena `||` vede il primo `TRUE` restituisce `TRUE` senza calcolare nient'altro. Non appena `&&` vede il primo `FALSE` restituisce `FALSE`. Non dovreste mai usare `|` o `&` in una dichiarazione `if`: queste sono operazioni vettoriali che si applicano a valori multipli (ecco perché le usate in `filter()`). Se avete un vettore logico, potete usare `any()` o `all()` per farlo collassare ad un singolo valore.

Fate attenzione quando testate l'uguaglianza. `==` è vettoriale, il che significa che è facile ottenere più di un output.  Controllate che la lunghezza sia già 1, collassate con `all()` o `any()`, o usate la non vettorializzata `identical()`. `identical()` è molto rigoroso: restituisce sempre o un singolo `TRUE` o un singolo `FALSE`, e non coordina i tipi. Questo significa che dovete fare attenzione quando confrontate interi e doppi:


```r
identical(0L, 0)
#> [1] FALSE
```

Bisogna anche diffidare dei numeri in virgola mobile:


```r
x <- sqrt(2) ^ 2
x
#> [1] 2
x == 2
#> [1] FALSE
x - 2
#> [1] 4.440892e-16
```

Usate invece `dplyr::near()` per i confronti, come descritto in [confronti].

E ricordate, `x == NA` non fa nulla di utile!

### Condizioni multiple

Puoi concatenare più dichiarazioni if insieme:


```r
if (this) {
  # fai quello
} else if (that) {
  # fai quell'altro
} else {
  # 
}
```

Ma se vi ritrovate con una serie molto lunga di dichiarazioni `if` concatenate, dovreste considerare la riscrittura. Una tecnica utile è la funzione `switch()`. Vi permette di valutare il codice selezionato in base alla posizione o al nome.


```
#> function(x, y, op) {
#>   switch(op,
#>     plus = x + y,
#>     minus = x - y,
#>     times = x * y,
#>     divide = x / y,
#>     stop("Unknown op!")
#>   )
#> }
```

Un'altra funzione utile che spesso può eliminare lunghe catene di istruzioni `if` è `cut()`. Viene usata per discretizzare le variabili continue.

### Stile del codice

Sia `if` che `function` dovrebbero (quasi) sempre essere seguite da parentesi graffe (`{}`), e il contenuto dovrebbe essere indentato di due spazi. Questo rende più facile vedere la gerarchia nel tuo codice scorrendo il margine sinistro.

Una parentesi graffa di apertura non dovrebbe mai andare sulla propria linea e dovrebbe essere sempre seguita da una nuova linea. Una parentesi graffa di chiusura dovrebbe sempre andare sulla propria riga, a meno che non sia seguita da `else`. Fai sempre rientrare il codice all'interno delle parentesi graffe.


```r
# Buono
if (y < 0 && debug) {
  message("Y is negative")
}

if (y == 0) {
  log(x)
} else {
  y ^ x
}

# Cattivo
if (y < 0 && debug)
message("Y is negative")

if (y == 0) {
  log(x)
} 
else {
  y ^ x
}
```

Va bene non usare le parentesi graffe se hai una dichiarazione `if` molto breve che può stare su una sola riga:


```r
y <- 10
x <- if (y < 20) "Too low" else "Too high"
```

Lo raccomando solo per dichiarazioni molto brevi "if". Altrimenti, la forma completa è più facile da leggere:


```r
if (y < 20) {
  x <- "Too low" 
} else {
  x <- "Too high"
}
```

### Esercizi

1.  Qual è la differenza tra `if` e `ifelse()`? Leggete attentamente l'aiuto
    e costruisci tre esempi che illustrino le differenze chiave.

1.  Scrivi una funzione di saluto che dica "buongiorno", "buon pomeriggio",
    o "buona sera", a seconda dell'ora del giorno. (Suggerimento: usate un argomento
    che sia predefinito a `lubridate::now()`. Questo renderà 
    più facile testare la vostra funzione).

1.  Implementare una funzione `fizzbuzz`. Prende un singolo numero come input. Se
    il numero è divisibile per tre, restituisce "fizz". Se è divisibile per
    cinque, restituisce "buzz". Se è divisibile per tre e per cinque, restituisce
    "fizzbuzz". Altrimenti, restituisce il numero. Assicuratevi di scrivere prima 
    codice funzionante prima di creare la funzione.
    
1.  Come potreste usare `cut()` per semplificare questo insieme di dichiarazioni if-else annidate?

    
    ```r
    if (temp <= 0) {
      "freezing"
    } else if (temp <= 10) {
      "cold"
    } else if (temp <= 20) {
      "cool"
    } else if (temp <= 30) {
      "warm"
    } else {
      "hot"
    }
    ```
    
    Come cambierebbe la chiamata a `cut()` se avessi usato `<` invece di `<=`?
    Qual è l'altro vantaggio principale di `cut()` per questo problema? (Suggerimento:
    cosa succede se hai molti valori in `temp`?)

1.  Cosa succede se usate `switch()` con valori numerici?

1.  Cosa fa questa chiamata `switch()`? 2. Cosa succede se `x` è "e"?

    
    ```r
    switch(x, 
      a = ,
      b = "ab",
      c = ,
      d = "cd"
    )
    ```
    
    Sperimentate, poi leggete attentamente la documentazione. 

## Argomenti delle funzioni

Gli argomenti di una funzione rientrano tipicamente in due grandi insiemi: un insieme fornisce i __dati__ su cui calcolare, e l'altro fornisce argomenti che controllano i __dettagli__ del calcolo. Per esempio:

* In `log()`, i dati sono `x`, e il dettaglio è la `base` del logaritmo.

* In `mean()`, i dati sono `x`, e i dettagli sono quanti dati tagliare
  dalle estremità (`trim`) e come gestire i valori mancanti (`na.rm`).

* In `t.test()`, i dati sono `x` e `y`, e i dettagli del test sono
  `alternative`, `mu`, `paired`, `var.equal`, e `conf.level`.
  
* In `str_c()` potete fornire qualsiasi numero di stringhe a `...`, e i dettagli
  della concatenazione sono controllati da `sep` e `collapse`.
  
Generalmente, gli argomenti di dati dovrebbero venire per primi. Gli argomenti di dettaglio dovrebbero andare alla fine, e di solito dovrebbero avere valori predefiniti. Si specifica un valore predefinito nello stesso modo in cui si chiama una funzione con un argomento con nome:


```r
# Calcolare l'intervallo di confidenza intorno alla media usando l'approssimazione normale
mean_ci <- function(x, conf = 0.95) {
  se <- sd(x) / sqrt(length(x))
  alpha <- 1 - conf
  mean(x) + se * qnorm(c(alpha / 2, 1 - alpha / 2))
}

x <- runif(100)
mean_ci(x)
#> [1] 0.4976111 0.6099594
mean_ci(x, conf = 0.99)
#> [1] 0.4799599 0.6276105
```

Il valore di default dovrebbe essere quasi sempre il valore più comune. Le poche eccezioni a questa regola hanno a che fare con la sicurezza. Per esempio, ha senso che `na.rm` abbia come valore predefinito `FALSE` perché i valori mancanti sono importanti. Anche se `na.rm = TRUE` è quello che di solito mettete nel vostro codice, è una cattiva idea ignorare silenziosamente i valori mancanti per default.

Quando chiamate una funzione, di solito omettete i nomi degli argomenti dei dati, perché sono usati così comunemente. Se sovrascrivete il valore predefinito di un argomento di dettaglio, dovreste usare il nome completo:


```r
# Good
mean(1:10, na.rm = TRUE)

# Bad
mean(x = 1:10, , FALSE)
mean(, TRUE, x = c(1:10, NA))
```

Potete riferirvi ad un argomento con il suo unico prefisso (ad esempio `mean(x, n = TRUE)`), ma questo è generalmente meglio evitarlo, date le possibilità di confusione.

Notate che quando chiamate una funzione, dovreste mettere uno spazio intorno a `=` nelle chiamate di funzione, e mettere sempre uno spazio dopo una virgola, non prima (proprio come nel normale inglese). Usare gli spazi bianchi rende più facile scremare la funzione per i componenti importanti.


```r
# Buono
average <- mean(feet / 12 + inches, na.rm = TRUE)

# Cattivo
average<-mean(feet/12+inches,na.rm=TRUE)
```

### Scegliere i nomi

Anche i nomi degli argomenti sono importanti. A R non importa, ma ai lettori del vostro codice (compresi i futuri voi!) sì. Generalmente dovreste preferire nomi più lunghi e descrittivi, ma ci sono una manciata di nomi molto comuni e molto brevi. Vale la pena memorizzarli:

* `x`, `y`, `z`: vettori. * `w`: un vettore di pesi. * `df`: un data frame. * `i`, `j`: indici numerici (tipicamente righe e colonne). * `n`: lunghezza, o numero di righe. * `p`: numero di colonne.

Altrimenti, considerate la corrispondenza dei nomi degli argomenti nelle funzioni R esistenti. Per esempio, usate `na.rm` per determinare se i valori mancanti devono essere rimossi.

### Controllo dei valori

Quando inizierete a scrivere più funzioni, alla fine arriverete al punto in cui non ricorderete esattamente come funziona la vostra funzione. A questo punto è facile chiamare la vostra funzione con input non validi. Per evitare questo problema, è spesso utile rendere espliciti i vincoli. Per esempio, immaginate di aver scritto alcune funzioni per calcolare statistiche sommarie ponderate:


```r
wt_mean <- function(x, w) {
  sum(x * w) / sum(w)
}
wt_var <- function(x, w) {
  mu <- wt_mean(x, w)
  sum(w * (x - mu) ^ 2) / sum(w)
}
wt_sd <- function(x, w) {
  sqrt(wt_var(x, w))
}
```

Cosa succede se `x` e `w` non hanno la stessa lunghezza?


```r
wt_mean(1:6, 1:3)
#> [1] 7.666667
```

In questo caso, a causa delle regole di riciclaggio dei vettori di R, non otteniamo un errore.

È buona pratica controllare le precondizioni importanti e lanciare un errore (con `stop()`), se non sono vere:


```r
wt_mean <- function(x, w) {
  if (length(x) != length(w)) {
    stop("`x` and `w` must be the same length", call. = FALSE)
  }
  sum(w * x) / sum(w)
}
```

Fate attenzione a non esagerare. C'è un compromesso tra quanto tempo spendete per rendere la vostra funzione robusta e quanto tempo spendete per scriverla. Per esempio, se hai aggiunto anche un argomento `na.rm`, probabilmente non lo controllerei attentamente:


```r
wt_mean <- function(x, w, na.rm = FALSE) {
  if (!is.logical(na.rm)) {
    stop("`na.rm` must be logical")
  }
  if (length(na.rm) != 1) {
    stop("`na.rm` must be length 1")
  }
  if (length(x) != length(w)) {
    stop("`x` and `w` must be the same length", call. = FALSE)
  }
  
  if (na.rm) {
    miss <- is.na(x) | is.na(w)
    x <- x[!miss]
    w <- w[!miss]
  }
  sum(w * x) / sum(w)
}
```

Questo è un sacco di lavoro extra per un piccolo guadagno aggiuntivo. Un utile compromesso è il built-in `stopifnot()`: controlla che ogni argomento sia `TRUE`, e produce un generico messaggio di errore in caso contrario.


```r
wt_mean <- function(x, w, na.rm = FALSE) {
  stopifnot(is.logical(na.rm), length(na.rm) == 1)
  stopifnot(length(x) == length(w))
  
  if (na.rm) {
    miss <- is.na(x) | is.na(w)
    x <- x[!miss]
    w <- w[!miss]
  }
  sum(w * x) / sum(w)
}
wt_mean(1:6, 6:1, na.rm = "foo")
#> Error in wt_mean(1:6, 6:1, na.rm = "foo"): is.logical(na.rm) is not TRUE
```

Si noti che quando si usa `stopifnot()` si afferma ciò che dovrebbe essere vero piuttosto che controllare ciò che potrebbe essere sbagliato.

### Dot-dot-dot (...)

Molte funzioni in R accettano un numero arbitrario di input:


```r
sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
#> [1] 55
stringr::str_c("a", "b", "c", "d", "e", "f")
#> [1] "abcdef"
```

Come funzionano queste funzioni? Si basano su un argomento speciale: `...` (pronunciato dot-dot-dot). Questo argomento speciale cattura qualsiasi numero di argomenti che non sono altrimenti abbinati.

È utile perché è possibile inviare questi `...` ad un'altra funzione. Questo è un utile catch-all se la vostra funzione avvolge principalmente un'altra funzione. Per esempio, di solito creo queste funzioni helper che avvolgono `str_c()`:


```r
commas <- function(...) stringr::str_c(..., collapse = ", ")
commas(letters[1:10])
#> [1] "a, b, c, d, e, f, g, h, i, j"

rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}
rule("Important output")
#> Important output -----------------------------------------------------------
```

Qui `...` mi permette di inoltrare qualsiasi argomento che non voglio trattare a `str_c()`. È una tecnica molto comoda. Ma ha un prezzo: qualsiasi argomento scritto male non genererà un errore. Questo rende facile che gli errori di battitura passino inosservati:


```r
x <- c(1, 2)
sum(x, na.mr = TRUE)
#> [1] 4
```

Se volete solo catturare i valori del `...`, usate `list(...)`.

### Valutazione pigra (lazy evaluation)

Gli argomenti in R sono valutati pigramente: non sono calcolati finché non sono necessari. Questo significa che se non sono mai usati, non sono mai chiamati. Questa è un'importante proprietà di R come linguaggio di programmazione, ma generalmente non è importante quando si scrivono le proprie funzioni per l'analisi dei dati. Potete leggere di più sulla valutazione pigra a <http://adv-r.had.co.nz/Functions.html#lazy-evaluation>.

### Esercizi

1.  Cosa fa `commas(letters, collapse = "-")`? Perché?

1.  Sarebbe bello se tu potessi fornire più caratteri all'argomento `pad`, 
    per esempio `rule("Titolo", pad = "-+")`. Perché attualmente questo non funziona? Come 
    si potrebbe risolvere?
    
1.  Cosa fa l'argomento `trim` di `mean()`? Quando potreste usarlo?

1.  Il valore predefinito per l'argomento `metodo` di `cor()` è 
    `c("pearson", "kendall", "spearman")`. Cosa significa? Quale 
    valore è usato per default?

## Valori di ritorno

Capire cosa dovrebbe restituire la vostra funzione è di solito semplice: è il motivo per cui avete creato la funzione in primo luogo! Ci sono due cose da considerare quando si restituisce un valore: 

1. Restituire in anticipo rende la vostra funzione più facile da leggere? 

2. Potete rendere la vostra funzione 'pipeable' (usabile con una pipe)?

### Dichiarazioni di ritorno esplicite

Il valore restituito dalla funzione è di solito l'ultima affermazione che valuta, ma potete scegliere di tornare in anticipo usando `return()`. Penso che sia meglio salvare l'uso di `return()` per segnalare che si può tornare in anticipo con una soluzione più semplice. Una ragione comune per farlo è perché gli input sono vuoti:


```r
complicated_function <- function(x, y, z) {
  if (length(x) == 0 || length(y) == 0) {
    return(0)
  }
    
  # Codice complicato qui
}
```

Un'altra ragione è che avete una dichiarazione `if` con un blocco complesso e un blocco semplice. Per esempio, potreste scrivere un'istruzione if come questa:


```r
f <- function() {
  if (x) {
    # fare 
    # qualcosa
    # che
    # prende
    # molte
    # linee
    # per 
    # essere
    # espresso
  } else {
    # restituire qualcosa di breve
  }
}
```

Ma se il primo blocco è molto lungo, quando si arriva al `else`, si è dimenticata la `condizione`. Un modo per riscriverlo è usare un ritorno anticipato per il caso semplice:


```r

f <- function() {
  if (!x) {
    return(something_short)
  }

    # fare 
    # qualcosa
    # che
    # prende
    # molte
    # linee
    # per 
    # essere
    # espresso
}
```

Questo tende a rendere il codice più facile da capire, perché non avete bisogno di così tanto contesto per capirlo.

### Scrivere funzioni 'pipeable' (usabili con la pipe)

Se volete scrivere le vostre funzioni pipeable, è importante pensare al valore di ritorno. Conoscere il tipo di oggetto del valore di ritorno significherà che la vostra pipeline "funzionerà". Per esempio, con dplyr e tidyr il tipo di oggetto è il data frame.

Ci sono due tipi base di funzioni pipeable: le trasformazioni e gli effetti collaterali. Con __trasformazioni__, un oggetto viene passato come primo argomento della funzione e viene restituito un oggetto modificato. Con __effetti collaterali__, l'oggetto passato non viene trasformato. Invece, la funzione esegue un'azione sull'oggetto, come disegnare una grafico o salvare un file. Le funzioni effetti collaterali dovrebbero restituire "invisibilmente" il primo argomento, in modo che mentre non vengono stampate possono ancora essere utilizzate in una pipe. Per esempio, questa semplice funzione stampa il numero di valori mancanti in un frame di dati:


```r
show_missings <- function(df) {
  n <- sum(is.na(df))
  cat("Missing values: ", n, "\n", sep = "")
  
  invisible(df)
}
```

Se lo chiamiamo interattivamente, l'opzione `invisible()` significa che l'input `df` non viene stampato:


```r
show_missings(mtcars)
#> Missing values: 0
```

Ma c'è ancora, solo che non è stampato di default:


```r
x <- show_missings(mtcars) 
#> Missing values: 0
class(x)
#> [1] "data.frame"
dim(x)
#> [1] 32 11
```

E possiamo ancora usarlo in una pipe:



```r
mtcars %>% 
  show_missings() %>% 
  mutate(mpg = ifelse(mpg < 20, NA, mpg)) %>% 
  show_missings() 
#> Missing values: 0
#> Missing values: 18
```

## Ambiente

L'ultimo componente di una funzione è il suo ambiente. Questo non è qualcosa che dovete capire a fondo quando iniziate a scrivere funzioni. Tuttavia, è importante conoscere un po' gli ambienti perché sono cruciali per il funzionamento delle funzioni. L'ambiente di una funzione controlla come R trova il valore associato ad un nome. Per esempio, prendete questa funzione:


```r
f <- function(x) {
  x + y
} 
```

In molti linguaggi di programmazione, questo sarebbe un errore, perché `y` non è definito all'interno della funzione. In R, questo è un codice valido perché R usa delle regole chiamate __lexical scoping__ per trovare il valore associato ad un nome. Poiché `y` non è definito all'interno della funzione, R cercherà nell' __ambiente__ dove la funzione è stata definita:


```r
y <- 100
f(10)
#> [1] 110

y <- 1000
f(10)
#> [1] 1010
```

Questo comportamento sembra una ricetta per i bug, e in effetti si dovrebbe evitare di creare deliberatamente funzioni come questa, ma in generale non causa troppi problemi (soprattutto se si riavvia regolarmente R per arrivare a una tabula rasa).

Il vantaggio di questo comportamento è che dal punto di vista del linguaggio permette a R di essere molto coerente. Ogni nome viene cercato usando lo stesso insieme di regole. Per `f()` questo include il comportamento di due cose che potreste non aspettarvi: `{` e `+`. Questo vi permette di fare cose subdole come:


```r
`+` <- function(x, y) {
  if (runif(1) < 0.1) {
    sum(x, y)
  } else {
    sum(x, y) * 1.1
  }
}
table(replicate(1000, 1 + 2))
#> 
#>   3 3.3 
#> 100 900
rm(`+`)
```

Questo è un fenomeno comune in R. R pone pochi limiti al vostro potere. Potete fare molte cose che non potete fare in altri linguaggi di programmazione. Potete fare molte cose che il 99% delle volte sono estremamente sconsigliate (come sovrascrivere il funzionamento dell'addizione!). Ma questa potenza e flessibilità è ciò che rende possibili strumenti come ggplot2 e dplyr. Imparare come usare al meglio questa flessibilità va oltre lo scopo di questo libro, ma si può leggere in [_Advanced R_](http://adv-r.had.co.nz).
