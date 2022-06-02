# R Markdown workflow

In precedenza, abbiamo discusso un flusso di lavoro di base per la cattura del codice R in cui si lavora interattivamente nella _console_, quindi si cattura ciò che funziona nell'_editor di script_. R Markdown riunisce la console e l'editor di script, attenuando i confini tra l'esplorazione interattiva e la cattura del codice a lungo termine. È possibile iterare rapidamente all'interno di un pezzo, modificando e rieseguendo con Cmd/Ctrl + Maiusc + Invio. Quando si è soddisfatti, si passa a un nuovo pezzo.

R Markdown è importante anche perché integra strettamente prosa e codice. Questo lo rende un ottimo __quaderno di analisi__ perché consente di sviluppare codice e registrare i propri pensieri. Un quaderno di analisi condivide molti degli stessi obiettivi di un classico quaderno di laboratorio nelle scienze fisiche. Esso:

* Registra ciò che avete fatto e perché lo avete fatto. Indipendentemente da quanto sia grande la vostra
    memoria, se non si registra quello che si fa, arriverà un momento in cui
    dimenticarsi di dettagli importanti. Scriveteli per non dimenticarli!

* Supporta il pensiero rigoroso. È più probabile che si arrivi a un'analisi forte
    analisi forte se si registrano i propri pensieri e si continua a riflettere su di essi.
    su di essi. In questo modo si risparmia tempo quando si scrive l'analisi per condividerla con gli altri.
    analisi da condividere con gli altri.

* Aiuta gli altri a capire il vostro lavoro. È raro che l'analisi dei dati venga svolta da
    da soli e spesso si lavora in gruppo. Un quaderno di laboratorio
    aiuta a condividere con i colleghi o i compagni di laboratorio non solo ciò che si è fatto, ma anche il motivo per cui lo si è fatto.
    colleghi o compagni di laboratorio.

Molti dei buoni consigli sull'uso efficace dei quaderni di laboratorio possono essere applicati anche ai quaderni di analisi. Ho attinto alle mie esperienze personali e ai consigli di Colin Purrington sui taccuini di laboratorio (<http://colinpurrington.com/tips/lab-notebooks>) per trovare i seguenti suggerimenti:

* Assicurarsi che ogni quaderno abbia un titolo descrittivo, un nome di file evocativo e un
    un primo paragrafo che descriva brevemente gli obiettivi dell'analisi.

* Usare il campo data dell'intestazione YAML per registrare la data in cui si è cominciato a lavorare sul
    taccuino:

    ```yaml
    data: 2016-08-23
    ```

    Usare il formato ISO8601 YYYY-MM-DD per evitare ambiguità. Usarlo
    anche se normalmente non si scrivono le date in questo modo!

* Se si dedica molto tempo a un'idea di analisi e questa si rivela un vicolo cieco, non cancellarla.
    un vicolo cieco, non cancellatela! Scrivete una breve nota sul motivo del fallimento e lasciatela nel quaderno.
    e lasciatela nel quaderno. Questo vi aiuterà a evitare di imboccare lo stesso
    vicolo cieco quando si tornerà all'analisi in futuro.

* In generale, è meglio fare l'inserimento dei dati al di fuori di R. Ma se si ha la necessità di registrare un piccolo frammento di dati. 
    di registrare un piccolo frammento di dati, è consigliabile definirlo in modo chiaro usando
    `tibble::tribble()`.

* Se si scopre un errore in un file di dati, non modificarlo mai direttamente, ma scrivere codice per correggere il valore.
    invece scrivere del codice per correggere il valore. Spiegate perché avete fatto la correzione.

* Prima di terminare la giornata, assicurarsi di poter collegare il blocco note
    (se si utilizza la cache, assicurarsi di cancellare la cache). In questo modo
    Questo vi permetterà di risolvere eventuali problemi mentre il codice è ancora fresco nella vostra mente.

* Se volete che il vostro codice sia riproducibile a lungo termine (cioè in modo da poter tornare a eseguirlo il mese prossimo o il giorno dopo).
    tornare a eseguirlo il mese prossimo o l'anno prossimo), è necessario tenere traccia delle versioni dei pacchetti
    delle versioni dei pacchetti che il codice utilizza. Un approccio rigoroso è quello di usare
    __packrat__, <http://rstudio.github.io/packrat/>, che memorizza i pacchetti 
    nella cartella del progetto, oppure __checkpoint__,
    <https://github.com/RevolutionAnalytics/checkpoint>, che reinstallerà i pacchetti
    disponibili in una data specifica. Un trucco veloce e sporco è quello di includere
    un pezzo che esegue `sessionInfo()` --- che non permette di ricreare facilmente 
    i pacchetti come sono oggi, ma almeno si saprà quali erano.

* Nel corso della vostra carriera creerete molti, molti, molti quaderni di analisi.
    della vostra carriera. Come li organizzerete per poterli ritrovare in futuro?
    in futuro? Vi consiglio di archiviarli in progetti individuali,
    e di trovare un buon schema di denominazione.
