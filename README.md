# Relazione del Progetto di "Sicurezza dei Sistemi e Privacy"
## introduzione
Il presente progetto si propone di implementare un processo di sviluppo sicuro (SSDLC - Secure Software Development Life Cycle) secondo l'approccio SecDevOps, attraverso la configurazione di una pipeline CI/CD per un'applicazione sviluppata in Java.
Il repository selezionato per l'attività è [onlinebookstore](https://github.com/shashirajraja/onlinebookstore), un progetto Java che simula un sistema di gestione per una libreria online..

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

### Step 1 (Stage 1-2-6)
- Checkout del codice - viene clonato l'ultimo commit del repository, su cui lavora la pipeline
  ```yaml
  checkout:
    name: Checkout code
    runs-on: ubuntu-latest
    outputs:
      code-dir: ${{ steps.checkout.outputs.path }}
    steps:
      - name: Checkout code
        id: checkout
        uses: actions/checkout@v3
  ```
- Avvio del servizio MySQL - crea un secondo container con MySQL e avvia il servizio in background.
 ```yaml
services:
   mysql:
   image: mysql:5.7
      env:
        MYSQL_ROOT_PASSWORD: root
        MYSQL_DATABASE: onlinebookstore
      ports:
        - 3306:3306
      options: >-
        --health-cmd="mysqladmin ping -h 127.0.0.1 --silent"
        --health-interval=10s
        --health-timeout=5s
        --health-retries=5
```
- Setup di Java SDK (versione 17, distribuzione temurin)
  ```yaml
  name: Set up JDK 17 (Temurin)
    uses: actions/setup-java@v3
    with:
      java-version: 17
      distribution: temurin
  ```
- Attesa dell'avvio del servizio di di MySQL (tramite ping) e nizializzazione del database con dei valori di test.
```yaml
- name: Wait for MySQL to be ready
        run: |
          for i in {1..30}; do
          if mysqladmin ping -h127.0.0.1 -uroot -proot --silent; then
          echo "MySQL is ready!"
          break
          fi
          echo "Waiting for MySQL..."
          sleep 2
          done

      - name: Initialize database
        run: |
          mysql -h127.0.0.1 -uroot -proot <<EOF
          USE onlinebookstore;
          CREATE TABLE IF NOT EXISTS books(
          barcode VARCHAR(100) PRIMARY KEY,
          name VARCHAR(100),
          author VARCHAR(100),
          price INT,
          quantity INT
          );
          CREATE TABLE IF NOT EXISTS users(
          username VARCHAR(100) PRIMARY KEY,
          password VARCHAR(100),
          firstname VARCHAR(100),
          lastname VARCHAR(100),
          address TEXT,
          phone VARCHAR(100),
          mailid VARCHAR(100),
          usertype INT
          );
          INSERT INTO books VALUES(
          '9780134190563',
          'The Go Programming Language',
          'Alan A. A. Donovan and Brian W. Kernighan',
          400,
          8
          );
          INSERT INTO users VALUES(
          'shashi',
          'shashi',
          'Shashi',
          'Raj',
          'Bihar',
          '1236547089',
          'shashi@gmail.com',
          2
          );
          COMMIT;
          EOF

```
- Build con Maven e upload dell'artefatto eseguibile (`onlinebookstore.war`) in modo da allegarlo all'output finale.
```yaml
 - name: Build (Maven)
        run: mvn clean install -B

      - name: Upload JAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-artifact
          path: target/*.war
```

### Step 2 (Stage 3)
Effettua una scansione SAST con SonarQube per l'analisi statica del codice.
```yaml
 sast:
    name: Static Application Security Testing (SAST)
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up JDK 17 (Temurin)
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: temurin

      - name: Cache SonarQube packages
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar

      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2

      - name: Run SonarQube Scanner
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=jaluspeet_onlinebookstore
```

### Step 3 (Stage 4)
Effettua una scansione SCA con Dependecy Check e allega il report delle vulnerabilità (`dependency-check-report.html`) all'output.
```yaml
sca:
    name: Software Composition Analysis (SCA)
    runs-on: ubuntu-latest
    needs: sast
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Build project with Maven
        run: mvn clean install -B

      - name: Run Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        continue-on-error: true
        env:
          JAVA_HOME: /opt/jdk
        with:
          project: onlinebookstore
          path: '.'
          format: HTML
          out: reports
          args: >
            --failOnCVSS 7
            --enableRetired

      - name: Upload Dependency Check report
        uses: actions/upload-artifact@v4
        with:
          name: Dependency-Check-Report
          path: reports
```

### Stage 5
La pipeline viene automaticamente interrotta se uno dei due scan (SCA e SAST) fallisce. La causa del fallimento viene indicata nell'output del workflow di Github Actions, accessibile ad un eventuale team di sicurezza. Qui sotto è riportato un esempio di esito positivo della pipeline
![Esempio di build](images/gate.png)

### Stage 7
Viene automaticamente notificato tramite email un eventuale team di sicurezza, sfruttando la reportistica automatizzata di Github Actions.

## Analisi delle vulnerabilità trovate e soluzioni


### Membri del gruppo:
- Jacopo Maria Spitaleri
- Alessandro Dominici
