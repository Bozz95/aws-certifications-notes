# Codepipeline <!-- omit in toc -->

- [Stages](#stages)
- [Artefatti](#artefatti)
- [Troubleshooting](#troubleshooting)
- [Version](#version)
- [Triggers](#triggers)
- [Manual Approval - Approfondimento](#manual-approval---approfondimento)
- [Cloudformation - Deploy](#cloudformation---deploy)

Strumento per automaitizzare le pipeline.

## Stages

Ha diversi stage di esecuzione:

- **Source** - Dove ottiene l'artefatto dal quale deve partire la CICD -> Servizi: CodeCommit, ECR, S3, Bitbucket, Github
- **Build** - Fase in cui viene compilato il sorgente per creare l'artefatto da usare nelle fasi successive -> Servizi: CodeBuild, Jenkins, CloudBees, TeamCity
- **Test** - Viene testata la bontà dell'artefatto -> Servizi: CodeBuild, AWS Device Farm, 3rd party tool
- **Deploy** - Viene rilasciato l'artefatto oppure installato -> CodeDeploy, Elastic Beanstalk, Cloudformation, ECS, S3, Codebuild con automazioni custom
- **Invoke** - É possibile invokare funzioni custom per azioni custom - Lambda, Step Functions
- **Manual Approval** - Interrompe il flusso della pipeline finchè un attore non interviene ad approvare manualmente

## Artefatti

Ogni stage crea degli artefatti che sono passati attraverso diversi stage, ogni artefatto viene salvato in bucket S3.

**Codepipeline** si occupa di passare come input/output questi artefatti tra gli stage e S3.

## Troubleshooting

- Verificare gli stati degli stage di Codepipeline
- Usare **Eventi Cloudwatch** tramite regole EventBridge per monitorare fallimenti o eventi di cancellazione
- Infine la console dove se la pipeline fallisce vengono restituiti errori
- Potrebbero esserci errori di permessi
- Cloudwtrail per verificaree le chiamate sono andate a buon fine

## Version

- **V1** - Deprecated Paghi 1 dollaro al mese per pipeline attiva
- **V2** - Pay what you use

## Triggers

- **Events** - _Consigliato_, crea delle regole EventBridge che "ascoltano" determinati eventi e notificano CodePipeline di conseguenza.
    >Per sorgenti esterni, come Github, viene creata utilizzato il servizio differente per generare questi eventi, chiamato Codestart Source Connection
- **Webhook** - Old way che consiste in un endpoint esposto di Codepipeline di essere chiamato tramite script o altri metodi per decidere quanto avviare la pipeline
- **Polling** - Metodo deprecato nel quale Codepipeline ad intervalli esegue delle chiamate per verificare se qualcosa è cambiato

## Manual Approval - Approfondimento

Si tratta di una tipologia di azione di AWS, perchè è legata alle capacità del servizio Codepipeline.

Necessità di 0 artefatti di input e genera 0 artefatti di output.

Consiste nel notificare degli attori tramite notifiche SNS per mail.

L'utente notificato per poter approvare la pipeline necessita dei seguenti permessi:

- `codepipeline:GetPipeline*` - Per poter visualizzare la pipeline completa
- `codepipeline:PutApprovalResult` - Per approvare in sè lo step di approvazione manuale

## Cloudformation - Deploy

Cloudformation può essere un "target" di Codepipeline per essere utilizzato per deployare risorse in differenti Account o regioni, usando degli `StackSets`.

Es: rilasciare diverse Lambda utilizzando CDK o SAM (Serverless Application Model).

Un esempio di pipeline completa potrebbe consistere in questa sequenza:

1. **Codebuild Build App** - Crea il codice e gli artefatti per rilasciare la nuova versione del servizio
2. **Cloudformation CREATE/REPLACE** - Crea o aggiorna un ambiente di QA dove viene installata questa nuova versione
3. **Codebuild Test App** - Vengono eseguiti dei test automatici o di load verso questo nuovo ambiente
4. **Cloudformation DELETE QA ENV** - Una volta terminati i test l'ambiente di QA viene distrutto
5. **Cloudformation UPDATE PROD** - Se i test sono stati positivi Cloudformation può procedere ad aggiornare l'ambiente di produzione con la nuova versione

Cloudformtion usato come Target in una pipeline può essere parametrato in questi modi:

Scegliendo l'`Action Mode`:

- Crea o aggiorna un Change Set oppure ne esegue uno precedentemente generato
- Crea o aggiorna uno Stack, può Cancellarlo o Sostituirne uno fallito

Sovrascrivendo dei `Template JSON`:

