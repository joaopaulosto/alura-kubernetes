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


<b>ConfigMap</b> é um recurso onde é possível armazenar chaves e valores evitando a necessidade de informar configurações de aplicação na declaração dos Pod na área ENV.

Exemplo de ConfigMap:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-configmap
data:
  MYSQL_ROOT_PASSWORD: q1w2e3r4
  MYSQL_DATABASE: empresa
  MYSQL_PASSWORD: q1w2e3r4
```

É possível acessar as configurações atráves dos comandos:

`kubectl get configMaps` lista todos os serviços criados

`kubectl describe configMaps {nomde-do-config}` exibe as configurações do Config Map

Para utilizar o ConfigMap, na declaração do Pod é necessário utilizar o comando abaixo dentro do _container_

```
envFrom:
    - configMapRef:
        name: db-configMap
```

<b>ReplicaSet</b> É serviço que garante de forma automáticas replicas do mesmo pod rodando dentro do kubernetes. Em caso de falhas de algum deles, é carregado um novo pod para manter a configuração desejada de replicas.
Abaixo um exemplo de criação de um replicaSet

```

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: portal-noticias-replicaset
spec:
  template:
    metadata:
      name: portal-noticias
      labels:
        app: portal-noticias
    spec:
      containers:
        - name: portal-noticias-container
          image: aluracursos/portal-noticias:1
          ports:
            - containerPort: 80
          envFrom:
            - configMapRef:
                name: portal-configmap
  replicas: 3
  selector:
    matchLabels:
      app: portal-noticias

```

`kubectl get rs` lista todos os replicasets criados


<b>Deployment</b>  é um conjunto de Replicaset adicionado uma camada de controle de versão. Com deployment é possível implantar ReplicaSets de forma gerenciada com todo o histório de manutenção a partir de revisões.

Um exemplo de Deploy abaixo. Observe que mudou apenas o tipo (kind) do serviço em comparação do ReplicaSet:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx-contaider
          image: nginx:stable
          ports:
            - containerPort: 80          
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
```
`kubectl get deployment` lista todos os deployments criados


`kubectl rollout history deployment nginx-deployment` apresenta o histórico de revisão do nosso deploy  _nginx-deployment_

|REVISION  |CHANGE-CAUSE|
|----------|------------|
|1         | < none >   |


O comando a seguir `kubectl apply -f '.\Exercicios Aulas - Parte 2\deployment-nginx.yaml' --record` grava um histórico de alteração no Deployment

`kubectl rollout history deployment nginx-deployment`


|REVISION  |CHANGE-CAUSE|
|----------|------------|
|1         |< none >|
|2         |kubectl.exe apply --filename=.\Exercicios Aulas - Parte 2\deployment-nginx.yaml --record=true|

Para incluir um comentário no que foi realizado na última revisão, neste caso a 2. O comando abaixo cria um comentário acerta da alteração.

`kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Definindo a imagem com versão lastest"`

Na sequencia, temos o seguinte resultado do comando `kubectl rollout history deployment nginx-deployment`

|REVISION  |CHANGE-CAUSE|
|----------|------------|
|1         |< none >|
|2         |Definindo a imagem com versão lastest|


Se alterarmos a versão do Pod para 1 e executar novamente o comando para registar o que foi realizado `kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Definindo a imagem com versão 1"`, temos o seguinte resultado:

|REVISION  |CHANGE-CAUSE|
|----------|------------|
|1         |< none >|
|2         |Definindo a imagem com versão lastest|
|3         |Definindo a imagem com versão 1|

<b> ROLLBACK </b>
Caso seja necessário voltar uma determinada versão, não é necessário a execução do arquivo yaml com a versão desejada, basta executar o comando de rollback especificando a _revision_ `kubectl rollout undo deployment nginx-deployment --to-revision=2`

|REVISION  |CHANGE-CAUSE|
|----------|------------|
|1         |< none >|
|3         |Definindo a imagem com versão 1|
|4         |Definindo a imagem com versão lastest|





