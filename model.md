# (PART) Model {-}

# Introduzione {#model-intro}

Ora che siete dotati di potenti strumenti di programmazione possiamo finalmente tornare alla modellazione. Userete i vostri nuovi strumenti di gestione dei dati e di programmazione per adattare molti modelli e capire come funzionano. Il focus di questo libro è l'esplorazione, non la conferma o l'inferenza formale. Ma imparerete alcuni strumenti di base che vi aiuteranno a capire la variazione all'interno dei vostri modelli.

<img src="diagrams/data-science-model.png" width="75%" style="display: block; margin: auto;" />

L'obiettivo di un modello è quello di fornire un semplice riassunto a bassa dimesionalità di un set di dati. Idealmente, il modello catturerà i veri "segnali" (cioè i modelli generati dal fenomeno di interesse), e ignorerà il "rumore" (cioè la variazione casuale che non vi interessa). Qui ci occupiamo solo dei modelli "predittivi", che, come suggerisce il nome, generano previsioni. C'è un altro tipo di modello che non discuteremo: I modelli "data discovery". Questi modelli non fanno previsioni, ma vi aiutano a scoprire relazioni interessanti all'interno dei vostri dati. (Queste due categorie di modelli sono talvolta chiamate supervisionate e non supervisionate, ma non credo che questa terminologia sia particolarmente illuminante).

Questo libro non vi darà una profonda comprensione della teoria matematica che sta alla base dei modelli. Tuttavia, costruirà la vostra intuizione su come funzionano i modelli statistici e vi darà una famiglia di strumenti utili che vi permetteranno di usare i modelli per capire meglio i vostri dati:

* In [modelli base], imparerete come funzionano meccanicamente i modelli, concentrandovi su
  l'importante famiglia dei modelli lineari. Imparerete strumenti generali per ottenere
  comprensione di ciò che un modello predittivo ti dice sui tuoi dati, concentrandoti su
  semplici insiemi di dati simulati.

* In [costruire modelli], imparerete come usare i modelli per estrarre modelli noti
  modelli nei dati reali. Una volta riconosciuto un modello importante
  è utile renderlo esplicito in un modello, perché allora si possono
  più facilmente vedere i segnali più sottili che rimangono.

* In [molti modelli], imparerai come usare molti modelli semplici per aiutare 
  comprendere insiemi di dati complessi. Questa è una tecnica potente, ma per accedervi
  ma per accedervi è necessario combinare strumenti di modellazione e programmazione.

Questi argomenti sono notevoli per quello che non includono: qualsiasi strumento per valutare quantitativamente i modelli. Questo è deliberato: quantificare precisamente un modello richiede un paio di grandi idee che non abbiamo lo spazio per coprire qui. Per ora, farete affidamento sulla valutazione qualitativa e sul vostro naturale scetticismo. In [Imparare di più sui modelli], vi indicheremo altre risorse dove potrete imparare di più.

## Generazione di ipotesi vs. conferma di ipotesi

In questo libro, useremo i modelli come strumento di esplorazione, completando la triade degli strumenti per l'analisi esplorativa dei dati (EDA) che sono stati introdotti nella Parte 1. Questo non è il modo in cui i modelli vengono solitamente insegnati, ma come vedrete, i modelli sono uno strumento importante per l'esplorazione. Tradizionalmente, il focus della modellazione è sull'inferenza, o per confermare che un'ipotesi è vera. Fare questo correttamente non è complicato, ma è difficile. Ci sono un paio di idee che dovete capire per fare l'inferenza correttamente:

1. Ogni osservazione può essere usata o per l'esplorazione o per la conferma, 
   non per entrambe.

1. Puoi usare un'osservazione quante volte vuoi per l'esplorazione,
   ma puoi usarla solo una volta per la conferma. Non appena usi un'osservazione 
   osservazione due volte, si è passati dalla conferma all'esplorazione.
   
Questo è necessario perché per confermare un'ipotesi dovete usare dati indipendenti dai dati che avete usato per generare l'ipotesi. Altrimenti sarete troppo ottimisti. Non c'è assolutamente nulla di sbagliato nell'esplorazione, ma non dovreste mai vendere un'analisi esplorativa come un'analisi di conferma perché è fondamentalmente fuorviante. 

Se siete seriamente intenzionati a fare un'analisi confermativa, un approccio è quello di dividere i vostri dati in tre parti prima di iniziare l'analisi:

1.  Il 60% dei vostri dati va in un __training__ (o set di esplorazione). Siete autorizzati a 
    permesso di fare tutto quello che vuoi con questi dati: visualizzarli e adattarvi tonnellate 
    di modelli.
  
1. Il 20% va in un set di __query__. Puoi usare questi dati per confrontare i modelli 
    o visualizzazioni a mano, ma non ti è permesso usarli come parte di
    un processo automatizzato.

1. Il 20% viene trattenuto per un __test__ set. Puoi usare questi dati solo UNA VOLTA, per 
    testare il vostro modello finale. 
    
Questo partizionamento vi permette di esplorare i dati di addestramento, generando occasionalmente ipotesi candidate che controllate con il set di query. Quando siete sicuri di avere il modello giusto, potete controllarlo una volta con i dati di test.

(Si noti che anche quando si fa la modellazione confermativa, è necessario fare l'EDA. Se non fate nessuna EDA rimarrete ciechi ai problemi di qualità dei vostri dati).
