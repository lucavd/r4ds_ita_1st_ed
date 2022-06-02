# Stringhe

## Introduzione

Questo capitolo vi introduce alla manipolazione delle stringhe in R. Imparerete le basi di come funzionano le stringhe e come crearle a mano, ma il focus di questo capitolo sarà sulle espressioni regolari, o regexps in breve. Le espressioni regolari sono utili perché le stringhe di solito contengono dati non strutturati o semi-strutturati, e le regexp sono un linguaggio conciso per descrivere modelli nelle stringhe. Quando guardi per la prima volta una regexp, penserai che un gatto abbia camminato sulla tua tastiera, ma man mano che la tua comprensione migliora cominceranno presto ad avere senso.

### Prerequisiti

Questo capitolo si concentrerà sul pacchetto __stringr__ per la manipolazione delle stringhe, che fa parte del core tidyverse.


```r
library(tidyverse)
```

## Nozioni di base sulle stringhe

È possibile creare stringhe con apici singoli o doppi. A differenza di altri linguaggi, non c'è differenza di comportamento. Raccomando di usare sempre `"`, a meno che non vogliate creare una stringa che contenga più `"`.


```r
string1 <- "Questa è una stringa"
string2 <- 'If I want to include a "quote" inside a string, I use single quotes' # non traducibile (altrimenti non si capisce l'esempio)
```

Se dimenticate di chiudere una citazione, vedrete `+`, il carattere di continuazione:

```
> "Questa è una stringa senza virgolette di chiusura
+ 
+ 
+ AIUTO SONO BLOCCATO
```

Se ti succede questo, premi Escape e prova di nuovo!

Per includere una citazione singola o doppia letterale in una stringa puoi usare `\` per "escape":


```r
double_quote <- "\"" # o '"'
single_quote <- '\'' # o "'"
```

Ciò significa che se volete includere un backslash letterale, dovrete raddoppiarlo: `"\\"`.

Attenzione che la rappresentazione stampata di una stringa non è la stessa della stringa stessa, perché la rappresentazione stampata mostra gli escape. Per vedere il contenuto grezzo della stringa, usa `writeLines()`:


```r
x <- c("\"", "\\")
x
#> [1] "\"" "\\"
writeLines(x)
#> "
#> \
```

Ci sono una manciata di altri caratteri speciali. I più comuni sono `"\n"`, newline, e `"\t"`, tab, ma puoi vedere la lista completa chiedendo aiuto su `"`: `?'"'`, o `?"'"`. A volte vedrai anche stringhe come `"\u00b5"`, questo è un modo di scrivere caratteri non inglesi che funziona su tutte le piattaforme:


```r
x <- "\u00b5"
x
#> [1] "µ"
```

Le stringhe multiple sono spesso memorizzate in un vettore di caratteri, che puoi creare con `c()`:


```r
c("una", "due", "tre")
#> [1] "una" "due" "tre"
```

### Lunghezza della stringa

Base R contiene molte funzioni per lavorare con le stringhe, ma le eviteremo perché possono essere incoerenti, il che le rende difficili da ricordare. Invece useremo le funzioni di stringr. Queste hanno nomi più intuitivi e iniziano tutte con `str_`. Per esempio, `str_length()` vi dice il numero di caratteri in una stringa:


```r
str_length(c("a", "R per data science", NA))
#> [1]  1 18 NA
```

Il prefisso comune `str_` è particolarmente utile se usate RStudio, perché digitando `str_` si attiva il completamento automatico, permettendovi di vedere tutte le funzioni di stringr:

<img src="screenshots/stringr-autocomplete.png" width="70%" style="display: block; margin: auto;" />

### Combinazione di stringhe

Per combinare due o più stringhe, usate `str_c()`:


```r
str_c("x", "y")
#> [1] "xy"
str_c("x", "y", "z")
#> [1] "xyz"
```

Usa l'argomento `sep` per controllare come sono separati:


```r
str_c("x", "y", sep = ", ")
#> [1] "x, y"
```

Come molte altre funzioni in R, i valori mancanti sono contagiosi. Se volete stamparli come `"NA"`, usate `str_replace_na()`:


```r
x <- c("abc", NA)
str_c("|-", x, "-|")
#> [1] "|-abc-|" NA
str_c("|-", str_replace_na(x), "-|")
#> [1] "|-abc-|" "|-NA-|"
```

Come mostrato sopra, `str_c()` è vettorializzata, e ricicla automaticamente i vettori più corti alla stessa lunghezza del più lungo:


```r
str_c("prefix-", c("a", "b", "c"), "-suffix")
#> [1] "prefix-a-suffix" "prefix-b-suffix" "prefix-c-suffix"
```

Gli oggetti di lunghezza 0 vengono eliminati silenziosamente. Questo è particolarmente utile insieme a `if`:


```r
name <- "Hadley"
time_of_day <- "morning"
birthday <- FALSE

str_c(
  "Good ", time_of_day, " ", name,
  if (birthday) " and HAPPY BIRTHDAY",
  "."
)
#> [1] "Good morning Hadley."
```

Per far collassare un vettore di stringhe in una singola stringa, usate `collapse`:


```r
str_c(c("x", "y", "z"), collapse = ", ")
#> [1] "x, y, z"
```

### Sottoscrizione di stringhe

Potete estrarre parti di una stringa usando `str_sub()`. Oltre alla stringa, `str_sub()` prende gli argomenti `start` e `end` che danno la posizione (inclusa) della sottostringa:


```r
x <- c("Apple", "Banana", "Pear")
str_sub(x, 1, 3)
#> [1] "App" "Ban" "Pea"
# i numeri negativi contano all'indietro dalla fine
str_sub(x, -3, -1)
#> [1] "ple" "ana" "ear"
```

Si noti che `str_sub()` non fallirà se la stringa è troppo corta: semplicemente restituirà il più possibile:


```r
str_sub("a", 1, 5)
#> [1] "a"
```

Potete anche usare la forma di assegnazione di `str_sub()` per modificare le stringhe:


```r
str_sub(x, 1, 1) <- str_to_lower(str_sub(x, 1, 1))
x
#> [1] "apple"  "banana" "pear"
```

### Locales

Sopra ho usato `str_to_lower()` per cambiare il testo in minuscolo. Puoi anche usare `str_to_upper()` o `str_to_title()`. Tuttavia, cambiare le maiuscole è più complicato di quanto possa sembrare a prima vista, perché lingue diverse hanno regole diverse per cambiare le maiuscole. Puoi scegliere quale insieme di regole usare specificando un locale:


```r
# Il turco ha due "i": con e senza punto, e ha una regola diversa per la loro capitalizzazione:
str_to_upper(c("i", "ı"))
#> [1] "I" "I"
str_to_upper(c("i", "ı"), locale = "tr")
#> [1] "İ" "I"
```

Il locale è specificato come codice di lingua ISO 639, che è un'abbreviazione di due o tre lettere. Se non conosci già il codice della tua lingua, [Wikipedia](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) ha una buona lista. Se lasciate vuoto il locale, verrà usato il locale corrente, come fornito dal vostro sistema operativo.

