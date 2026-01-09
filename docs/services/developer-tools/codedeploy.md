---
tags:
  - devops
  - cicd
---

# CodeDeploy <!-- omit in toc -->

- [EC2 / On-Prem](#ec2--on-prem)
  - [Blue-Green Deployment](#blue-green-deployment)
    - [Instance Termination](#instance-termination)
  - [Avanzato](#avanzato)
    - [in-place](#in-place)
    - [EC2 - Deployment Hooks](#ec2---deployment-hooks)
    - [Deployment Configurations](#deployment-configurations)
    - [Triggers](#triggers)
  - [Permessi](#permessi)
- [Lambda](#lambda)
  - [Pipeline Tipica](#pipeline-tipica)
  - [Requisiti `appspec.yaml`](#requisiti-appspecyaml)
  - [Lambda - Deployment Hooks](#lambda---deployment-hooks)
- [ECS](#ecs)
  - [Velicità di deploy](#velicità-di-deploy)
  - [ECS - Deployment Hooks](#ecs---deployment-hooks)
- [Rollbacks](#rollbacks)
  - [Troubleshooting](#troubleshooting)
    - [`InvalidSignatureException`](#invalidsignatureexception)
    - [Deployment e Lifecycle Events sono ignorati](#deployment-e-lifecycle-events-sono-ignorati)
    - [ASG ScaleOut Old Version](#asg-scaleout-old-version)
    - [Allow Traffic Fail](#allow-traffic-fail)

Sistema per rilasciare nuovi update o rollback di applicazioni, Lambda, ECS, EC2 o on-prem services.

## EC2 / On-Prem

- Due tipologie di deploy: **in-place** o **blue-green**
- L'istanza target deve avere l'agent CodeDeploy running per poter essere gestita da questo servizio
- DeploymentSpeed:
  - `AllAtOnce` - Più downtime possibile ma più veloce
  - `HalfAtATime` - 50% alal volta, quindi capacità ridotta, compromesso tra velocità e uptime
  - `OneAtATime` - Più lento ma quello con più affidabilità di servizio e meno impatto possibile

### Blue-Green Deployment

Questa tipologia di rilascio prevede che vi sia un ALB prima del servizio delle istanze EC2 o del loro Auto Scaling Group (ASG).

1. Viene creato un nuovo Auto Scaling Group con delle EC2 con la nuova versione del software.
2. Viene indirizzato ALB verso questo nuovo gruppo
3. Il gruppo vecchio viene distrutto

**Rilascio manuale** - Quando si devono aggiornare delle EC2 standalone è necessario sempre usare dei tag per individuare i gruppi di istanze "blue" e "green". Quindi deve essere pre-configurato il tag che indica il valore delle istanze che utilizzeranno la nuova versione.

**Rilascio Automatico** - Quando il blue-green è applicato su ASG.
In questo caso verrà creato un intero ASG nuovo con la versione "green", una volta creato e configurato il load balancer sposterà il traffico.

Nel capitolo riguardo ai [deployment hook](#deployment-hooks), nel caso del blue-green delle fasi sono differenziate in base alla tipologia di istanza, quindi blue green.

Nelle istanze Blue tendenzialmente avverranno le fasi solo di blocco del traffico.

#### Instance Termination

CodeDeploy consente di decidere quando terminare le istanze "blue" attraverso la `BlueInstanceTerminationOption`:

- `Action Terminate` - Basta indicare un massimo tempo di attesa da 1H a 2GG max
- `Acition Keep Alive` - Le istanze sono mantenute ma deregistrate dall'ALB, il più sicuro ma il più costoso

### Avanzato

#### in-place

Per eseguire degi deploy `in-place` è necessario indentificare tutte le istanze che devono essere sostituite.

- Per delle semplici EC2 -> si usano dei tag per consentire a CodeDeploy di sostituire solo quelle necessarie
- Se si tratta di ASG CodeDeploy opererà autonomamente per il rilascio di nuove istanze.

In questa tipologia di update il traffico del load balancer viene interrotto finchè l'aggiornamento non sarà terminato

#### EC2 - Deployment Hooks

Sono degli script che CodeDeploy potrà eseguire ad ogni update di istanza Ec2.

Un aggiornamento tipico segue queste fasi:

1. **Start**
2. **Before Block Traffic** -> è possibile usare uno script
3. **Block Traffic**
4. **AfterBlockTraffic** -> è possibile usare uno script
5. **Application Stop** -> Script necessario per indicare a CodeDeploy come fermare il servizio
6. **Download Bundle**
7. **Before Install** -> è possibile usare uno script
8. **After Install** -> è possibile usare uno script
9. **Application Start** -> Script necessario per indicare a CodeDeploy come avviare il servizio
10. **Validate Service** -> è possibile usare uno script per validare il servizio
11. **Before Allow Traffic** -> è possibile usare uno script
12. **Allow Traffic**
13. **After Allow Traffic** -> è possibile usare uno script

Gli step _1, 2, 3, 4, 11, 12, 13_ accadono **SOLO** quando il servizio da aggiornare è fornito di Load Balancer.

CodeDeploy usa un file `appspec.yaml` per individuare quali script deve eseguire l'agente all'interno della macchina.

#### Deployment Configurations

Permette di indicare a CodeDeploy quante istanze devono rimanere disponibili durante gli aggiornamenti.

- `CodeDeployDefault.AllAtOnce` - Rilascia il maggior numero di istanze contemporaneamente, maggior downtime però più rapido
- `CodeDeployDefault.HalfAtATime` - Rilascia solo il 50% alla volta
- `CodeDeployDefault.OneAtATime` - Sistema più lento ma più sicuro per l'uptime del servizio.

#### Triggers

Per ogni aggiornameto verso istanze EC2 CodeDeploy notifica il sistema SNS per far giungere notifiche di eventuali errori agli utenti finali.

### Permessi

Per poter funzionare l'agente CodeDeploy deve poter accesso ad S3, dove andrà a salvare la versione dell'applicazione che deve rialsciare.

## Lambda

CodeDeploy con il servizio Lambda aiuta a dirottare il traffico delle richieste tra `Alias` di Lambda.

Completamente integrato con SAM, Serverless Application Model.

Crea quindi una nuova versione per lambda che viene poi assegnata ad un Alias.
Il traffico viene spostato tra l'alias di produzione con la vecchia versione e il nuovo alias.

CodeDeploy per Lambda viene sempre configurato tramite il file `appspec.yaml` salvato in un bucket S3.

> CodeDeploy agent non è necessario perchè si tratta di un servizio serverless

Questo spostamento può seguire diverse velocità:

- `Lineare` - Aumento il traffico di X% ogni N minuti
- `Canary` - X% verso la nuova versione e poi sposto il traffico completamente dopo i test
- `AllAtOnce` - Più veloce in assoluto ma non vi è la possibilità di fare del testing.

### Pipeline Tipica

In una pipeline CodeBuild avrà il compito di:

- creare la nuova versione della funzione Lambda
- Aggiornare il file `appspec.yaml` nel bucket S3 per poi passarlo come input allo step CodeDeploy

### Requisiti `appspec.yaml`

I requisiti per consentire a CodeDeploy di aggiornare una funzione Lambda sono:

- `Nome` della funzione lambda da rilasciare
- `Alias` della funzione lambda
- `CurrentVersion` la versione corrente "blue"
- `TargetVersion` la versione nuova "green", dove sarà trasferito il traffico

### Lambda - Deployment Hooks

Come per ECS sono eseguiti da Lambda functions.

É molto più semplice perchè ci sono solo due fasi nel quale si possono eseguire lambda custom:

- `BeforeAllowTraffic`
- `AfterAllowTraffic`

## ECS

Molto simile a Lambda, stesse modalità di deploy.

Supporta SOLO Blue-Green e l'immagine deve già esiste in ECR.

Siccome aggiorna la configurazione di container CodeDeploy può solo creare nuove `Task Definition`.

Il file `appspec.yaml` necessario a CodeDeploy dovrà essere salvato in un file S3 e conterrà le info riguardo al Task Definition e al Load balancer.

Rilascerà nuovi task dentro il cluster ECS e cancellerà i vecchi task una volta rialsciati i nuovi.

In una possibile pipeline il task di Codebuild si occuperà di:

- Creare l'immagine del container e pusharla in ECR
- Creare la nuova task definition in ECS
- Aggiornare il file appspec.yaml in S3 per consentire a CodeDeploy di lavorare sul rilascio.
- Passare come input artifact a Codedeploy l'arn al file appspec.yaml a CodeDeploy

### Velicità di deploy

- Linear -> Sostituisce X% istanze ogni N minuti
- Canary -> Crea tutte le istanze in un nuovo gruppo e invia X% del traffico per N Minuti, Se ok passa al 100%
- AllAtOnce -> più Veloce e meno costoso ma perdita di servizio

É possibile definire anche un ELB di test per testare il gruppo "green" prima del ribilanciamento di traffico

### ECS - Deployment Hooks

Sono funzioni Lambda lanciare per ogni Deploy, come per il deploy con le istanze EC2 anche qui ci sono varie fasi nel quale con le lambda si può testare la corretta progressione del deploy.

## Rollbacks

In caso di fallimento CodeDeploy può rilasciare una vecchia versione del servizio, il rollback figura come una attivazione di CodeDeploy per un nuovo rilascio.

Questa azione può essere attivata:

- **Automaticamente** - Se CodeDeploy ha la possibilità di verificare la salute della nuova versione o se sono state settate delle Cloudwatch Rules con dei threshold
- **Manualmente** - Attivata da un utente

La possibilità di fare rollback può anche essere completamente disattivata.

### Troubleshooting

#### `InvalidSignatureException`

É dovuto ad un time mismatch tra CodeDeploy e l'istanza EC2 o on-prem sul quale deve operare.

#### Deployment e Lifecycle Events sono ignorati

Quando nel gruppo di istanze da sottoporre a CodeDeploy sono presenti troppe istanze `unhealthy` o **troppi Deploy sono falliti**.

Le cause possono essere:

- L'agente CodeDeploy non è attivo sulle istanze o non installato, oppure CodeDeploy non può raggiungere le istanze
- I permessi di CodeDeploy non sono impostati correttamente
- Se posto dietro ad un proxy, assicurarsi che l'agente sia stato configurato con il parametro `:proxy_uri:`
- Anche qui può esserci un problema di sincronizzazione del tempo tra l'agente e CodeDeploy

#### ASG ScaleOut Old Version

Nel caso di aggiornamento ad un Austo Scaling Group, se avviene un evento di `Scale Out` durante l'operazione le nuove istanze saranno lanciate con la versione precedente e non quella nuova desiderata.

In questi casi dove un ASG si troverà con ppiù versioni nello stesso istante CodeDeploy effettuerà un follow-up deployment per assicurarsi di aggiornare tutte le istanze correttamente.

#### Allow Traffic Fail

Se il check per consentire il traffico durante un deploy continua a fallire può essere un problema relativo alla configurazione errata negli ELB.
