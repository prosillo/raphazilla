---
title: "Aprofundando no KVM: Desvendando Recursos Avançados de Virtualização"
date: 2024-02-07
draft: false
tags:
  - KVM
  - QEMU
  - Cloud-init
  - Infraestrutura como Código
image: "/raphazilla/images/blog/imagem-kvm-qemu.png"
comments: true
---
![KVM Avançado](/raphazilla/images/blog/imagem-kvm-qemu.png)

## Introdução

Em nosso artigo anterior, exploramos o fascinante mundo da virtualização com **KVM (Kernel-based Virtual Machine)** e **QEMU (Quick Emulator)**. Agora, vamos aprofundar nosso conhecimento no KVM, explorando recursos avançados que elevam a virtualização a um novo patamar. Este artigo é a sequência natural, então, se você ainda não leu o artigo anterior, confira [aqui](/raphazilla/blog/kvm-qemu/).

## Snapshot e Clonagem de Máquinas Virtuais

Uma das características poderosas do KVM é a capacidade de criar snapshots de máquinas virtuais, permitindo a captura do estado atual da VM. Esses snapshots podem ser usados para backup, replicação ou para retornar a máquina virtual a um estado anterior. Vamos ver como criar um snapshot:

```bash
virsh snapshot-create-as --domain nome-da-vm --name snapshot-nome --description "Descrição do Snapshot"
```

Da mesma forma, a clonagem de máquinas virtuais no KVM é eficiente e facilita a replicação de ambientes para fins de teste ou desenvolvimento.

```bash
virt-clone --original nome-da-vm --name clone-nome --file /caminho/para/clone.qcow2
```

## Testes de Escalabilidade para Planejamento Futuro

Empresas que buscam escalar suas operações podem utilizar o KVM para criar clusters de máquinas virtuais e testar o desempenho em diferentes cenários. Isso fornece insights valiosos para otimizar a infraestrutura antes da implementação em larga escala.

### Configuração do Cluster para Testes de Escalabilidade

**Exemplo de Criação de Máquina Virtual no Cluster:**
```bash
sudo virt-install --name servidor-1 --memory 8192 --vcpus 8 --disk tamanho=50 --os-variant rhel8 --numatune memory=auto --cpu host-passthrough
```

Aqui, a máquina virtual "servidor-1" é configurada com 8 VCPUs, 8 GB de memória e 50 GB de espaço em disco. Esta configuração pode ser replicada para criar várias instâncias no cluster.

**Exemplo de Migração entre Hosts no Cluster:**
```bash
virsh migrate --live --persistent --domain servidor-1 qemu+ssh://host-destino/system
```

A capacidade de migração ao vivo permite transferir uma máquina virtual entre hosts sem interrupção. Essa funcionalidade é essencial para balanceamento de carga e manutenção proativa.

**Exemplo de Teste de Carga no Cluster:**
```bash
ab -n 10000 -c 100 http://endereco-da-maquina-virtual
```

O Apache Benchmark (`ab`) é uma ferramenta útil para simular carga em servidores web. Essa abordagem permite avaliar como o cluster se comporta sob condições de tráfego intenso.

**Exemplo de Monitoramento do Cluster:**
```bash
sudo virsh domstats servidor-1
```

O monitoramento constante é crucial para identificar gargalos e ajustar a configuração do cluster conforme necessário. O comando `domstats` fornece informações detalhadas sobre o desempenho da máquina virtual.

Esses exemplos demonstram como o KVM pode ser utilizado de maneira prática e eficaz para testar a escalabilidade da infraestrutura. Ao realizar esses testes em um ambiente controlado, as organizações podem tomar decisões fundamentadas sobre expansão e dimensionamento, garantindo uma base sólida para o crescimento futuro.

## Gerenciamento de Recursos com Cgroups

O KVM oferece recursos avançados de gerenciamento de recursos por meio do uso de **Cgroups (Control Groups)**. Essa funcionalidade permite a alocação de recursos como CPU, memória e I/O, proporcionando um controle granular sobre o desempenho das máquinas virtuais.