Un'altra importante operazione che è influenzata dal locale è l'ordinamento. Le funzioni di base R `order()` e `sort()` ordinano le stringhe usando il locale corrente. Se volete un comportamento robusto su diversi computer, potreste voler usare `str_sort()` e `str_order()` che prendono un argomento aggiuntivo `locale`:


```r
x <- c("apple", "eggplant", "banana")

str_sort(x, locale = "en")  # English
#> [1] "apple"    "banana"   "eggplant"

str_sort(x, locale = "haw") # Hawaiian
#> [1] "apple"    "eggplant" "banana"
```

### Esercizi

1.  Nel codice che non usa stringr, vedrete spesso `paste()` e `paste0()`.
    Qual è la differenza tra le due funzioni? A quale funzione di stringr sono
    sono equivalenti? In che modo le funzioni differiscono nella gestione di 
    `NA`?
    
1.  Con parole tue, descrivi la differenza tra gli argomenti `sep` e `collapse
    di `str_c()`.

1.  Usate `str_length()` e `str_sub()` per estrarre il carattere centrale da 
    una stringa. Cosa farete se la stringa ha un numero pari di caratteri?

1.  Cosa fa `str_wrap()`? Quando potreste volerlo usare?

1.  Cosa fa `str_trim()`? Qual è l'opposto di `str_trim()`?

1.  Scrivi una funzione che trasformi (per esempio) un vettore `c("a", "b", "c")` in 
    la stringa `a, b, e c``. Pensa attentamente a cosa dovrebbe fare se
    dato un vettore di lunghezza 0, 1, o 2.

## Corrispondenza di schemi con le espressioni regolari

Le espressioni regolari sono un linguaggio molto conciso che permette di descrivere schemi nelle stringhe. Ci vuole un po' di tempo per capirle, ma una volta che le avete capite, le troverete estremamente utili. 

Per imparare le espressioni regolari, useremo `str_view()` e `str_view_all()`. Queste funzioni prendono un vettore di caratteri e un'espressione regolare, e vi mostrano come corrispondono. Inizieremo con espressioni regolari molto semplici e poi gradualmente diventeremo sempre più complicati. Una volta che hai imparato la corrispondenza dei pattern, imparerai come applicare queste idee con varie funzioni di stringr.

### Corrispondenze di base

I pattern più semplici corrispondono a stringhe esatte:


```r
x <- c("apple", "banana", "pear")
str_view(x, "an")
```

```{=html}
<div id="htmlwidget-ac96cb3ee4656e2e9ec3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-ac96cb3ee4656e2e9ec3">{"x":{"html":"<ul>\n  <li>apple<\/li>\n  <li>b<span class='match'>an<\/span>ana<\/li>\n  <li>pear<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Il passo successivo in termini di complessità è `.`, che corrisponde a qualsiasi carattere (eccetto un newline):


```r
str_view(x, ".a.")
```

```{=html}
<div id="htmlwidget-e5c8c404fe174e4c81bd" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-e5c8c404fe174e4c81bd">{"x":{"html":"<ul>\n  <li>apple<\/li>\n  <li><span class='match'>ban<\/span>ana<\/li>\n  <li>p<span class='match'>ear<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

But if "`.`" matches any character, how do you match the character "`.`"? You need to use an "escape" to tell the regular expression you want to match it exactly, not use its special behaviour. Like strings, regexps use the backslash, `\`, to escape special behaviour. So to match an `.`, you need the regexp `\.`. Unfortunately this creates a problem. We use strings to represent regular expressions, and `\` is also used as an escape symbol in strings. So to create the regular expression `\.` we need the string `"\\."`. `


```r
# Per creare l'espressione regolare, abbiamo bisogno di \
dot <- "\\."

# Ma l'espressione stessa ne contiene solo uno:
writeLines(dot)
#> \.

# E questo dice a R di cercare un esplicito .
str_view(c("abc", "a.c", "bef"), "a\\.c")
```

```{=html}
<div id="htmlwidget-ac96cb3ee4656e2e9ec3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-ac96cb3ee4656e2e9ec3">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li><span class='match'>a.c<\/span><\/li>\n  <li>bef<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Se `\` è usato come carattere di escape nelle espressioni regolari, come si fa a far corrispondere un letterale `\`? Beh, devi fare l'escape, creando l'espressione regolare `\\`. Per creare questa espressione regolare, hai bisogno di usare una stringa, che deve anche fare l'escape di `\`. Questo significa che per far corrispondere un letterale `\` hai bisogno di scrivere `"\\\\"` --- hai bisogno di quattro backslash per corrispondere a uno!


```r
x <- "a\\b"
writeLines(x)
#> a\b

str_view(x, "\\\\")
```

```{=html}
<div id="htmlwidget-febe03efa1a2d8d52a86" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-febe03efa1a2d8d52a86">{"x":{"html":"<ul>\n  <li>a<span class='match'>\\<\/span>b<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

In questo libro, scriverò l'espressione regolare come "\" e le stringhe che rappresentano l'espressione regolare come "\".

#### Esercizi

1.  Spiega perché ognuna di queste stringhe non corrisponde a un'espressione regolare: `\`: `"\"`, `"\\"`, `"\\\"`.

