# cicd
## jenkins
http://localhost/jenkins

## gitlab
http://localhost/gitlab
ssh://git@localhost:root

# alicloud login
https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fecs.console.aliyun.com%2F%3Fspm%3D5176.smartservice_service_robot-chat.recommends.decs.67c0709axRgJth#/home

# alicloud terrafrom 


# why can't we use the minikube
https://github.com/kubernetes/ingress-nginx/issues/6335

# all in one
[github mirror](https://gitee.com/easzlab/kubeasz.git)
[all in one guide](https://github.com/easzlab/kubeasz/blob/master/docs/setup/quickStart.md)


# helm installation for prometheus
## guide in kubeasz
[kubeasz guide](https://github.com/easzlab/kubeasz/blob/master/docs/guide/prometheus.md)
```bash
vi /etc/kubeasz/clusters/default/config.yml
#prom_install
#name_space
docker exec -it kubeasz ezctl setup default 07
```
[official github](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm pull prometheus-community/kube-prometheus-stack
kubectl create namespace monitoring
helm upgrade -i prometheus -n monitoring prometheus-community/kube-prometheus-stack
```

# renewal one docker image
```bash
docker pull bitnami/kube-state-metrics:2.0.0
docker tag d38a k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.0.0
docker rmi quay.io/coreos/kube-state-metrics:v1.9.8
```

# efk for logging
https://github.com/easzlab/kubeasz/blob/master/docs/guide/efk.md

# install nginx-ingress for L7 LoadBalancer
[official helm installation](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/)
[china mirror site of github](https://gitee.com/mirrors_nginxinc/kubernetes-ingress?_from=gitee_search)
[L7 loadbalancer](https://medium.com/codecademy-engineering/kubernetes-nginx-and-zero-downtime-in-production-2c910c6a5ed8)
```
git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/deployments/helm-chart
git checkout v1.11.3
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install gene-ingress nginx-stable/ingress-nginx -n gene
```

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# install OPs binary
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
https://github.com/helm/helm/releases

# harbor installtion
https://ruzickap.github.io/k8s-harbor/#lab-architecture
https://github.com/goharbor/harbor-helm


```
helm repo add harbor https://helm.goharbor.io
helm repo update
helm upgrade my-harbor harbor/harbor --set persistence.enabled=false --set externalURL=${redacted}:30002 --set expose.tls.enabled=false --set expose.type=nodePort
```

```
default       harbor                                               NodePort       10.68.41.131    <none>        80:30002/TCP,4443:30004/TCP    19m
kube-system   kubernetes-dashboard                                 NodePort       10.68.215.181   <none>        443:30591/TCP                  13h
monitor       prometheus-grafana                                   NodePort       10.68.174.42    <none>        80:30903/TCP                   11h
monitor       prometheus-kube-prometheus-alertmanager              NodePort       10.68.148.56    <none>        9093:30902/TCP                 11h
monitor       prometheus-kube-prometheus-operator                  NodePort       10.68.78.168    <none>        443:30900/TCP                  11h
monitor       prometheus-kube-prometheus-prometheus                NodePort       10.68.87.18     <none>        9090:30901/TCP                 11h
default       nginx-ingress-nginx-ingress                          LoadBalancer   10.68.87.86     <pending>     80:31330/TCP,443:32537/TCP     12h
```

# harbor
http://redacted:30002/
kubectl get secret my-harbor-harbor-core -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}"|base64 -d
``
Harbor12345
``

# dashboard
https://redacted:30591/#/login

``
redacted
``

kubectl get sa dashboard-read-user -n kube-system -o jsonpath="{.secrets[0].name}"
kubectl get secret dashboard-read-user-token-j7w6f -n kube-system -o jsonpath="{.data.token}|base64 -d"

# prometheus
http://redacted:30901/graph
http://redacted:30902/#/alerts
# grafana
http://redacted:30903/
kubectl get secret -n monitor prometheus-grafana -o jsonpath="{.data.admin-password}"|base64 -d
``
Admin1234!
``


# digits-ocr
docker build . -t 114.55.36.183:30002/library/gene-digits-ocr:0.0.1
helm upgrade gene-test -n gene gene_test/ --set image.repository=114.55.36.183:30002/library/gene-digits-ocr
http://redacted:31505/
http://redacted:30357/

# password issue of harbor helm
https://github.com/goharbor/harbor-helm/issues/75

# insecure docker registry
https://github.com/docker/for-win/issues/5692

# pull image from harbor

root@iZbp1607jn3ubt4hwupeasZ:~# docker login http://redacted:30002/
Authenticating with existing credentials...
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

after docker login
kubectl create secret generic regcred \
--from-file=.dockerconfigjson=.docker/config.json \
--type=kubernetes.io/dockerconfigjson

# deploying the digits-ocr by yaml directly
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gene-deployment
  labels:
    app: gene
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gene
  template:
    metadata:
      labels:
        app: gene
    spec:
      containers:
      - name: digits-ocr
        image: redacted:30002/library/gene-digits-ocr:0.0.1
        ports:
        - containerPort: 8000
      imagePullSecrets:
      - name: regcred

---

apiVersion: v1
kind: Service
metadata:
  name: gene-service
  labels:
    app: gene
spec:
  type: NodePort
  ports:
  - port: 8000
    targetPort: 8000
  selector:
    app: gene


# use configMap to store the django settings, due to I hardcode the AllowedHost to localhost

kubectl create configmap gene-django-settings -n gene --from-file settings.py

# use the configMap as settings.py file

      containers:
      - name: digits-ocr
        image: redacted:30002/library/gene-digits-ocr:0.0.1
        ports:
        - containerPort: 8000
        volumeMounts:
        - name: config-volume
          mountPath: /app/gene/gene/settings.py
          subPath: settings.py # only wants this file
      imagePullSecrets:
      - name: regcred
      volumes:
        - name: config-volume
          configMap:
            name: gene-django-settings

# prometheus service-discovery (reflection)

- job_name: gene-test 
  honor_timestamps: true
  scrape_interval: 30s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  kubernetes_sd_configs:
  - role: endpoints
    namespaces:
      names:
      - gene

kubectl create secret generic prometheus-additional-configs --from-file=prometheus-additional.yaml -n monitor

kubectl get secret prometheus-additional-configs -n monitor -o yaml

    additionalScrapeConfigs:
      name: prometheus-additional-configs
      key: prometheus-additional.yaml

https://github.com/prometheus-operator/prometheus-operator/tree/master/example
https://github.com/prometheus-operator/prometheus-operator/blob/master/example/user-guides/getting-started/example-app-pod-monitor.yaml
https://www.cnblogs.com/skyflask/p/11498834.html

Warning: resource prometheuses/prometheus-kube-prometheus-prometheus is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.

# retrospective for monitoring

additional scrape config is more dangerous for auto service-discovery. it couldn't be maintainable if we have thousands services.

i will use the pod/serviceMonitor to do the auto SD.

    serviceMonitorNamespaceSelector: {}
    serviceMonitorSelector:
      matchLabels:
        release: prometheus
    podMonitorNamespaceSelector: {}
    podMonitorSelector:
      matchLabels:
        release: prometheus

# drafting helm chart

- deployments
- svc
- ing
- rbac for podMonitor creation
- create podMonitor

helm install test --debug --dry-run gene_test/
helm repo add gene_test http://redacted:30002/chartrepo/library

TODO:
- jenkins building image with / ssh gitlab / helm / kubectl / docker /
- alicloud terraform fulfill
- training.py fulfill with MLP FC overfitting dropout/stop early
- digits-ocr-enhancement, can ingest one line of digits

# Future Improvment
- django with uwsgi in production-tier
- cicd with DCT & code scanning
- adding test coverage in digits-ocr project
- gitOps (PR build) & gitflow implementation between Gitlab <=> Jenkins
- EFK implemention  
- istio with kiali, circuit breaker , canary , chaos engineering , A/B test , distributed tracing

# kubectl connect remote apiserver
Gene@GENE ~
$ kubectl.exe config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://k8s.test.io:6443
  name: cluster1
contexts:
- context:
    cluster: cluster1
    user: admin
  name: context-cluster1
current-context: context-cluster1
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

in hosts
redacted k8s.test.io




