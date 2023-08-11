### CMake

L'uso di CMake in un progetto in C non è obbligatorio, ma può essere estremamente utile e consigliato, soprattutto per progetti più grandi o complessi. CMake è uno strumento di generazione di build che semplifica il processo di compilazione e gestione del progetto, garantendo che il codice possa essere compilato correttamente su diverse piattaforme e ambienti.

Ecco alcune ragioni per cui potresti voler considerare l'uso di CMake nel tuo progetto C:

1. **portabilità**: CMake ti consente di definire in modo dichiarativo le dipendenze, le opzioni di compilazione e altre impostazioni, il che facilita la portabilità del tuo codice su diverse piattaforme e compilatori;

2. **struttura organizzata**: CMake ti permette di organizzare il tuo progetto in una struttura gerarchica, suddividendo il codice sorgente in moduli, librerie e eseguibili. Questo rende più facile la gestione dei componenti del progetto;

3. **generazione di build personalizzate**: CMake supporta la generazione di build personalizzate, tra cui makefiles, progetti per IDE come Visual Studio o Xcode, e altri sistemi di build. Ciò consente ai membri del team di utilizzare l'IDE o il sistema di build preferito;
   
4. **facilità di configurazione**: CMake semplifica la configurazione del progetto e l'impostazione di variabili, opzioni di compilazione e definizioni pre-processore;

5. **aggiornamenti semplificati: quando devi apportare modifiche al progetto, apportare modifiche al sistema di build tramite CMake può rendere il processo di aggiornamento più agevole e meno soggetto a errori;

6. **gestione delle dipendenze**: CMake offre supporto per la gestione delle dipendenze esterne, come librerie di terze parti, semplificando l'aggiunta e l'uso di tali librerie nel tuo progetto;

7. **build Out-of-Source**: CMake supporta la creazione di build "out-of-source", il che significa che la directory di build è separata dalla directory del codice sorgente. Questo aiuta a mantenere pulita la directory del codice sorgente e semplifica la gestione delle build.
