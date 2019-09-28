# minikube-lab

## Links úteis

* [NodeJS](https://nodejs.org/en/download/)
* [Chocolatey](https://tinyurl.com/hpaeoy8)
* [Helm](https://tinyurl.com/y4a6pnkd)
* [VirtualBox](https://tinyurl.com/5vgw4mp)
* [Kubectl Command Reference](https://tinyurl.com/yxo3qhap)
* [mongo-express](https://hub.docker.com/_/mongo-express)

## Instalando o minikube

Instalar a kubectl
```sh
$ choco install kubernetes-cli
```

Instalar o minikube
```sh
$ choco install minikube
```

Inicializar o minikube
```sh
$ minikube config set kubernetes-version v1.15.3

$ minikube start 
```

> Caso utilize o  Hyper-v
> ```sh
> $ minikube start --vm-driver hyperv --hyperv-virtual-switch "minikube_switch”
> ```

## Habilitando o Helm

Instalando o gerenciador de pacotes Helm
```sh
$ choco install kubernetes-helm
```

Inicializando o Helm e instalando o pacote tiller no minikube
```sh
$ helm init --wait
```

Adicionando o repositório de pacotes do helm
```sh
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```
## Implantando um MongoDB em ReplicaSet

Para facilitar a tarefa utilizaremos um Helm Chart o [mongodb-replicaset](https://tinyurl.com/y2cgwf9f)

Instalando o [mongodb-replicaset](https://tinyurl.com/y2cgwf9f) no minikube
```sh
$ helm install --name lab stable/mongodb-replicaset
```
Recuperando informações para conexao do mongo express

```sh
$ kubectl get pod
```

Recuperar as informações referente ao cluster
```sh
$ for ((i = 0; i < 3; ++i)); do kubectl exec --namespace default lab-mongodb-replicaset-$i -- sh -c 'mongo --eval="printjson(rs.isMaster())"'; done

...
"hosts" : [
        "lab-mongodb-replicaset-0.lab-mongodb-replicaset.default.svc.cluster.local:27017",
        "lab-mongodb-replicaset-1.lab-mongodb-replicaset.default.svc.cluster.local:27017",
        "lab-mongodb-replicaset-2.lab-mongodb-replicaset.default.svc.cluster.local:27017"
],
...
```

## Configurando o mongo-express para acesso Externo ao ReplicaSet do MongoDB
Agora vamos efetuar a implantacao de um deployment do mongo-express dentro do nosso minikube

> ### Para efetuar o replace das variaveis dentro dos templates utilizaremos o pacote npm [envsub](https://www.npmjs.com/package/envsub)
> ```sh
> $ npm i -g envsub
>```

Agora utilizando o envsub vamos efetuar a substituicao da variavel {{HOSTS_LIST}} e criar o arquivo de deployment
```sh
$ envsub --syntax handlebars --env HOSTS_LIST=lab-mongodb-replicaset-0.lab-mongodb-replicaset.default.svc.cluster.local,lab-mongodb-replicaset-1.lab-mongodb-replicaset.default.svc.cluster.local,lab-mongodb-replicaset-2.lab-mongodb-replicaset.default.svc.cluster.local mongo-express/deployment.template.yaml mongo-express/deployment.yaml
```

Agora vamos executar o deploy

```sh
$ kubectl apply -f mongo-express/deployment.yaml
```

Conferindo a saude do deploy
```sh
$ kubectl get deployment

$ kubectl describe deployment mongo-express

$ kubectl get pods -l=app=mongo-express
```

Expondo o deploy via porta do cluster
```sh
$ kubectl expose deployment mongo-express --port=8081 --type=NodePort

$ kubectl get service
```

Agora precisamos descobrir qual o ip que é exposto nosso cluster do minikube

```sh
$ minikube status

host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 172.18.70.199
```

> Para facilitar voce pode adicionar esse ip ao hosts da sua maquina

Agora verifique o acesso ao servico.