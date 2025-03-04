# Jupyterhub 

- Versao jupyterhub: 4.1.0
- Versao Helm: 3.17.1


### Preparação
- Certifique-se que voce ja tenha configurado:
  - Dynamic Volume Provisioning (StorageClass)
  - Load Balancer ou Ingress para o acesso ao jupyterhub
  - Helm v3

- Com isso configurado, vamos para instalação:

- Adicione o repositorio do jupyterhub:
```
helm repo add jupyterhub https://hub.jupyter.org/helm-chart/
```


- Atualize os repositorios:
  
```
helm repo update
```

- Crie um arquivo values 

**Nota:** Só faça isso caso voce queira customizar a instalação e alguns parametros que podem ser importantes para o deploy, acesse a documentação.
- Campos a serem editados:
  - StorageClass: Coloque o nome da sua storageclass
  - Baseurl: /"Url que voce quer" (Caso voce va usar o ingress)
- Observe o arquivo 'values.yaml' deixado no repositório como exemplo

### Instalação:
```
helm install jupyterhub jupyterhub/jupyterhub --namespace jupyterhub --create-namespace --values values.yaml
```

### Verificar instalação
```
kubectl get all -n jupyterhub
```






### Referências:
- [Zero to JupyterHub with Kubernetes](https://z2jh.jupyter.org/en/stable/)
