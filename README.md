# GitOps na Prática – Online Boutique

## Sobre o Projeto

Este repositório apresenta uma demonstração prática de GitOps, utilizando Kubernetes local (via Docker Desktop), ArgoCD e GitHub para realizar o deploy automatizado de uma aplicação de microserviços.  
O objetivo é mostrar, de ponta a ponta, como é possível versionar, auditar e gerenciar infraestrutura e aplicações a partir de arquivos YAML no GitHub, seguindo boas práticas modernas de DevOps.

A aplicação utilizada é o [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo), um demo open source do Google que simula um e-commerce moderno com diversos microserviços.

---

## Sumário

- [Arquitetura](#arquitetura)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Objetivo](#objetivo)
- [Pré-requisitos](#pré-requisitos)
- [Etapas do Projeto](#etapas-do-projeto)
- [Como Criar o App no ArgoCD](#como-criar-o-app-no-argocd)
- [Sincronizando e Aplicando no Cluster](#sincronizando-e-aplicando-no-cluster)
- [Visualização dos Componentes no ArgoCD](#visualização-dos-componentes-no-argocd)
- [Histórico de Logs e Estados](#histórico-de-logs-e-estados)
- [Entregas e Resultados](#entregas-e-resultados)
- [Problemas e Soluções](#problemas-e-soluções)
- [Customização Realizada](#customização-realizada)
- [Considerações Finais](#considerações-finais)
- [Referências](#referências)

---

## Arquitetura

- **Kubernetes local** rodando via Docker Desktop
- **ArgoCD** para GitOps e automação de deploys
- **GitHub** como fonte de verdade dos manifests YAML
- **Online Boutique** como aplicação de referência

```
gitops-microservices/
└── k8s/
    └── online-boutique.yaml
```

---

## Tecnologias Utilizadas

- Docker Desktop (Kubernetes habilitado)
- kubectl
- ArgoCD
- GitHub
- Git
- Online Boutique (Google Cloud Platform Microservices Demo)

---

## Objetivo

Executar e orquestrar, via GitOps, a aplicação Online Boutique em um cluster Kubernetes local, aplicando as melhores práticas de versionamento, auditabilidade e automação de infraestrutura como código.

---

## Pré-requisitos

- Docker Desktop instalado (com Kubernetes ativado)
- kubectl instalado e configurado
- ArgoCD instalado no cluster
- Conta no GitHub e repositório público para os manifests
- Git instalado

---

## Etapas do Projeto

### 1. Organização do repositório GitHub

- Realizado fork do repositório original ([microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)).
- Criado repositório próprio contendo apenas o arquivo YAML necessário (`release/kubernetes-manifests.yaml`), estruturado da seguinte forma:
    ```
    gitops-microservices/
    └── k8s/
        └── online-boutique.yaml
    ```
- Commit e push do manifesto ajustado.

### 2. Instalação do ArgoCD

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/release-2.11/manifests/install.yaml
kubectl get pods -n argocd
```

### 3. Acesso ao ArgoCD

- Port-forward para acessar o dashboard web:
    ```sh
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
- Acesse [https://localhost:8080](https://localhost:8080)

- Senha inicial do Argo CD:
    ```sh
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
    ```
  (usuário: `admin` — senha: use o comando acima)

---

## Como Criar o App no ArgoCD

🔷 **1. Clique em "New App" no painel do ArgoCD**  
Você será levado para o formulário de criação de aplicação. Preencha conforme abaixo:

### 🔹 General (Configuração Geral)

- **Application Name:**  
  `online-boutique`  
  Esse será o nome da aplicação dentro do ArgoCD. Pode ser qualquer nome, mas use algo representativo.

- **Project:**  
  `default`  
  O ArgoCD permite separar aplicações em "projetos". Usaremos o padrão chamado default.

- **Sync Policy:**  
  `Manual`  
  Deixe como manual inicialmente. Assim, você terá que clicar em "Sync" para aplicar as alterações quando desejar.  
  Depois, pode mudar para automática (auto-sync) se quiser que o ArgoCD sempre aplique mudanças do Git automaticamente.

### 🔹 Source (Fonte dos arquivos YAML)

- **Repository URL:**  
  `https://github.com/SEU_USUARIO/SEU_REPO.git`  
  Cole a URL exata do seu repositório GitHub onde está o arquivo `online-boutique.yaml`.  
  Substitua `SEU_USUARIO` e `SEU_REPO` pelo nome do seu perfil e do repositório.  
  Exemplo: `https://github.com/diogodantas/gitops-microservices.git`

- **Revision:**  
  `HEAD`  
  O ArgoCD sempre irá buscar a última versão da branch principal (main ou master).

- **Path:**  
  `k8s`  
  Caminho dentro do repositório até o arquivo de manifesto.  
  Se seu `online-boutique.yaml` está dentro da pasta `k8s/`, é isso que você coloca.

### 🔹 Destination (Cluster Kubernetes de destino)

- **Cluster URL:**  
  `https://kubernetes.default.svc`  
  Endereço padrão para o cluster Kubernetes local. Esse valor funciona automaticamente em ambientes locais como Docker Desktop ou Rancher Desktop.

- **Namespace:**  
  `default`  
  O namespace Kubernetes onde os recursos da aplicação serão criados. Pode deixar default, a menos que tenha criado outro namespace.

---

🔄 **Finalize**  
Depois de preencher tudo, clique em "Create".

O ArgoCD criará a aplicação e você verá o status dela. Clique em "Sync" para que ele aplique os arquivos YAML no cluster e comece a criar os pods, serviços e outros recursos.

---

## Sincronizando e Aplicando no Cluster

🔷 **2. Clique em "Sync" para aplicar os arquivos no cluster**  
Depois de criar a aplicação com sucesso no ArgoCD, é hora de fazer o deploy dos recursos no Kubernetes. Neste primeiro momento, a sincronização é manual e você deve clicar no botão **Sync** na interface do ArgoCD.

### 🔄 O que o botão "Sync" faz?

Ao clicar em **Sync**, o ArgoCD irá:

- Acessar o repositório Git configurado;
- Ler os arquivos YAML no caminho (Path) especificado;
- Aplicar todos os recursos definidos (pods, services, deployments, etc.) no cluster Kubernetes de destino (Destination);
- Atualizar o estado da aplicação e mostrar visualmente os pods e serviços criados na interface do ArgoCD.

> **Dica:** Sempre que fizer uma alteração nos arquivos YAML do repositório (ex: ajustar recursos, adicionar serviços, modificar réplicas), clique novamente em **Sync** para atualizar o ambiente Kubernetes de acordo com as últimas configurações do Git.

---

## Visualização dos Componentes no ArgoCD

🖼️ **Visualizando os componentes no ArgoCD**

Após clicar em **Sync** e a sincronização ser concluída com sucesso, você verá um diagrama semelhante ao exemplo abaixo na interface do ArgoCD:

> *(Inclua aqui um print ou screenshot do painel do ArgoCD mostrando a aplicação online-boutique com os componentes — se possível)*

Esse diagrama representa todos os recursos da aplicação **online-boutique** criados a partir do seu arquivo YAML. Cada linha conecta os serviços e pods, mostrando:

- Os serviços (Service - SVC) de cada microserviço;
- Os pods ativos criados a partir dos deployments;
- O status em tempo real de cada recurso (ícones verdes indicam que está tudo OK).

Essa visualização facilita o acompanhamento e o gerenciamento da aplicação diretamente pelo ArgoCD, sem precisar usar a linha de comando para ver se os pods estão no ar.

---

## 🧾 Histórico de Logs e Estados – Projeto GitOps com ArgoCD + Online Boutique

### ✅ Pods iniciais – Estado logo após o deploy

```bash
NAME                                     READY   STATUS             RESTARTS        AGE
adservice-6fd5cbf8bb-qwtw8               0/1     CrashLoopBackOff   4 (53s ago)     6m9s
emailservice-65f76bddc4-q9jtr            0/1     CrashLoopBackOff   6 (37s ago)     6m11s
recommendationservice-7bf646dd58-5rwcz   0/1     CrashLoopBackOff   7 (7s ago)      6m10s
```

#### 📜 Logs – emailservice

```json
{
  "message": "starting the email service in dummy mode.",
  "message": "Profiler disabled.",
  "message": "Tracing disabled.",
  "message": "listening on port: 8080"
}
```

#### 📜 Logs – recommendationservice

```json
{
  "message": "initializing recommendationservice",
  "message": "Profiler disabled.",
  "message": "Tracing disabled.",
  "message": "product catalog address: productcatalogservice:3550",
  "message": "listening on port: 8080"
}
```

#### 📜 Logs – adservice

```json
{
  "message": "AdService starting.",
  "message": "Stats enabled, but temporarily unavailable",
  "message": "Tracing enabled but temporarily unavailable",
  "message": "Ad Service started, listening on 9555"
}
```

#### 🛠️ Problema detectado com adservice – describe pod

```bash
Reason:       Error
Exit Code:    143
Message:      Readiness probe failed: timeout: failed to connect service "10.1.0.88:9555" within 1s
```

### ✅ Estado final – Após todos os pods estabilizarem

```bash
NAME                                     READY   STATUS    RESTARTS         AGE
adservice-6fd5cbf8bb-qwtw8               1/1     Running   8 (13m ago)      24m
emailservice-65f76bddc4-q9jtr            1/1     Running   10 (9m48s ago)   24m
recommendationservice-7bf646dd58-5rwcz   1/1     Running   11 (9m33s ago)   24m
```

### ❌ Tentativa de apagar o app online-boutique do ArgoCD

```bash
kubectl delete application online-boutique -n argocd

Error from server (NotFound): applications.argoproj.io "online-boutique" not found
```

Logo após:  
Você removeu os manifests do GitHub ou desincronizou, mas como o ArgoCD ainda estava apontando para o repositório, ele recriou os pods automaticamente.

### 🔁 Estado após "recriação" pelo ArgoCD

```bash
NAME                                     READY   STATUS    RESTARTS         AGE
adservice                                1/1     Running   8                119m
recommendationservice                    1/1     Running   11               119m
emailservice                             1/1     Running   10               119m
# (e todos os outros também em Running)
```

---

## Entregas e Resultados

- Repositório público contendo os manifests YAML da aplicação
- ArgoCD instalado e rodando no cluster local
- Deploy da Online Boutique orquestrado via GitOps
- Sincronização automática entre GitHub e cluster via ArgoCD
- Frontend acessível localmente via port-forward
- Histórico de deploys versionado e auditável pelo Git

---

## Problemas e Soluções

- **Pods em CrashLoopBackOff:**  
  Ajustado valores de `initialDelaySeconds` e `timeoutSeconds` nas probes para evitar falhas por timeout em serviços que demoram a subir.

- **Porta já em uso no port-forward:**  
  Alterada a porta local para uma disponível (ex: 8082).

- **Path incorreto no Application do ArgoCD:**  
  Corrigido para apontar corretamente para a pasta dos manifests.

---

## Customização Realizada

Para demonstrar domínio sobre os manifests, foi alterado o número de réplicas do microserviço `loadgenerator` de 1 para 3, promovendo maior geração de carga para testes.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: loadgenerator
  labels:
    app: loadgenerator
spec:
  selector:
    matchLabels:
      app: loadgenerator
  replicas: 3  # Valor alterado de 1 para 3
  ...
```

---

## Considerações Finais

O projeto mostrou, na prática, como é possível aplicar GitOps para garantir automação, rastreabilidade e segurança nos processos de deploy em Kubernetes.  
Além de exercitar conceitos essenciais de infraestrutura como código, integração contínua e arquitetura de microserviços, a experiência proporcionou uma visão clara de como grandes empresas gestionam ambientes cloud-native.

---

## Referências

- [Documentação ArgoCD](https://argo-cd.readthedocs.io/)
- [Online Boutique (GoogleCloudPlatform/microservices-demo)](https://github.com/GoogleCloudPlatform/microservices-demo)
- [Kubernetes](https://kubernetes.io/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