1.  Come faresti a far corrispondere la sequenza `"'\`?

1.  A quali schemi corrisponderà l'espressione regolare  `\..\..\..` ? 
    Come la rappresenteresti come stringa?

### Ancore

Per default, le espressioni regolari corrispondono a qualsiasi parte di una stringa. E' spesso utile _ancorare_ l'espressione regolare in modo che corrisponda all'inizio o alla fine della stringa. Puoi usare:

* `^` per far corrispondere l'inizio della stringa.
* `$` per corrispondere alla fine della stringa.


```r
x <- c("apple", "banana", "pear")
str_view(x, "^a")
```

```{=html}
<div id="htmlwidget-1fb4450895fe099f74a1" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-1fb4450895fe099f74a1">{"x":{"html":"<ul>\n  <li><span class='match'>a<\/span>pple<\/li>\n  <li>banana<\/li>\n  <li>pear<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(x, "a$")
```

```{=html}
<div id="htmlwidget-10b3b7155e8045a1b2ad" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-10b3b7155e8045a1b2ad">{"x":{"html":"<ul>\n  <li>apple<\/li>\n  <li>banan<span class='match'>a<\/span><\/li>\n  <li>pear<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Per ricordare quale sia, provate questo mnemonico che ho imparato da [Evan Misshula](https://twitter.com/emisshula/status/323863393167613953): se iniziate con power (`^`), finite con money (`$`).

Per forzare un'espressione regolare a corrispondere solo ad una stringa completa, ancorala con entrambi `^` e `$`:


```r
x <- c("apple pie", "apple", "apple cake")
str_view(x, "apple")
```

```{=html}
<div id="htmlwidget-4018eef1a407a0df6b52" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-4018eef1a407a0df6b52">{"x":{"html":"<ul>\n  <li><span class='match'>apple<\/span> pie<\/li>\n  <li><span class='match'>apple<\/span><\/li>\n  <li><span class='match'>apple<\/span> cake<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(x, "^apple$")
```

```{=html}
<div id="htmlwidget-5b1b2f4ad92281566982" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-5b1b2f4ad92281566982">{"x":{"html":"<ul>\n  <li>apple pie<\/li>\n  <li><span class='match'>apple<\/span><\/li>\n  <li>apple cake<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Potete anche abbinare il confine tra le parole con `\b`. Non lo uso spesso in R, ma a volte lo uso quando faccio una ricerca in RStudio quando voglio trovare il nome di una funzione che è un componente di altre funzioni. Per esempio, cercherò `\bsum\b` per evitare di abbinare `summarise`, `summary`, `rowsum` e così via.

#### Esercizi

1.  Come faresti ad abbinare la stringa letterale `"$^$"`?

1.  Dato il corpus di parole comuni in `stringr::words`, create espressioni regolari
    espressioni regolari che trovino tutte le parole che:
    
    1. Inizia con "y".
    1.  Finisce con "x".
    1. Sono esattamente tre lettere. (Non barare usando `str_length()`!)
    1. Hanno sette o più lettere.

    Poiché questa lista è lunga, potresti voler usare l'argomento `match` a
    `str_view()` per mostrare solo le parole corrispondenti o non corrispondenti.

### Classi di caratteri e alternative

Ci sono un certo numero di modelli speciali che corrispondono a più di un carattere. Hai già visto `.`, che corrisponde a qualsiasi carattere a parte un newline. Ci sono altri quattro utili strumenti:

* `\d`: corrisponde a qualsiasi cifra.
* `\s`: corrisponde a qualsiasi spazio bianco (es. spazio, tabulazione, newline).
* `[abc]`: corrisponde ad a, b, o c.
* `[^abc]`: corrisponde a qualsiasi cosa tranne a, b, o c.

Ricorda, per creare un'espressione regolare che contenga `\d` o `\s`, dovrai fare l'escape del `\d` per la stringa, quindi digiterai `"\\d"` o `"\\s"`.

Una classe di caratteri contenente un singolo carattere è una buona alternativa alle escape di backslash quando vuoi includere un singolo metacarattere in una regex. Molte persone lo trovano più leggibile.


```r
# Cerca un carattere letterale che normalmente ha un significato speciale in una regex
str_view(c("abc", "a.c", "a*c", "a c"), "a[.]c")
```

```{=html}
<div id="htmlwidget-e5c8c404fe174e4c81bd" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-e5c8c404fe174e4c81bd">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li><span class='match'>a.c<\/span><\/li>\n  <li>a*c<\/li>\n  <li>a c<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(c("abc", "a.c", "a*c", "a c"), ".[*]c")
```

```{=html}
<div id="htmlwidget-36aa3d2a04d42bbc2145" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-36aa3d2a04d42bbc2145">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li>a.c<\/li>\n  <li><span class='match'>a*c<\/span><\/li>\n  <li>a c<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(c("abc", "a.c", "a*c", "a c"), "a[ ]")
```

```{=html}
<div id="htmlwidget-febe03efa1a2d8d52a86" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-febe03efa1a2d8d52a86">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li>a.c<\/li>\n  <li>a*c<\/li>\n  <li><span class='match'>a <\/span>c<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Questo funziona per la maggior parte (ma non per tutti) i metacaratteri regex: `$` `.` `|` `?` `*` `+` `(` `)` `[` `{`. Sfortunatamente, alcuni caratteri hanno un significato speciale anche all'interno di una classe di caratteri e devono essere gestiti con escape di backslash: `]` `\` `^` e `-`.

Puoi usare _alternation_ per scegliere tra uno o più schemi alternativi. Per esempio, `abc|d..f` corrisponderà sia a '"abc"', sia a `"deaf"`. Nota che la precedenza per `|` è bassa, così che `abc|xyz`` corrisponde a `abc` o `xyz`, non a `abcyz` o `abxyz`. Come con le espressioni matematiche, se la precedenza dovesse confondere, usate le parentesi per rendere chiaro ciò che volete:


```r
str_view(c("grey", "gray"), "gr(e|a)y")
```

```{=html}
<div id="htmlwidget-72cbf064100ce560a04c" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-72cbf064100ce560a04c">{"x":{"html":"<ul>\n  <li><span class='match'>grey<\/span><\/li>\n  <li><span class='match'>gray<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

#### Esercizi

1.  Creare espressioni regolari per trovare tutte le parole che:

    1. Inizia con una vocale.

    1. 2. Che contengono solo consonanti. (Suggerimento: pensare di abbinare 
       "non" vocali).

    1. Finiscono con `ed`, ma non con `eed`.
    
    1. Termina con `ing` o `ise`.
    
1.  Verificare empiricamente la regola "i prima di e tranne dopo c".

1.  La "q" è sempre seguita da una "u"?

1.  Scrivi un'espressione regolare che corrisponda ad una parola se è probabilmente scritta
    in inglese britannico e non in inglese americano.

1.  Crea un'espressione regolare che corrisponda ai numeri di telefono come comunemente
    scritto nel tuo paese.

### Ripetizione

Il prossimo passo in termini di potenza coinvolge il controllo di quante volte un pattern corrisponde:

* `?`: 0 o 1
* `+`: 1 o più
* `*`: 0 o più


```r
x <- "1888 è l'anno più lungo in numeri romani: MDCCCLXXXVIII"
str_view(x, "CC?")
```

```{=html}
<div id="htmlwidget-1fb4450895fe099f74a1" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-1fb4450895fe099f74a1">{"x":{"html":"<ul>\n  <li>1888 è l'anno più lungo in numeri romani: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(x, "CC+")
```

```{=html}
<div id="htmlwidget-10b3b7155e8045a1b2ad" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-10b3b7155e8045a1b2ad">{"x":{"html":"<ul>\n  <li>1888 è l'anno più lungo in numeri romani: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(x, 'C[LX]+')
```

```{=html}
<div id="htmlwidget-4018eef1a407a0df6b52" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-4018eef1a407a0df6b52">{"x":{"html":"<ul>\n  <li>1888 è l'anno più lungo in numeri romani: MDCC<span class='match'>CLXXX<\/span>VIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Notate che la precedenza di questi operatori è alta, quindi potete scrivere: `colou?r` per abbinare sia l'ortografia americana che quella britannica. Ciò significa che la maggior parte degli usi avrà bisogno di parentesi, come `bana(na)+`.

Puoi anche specificare il numero di corrispondenze in modo preciso:

* `{n}`: esattamente n
* `{n,}`: n o più
* `{,m}`: al massimo m
* `{n,m}`: tra n e m


```r
str_view(x, "C{2}")
```

```{=html}
<div id="htmlwidget-28515d92cb327f90c9eb" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-28515d92cb327f90c9eb">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(x, "C{2,}")
```

```{=html}
<div id="htmlwidget-0caf26d4e3c00206b0c5" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-0caf26d4e3c00206b0c5">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(x, "C{2,3}")
```

```{=html}
<div id="htmlwidget-da0b268a2927f570ebf3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-da0b268a2927f570ebf3">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Per default queste corrispondenze sono "avide": corrisponderanno alla stringa più lunga possibile. Potete renderle "pigre", facendo corrispondere la stringa più corta possibile mettendo un `?` dopo di esse. Questa è una caratteristica avanzata delle espressioni regolari, ma è utile sapere che esiste:


```r
str_view(x, 'C{2,3}?')
```

```{=html}
<div id="htmlwidget-0ed12bb554391c49c2e3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-0ed12bb554391c49c2e3">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
str_view(x, 'C[LX]+?')
```

```{=html}
<div id="htmlwidget-ec658d41f8c4f2d124e9" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-ec658d41f8c4f2d124e9">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MDCC<span class='match'>CL<\/span>XXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

#### Esercizi

1.  Descrivi gli equivalenti di `?`, `+`, `*` in forma `{m,n}`.

1.  Descrivi a parole a cosa corrispondono queste espressioni regolari:
    (leggi attentamente per vedere se sto usando un'espressione regolare o una stringa che definisce un'espressione regolare).
    che definisce un'espressione regolare).

    1. `^.*$`
    1. `"\\{.+\\}"`
    1. 1. "4" - "2" - "2
    1. `"\\\\{4}"`

1.  Creare espressioni regolari per trovare tutte le parole che:

    1. Iniziano con tre consonanti.
    1. 2. Hanno tre o più vocali in fila.
    1. Avere due o più coppie vocale-consonante in fila.

1.  Risolvi i cruciverba regexp per principianti su
    <https://regexcrossword.com/challenges/beginner>.

### Raggruppamento e backreferences

Prima hai imparato a conoscere le parentesi come un modo per disambiguare espressioni complesse. Le parentesi creano anche un gruppo di cattura _numerato_ (numero 1, 2 ecc.). Un gruppo di cattura memorizza _la parte di stringa_ a cui corrisponde la parte dell'espressione regolare all'interno delle parentesi. Si può fare riferimento allo stesso testo precedentemente trovato da un gruppo di cattura con _backreferences_, come ``1`, ``2`` ecc. Per esempio, la seguente espressione regolare trova tutti i frutti che hanno una coppia di lettere ripetute.


```r
str_view(fruit, "(..)\\1", match = TRUE)
```

```{=html}
<div id="htmlwidget-6b83523733b890d61edc" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-6b83523733b890d61edc">{"x":{"html":"<ul>\n  <li>b<span class='match'>anan<\/span>a<\/li>\n  <li><span class='match'>coco<\/span>nut<\/li>\n  <li><span class='match'>cucu<\/span>mber<\/li>\n  <li><span class='match'>juju<\/span>be<\/li>\n  <li><span class='match'>papa<\/span>ya<\/li>\n  <li>s<span class='match'>alal<\/span> berry<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

(A breve, vedrete anche come sono utili insieme a `str_match()`.)

#### Esercizi

1.  Descrivete, a parole, a cosa corrispondono queste espressioni:

    1. `(.)\1\1`
    1. `"(.)(.)\\2\\1"`
    1. `(..)\1`
    1. `"(.).\\1.\\1"`
    1. `"(.)(.)(.).*\\3\\2\\1"`

1.  Costruire espressioni regolari per far corrispondere parole che:

    1. Iniziano e finiscono con lo stesso carattere.
    
    1. 2. Contengono una coppia di lettere ripetute
       (es. "church" contiene "ch" ripetuto due volte).
    
    1. Contiene una lettera ripetuta in almeno tre punti
       (es. "eleven" contiene tre "e").

## Strumenti

Ora che avete imparato le basi delle espressioni regolari, è il momento di imparare come applicarle ai problemi reali. In questa sezione imparerete una vasta gamma di funzioni di stringr che vi permettono di:

* Determinare quali stringhe corrispondono ad uno schema.
* Trovare le posizioni delle corrispondenze.
* Estrarre il contenuto delle corrispondenze.
* Sostituire le corrispondenze con nuovi valori.
* Dividere una stringa sulla base di una corrispondenza.

Una parola di cautela prima di continuare: poiché le espressioni regolari sono così potenti, è facile provare a risolvere ogni problema con una singola espressione regolare. Nelle parole di Jamie Zawinski:

> Alcune persone, di fronte ad un problema pensano: "Lo so, userò le espressioni regolari". Ora hanno due problemi. 

Come racconto ammonitore, guardate questa espressione regolare che controlla se un indirizzo email è valido:

```
(?:(?:\r\n)?[ \t])*(?:(?:(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t]
)+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:
\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(
?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ 
\t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\0
31]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\
](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+
(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:
(?:\r\n)?[ \t])*))*|(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z
|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)
?[ \t])*)*\<(?:(?:\r\n)?[ \t])*(?:@(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\
r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[
 \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)
?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t]
)*))*(?:,@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[
 \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*
)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t]
)+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*)
*:(?:(?:\r\n)?[ \t])*)?(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+
|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r
\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:
\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t
]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031
]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](
?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?
:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?
:\r\n)?[ \t])*))*\>(?:(?:\r\n)?[ \t])*)|(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?
:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?
[ \t]))*"(?:(?:\r\n)?[ \t])*)*:(?:(?:\r\n)?[ \t])*(?:(?:(?:[^()<>@,;:\\".\[\] 
\000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|
\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>
@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"
(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t]
)*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?
:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[
\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*|(?:[^()<>@,;:\\".\[\] \000-
\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(
?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)*\<(?:(?:\r\n)?[ \t])*(?:@(?:[^()<>@,;
:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([
^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\"
.\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\
]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*(?:,@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\
[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\
r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] 
\000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]
|\\.)*\](?:(?:\r\n)?[ \t])*))*)*:(?:(?:\r\n)?[ \t])*)?(?:[^()<>@,;:\\".\[\] \0
00-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\
.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,
;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?
:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*
(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".
\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[
^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]
]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*\>(?:(?:\r\n)?[ \t])*)(?:,\s*(
?:(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(
?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[
\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t
])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t
])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?
:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|
\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*|(?:
[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\
]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)*\<(?:(?:\r\n)
?[ \t])*(?:@(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["
()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)
?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>
@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*(?:,@(?:(?:\r\n)?[
 \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,
;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t]
)*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*)*:(?:(?:\r\n)?[ \t])*)?
(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".
\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:
\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\[
"()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])
*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])
+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\
.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z
|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*\>(?:(
?:\r\n)?[ \t])*))*)?;\s*)
```

Questo è un esempio un po' patologico (perché gli indirizzi e-mail sono in realtà sorprendentemente complessi), ma è usato nel codice reale. Vedi la discussione su stackoverflow a <http://stackoverflow.com/a/201378> per maggiori dettagli.

Non dimenticate che siete in un linguaggio di programmazione e avete altri strumenti a vostra disposizione. Invece di creare un'espressione regolare complessa, spesso è più facile scrivere una serie di regexp più semplici. Se vi bloccate cercando di creare una singola espressione regolare che risolva il vostro problema, fate un passo indietro e pensate se potete suddividere il problema in pezzi più piccoli, risolvendo ogni sfida prima di passare a quella successiva.

### Rilevare le corrispondenze

Per determinare se un vettore di caratteri corrisponde ad un pattern, usate `str_detect()`. Restituisce un vettore logico della stessa lunghezza dell'input:


```r
x <- c("apple", "banana", "pear")
str_detect(x, "e")
#> [1]  TRUE FALSE  TRUE
```

Ricordate che quando usate un vettore logico in un contesto numerico, `FALSE` diventa 0 e `TRUE` diventa 1. Questo rende `sum()` e `mean()` utili se volete rispondere a domande sulle corrispondenze in un vettore più grande:


```r
# Quante parole comuni iniziano con la t?
sum(str_detect(words, "^t"))
#> [1] 65
# Quale proporzione di parole comuni finisce con una vocale?
mean(str_detect(words, "[aeiou]$"))
#> [1] 0.2765306
```

Quando si hanno condizioni logiche complesse (ad es. corrisponde a o b ma non c a meno che d) è spesso più facile combinare più chiamate `str_detect()` con operatori logici, piuttosto che cercare di creare una singola espressione regolare. Per esempio, ecco due modi per trovare tutte le parole che non contengono alcuna vocale:


```r
# Trova tutte le parole che contengono almeno una vocale e nega
no_vowels_1 <- !str_detect(words, "[aeiou]")
# Trova tutte le parole composte solo da consonanti (non vocali)
no_vowels_2 <- str_detect(words, "^[^aeiou]+$")
identical(no_vowels_1, no_vowels_2)
#> [1] TRUE
```

I risultati sono identici, ma penso che il primo approccio sia significativamente più facile da capire. Se la vostra espressione regolare diventa troppo complicata, provate a scomporla in pezzi più piccoli, dando ad ogni pezzo un nome, e poi combinando i pezzi con operazioni logiche.

Un uso comune di `str_detect()` è quello di selezionare gli elementi che corrispondono ad un pattern. Potete farlo con il sottoinsieme logico o con il comodo wrapper `str_subset()`:


```r
words[str_detect(words, "x$")]
#> [1] "box" "sex" "six" "tax"
str_subset(words, "x$")
#> [1] "box" "sex" "six" "tax"
```

Tipicamente, però, le vostre stringhe saranno una colonna di un frame di dati, e vorrete invece usare filter:


```r
df <- tibble(
  word = words, 
  i = seq_along(word)
)
df %>% 
  filter(str_detect(word, "x$"))
