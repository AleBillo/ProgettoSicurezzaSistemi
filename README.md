# Relazione del Progetto di "Sicurezza dei Sistemi e Privacy"
## introduzione
Il presente progetto si propone di implementare un processo di sviluppo sicuro (SSDLC - Secure Software Development Life Cycle) secondo l'approccio SecDevOps, attraverso la configurazione di una pipeline CI/CD per un'applicazione sviluppata in Java.
Il repository selezionato per l'attività è [onlinebookstore](https://github.com/shashirajraja/onlinebookstore), un progetto Java che simula un sistema di gestione per una libreria online.

Per la realizzazione della pipeline CI/CD sono stati adottati i seguenti strumenti e tecnologie: 
* GitHub Actions per l’automazione del processo CI/CD;
* Maven per la gestione del ciclo di vita del progetto e della fase di build;
* SonarQube per la scansione SAST (Static Application Security Testing) e la valutazione della qualità del codice;
* OWASP Dependency-Check per l’analisi SCA (Software Composition Analysis) delle librerie di terze parti utilizzate;
* Quality/Security Gate configurato per bloccare la pipeline in presenza di vulnerabilità critiche o violazioni gravi delle regole di qualità;
* Artifact archiving per il salvataggio dell’output di build (JAR file);
* Sistema di notifica per segnalare l’esito delle analisi al team di sicurezza (GitHub Actions).

L’obiettivo principale è dimostrare come l'integrazione di strumenti di sicurezza nel ciclo di vita del software possa aumentare l'affidabilità e la resilienza delle applicazioni, promuovendo al contempo pratiche di sviluppo sicuro.

## Configurazione dell'ambiente di sviluppo
* ### Step 1 (Stage 1-2-6)
Checkout del codice cioè clonare la repository per partire dall'ultimo commit ergo dallo stato corrente.
Avvio del servizio: Crea un secondo container con MySQL e fa partire il suo servizio.
Setup di Java 11 e ping di MySQL e quando parte si inizializza il database con dei dati di testo.
Build con Maven e carica l'eseguibile (Artifact) in modo da allegarlo all'output finale.
* ### Step 2 (Stage 3)
Effettua una scansione SAST con SonarQube per l'analisi del codice statico.
* ### Step 3 (Stage 4)
Effettua una scansione SCA con Dependecy Check e allega il report delle dipendenze all'output.

* ### Stage 5
Per quanto riguarda i check dei gate di sicurezza e qualità.
* ### Stage 7
Notifica del report integrata automaticamente con GitHub Actions.

## Analisi delle vulnerabilità trovate e soluzioni


### Membri del gruppo:
- Jacopo Maria Spitaleri
- Alessandro Dominici
