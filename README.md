# Artefatos do curso Kubernetes na platorma Alura

# Arquivo exemplo YAML para criação de pods de forma declarativa

```
apiVersion: v1
kind: Pod
metadata:
    name: primeiro-pod-declarativo
spec:
    containers:
        - name: nginx-container
          image: nginx:latest
```



# comandos do Kuberntes

`kubectl apply -f nome-arquivo.yaml` -> cria um arquivo com base no arquivo de definição

`kubectl get pods` -> lista os pods instalados

|NAME           | READY|     STATUS|       RESTARTS|      AGE|
|---------------|------|-----------|---------------|---------|
|portal-noticias|   1/1|    Running|    9 (44m ago)|    6d23h|

`kubectl get pods -o  wide` -> lista os pods com mais informações

|NAME              |READY   |STATUS    |RESTARTS      |AGE     |IP          |NODE             |NOMINATED NODE   |READINESS GATES|
|------------------|--------|----------|--------------|--------|------------|-----------------|-----------------|---------------|
|portal-noticias   |1/1     |Running   |9 (45m ago)   |6d23h   |10.1.0.69   |docker-desktop   |<none>           |<none>         |


`kubectl delete pod -f nome-arquivo.yaml` -> deleta um pod com base no arquivo de configuração


<b>SVC - Service</b> permite alocar um IP fixo para os PODS, assim quando eles são reciclados ou substituidos, garante a comunicação entre os demais. Além disso fornece serviço de LoadBalance, NodePOrt e ClusterIP


`kubectl describe pod {nomde-do-pod}` exibe as configurações do Pod

`kubectl get services` lista todos os serviços criados

<b>NodePort</b> Permite expor Pods para fora do Cluter sendo acesso via IP (Do host) e Porta definida.

No exemplo foi criado o serviço `svc-pod-1` por meio do arquivo `svc-pod-1.yaml` onde foi definido a porta de entrada externa 31000 e a saida interna 80, como mostra na lista de serviço `kubectl get svc` 

|NAME                     |TYPE           |CLUSTER-IP      |EXTERNAL-IP   |PORT(S)        |AGE | 
|-------------------------|---------------|----------------|--------------|---------------|----|
|svc-pod-1                |NodePort       |10.101.165.35   |<none>        |<b>80:31000/TCP</b>   |23m |


<b>LoadBanlancer</b> também é um NodePort e um CluterIP e é capaz a de interagir com um balanceador de Cloud Provider. Exemplo no arquivo `svc-pod-1-loadbalabcer.yaml`
