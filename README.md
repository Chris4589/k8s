hacer un template deply + service con #cambiar cosas que hay que cambiar

imgen: img-mia //imagen construida localmente
imagePullPolicy: IfNotPresent

si el pod da error en el deplyment darle decribe o logs al RS porque el crea los pods

pod = contiene uno o varios contenedores = un pod es el mas pequeño
en k8s, su contenedores no deben ser modificados desde docker ya que son administrador por k8s

por cada deployment debe usar labels diferentes

los comandos se pueden hacer sobre pods replicaset, y otros
EJ
kubectl get pods
kubectl get rs
kubectl get pods -n nombre-namespáce //para sacar pods de un namespace

kubecrl api-resources 
para saber el apiVersion, Kind de los yaml

si se hace un kubectl get pods saldra el pod que contiene contenedores
y si se hace un docker ps -a saldra el/los contenedor/s manejado por k8s 
no se puede modificar un contenedor desde docker si esta manejado por k8s porque 
lo mas pequeño en k8s es un pod

kubectl run --generator=run-pod/v1 name-pod --image=nginx:alpine

* saber los pods
kubectl get pods ó kubectl get po
kubectl get pod pod-name //buscar uno X
kubectl get pod pod-name -o yaml //entrega la info del pod X

* logs
kubectl logs name-pod -f
*logs a X contenedor
kubectl logs name-pod -c contenedor-1-name

*error al crear
kubectl describe pod name-pod

* eliminar
kubectl delete pod pod1 pod2

* consola del pod
kubectl exec -ti name-post -- sh



*manifiesto kubectl

archivo yaml

pod.yaml ->

buscar un themplate de un pod yaml en google o copiar el que da kubectl get pods -o yaml

-gugle
https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

apiVersion: v1
kind: Pod <- tipo, ahora es un pod
metadata:
  name: name-pod
  labels: <---- se pueden definir los labels, se usan para diferenciar pods
    role: myrole
    nombre: chris
    app: front-end
    env dev
spec:
  containers:
    - name: contenedor-1-name
      image: nginx <- img que usara
      resources:
      	requests:
        	memory: "64Mi"
        	cpu: "250m"
      	limits:
        	memory: "128Mi"
        	cpu: "500m"
--- <--se usan los 3 guillones para crear otro pod en ese archivo
apiVersion: v1
kind: Pod <- tipo, ahora es un pod
metadata:
  name: name-pod2
  labels: <---- se pueden definir los labels, se usan para diferenciar pods
    role: myrole
    nombre: chris
    app: back-end
    env dev
spec:
  containers:
    - name: contenedor-2-name
      image: nginx <- img que usara


kubectl apply -f archivo.yaml
crea el pod

borrar
kubectl delete -f archivo.yaml

filtrar por labels -l = label
kubectl get pods -k label_name_declarado = backend

* pods con mas contenedores

archivo.yaml

apiVersion: v1
kind: Pod <- tipo, ahora es un pod
metadata:
  name: name-pod
spec:
  containers: <- 2 contenedores se comunican por IP
    - name: contenedor-1-name
      image: nginx <- img que usara
      command: ['sh', '-c', 'echo hola > index.html && py -m http.server 8082'] <- re escribir el cmd del contenedor EJECUTAR CMD CREO QUE ES
    - name: contenedor-2-name
      command: ['sh', '-c', 'echo hola > index.html && py -m http.server 8083'] <- re escribir el cmd del contenedor EJECUTAR CMD CREO QUE ES
      image: python



*** replica set ****
crea cantidadesde pods y si se borra lo crea de nuevo
siempre deben llevar labels para usar replicaset

-googlear como crear replicaset para template
https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/

apiVersion: apps/v1 <- dice apps/v1
kind: ReplicaSet <-tipo replica set
metadata:
  name: frontend <- replicaset-nombre
  labels:
    app: guestbook  <- replicaset-labels
    tier: frontend  <- replicaset-labels