#> # A tibble: 4 × 2
#>   word      i
#>   <chr> <int>
#> 1 box     108
#> 2 sex     747
#> 3 six     772
#> 4 tax     841
```


Una variazione di `str_detect()` è `str_count()`: piuttosto che un semplice sì o no, vi dice quante corrispondenze ci sono in una stringa:

```r
x <- c("apple", "banana", "pear")
str_count(x, "a")
#> [1] 1 3 1

# In media, quante vocali per parola?
mean(str_count(words, "[aeiou]"))
#> [1] 1.991837
```

È naturale usare `str_count()` con `mutate()`:


```r
df %>% 
  mutate(
    vowels = str_count(word, "[aeiou]"),
    consonants = str_count(word, "[^aeiou]")
  )
#> # A tibble: 980 × 4
#>   word         i vowels consonants
#>   <chr>    <int>  <int>      <int>
#> 1 a            1      1          0
#> 2 able         2      2          2
#> 3 about        3      3          2
#> 4 absolute     4      4          4
#> 5 accept       5      2          4
#> 6 account      6      3          4
#> # … with 974 more rows
```

Notate che le corrispondenze non si sovrappongono mai. Per esempio, in `"abababa"`, quante volte corrisponderà il pattern `"aba"`? Le espressioni regolari dicono due, non tre:


```r
str_count("abababa", "aba")
#> [1] 2
str_view_all("abababa", "aba")
```

```{=html}
<div id="htmlwidget-b3f7c917b6c8ff580948" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-b3f7c917b6c8ff580948">{"x":{"html":"<ul>\n  <li><span class='match'>aba<\/span>b<span class='match'>aba<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
```

Notate l'uso di `str_view_all()`. Come imparerete a breve, molte funzioni di stringr sono in coppia: una funzione lavora con una singola corrispondenza, e l'altra lavora con tutte le corrispondenze. La seconda funzione avrà il suffisso `_all`.

#### Esercizi

1.  Per ciascuna delle seguenti sfide, prova a risolverla usando sia una singola
    espressione regolare che una combinazione di chiamate multiple `str_detect()`.
    
    1.  Trova tutte le parole che iniziano o finiscono con `x`.
    
    1.  Trova tutte le parole che iniziano con una vocale e finiscono con una consonante.
    
    1.  Ci sono parole che contengono almeno una di ogni diversa
        vocale?

1.  Quale parola ha il maggior numero di vocali? Quale parola ha la più alta
    proporzione di vocali? (Suggerimento: qual è il denominatore?)

### Estrarre le corrispondenze

Per estrarre il testo effettivo di una corrispondenza, usa `str_extract()`. Per mostrarlo, avremo bisogno di un esempio più complicato. Userò le [frasi di Harvard](https://en.wikipedia.org/wiki/Harvard_sentences), che sono state progettate per testare i sistemi VOIP, ma sono anche utili per fare pratica con le regexp. Queste sono fornite in `stringr::sentences`:


```r
length(sentences)
#> [1] 720
head(sentences)
#> [1] "The birch canoe slid on the smooth planks." 
#> [2] "Glue the sheet to the dark blue background."
#> [3] "It's easy to tell the depth of a well."     
#> [4] "These days a chicken leg is a rare dish."   
#> [5] "Rice is often served in round bowls."       
#> [6] "The juice of lemons makes fine punch."
```

Immaginiamo di voler trovare tutte le frasi che contengono un colore. Creiamo prima un vettore di nomi di colori e poi lo trasformiamo in un'unica espressione regolare:


```r
colours <- c("red", "orange", "yellow", "green", "blue", "purple")
colour_match <- str_c(colours, collapse = "|")
colour_match
#> [1] "red|orange|yellow|green|blue|purple"
```

Ora possiamo selezionare le frasi che contengono un colore, e poi estrarre il colore per capire qual è:


```r
has_colour <- str_subset(sentences, colour_match)
matches <- str_extract(has_colour, colour_match)
head(matches)
#> [1] "blue" "blue" "red"  "red"  "red"  "blue"
```

Notate che `str_extract()` estrae solo la prima corrispondenza. Possiamo vederlo più facilmente selezionando prima tutte le frasi che hanno più di 1 corrispondenza:

```r
more <- sentences[str_count(sentences, colour_match) > 1]
str_view_all(more, colour_match)
```

```{=html}
<div id="htmlwidget-d258b2ee1c304ebe1664" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-d258b2ee1c304ebe1664">{"x":{"html":"<ul>\n  <li>It is hard to erase <span class='match'>blue<\/span> or <span class='match'>red<\/span> ink.<\/li>\n  <li>The <span class='match'>green<\/span> light in the brown box flicke<span class='match'>red<\/span>.<\/li>\n  <li>The sky in the west is tinged with <span class='match'>orange<\/span> <span class='match'>red<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>

