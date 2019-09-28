# minikube-lab

## Links úteis

* [NodeJS](https://nodejs.org/en/download/)
* [Chocolatey](https://tinyurl.com/hpaeoy8)
* [Helm](https://tinyurl.com/y4a6pnkd)
* [VirtualBox](https://tinyurl.com/5vgw4mp)
* [Kubectl Command Reference](ttps://tinyurl.com/yxo3qhap)
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

