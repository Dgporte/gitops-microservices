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

### 4. Deploy da aplicação via ArgoCD

- Criação de um "Application" no ArgoCD apontando para o repositório Git, no path correto (ex: `k8s/`).
- Sincronização para aplicar os manifests no cluster.
- Monitoramento do status dos pods pela interface do ArgoCD.

### 5. Exposição do frontend

- O serviço frontend é do tipo ClusterIP. Para acessar:
    ```sh
    kubectl port-forward svc/frontend-external 8082:80
    ```
- Aplicação disponível em [http://localhost:8082](http://localhost:8082)

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
