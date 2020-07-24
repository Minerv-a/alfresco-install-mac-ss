# alfresco-install-mac-ss

Ovvero: come effettuare un'installazione di sviluppo di **Alfresco Content Services (ACS)** su MacOS (Catalina) senza farsi venire un'ulcera.

**N.B.** Nel seguito, salvo dove indicato altrimenti, con "Alfresco" si farà sempre riferimento ad Alfresco Content Services.  


## PREMESSA

Tutte le versioni più recenti di Alfresco Content Manager non prevedono un installatore, ma solo:
- l'installazione su Docker (che su un Mac pare offra scarse prestazioni, e comunque nel mio caso rognava su un modulo mancante)
- l'installazione manuale, per la quale sono richiesti svariati giorni, un rosario di maledizioni e un valido antiacido.  

Lo script fornito a corredo di questo documento serve per automatizzare parzialmente l'installazione "manuale" di ACS 6.0.7, ed è una versione leggermente modificata di quello creato da [michel_chen_ri](https://hub.alfresco.com/t5/user/viewprofilepage/user-id/16578), descritto e disponibile sull'[hub di Alfresco](https://hub.alfresco.com/t5/alfresco-content-services-blog/a-script-to-install-alfresco-community-6-0/ba-p/288988).  

Nella fattispecie, lo script originario è stato semplicemente adattato per:
- utilizzare **PostgreSQL** al posto di MySQL
- modificare le impostazioni di memoria per la Java Virtual Machine nel **setenv.sh** di Tomcat
- rimuovere un riferimento al fuso orario (TimeZone) "PST" durante la generazione di **alfresco.sh**, che con le versioni di Postgres successive alla 8 darebbe errore.  

Lo script **install-alfresco-6x-ss.sh** procede a installare Alfresco Content Services 6.0.7 su Catalina OS, una volta che siano stati soddisfatti alcuni prerequisiti.  


## PREREQUISITI

Alfresco Content Services richiede, come prerequisito per l'installazione e/o come condizione per il proprio corretto funzionamento, l'installazione preliminare di altre applicazioni.    

Le applicazioni in questione, nelle loro versioni dichiarate [compatibili con ACS 6.0.7](https://docs.alfresco.com/6.0/concepts/supported-platforms-ACS.html), sono:  
- **Java 1.8** (OpenJDK 11.0.1 oppure Oracle JDK 1.8.0_161 - io ho usato Oracle JDK 1.8.0_251 e non ho avuto problemi)
- **Ghostscript** (versione non fornita, io ho usato quella su *Homebrew* al 21 luglio 2020 e pare funzionare)
- **ImageMagick 7.0.7**
- **LibreOffice 5.4.6**
- **PostgreSQL 10.1** (il cui relativo JDBC driver è **Postgresql-42.2.1.jar**) 

Prima di installare Alfresco C.E. 6.0.7 ho quindi effettuato i seguenti passaggi.  


### 1  - INSTALLAZIONE DI JAVA 1.8

Ho scaricato e installato l'ultimo JDK di Java 8 (JDK 1.8.0_251) dal sito Oracle.  

Ho aggiunto la variabile d'ambiente **JAVA_HOME** (per verificarne la presenza/assenza usare il comando **printenv** da terminale), valorizzandola con il percorso della cartella *Contents/Home* della propria installazione Java: vedere sotto */Library/Java/JavaVirtualMachines/*.  
Ipotizziamo che, come nel mio caso, questo percorso sia:
> /Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home  

Per impostare **JAVA_HOME** sotto **zsh**, aggiungere nel file **.zprofile** (creandolo se manca) della propria home directory questa riga:
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

Si vedano le [istruzioni sul sito ufficiale di LibreOffice](https://www.libreoffice.org/get-help/install-howto/macos/).  


### 5 - INSTALLAZIONE DI POSTGRESQL

Nella documentazione ufficiale Alfresco, la versione 6.0.7 di Alfresco Content Service è dichiarata compatibile con la versione 10.1 di PostgreSQL.  
Io ho installato la versione 12, e - per il momento - non ho avuto problemi.  
Seguire ad esempio le [istruzioni riportate qui](http://databasemaster.it/installare-postgresql-mac/).  

Una volta installato Postgres, consiglio di aprire una console **pgsql** da */Applications/PostgreSQL 12* e di creare un nuovo DB ("alfresco"), appartenente all'utente root default "postgres", e una nuova utenza abilitata a tale DB avente nome (e password) "alfresco":
> CREATE USER alfresco WITH PASSWORD 'alfresco';  
> CREATE DATABASE alfresco OWNER postgres ENCODING 'UTF8';  
> GRANT ALL PRIVILEGES ON DATABASE alfresco TO alfresco;  

**ATTENZIONE:** L'utenza di S.O. che viene creata automaticamente a seguito di quella PostgreSQL, nel mio caso aveva una password diversa da "alfresco": ero dovuta andare su *Mela > Preferenze di sistema... > Utenti e Gruppi* per modificarla (riportandola ad "alfresco").  


## INSTALLAZIONE DI ALFRESCO CONTENT SERVICES

Le istruzioni che seguono sono la copia, tradotta e leggermente modificata, del [post sull'hub di Alfresco](https://hub.alfresco.com/t5/alfresco-content-services-blog/a-script-to-install-alfresco-community-6-0/ba-p/288988) citato in precedenza.  

Per usare lo script è necessario prima scaricare i seguenti file:
- [Tomcat 8.5.37](https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.zip)
- [Alfresco Content Services 6.0.7-ga](https://artifacts.alfresco.com/nexus/content/repositories/public/org/alfresco/alfresco-content-services-community-distribution/6.0.7-ga/alfresco-content-services-community-distribution-6.0.7-ga.zip)
- [Alfresco Search Services 1.3.0](https://artifacts.alfresco.com/nexus/content/repositories/public/org/alfresco/alfresco-search-services/1.3.0/alfresco-search-services-1.3.0.zip)

Dopodiché mettere lo script e i tre file appena scaricati in una apposita cartella, ad esempio */Applications/alfresco*:  
> mkdir /Applications/alfresco  
> cd /Applications/alfresco  

Sotto la cartella con lo script e gli zip di installazione, creare una nuova cartella, vuota: questa nuova cartella andrà a contenere la nostra installazione Alfresco...  
> mkdir 6.0.7-ga  

Posizioniamoci nella cartella vuota appena creata per eseguire lo script:  
> cd 6.0.7-ga  
> ../install-alfresco-6x-ss.sh  

 
## APPENDICE: COME AVVIARE E FERMARE I SERVER

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

