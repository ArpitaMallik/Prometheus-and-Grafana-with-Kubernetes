# Flask Monitoring with Kubernetes (Minikube), Prometheus and Grafana

This project deploys a Flask application on Minikube and monitors it using Prometheus and Grafana via Helm.

## Prerequisites

* Docker
* Minikube
* kubectl
* Helm

## Deploy Flask Application

```bash
docker build -t flask-metrics-app:latest .
minikube start
minikube image load flask-metrics-app:latest
kubectl apply -f flask-app.yaml
minikube service flask-metrics-app --url
```

## Install Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
```

Access Prometheus:

```bash
kubectl port-forward -n monitoring svc/prometheus-server 9090:80
```

## Configure Scraping

Edit ConfigMap:

```bash
kubectl edit configmap prometheus-server -n monitoring
```

Add:

```yaml
scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['<FLASK_SERVICE_IP>:5000']
```

Restart:

```bash
kubectl rollout restart deployment prometheus-server -n monitoring
```

## Install Grafana

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana -n monitoring
```

Access Grafana:

```bash
kubectl port-forward svc/grafana -n monitoring 3000:80
```

Get admin password:

```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}"
```

## Connect Prometheus to Grafana

* URL:

  ```
  http://prometheus-server.monitoring.svc.cluster.local:80
  ```

## Result

* Flask app running on Minikube
* Prometheus scraping metrics
* Grafana visualizing metrics