spec:
  # modifica las réplicas según tu caso de uso
  replicas: 3 <- replicaset-numero de replicaciones de un pod
  selector:
    matchLabels:
      tier: frontend  <- replicaset- buscar pods por labels y si no lo encuentra los vuelva a crear
  template: <-- de aca para bajo es el pod
    metadata:
      labels:
        tier: frontend <- debe ser el mismo label que el replica set
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3


kubectl apply -f replica.yaml se puede cambiar el numero de replicas y aplicar
el cmd de nuevo y se actualiza

kubectl get replicaset o kubectl get rs

si se elimina un pod crea el faltante


* owner reference

kubectl get pod name -o yaml 
asi se ve la referencia al dueño, se copia el UID
kubectl get rs name -o yaml
UDI y el uid de pod es igual al de replicaset

* pods planos sin replicaset
no se deben crear solo pods con o sin levels, siempre deben ser por objetos de mayor level

porque si tienen un label que coincida con el del RS los va a adoptar aunque seas cosas distintas

* problemas replicaset

no se puede cambiar la imagen, version, numero de pods, 
o cambiar algo en el pod, ya que no se actualizara,

osea no puede actualizar los pods, ya que solo actualiza la configuracion del rs

***** deployment *****

replicasets por default = 10

el deployment crea un RS que este crea un POD

maxV = 25 default
maxS = 25 ''

* crear deployment

googlear 
https://kubernetes.io/es/docs/concepts/workloads/controllers/deployment/

apiVersion: apps/v1
kind: Deployment <- tipo desde api-resources
metadata: <- meta data del deployment
  namespace: EspacioDondeSeGuardara ##### ejemplo namespace
  name: name-deployment
  labels:
    app: label-deployment
spec: <- Replicaset
  replicas: 3
  selector:
    matchLabels:
      app: label-deployment
  template: <- pod
    metadata:
      labels:
        app: label-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

kubectl apply -f arch.yaml

kubectl get deployment
* para ver el estado del deployment para saber si esta correcto
kubectl rollout status deployment name-deployment

* actualizar deployment
cuando actualiza crea un RS nuevo y conserva el otro viejo 

estos si actualiza el pod 

apiVersion: apps/v1
kind: Deployment <- tipo desde api-resources
metadata: <- meta data del deployment
  name: name-deployment
  labels:
    app: label-deployment
spec: <- Replicaset
  replicas: 3
  selector:
    matchLabels:
      app: label-deployment
  template: <- pod
    metadata:
      labels:
        app: label-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:alpine <- nueva img
      - name: nginx2
        image: nginx:alpine <- nueva img

kubectl apply -f deploy.yaml y asi se actualiza todo

* eventos
kubectl describe deployment name-deploy

* revision y historial
cuando se actualiza un deployment se crea un nuevo RS y conserva el anterior

* saber versión a las cuales se pueden regresas

kubectl rollout history deployment name-deploy

* annotations

es lo que queda en el historico
	annotations
		kubernetes.io/change-cause: "cambiar version"

googlear mas opciones

* regresar a una versión old
kubectl rollout undo deployment --to-revision=3 //el tres es la version atrasada

***** servicios kubernetes *****

observa pods con X label, un servicio se crea para observar PODS el cual posee una IP fija
a la cual los usuarios consultan y este consulta a cualquier POD disponible y devuelve INFO


* crear servicio k8s

puede agregarse en el archivo del deployment con un --- debajo

apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
     app=backend <-label service
spec:
  type: ClusterIP
  selector:
    app: front <- aca va el label del POD
    department: engineering
  ports:
     - protocol: TCP
       port: 8080
       targetPort: 80 <- puerto del contenedor, si fuera una img hecha por mi seria el de expose

kubectl apply -f arch.yaml

* info del servicio

kubectl get svc -l app=front //app es el label creado en el service
ó
kubectl describe svc name-service

-> para saber la ip del servicio + puerto

* tipos de servicios mas usados en kubernetes

- ClusterIP
es interna al cluster, desde fuera no se puede ver, si quiero consumir un backend desde afuera debe ser NodePort igual que el front, pero si es ClusterIp
el backend solo existe en el cluster y solo ahi puede ser consumido

apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
     app=backend <-label service
