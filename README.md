# APPUNTI SPARSI SU ALFRESCO


## INTRODUZIONE

Alfresco, come azienda, fornisce diversi prodottti. Tra i più noti, vi sono:
- *Alfresco Content Services (ACS)*, che è un sistema di *Enterprise Content Management (ECM)*, e
- *Alfresco Process Services (APS)*, che è un sistema di *Business Process Management (BPM)*.  

Nel seguito, salvo dove indicato altrimenti, con "Alfresco" si farà sempre riferimento ad Alfresco Content Services.  

Alfresco Content Services consta di due componenti principali, Share e Platform:
- *Share* rappresenta il front-end di ACS, e corrisponde all'URL \<alfrescoHost\>/share
- *Platform* rappresenta il back-end di ACS, e corrisponde all'URL \<alfrescoHost\>/alfresco  


## SVILUPPO - GENERALIA

Gli strumenti usati prevalentemente in Sourcesense per lo sviluppo su Alfresco sono i seguenti:
- Maven + XML + Java 1.8 per il back-end,
- Angular per il front-end.

Ci sono due modalità di lavoro con Alfresco:
1. con l'SDK, attraverso l'IDE di preferenza, usando strumenti di versionamento e deploy
2. direttamente nel sito Alfresco installato, senza SDK: questa modalità è suggerita per modifiche veloci, si usa JavaScript e non sono necessari né un vero e proprio deploy, né il riavvio del server Alfresco

Per lanciare il server Alfresco C.E. (Community Edition) tramite SDK, posizionarsin nella cartella del progetto Maven/Eclipse (all-in-one) e lanciare:
> ./run.sh

Può essere necessario un **chmod 755** dei **.sh**, qualora appena scaricati non risultino eseguibili.
Il server Alfresco dell'SDK rischia però di essere estremamente lento, ai limiti dell'usabilità.  


## INSTALLAZIONE MANUALE

Tutte le verioni più recenti di Alfresco Content Manager non prevedono un installatore, ma solo:
- l'installazione su Docker (che su un Mac offre scarse prestazioni, e comunque rogna su un modulo mancante)
- l'installazione manuale, per la quale sono richiesti svariati giorni, un rosario di maledizioni e un valido antiacido. E' l'installazione d'obbligo presso i clienti.

Alfresco Content Services richiede, come prerequisito per l'installazione e/o come condizione per il proprio corretto funzionamento, l'installazione di altre applicazioni, nelle versioni dichiarate compatibili sulla documentazione ufficiale; si veda:
<https://docs.alfresco.com/6.0/concepts/supported-platforms-ACS.html>

Le applicazioni in questione sono, come indicato nel link precedente:
- Java 1.8 (OpenJDK 11.0.1 oppure Oracle JDK 1.8.0_161 - io ho usato Oracle JDK 1.8.0_251 e non ho avuto problemi)
- Ghostscript
- ImageMagick 7.0.7
- LibreOffice 5.4.6
- PostgreSQL 10.1 (il cui relativo JDBC driver è Postgresql-42.2.1.jar) - da preferire agli altri RDBMS supportati per l'installazione di sviluppo

Per l'installare manualmente Alfresco C.E. 6.0.7 su un MacBook Pro con Catalina OS ho effettuato i seguenti passaggi, sfruttando uno script reperito nell'hub di Alfresco e adattato per le mie esigenze.  

### 1  - INSTALLAZIONE DI JAVA 1.8

Ho scaricato e installato l'ultimo JDK di Java 8 (JDK 1.8.0_251) dal sito Oracle.  

Ho aggiunto la variabile d'ambiente **JAVA_HOME** (per verificarne la presenza/assenza usare il comando **printenv** da terminale), valorizzandola con il percorso della cartella Contents/Home della propria installazione Java: vedere sotto */Library/Java/JavaVirtualMachines/*.
Ipotizziamo che, come nel mio caso, questo percorso sia:
> /Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home  

Per impostare JAVA_HOME sotto **zsh**, aggiungere nel file **.zprofile** (creandolo se manca) della propria home directory questa riga:
> export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home  

Se invece si usa la Bash shell (**bash**), la stessa riga va aggiunta al file **.bash_profile** (creandolo se manca), sempre nella propria home directory.  

### 2 - INSTALLAZIONE DI GHOSTSCRIPT

