<p align="left"> <a href="https://www.docker.com/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-original-wordmark.svg" alt="docker" width="40" height="40"/> </a> <a href="https://grafana.com" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/grafana/grafana-icon.svg" alt="grafana" width="40" height="40"/> </a> <a href="https://kubernetes.io" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg" alt="kubernetes" width="40" height="40"/> </a> <a href="https://www.linux.org/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/linux/linux-original.svg" alt="linux" width="40" height="40"/> </a> <a href="https://www.python.org" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python" width="40" height="40"/> </a> </p>



# Challenge-4

## Comandos utilizados.

Guardo configuración del cluster para poder conectarme

```bash
cat whitestackchallenge.yaml >> .kube/config
```

Me conecto al cluster.

```bash
kubectx whitestackchallenge
```

Listar servicios

```bash
kubectl get svc
```
------------------

## Despliegue

**Clonar el repo del challenge.**

```bash
git clone https://github.com/whitestack/ws-challenge-4.git
```

**Agregar contador en app.py.** 

```bash
from bottle import Bottle, request, response
import time
from flask import Flask, jsonify, request
from prometheus_client import make_wsgi_app, Counter

app = Bottle()

# Defino contador
requests_counter = Counter('requests_total', 'Total number of requests', ['path'])

@app.post('/heavywork')
def heavywork():
    response.status = 202
    # Incremento contador
    requests_counter.labels(path='/heavywork').inc()
    return {"message": "Heavy work started"}


@app.post('/lightwork')
def lightwork():
    return {"message": "Light work done"}

# Ruta para las métricas de Prometheus
app.mount('/metrics', make_wsgi_app())

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

**Crear la imágen docker.** 

```bash
docker build -t eduffaut/web-app:latest .
```

**Subo imágen a Docker Hub.** 

```bash
docker push eduffaut/web-app:latest
```

**Cluster utilizado para el despliegue - MINIKUBE.**

```bash
minikube start
```

**Instalación de Prometheus con Helm en namespace monitoring.**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add kube-prometheus-stack https://prometheus-community.github.io/helm-charts
helm repo update
```

**Chequeo que los deployments se crearon correctamente.**

```bash
kubectl get all -n monitoring
NAME                                       READY   STATUS    RESTARTS       AGE
pod/alertmanager-main-0                    2/2     Running   6 (2d8h ago)   4d
pod/alertmanager-main-1                    2/2     Running   6 (2d8h ago)   4d
pod/alertmanager-main-2                    2/2     Running   6 (2d8h ago)   4d
pod/blackbox-exporter-7dbfd945cb-tvzlm     3/3     Running   9 (2d8h ago)   4d
pod/grafana-9b8b4b9d-sx98f                 1/1     Running   3 (2d8h ago)   4d
pod/kube-state-metrics-7c6ff549b7-qmhbr    3/3     Running   10 (8h ago)    4d
pod/node-exporter-mq9r9                    2/2     Running   6 (2d8h ago)   4d
pod/prometheus-adapter-57854964cf-4kdts    1/1     Running   5 (8h ago)     3d7h
pod/prometheus-adapter-57854964cf-75b4j    1/1     Running   5 (8h ago)     3d7h
pod/prometheus-k8s-0                       2/2     Running   6 (2d8h ago)   4d
pod/prometheus-k8s-1                       2/2     Running   6 (2d8h ago)   4d
pod/prometheus-operator-79569d9ccc-rnkpv   2/2     Running   7 (8h ago)     4d

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-main       ClusterIP   10.99.78.166     <none>        9093/TCP,8080/TCP            4d
service/alertmanager-operated   ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   4d
service/blackbox-exporter       ClusterIP   10.110.44.39     <none>        9115/TCP,19115/TCP           4d
service/grafana                 ClusterIP   10.105.116.202   <none>        3000/TCP                     4d
service/kube-state-metrics      ClusterIP   None             <none>        8443/TCP,9443/TCP            4d
service/node-exporter           ClusterIP   None             <none>        9100/TCP                     4d
service/prometheus-adapter      ClusterIP   10.100.101.116   <none>        443/TCP                      4d
service/prometheus-k8s          ClusterIP   10.99.202.254    <none>        9090/TCP,8080/TCP            4d
service/prometheus-operated     ClusterIP   None             <none>        9090/TCP                     4d
service/prometheus-operator     ClusterIP   None             <none>        8443/TCP                     4d

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/node-exporter   1         1         1       1            1           kubernetes.io/os=linux   4d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blackbox-exporter     1/1     1            1           4d
deployment.apps/grafana               1/1     1            1           4d
deployment.apps/kube-state-metrics    1/1     1            1           4d
deployment.apps/prometheus-adapter    2/2     2            2           4d
deployment.apps/prometheus-operator   1/1     1            1           4d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/blackbox-exporter-7dbfd945cb     1         1         1       4d
replicaset.apps/grafana-9b8b4b9d                 1         1         1       4d
replicaset.apps/kube-state-metrics-7c6ff549b7    1         1         1       4d
replicaset.apps/prometheus-adapter-57854964cf    2         2         2       3d7h
replicaset.apps/prometheus-adapter-85cff7b5f8    0         0         0       4d
replicaset.apps/prometheus-adapter-9d44c4c87     0         0         0       3d9h
replicaset.apps/prometheus-operator-79569d9ccc   1         1         1       4d

NAME                                 READY   AGE
statefulset.apps/alertmanager-main   3/3     4d
statefulset.apps/prometheus-k8s      2/2     4d
```

**Instalación de Prometheus-Adapter con Helm y ConfigMap personalizado en namespace apps para leer métrica requests_total.**

```bash
helm install -f prometheus-adapter/values.yaml prometheus-adapter prometheus-community/prometheus-adapter
```

**Chequeo que los deployments se crearon correctamente.**

```bash
kubectl get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/prometheus-adapter-5b48d9d766-jvjj5   1/1     Running   0          5h22m
pod/web-app-69664b659-hdwfj               1/1     Running   0          7h51m

NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/prometheus-adapter   ClusterIP   10.101.243.69   <none>        443/TCP          5h22m
service/web-app-svc          NodePort    10.99.92.39     <none>        8080:31775/TCP   7h50m

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-adapter   1/1     1            1           5h22m
deployment.apps/web-app              1/1     1            1           7h51m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-adapter-5b48d9d766   1         1         1       5h22m
replicaset.apps/web-app-69664b659               1         1         1       7h51m

NAME                                                     REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/web-app-hpa-object   Deployment/web-app   0/10      1         5         1          5h27m
```

**Aplico todos los manifest necesarios para el despliegue de la solución.**

```bash
kubectl apply -f ./manifests
```

**Estado del HPA sin carga.**

```bash
kubectl get hpa
NAME                 REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web-app-hpa-object   Deployment/web-app   0/10      1         5         1          5h28m
```

# Demo.

[![asciicast](https://asciinema.org/a/1kq3eMb3r9UDNNwtcCojCFeTk.svg)](https://asciinema.org/a/1kq3eMb3r9UDNNwtcCojCFeTk)