### Atribuição de CPU com Vcpupin

Você pode utilizar o recurso `vcpupin` para definir a afinidade de CPU para cada VCPU (Virtual CPU) da sua máquina virtual. Isso ajuda a otimizar o desempenho e garantir que cada VCPU esteja alinhado com núcleos físicos específicos.

Exemplo de configuração do Cgroups no XML da VM:

```xml
<cputune>
  <vcpupin vcpu='0' cpuset='1'/>
  <vcpupin vcpu='1' cpuset='2'/>
</cputune>
```

Neste exemplo, as VCPUs 0 e 1 da máquina virtual estão associadas aos núcleos físicos 1 e 2, respectivamente.

### Controle de Memória com MemoryBacking

O elemento `memoryBacking` permite ajustar configurações relacionadas à memória. O uso de **hugepages** é uma prática comum para otimizar o desempenho da VM ao manipular grandes quantidades de memória.

Exemplo de configuração do Cgroups no XML da VM:

```xml
<memoryBacking>
  <hugepages/>
</memoryBacking>
```

Ao configurar `<hugepages/>`, você está permitindo que a VM utilize páginas grandes de memória, reduzindo a sobrecarga do sistema e melhorando a eficiência.

### Limite de E/S com Blkiotune

O `blkiotune` controla o acesso à E/S da VM. Ao definir o peso (`weight`), você ajusta a prioridade relativa de acesso ao disco entre várias máquinas virtuais.

Exemplo de configuração do Cgroups no XML da VM:

```xml
<blkiotune>
  <weight>200</weight>
</blkiotune>
```

Neste exemplo, a VM tem um peso de 200, indicando uma prioridade mais alta em comparação com máquinas virtuais com pesos menores.

### Criação de Cgroups Manuais

Você também pode criar grupos de controle manualmente usando comandos como `cgcreate` e `cgset`. Essa abordagem é mais flexível, permitindo ajustes dinâmicos em tempo real.

Exemplo de criação de um grupo de controle para uma VM chamada "nome-da-vm":

```bash
sudo cgcreate -g cpu,memory:/nome-da-vm
sudo cgset -r memory.limit_in_bytes=2G nome-da-vm
```

Neste exemplo, estamos limitando a VM a 2 gigabytes de memória utilizando Cgroups.

## Migração de Máquinas Virtuais

A migração de máquinas virtuais entre hosts é um recurso essencial para manutenção, balanceamento de carga e tolerância a falhas. O KVM oferece suporte a dois tipos de migração: ao vivo (live) e pausada (paused).

Exemplo de migração ao vivo:

```bash
virsh migrate --live --persistent --domain nome-da-vm qemu+ssh://host-destino/system
```

## Armazenamento com Pool de Armazenamento

Os **Pools de Armazenamento** no KVM facilitam a organização e gerenciamento de imagens de máquinas virtuais. Eles permitem que você agrupe várias imagens em um local centralizado, simplificando a gestão de armazenamento.

Exemplo de criação de um pool de armazenamento:

```bash
virsh pool-create-as --name nome-do-pool --type dir --target /caminho/para/armazenamento
```

## Conclusão

Exploramos alguns recursos avançados do KVM que aprimoram a administração e o desempenho de máquinas virtuais. A capacidade de criar snapshots, clonar VMs, gerenciar recursos com Cgroups, realizar migração e organizar o armazenamento com Pools de Armazenamento tornam o KVM uma escolha poderosa para ambientes de virtualização.

Este é apenas o começo. O KVM oferece uma ampla gama de recursos que podem ser explorados e ajustados de acordo com as necessidades específicas de cada ambiente. Se você está interessado em otimizar ainda mais sua infraestrutura virtual, continue aprofundando seus conhecimentos no KVM e suas possibilidades.

Espero que este artigo tenha sido útil para aprofundar seu entendimento sobre as capacidades avançadas do KVM. Fique à vontade para compartilhar suas experiências e dúvidas nos comentários abaixo!