Ho installato GhostScript da terminale, tramite [Homebrew](http://mxcl.github.com/homebrew/):
> brew install ghostscript  

### 3 - INSTALLAZIONE DI IMAGE MAGICK

Viene usato da Alfresco per l'anteprima delle immagini.
Ho installato ImageMagick da terminale, tramite [Homebrew](http://mxcl.github.com/homebrew/):
> brew install imagemagick  

### 4 - INSTALLAZIONE DI LIBRE OFFICE

Si vedano le istruzioni sul sito di LibreOffice: 
<https://www.libreoffice.org/get-help/install-howto/macos/>  

### 5 - INSTALLAZIONE DI POSTGRESQL

Nella documentazione ufficiale Alfresco, la versione 6.0.7 di Alfresco Content Service è dichiarata compatibile con la versione 10.1 di PostgreSQL.
Io ho installato la versione 12, e - per il momento - non ho avuto problemi.
Seguire ad esempio le istruzioni riportate qui:
<http://databasemaster.it/installare-postgresql-mac/>  

Una volta installato Postgres, consiglio di aprire una console **pgsql** da */Applications/PostgreSQL 12* e di creare un nuovo DB ("alfresco"), appartenente all'utente root default "postgres", e una nuova utenza abilitata a tale DB avente nome (e password) "alfresco":
> CREATE USER alfresco WITH PASSWORD 'alfresco';  
> CREATE DATABASE alfresco OWNER postgres ENCODING 'UTF8';  
> GRANT ALL PRIVILEGES ON DATABASE alfresco TO alfresco;  

TODO: verificare il paragrafo che segue: l'utenza viene creata automaticamente da Postgres, o dallo script?
**ATTENZIONE:** L'utenza di S.O. che viene creata automaticamente a seguito di quella PostgreSQL, nel mio caso aveva una password diversa da "alfresco": ero dovuta andare su *Mela > Preferenze di sistema... > Utenti e Gruppi* per modificarne la password.

### 6 - INSTALLAZIONE DI ALFRESCO CONTENT SERVICES

Le istruzioni che seguono sono la copia, tradotta e leggermente modificata, del seguente post sull'hub di Alfresco:
<http://databasemaster.it/installare-postgresql-mac/>  
Lo script che utilizzeremo non sarà quello presente su tale sito, ma una sua copia modificata, reperibile qui:
<???>

Per usare lo script è necessario prima scaricare i seguenti file:
- [Tomcat 8.5.37](https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.zip)
- [Alfresco Content Services 6.0.7-ga](https://artifacts.alfresco.com/nexus/content/repositories/public/org/alfresco/alfresco-content-services-community-distribution/6.0.7-ga/alfresco-content-services-community-distribution-6.0.7-ga.zip)
- [Alfresco Search Services 1.3.0](https://artifacts.alfresco.com/nexus/content/repositories/public/org/alfresco/alfresco-search-services/1.3.0/alfresco-search-services-1.3.0.zip)

Si consiglia di scaricare lo script e i tre file precedenti in una apposita cartella, ad esempio */Applications/alfresco*:  
> mkdir /Applications/alfresco  
> cd /Applications/alfresco  

sotto la quale creare una nuova cartella, vuota, che andrà a contenere la nostra installazione Alfresco...  
> mkdir 6.0.7-ga  

... e in cui ci posizioneremo per eseguire lo script:  
> cd 6.0.7-ga  
> ../install-alfresco-6x-ss.sh  
 
### AVVIARE E FERMARE I SERVER

A questo punto, abbiamo a disposizione tre server, nel microcosmo di Alfresco:
- il server Postgres
- il server Alfresco/Tomcat
- il server Solr

Il server Postgres deve essere sempre avviato *prima* di quelli Alfresco e Solr, e va fermato *dopo*.  
I server Alfresco e Solr possono invece essere avviati in qualsiasi ordine, purché dopo il server Postgres.  

Per avviare e fermare il server Postgres:
> cd /Library/PostgreSQL/12     # da sostituire col percorso corretto per la propria installazione Postgres  
> su postgres                   # verrà richiesta la password, "postgres"
> bin/pg_ctl -D data start
> bin/pg_ctl -D data stop

Per avviare e fermare il server Alfresco/Tomcat:
> cd /Applications/alfresco/6.0.7-ga  
> ./alfresco.sh start  
> ./alfresco.sh jpda start      # avvio in modalità debug  
> ./alfresco.sh stop  

Per avviare e fermare il server Solr 6:
> cd /Applications/alfresco/6.0.7-ga  
> search-services/solr/bin/solr start  
> search-services/solr/bin/solr stop  


## WEB SCRIPT

Il *Web Script Framework (WSF)* di Alfresco prevede che i template delle risposte degli script siano in *FreeMarker* (vedi sezione apposita): l'estensione **.ftl** usata sta infatti per "FreeMarker TempLate".  


### FREEMARKER

*FreeMarker* è un motore di template basato su Java (cfr. JSP e Thymeleaf).
Per una veloce introduzione a FreeMarker, si veda: 
<https://freemarker.apache.org/docs/dgui_quickstart_template.html>  

Per provare FreeMarker in versione stand-alone, senza passare da Alfresco, si può far riferimento a:  
<https://www.vogella.com/tutorials/FreeMarker/article.html>  
o a:  
<https://www.html.it/articoli/freemarker-template-engine-basato-su-java/>  




