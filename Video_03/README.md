# Como usar Artifacts en Argo Workflows 🐙

![ArgoWorkflows_Video_03.jpg](images%2FArgoWorkflows_Video_03.jpg)

Hoy vamos a ver como utilizar Artifacts en ArgoWorkflows, para ello veremos ejemplos sencillos descargando paquetes o instaladores por HTTP y un ejemplo con MIN.IO un servicio de S3 Storage de que hemos hablado en otro video, no te pierdas el video entero y recuerda darme un "Me Gusta!" subscribete y comparte!

0:00 Inicio

0:05 INTRO

0:17 Descripción

1:06 Ejemplo 01

3:53 Ejemplo 02

6:11 Ejemplo 03

13:17 Conclusión

13:37 OUTRO

GitHub:
https://github.com/TheAutomationRules/argoworkflows/blob/main/Video_03/README.md

Twitter: @TheAutomaRules
Instagram: TheAutomationRules

Visita y suscríbete en mi otro canal!
https://www.youtube.com/@TheRavenManCave

#kubernetes #argo #workflows #artifacts #application #gitops #continuousdelivery #helm #chart

# TEMPLATES

Para nuestras pruebas vamos a necesitar un servicio de S3 para lo que vamos a usar MIN.IO que es un servicio del que ya hemos hablado en videos anteriores, para acceder a este servicio he definido los siguientes datos de acceso:

accesskey: 
````
AOTdX1auNfvQM1osUS32
````
secretkey:
````
SiDuC7Idfst3KWtCQymteFn1ekKvswWvmzdTLWX9
````

## EJEMPLO 1

En este primer ejemplo lo que haremos sera hacer la instanacion del binario de Kubectl y lo ejecutaremos para saber su version, lo haremos desde un contenedor con Linux Debian en su version 9.4.

````yaml
metadata:
  generateName: arguments-artifacts-
spec:
  entrypoint: kubectl-input-artifact
  arguments:
    artifacts:
      - name: kubectl
        http:
          url: https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/linux/amd64/kubectl

  templates:
    - name: kubectl-input-artifact
      inputs:
        artifacts:
          - name: kubectl
            path: /usr/local/bin/kubectl
            mode: 0755
      container:
        image: debian:9.4
        command: [sh, -c]
        args: ["kubectl version"]
  ttlStrategy:
    secondsAfterCompletion: 300
    podGC:
      strategy: OnPodCompletion
````

## EJEMPLO 2

Ahora vamos a declarar un artifact que apuntara a un fichero zip que en este caso se trata del binario de HashiCorp Consul, lo instalaremos y lo ejecutaremos para saber que version es, en este caso usaremos una imagen de Docker propia y pasaremos como parametros la version y arquitectura del binario para que descargue la version exacta que necesitamos.

````yaml
metadata:
  generateName: arguments-artifacts-consul-
spec:
  entrypoint: consul-input-artifact
  arguments:
    arguments:
    parameters:
      - name: docker-image
        value: "theautomationrules/custom-vault:0.1.0"
      - name: consul-version
        value: "1.18.2"
      - name: arch
        value: "linux_amd64"
    artifacts:
      - name: consul
        http:
          url: https://releases.hashicorp.com/consul/{{workflow.parameters.consul-version}}/consul_{{workflow.parameters.consul-version}}_{{workflow.parameters.arch}}.zip

  templates:
    - name: consul-input-artifact
      inputs:
        artifacts:
          - name: consul
            path: /tmp/consul.zip
            mode: 0755
      script:
        imagePullPolicy: "Always"
        image: "{{workflow.parameters.docker-image}}"
        command: ["sh"]
        source: |
          cd /tmp/
          ls -la
          unzip consul.zip
          install consul /usr/local/bin/
          ls -la /usr/local/bin/
          consul -v
  ttlStrategy:
    secondsAfterCompletion: 300
    podGC:
      strategy: OnPodCompletion
````

## EJEMPLO 3

Aqui lo vamos a complicar un poco mas añadiendo como Artifact el uso de un bucket S3 para lo que vamos a usar el servicio MIN.IO del cual ya hemos hablado en un video anterior, lo que haremos sera decirle a nuestro Workflow que queremos que almacene en ese bucket el contenido completo de nuestro directorio /tmp donde hemos clonado un repositorio de codigo privado y el binario de consul descomprimido.

````yaml
metadata:
  generateName: arguments-artifacts-minio-
spec:
  serviceAccountName: argo-artifacts
  entrypoint: minio-input-artifact
  arguments:
    arguments:
    parameters:
      - name: docker-image
        value: "theautomationrules/custom-vault:0.1.0"
      - name: consul-version
        value: "1.18.2"
      - name: arch
        value: "linux_amd64"
      - name: git-repository
        value: "https://github.com/TheAutomationRules/testing.git"
    artifacts:
      - name: consul
        http:
          url: https://releases.hashicorp.com/consul/{{workflow.parameters.consul-version}}/consul_{{workflow.parameters.consul-version}}_{{workflow.parameters.arch}}.zip
      - name: private-repo-artifact
        path: /tmp/
        git:
          repo: "{{workflow.parameters.git-repository}}"
          depth: 1
          revision: "main"
          insecureIgnoreHostKey: true
          usernameSecret:
            name: git-credentials
            key: username
          passwordSecret:
            name: git-credentials
            key: token

  templates:
    - name: minio-input-artifact
      inputs:
        artifacts:
          - name: consul
            path: /tmp/consul.zip
            mode: 0755
          - name: private-repo-artifact
            path: /tmp/private-repo-artifact/
            mode: 0755
      script:
        imagePullPolicy: "Always"
        image: "{{workflow.parameters.docker-image}}"
        command: ["sh"]
        source: |
          cd /tmp/
          ls -la
          unzip consul.zip
          install consul /usr/local/bin/
          ls -la /usr/local/bin/
          consul -v
          touch test.txt
          ls -la /tmp/
        env:
          - name: GH_TOKEN
            valueFrom:
              secretKeyRef:
                name: git-credentials
                key: token
      outputs:
        artifacts:
          - name: repository
            path: /tmp/
            mode: 0777
            recurseMode: true
            s3:
              endpoint: argo-artifacts:9000
              bucket: argo-workflows-s3 # S3 Bucket Name
              insecure: true
              key: "test-push-to-minio"
              accessKeySecret:
                name: argo-artifacts # name of secret
                key: accesskey # key in secret
              secretKeySecret:
                name: argo-artifacts # name of secret
                key: secretkey # key in secret
            archive:
              none: { }
  ttlStrategy:
    secondsAfterCompletion: 300
    podGC:
      strategy: OnPodCompletion
````