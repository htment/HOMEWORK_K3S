#  Установим K3s
```
curl -sfL https://get.k3s.io | sh -
```
```
sudo systemctl status k3s

```

Получим конфиг для kubectl:
```
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config
```

Проверим работоспособность:

```
kubectl get nodes
kubectl get pods -n kube-system
```
Проверим так же

```
 kubectl get pods -A
```

![alt text](image.png)

## Deployment и Service для Redis
https://github.com/netology-code/sdvps-homeworks/blob/main/6-05.md


```
root@Deb-test:/HOMEWORK_K3s# echo 'apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: master
        image: bitnami/redis:6.0.13
        env:
         - name: ALLOW_EMPTY_PASSWORD
           value: "yes"
        ports:
        - containerPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379' > redis-deployment.yaml
      
```

```
kubectl apply -f redis-deployment.yaml
```

![alt text](image-1.png)    


Проверим статус Pod'а:
```
kubectl get pods
```
![alt text](image-2.png)

Проверим Service
```
kubectl get svc
```
![alt text](image-3.png)

Проверим логи Redis:

```
kubectl logs redis-54457d549d-vg4dg  # (подставим имя Pod из `kubectl get pods`)

```

![alt text](image-4.png)



```
kubectl exec -it <pod-name> -- redis-cli
kubectl exec -it redis-54457d549d-vg4dg -- redis-cli
```
Эта команда:
exec - выполняет команду в контейнере
-it - делает сессию интерактивной
redis-54457d549d-vg4dg - имя вашего Pod
-- redis-cli - команда для запуска Redis CLI внутри контейнера

![alt text](image-5.png)



Выполните тестовые команды:

```
SET test "Hello Kubernetes"
GET test
```

![alt text](image-6.png)


### Проверить процессы в контейнере
kubectl exec redis-54457d549d-vg4dg -- ps aux

![alt text](image-7.png)

### Посмотреть логи Redis
kubectl logs redis-54457d549d-vg4dg

![alt text](image-8.png)

### Проверить переменные окружения
kubectl exec redis-54457d549d-vg4dg -- env

![alt text](image-9.png)


### Проброс порта для отладки:
```
kubectl port-forward redis-54457d549d-vg4dg 6379:6379
```

### Удаление контейнера (Pod):
```
kubectl delete pod redis-54457d549d-vg4dg
```
![img](image-10.png)


#### Если нужно полностью пересоздать Deployment

```
kubectl delete -f redis-deployment.yaml

или 
kubectl delete deployment redis
kubectl delete service redis-service

```
резвернем заново
```
kubectl apply -f redis-deployment.yaml
```


#### Масштабирование (изменить количество реплик):
```
kubectl scale deployment redis --replicas=2
```
#### Полная пересборка Deployment (принудительный перезапуск):
```
kubectl rollout restart deployment redis
```
#### Просмотр истории развертывания:
```
kubectl rollout history deployment redis
```
![alt text](image-11.png)


Создадим файл nginx-k8s.yaml со следующим содержимым:
[text](nginx-k3s.yaml)

```
kubectl apply -f nginx-k8s.yaml
```

![alt text](image-12.png)


Проверим ConfigMap:
```
 kubectl describe configmap nginx-config
```
![alt text](image-13.png)

```
kubectl get pods -l app=nginx

```
![alt text](image-14.png)