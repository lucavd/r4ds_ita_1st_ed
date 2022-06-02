# Workflow: basi

Ora hai un po' di esperienza nell'esecuzione del codice R. Non vi abbiamo dato molti dettagli, ma ovviamente avete capito le basi, o avreste buttato via questo libro per la frustrazione! La frustrazione è naturale quando si inizia a programmare in R, perché è così pignolo per la punteggiatura, e anche un solo carattere fuori posto causerà una lamentela. Ma mentre dovreste aspettarvi di essere un po' frustrati, consolatevi con il fatto che è sia tipico che temporaneo: succede a tutti, e l'unico modo per superarlo è continuare a provare.

Prima di andare avanti, assicuriamoci che abbiate una solida base nell'esecuzione del codice R, e che conosciate alcune delle caratteristiche più utili di RStudio.

## Basi di codifica

Rivediamo alcune nozioni di base che abbiamo finora omesso nell'interesse di farvi tracciare i grafici il più velocemente possibile. Potete usare R come una calcolatrice:


```r
1 / 200 * 30
#> [1] 0.15
(59 + 73 + 2) / 3
#> [1] 44.66667
sin(pi / 2)
#> [1] 1
```

Puoi creare nuovi oggetti con `<-`:


```r
x <- 3 * 4
```

Tutte le istruzioni di R in cui si creano oggetti, le istruzioni di __assegnazione__, hanno la stessa forma:


```r
object_name <- value
```

Quando leggete quel codice dite "il nome dell'oggetto ottiene il valore" nella vostra testa.

Farete molte assegnazioni e `<-` è un dolore da digitare. Non siate pigri e usate `=`: funzionerà, ma causerà confusione in seguito. Invece, usa la scorciatoia da tastiera di RStudio: Alt + - (il segno meno). Notate che RStudio circonda automaticamente `<-` con spazi, che è una buona pratica di formattazione del codice. Il codice è penoso da leggere in una buona giornata, quindi datevi tregua e usate gli spazi.

## Cosa c'è in un nome?

I nomi degli oggetti devono iniziare con una lettera e possono contenere solo lettere, numeri, `_` e `.`. Vuoi che i nomi dei tuoi oggetti siano descrittivi, quindi avrai bisogno di una convenzione per le parole multiple. Noi raccomandiamo __snake_case__ dove separi le parole minuscole con `_`. 


```r
io_uso_lo_snake_case
altrePersoneUsanoIlCamelCase
alcune.persone.usano.i.punti
E_poche.Persone_RINUNCIANO.Alle_convenzioni
```

Torneremo sullo stile del codice più tardi, in [funzioni].

Si può ispezionare un oggetto digitando il suo nome:


```r
x
#> [1] 12
```

Facciamo un'altra assegnazione:


```r
this_is_a_really_long_name <- 2.5
```

Per ispezionare questo oggetto, provate la funzione di completamento di RStudio: digitate "this", premete TAB, aggiungete caratteri fino ad avere un prefisso unico, poi premete return.

Ooops, hai fatto un errore! `this_is_a_really_long_name` dovrebbe avere valore 3.5 non 2.5. Usa un'altra scorciatoia da tastiera per aiutarti a correggerlo.  Digita "this" e poi premi Cmd/Ctrl + ↑. Questo elencherà tutti i comandi che hai digitato che iniziano con quelle lettere. Usa i tasti freccia per navigare, poi premi enter per ridigitare il comando. Cambia 2.5 in 3.5 e rilancia.

Facciamo un'altra assegnazione:


```r
r_rocks <- 2 ^ 3
```

Proviamo ad ispezionarlo:


```r
r_rock
#> Error: object 'r_rock' not found
R_rocks
#> Error: object 'R_rocks' not found
```

C'è un contratto implicito tra voi e R: esso farà il noioso calcolo per voi, ma in cambio voi dovete essere completamente precisi nelle vostre istruzioni. Gli errori di battitura contano. Le maiuscole contano.

## Chiamare le funzioni

R ha una vasta collezione di funzioni integrate che vengono chiamate in questo modo:


```r
function_name(arg1 = val1, arg2 = val2, ...)
```

Proviamo ad usare `seq()` che fa regolari **seq**uenze di numeri e, mentre ci siamo, impariamo altre utili caratteristiche di RStudio. Digitate `se` e premete TAB. Un popup vi mostra i possibili completamenti. Specificate `seq()` digitando più (una "q") per disambiguare, o usando le frecce ↑/↓ per selezionare. Notate il tooltip galleggiante che appare, ricordandovi gli argomenti e lo scopo della funzione. Se volete più aiuto, premete F1 per ottenere tutti i dettagli nella scheda Aiuto ('Help') nel pannello in basso a destra. 

Premete ancora una volta TAB quando avete selezionato la funzione che volete. RStudio aggiungerà le corrispondenti parentesi di apertura (`(`) e di chiusura (`)`) per voi. Digitate gli argomenti `1, 10` e premete invio.


```r
seq(1, 10)
#>  [1]  1  2  3  4  5  6  7  8  9 10
```

Digitate questo codice e notate che ottenete un'assistenza simile con le virgolette accoppiate:


```r
x <- "hello world"
```

Le virgolette e le parentesi devono sempre essere in coppia. RStudio fa del suo meglio per aiutarvi, ma è ancora possibile sbagliare e finire con una mancata corrispondenza. Se questo accade, R vi mostrerà il carattere di continuazione "+":

```
> x <- "hello
+
```

Il `+` vi dice che R sta aspettando altri input; non pensa che abbiate già finito. Di solito significa che avete dimenticato un `"` o un `)`. O aggiungi la coppia mancante, o premi ESC per interrompere l'espressione e riprovare.

Se fai un'assegnazione, non puoi vedere il valore. Sei quindi tentato di ricontrollare immediatamente il risultato:


```r
y <- seq(1, 10, length.out = 5)
y
#> [1]  1.00  3.25  5.50  7.75 10.00
```

Questa azione comune può essere abbreviata circondando l'assegnazione con delle parentesi, il che provoca l'assegnazione e la "stampa a schermo".


```r
(y <- seq(1, 10, length.out = 5))
#> [1]  1.00  3.25  5.50  7.75 10.00
```

Ora guarda il tuo environment ('ambiente di lavoro') nel riquadro in alto a destra:

<img src="screenshots/rstudio-env.png" width="597" style="display: block; margin: auto;" />

Qui puoi vedere tutti gli oggetti che hai creato.

## Esercizi

1.  Perché questo codice non funziona?[^index-1]

    
    ```r
    my_variable <- 10
    my_varıable
    #> Error in eval(expr, envir, enclos): object 'my_varıable' not found
    ```

[^index-1]: In realtà il codice funziona con il `locale` in italiano, ma non con il `locale` inglese (SPOILER: manca il puntino sulla "i", NdT) 
  
  Guardate attentamente! (Questo può sembrare un esercizio inutile, ma
  allenare il vostro cervello a notare anche la più piccola differenza vi ripagherà
  quando si programma).
    
1.  Modificate ciascuno dei seguenti comandi R in modo che vengano eseguiti correttamente:

    
    ```r
    library(tidyverse)
    
    ggplot(dota = mpg) + 
      geom_point(mapping = aes(x = displ, y = hwy))
    
    fliter(mpg, cyl = 8)
    filter(diamond, carat > 3)
    ```
    
1.  Premi Alt + Shift + K. Cosa succede? Come puoi arrivare allo stesso punto
    usando i menu?

