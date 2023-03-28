# Descomplicando ArgoCD - DAY-1

## O que vamos aprender no Day-1?

Durante o day-1 iremos aprender o que é o ArgoCD, vamos entender o que é o GitOps e o que é estado desejado, vamos entender o que é um continuous delivery e como o ArgoCD pode nos ajudar a entregar nossas aplicações no Kubernetes de forma contínua e segura.

Vamos ainda ver como instalar o nosso ArgoCD no Kubernetes como um operador, e também iremos instalar o ArgoCD CLI, que é a ferramenta que vamos utilizar para gerenciar as nossas aplicações no ArgoCD.

Vamos adicionar o nosso cluster e criar a nossa primeira aplicação no ArgoCD, para que seja possível o deploy em nosso cluster.

Enfim, muita coisa interessante para aprender no day-1, e para isso, vamos precisar de alguns pré-requisitos.

- [ ] Ter um cluster Kubernetes rodando
- [ ] Ter o kubectl instalado
- [ ] Ter força de vontade para aprender

Preencheu os pré-requisitos? Então vamos lá!

&nbsp;


## Conteúdo do Day-1
- [Descomplicando ArgoCD - DAY-1](#descomplicando-argocd---day-1)
  - [O que vamos aprender no Day-1?](#o-que-vamos-aprender-no-day-1)
  - [Conteúdo do Day-1](#conteúdo-do-day-1)
  - [O que é o ArgoCD?](#o-que-é-o-argocd)
  - [O que é GitOps?](#o-que-é-gitops)
  - [Pré-requisitos](#pré-requisitos)
  - [Instalando o ArgoCD](#instalando-o-argocd)
    - [Instalando o ArgoCD como um operador no Kubernetes](#instalando-o-argocd-como-um-operador-no-kubernetes)
  - [Instalando o ArgoCD CLI](#instalando-o-argocd-cli)
  - [Autenticando no ArgoCD](#autenticando-no-argocd)
  - [Criando a aplicação no ArgoCD](#criando-a-aplicação-no-argocd)
    - [Criando a nossa app exemplo](#criando-a-nossa-app-exemplo)
    - [Criando a app no ArgoCD usando o ArgoCD CLI](#criando-a-app-no-argocd-usando-o-argocd-cli)
    - [Primeiros passos com o ArgoCD e nossa app](#primeiros-passos-com-o-argocd-e-nossa-app)
  - [Final Day-1](#final-day-1)

&nbsp;

## O que é o ArgoCD?

O ArgoCD é uma poderoso ferramenta quando você pensa em GitOps ou Continous Delivery. O ArgoCD é um projeto open source, criado pela [Argo](https://argoproj.github.io/argo/), que tem como objetivo facilitar a implantação e gerenciamento de aplicações em Kubernetes.

O ArgoCD foi escrito em Go e utiliza o [Kubernetes Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) para gerenciar os recursos do Kubernetes.

Dito isso, fica claro que quando você instala o ArgoCD, você está instalando um operador do Kubernetes, que está extendendo o Kubernetes, adicionando novos `Custom Resources` e novos `Controllers` para gerenciar esses `Custom Resources`.

Vamos ver isso com mais detalhes mais pra frente, mas por agora saiba que o ArgoCD vai mudar a forma como você pensa e trabalha com Kubernetes.

Talvez esse seja somente o seu primeiro passo para o mundo do GitOps e do Continous Delivery, sim, entregas contínuas.

Você e sua equipe terá que se adaptar e criar maturidade para trabalhar com entregas contínuas, pois entregar é a tarefa mais fácil, o difícil é entregar com qualidade e com segurança de que não vai quebrar nada, que foi testado e que não vai afetar o negócio.

Um forte característica do ArgoCD é a sua separação de responsabilidades quando estamos falando de CI/CD. Ele não se preocupa em ser um solução completa para a sua esteira de CI/CD, ele se preocupa em ser uma ferramenta que vai gerenciar as entregas contínuas no Kubernetes, ele se preocupa com a parte CD, e não com a parte CI.

Acho que essa é uma boa apresentação do ArgoCD, e nem vou vou precisar falar que ele é peça fundamental nas melhores engenharías de software do mundo.

Está preparado para mais essa viajem com o objetivo de descomplicar mais um assunto, o ArgoCD?

#VAIIII


## O que é GitOps?

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

Eu não vou entrar em muito detalhes aqui sobre o que é o GitOps, pois teremos um repo só para isso. Mas basicamente o GitOps é uma metodologia de gerenciamento de configurações, onde o Git é a única fonte de verdade, e o Git é o único responsável por declarar o estado desejado da aplicação.

&nbsp;

## Pré-requisitos

Para que possamos continuar daqui para frente, precisamos ter o seguinte instalado:

- Um cluster Kubernetes
- kubectl instalado
- E muita vontade de aprender

## Instalando o ArgoCD

Primeira coisa, como eu falei anteriormente, o ArgoCD é escrito em GO, o que nos ajuda demais no processo de instalação.

Aqui precisamos dividir essa instalação em duas partes, a instalação do ArgoCD como um operador no Kubernetes, e a instalação do ArgoCD como CLI, para que você possa utilizar o ArgoCD no seu dia a dia.

Ele possui ainda uma interface gráfica, que é o ArgoCD UI, mas não iremos abordar por enquanto, eu quero que a gente fique antes muito confortável com o ArgoCD CLI, que é o que iremos utilizar no nosso dia a dia.

No começo ainda vamos utilizar somente o CLI, mas muito em breve vamos utilizar manifestos para definir as nossa aplicações dentro do ArgoCD.

&nbsp;

### Instalando o ArgoCD como um operador no Kubernetes

Para instalar o ArgoCD como um operador no Kubernetes, antes precisamos criar uma namespace chamada `argocd`, e para isso basta executar o seguinte comando:

```bash
kubectl create namespace argocd
```

&nbsp;

A saída desse comando será algo parecido com isso:

```bash
namespace/argocd created
```

Agora vamos instalar o ArgoCD como um operador no Kubernetes:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
&nbsp;

A saída desse comando será algo parecido com isso:

```bash
namespace/argocd created
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
```

&nbsp;

Como você pode ver, com o comando acima configuramos o ArgoCD através da criação de vários objetos no Kubernetes, como por exemplo, um `deployment` para o `argocd-server`, um `service` para o `argocd-server`, um `configmap` para o `argocd-cm`, e por aí vai.

Caso você queira conhecer mais sobre o projeto, vá até o [repositório oficial](https://github.com/argoproj/argo-cd)

&nbsp;

Vamos ver se os pods do ArgoCD foram criados com sucesso:

```bash
kubectl get pods -n argocd
```

&nbsp;

A saída desse comando será algo parecido com isso:

```bash
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          115s
argocd-applicationset-controller-5f67f4c987-vdtpr   1/1     Running   0          117s
argocd-dex-server-5859d89dcc-c69fx                  1/1     Running   0          117s
argocd-notifications-controller-75c986587-7jznn     1/1     Running   0          116s
argocd-redis-74c8c9c8c6-mzdlv                       1/1     Running   0          116s
argocd-repo-server-76f77874d7-8qscp                 1/1     Running   0          116s
argocd-server-64d5654c48-tkv65                      1/1     Running   0          116s
```

&nbsp;

Onde temos os seguintes pods:

* argocd-application-controller-0 - Responsável por gerenciar os recursos do Kubernetes
* argocd-applicationset-controller-5f67f4c987-vdtpr - Controller responsável por gerenciar os `ApplicationSets`
* argocd-dex-server-5859d89dcc-c69fx - Responsável por gerenciar a autenticação
* argocd-notifications-controller-75c986587-7jznn - Responsável por gerenciar as notificações, como por exemplo, quando um `Application` é atualizado
* argocd-redis-74c8c9c8c6-mzdlv - Responsável por armazenar os dados do ArgoCD
* argocd-repo-server-76f77874d7-8qscp - Responsável por gerenciar os repositórios
* argocd-server-64d5654c48-tkv65 - Responsável por expor a interface gráfica do ArgoCD

&nbsp;

Pronto, apresentados. No decorrer do livro iremos falar mais sobre cada um desses componentes, mas por agora é o que você precisa saber.

Todos os nossos podes estão com o status `Running`, o que significa que eles estão funcionando corretamente.

&nbsp;

## Instalando o ArgoCD CLI

Como eu falei, o ArgoCD possui uma interface gráfica, mas também é possível interagir com ele através de comandos. Para isso, precisamos instalar o `argocd` CLI.

Nós vamos focar a primeira parte desse livro no CLI, para que você consiga entender como funciona o ArgoCD por baixo dos panos, e depois sim, se delicie com a interface gráfica.

Para instalar o `argocd` CLI no Linux, basta executar o seguinte comando:

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

rm argocd-linux-amd64
```

&nbsp;

Com o comando acima fizemos o download do binário do `argocd` CLI, e o instalamos no diretório `/usr/local/bin/argocd`, para fazer a instalação utilizamos o comando `install` do Linux, que é um comando que faz a instalação de arquivos e diretórios. Passamos os parâmetros `-m 555` para definir as permissões do arquivo, e o nome do arquivo que queremos instalar.

Pronto! O nosso `argocd` CLI está instalado.

Vamos ver se ele está funcionando corretamente:

```bash
argocd version
```

&nbsp;

Qual a versão do `argocd` CLI que você está utilizando? Comenta lá no Twitter e me marca para eu saber como está sendo essa sua abordagem com o ArgoCD. @badtux_, esse é o meu arroba lá no Twitter.

&nbsp;

## Autenticando no ArgoCD

Agora que já temos o ArgoCD instalado, tanto o CLI quanto o operador, precisamos fazer a autenticação no ArgoCD para que possamos dar os primeiros passos.

Antes de mais nada, precisamos saber qual o endereço do ArgoCD. Para isso, vamos executar o seguinte comando:

```bash
kubectl get svc -n argocd
```

&nbsp;

A saída desse comando será algo parecido com isso:

```bash
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.100.164.34    <none>        7000/TCP,8080/TCP            12m
argocd-dex-server                         ClusterIP   10.100.14.112    <none>        5556/TCP,5557/TCP,5558/TCP   12m
argocd-metrics                            ClusterIP   10.100.146.115   <none>        8082/TCP                     12m
argocd-notifications-controller-metrics   ClusterIP   10.100.81.159    <none>        9001/TCP                     12m
argocd-redis                              ClusterIP   10.100.174.178   <none>        6379/TCP                     12m
argocd-repo-server                        ClusterIP   10.100.148.141   <none>        8081/TCP,8084/TCP            12m
argocd-server                             ClusterIP   10.100.25.239    <none>        80/TCP,443/TCP               12m
argocd-server-metrics                     ClusterIP   10.100.46.64     <none>        8083/TCP                     12m
```

&nbsp;

O service que precisamos por agora do ArgoCD é o `argocd-server`, e o endereço completo é `argocd-server.argocd.svc.cluster.local`.

Vamos fazer o port-forward para acessar o ArgoCD sem precisar expor:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

&nbsp;

Pronto, agora podemos acessar o ArgoCD através do endereço `localhost:8080`, tanto pelo navegador quanto pelo CLI.

Vamos continuar com a nossa saga utilizando o CLI, então vamos fazer a autenticação no ArgoCD.

Para fazer a autenticação no ArgoCD, precisamos executar o seguinte comando:

```bash
argocd login localhost:8080
```

&nbsp;

Perceba que ele irá pedir o usuário e a senha, mas não se preocupe, pois o usuário padrão do ArgoCD é o `admin`, e a senha inicial está armazenada em um secret, então vamos executar o seguinte comando para pegar a senha:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

&nbsp;

A saída será a sua senha inicial, copie ela para que possamos utilizar no próximo comando:

```bash
argocd login localhost:8080
WARNING: server certificate had error: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context 'localhost:8080' updated
```

&nbsp;

Pronto, estamos autenticados no ArgoCD. Agora vamos adicionar o nosso cluster Kubernetes ao ArgoCD.

Para isso, vamos ver qual o contexto do nosso cluster Kubernetes:

```bash
kubectl config current-context
```

&nbsp;

A saída será algo parecido com isso:

```bash
girus@eks-cluster.us-east-1.eksctl.io
```

&nbsp;

Isso no meu caso que somente estou utilizando um cluster e é um EKS, lá da AWS.

Agora vamos adicionar o nosso cluster ao ArgoCD:

```bash
argocd cluster add O_NOME_DO_SEU_CONTEXT
```

&nbsp;

No meu caso:
    
```bash
argocd cluster add girus@eks-cluster.us-east-1.eksctl.io
``` 

&nbsp;

A saída será algo parecido com isso:

```bash
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `girus@eks-cluster.us-east-1.eksctl.io` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0005] ServiceAccount "argocd-manager" created in namespace "kube-system" 
INFO[0005] ClusterRole "argocd-manager-role" created    
INFO[0005] ClusterRoleBinding "argocd-manager-role-binding" created 
Cluster 'https://F40E37CE91565CC520A53CB1B191CCCA.gr7.us-east-1.eks.amazonaws.com' added
```

&nbsp;

Caso esteja utilizando um cluster k8s no mesmo host em que está executando o kubectl, como é o que acontece quando usamos um cluster via kind ou minikube por exemplo, você pode ter o seguinte erro:

```bash
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-kind` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0020] ServiceAccount "argocd-manager" created in namespace "kube-system" 
INFO[0020] ClusterRole "argocd-manager-role" created    
INFO[0020] ClusterRoleBinding "argocd-manager-role-binding" created 
INFO[0025] Created bearer token secret for ServiceAccount "argocd-manager" 
FATA[0025] rpc error: code = Unknown desc = Get "https://127.0.0.1:32919/version?timeout=32s": dial tcp 127.0.0.1:32919: connect: connection refused
```

Para contornar esse erro execute o comando `kubectl get -n default endpoints`. A saída será algo parecido com isso:

```bash
NAME         ENDPOINTS         AGE
kubernetes   172.18.0.2:6443   103m
```

Agora copie o ip e porta que foi mostrado com a execução do comando anterior e altere somente o valor de endereço do server no seu arquivo `.kube/config`, como no exemplo abaixo onde o ip antigo foi comentado e o novo endereço foi configurado:

```yaml
apiVersion: v1
clusters:
- cluster:
    #server: https://127.0.0.1:32919
    server: https://172.18.0.2:6443
  name: kind-kind

```


Após essa modificação execute novamente o comando para adicionar o cluster ao ArgoCD

```bash
argocd cluster add O_NOME_DO_SEU_CONTEXT
```

E desta vez a saída sem erro será parecida com isso:

```bash
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-kind` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0001] ServiceAccount "argocd-manager" already exists in namespace "kube-system" 
INFO[0001] ClusterRole "argocd-manager-role" updated    
INFO[0001] ClusterRoleBinding "argocd-manager-role-binding" updated 
Cluster 'https://172.18.0.2:6443' added
```

&nbsp;

Pronto, nosso cluster foi adicionado ao ArgoCD.

Vamos confirmar se o nosso cluster foi adicionado ao ArgoCD:

```bash
argocd cluster list
```

A saída será algo parecido com isso:

```bash
SERVER                                                                    NAME                                   VERSION  STATUS   MESSAGE                                                  PROJECT
https://F40E37CE91565CC520A53CB1B191CCCA.gr7.us-east-1.eks.amazonaws.com  girus@eks-cluster.us-east-1.eksctl.io           Unknown  Cluster has no applications and is not being monitored.  
https://kubernetes.default.svc                                            in-cluster                                      Unknown  Cluster has no applications and is not being monitored. 
```

&nbsp;

Na saída, temos o nosso cluster adicionado, e o cluster local, que é o `in-cluster`, que vem por padrão.

&nbsp;

Pronto, já temos onde a nossa aplicação vai ser implantada, agora vamos criar a nossa aplicação para o ArgoCD.

&nbsp;

## Criando a aplicação no ArgoCD

Agora que já temos o nosso cluster adicionado ao ArgoCD, vamos criar a nossa aplicação. Para isso, temos que ter um repositório Git com o nosso código, e o ArgoCD vai monitorar esse repositório e vai fazer o deploy da nossa aplicação sempre que tiver uma alteração.

### Criando a nossa app exemplo

Para o nosso exemplo, vamos utilizar um repo que criem no GitHub, e o código está disponível [aqui](https://github.com/badtuxx/k8s-deploy-nginx-example). 

Eu criei esse repo somente para servir de exemplo para essa nossa primeira parte. O que temos nesse repo são somente quatro arquivos, um que define o nosso `Deployment`, outro que define o nosso `Service`, temos um que define um `ConfigMap` e outro que define um `Pod`.

O nosso `Deployment` é bem simples, ele cria um `Pod` com dois containers, um que é o `nginx` e outro que é o `nginx-exporter`, que é um container que vai expor as métricas do nginx para o Prometheus.

O nosso `Service` é bem simples também, ele expõe a porta `9113` do nosso `Pod`, que é a porta que o `nginx-exporter`.

Já o nosso `ConfigMap` é um `ConfigMap` que terá a configuração default do nginx, que é o `default.conf`.

Os arquivos são esses:

* nginx-deployment.yaml

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

* nginx-service.yaml

```yaml
apiVersion: v1 # versão da API
kind: Service # tipo de recurso, no caso, um Service
metadata: # metadados do recurso
  name: nginx-svc # nome do recurso
  labels: # labels do recurso
    app: nginx # label para identificar o svc
spec: # especificação do recurso
  ports: # definição da porta do svc 
  - port: 9113 # porta do svc
    name: metrics # nome da porta
  selector: # seletor para identificar os pods/deployment que esse svc irá expor
    app: nginx # label que identifica o pod/deployment que será exposto
```

&nbsp;

* nginx-configmap.yaml

```yaml
apiVersion: v1 # versão da API
kind: ConfigMap # tipo de recurso, no caso, um ConfigMap
metadata: # metadados do recurso
  name: nginx-config # nome do recurso
data: # dados do recurso
  nginx.conf: | # inicio da definição do arquivo de configuração do Nginx
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
      location /metrics {
        stub_status on;
        access_log off;
      }
    }
```

&nbsp;

* nginx-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers: # containers do template 
    - name: nginx-container # nome do container
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

Pronto, explicado o que temos nesse repo. Basicamente quatro manifestos que irão criar um `Deployment` com dois `Pods`, um `Service`, um `ConfigMap`, além de um `Pod`solto.

A mágica que queremos aqui é fazer com que o ArgoCD faça o deploy do nosso `Deployment` e `Service` e tudo mais que está no nosso repo, mas para isso precisamos criar um `Application` que irá fazer o deploy do nosso `Deployment` e `Service`.

### Criando a app no ArgoCD usando o ArgoCD CLI

Já sabemos o que queremos ter em nosso cluster, agora bora criar a nossa aplicação no ArgoCD com o seguinte comando:

```bash
argocd app create nginx-app --repo https://github.com/badtuxx/k8s-deploy-nginx-example.git --path . --dest-server https://F40E37CE91565CC520A53CB1B191CCCA.gr7.us-east-1.eks.amazonaws.com --dest-namespace default
```

&nbsp;


Onde:
* `nginx-app` é o nome da nossa aplicação
* `repo` é o repo onde está o nosso código
* `path` é o caminho onde está o nosso código
* `dest-server` é o cluster onde queremos fazer o deploy
* `dest-namespace` é o namespace onde queremos fazer o deploy

A saída do comando será algo como:

```bash
Application 'nginx-app' created
```

&nbsp;

### Primeiros passos com o ArgoCD e nossa app

Agora vamos ver se o nosso `Application` foi criado com sucesso:

```bash
argocd app list
```

&nbsp;

A saída do comando será algo como:

```bash
NAME     CLUSTER                                                                   NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                     PATH  TARGET
argocd/nginx-app  https://F40E37CE91565CC520A53CB1B191CCCA.gr7.us-east-1.eks.amazonaws.com  default    default  OutOfSync  Missing  <none>      <none>      https://github.com/badtuxx/k8s-deploy-nginx-example.git  . 
```

&nbsp;

Agora vamos ver o que está acontecendo com o nosso `Application`:

```bash
argocd app get nginx-app
```

&nbsp;

Ele irá retornar algo como:

```bash
Name:               argocd/nginx-app
Project:            default
Server:             https://F40E37CE91565CC520A53CB1B191CCCA.gr7.us-east-1.eks.amazonaws.com
Namespace:          default
URL:                https://localhost:8080/applications/nginx-app
Repo:               https://github.com/badtuxx/k8s-deploy-nginx-example.git
Target:             
Path:               .
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (18397fa)
Health Status:      Missing

GROUP  KIND        NAMESPACE  NAME          STATUS     HEALTH   HOOK  MESSAGE
       ConfigMap   default    nginx-config  OutOfSync  Missing        
       Pod         default    nginx-pod     OutOfSync  Missing        
       Service     default    nginx-svc     OutOfSync  Missing        
apps   Deployment  default    nginx-server  OutOfSync  Missing        
```

&nbsp;

Duas informações importantes aqui:

* O `Application` está com o status `OutOfSync` e o `Sync Status` também está `OutOfSync`;
* O `Application` está com o status `Missing` e o `Health Status` também está `Missing`.

&nbsp;

Precisa de mais alguma informação? Vamos ver o que o ArgoCD está tentando fazer:

```bash
argocd app logs nginx-app
```

&nbsp;

Ainda não temos nada, pois o ArgoCD ainda não fez nada, pois o nosso `Application` está com o status `OutOfSync` e o `Sync Status` também está `OutOfSync`, então precisamos fazer o sync do nosso `Application` para que o ArgoCD possa fazer o deploy do nosso `Deployment` e `Service`:

```bash
argocd app sync nginx-app
```

&nbsp;

A saída do comando será algo como:

```bash
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2023-03-05T19:04:16+01:00          ConfigMap     default          nginx-config  OutOfSync  Missing              
2023-03-05T19:04:16+01:00                Pod     default             nginx-pod  OutOfSync  Missing              
2023-03-05T19:04:16+01:00            Service     default             nginx-svc  OutOfSync  Missing              
2023-03-05T19:04:16+01:00   apps  Deployment     default          nginx-server  OutOfSync  Missing              
2023-03-05T19:04:17+01:00          ConfigMap     default          nginx-config    Synced  Missing              
2023-03-05T19:04:17+01:00            Service     default             nginx-svc    Synced  Healthy              
2023-03-05T19:04:17+01:00          ConfigMap     default          nginx-config    Synced   Missing              configmap/nginx-config created
2023-03-05T19:04:17+01:00            Service     default             nginx-svc    Synced   Healthy              service/nginx-svc created
2023-03-05T19:04:17+01:00                Pod     default             nginx-pod  OutOfSync  Missing              pod/nginx-pod created
2023-03-05T19:04:17+01:00   apps  Deployment     default          nginx-server  OutOfSync  Missing              deployment.apps/nginx-server created
2023-03-05T19:04:17+01:00                Pod     default             nginx-pod    Synced  Progressing              pod/nginx-pod created
2023-03-05T19:04:17+01:00   apps  Deployment     default          nginx-server    Synced  Progressing              deployment.apps/nginx-server created

Name:               argocd/nginx-app
Project:            default
Server:             https://F40E37CE91565CC520A53CB1B191CCCA.gr7.us-east-1.eks.amazonaws.com
Namespace:          default
URL:                https://localhost:8080/applications/nginx-app
Repo:               https://github.com/badtuxx/k8s-deploy-nginx-example.git
Target:             
Path:               .
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (18397fa)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      18397faeb8c9f6a10a63d3091d8021655778db7c
Phase:              Succeeded
Start:              2023-03-05 19:04:15 +0100 CET
Finished:           2023-03-05 19:04:17 +0100 CET
Duration:           2s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME          STATUS  HEALTH       HOOK  MESSAGE
       ConfigMap   default    nginx-config  Synced                     configmap/nginx-config created
       Service     default    nginx-svc     Synced  Healthy            service/nginx-svc created
       Pod         default    nginx-pod     Synced  Progressing        pod/nginx-pod created
apps   Deployment  default    nginx-server  Synced  Progressing        deployment.apps/nginx-server created
```

&nbsp;

Preste atenção nessa parte aqui:

```bash
GROUP  KIND        NAMESPACE  NAME          STATUS  HEALTH       HOOK  MESSAGE
       ConfigMap   default    nginx-config  Synced                     configmap/nginx-config created
       Service     default    nginx-svc     Synced  Healthy            service/nginx-svc created
       Pod         default    nginx-pod     Synced  Progressing        pod/nginx-pod created
apps   Deployment  default    nginx-server  Synced  Progressing        deployment.apps/nginx-server created
```

&nbsp;

Aqui ele está dizendo que o `ConfigMap` foi criado, o `Service` foi criado, o `Pod` foi criado e o `Deployment` foi criado. Parece que está tudo certo, certo? 

Vamos ver se ele criou algo no Kubernetes:

```bash
kubectl get pods
```

&nbsp;

Parece que sim hein?

```bash
NAME                            READY   STATUS    RESTARTS   AGE
nginx-pod                       2/2     Running   0          2m14s
nginx-server-6949f64b59-jc7pj   2/2     Running   0          2m14s
nginx-server-6949f64b59-l42zn   2/2     Running   0          2m14s
```

&nbsp;

Agora vamos ver se o `Service` está funcionando:

```bash
kubectl get svc
```

&nbsp;

```bash
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.100.0.1       <none>        443/TCP    59m
nginx-svc    ClusterIP   10.100.146.231   <none>        9113/TCP   2m40s
```

&nbsp;

Somente falta ver se o `ConfigMap` foi criado:

```bash
kubectl get cm
```

&nbsp;

```bash
NAME               DATA   AGE
kube-root-ca.crt   1      60m
nginx-config       1      3m13s
```

&nbsp;

Pronto, tudo funcionando.

Bora ver novamente o status do nosso `Application`:

```bash
argocd app get nginx-app
```

&nbsp;

```bash
Name:               argocd/nginx-app
Project:            default
Server:             https://F40E37CE91565CC520A53CB1B191CCCA.gr7.us-east-1.eks.amazonaws.com
Namespace:          default
URL:                https://localhost:8080/applications/nginx-app
Repo:               https://github.com/badtuxx/k8s-deploy-nginx-example.git
Target:             
Path:               .
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (18397fa)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME          STATUS  HEALTH   HOOK  MESSAGE
       ConfigMap   default    nginx-config  Synced                 configmap/nginx-config created
       Service     default    nginx-svc     Synced  Healthy        service/nginx-svc created
       Pod         default    nginx-pod     Synced  Healthy        pod/nginx-pod created
apps   Deployment  default    nginx-server  Synced  Healthy        deployment.apps/nginx-server created
```

&nbsp;

Tudo está `Synced` e `Healthy`.

E os logs?

```bash
argocd app logs nginx-app --container nginx
```

&nbsp;

```bash
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/03/05 18:04:22 [notice] 1#1: using the "epoll" event method
2023/03/05 18:04:22 [notice] 1#1: nginx/1.23.3
2023/03/05 18:04:22 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2023/03/05 18:04:22 [notice] 1#1: OS: Linux 5.4.228-132.418.amzn2.x86_64
2023/03/05 18:04:22 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/03/05 18:04:22 [notice] 1#1: start worker processes
2023/03/05 18:04:22 [notice] 1#1: start worker process 21
2023/03/05 18:04:22 [notice] 1#1: start worker process 22
```

&nbsp;

O que ele faz é trazer o log do container `nginx` do pod `nginx-pod` que foi criado pelo `Deployment` `nginx-server`, mesma coisa que se você tivesse feito:

```bash
kubectl logs -f nginx-pod -c nginx
```

&nbsp;

Simples demais!

&nbsp;

Vamos recapitular os comandos do ArgoCD:

```bash
argocd login localhost:8080 # Faz o login no ArgoCD
argocd add cluster NOME_DO_SEU_CONTEXT # Adiciona um cluster ao ArgoCD
argocd app create nginx-app --repo https://github.com/badtuxx/k8s-deploy-nginx-example.git --path . --dest-server NOME_DO_SEU_CONTEXT --dest-namespace default
argocd app list # Lista os aplicativos
argocd app get nginx-app # Mostra o status do aplicativo
argocd app logs nginx-app --container nginx # Mostra os logs do aplicativo
argocd app sync nginx-app # Sincroniza o aplicativo com o repositório
```

&nbsp;


## Final Day-1

Acho que é o que precisamos para o nosso primeiro dia de trabalho com o ArgoCD. Durante o dia de hoje você aprendeu:
*   Como instalar o ArgoCD no Kubernetes
*   Como autenticar no ArgoCD
*   Como adicionar um cluster ao ArgoCD
*   Os detalhes de como criar um deployment, service e configmap no Kubernetes
*   Como criar um aplicativo no ArgoCD
*   Como sincronizar um aplicativo no ArgoCD
*   Como ver o status de um aplicativo no ArgoCD
*   Como ver os logs de um aplicativo no ArgoCD