spec:
  type: ClusterIP
  selector:
    app: front <- aca va el label del POD
    department: engineering
  ports:
     - protocol: TCP
       port: 8080
       targetPort: 80 <- puerto del contenedor, si fuera una img hecha por mi seria el de expose

- NodePort
Expone la ip fuera del cluster

apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
     app=backend <-label service
spec:
  type: NodePort
  selector:
    app: front <- aca va el label del POD
    department: engineering
  ports:
     - protocol: TCP
       port: 8080
       targetPort: 80 <- puerto del contenedor, si fuera una img hecha por mi seria el de expose


y quedaria 

ipRED:puerto_dato_port_kubernetes

- LoadBalancer
solo se puede en google cloud, aws, amazon, cualquiera que de clouds

***** frontend ej ***


apiVersion: apps/v1
kind: Deployment <- tipo desde api-resources
metadata: <- meta data del deployment
  name: name-deployment
  namespace: EspacioDondeSeGuardara ##### ejemplo namespace
  labels:
    app: label-deployment
spec: <- Replicaset
  replicas: 3
  selector:
    matchLabels:
      app: label-deployment
  template: <- pod
    metadata:
      labels:
        app: label-deployment
    spec:
      containers:
      - name: nginx
        image: img-front
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: EspacioDondeSeGuardara ##### ejemplo namespace
  labels:
     app=frond <-label service
spec:
  type: NodePort #expongo la ip fuera del cluster
  selector:
    app: front <- aca va el label del POD
    department: engineering
  ports:
     - protocol: TCP
       port: 80
       targetPort: 9000 <- puerto del contenedor, si fuera una img hecha por mi seria el de expose

kubectl apply -f archi.yaml


ip_de_la_pc:puerto

*** namespace **
es un cluster virtual
separa deploy rs pods por ambientes, es decir puedo separarlos dentro del mismo cluster y solo poder ver los que deseo
se le pueden agregar limites

kubectl get namespaces

** ver que esta creado en el namespace

kubectl get all -n name-namspace

* crear namespace

kubectl create namespace test-namespace

en yaml

apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace
  labels:
	x: asdsd

en el deployment debajo de name se agrega namespace:test-namespace

///va arriba del deployment en el yaml

kubectl apply arch.yaml

* dns namespace
con ClusterIP

curl nombre servicio . nombre name space .cluster.local //leva el .

***************** limitar pod

* limites disponibles

ram y cpu

1cpu = 1k mini cpus

*requets
cantidad de recursos de las que el pod dispone, SON GARANTIZADOS si o si


*limits
es el limite max despues de requets (si hay espacio y necesita mas llega al limite, si no hay recursos no), 
no son garantizados

va recursos debajo de la img en el pod


apiVersion: v1
kind: Pod <- tipo, ahora es un pod
metadata:
  name: name-pod
  labels: <---- se pueden definir los labels, se usan para diferenciar pods
    role: myrole
    nombre: chris
    app: front-end
    env dev
spec:
  containers:
    - name: contenedor-1-name
      image: nginx <- img que usara
      resources: //limites
      	requests:
        	memory: "64Mi"
        	cpu: "250m"
      	limits:
       		memory: "128Mi"
        	cpu: "500m"


** ver razones

kubectl describe pods
kubectl logs 

* limitar namespaces limit range

se le da politicas de limites, usos:

Si un contenedor no tiene limites limitrange le configura automaticamente segun sus politicas

* aplicar por defecto valores a pods que no tengan limites

apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace
  labels:
	x: asdsd
----
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-cpu-limit-range
  namespace: test-namespace //lo enlazamos al namespace
spec:
  limits:
  - default: //maximo
      memory: 512Mi
      cpu: 1
    defaultRequest: //minimo
      memory: 256Mi
      cpu: 0.5
    type: Container

kubectl apply -f a.yaml

kubectl limitranges name -n namespace-querido


** valores min y max de NS

apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace
  labels:
	x: asdsd
----
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-cpu-limit-range
  namespace: test-namespace //lo enlazamos al namespace
spec:
  limits:
  - max: //maximo
      memory: 512Mi
      cpu: 1
    min: //minimo
      memory: 256Mi
      cpu: 0.5
    type: Container

