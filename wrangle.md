# (PART) Wrangle {-}

# Introduzione {#wrangle-intro}

In questa parte del libro, imparerete il data wrangling, l'arte di portare i vostri dati in R in una forma utile per la visualizzazione e la modellazione. Il data wrangling è molto importante: senza di esso non potete lavorare con i vostri dati! Ci sono tre parti principali per il data wrangling:

<img src="diagrams/data-science-wrangle.png" width="75%" style="display: block; margin: auto;" />

Questa parte del libro procede come segue:

* In [tibble], imparerete la variante del data frame che usiamo in questo libro: il __tibble__.  Imparerete cosa li rende diversi dai normali data frame, e come potete costruirli "a mano".

* In [importazione dei dati], imparerete come prendere i vostri dati dal disco e inserirli in R. Ci concentreremo sui formati rettangolari di testo semplice, ma vi daremo dei riferimenti a pacchetti che aiutano con altri tipi di dati.

* In [tidy data], imparerete a conoscere i tidy data, un modo coerente di memorizzare i vostri dati che rende più facile la trasformazione, la visualizzazione e la modellazione. Imparerete i principi di base, e come ottenere i vostri dati in una forma ordinata.

Il data wrangling comprende anche la trasformazione dei dati, di cui avete già imparato qualcosa. Ora ci concentreremo sulle nuove competenze per tre tipi specifici di dati che incontrerete spesso nella pratica:

* [Dati relazionali] vi darà gli strumenti per lavorare con più insiemi di dati interconnessi.
    
* [Stringhe] introdurrà le espressioni regolari, un potente strumento per manipolare le stringhe.

* I [Fattori] sono il modo in cui R memorizza i dati categorici. Sono usati quando una variabile ha un insieme fisso di valori possibili, o quando si vuole usare un ordinamento non alfabetico di una stringa.
    
* [Date e tempi] vi darà gli strumenti chiave per lavorare con date e date-ora.
