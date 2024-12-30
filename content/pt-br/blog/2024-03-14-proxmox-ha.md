---
title: "Proxmox VE: Configurando um Cluster de Alta Disponibilidade"
date: 2024-03-14
draft: false
tags:
  - Proxmox VE
  - Alta Disponibilidade
  - Cluster
image: "/raphazilla/images/blog/imagem-proxmox.png"
comments: true
---
![Proxmox VE: Configurando um Cluster de Alta Disponibilidade](/raphazilla/images/blog/imagem-proxmox.png)

O Proxmox VE é uma plataforma de virtualização poderosa que oferece a capacidade de criar clusters para garantir alta disponibilidade (HA) e tolerância a falhas. Neste artigo, exploraremos o processo de configuração passo a passo para criar um cluster Proxmox com alta disponibilidade.

## Pré-requisitos

Antes de começar, certifique-se de ter pelo menos dois nós Proxmox VE configurados e conectados na mesma rede. Certifique-se de que cada nó possua endereços IP estáticos e resolução de DNS.

## Passo 1: Instalação do Pacemaker e Corosync

A primeira etapa é instalar os pacotes necessários para a configuração do cluster. Utilize o seguinte comando em cada nó:

```bash
sudo apt-get update
sudo apt-get install corosync pacemaker fence-agents
```

## Passo 2: Configuração do Corosync

Edite o arquivo de configuração do Corosync no diretório `/etc/corosync/corosync.conf`:

```bash
sudo nano /etc/corosync/corosync.conf
```

Adicione as seguintes linhas para configurar o transporte e a interface de rede:

```plaintext
totem {
  version: 2
  secauth: off
  cluster_name: mycluster
  transport: udpu
}

nodelist {
  node {
    ring0_addr: <IP_Node_1>
    name: <Node_1_Hostname>
    nodeid: 1
  }
  node {
    ring0_addr: <IP_Node_2>
    name: <Node_2_Hostname>
    nodeid: 2
  }
}
```

Substitua `<IP_Node_1>`, `<IP_Node_2>`, `<Node_1_Hostname>`, e `<Node_2_Hostname>` pelos valores correspondentes de seus nós.

## Passo 3: Inicialização e Verificação do Corosync

Inicie o Corosync nos dois nós:

```bash
sudo systemctl start corosync
sudo systemctl enable corosync
```

Verifique o status do Corosync:

```bash
sudo corosync-cfgtool -s
```

## Passo 4: Configuração do Pacemaker

Edite o arquivo de configuração do Pacemaker no diretório `/etc/pve/corosync.conf`:

```bash
sudo nano /etc/pve/corosync.conf
```

Adicione a seguinte linha para configurar o uso do Corosync:

```plaintext
CMAN_NODES=/etc/corosync/corosync.conf
```

## Passo 5: Inicialização do Pacemaker

Inicie o serviço do Pacemaker:

```bash
sudo systemctl start pve-cluster
sudo systemctl enable pve-cluster
```

Verifique o status do Pacemaker:

```bash
sudo pvecm status
```

## Passo 6: Adição de Nós ao Cluster

Adicione os nós ao cluster com o seguinte comando (executado em um nó):

```bash
sudo pvecm add <Node_2_Hostname>
```

## Conclusão

Parabéns! Você configurou com sucesso um cluster de alta disponibilidade no Proxmox VE. Certifique-se de revisar a documentação oficial do Proxmox para obter informações adicionais sobre configurações avançadas e otimizações para ambientes específicos.

Este é apenas o primeiro passo para criar uma infraestrutura virtualizada robusta e resiliente. Em futuros artigos, exploraremos estratégias de migração, balanceamento de carga e otimizações adicionais para tirar o máximo proveito do seu ambiente Proxmox VE com cluster de alta disponibilidade.