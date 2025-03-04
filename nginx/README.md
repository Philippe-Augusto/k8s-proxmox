# NGINX

### Instalação do nginx:

- Utilizando Helm:
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

```
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer
```

- Utilizando um yaml:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
```

### Verificando instalação
- Observe se a instalação funcionou com o seguinte comando (todos os pods devem estar em running):
```
kubectl get pods --namespace=ingress-nginx
```


- Para testarmos o metalLB, execute o seguinte comando:
```
kubectl get service ingress-nginx-controller --namespace=ingress-nginx
```

- Verifique se no campo "External-IP" foi designado um ip com base no arquivo fornecido ao metalLB (no arquivo IPAdressessPool.yaml), 
caso esteja "Pending" retorne ao passo do metalLB, deve haver algum erro de instalação/configuração.

Agora, para testar o nginx crie os seguintes arquivos:
```
nginx-deployment.yaml:
####################################
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.23
        ports:
        - containerPort: 80
####################################
```

```
nginx-service.yaml:
####################################
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP 
####################################
```

```
nginx-service-ingress.yaml:
####################################
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
# Observe que não há um campo "host", uma vez que no meu cenário não possuo um servidor DNS, ao deixar sem o campo host, o host passa a ser o IP atribuído pelo metalLB ao serviço do nginx
    - http: 
        paths:
          - path: /nginx
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
####################################
```

- Aplique os arquivos:
```
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
kubectl apply -f nginx-ingress.yaml
```

- Teste o Nginx:
```
curl 192.168.20.200/nginx
```

- Voce deve obter uma saida semelhante a essa:
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
```

### Referências
- [Nginx Documentation](https://docs.nginx.com/nginx-ingress-controller/)
