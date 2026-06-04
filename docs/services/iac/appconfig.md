# AWS AppConfig <!-- omit in toc -->

- [Intro](#intro)

## Intro

Il Serviizo AWS Appconfig è necessario per poter configurare i servizi Cloud fornendo oro una configurazione in modo sicuro.

I valori passati alla configurazione didnamica sono caricati automaticamente nel servizio senza la necessità di riavviare quesl'utlimo.

Questo servizio è molto utile anche per poter configurare un servizio di "**FeatureFlags**".

Apponfig può essere utilizzato per rilasciare delle configurazioni per servizi su EC2, Lambda, ECS o EKS.

I deploy di queste configurazioni possono graduali e controllati, ovvero la configurazione può essere validata secondo uno **schema json** o da una **funzione Lambda**.

Si tratta quindi di un servizio necessario per rilasciare modifiche alle configurazioni del codice senza effettivamente eseguire una nuova release o modificare il codice dipartenza.
