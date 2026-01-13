# CodeArtifact

## Intro

Servizio gestito AWS per `artifact management` salvare libreria o dipendenze necessari durante le build dei servizi.

Si integra con i maggiori gestori di pacchetti di vari languaggi.

## Come funziona

É un servizio che è situato dentro la tua VPC, i principali meccanismo sono 2:

1. **Proxy**
    - Servizi (codeBuild) o sviluppatori punteranno sempre al repository nel dominio CodeArtifact
    - CodeArtifact instraderà la richiesta al gestore di pacchetti del caso (npm, pip, maven...)
    - Salverà l'artefatto nel proprio repository e lo fornirà al richiedente
    - Se sarà richiesto nuovamente non ci sarà bisogno di fare richeiste al servizio terzo
    - L'artefatto rimarrà disponibile in CodeArtifact finchè non esplicitamente rimosso
2. **Repository Privata**
   - Non instraderà le richeiste a nessuna terza parte
   - Sarà usato come repository privato per i pacchetti privati

## Eventi

É possibile ascoltare eventi emessi da CodeArtifact utilizzando regole EventBridge.

Può essere usato per mantenere aggiornate le dipendenze nei repository.

## Resource Policy

Le policy di CodeArtifact forniscono accesso a tutti gli artefatti nel repo o a nessuno di essi.

I permessi possono essere forniti anche a livello di Account AWS, quindi entità provenienti da un altro account potranno accedere a tutte le risorse di quel repository. Utile per distribuire il codice.

## Upstream

Ogni repository di CodeArtifact può avere degli upstream, ovvero dei collegamenti a differenti repository.
Questo comporta il vantaggio di avere un unico punto di ricerca dove dover cercare l'albero delle dipendenze.

C'è la possibilità di configurare dei collegamenti a reposiory esterne, però pò essercene al **massimo uno per repository**, questo per garantire un meccanismo di caching ottimale.

Un'architettura tipica è quella di definire un repository in CodeArtifact con lo scopo di puntare ad una libreria npm esterna, in questo modo tutti gli altri repository punteranno a quello ogni volta che dovranno scaricare quella dipendenza.

Una volta che il package viene salvato nella repository di downstream i cambiamenti nell'upstream non lo influeneranno (cancelazione, aggiornamento etc...).

Ogni repository intermedia nella catena delle dipendenze non salverà il downstream nella propria cache per motivi di afficienza.

## Domains

Si tratta di un raggruppamento di repository, anche attraverso più account AWS.

Vantaggi:

- Centralizzazione del riferimenti
- De-duplicazione delle dipenze utilizzate più volte, infatti se un paccketto è utilizato in più repository sarà salvato in un o storage condiviso tra gli account in modo da diminuire i costi e aumentare l'efficienza.
- Consente di condividere l'accesso meglio tra Team e repository usando una singola chiave KMS per criptare i dati.
- La gestione dell'accesso è semplificata attraverso le `Domain Resource-based Policy`, con le quali è possibile stabile quale account accede alle repository e chi può configurare upstream esterni.
