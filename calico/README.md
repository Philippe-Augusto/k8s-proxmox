# Configuração Calico

- Nesta seção você verá um tutorial para instalação do Calico, um plugin CNI para o seu cluster Kubernetes
- Versão: 3.29.2

### Instalando o Calico 
- Tendo todos os nós inseridos no cluster, vamos prosseguir com a instalação do Calico.

- Instalando o Calico.
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml
```

- Editando o arquivo custom-resources.yaml.

**Nota:** Você só precisa editar esse arquivo caso você deseje mudar a rede dos pods (por padrão a rede é 192.168.0.0/16) ou caso você esteja utilizando essa rede, como é o meu caso.

```
wget https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/custom-resources.yaml
```
  
- Edite o cidr para o range de IP que voce utilizou no pod, com o comando kubeinit.
  
- Aplicando o arquivo.
  
```
kubectl create -f custom-resources.yaml
```

- Para mais customizações consulte as [referências](#referências).

### Verificação da instalação

```
kubectl get pods -A
```

- Todos os pods devem estar "Running", isso pode levar até 5 minutos a depender da máquina.

- Caso você queira atribuir pods ao master, execute:

```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
- Confirme que o cluster esta operante com:
```
kubectl get nodes -o wide
```
**Nota:** Todos os nós deverão estar em "Ready".

### Referências
- [Calico Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart).


