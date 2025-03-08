docker login
 docker pull shestera/scaletestapp
 docker run -p 8080:8080 shestera/scaletestapp

Шаг 1. Запустить Minikube
//minikube start --cpus=4 --memory=8192
!! minikube start --extra-config=kubelet.authentication-token-webhook=true --extra-config=kubelet.authorization-mode=Webhook
Это запустит кластер с достаточными ресурсами.

Шаг 2. Активировать metrics-server
minikube addons enable metrics-server
Проверить, что сервер метрик работает:

kubectl get apiservices | grep metrics
Ожидаемый вывод:

v1beta1.metrics.k8s.io                      True


Применить манифест
kubectl apply -f Exc2/deployment.yaml

Проверить:
kubectl get pods
результат:
NAME                            READY   STATUS    RESTARTS   AGE
scaletestapp-6845c99ff5-ccbmt   1/1     Running   0          18s

Применить манифест
kubectl apply -f Exc2/service.yaml

Проверить сервис:
kubectl get svc
рузельтат
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes             ClusterIP      10.96.0.1      <none>        443/TCP        9m46s
scaletestapp-service   LoadBalancer   10.99.83.194   <pending>     80:30889/TCP   2m22s

Получить URL сервиса:

minikube service scaletestapp-service --url
Результат:
http://127.0.0.1:59601
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.


Применить манифест
kubectl apply -f Exc2/hpa.yaml
Проверить статус HPA:

kubectl get hpa
результат 
NAME               REFERENCE                 TARGETS           MINPODS   MAXPODS   REPLICAS   AGE
scaletestapp-hpa   Deployment/scaletestapp   memory: 30%/80%   1         10        1          25s



Запустить Locust
locust
Перейти в браузере:
➡ http://localhost:8089

Number of users: 50
Hatch rate: 5
Host: <minikube service scaletestapp-service --url>

Шаг 4. Запустить тест и проверить в Kubernetes
Открыть Kubernetes Dashboard:

minikube dashboard
Отслеживать реплики:

kubectl get hpa -w
kubectl get pods


minikube delete