si el POD sobrepasa el valor del namespace entonces k8s avisa que no puede crear el POD

si el POD es menor al valor del namespace da error igual



**** resource quota

limita a nivel de namespace, puede limitar cpus y ram
limita la sumatoria de todos los recursos dentro del namespace

apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-high
    namespace: namespace nombre
  spec:
    hard:
      requests.cpu: "1000"
      requests.memory: 200Gi
      limits.cpu: "1000"
      limits.memory: 200Gi
      pods: "10"


***** Probes - diagnosticos

comando, tcp, http

* tipos

-liveness
verifica que la app funcione como debe
PRUEBA QUE SE EJECUTA CADA x TIEMPO

* liveness command

apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

TCP
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe: #liveness tcp
      tcpSocket:
        port: 8080 #en el puerto 8080
      initialDelaySeconds: 15 #espera 15s despues de crear
      periodSeconds: 20 #cada 20 s

HTTP

apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3

-rediness
si la app inicio como debe
DIAGNOSTICO QUE SE EJECUTA EN EL POD ANTES DE SALIR EN LA LISTA DEL SERVICIO, CUANDO ESTA READY YA DEVUELVE DATOS

readinessProbe: #Reddinesprobe http
          httpGet:
            path: /_status/healthz
            port: 5000
          initialDelaySeconds: 30
          timeoutSeconds: 10


-startup
SE usan para apps que demoran al iniciar
SI HAY UN STARTUP EL LIVENESS O REDINESS SE EJECUTAN CUANDO YA SE HAYA EJECUTADO EL STARTUP



* VARIABLES DE ENTORNO 
https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"


*** config map

objeto distinto a un pod que separa las configuraciones así se evita el hardcode dentro del pod
se conecta por su llave ref


kubectl configmap nombre --from-file=archivo.conf

**** secret k8s
es igual al configmap pero solo se guarda INFO sensitiva

se puede montar como ENV o VOL


********* volumenes

emptyDir -> si el POD muere la data guarda se muere

hostPath -> el contenedor guarda y consume la data el volumeMount que guarda en SU nodo, si se crea en otro nodo fallara
se usa mas es desarrollo

Cloud Volumes ->

EBS disco amazon
PD disco cloud

se le dice que volMount viene de un server aws, amazon google


PV = persisten volume
PV crea el recurso en cloud

PVC = persisten volume claim
reclama un PV y el PV crea el recurso en cloud

si eliminamos el PVC, k8s nos da 3 alternativas

retain = el pv no se elimina y guarda la data
recycle = el PV no se elimina pero si se elimina el disco guardado //peligroso usar si la data se quiere usar adelante
delete = el pv se borra y la data tambien

PV 
* hostPath

apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    mysql: ready
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual # mismo classname del otro
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi #mismo espace que el pv
  selector:
    matchLabels:
      mysql: ready
      #aws-availability-zone: usd-east-l

https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

containers:
    - name: config-volume #cualquier name igual al de abajo
      mountPath: /etc/config #carpeta donde se monta el volumen

volumes:
    - name: cualquier name
      persistentVolumeClaim:
        ClaimName: config-volume 

-- PVC
pueden crear un PV automatico
    quitando esto

storageClassName: manual # mismo classname del otro, el cloud se usa mucho esto


* tipos de reclaim

kubectl edit pv name

y se edita a recycle





***** roles

* rol 

solo aplica al NS

role binding

* cluster role

se puede acceder a todos los NS

cluster role binding



*** exponer app INGRESS

luego de NodePort y LoadBalancer va ingress

esta nginx y cloud

https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/cloud/deploy.yaml


regla despues de instaklar lo anterior

https://kubernetes.io/docs/concepts/services-networking/ingress/

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test #nombre del servicio
            port:
              number: 80 #puerto


las reglas se pueden crear por host algo.host.com y por path 666/path



**** HPA

autoescala cuando hay mucho trafico y crea mas pods para repartir el consumo

tiene un min y max de servicios

Se crea en base al cpu

**** CLOSTER AUTO ESCALER

crea maquinas si los pods sobrepasan los limites

tiene un min y max de makinas