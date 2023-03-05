# DescomplicandoArgoCD - WIP

## Descrição

Este repositório tem como objetivo descomplicar o uso do ArgoCD, bem como a utilização do GitOps.

Esse repositório faz parte do projeto [Certified Containers Expert](https://github.com/badtuxx/CertifiedContainersExpert)], onde o objetivo é preparar a pessoa para trabalhar com containers, não somente com uma tecnologia específica, mas sim com o conceito de containers e todas as ferramentas que são normalmente utilizadas nas melhores empresas de tecnologia do mundo.

&nbsp;


## Conteúdo do repositório
- [DescomplicandoArgoCD - WIP](#descomplicandoargocd---wip)
  - [Descrição](#descrição)
  - [Conteúdo do repositório](#conteúdo-do-repositório)
  - [ArgoCD](#argocd)
  - [GitOps](#gitops)

&nbsp;

## ArgoCD

O ArgoCD é uma poderoso ferramenta quando você pensa em GitOps ou Continous Delivery. O ArgoCD é um projeto open source, criado pela [Argo](https://argoproj.github.io/argo/), que tem como objetivo facilitar a implantação e gerenciamento de aplicações em Kubernetes.

O ArgoCD foi escrito em Go e utiliza o [Kubernetes Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) para gerenciar os recursos do Kubernetes.

Dito isso, fica claro que quando você instala o ArgoCD, você está instalando um operador do Kubernetes, que está extendendo o Kubernetes, adicionando novos `Custom Resources` e novos `Controllers` para gerenciar esses `Custom Resources`.

Vamos ver isso com mais detalhes mais pra frente, mas por agora saiba que o ArgoCD vai mudar a forma como você pensa e trabalha com Kubernetes.

Talvez esse seja somente o seu primeiro passo para o mundo do GitOps e do Continous Delivery, sim, entregas contínuas.

Você e sua equipe terá que se adaptar e criar maturidade para trabalhar com entregas contínuas, pois entregar é a tarefa mais fácil, o difícil é entregar com qualidade e com segurança de que não vai quebrar nada, que foi testado e que não vai afetar o negócio.

Acho que essa é uma boa apresentação do ArgoCD, e nem vou vou precisar falar que ele é peça fundamental nas melhores engenharías de software do mundo.

Está preparado para mais essa viajem com o objetivo de descomplicar mais um assunto, o ArgoCD?

#VAIIII


## GitOps

Esse livro não tem como objetivo descomplicar o GitOps. Nesse livro o nosso objetivo é descomplicar uma das etapas fundamentais do GitOps, que é a utilização do ArgoCD para gerenciar as entregas contínuas no Kubernetes.

O GitOps é um conceito que foi criado pela [Weaveworks](https://www.weave.works/), e que tem como objetivo facilitar a entrega de aplicações no Kubernetes, utilizando o Git como fonte de verdade. O Git é a fonte de verdade, e o Git é o único lugar onde você vai encontrar a verdade sobre o estado da sua aplicação.

Se lá é a fonte da verdade, vale a pena falar que quando estamos falando de GitOps, estamos falando sobre modo declarativo de gerenciar as aplicações no Kubernetes. Quando falamos em declarativo, estamos falando que o estado que das suas aplicação no Kubernetes, é o mesmo que está no Git, que é o mesmo que você deseja que esteja no Kubernetes.

Confuso? Calma, eu te explico.

Vamos imaginar que você tenha o seguinte arquivo no Git:

```yaml
apiVersion: apps/v1 # versão da API
kind: Deployment # tipo de recurso, no caso, um Deployment
metadata: # metadados do recurso 
  name: nginx-server # nome do recurso
spec: # especificação do recurso
  selector: # seletor para identificar os pods que serão gerenciados pelo deployment
    matchLabels: # labels que identificam os pods que serão gerenciados pelo deployment
      app: nginx # label que identifica o app que será gerenciado pelo deployment
  replicas: 2 # quantidade de réplicas do deployment
  template: # template do deployment
    metadata: # metadados do template
      labels: # labels do template
        app: nginx # label que identifica o app
      annotations: # annotations do template
        prometheus.io/scrape: 'true' # habilita o scraping do Prometheus
        prometheus.io/port: '9113' # porta do target
    spec: # especificação do template
      containers: # containers do template 
        - name: nginx # nome do container
          image: nginx # imagem do container do Nginx
          ports: # portas do container
            - containerPort: 80 # porta do container
              name: http # nome da porta
          volumeMounts: # volumes que serão montados no container
            - name: nginx-config # nome do volume
              mountPath: /etc/nginx/conf.d/default.conf # caminho de montagem do volume
              subPath: nginx.conf # subpath do volume
        - name: nginx-exporter # nome do container que será o exporter
          image: 'nginx/nginx-prometheus-exporter:0.11.0' # imagem do container do exporter
          args: # argumentos do container
            - '-nginx.scrape-uri=http://localhost/metrics' # argumento para definir a URI de scraping
          resources: # recursos do container
            limits: # limites de recursos
              memory: 128Mi # limite de memória
              cpu: 0.3 # limite de CPU
          ports: # portas do container
            - containerPort: 9113 # porta do container que será exposta
              name: metrics # nome da porta
      volumes: # volumes do template
        - configMap: # configmap do volume, nós iremos criar esse volume através de um configmap
            defaultMode: 420 # modo padrão do volume
            name: nginx-config # nome do configmap
          name: nginx-config # nome do volume
```

&nbsp;

Vamos dar o nome para esse arquivo de `nginx-deployment.yaml`.

Esse arquivo é somente um manifesto do Kubernetes, onde estamos especificando um `Deployment` do Nginx, com 2 réplicas, onde o Nginx está exposto na porta 80, e o Prometheus está fazendo o scraping da porta 9113.

Perceba, nesse arquivo estamos falando para o Kubernetes, que queremos que o Nginx esteja rodando com 2 réplicas, e que o Prometheus está fazendo o scraping da porta 9113, estamos declarando o estado desejado da nossa aplicação.

Para aplicar esse arquivo no Kubernetes, basta executar o seguinte comando:

```bash
kubectl apply -f nginx-deployment.yaml
```

&nbsp;

Vamos imaginar que por algum motivo precisamos mudar a quantidade de réplicas do Nginx para 3. Nós desejamos declarar o estado da nossa aplicação para 3 réplicas. 

Para isso, basta declarar, alterar a quantidade de réplicas do Nginx para 3 no arquivo `nginx-deployment.yaml`:


```yaml
apiVersion: apps/v1 # versão da API
kind: Deployment # tipo de recurso, no caso, um Deployment
metadata: # metadados do recurso 
  name: nginx-server # nome do recurso
spec: # especificação do recurso
  selector: # seletor para identificar os pods que serão gerenciados pelo deployment
    matchLabels: # labels que identificam os pods que serão gerenciados pelo deployment
      app: nginx # label que identifica o app que será gerenciado pelo deployment
  replicas: 3 # quantidade de réplicas do deployment
  template: # template do deployment
    metadata: # metadados do template
      labels: # labels do template
        app: nginx # label que identifica o app
      annotations: # annotations do template
        prometheus.io/scrape: 'true' # habilita o scraping do Prometheus
        prometheus.io/port: '9113' # porta do target
    spec: # especificação do template
      containers: # containers do template 
        - name: nginx # nome do container
          image: nginx # imagem do container do Nginx
          ports: # portas do container
            - containerPort: 80 # porta do container
              name: http # nome da porta
          volumeMounts: # volumes que serão montados no container
            - name: nginx-config # nome do volume
              mountPath: /etc/nginx/conf.d/default.conf # caminho de montagem do volume
              subPath: nginx.conf # subpath do volume
        - name: nginx-exporter # nome do container que será o exporter
          image: 'nginx/nginx-prometheus-exporter:0.11.0' # imagem do container do exporter
          args: # argumentos do container
            - '-nginx.scrape-uri=http://localhost/metrics' # argumento para definir a URI de scraping
          resources: # recursos do container
            limits: # limites de recursos
              memory: 128Mi # limite de memória
              cpu: 0.3 # limite de CPU
          ports: # portas do container
            - containerPort: 9113 # porta do container que será exposta
              name: metrics # nome da porta
      volumes: # volumes do template
        - configMap: # configmap do volume, nós iremos criar esse volume através de um configmap
            defaultMode: 420 # modo padrão do volume
            name: nginx-config # nome do configmap
          name: nginx-config # nome do volume
```

&nbsp;

Para que sua vonta seja aplicada, basta executar o seguinte comando:

```bash
kubectl apply -f nginx-deployment.yaml
```

&nbsp;

Pronto, agora o Kubernetes sabe que você deseja que o Nginx esteja rodando com 3 réplicas e assim o fez.

Isso é o que chamamos de **estado desejado**, estado que declaramos para o Kubernetes, e o Kubernetes mudou o estado atual para o estado desejado.

Acredito que agora você tenha entendido um conceito muito importante do Kubernetes e no GitOps, que é o **estado desejado**.

Se é importante para o GitOps, é importante para o ArgoCD, e é por isso que o ArgoCD trabalha com o conceito de **estado desejado**.

