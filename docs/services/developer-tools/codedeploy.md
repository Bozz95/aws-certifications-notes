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
    - [Deployment Hooks](#deployment-hooks)
    - [Deployment Configurations](#deployment-configurations)
    - [Triggers](#triggers)
  - [Permessi](#permessi)
- [Lambda](#lambda)
- [ECS](#ecs)

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

#### Deployment Hooks

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

Il traffico viene spostato tra l'alias di produzione con la vecchia versione e il nuovo alias.

Questo spostamento può seguire diverse velocità:

- `Lineare` - Aumento il traffico di X% ogni N minuti
- `Canary` - X% verso la nuova versione e poi sposto il traffico completamente dopo i test
- `AllAtOnce` - Più veloce in assoluto ma non vi è la possibilità di fare del testing.

## ECS

Molto simile a Lambda, stesse modalità di deploy.