str_extract(more, colour_match)
#> [1] "blue"   "green"  "orange"
```

Questo è uno schema comune per le funzioni di stringr, perché lavorare con una singola corrispondenza vi permette di usare strutture dati molto più semplici. Per ottenere tutte le corrispondenze, usate `str_extract_all()`. Restituisce una lista:


```r
str_extract_all(more, colour_match)
#> [[1]]
#> [1] "blue" "red" 
#> 
#> [[2]]
#> [1] "green" "red"  
#> 
#> [[3]]
#> [1] "orange" "red"
```

Imparerai di più sulle liste in [liste](#lists) e [iterazioni].

Se usate `simplify = TRUE`, `str_extract_all()` restituirà una matrice con le corrispondenze brevi espanse alla stessa lunghezza della più lunga:


```r
str_extract_all(more, colour_match, simplify = TRUE)
#>      [,1]     [,2] 
#> [1,] "blue"   "red"
#> [2,] "green"  "red"
#> [3,] "orange" "red"

x <- c("a", "a b", "a b c")
str_extract_all(x, "[a-z]", simplify = TRUE)
#>      [,1] [,2] [,3]
#> [1,] "a"  ""   ""  
#> [2,] "a"  "b"  ""  
#> [3,] "a"  "b"  "c"
```

#### Esercizi

1.  Nell'esempio precedente, potreste aver notato che l'espressione regolare
    corrispondeva a "flickered", che non è un colore. Modificate la 
    regex per risolvere il problema.

1.  Dai dati delle frasi di Harvard, estrai:

    1. La prima parola di ogni frase.
    1. Tutte le parole che finiscono in `ing`.
    1. Tutti i plurali.

### Corrispondenze raggruppate

All'inizio di questo capitolo abbiamo parlato dell'uso delle parentesi per chiarire le precedenze e per i rinvii durante la corrispondenza. Si possono anche usare le parentesi per estrarre parti di una corrispondenza complessa. Per esempio, immaginiamo di voler estrarre i nomi dalle frasi. Come euristica, cercheremo qualsiasi parola che viene dopo "a" o "the". Definire una "parola" in un'espressione regolare è un po' complicato, quindi qui uso una semplice approssimazione: una sequenza di almeno un carattere che non sia uno spazio.


```r
noun <- "(a|the) ([^ ]+)"

