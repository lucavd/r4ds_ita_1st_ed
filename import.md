# Importazione dei dati

## Introduzione

Lavorare con i dati forniti dai pacchetti di R è un ottimo modo per imparare gli strumenti della data science, ma ad un certo punto si vuole smettere di imparare e iniziare a lavorare con i propri dati. In questo capitolo, imparerete come leggere file rettangolari di testo semplice in R. Qui, gratteremo solo la superficie dell'importazione dei dati, ma molti dei principi saranno trasferiti ad altre forme di dati. Finiremo con alcune indicazioni di pacchetti che sono utili per altri tipi di dati.

### Prerequisiti

In questo capitolo imparerete come caricare file piatti in R con il pacchetto __readr__, che fa parte del core tidyverse.


```r
library(tidyverse)
```

## Iniziamo!

La maggior parte delle funzioni di readr si occupa di trasformare file piatti in data frame:

* `read_csv()` legge file delimitati da virgole, `read_csv2()` legge file separati da punto e virgola
  separati da punto e virgola (comune nei paesi dove `,` è usato come posto decimale come l'Italia),
  `read_tsv()` legge i file delimitati da tabulazione, e `read_delim()` legge i file
  con qualsiasi delimitatore.

* `read_fwf()` legge i file a larghezza fissa. Potete specificare i campi sia per la loro
  larghezza con `fwf_widths()` o la loro posizione con `fwf_positions()`.
  `read_table()` legge una variazione comune dei file a larghezza fissa in cui le colonne
  sono separate da spazi bianchi.

* `read_log()` legge i file di log in stile Apache. (Ma controllate anche
  [webreadr](https://github.com/Ironholds/webreadr) che è costruito sopra
  `read_log()` e fornisce molti altri strumenti utili).

Queste funzioni hanno tutte una sintassi simile: una volta che ne hai imparato una, puoi usare le altre con facilità. Per il resto di questo capitolo ci concentreremo su `read_csv()`. Non solo i file csv sono una delle forme più comuni di memorizzazione dei dati, ma una volta che hai capito `read_csv()`, puoi facilmente applicare la tua conoscenza a tutte le altre funzioni di readr.

Il primo argomento di `read_csv()` è il più importante: è il percorso del file da leggere.


```r
heights <- read_csv("data/heights.csv")
#> Rows: 1192 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (2): sex, race
#> dbl (4): earn, height, ed, age
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

Quando si esegue `read_csv()` viene stampata una specifica di colonna che fornisce il nome e il tipo di ogni colonna. Questa è una parte importante di readr, su cui torneremo in [analizzare un file].

Puoi anche fornire un file csv in linea. Questo è utile per sperimentare con readr e per creare esempi riproducibili da condividere con altri:


```r
read_csv("a,b,c
1,2,3
4,5,6")
#> Rows: 2 Columns: 3
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> dbl (3): a, b, c
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 2 × 3
#>       a     b     c
#>   <dbl> <dbl> <dbl>
#> 1     1     2     3
#> 2     4     5     6
```

In entrambi i casi `read_csv()` usa la prima riga dei dati per i nomi delle colonne, che è una convenzione molto comune. Ci sono due casi in cui si potrebbe voler modificare questo comportamento:

1.  A volte ci sono alcune righe di metadati all'inizio del file. Si può
    usare `skip = n` per saltare le prime `n` linee; o usare `comment = "#"` per eliminare
    tutte le linee che iniziano con (per esempio) `#`.
    
    
    ```r
    read_csv("The first line of metadata
      The second line of metadata
      x,y,z
      1,2,3", skip = 2)
    #> Rows: 1 Columns: 3
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> dbl (3): x, y, z
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    #> # A tibble: 1 × 3
    #>       x     y     z
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    
    read_csv("# A comment I want to skip
      x,y,z
      1,2,3", comment = "#")
    #> Rows: 1 Columns: 3
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> dbl (3): x, y, z
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    #> # A tibble: 1 × 3
    #>       x     y     z
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    ```
    
1.  I dati potrebbero non avere nomi di colonna. Puoi usare `col_names = FALSE` per dire a `read_csv()` di non trattare la prima riga come intestazioni, e invece etichettarle sequenzialmente da `X1` a `Xn`:
    
    
    ```r
    read_csv("1,2,3\n4,5,6", col_names = FALSE)
    #> Rows: 2 Columns: 3
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> dbl (3): X1, X2, X3
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    #> # A tibble: 2 × 3
    #>      X1    X2    X3
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    #> 2     4     5     6
    ```
    
    (`"\n"` è una comoda scorciatoia per aggiungere una nuova riga. Imparerai di più su di essa e su altri tipi di escape delle stringhe in [nozioni di base sulle stringhe]).
    
    In alternativa puoi passare `col_names` un vettore di caratteri che sarà usato come nome delle colonne:
    
    
    ```r
    read_csv("1,2,3\n4,5,6", col_names = c("x", "y", "z"))
    #> Rows: 2 Columns: 3
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> dbl (3): x, y, z
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    #> # A tibble: 2 × 3
    #>       x     y     z
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    #> 2     4     5     6
    ```

Un'altra opzione che comunemente ha bisogno di modifiche è `na`: questa specifica il valore (o i valori) che sono usati per rappresentare i valori mancanti nel tuo file:


```r
read_csv("a,b,c\n1,2,.", na = ".")
#> Rows: 1 Columns: 3
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> dbl (2): a, b
#> lgl (1): c
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 1 × 3
#>       a     b c    
#>   <dbl> <dbl> <lgl>
#> 1     1     2 NA
```

Questo è tutto ciò che dovete sapere per leggere ~75% dei file CSV che incontrerete nella pratica. Puoi anche adattare facilmente ciò che hai imparato per leggere file separati da tabulazione con `read_tsv()` e file a larghezza fissa con `read_fwf()`. Per leggere file più impegnativi, avrete bisogno di imparare di più su come readr analizza ogni colonna, trasformandola in vettori R.

### Rispetto a R di base

Se avete usato R in precedenza, potreste chiedervi perché non stiamo usando `read.csv()`. Ci sono alcune buone ragioni per favorire le funzioni readr rispetto alle equivalenti di base:

* Sono tipicamente molto più veloci (~10x) dei loro equivalenti di base.
  I lavori lunghi hanno una barra di avanzamento, così puoi vedere cosa sta succedendo. 
  Se state cercando la velocità pura, provate `data.table::fread()`. Non si adatta 
  così bene al tidyverse, ma può essere un po' più veloce.

* Producono tibble, non convertono i vettori di caratteri in fattori,
  non usano i nomi delle righe, o non pasticciano i nomi delle colonne. Queste sono fonti comuni di
  frustrazione con le funzioni R di base.

* Sono più riproducibili. Le funzioni R di base ereditano alcuni comportamenti dal
  dal vostro sistema operativo e dalle variabili d'ambiente, quindi il codice d'importazione che funziona 
  sul vostro computer potrebbe non funzionare su quello di qualcun altro.

### Esercizi

1.  Quale funzione usereste per leggere un file in cui i campi sono separati con  
    "|"?
    
1.  Oltre a `file`, `skip` e `comment`, quali altri argomenti hanno
    `read_csv()` e `read_tsv()` hanno in comune?
    
1.  Quali sono gli argomenti più importanti di `read_fwf()`?
   
1.  A volte le stringhe in un file CSV contengono virgole. Per evitare che
    causare problemi, esse devono essere circondate da un carattere di citazione, come
    `"` o `'`. Per default, `read_csv()` assume che il carattere di citazione
    sarà `"`. Quale argomento a `read_csv()` devi specificare
    per leggere il seguente testo in un frame di dati?


    
    
    ```r
    "x,y\n1,'a,b'"
    ```
    
1.  Identifica cosa c'è di sbagliato in ciascuno dei seguenti file CSV in linea. 
    Cosa succede quando si esegue il codice?
    
    
    ```r
    read_csv("a,b\n1,2,3\n4,5,6")
    read_csv("a,b,c\n1,2\n1,2,3,4")
    read_csv("a,b\n\"1")
    read_csv("a,b\n1,2\na,b")
    read_csv("a;b\n1;3")
    ```

## Analizzare un vettore

Prima di entrare nei dettagli di come readr legge i file dal disco, dobbiamo fare una piccola deviazione per parlare delle funzioni `parse_*()`. Queste funzioni prendono un vettore di caratteri e restituiscono un vettore più specializzato come un logico, un intero o una data:


```r
str(parse_logical(c("TRUE", "FALSE", "NA")))
#>  logi [1:3] TRUE FALSE NA
str(parse_integer(c("1", "2", "3")))
#>  int [1:3] 1 2 3
str(parse_date(c("2010-01-01", "1979-10-14")))
#>  Date[1:2], format: "2010-01-01" "1979-10-14"
```

Queste funzioni sono utili di per sé, ma sono anche un importante elemento costitutivo di readr. Una volta che avrete imparato come funzionano i singoli analizzatori in questa sezione, torneremo indietro e vedremo come si adattano insieme per analizzare un file completo nella prossima sezione.

Come tutte le funzioni del tidyverse, le funzioni `parse_*()` sono uniformi: il primo argomento è un vettore di caratteri da analizzare, e l'argomento `na` specifica quali stringhe dovrebbero essere trattate come mancanti:


```r
parse_integer(c("1", "231", ".", "456"), na = ".")
#> [1]   1 231  NA 456
```

Se il parsing fallisce, riceverai un avviso:


```r
x <- parse_integer(c("123", "345", "abc", "123.45"))
#> Warning: 2 parsing failures.
#> row col               expected actual
#>   3  -- no trailing characters abc   
#>   4  -- no trailing characters 123.45
```

E i fallimenti mancheranno nell'output:


```r
x
#> [1] 123 345  NA  NA
#> attr(,"problems")
#> # A tibble: 2 × 4
#>     row   col expected               actual
#>   <int> <int> <chr>                  <chr> 
#> 1     3    NA no trailing characters abc   
#> 2     4    NA no trailing characters 123.45
```

Se ci sono molti fallimenti nell'analisi, avrete bisogno di usare `problems()` per ottenere l'insieme completo. Questo restituisce una tibble, che potete poi manipolare con dplyr.


```r
problems(x)
#> # A tibble: 2 × 4
#>     row   col expected               actual
#>   <int> <int> <chr>                  <chr> 
#> 1     3    NA no trailing characters abc   
#> 2     4    NA no trailing characters 123.45
```

Usare i parser è soprattutto una questione di comprensione di ciò che è disponibile e di come trattano i diversi tipi di input. Ci sono otto analizzatori particolarmente importanti:

1.  `parse_logical()` e `parse_integer()` analizzano i logici e gli interi,
    rispettivamente. Non c'è praticamente nulla che possa andare storto con questi
    quindi non li descriverò ulteriormente qui.
    
1.  `parse_double()`è un parser numerico rigoroso, e `parse_number()` 
    è un analizzatore numerico flessibile. Questi sono più complicati di quanto ci si possa
    aspettare perché diverse parti del mondo scrivono i numeri in modi
    modi diversi.
    
1.  `parse_character()` sembra così semplice che non dovrebbe essere necessario. Ma
    una complicazione lo rende abbastanza importante: le codifiche dei caratteri.

1.  `parse_factor()` crea fattori, la struttura dati che R usa per rappresentare
    variabili categoriche con valori fissi e noti.

1.  `parse_datetime()`, `parse_date()`, e `parse_time()` vi permettono di
    analizzare varie specifiche di data e ora. Queste sono le più complicate
    perché ci sono molti modi diversi di scrivere le date.

Le sezioni seguenti descrivono questi analizzatori in modo più dettagliato.

### Numeri

Sembra che dovrebbe essere semplice analizzare un numero, ma tre problemi lo rendono difficile:

1. La gente scrive i numeri in modo diverso nelle diverse parti del mondo.
   Per esempio, alcuni paesi usano `.` tra la parte intera e quella frazionaria 
   di un numero reale, mentre altri usano `,`.
   
1. I numeri sono spesso circondati da altri caratteri che forniscono qualche
   contesto, come "$1000" o "10%".

1. I numeri spesso contengono caratteri di "raggruppamento" per renderli più facili da leggere, 
   come "1.000.000", e questi caratteri di raggruppamento variano nel mondo.

Per affrontare il primo problema, readr ha la nozione di "locale", un oggetto che specifica le opzioni di analisi che differiscono da luogo a luogo. Quando si analizzano i numeri, l'opzione più importante è il carattere che si usa per il segno decimale. Puoi sovrascrivere il valore predefinito di `.` creando un nuovo locale e impostando l'argomento `decimal_mark`:


```r
parse_double("1.23")
#> [1] 1.23
parse_double("1,23", locale = locale(decimal_mark = ","))
#> [1] 1.23
```

Il locale predefinito di readr è US-centrico, perché generalmente R è US-centrico (cioè la documentazione di base di R è scritta in inglese americano). Un approccio alternativo sarebbe cercare di indovinare i valori predefiniti dal vostro sistema operativo. Questo è difficile da fare bene e, cosa più importante, rende il vostro codice fragile: anche se funziona sul vostro computer, potrebbe fallire quando lo inviate per email ad un collega in un altro paese.

`parse_number()` affronta il secondo problema: ignora i caratteri non numerici prima e dopo il numero. Questo è particolarmente utile per le valute e le percentuali, ma funziona anche per estrarre numeri incorporati nel testo.


```r
parse_number("$100")
#> [1] 100
parse_number("20%")
#> [1] 20
parse_number("It cost $123.45")
#> [1] 123.45
```

L'ultimo problema è affrontato dalla combinazione di `parse_number()` e il locale, poiché `parse_number()` ignorerà il "grouping mark" (segno di raggruppamento):


```r
# Used in America
parse_number("$123,456,789")
#> [1] 123456789

# Used in many parts of Europe
parse_number("123.456.789", locale = locale(grouping_mark = "."))
#> [1] 123456789

# Used in Switzerland
parse_number("123'456'789", locale = locale(grouping_mark = "'"))
#> [1] 123456789
```

### Stringhe {#readr-strings}

Sembra che `parse_character()` dovrebbe essere molto semplice --- potrebbe semplicemente restituire il suo input. Sfortunatamente la vita non è così semplice, poiché ci sono più modi di rappresentare la stessa stringa. Per capire cosa sta succedendo, dobbiamo immergerci nei dettagli di come i computer rappresentano le stringhe. In R, possiamo arrivare alla rappresentazione sottostante di una stringa usando `charToRaw()`:


```r
charToRaw("Hadley")
#> [1] 48 61 64 6c 65 79
```

Ogni numero esadecimale rappresenta un byte di informazione: `48` è H, `61` è a, e così via. La mappatura dal numero esadecimale al carattere è chiamata codifica, e in questo caso la codifica è chiamata ASCII. ASCII fa un ottimo lavoro nel rappresentare i caratteri inglesi, perché è il __American__ Standard Code for Information Interchange.

Le cose diventano più complicate per le lingue diverse dall'inglese. Nei primi giorni dell'informatica c'erano molti standard concorrenti per la codifica dei caratteri non inglesi, e per interpretare correttamente una stringa bisognava conoscere sia i valori che la codifica. Per esempio, due codifiche comuni sono Latin1 (alias ISO-8859-1, usato per le lingue dell'Europa occidentale) e Latin2 (alias ISO-8859-2, usato per le lingue dell'Europa orientale). In Latin1, il byte `b1` è "±", ma in Latin2, è "ą"! Fortunatamente, oggi c'è uno standard che è supportato quasi ovunque: UTF-8. UTF-8 può codificare quasi tutti i caratteri usati dagli esseri umani oggi, così come molti simboli extra (come le emoji!).

readr usa UTF-8 ovunque: assume che i tuoi dati siano codificati in UTF-8 quando li leggi, e li usa sempre quando scrivi. Questo è un buon default, ma fallirà per i dati prodotti da vecchi sistemi che non capiscono UTF-8. Se questo succede a voi, le vostre stringhe avranno un aspetto strano quando le stampate. A volte solo uno o due caratteri potrebbero essere incasinati; altre volte si otterrà una completa incomprensione. Per esempio:


```r
x1 <- "El Ni\xf1o was particularly bad this year"
x2 <- "\x82\xb1\x82\xf1\x82\xc9\x82\xbf\x82\xcd"

x1
#> [1] "El Ni\xf1o was particularly bad this year"
x2
#> [1] "\x82\xb1\x82\xf1\x82ɂ\xbf\x82\xcd"
```

Per risolvere il problema è necessario specificare la codifica in `parse_character()`:


```r
parse_character(x1, locale = locale(encoding = "Latin1"))
#> [1] "El Niño was particularly bad this year"
parse_character(x2, locale = locale(encoding = "Shift-JIS"))
#> [1] "こんにちは"
```

Come si fa a trovare la codifica corretta? Se sei fortunato, sarà inclusa da qualche parte nella documentazione dei dati. Sfortunatamente, questo è raramente il caso, così readr fornisce `guess_encoding()` per aiutarvi a capirlo. Non è infallibile, e funziona meglio quando hai molto testo (diversamente da qui), ma è un punto di partenza ragionevole. Aspettatevi di provare alcune codifiche diverse prima di trovare quella giusta.


```r
guess_encoding(charToRaw(x1))
#> # A tibble: 2 × 2
#>   encoding   confidence
#>   <chr>           <dbl>
#> 1 ISO-8859-1       0.46
#> 2 ISO-8859-9       0.23
guess_encoding(charToRaw(x2))
#> # A tibble: 1 × 2
#>   encoding confidence
#>   <chr>         <dbl>
#> 1 KOI8-R         0.42
```

Il primo argomento di `guess_encoding()` può essere un percorso verso un file, oppure, come in questo caso, un vettore grezzo (utile se le stringhe sono già in R).

Le codifiche sono un argomento ricco e complesso, e qui ho solo grattato la superficie. Se volete saperne di più, vi consiglio di leggere la spiegazione dettagliata su <http://kunststube.net/encoding/>.

### Fattori {#readr-factors}

R usa i fattori per rappresentare variabili categoriche che hanno un insieme noto di possibili valori. Date a `parse_factor()` un vettore di `livelli` noti per generare un avviso ogni volta che è presente un valore inaspettato:


```r
fruit <- c("apple", "banana")
parse_factor(c("apple", "banana", "bananana"), levels = fruit)
#> Warning: 1 parsing failure.
#> row col           expected   actual
#>   3  -- value in level set bananana
#> [1] apple  banana <NA>  
#> attr(,"problems")
#> # A tibble: 1 × 4
#>     row   col expected           actual  
#>   <int> <int> <chr>              <chr>   
#> 1     3    NA value in level set bananana
#> Levels: apple banana
```

Ma se hai molte voci problematiche, spesso è più facile lasciarle come vettori di caratteri e poi usare gli strumenti che imparerai in [stringhe] e [fattori] per pulirle.

### Date, date e orari {#readr-datetimes}

Puoi scegliere tra tre analizzatori a seconda che tu voglia una data (il numero di giorni dal 1970-01-01), una data-ora (il numero di secondi dalla mezzanotte del 1970-01-01), o un'ora (il numero di secondi dalla mezzanotte). Se chiamato senza ulteriori argomenti:

* `parse_datetime()` si aspetta una data-ora ISO8601. ISO8601 è uno
    standard internazionale in cui i componenti di una data sono
    organizzati dal più grande al più piccolo: anno, mese, giorno, ora, minuto, 
    secondo.
    
    
    ```r
    parse_datetime("2010-10-01T2010")
    #> [1] "2010-10-01 20:10:00 UTC"
    # If time is omitted, it will be set to midnight
    parse_datetime("20101010")
    #> [1] "2010-10-10 UTC"
    ```
    
    Questo è lo standard di data/ora più importante, e se lavorate con
    date e ore frequentemente, vi consiglio di leggere
    <https://en.wikipedia.org/wiki/ISO_8601>
    
* `parse_date()` si aspetta un anno a quattro cifre, un `-` o `/`, il mese, un `-` 
    o `/`, poi il giorno:
    
    
    ```r
    parse_date("2010-10-01")
    #> [1] "2010-10-01"
    ```

* `parse_time()` si aspetta l'ora, `:`, i minuti, opzionalmente `:` e i secondi, 
    e uno specificatore opzionale am/pm:
  
    
    ```r
    library(hms)
    parse_time("01:10 am")
    #> 01:10:00
    parse_time("20:10:01")
    #> 20:10:01
    ```
    
    R base non ha integrata una buona classe per i dati temporali, quindi usiamo 
    quella fornita nel pacchetto hms.

Se queste impostazioni predefinite non funzionano per i vostri dati, potete fornire il vostro `formato` di data e ora, composto dai seguenti elementi:

Anno
: `%Y` (4 cifre). 
Anno : `%y` (2 cifre); 00-69 -> 2000-2069, 70-99 -> 1970-1999.

Mese
: `%m` (2 cifre).
: `%b` (nome abbreviato, come "Jan").
: `%B` (nome completo, "Gennaio").

Giorno
: `%d` (2 cifre).
: `%e` (spazio iniziale opzionale).

Ora
: `%H` 0-23 ore.
: `%I` 0-12, deve essere usato con `%p`.
: `%p` indicatore AM/PM.
: `%M` minuti.
: `%S` secondi interi.
: `%OS` secondi reali. 
: `%Z` Fuso orario (come nome, per esempio `America/Chicago`). Attenzione alle abbreviazioni:
  se sei americano, nota che "EST" è un fuso orario canadese che non
  ha l'ora legale. Non è _non_ Eastern Standard Time! Torneremo
  torneremo su questo [fusi orari].
: `%z` (come offset da UTC, ad esempio `+0800`). 

Non cifre
: `%.` salta un carattere non numerico.
: `%*` salta qualsiasi numero di non cifre.

Il modo migliore per capire il formato corretto è quello di creare alcuni esempi in un vettore di caratteri, e testare con una delle funzioni di analisi. Per esempio:


```r
parse_date("01/02/15", "%m/%d/%y")
#> [1] "2015-01-02"
parse_date("01/02/15", "%d/%m/%y")
#> [1] "2015-02-01"
parse_date("01/02/15", "%y/%m/%d")
#> [1] "2001-02-15"
```

Se stai usando `%b` o `%B` con nomi di mesi non inglesi, dovrai impostare l'argomento `lang` a `locale()`. Vedi la lista delle lingue incorporate in `date_names_langs()`, o se la tua lingua non è già inclusa, crea la tua con `date_names()`.


```r
parse_date("1 gennaio 2015", "%d %B %Y", locale = locale("it"))
#> [1] "2015-01-01"
```

### Esercizi

1.  Quali sono gli argomenti più importanti di `locale()`? 

1.  Cosa succede se provate a impostare `decimal_mark` e `grouping_mark`. 
    allo stesso carattere? Cosa succede al valore predefinito di 
    `grouping_mark` quando si imposta `decimal_mark` a ","? Cosa succede
    al valore predefinito di `decimal_mark` quando imposti il `grouping_mark` a ".
    su "."?

1.  Non ho discusso le opzioni `date_format` e `time_format` per
    `locale()`. Cosa fanno? Costruisci un esempio che mostri quando 
    potrebbero essere utili.

1.  Se vivete fuori dagli Stati Uniti, create un nuovo oggetto locale che incapsuli 
    le impostazioni per i tipi di file che leggete più comunemente.
    
1.  Qual è la differenza tra `read_csv()` e `read_csv2()`?
    
1.  Quali sono le codifiche più comuni usate in Europa? Quali sono le
    codifiche più comuni usate in Asia? Fai qualche ricerca su Google per scoprirlo.

1.  Genera la corretta stringa di formato per analizzare ciascuna delle seguenti 
    date e orari:
    
    
    ```r
    d1 <- "January 1, 2010"
    d2 <- "2015-Mar-07"
    d3 <- "06-Jun-2017"
    d4 <- c("August 19 (2015)", "July 1 (2015)")
    d5 <- "12/30/14" # Dec 30, 2014
    t1 <- "1705"
    t2 <- "11:15:10.12 PM"
    ```

## Analizzare un file

Ora che hai imparato come analizzare un singolo vettore, è tempo di tornare all'inizio ed esplorare come readr analizza un file. Ci sono due cose nuove che imparerete in questa sezione:

1. Come readr indovina automaticamente il tipo di ogni colonna.
1. Come sovrascrivere la specifica predefinita.

### Strategia

readr usa un'euristica per capire il tipo di ogni colonna: legge le prime 1000 righe e usa alcune euristiche (moderatamente conservative) per capire il tipo di ogni colonna. Puoi emulare questo processo con un vettore di caratteri usando `guess_parser()`, che restituisce la migliore ipotesi di readr, e `parse_guess()` che usa tale ipotesi per analizzare la colonna:


```r
guess_parser("2010-10-01")
#> [1] "date"
guess_parser("15:01")
#> [1] "time"
guess_parser(c("TRUE", "FALSE"))
#> [1] "logical"
guess_parser(c("1", "5", "9"))
#> [1] "double"
guess_parser(c("12,352,561"))
#> [1] "number"

str(parse_guess("2010-10-10"))
#>  Date[1:1], format: "2010-10-10"
```

L'euristica prova ciascuno dei seguenti tipi, fermandosi quando trova una corrispondenza:

* logico: contiene solo "F", "T", "FALSE" o "TRUE".
* integer: contiene solo caratteri numerici (e `-`).
* double: contiene solo doppi validi (inclusi numeri come `4.5e-5`).
* number: contiene doppi validi con il segno di raggruppamento all'interno.
* time: corrisponde al formato predefinito `time_format`.
* date: corrisponde al formato predefinito `date_format`.
* date-time: qualsiasi data ISO8601.

Se nessuna di queste regole è applicabile, allora la colonna rimarrà come un vettore di stringhe.

### Problemi

Questi valori predefiniti non sempre funzionano per i file più grandi. Ci sono due problemi di base:

1.  Le prime mille righe potrebbero essere un caso speciale, e readr indovina
    un tipo che non è sufficientemente generale. Per esempio, si potrebbe avere 
    una colonna di doppi che contiene solo interi nelle prime 1000 righe. 

1.  La colonna potrebbe contenere molti valori mancanti. Se le prime 1000
    righe contengono solo `NA`, readr penserà che sia un vettore logico 
    vettoriale logico, mentre voi probabilmente volete analizzarlo come qualcosa di più
    specifico.

readr contiene un CSV impegnativo che illustra entrambi questi problemi:


```r
challenge <- read_csv(readr_example("challenge.csv"))
#> Rows: 2000 Columns: 2
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> dbl  (1): x
#> date (1): y
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

(Si noti l'uso di `readr_example()` che trova il percorso di uno dei file inclusi nel pacchetto)

Ci sono due output stampati: la specifica della colonna generata guardando le prime 1000 righe e i primi cinque fallimenti dell'analisi. È sempre una buona idea estrarre esplicitamente i `problems()`, in modo da poterli esplorare più a fondo:


```r
problems(challenge)
#> # A tibble: 0 × 5
#> # … with 5 variables: row <int>, col <int>, expected <chr>, actual <chr>,
#> #   file <chr>
```

Una buona strategia è quella di lavorare colonna per colonna finché non ci sono più problemi. Qui possiamo vedere che ci sono molti problemi di parsing con la colonna `y`. Se guardiamo le ultime righe, vedrete che sono date memorizzate in un vettore di caratteri: 


```r
tail(challenge)
#> # A tibble: 6 × 2
#>       x y         
#>   <dbl> <date>    
#> 1 0.805 2019-11-21
#> 2 0.164 2018-03-29
#> 3 0.472 2014-08-04
#> 4 0.718 2015-08-16
#> 5 0.270 2020-02-04
#> 6 0.608 2019-01-06
```

Questo suggerisce che invece abbiamo bisogno di usare un parser di date. Per correggere la chiamata, iniziate copiando e incollando la specifica della colonna nella vostra chiamata originale:


```r
challenge <- read_csv(
  readr_example("challenge.csv"), 
  col_types = cols(
    x = col_double(),
    y = col_logical()
  )
)
```

Poi potete fissare il tipo della colonna `y` specificando che `y` è una colonna di data:


```r
challenge <- read_csv(
  readr_example("challenge.csv"), 
  col_types = cols(
    x = col_double(),
    y = col_date()
  )
)
tail(challenge)
#> # A tibble: 6 × 2
#>       x y         
#>   <dbl> <date>    
#> 1 0.805 2019-11-21
#> 2 0.164 2018-03-29
#> 3 0.472 2014-08-04
#> 4 0.718 2015-08-16
#> 5 0.270 2020-02-04
#> 6 0.608 2019-01-06
```

Ogni funzione `parse_xyz()` ha una corrispondente funzione `col_xyz()`. Si usa `parse_xyz()` quando i dati sono già in un vettore di caratteri in R; si usa `col_xyz()` quando si vuole dire a readr come caricare i dati.

Raccomando vivamente di fornire sempre `col_types`, partendo dalla stampa fornita da readr. Questo assicura che tu abbia uno script di importazione dati coerente e riproducibile. Se ti affidi alle ipotesi di default e i tuoi dati cambiano, readr continuerà a leggerli. Se vuoi essere davvero rigoroso, usa `stop_for_problems()`: questo lancerà un errore e fermerà lo script se ci sono problemi di analisi.

### Altre strategie

Ci sono alcune altre strategie generali per aiutarti ad analizzare i file:

* Nell'esempio precedente, siamo stati solo sfortunati: se guardiamo solo una riga in più rispetto a quella di default, possiamo analizzare correttamente in un colpo solo:
   
    
    ```r
    challenge2 <- read_csv(readr_example("challenge.csv"), guess_max = 1001)
    #> Rows: 2000 Columns: 2
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> dbl  (1): x
    #> date (1): y
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    challenge2
    #> # A tibble: 2,000 × 2
    #>       x y     
    #>   <dbl> <date>
    #> 1   404 NA    
    #> 2  4172 NA    
    #> 3  3004 NA    
    #> 4   787 NA    
    #> 5    37 NA    
    #> 6  2332 NA    
    #> # … with 1,994 more rows
    ```

* A volte è più facile diagnosticare i problemi se si leggono tutte le colonne come vettori di caratteri:
   
    
    ```r
    challenge2 <- read_csv(readr_example("challenge.csv"), 
      col_types = cols(.default = col_character())
    )
    ```
    
    This is particularly useful in conjunction with `type_convert()`,
    which applies the parsing heuristics to the character columns in a data
    frame.

    
    ```r
    df <- tribble(
      ~x,  ~y,
      "1", "1.21",
      "2", "2.32",
      "3", "4.56"
    )
    df
    #> # A tibble: 3 × 2
    #>   x     y    
    #>   <chr> <chr>
    #> 1 1     1.21 
    #> 2 2     2.32 
    #> 3 3     4.56
    
    # Note the column types
    type_convert(df)
    #> 
    #> ── Column specification ────────────────────────────────────────────────────────
    #> cols(
    #>   x = col_double(),
    #>   y = col_double()
    #> )
    #> # A tibble: 3 × 2
    #>       x     y
    #>   <dbl> <dbl>
    #> 1     1  1.21
    #> 2     2  2.32
    #> 3     3  4.56
    ```
    
* Se stai leggendo un file molto grande, potresti voler impostare `n_max` a
    un numero piccolo come 10.000 o 100.000. Questo accelererà le vostre 
    iterazioni mentre si eliminano i problemi comuni.

* Se stai avendo grossi problemi di analisi, a volte è più facile
    leggere semplicemente un vettore di caratteri di linee con `read_lines()`,
    o anche un vettore di caratteri di lunghezza 1 con `read_file()`. Poi si
    potete usare le abilità di analisi delle stringhe che imparerete in seguito per analizzare
    formati più esotici.

## Scrivere su un file

readr dispone anche di due utili funzioni per scrivere i dati su disco: `write_csv()` e `write_tsv()`. Entrambe le funzioni aumentano le possibilità che il file di output venga letto correttamente:

* codificando sempre le stringhe in UTF-8.
  
* Salvando date e orari in formato ISO8601 in modo che siano facilmente
  analizzati altrove.

Se vuoi esportare un file csv in Excel, usa `write_excel_csv()` --- questo scrive un carattere speciale (un "byte order mark") all'inizio del file che dice a Excel che stai usando la codifica UTF-8.

Gli argomenti più importanti sono `x` (il frame di dati da salvare) e `path` (la posizione in cui salvarlo). Puoi anche specificare come vengono scritti i valori mancanti con `na`, e se vuoi `applicare` ad un file esistente.


```r
write_csv(challenge, "challenge.csv")
```

Si noti che le informazioni sul tipo vengono perse quando si salva in csv:


```r
challenge
#> # A tibble: 2,000 × 2
#>       x y     
#>   <dbl> <date>
#> 1   404 NA    
#> 2  4172 NA    
#> 3  3004 NA    
#> 4   787 NA    
#> 5    37 NA    
#> 6  2332 NA    
#> # … with 1,994 more rows
write_csv(challenge, "challenge-2.csv")
read_csv("challenge-2.csv")
#> Rows: 2000 Columns: 2
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> dbl  (1): x
#> date (1): y
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 2,000 × 2
#>       x y     
#>   <dbl> <date>
#> 1   404 NA    
#> 2  4172 NA    
#> 3  3004 NA    
#> 4   787 NA    
#> 5    37 NA    
#> 6  2332 NA    
#> # … with 1,994 more rows
```

Questo rende i CSV un po' inaffidabili per la memorizzazione dei risultati intermedi - è necessario ricreare la specifica della colonna ogni volta che si carica. Ci sono due alternative:

1.  1. `write_rds()` e `read_rds()` sono wrapper uniformi intorno alle funzioni di base 
    funzioni di base `readRDS()` e `saveRDS()`. Queste memorizzano i dati nel formato binario personalizzato di R 
    formato binario personalizzato di R chiamato RDS:
    
    
    ```r
    write_rds(challenge, "challenge.rds")
    read_rds("challenge.rds")
    #> # A tibble: 2,000 × 2
    #>       x y     
    #>   <dbl> <date>
    #> 1   404 NA    
    #> 2  4172 NA    
    #> 3  3004 NA    
    #> 4   787 NA    
    #> 5    37 NA    
    #> 6  2332 NA    
    #> # … with 1,994 more rows
    ```
  
1.  Il pacchetto `feather` implementa un formato di file binario veloce che può essere condiviso attraverso i linguaggi di programmazione:
    
    
    ```r
    library(feather)
    write_feather(challenge, "challenge.feather")
    read_feather("challenge.feather")
    #> # A tibble: 2,000 x 2
    #>       x      y
    #>   <dbl> <date>
    #> 1   404   <NA>
    #> 2  4172   <NA>
    #> 3  3004   <NA>
    #> 4   787   <NA>
    #> 5    37   <NA>
    #> 6  2332   <NA>
    #> # ... with 1,994 more rows
    ```

Feather tende ad essere più veloce di RDS ed è utilizzabile al di fuori di R. RDS supporta le liste-colonne (che imparerete in [molti modelli]); feather attualmente non lo fa.



## Altri tipi di dati

Per ottenere altri tipi di dati in R, raccomandiamo di iniziare con i pacchetti tidyverse elencati qui sotto. Non sono certamente perfetti, ma sono un buon punto di partenza. Per dati rettangolari:

* __haven__ legge i file SPSS, Stata e SAS.

* __readxl__ legge i file excel (sia `.xls` che `.xlsx`).

* __DBI__, insieme ad un backend specifico per i database (ad esempio __RMySQL__, 
  __RSQLite__, __RPostgreSQL__ ecc.) ti permette di eseguire query SQL su un 
  database e restituire un frame di dati.

Per i dati gerarchici: usate __jsonlite__ (di Jeroen Ooms) per json, e __xml2__ per XML. Jenny Bryan ha alcuni eccellenti esempi funzionanti su <https://jennybc.github.io/purrr-tutorial/>.

Per altri tipi di file, provate il [R data import/export manual](https://cran.r-project.org/doc/manuals/r-release/R-data.html) e il pacchetto [__rio__](https://github.com/leeper/rio).
