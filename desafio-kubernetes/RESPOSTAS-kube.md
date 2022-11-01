# Desafio 5: Kubernetes

> Proposta de resolução para o Desafio 5 (Kubernetes) do programa Formando DevOps - GetUp
> 

## Respostas
Segue abaixo os prints/comandos com as respostas para os exercícios:


1- 

```yaml
kubectl logs --namespace meusite -l app=ovo,run=serverweb | grep -i "erro"
```

2 - Utilizou-se o recurso daemonset (executa pods em todos os nodes) e o toleration

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: meu-spread
spec:
  selector:
    matchLabels:
      app: meu-spread
  template:
    metadata:
      labels:
        app: meu-spread
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        resources: {}
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: NoSchedule
```

3-

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: meu-webserver
  name: meu-webserver
spec:
  replicas: 1
  selector:
    matchLabels:
      app: meu-webserver
  template:
    metadata:
      labels:
        app: meu-webserver
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: volume-nginx
      initContainers:
      - image: alpine
        name: alpine-create-index
        command:
          - "/bin/sh"
          - "-c"
          - "echo HelloGetup > /app/index.html"
        volumeMounts:
        - mountPath: /app
          name: volume-nginx
      volumes:
      - name: volume-nginx
        emptyDir: {}
```

4- 

Inicialmente, foi adicionado ao node master a label abaixo:

```yaml
kubectl label nodes meuk8s-control-plane dedicated=master
```

Em seguida, foi utilizado o seguinte manifesto

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: meuweb
  name: meuweb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: meuweb
  template:
    metadata:
      labels:
        app: meuweb
    spec:
      nodeSelector:
        dedicated: master
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - image: nginx:1.16
        name: nginx
```

5 - 

```yaml
kubectl set image deployment meuweb nginx=nginx:1.19
```

6 - 

```yaml
# Add repositorio e sincronizando
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Instalando com os parametros desejados
helm install my-ingress-nginx ingress-nginx/ingress-nginx --version 4.3.0 --set controller.hostPort.enabled=true,controller.service.type=NodePort,controller.updateStrategy.type=Recreate
```

7 

```yaml
kubectl create deployment --image nginx:1.11.9-alpine pombo --replicas 4

kubectl set image deployment/pombo nginx=nginx:1.16 --record

kubectl set image deployment/pombo nginx=nginx:1.19 --record

kubectl rollout history deployment/pombo

kubectl rollout undo deployment pombo --to-revision 1
```

8 -

```yaml
## Criando o deployment
kubectl create deployment guardaroupa --image redis

## Criando o serviço do tipo ClusterIP
kubectl expose deployment guardaroupa --type ClusterIP --port 6379
```

9 - 

```yaml
## Criacao do namespace backend
kubectl create namespace backend
```

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: meusiteset
  namespace: backend
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: www-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

10 -

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: balaclava
  namespace: backend
  labels:
    backend: balaclava
    minhachave: semvalor
spec:
  replicas: 2
  selector:
    matchLabels:
      backend: balaclava
      minhachave: semvalor
  template:
    metadata:
      labels:
        backend: balaclava
        minhachave: semvalor
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379
```

11 -

```yaml
kubectl get service -o wide | grep LoadBalancer
```

12 -

```yaml
## Criando o namespace
kubectl create namespace segredosdesucesso

## Criando a secret
kubectl create secret --namespace segredosdesucesso generic meusegredo --from-literal segredo=azul --from-file chave-secreta
```

13 -

```yaml
## Criando o namespace
kubectl create namespace site

## Criando o configmap
kubectl create configmap configsite --namespace site --from-literal index.html=Hugo
```

14 -

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meudeploy
  namespace: segredosdesucesso
spec:
  selector:
    matchLabels:
      app: meudeploy
  template:
    metadata:
      labels:
        app: meudeploy
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
          - mountPath: /app
            name: meu-secret
        ports:
        - containerPort: 80
      volumes:
        - name: meu-secret
          secret:
            secretName: meusegredo
```

15 -

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: depconfigs
  namespace: site
spec:
  selector:
    matchLabels:
      app: depconfigs
  template:
    metadata:
      labels:
        app: depconfigs
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
          - mountPath: /usr/share/nginx/html
            name: html-index
        ports:
        - containerPort: 80
      volumes:
      - name: html-index
        configMap:
          name: configsite
```

16 -

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meudeploy-2
  namespace: segredosdesucesso
  labels:
    chaves: secretas
spec:
  selector:
    matchLabels:
      app: meudeploy-2
  template:
    metadata:
      labels:
        app: meudeploy-2
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        envFrom:
          - secretRef:
              name: meusegredo
```

17 - Obs: nao foi solicitado a criacao do deploy nem da secret no namespace cabeludo.

```yaml
## Criando namespace cabeludo
kubectl create namespace cabeludo

## Deploy
kubectl create deploy cabelo --image nginx:latest

## Secret
kubectl create secret generic acesso --from-literal username=pavao --from-literal password=asabranca

# Expondo variaveis de ambiente
export USUARIO=$(kubectl get secret acesso -o jsonpath="{.data.username}" | base64 --decode; echo)

export SENHA=$(kubectl get secret acesso -o jsonpath="{.data.password}" | base64 --decode; echo)
```

18 -

```yaml
apiVersion: apps/v1   
kind: Deployment      
metadata:
  labels:
    app: redis        
  name: redis
  namespace: cachehits
spec:
  replicas: 1
  selector:
    matchLabels:      
      app: redis      
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis
        name: redis
        ports:
        - containerPort: 6379
        volumeMounts:
        - mountPath: /data/redis
          name: app-cache
      volumes:
      - name: app-cache
        emptyDir: {}
```

19 -

```yaml
kubectl scale deployment basico --namespace azul --replicas=10
```

20 -

```yaml
kubectl autoscale deployment site --namespace frontend --cpu-percent=90 --min=2 --max=5
```

21 - 

```yaml
kubectl get secret piadas --namespace meussegredos -o jsonpath="{.data.segredos}" | base64 --decode; echo
```

22 - Realizar o taint do 

```yaml
kubectl taint nodes k8s-worker1 key1=value1:NoSchedule
```

23 -

```yaml
kubectl drain k8s-worker1 
```

24 -

É possível através da criação de um Pod Estático, salvando o arquivo no diretório `/etc/kubernetes/manifests` (se configuração default)

25 -

```yaml
kubectl create serviceaccount userx --namespace developer
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: userx-role
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - pods/log
  verbs:
  - '*'
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - '*'

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: userx-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: userx-role
subjects:
- kind: ServiceAccount
  name: userx
  namespace: developer
```

```yaml
## Validacao

kubectl auth can-i {get,list,watch,create,delete,update,patch} {pods,pods/log,deployments} --namespace developer --as system:serviceaccount:developer:userx
```

26 -

27 -

```yaml
kubectl get componentstatuses
```
