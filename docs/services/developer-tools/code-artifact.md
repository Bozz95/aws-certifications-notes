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
