# elasticsearch7-Kibana-apm-docker-compose-autenticação-tls

Este projeto vem com a intensão de adaptar aos ambientes que ainda não estão atualizados para o novo elasticsearch 8 e necessita de aumentar a segurança com login de usuário e criptografia entre os node do cluster elasticsearch. Mas é importante salientar que a atualização é muito importante e irei  adicionar depois o projeto de instalação do elasticsearch 8 também.

# Topologia do projeto

O projeto é constituído de 7 container sendo:
* Um cluster de elasticsearch com 4 nós
* Um container com setup para criar os certificados e definir usuários e senha que será utilizado para comunicação.
* Um container com o Kibana
* Um container com apm-server.

# Dependências 
Este projeto foi desenhado para rodar em uma VM ou servidor físico e necessita de ter alguns software e configurações mínimas realizadas, são eles:

* Instalar Docker e Docker-compose
* Configurar o sysctl max_map_count
    * systcl -w vm.max_map_count=262144

# Arquivos do projeto
* .env
    * definir as variáveis utilizar para criar o cluster elasticsearch na versão 7.17.6
* docker-compose.yml
    * Configuração do cluster em docker

# Definicões das variáveis.
* *ELASTIC_PASSWORD=vou-trocar-a-senha* => senha utilizada para acesso o elasticsearch
* *KIBANA_PASSWORD=vou-trocar-a-senha* => senha de conexão do Kibana com o elasticsearch
* *STACK_VERSION=7.17.6* => definir a versão do elasticsearch a ser instanciado
* *ES_PORT=9200* => Definir porta de comunicação do elasticsearch
* *KIBANA_PORT=5601* => definir porta de comunicação do Kibana
* *NODE_OPTIONS="--max-old-space-size=1024"* Definir quantidade de memória do Kibana
* *APM_PORT=8200* => definir porta de comunicação do apm
* *MEM_JAVA_OPTS=-Xms512m -Xmx512m* => Definir quantidade de memória utilizada por cada instância do elasticsearch, **não é a quantidade de memória do container**.
    * A abordagem limitando a memória no container será abordado no projeto do elasticsearch 8.
* *XPACK_SECURITY_ENCRYPTIONKEY=hymJlhMVbDQYZIwTWPpPiAJdjpAuyYFZ* => hash de criptografia para comunicação entre Kibana e elasticsearch
* *XPACK_REPORTING_ENCRYPTIONKEY=hymJlhMVbDQYZIwTWPpPiAJdjpAuyYFZ* => hash de criptografia para comunicação entre Kibana e elasticsearch
* *XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=hymJlhMVbDQYZIwTWPpPiAJdjpAuyYFZ* => hash de criptografia para comunicação entre Kibana e elasticsearch

**Os hash pode ser gerado um específico para cada variável.**

# hardening realizado na stack do elasticsearch

* *indices.memory.index_buffer_size=1000mb* => Aumentar o tamanho do buffer.
* *xpack.security.authc.api_key.enabled=true* => Habilitar o uso de API
* *apm-server.queue.mem.events=8192* => Aumentar o tamanho da fila de eventos do APM server.
* *apm-server.auth.api_key.limit=100* => definir limite de api sem token.
* *apm-server.rum.enabled=false* => desabilitar o uso de API sem o token.
* *apm-server.auth.api_key.enabled=true* => habilitar o uso de API para comunicação com apm-server 
* *SERVER_PUBLICBASEURL=http://ip-publico-do-Kibana:5602*  => Definir o domínio público o interno que será utilizado para acessar o Kibana.
* *XPACK_SECURITY_SESSION_IDLETIMEOUT=1h* => Adicionar um timeout para encerrar as sessões.

# Validação do docker compose

Todos os container possui um health check validando que todos os container estejam saudáveis e só inicia um novo container após o último está pronto.

## Hardening dos container
As opções abaixo aumenta a capacidade de trabalhar com arquivos para cada container do cluster.

```
ulimits:
    memlock:
    soft: -1
    hard: -1
    nproc: 65535
    nofile:
    soft: 65535
    hard: 65535

```

```
healthcheck:
    test:
    [
        "CMD-SHELL",
        "curl -s --cacert config/certs/ca/ca.crt http://localhost:9200 | grep -q 'missing authentication credentials'",
    ]
    interval: 10s
    timeout: 10s
    retries: 120
```
