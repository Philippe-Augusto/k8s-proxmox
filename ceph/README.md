# CephFS For Kubernetes

- Versao do kubernetes: 1.32
- Lembrando que no nosso cenário estamos utilizando o ceph configurado dentro do proxmox, para um tutorial para configurar o ceph em seu Proxmox siga: [Tutorial Ceph](../../README.md/#referencias--agradecimentos)


### Preparação
- Para fazer a instalação do CephFS, precisamos do ceph-csi

- Iremos realizar a instalação do ceph-csi através do Helm, certifique-se de que você tenha o Helm configurado em sua máquina.

- Adicione o repositorio a sua lista de repositorios Helm:
```
helm repo add ceph-csi https://ceph.github.io/csi-charts
```

- Atualize os repositorios:
```
helm repo update
```

- Crie um arquivo 'values.yaml' para configuração da instalação do ceph
```
####################################
csiConfig:
 - clusterID: "ID do seu cluster" # Obtenha o id com ceph fsid
   monitors: #IP dos monitores do cluster
    - "192.168.20.50:6789" 
    - "192.168.20.51:6789"
    - "192.168.20.52:6789"
   cephFS:
     subvolumeGroup: "kubernetes" # Caso voce queira organizar melhor os dados que serao utilizados, sugiro que voce crie um subvolumeGroup dentro do ceph
####################################
```

### Instalação

- Instalar o ceph-csi com o seguinte comando:
```
kubectl create namespace ceph-csi-cephfs #Crie a namespace primeiro
helm install --namespace "ceph-csi-cephfs" "ceph-csi-cephfs" ceph-csi/ceph-csi-cephfs -f values.yaml
```

### Verifcar instalação:
```
kubectl get all -n ceph-csi-cephfs
```

### Configurando o Ceph

- Agora vamos criar uma storageClass no nosso cluster para fazer provisionamento dinamico de volume (Isso é necessário para que o jupyterhub seja configurado).

[Exemplos de arquivos - Documentação Oficial](https://github.com/ceph/ceph-csi/tree/devel/examples/cephfs)

- Crie os arquivos (Os arquivos estão no repositório):
  - secret.yaml
  - storageclass.yaml
**Nota:** A storage class no Kubernetes é quem vai fazer o provisionamento dinâmico de volumes.
  - pvc.yaml

### Teste a configuração
```
kubectl get pvc
```
- Se o pvc constar como "Bound" o ceph esta configurado

### Referências 
[Ceph CSI Documentation](https://github.com/ceph/ceph-csi)
[Helm](https://helm.sh/)
