# minikube-lab

## Links úteis

* [Chocolatey](https://tinyurl.com/hpaeoy8)
* [Helm](https://tinyurl.com/y4a6pnkd)
* [VirtualBox](https://tinyurl.com/5vgw4mp)

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
$ minikube start --kubernetes-version=v1.15.3
```

> Caso utilize o  Hyper-v
> ```sh
> $ minikube start --vm-driver hyperv --hyperv-virtual-switch "minikube_switch” --kubernetes-version=v1.15.3
> ```

## Instalando o Helm

Instalando o gerenciador de pacotes Helm
```sh
$ choco install kubernetes-helm
```

Inicializando o Helm e instalando o pacote tiller no minikube
```sh
$ helm init
```

Adicionando o repositório de pacotes do helm
```sh
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```
