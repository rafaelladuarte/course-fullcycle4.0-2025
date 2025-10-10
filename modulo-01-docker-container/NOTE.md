# Docker e Containers

## Docker na Prática

### Introdução e primeiros passos

* O que são containers?

Unidade personalizada de software que empacota o código e todas as suas dependências para que um software possa ser executado de forma rápida, consistente e em qualquer ambiente.

* Caractéristicas
    * Imutabilidade (utilizam imagens)
    * Isolamento
    * Leve e rápido de iniciar, remover e parar
    * Executavel em Linux
    * Utiliza os recursos do Kernel do SO

* Qual a diferença entre Containers e Máquinas Virtuais?

Cada VM executa um SO completo, incluindo o kernel e hypervisor, já um container ele compartilha o kernel do SO do host, isolando apenas o app e suas dependências.

Com isso, um container utiliza menos recursos computacionais e possui menor tempo de inicialização, tendo maior eficiência em termos de uso de mémoria e CPU.

![alt text](modulo-01-docker-container/images/containerVSvm.png)

* Container Runtimes
É o software que permite a execução de containers.

```flowchart
A[LXCs - 2008] --> B[Docker - 2013] --> C[OCI - 2015] --> D[CNCF - 2017] -->

```
* Qual a diferençã entre Docker Engine e Dcoker Desktop

O Docker Engine é uma ferramenta Open Source já o o Docker Desktop é um porduto da Docker Inc, (Antiga dotCloud). O Docker é um container runtime que utiliza os recursos do Linux e outros componentes.

* Ferramentas/Produtos da Docker Inc.
    * Docker Desktop - 
    * Docker Hub - Repositório imagens de containers
    * Docker Build Cloud - Serviço Docker que acelera a criação de imagens
    * Docker Scout - Voltado para Segurança
    * Docker AI - Container para modelos de IA
    * InfoSiftr  - Docker Hub/ Docker Verified Publisher
    * Tilt - Gerenciamente de ambientes de desenvolvimento para aplicações
    * AtomicJar - Testcontainers/ Cloud/ Testcontainers Desktop
    * Mutagen.io - Synchronized file shares