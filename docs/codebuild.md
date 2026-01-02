# Codebuild <!-- omit in toc -->

Codebuild è un servizio per eseguire delle sequenze di azioni partendo da un artefatto di input, simile a Jenkins ma completamente gestito.

## Info base

Ogni esecuzione di Codebuild può essere configurata con queste opzioni:

- **Source** - Dove il servizio deve ottenere l'artefatto di input dal quale eseguire le azioni. Codecommit, S3, Bitbucket, Github etc...
- **Build Instructions** - Sono le sequenze di comandi che deve eseguire, sono specificate nel file `buildspec.yaml` oppure configurate direttamente nel progetto Codebuild in modo da non dover salvare nessun file aggiuntivo nel progetto
- **Log di Output** - Possono essere salvati in S3 o direttamente in Cloudwatch con una retention policy
- **Cloudwatch metrics** - per monitorare l'istanza che esegue i comandi
- **EventBridge rules** - Per identificare fallimenti o innescare notifiche nel caso

## Runtime Environment

I seguenti linguaggi hanno delle loro immagini Codebuild predefinite per poter compilare progetti scritti in quel modo:

- java
- ruby
- python
- Go
- Node.js
- Android
- .NET Core
- PHP
- `Docker` - Per customizzare queste immagini e agigungere il runtime a piacimento

## Cache

Per poter usare dei file in più esecuzioni di Codebuild successive è possibile usare un layer di cache, come un bucket S3, per riutilizzare file.

## buildspec.yaml

- Deve essere nella root del progetto
- è possibile dichiarare delle variabili d'ambiente:
  - hardcoded nel file yaml stesso
  - pescate da SSM o SecretManager per segreti
- É possibile dichiarare delle Phases per raggruppare i comandi:
  - `install` - fase in cui vengono installati pacchetti necessari alle fasi successive
  - `pre_build` - comandi in preparazione alla build
  - `build` - Comandi effettivi per la build
  - `post_build` - Comandi per preparare gli artefatti, come creazione di zip o tagging di immagini etc etc
- `Artifacts` - Sezione in cui vengono definiti quali file devono essere caricati come artefatti di output in S3 (cryptati con lachiave KMS scelta)
- `cache` - file che devono essere salvati nel layer di cache S3, solitamente dipendenze per la build che non è necessario ri-creare ogni volta

## Local Testing/Build

É possibile eseguire un'istanza di Codebuild localmente sfruttando docker.

Può essere utile se è necessario fare del debugging avanzato.

Si può sfruttare il Codebuild Agent per eseguirlo localemente come se fosse un'istanza gestita.

## Integrazioni VPC

Di default Codebuild è slegato da ogni VPC nell'account.

Come per le funzioni Lambda è possibile definire una configurazione per farlo accedere alle risorse all'interno della VPC. Si necessita di:

- ID VPC
- ID delle subnet
- ID dei security Group

In questo modo Codebuild potrà accedere a risorse che normalmente sono chiude dietro una VPC, come RDS, EC2, Elasticache, ALB ...

Use case:

- Fare dei test su dati veritieri
- Load testing su servizi rilasciati dietro un ALB