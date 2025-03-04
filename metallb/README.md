# Configuração MetalLB

### Nesta seção você verá um tutorial para instalar e configurar o metalLB, uma solução load balancer para um claster bare-metal.

### Preparação

- Se você esta usando o kube-proxy no IPVS mode, voce precisa habilitar o strict ARP mode, consulte a documentação.

### Instalação por manifesto

- Instalar o manifesto:
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```
### Configurando o MetalLB

- Neste cluster, vamos utilizar uma configuração bem simples do MetalLB (Layer 2 Configuration), basicamente precisamos de 2 arquivos:
  - IPAddressPool.yaml
  - L2Advertisement.yaml

- Exemplo de arquivos:

**Nota:** Demais exemplos podem ser obtidos no link: [Exemplos metalLB](https://metallb.io/configuration/)
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.240-192.168.1.250 # Esse range de IP devera conter os IPs disponiveis em sua rede para serem delegados pelo MetalLB

apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
```

- Criado os dois arquivos, precisamos aplicar eles:
```
kubectl apply -f IPAddressPool.yaml
kubectl apply -f L2Advertisement.yaml
```

### Verificando instalação
```
kubectl get pods -n metallb-system
```
**Nota:** Todos os pods devem estar em "Running"

### Referências
- [MetalLB Documentation](https://metallb.io/installation/)
