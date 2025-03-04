# k8s-proxmox
Este repositório documenta o processo de criação e configuração de um cluster Kubernetes (K8s) dentro de um ambiente de virtualização baseado em Proxmox. O objetivo é fornecer um guia detalhado para ajudar na configuração desde a instalação do Proxmox até a instalação e configuração do **jupyterhub** no Kubernetes.

## Índice

1. [Instalação do Proxmox](#instalação-proxmox)
2. [Instalação do Kubernetes](#kubernetes)
3. [Referências & Agradecimentos](#referencias--agradecimentos)

## Instalação Proxmox:
- Baixe a ISO no link: [https://atxfiles.netgate.com/mirror/downloads/](https://atxfiles.netgate.com/mirror/downloads/)
- Prossiga com o passo a passo da instalação (Recomendo que você utilize o padrão da instalação), se ficar com duvidas durante a instalação consulte as [Referências](#referencias--agradecimentos).

**OBS:** Caso você for criar um cluster Proxmox, os nós não podem conter o mesmo nome, então lembre-se de dar nomes diferentes aos nós para evitar trabalho no futuro.

### Acesso à Interface Web

- Ao fim da instalação, acesse a interface web do servidor Proxmox: `SEU-IP:8006`

- Se o intuito é utilizar o sistema de forma livre (sem adquirir a licença)
	- Acesse: `node -> updates -> repositories`
 	- Desative os seguintes repositórios enterprise
  		- pve-enterprise
	- Adicione os seguintes repositórios:
   		- No-Subscription
   		- Ceph Reef No-Subscription (Se você for instalar o Ceph).

Execute os seguintes comandos para atualização do proxmox:
```bash 
apt update
pveupgrade -y
```

### Para criar um cluster Proxmox:
1. Acesse a interface web
2. Vá em `Datacenter -> Cluster -> Create Cluster`
3. Com o cluster criado voce obtera o Join Information, que voce ira utilizar para adicionar os demais nós ao cluster.

### Para carregar uma VM:
- Primeiro você precisa criar a VM para obter o ID.
```bash
qm importdisk ID-VM imagem.raw.qcow2 storage_destino
```
Dica: crie uma vm e utilize ela como template para clonar os demais nós, caso queira utilizar o meu tamplate haverá um template pronto no repositório, se ficar com duvidas consulte as [Referências](#referencias--agradecimentos).

```bash
Editar os hosts:
Exemplo de arquivo de host
127.0.0.1 localhost.localdomain localhost
192.168.20.51 pve1.inf.ufg.br pve1

# The following lines are desirable for IPv6 capable hosts

::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

192.168.20.50 pve0.inf.ufg.br pve0 #Observe que estou declarando os demais nós dos cluster como hosts 
192.168.20.52 pve2.inf.ufg.br pve2
192.168.20.53 pve3.inf.ufg.br pve3
```


### Excluir local-lvm (Opcional):
O comando pode variar de acordo com o mapeamento/instalação do Proxmox
```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

### Incluir no tailscale (Opcional):
- Faça isso caso voce deseje configurar algum tipo de acesso externo, pode utilizar outra VPN de sua preferência, ou outro método.

- Download:
```
curl -fsSL https://tailscale.com/install.sh | sh
```
Execute: 
```
taiscale up
```
Faça o login

## Instalação Kubernetes
- Esta seção fornece instruções detalhadas sobre como instalar o Kubernetes v1.32 em um servidor Ubuntu 24.04.

**IMPORTANTE:** O Kubelet não suporta o swap ativo, desative-o antes de continuar:
- Desative o swap temporariamente com o seguinte comando:
```
sudo swapoff -a
```

- Agora precisamos editar o arquivo /etc/fstab para garantir que o swap nao seja mais utilizado pelo SO.
```
sudo nano /etc/fstab
```
- Procure por uma linha semelhante a essa:
```
/swap.img       none    swap    sw      0       0
```
E entao comente a linha com uma "#", da seguinte forma:
```
#/swap.img       none    swap    sw      0       0
```

- Atualize os pacotes do SO e instale os pacotes necessários para usar o repositório apt do Kubernetes:
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
```

### Baixe a chave pública de assinatura para os repositórios de pacotes do Kubernetes

- Baixe a chave pública de assinatura para os repositórios de pacotes do Kubernetes. A mesma chave de assinatura é usada para todos os repositórios, então você pode ignorar a versão na URL. Note que em lançamentos anteriores ao Debian 12 e Ubuntu 22.04, o diretório /etc/apt/keyrings não existe por padrão, e deve ser criado antes do comando curl.
```    
# Se a pasta `/etc/apt/keyrings` não existir, ela deve ser criada antes do comando curl, leia a nota abaixo.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # permitir que programas APT sem acesso privilegiado leiam este keyring
```

### Adicione o repositório apt apropriado do Kubernetes
- Adicione o repositório apt do Kubernetes. Se você quiser usar uma versão do Kubernetes diferente de v1.32, substitua v1.32 com a versão menor desejada no comando a seguir:
```
# Isto substitui qualquer configuração existente na pasta /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list #ajuda ferramentas tais como command-not-found a funcionar corretamente
```

### Atualize o índice de pacotes apt, instalar kubelet, kubeadm e kubectl, e fixe suas versões

- Atualize o índice de pacotes apt, instale o kubelet, kubeadm e kubectl, e fixe suas versões para evitar atualizações inesperadas:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Verificando a instalação
```
kubectl cluster-info
kubeadm version	
systemctl status kubelet
```

- Caso voce obtenha uma resposta semelhante a essa:
```
The connection to the server <server-name:port> was refused - did you specify the right host or port?
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.1"
```
- A instalação foi concluída

### Ativando o bash completation
- Ative o bash completation do kubectl (Isso vai te ajudar muito durante o desenvolvimento):
```
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
sudo chmod a+r /etc/bash_completion.d/kubectl
```
- Caso voce queira usar um Alias (um apelido, kubectl --> k), execute:
```
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
```

- Reinicie o bash
```
source ~/.bashrc
```

### Instalar o Containerd

- Instalar o Containerd
```
sudo apt-get install -y containerd
```
- Configurar o Containerd
```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```
### Configurar o Kernel

- Habilitar o módulo br_netfilter
```
sudo modprobe br_netfilter
```
- Aplica configurações sysctl necessárias:
```
sudo bash -c 'cat <<EOF >/etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF'
sudo sysctl --system
```
- Habilitando o ip forward:
```
sudo nano /etc/sysctl.conf
```
- Descomente a linha:
```
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

### Habilitando o kubelet
```
sudo systemctl enable --now kubelet
```

### Inicializando o cluster
```
sudo kubeadm init --control-plane-endpoint "192.168.20.100:6443" --upload-certs --pod-network-cidr=10.244.0.0/16 
```
**Notas:**
- --control-plane-endpoint: define o endpoint, o IP do nó master, control-plane.
- --upolad-certs: faz a atualização dos certificados do cluster.
- --pod-network-cidr: declara a rede de pods (por padrao a rede é 192.168.0.0/16), como esta rede esta sendo utilizada para o meu cluster eu fiz a troca para a outra rede.
- Para mais personalizações dos parametros de inicialização do ```kubeadm init``` consulte a documentação.

- Finalizada a inicialização, execute (Esses comandos configuram o arquivo /.kube/config e permitem que você execute o comando kubectl sem o uso de sudo:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Encontre o token de JOIN que aparecera no terminal e utilize esse token para inserir os demais nós do cluster k8s.
```
kubeadm join <IP-ENDPOINT>:6443 --token <SEU-TOKEN> --discovery-token-ca-cert-hash sha256:<SEU-CERT-HASH>
```

- Exemplo de comando JOIN:
```
kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
```

- Caso você perca o token, não se preocupe, é possível obte-lo com o comando:
```
kubeadm token create --print-join-command
```

### Próximos passos:
- Instalação e configuração do calico

## Referencias & Agradecimentos:
- [Calico Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)
- [Primeiros passos com Kubernetes na prática! Pods, Deployments e Services.](https://www.youtube.com/watch?v=xXUhepb5Ccc&t=1365s&ab_channel=Prof.GustavoLeit%C3%A3o)
- [Kubernetes - Configurando NGINX Ingress Controller (Bare Metal)](https://www.youtube.com/watch?v=AGSGcUzkqrE&t=879s&ab_channel=Prof.GustavoLeit%C3%A3o)
- [Kubernetes Documentation](https://kubernetes.io/pt-br/docs/home/).
- [Instalação kubernetes](https://github.com/nrcinfufg/kubekubeflow/blob/main/README.md?plain=1)
- [How to Build an Awesome Kubernetes Cluster using Proxmox Virtual Environment](https://www.youtube.com/watch?v=U1VzcjCB_sY&t=2839s&ab_channel=LearnLinuxTV).
- [Tutorial Instalação do Ceph no Proxmox 8](https://www.youtube.com/watch?v=TS9KpuIMPfU&t=874s&ab_channel=TISemHesitar).