has_noun <- sentences %>%
  str_subset(noun) %>%
  head(10)
has_noun %>% 
  str_extract(noun)
#>  [1] "the smooth" "the sheet"  "the depth"  "a chicken"  "the parked"
#>  [6] "the sun"    "the huge"   "the ball"   "the woman"  "a helps"
```

`str_extract()` ci dà la corrispondenza completa; `str_match()` dà ogni singolo componente. Invece di un vettore di caratteri, restituisce una matrice, con una colonna per la corrispondenza completa seguita da una colonna per ogni gruppo:


```r
has_noun %>% 
  str_match(noun)
#>       [,1]         [,2]  [,3]     
#>  [1,] "the smooth" "the" "smooth" 
#>  [2,] "the sheet"  "the" "sheet"  
#>  [3,] "the depth"  "the" "depth"  
#>  [4,] "a chicken"  "a"   "chicken"
#>  [5,] "the parked" "the" "parked" 
#>  [6,] "the sun"    "the" "sun"    
#>  [7,] "the huge"   "the" "huge"   
#>  [8,] "the ball"   "the" "ball"   
#>  [9,] "the woman"  "the" "woman"  
#> [10,] "a helps"    "a"   "helps"
```

(Non sorprende che la nostra euristica per individuare i sostantivi sia povera, e che raccolga anche aggettivi come smooth e parked).

Se i vostri dati sono in un tibble, è spesso più facile usare `tidyr::extract()`. Funziona come `str_match()` ma richiede di dare un nome alle corrispondenze, che vengono poi inserite in nuove colonne:


```r
tibble(sentence = sentences) %>% 
  tidyr::extract(
    sentence, c("article", "noun"), "(a|the) ([^ ]+)", 
    remove = FALSE
  )
#> # A tibble: 720 × 3
#>   sentence                                    article noun   
#>   <chr>                                       <chr>   <chr>  
#> 1 The birch canoe slid on the smooth planks.  the     smooth 
#> 2 Glue the sheet to the dark blue background. the     sheet  
#> 3 It's easy to tell the depth of a well.      the     depth  
#> 4 These days a chicken leg is a rare dish.    a       chicken
#> 5 Rice is often served in round bowls.        <NA>    <NA>   
#> 6 The juice of lemons makes fine punch.       <NA>    <NA>   
#> # … with 714 more rows
```
Come `str_extract()`, se volete tutte le corrispondenze per ogni stringa, avrete bisogno di `str_match_all()`.

#### Esercizi

1. Trova tutte le parole che vengono dopo un "numero" come "uno", "due", "tre" ecc. Estrai sia il numero che la parola.

1. Trova tutte le contrazioni. Separa i pezzi prima e dopo l'apostrofo.

### Sostituzione delle corrispondenze

`str_replace()` e `str_replace_all()` vi permettono di sostituire le corrispondenze con nuove stringhe. L'uso più semplice è quello di sostituire un pattern con una stringa fissa:


```r
x <- c("apple", "pear", "banana")
str_replace(x, "[aeiou]", "-")
#> [1] "-pple"  "p-ar"   "b-nana"
str_replace_all(x, "[aeiou]", "-")
#> [1] "-ppl-"  "p--r"   "b-n-n-"
```

Con `str_replace_all()` potete eseguire sostituzioni multiple fornendo un vettore con nome:


```r
x <- c("1 house", "2 cars", "3 people")
str_replace_all(x, c("1" = "one", "2" = "two", "3" = "three"))
#> [1] "one house"    "two cars"     "three people"
```

Invece di sostituire con una stringa fissa potete usare i backreferences per inserire i componenti della corrispondenza. Nel codice seguente, inverto l'ordine della seconda e della terza parola.


```r
sentences %>% 
  str_replace("([^ ]+) ([^ ]+) ([^ ]+)", "\\1 \\3 \\2") %>% 
  head(5)
