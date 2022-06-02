# (PART) Programmare {-}

# Introduzione {#program-intro}

In questa parte del libro, migliorerai le tue abilità di programmazione. La programmazione è un'abilità trasversale necessaria per tutto il lavoro di scienza dei dati: dovete usare un computer per fare scienza dei dati; non potete farlo nella vostra testa, o con carta e penna.

<img src="diagrams/data-science-program.png" width="75%" style="display: block; margin: auto;" />

La programmazione produce codice, e il codice è uno strumento di comunicazione. Ovviamente il codice dice al computer cosa si vuole che faccia. Ma comunica anche il significato agli altri esseri umani. Pensare al codice come veicolo di comunicazione è importante perché ogni progetto che fate è fondamentalmente collaborativo. Anche se non stai lavorando con altre persone, stai sicuramente lavorando con il futuro! Scrivere codice chiaro è importante in modo che gli altri (come i futuri voi) possano capire perché avete affrontato un'analisi nel modo in cui l'avete fatto. Ciò significa che migliorare nella programmazione significa anche migliorare nella comunicazione. Nel tempo, volete che il vostro codice diventi non solo più facile da scrivere, ma più facile da leggere per gli altri. 

Scrivere codice è simile in molti modi allo scrivere in prosa. Un parallelo che trovo particolarmente utile è che in entrambi i casi la riscrittura è la chiave per la chiarezza. La prima espressione delle vostre idee difficilmente sarà particolarmente chiara, e potreste aver bisogno di riscrivere più volte. Dopo aver risolto una sfida di analisi dei dati, spesso vale la pena guardare il tuo codice e pensare se è ovvio o meno quello che hai fatto. Se passate un po' di tempo a riscrivere il vostro codice mentre le idee sono fresche, potete risparmiare un sacco di tempo più tardi cercando di ricreare ciò che il vostro codice ha fatto. Ma questo non significa che dovreste riscrivere ogni funzione: dovete bilanciare ciò che dovete ottenere ora con il risparmio di tempo nel lungo periodo. (Ma più riscrivete le vostre funzioni e più è probabile che il vostro primo tentativo sia chiaro).

Nei seguenti quattro capitoli, imparerete abilità che vi permetteranno sia di affrontare nuovi programmi che di risolvere problemi esistenti con maggiore chiarezza e facilità: 

1.  In [pipes], vi immergerete in profondità nel __pipe__, `%>%`, e imparerete di più 
    su come funziona, quali sono le alternative e quando non usarlo.

1.  Il copia-e-incolla è uno strumento potente, ma si dovrebbe evitare di farlo più di
    due volte. Ripetere se stessi nel codice è pericoloso perché può facilmente portare 
    errori e incongruenze. Invece, in [funzioni], imparerete
    come scrivere __funzioni__ che permettono di estrarre il codice ripetuto in modo che 
    possa essere facilmente riutilizzato.

1.  Quando inizierete a scrivere funzioni più potenti, avrete bisogno di una solida
    una solida base nelle __strutture di dati__ di R, fornite da [vettori]. Dovete padroneggiare 
    i quattro vettori atomici comuni, le tre importanti classi S3 costruite 
    sopra di loro, e capire i misteri della lista e del data frame. 

1.  Le funzioni estraggono il codice ripetuto, ma spesso è necessario ripetere le
    stesse azioni su input diversi. Avete bisogno di strumenti per l'__iterazione__ che
    vi permettano di fare cose simili più e più volte. Questi strumenti includono i cicli for 
    e la programmazione funzionale, che imparerete in [iterazione].

## Per saperne di più

L'obiettivo di questi capitoli è di insegnarvi il minimo di programmazione di cui avete bisogno per praticare la scienza dei dati, che risulta essere una quantità ragionevole. Una volta che avete imparato il materiale in questo libro, credo fortemente che dovreste investire ulteriormente nelle vostre capacità di programmazione. Imparare di più sulla programmazione è un investimento a lungo termine: non vi ripagherà immediatamente, ma a lungo termine vi permetterà di risolvere nuovi problemi più rapidamente, e vi permetterà di riutilizzare le vostre intuizioni dai problemi precedenti in nuovi scenari.

Per imparare di più è necessario studiare R come un linguaggio di programmazione, non solo un ambiente interattivo per la scienza dei dati. Abbiamo scritto due libri che vi aiuteranno a farlo:

* [_Hands on Programming with R_](https://amzn.com/1449359019),
  di Garrett Grolemund. Questa è un'introduzione a R come linguaggio di programmazione 
  ed è un ottimo punto di partenza se R è il vostro primo linguaggio di programmazione. Esso 
  copre materiale simile a questi capitoli, ma con uno stile diverso e
  diversi esempi di motivazione (basati sul casinò). È un utile complemento 
  se trovate che questi quattro capitoli passano troppo in fretta.
  
* [_Advanced R_](https://amzn.com/1466586966) di Hadley Wickham. Questo si immerge nei
  dettagli del linguaggio di programmazione R. Questo è un ottimo punto di partenza se
  avete già esperienza di programmazione. È anche un ottimo passo successivo una volta che avete 
  interiorizzato le idee in questi capitoli. Potete leggerlo online su
  <http://adv-r.had.co.nz>.
