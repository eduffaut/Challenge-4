# Challenge-4

## Comandos utilizados

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

**Clonar el repo del challenge**

```bash
git clone https://github.com/whitestack/ws-challenge-4.git
```

**Agregar contador en app.py** 

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

**Crear la imagen docker** 

```bash
docker build -t eduffaut/web-app:latest .
```

**Subo imagen a Docker Hub** 

```bash
docker push eduffaut/web-app:latest
```

** Cluster utilizado para el despliegue - MINIKUBE**

```bash
minikube start
```

** Instalación de Prometheus con Helm en namespace monitoring**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add kube-prometheus-stack https://prometheus-community.github.io/helm-charts
helm repo update
```

** Instalación de Prometheus-Adapter con Helm y ConfigMap personalizado en namespace apps para leer métric requests_total**

```bash
helm install -f prometheus/values.yaml prometheus-adapter prometheus-community/prometheus-adapter
```

** Aplico todos los manifest necesarios para el despliegue de la solución **

```bash
kubectl apply -f ./manifests
```






