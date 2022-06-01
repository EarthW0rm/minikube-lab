# KUBERNETES DEVELOPMENT LAB

## O que é o kubernetes?
* Kubernetes (K8s) é um sistema de orquestração de containers open-source que automatiza a implantação, o dimensionamento e a gestão de aplicações em containers. 
* Ele foi originalmente projetado pelo Google e agora é mantido pela Cloud Native Computing Foundation.
* A palavra "Kubernetes" vem do Grego (κυβερνήτης -kyvernítis) que significa Timoneiro, Comandante.
Kubernetes v1.0, foi lançado em 21 de julho de 2015.

---
## Objetos principais do K8s
* [POD](https://kubernetes.io/docs/concepts/workloads/pods/pod/) – um grupo de um ou mais containers.
* [SERVICE](https://kubernetes.io/docs/concepts/services-networking/service/) – expõe seu pods por uma rede interna ou externa.
* [DEPLOYMENT](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) – gera os pods, configura e os mantém vivos.
* [NAMESPACE](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) - é um mecanismo para isolar grupos de recursos
* [NODE](https://kubernetes.io/docs/concepts/architecture/nodes/) – uma VM ou maquina física que compõe o cluster do kubernetes.
* [CONFIGMAP](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) – objeto responsável por guardar configurações para execução dos pods

---
## Pré-requisitos para o lab
* [NodeJS](https://nodejs.org/en/download/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [minikube - *selecionar o driver compatível com seu sistema*](https://minikube.sigs.k8s.io/docs/drivers/)


## Instalando o minikube

O processo de instalação do minikube vai depender do driver que optar para instalação, a lista de drivers pode ser encontrada [nesse link](https://minikube.sigs.k8s.io/docs/drivers/)

> Para quem utiliza MAC M1, uma alternativa e utilizar em conjunto com o [driver do Podman](https://minikube.sigs.k8s.io/docs/drivers/podman/) (Experimental), nesse lab vamos utilizar essa opção.

---
## Inicializando o Podman Machine
```sh
podman machine init --cpus=2 -m=5000

podman machine start

podman machine set --rootful
```

Informando o parametro `--cpus` como 2, vamos dispor uma vm podman com 2 cores que é o valor mínimo para execução do minikube nesse ambiente

Para evitar problemas de compatibilidade, também vamos definir a propriedade `--rootful`, isso permite o acesso root pelos containers

---
## Inicializando o minikube
Inicializar o minikube
```sh
minikube start --driver=podman --insecure-registry="ip:5000"
```

---
## Verificando se o cluster foi criado
```sh
kubectl get nodes

kubectl get pods --namespace kube-system
```

---
## LET'S CODE
![alt](https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif)

---
## Criando um namespace
Vamos criar um namespace para separar os componentes do nosso software

```sh
kubectl create namespace todo-app
```

---
## Implantando uma instancia de MongoDB

Vamos aplicar nosso primeiro deployment, vamos iniciar uma instancia do MongoDB para ser utilizada em conjunto com o TODO App que será implementado

```sh
kubectl apply --namespace todo-app -f mongodb/deployment.yaml

kubectl get pod --namespace todo-app
```

Agora, já temos nosso container com o MongoDB rodando, mas isso nao é suficiente precisamos expor uma entrada para esse POD na porta 27017.

```sh
kubectl expose deployment mongodb --namespace=todo-app --port=27017 --type=ClusterIP --cluster-ip=None
```

Também podemos utilizar um arquivo yaml para criar uma Service, utilizando o `kubectl apply`.

```sh
kubectl apply --namespace todo-app -f mongodb/service.yaml
```

Verificando os serviços criados
```sh
kubectl get service --namespace=todo-app
```

---
## Utilizando o kubectl port-forward
Em alguma situações vamos precisar expor portas de nossas aplicações dentro do orquestrador, para isso podemos utilizar o `kubeclt port-forward`.
Vamos executar o comando e conectar no nosso MongoDB que foi criando dentro do cluster.

```sh
kubectl port-forward svc/mongodb-svc 27017:27017 --namespace=todo-app
```

Pronto, agora o deploy do nosso mongoDB está exposto na nossa porta 27017.

> Para outros drivers do minikube é possivel utilizar o `minikube service <service-name> --url`

> A connection string que vamos utilizar é: mongodb://root:techtalk@127.0.0.1:27017

---

## Configurando o mongo-express para acesso Externo ao MongoDB
Agora vamos efetuar a implantacao de um deployment do mongo-express dentro do nosso minikube

```sh
kubectl apply -f mongo-express/deployment.yaml --namespace=todo-app
```

---
### Conferindo a saude do deploy do mongo-express
```sh
kubectl get deployment --namespace=todo-app

kubectl describe deployment mongo-express --namespace=todo-app

kubectl get pods -l=app=mongo-express --namespace=todo-app
```

### Expondo o deploy utilizando o LoadBalance do minikube
Com o minikube também podemos simular serviços do tipo LoadBalance, serviços esses que podem ser responsáveis por expor seus aplicativos para fora do cluster

> Nativamente a aplicação expõe a porta 8081

```sh
kubectl expose deployment mongo-express --type=LoadBalancer --port=27000 --target-port=8081 --namespace=todo-app

kubectl get service --namespace=todo-app
```

Vamos inicializar o [tunel no minikube](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-tunnel)

```sh
minikube tunnel
```

---
### Agora verifique o acesso ao serviço.

* [http://localhost:27000/](http://localhost:27000/)

![alt](https://media.giphy.com/media/dIsw2AfNMSC1W/giphy.gif)

---
## Criando a imagem de container da aplicação Backend

Agora vamos disponibilizar dentro de nosso cluster uma Webapi para gerenciamento de tarefas backend-api

>O Código fonte dessa aplicacao esta na pasta source-files/backend/

>Nativamente a aplicação expõe a porta 3003

O Primeiro passo é efetuar o build da nossa imagem, nesse lab vamos utilizar o podman.

```sh
podman build -t minikube-lab-backend  -f Dockerfile source-files/backend/

podman image ls
```

Agora vamos exportar um .tar dessa imagem para carregar dentro da VM do minikube

```sh
podman save --format=docker-archive --output=minikube-lab-backend.tar localhost/minikube-lab-backend
```

Carregando a imagem na VM do minkube

```sh
minikube image load minikube-lab-backend.tar

minikube image ls
```

---
## Implantando a aplicação Backend
Vamos executar o apply para executar o deployment.yaml e do service.yaml

```sh
kubectl apply -f backend/deployment.yaml --namespace=todo-app

kubectl apply -f backend/service.yaml --namespace=todo-app
```

Verificando a saúde do deploy
```sh
kubectl describe deployment backend-api --namespace=todo-app

kubectl get pods -l=app=backend-api --namespace=todo-app
```

Para validar a exposicao da api podemos utilizar um `kubectl port-forward` executar um cURL

```sh
kubectl port-forward svc/backend-api-svc 3003:3003 --namespace=todo-app
```

```sh
curl --request POST \
  --url http://localhost:3003/api/todos \
  --header 'Content-Type: application/json' \
  --data '{"description": "Participar do kubernetes - development lab"}'

curl -X GET http://localhost:3003/api/todos
```

---
![Its Alive](https://media.giphy.com/media/RPwrO4b46mOdy/giphy.gif)

---
## Implementando o proxy para o backend-api usando o NGINX

No processo anterior vimos que o backend-api foi exporto por um serviço interno podemos utilizar o nginx para expor esse endpoint utilizando o LoadBalance.

Primeiro vamos implantar nosso ConfigMap que contém o arquivo de configuração do nginx

```sh
kubectl apply -f backend-proxy/configmap.yaml --namespace=todo-app
```

Agora podemos levantar o deployment

```sh
kubectl apply -f backend-proxy/deployment.yaml --namespace=todo-app
```

Verificando a saúde do deploy
```sh
kubectl describe deployment backend-proxy --namespace=todo-app

kubectl get pods -l=app=backend-proxy --namespace=todo-app
```

Com tudo pronto, vamos levantar o serviço LoadBalance e inicializar nosso tunnel no minikube

```sh
kubectl apply -f backend-proxy/service.yaml --namespace=todo-app

minikube tunnel
```

```sh
curl --request POST \
  --url http://localhost:8080/v1/backend-api/todos \
  --header 'Content-Type: application/json' \
  --data '{"description": "Colocar imagens engraçadas no treinamento"}'

curl -X GET http://localhost:8080/v1/backend-api/todos
```

---
## Criando a imagem de container da aplicação Frontend

Chegamos ao passo final, implantar o aplicativo de interface do usuário.
>O Código fonte dessa aplicacao esta na pasta source-files/frontend/

```sh
podman build -t minikube-lab-frontend  -f Dockerfile source-files/frontend/

podman image ls
```

Agora vamos exportar um .tar dessa imagem para carregar dentro da VM do minikube

```sh
podman save --format=docker-archive --output=minikube-lab-frontend.tar localhost/minikube-lab-frontend
```

Carregando a imagem na VM do minkube

```sh
minikube image load minikube-lab-frontend.tar

minikube image ls
```

## Implantando a aplicação Frontend
Agora vamos executar os applys necessários para o app frontend

```sh
kubectl apply -f frontend/configmap.yaml --namespace=todo-app

kubectl apply -f frontend/deployment.yaml --namespace=todo-app

kubectl apply -f frontend/service.yaml --namespace=todo-app

```

Inicialize o minikube tunnel, e teste seu TODO APP

```sh
minikube tunnel
```

> Acesse pelo link [http://localhost/](http://localhost/)


![Congrats](https://media.giphy.com/media/g9582DNuQppxC/giphy.gif)

---
## Ainda não acabou, hora de limpar a bagunça

```sh
minikube stop

minikube delete
```

```sh
podman machine stop

podman machine rm
```