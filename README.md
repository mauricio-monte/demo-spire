# SPIRE Demo

Este repositório contém uma demonstração do [SPIRE](https://spiffe.io/docs/latest/spiffe-about/overview/).  
Esta demonstração foi feita com base neste [roteiro](https://spiffe.io/docs/latest/try/spire101/), presente na documentação do SPIRE.  
Os binários utilizados para os SPIRE Server e o SPIRE Agent foram as releases 1.6.4 do .tar.gz disponível [aqui](https://spiffe.io/downloads/#spire-releases).  
Os binários utilizados para o watcher, o server_workload e o client_workload foram compilados a partir [deste código](https://github.com/spiffe/go-spiffe/tree/main/v2/examples/spiffe-watcher) e [deste código](https://github.com/spiffe/go-spiffe/tree/main/v2/examples/spiffe-http).

## Requisitos

Para executar esta demonstração, é necessário ter instalado docker e docker-compose na sua máquina

## Roteiro

A seguir estão os passos para configurar o ambiente e executar a demonstração do SPIRE:

1. Clone este repositório
2. Abra dois terminais da pasta raiz do repositório

### 3. Levantar Containers

Em um terminal, execute o container com o spire server usando o comando:

```
docker-compose run --rm --name spire_server spire_server
```

No outro terminal, execute o container com o agent e os serviços usando o comando:

```
docker-compose run --rm --name workloads workloads
```

### 4. Iniciar SPIRE Server

No terminal com o container do spire server, digite os seguintes comandos para...

Iniciar o servidor:

```
./spire-server run -config ./server.conf -logFile server.log &
```

Verificar que o servidor está rodando:

```
./spire-server healthcheck
```

### 5. Fazer a atestação do agent

Para realizar o processo de atestação do agent, usaremos o metódo de joinToken.
Neste metódo, o server gera um código aleatório de uso único e o agent se autentica apresentando este código ao servidor.

No terminal do servidor, execute o seguinte comando para gerar um código aleatório de uso único para autenticar um agent:

```
./spire-server token generate -spiffeID spiffe://projeto_sd.org/agent
```

Veja que no presente momento, nenhum agent está atestado com o comando:

```
./spire-server agent list
```

No terminal do agent, execute o seguinte o comando substituindo pelo código gerado no passo anterior:

```
./spire-agent run -config ./agent.conf -logFile ./logs/agent.log -joinToken <token_gerado> &
```

No terminal do servidor, verifique que o agent está atestado digitando o comando:

```
./spire-server agent list
```

### 6.Fazer a atestação do workload

No terminal do server, digite os seguintes comandos para criar os entries a serem usados pelo serviços que serão registrados

Para o servidor:

```
./spire-server entry create \
    -spiffeID spiffe://projeto_sd.org/server \
    -parentID spiffe://projeto_sd.org/agent \
    -selector unix:user:server-workload-user
```

Para o cliente:

```
./spire-server entry create \
    -spiffeID spiffe://projeto_sd.org/client \
    -parentID spiffe://projeto_sd.org/agent \
    -selector unix:user:client-workload-user
```

Para o watcher:

```
./spire-server entry create \
    -spiffeID spiffe://projeto_sd.org/spiffe-watcher \
    -parentID spiffe://projeto_sd.org/agent \
    -selector unix:user:spiffe-watcher-user
```

```
./spire-server entry show
```

### 7. Executar os serviços que irão se atestar, utilizando os usuários criados

Para inicializar o watcher:

```
su -c "./watcher" spiffe-watcher-user >> ./logs/watcher.log 2>&1 &
```

Para inicializar o server:

```
su -c "./server_workload" server-workload-user >> ./logs/server.log 2>&1 &
```

Para inicializar o client que vai falhar em se autenticar:

```
su -c "./client_workload" unnauthorized-user >> ./logs/unnauthorized_client.log 2>&1 &
```

Para inicializar o client que vai conseguir obter o seu SVID e se comunicar com o servidor:

```
su -c "./client_workload" client-workload-user >> ./logs/client.log 2>&1 &
```

Acompanhe os resultados pelos logs

### Limpar containers do docker

Para limpar containers, images e volumes criados:

```
docker-compose down --rmi all -v
```
