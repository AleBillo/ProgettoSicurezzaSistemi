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
Abbiamo utilizzato un container di GitHub Actions per contenere la pipeline

## Analisi delle vulnerabilità trovate e soluzioni


### Membri del gruppo:
- Jacopo Maria Spitaleri
- Alessandro Dominici