#> [1] "The canoe birch slid on the smooth planks." 
#> [2] "Glue sheet the to the dark blue background."
#> [3] "It's to easy tell the depth of a well."     
#> [4] "These a days chicken leg is a rare dish."   
#> [5] "Rice often is served in round bowls."
```

#### Esercizi

1.   Sostituisci tutte le barre in avanti in una stringa con barre rovesciate.

1.   Implementare una semplice versione di `str_to_lower()` usando `replace_all()`.

1.   Cambiate la prima e l'ultima lettera in `words`. Quali di queste stringhe sono ancora parole?

### Divisione

Usa `str_split()` per dividere una stringa in pezzi. Per esempio, possiamo dividere le frasi in parole:


```r
sentences %>%
  head(5) %>% 
  str_split(" ")
#> [[1]]
#> [1] "The"     "birch"   "canoe"   "slid"    "on"      "the"     "smooth" 
#> [8] "planks."
#> 
#> [[2]]
#> [1] "Glue"        "the"         "sheet"       "to"          "the"        
#> [6] "dark"        "blue"        "background."
#> 
#> [[3]]
#> [1] "It's"  "easy"  "to"    "tell"  "the"   "depth" "of"    "a"     "well."
#> 
#> [[4]]
#> [1] "These"   "days"    "a"       "chicken" "leg"     "is"      "a"      
#> [8] "rare"    "dish."  
#> 
#> [[5]]
#> [1] "Rice"   "is"     "often"  "served" "in"     "round"  "bowls."
```

Poiché ogni componente potrebbe contenere un numero diverso di pezzi, questo restituisce una lista. Se state lavorando con un vettore di lunghezza-1, la cosa più semplice è semplicemente estrarre il primo elemento della lista:


```r
"a|b|c|d" %>% 
  str_split("\\|") %>% 
  .[[1]]
#> [1] "a" "b" "c" "d"
```

Altrimenti, come le altre funzioni di stringr che restituiscono una lista, potete usare `simplify = TRUE` per restituire una matrice:


```r
sentences %>%
  head(5) %>% 
  str_split(" ", simplify = TRUE)
#>      [,1]    [,2]    [,3]    [,4]      [,5]  [,6]    [,7]     [,8]         
#> [1,] "The"   "birch" "canoe" "slid"    "on"  "the"   "smooth" "planks."    
#> [2,] "Glue"  "the"   "sheet" "to"      "the" "dark"  "blue"   "background."
#> [3,] "It's"  "easy"  "to"    "tell"    "the" "depth" "of"     "a"          
#> [4,] "These" "days"  "a"     "chicken" "leg" "is"    "a"      "rare"       
#> [5,] "Rice"  "is"    "often" "served"  "in"  "round" "bowls." ""           
#>      [,9]   
#> [1,] ""     
#> [2,] ""     
#> [3,] "well."
#> [4,] "dish."
#> [5,] ""
```

Puoi anche richiedere un numero massimo di pezzi:


```r
fields <- c("Name: Hadley", "Country: NZ", "Age: 35")
fields %>% str_split(": ", n = 2, simplify = TRUE)
#>      [,1]      [,2]    
#> [1,] "Name"    "Hadley"
#> [2,] "Country" "NZ"    
#> [3,] "Age"     "35"
```

Invece di dividere le stringhe per pattern, potete anche dividere per carattere, linea, frase e parola `boundary()`:


```r
x <- "This is a sentence.  This is another sentence."
str_view_all(x, boundary("word"))
```

```{=html}
<div id="htmlwidget-b8f31ebacaee3527bb86" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-b8f31ebacaee3527bb86">{"x":{"html":"<ul>\n  <li><span class='match'>This<\/span> <span class='match'>is<\/span> <span class='match'>a<\/span> <span class='match'>sentence<\/span>.  <span class='match'>This<\/span> <span class='match'>is<\/span> <span class='match'>another<\/span> <span class='match'>sentence<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>

str_split(x, " ")[[1]]
#> [1] "This"      "is"        "a"         "sentence." ""          "This"     
#> [7] "is"        "another"   "sentence."
str_split(x, boundary("word"))[[1]]
#> [1] "This"     "is"       "a"        "sentence" "This"     "is"       "another" 
#> [8] "sentence"
```

#### Esercizi

1.  Dividete una stringa come "mele, pere e banane" in singoli
    componenti.
    
1.  Perché è meglio dividere per `limite("parola")` che per `""`?

1.  Cosa fa la divisione con una stringa vuota (`""`)? Sperimentate, e
    poi leggete la documentazione.

### Trova le corrispondenze

`str_locate()` e `str_locate_all()` vi danno la posizione iniziale e finale di ogni corrispondenza. Queste sono particolarmente utili quando nessuna delle altre funzioni fa esattamente quello che vuoi. Puoi usare `str_locate()` per trovare il pattern corrispondente, `str_sub()` per estrarlo e/o modificarlo.

## Altri tipi di pattern

Quando si usa un pattern che è una stringa, esso viene automaticamente avvolto in una chiamata a `regex()`:


```r
# La chiamata regolare:
str_view(fruit, "nana")
# È l'abbreviazione di
str_view(fruit, regex("nana"))
```

Potete usare gli altri argomenti di `regex()` per controllare i dettagli della corrispondenza:

* `ignore_case = TRUE` permette ai caratteri di corrispondere sia alla loro forma maiuscola che a quella minuscola. Questo usa sempre il locale corrente.

    
    ```r
    bananas <- c("banana", "Banana", "BANANA")
    str_view(bananas, "banana")
    ```
    
    ```{=html}
    <div id="htmlwidget-b25b670b028f478bf741" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-b25b670b028f478bf741">{"x":{"html":"<ul>\n  <li><span class='match'>banana<\/span><\/li>\n  <li>Banana<\/li>\n  <li>BANANA<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
    str_view(bananas, regex("banana", ignore_case = TRUE))
    ```
    
    ```{=html}
    <div id="htmlwidget-46d1193f7ba074d981c8" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-46d1193f7ba074d981c8">{"x":{"html":"<ul>\n  <li><span class='match'>banana<\/span><\/li>\n  <li><span class='match'>Banana<\/span><\/li>\n  <li><span class='match'>BANANA<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
    ```
    
* `multiline = TRUE` permette a `^` e `$` di corrispondere all'inizio e alla fine di ogni linea piuttosto che all'inizio e alla fine della stringa completa.
    
    
    ```r
    x <- "Line 1\nLine 2\nLine 3"
    str_extract_all(x, "^Line")[[1]]
    #> [1] "Line"
    str_extract_all(x, regex("^Line", multiline = TRUE))[[1]]
    #> [1] "Line" "Line" "Line"
    ```
    
* `commenti = TRUE` ti permette di usare commenti e spazi bianchi per rendere le espressioni regolari complesse più comprensibili. Gli spazi sono ignorati, così come tutto ciò che viene dopo `#`. Per far corrispondere uno spazio letterale, dovrai fare l'escape: `"\"`.
    
    
    ```r
    phone <- regex("
      \\(? # parentesi di apertura opzionale
      (\\d{3}) # codice di zona
      [) -]?   # parentesi di chiusura opzionale, spazio o trattino
      (\\d{3}) # altri tre numeri
      [ -]?    # spazio o trattino opzionale
      (\\d{3}) # altri tre numeri
      ", comments = TRUE)
    
    str_match("514-791-8141", phone)
    #>      [,1]          [,2]  [,3]  [,4] 
    #> [1,] "514-791-814" "514" "791" "814"
    ```

