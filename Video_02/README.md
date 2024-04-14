# Ejemplos de Workflow Template con Argo Workflows 🐙

![ArgoWorkflows_Video_01.jpg](images%2FArgoWorkflows_Video_01.jpg)

Hoy vamos a ver como utilizar WorkflowTemplates de Argo Workflows con un caso de uso muy sencillo haciendo uso de HashiCorp Vault.

0:00 Inicio

0:26 INTRO

0:38 Descripción

1:49 Instalación

4:07 Ejemplo Workflow 01

5:52 Ejemplo Workflow 02

6:23 Ejemplo Workflow 03

8:50 Introducción Workflow Templates

9:15 Ejemplo Workflow Templates Separados

12:45 Ejemplo Workflow Templates 01

15:50 Ejemplo Workflow Templates 02

19:15 Conclusión

19:32 Despedida

19:41 OUTRO

GitHub:
https://github.com/TheAutomationRules/argoworkflows/blob/main/Video_02/README.md

Twitter: @TheAutomaRules
Instagram: TheAutomationRules

Visita y suscríbete en mi otro canal!
https://www.youtube.com/@TheRavenManCave

#kubernetes #argo #workflows #templates #application #gitops #continuousdelivery #helm #chart #vault #hashicorp #secret #automation

# TEMPLATES

## EJEMPLO 1

````yaml
metadata:
  name: tpl01-vault-version
  namespace: default
  labels:
    example: 'true'
spec:
  entrypoint: version
  arguments:
    parameters:
      - name: docker-image
        value: "theautomationrules/custom-vault:0.1.0"
  templates:
    - name: version
      container:
        name: main
        image: "{{workflow.parameters.docker-image}}"
        command:
          - vault
        args:
          - --version
            
  ttlStrategy:
    secondsAfterCompletion: 300
  podGC:
    strategy: OnPodCompletion
````

## EJEMPLO 2

````yaml
metadata:
  name: tpl02-vault-help
  namespace: default
  labels:
    example: 'true'
spec:
  entrypoint: help
  arguments:
    parameters:
      - name: docker-image
        value: "theautomationrules/custom-vault:0.1.0"
  templates:
    - name: help
      container:
        name: main
        image: "{{workflow.parameters.docker-image}}"
        command:
          - vault
        args:
          - --help

  ttlStrategy:
    secondsAfterCompletion: 300
  podGC:
    strategy: OnPodCompletion
````
## EJEMPLO 3

````yaml
metadata:
  name: tpl03-vault-login
  namespace: default
  labels:
    example: 'true'
spec:
  entrypoint: login
  arguments:
    parameters:
      - name: docker-image
        value: "theautomationrules/custom-vault:0.1.0"
      - name: vault_token
        value: "s.1234567890"
      - name: vault_addr
        value: "http://nomad1:8200"
  templates:
    - name: login
      script:
        imagePullPolicy: "Always"
        image: "{{workflow.parameters.docker-image}}"
        command: ["sh"]
        source: |
          export VAULT_ADDR="{{workflow.parameters.vault_addr}}" && vault login "{{workflow.parameters.vault_token}}"

  ttlStrategy:
    secondsAfterCompletion: 300
  podGC:
    strategy: OnPodCompletion
````

# WORKFLOW TEMPLATES

## EJEMPLO 01

````yaml
metadata:
  name: wftpl01-vault
  namespace: default
  labels:
    example: 'true'
spec:
  entrypoint: version
  arguments:
    parameters:
      - name: docker-image
        value: "theautomationrules/custom-vault:0.1.0"
      - name: vault_token
        value: "s.1234567890"
      - name: vault_addr
        value: "http://nomad1:8200"
      
  templates:
    - name: version
      container:
        name: main
        image: "{{workflow.parameters.docker-image}}"
        command:
          - vault
        args:
          - --version
    - name: help
      container:
        name: main
        image: "{{workflow.parameters.docker-image}}"
        command:
          - vault
        args:
          - --help
    - name: login
      script:
        imagePullPolicy: "Always"
        image: "{{workflow.parameters.docker-image}}"
        command: ["sh"]
        source: |
          export VAULT_ADDR="{{workflow.parameters.vault_addr}}" && vault login "{{workflow.parameters.vault_token}}"

  ttlStrategy:
    secondsAfterCompletion: 300
  podGC:
    strategy: OnPodCompletion
````

## EJEMPLO 02

````yaml
metadata:
  name: wftpl02-vault
  namespace: default
  labels:
    example: 'true'
spec:
  entrypoint: basic
  arguments:
    parameters:
      - name: docker-image
        value: "theautomationrules/custom-vault:0.1.0"
      - name: vault_token
        value: "s.1234567890"
      - name: vault_addr
        value: "http://nomad1:8200"
      - name: template
        value: version
        enum:
          - version
          - help
          - login
      
  templates:
    - name: version
      container:
        name: main
        image: "{{workflow.parameters.docker-image}}"
        command:
          - vault
        args:
          - --version
    - name: help
      container:
        name: main
        image: "{{workflow.parameters.docker-image}}"
        command:
          - vault
        args:
          - --help
    - name: login
      script:
        imagePullPolicy: "Always"
        image: "{{workflow.parameters.docker-image}}"
        command: ["sh"]
        source: |
          export VAULT_ADDR="{{workflow.parameters.vault_addr}}" && vault login "{{workflow.parameters.vault_token}}"
    - name: basic
      dag:
        tasks:
          - name: template
            template: "{{workflow.parameters.template}}"

  ttlStrategy:
    secondsAfterCompletion: 300
  podGC:
    strategy: OnPodCompletion
````