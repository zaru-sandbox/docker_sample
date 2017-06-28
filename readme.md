```
$ docker-compose build
$ docker-compose up
```

## k8s and GEK

### Create deployment and service

```
export PROJECT_ID=sakuraba-sample
export IMAGE_NGINX=nginx
export IMAGE_PHP=php-fpm
export TAG=1

docker build -t gcr.io/${PROJECT_ID}/${IMAGE_PHP}:${TAG} -f ./php/Dockerfile .
docker build -t gcr.io/${PROJECT_ID}/${IMAGE_NGINX}:${TAG} .
gcloud docker -- push gcr.io/${PROJECT_ID}/${IMAGE_PHP}:${TAG}
gcloud docker -- push gcr.io/${PROJECT_ID}/${IMAGE_NGINX}:${TAG}

envsubst < nginx-service.yaml | kubectl create -f -
envsubst < php-fpm-service.yaml | kubectl create -f -
envsubst < nginx-deployment.yaml | kubectl create -f -
envsubst < php-fpm-deployment.yaml | kubectl create -f -
```

### Update deployment

```
envsubst < nginx-deployment.yaml | kubectl replace -f -
envsubst < php-fpm-deployment.yaml | kubectl replace -f -
```

### RollingUpdate

```
export TAG=2
docker build -t gcr.io/${PROJECT_ID}/${IMAGE_PHP}:${TAG} -f ./php/Dockerfile .
gcloud docker -- push gcr.io/${PROJECT_ID}/${IMAGE_PHP}:${TAG}

kubectl set image deployment/php-fpm php-sample=gcr.io/${PROJECT_ID}/${IMAGE_PHP}:${TAG}
```

### Login to Pod

```
kubectl exec -it <POD_NAME> bash
```

### kubectl get all sample

```
NAME                                   READY     STATUS    RESTARTS   AGE
po/nginx-2718833065-0lngp              1/1       Running   0          14m
po/php-fpm-105323463-txnqr             1/1       Running   0          2m

NAME             CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
svc/kubernetes   10.23.240.1     <none>         443/TCP        1d
svc/nginx        10.23.255.14    35.189.155.7   80:32520/TCP   14m
svc/php-fpm      10.23.243.174   <none>         9000/TCP       14m

NAME                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/nginx              1         1         1            1           14m
deploy/php-fpm            1         1         1            1           16m
```

---

## Same Pod pattern

```
envsubst < combine-deployment.yaml | kubectl create -f -
envsubst < combine-service.yaml | kubectl create -f -
```

### Show container log

```
kubectl po/<POD_NAME> nginx
kubectl po/<POD_NAME> php-fpm
```

### Log in to specific container in Pod

```
kubectl exec -it <POD_NAME> -c nginx bash
kubectl exec -it <POD_NAME> -c php-fpm bash
```

### RollingUpdate

No different to split pattern.

You can update it on specific container in Pod, but Pod will be teminated.

```
export TAG=2
docker build -t gcr.io/${PROJECT_ID}/${IMAGE_PHP}:${TAG} -f ./php/Dockerfile .
kubectl set image deployment/combine php-sample=gcr.io/${PROJECT_ID}/${IMAGE_PHP}:${TAG}
```

### kubectl get all sample

```
NAME                        READY     STATUS    RESTARTS   AGE
po/combine-54602166-x111w   2/2       Running   0          9m

NAME             CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
svc/combine      10.23.248.164   35.189.155.7   80:32306/TCP   20m
svc/kubernetes   10.23.240.1     <none>         443/TCP        23m

NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/combine   1         1         1            1           15m
```