* `dotall = TRUE` permette a `.` di corrispondere a tutto, incluso `\n`.

Ci sono altre tre funzioni che puoi usare al posto di `regex()`:

* `fixed()`: corrisponde esattamente alla sequenza di byte specificata. Ignora
    tutte le espressioni regolari speciali e opera ad un livello molto basso. 
    Questo permette di evitare complessi escaping e può essere molto più veloce delle 
    delle espressioni regolari. Il seguente microbenchmark mostra che è circa
    3 volte più veloce per un semplice esempio.
  
    
    ```r
    microbenchmark::microbenchmark(
      fixed = str_detect(sentences, fixed("the")),
      regex = str_detect(sentences, "the"),
      times = 20
    )
    #> Unit: microseconds
    #>   expr     min      lq     mean   median      uq     max neval
    #>  fixed 111.243 127.750 186.6002 143.6405 239.083 415.857    20
    #>  regex 350.396 373.981 483.7899 443.3895 557.511 929.115    20
    ```
    
    Attenzione all'uso di `fixed()` con dati non inglesi. È problematico perché ci sono spesso più modi di rappresentare lo stesso carattere. Per esempio, ci sono due modi per definire "á": o come un singolo carattere o come una "a" più un accento:
    
    
    ```r
    a1 <- "\u00e1"
    a2 <- "a\u0301"
    c(a1, a2)
    #> [1] "á" "á"
    a1 == a2
    #> [1] FALSE
    ```

    Essi rendono in modo identico, ma poiché sono definiti in modo diverso, `fixed()` non trova una corrispondenza. Invece, puoi usare `coll()`, definito in seguito, per rispettare le regole di confronto dei caratteri umani:
    
    ```r
    str_detect(a1, fixed(a2))
    #> [1] FALSE
    str_detect(a1, coll(a2))
    #> [1] TRUE
    ```
    
* `coll()`: confronta le stringhe usando le regole standard di **coll**azione. Questo è utile per fare confronti insensibili alle maiuscole e alle minuscole. Si noti che `coll()` accetta un parametro `locale` che controlla quali regole sono usate per confrontare i caratteri. Sfortunatamente le diverse parti del mondo usano regole diverse!

    
    ```r
    # Questo significa che devi anche essere consapevole della differenza
    # quando si fanno corrispondenze insensibili alle maiuscole e alle minuscole:
    i <- c("I", "İ", "i", "ı")
    i
    #> [1] "I" "İ" "i" "ı"
    
    str_subset(i, coll("i", ignore_case = TRUE))
    #> [1] "I" "i"
    str_subset(i, coll("i", ignore_case = TRUE, locale = "tr"))
    #> [1] "İ" "i"
    ```
    
    Sia `fixed()` che `regex()` hanno argomenti `ignore_case`, ma non vi permettono di scegliere il locale: usano sempre il locale di default. Potete vedere cos'è con il seguente codice; più avanti ci saranno altre stringhe.
    
    
    ```r
    stringi::stri_locale_info()
    #> $Language
    #> [1] "en"
    #> 
    #> $Country
    #> [1] "US"
    #> 
    #> $Variant
    #> [1] ""
    #> 
    #> $Name
    #> [1] "en_US"
    ```
    
    Lo svantaggio di `coll()` è la velocità; poiché le regole per riconoscere quali
    caratteri sono uguali sono complicate, `coll()` è relativamente lento
    rispetto a `regex()` e `fixed()`.

* Come avete visto con `str_split()` potete usare `boundary()` per abbinare i confini.
    Potete anche usarla con le altre funzioni: 
    
    
    ```r
    x <- "This is a sentence."
    str_view_all(x, boundary("word"))
    ```
    
    ```{=html}
    <div id="htmlwidget-382a200f56fb8e6a1fd3" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-382a200f56fb8e6a1fd3">{"x":{"html":"<ul>\n  <li><span class='match'>This<\/span> <span class='match'>is<\/span> <span class='match'>a<\/span> <span class='match'>sentence<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script>
    str_extract_all(x, boundary("word"))
    #> [[1]]
    #> [1] "This"     "is"       "a"        "sentence"
    ```

### Esercizi

1.  Come trovereste tutte le stringhe che contengono "regex" con "regex()` vs.
    con `fixed()`?

1.  2. Quali sono le cinque parole più comuni nelle `sentenze`?

## Altri usi delle espressioni regolari

Ci sono due utili funzioni in R base che usano anche le espressioni regolari:

* `apropos()` cerca tutti gli oggetti disponibili nell'ambiente globale. Questo
    è utile se non riuscite a ricordare il nome della funzione.
    
    
    
    ```r
    apropos("replace")
    #> [1] "%+replace%"       "replace"          "replace_na"       "setReplaceMethod"
    #> [5] "str_replace"      "str_replace_all"  "str_replace_na"   "theme_replace"
    ```
    
* `dir()` elenca tutti i file in una directory. L'argomento `pattern` prende
    un'espressione regolare e restituisce solo i nomi dei file che corrispondono allo schema.
    Per esempio, si possono trovare tutti i file R Markdown nella directory corrente
    con:
    
    
    ```r
    head(dir(pattern = "\\.Rmd$"))
    #> [1] "communicate-plots.Rmd" "communicate.Rmd"       "datetimes.Rmd"        
    #> [4] "EDA.Rmd"               "explore.Rmd"           "factors.Rmd"
    ```
    
    (Se siete più a vostro agio con i "globi" come `*.Rmd`, potete convertirli
    in espressioni regolari con `glob2rx()`):

## stringi

stringr è costruito sopra il pacchetto __stringi__. stringr è utile quando si sta imparando perché espone un insieme minimo di funzioni, che sono state accuratamente scelte per gestire le più comuni funzioni di manipolazione delle stringhe. stringi, d'altra parte, è progettato per essere completo. Contiene quasi tutte le funzioni di cui potreste aver bisogno: stringi ha funzioni 256 per stringr 49.

Se ti trovi a lottare per fare qualcosa in stringr, vale la pena dare un'occhiata a stringi. I pacchetti funzionano in modo molto simile, quindi dovresti essere in grado di tradurre la tua conoscenza di stringr in modo naturale. La differenza principale è il prefisso: `str_` contro `stri_`.

### Esercizi

1.  Trova le funzioni di stringi che:

    1. Conta il numero di parole.
    1. Trova le stringhe duplicate.
    1. Genera un testo casuale.

1.  Come controllate il linguaggio che `stri_sort()` utilizza per 
    l'ordinamento?
