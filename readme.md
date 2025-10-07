# Projeto Grafana com Docker Compose

Este repositório fornece um ambiente Docker Compose para executar o Grafana, configurado via variáveis de ambiente.  

## Pré-requisitos

- Docker e Docker Compose.
- Rede Docker `network-share` já criada:
- Container ctr-mysql rodando

```bash
docker network create --driver bridge network-share --subnet=<SUBNET>
```

## Criar a rede externa se ainda não existir

```bash
docker network create --driver bridge network-share --subnet=172.18.0.0/16
```

### OBSERVAÇÃO

**Ajuste a subnet conforme necessário.**

```plaintext
bskp/
└── ctr-grafana              # Projeto Grafana
     ├── data/               # Dados persistentes do Grafana
     ├── grafana-config/     # Configurações personalizadas do Grafana
     ├── maps/               # Arquivos geojson
     ├── docker-compose.yml  # Definição dos serviços Docker
     ├── .env.example        # Exemplo de variáveis de ambiente
     ├── Dockerfile          # Dockerfile
     └── README.md           # Documentação do serviço
```





- Banco MySQL criado com o nome definido em `DATABASE_NAME`

## Estrutura de arquivos

```plaintext
├── docker-compose.yml    # Definição dos serviços Docker
├── .env.example          # Exemplo de variáveis de ambiente
└── README.md             # Este documento
```

## Configuração das variáveis de ambiente

Copie o template e preencha os valores:

```bash
cp .env.example .env
```

Edite o `.env`:

```bash
# Servidor Grafana
SRV_GRAFANA_HOST=<srv-grafana-host>
RELEASE=<latest-or-specific-version>
ADMIN_PASSWD=<admin-password>

# Banco de dados (deve já existir)
DB_SERVER_HOST=<db-server-host>
DATABASE_TYPE=<mysql|postgres|sqlite3>
DATABASE_HOST=<db-host>:<db-port>
DATABASE_NAME=<grafana-db-name>
DATABASE_USER=<grafana-db-user>
DATABASE_PASSWORD=<grafana-db-password>

# Volumes (host:container)
VOL_GRAFANA_PATH=<host-data-dir>:/var/lib/grafana
VOL_CONFIG_PATH=<host-grafana-config-dir>:/etc/grafana

# Date-time
VOL_LOCALTIME=/etc/localtime:/etc/localtime:ro
VOL_TZ=/etc/timezone:/etc/timezone:ro

# Portas (host:container)
PORTS=<host-port>:<container-port>

# Rede estática
IPV4_ADDRESS=<desired-ipv4>
SUBNET=<network-subnet>

# LDAP / Active Directory (opcional)
LDAP_HOST=<adds-ip-ou-hostname>
LDAP_PORT=<389-or-636>
LDAP_USE_SSL=<true|false>
LDAP_START_TLS=<true|false>
LDAP_BIND_DN=<CN=svcGrafana,…>
LDAP_BIND_PASSWORD=<svc-account-password>
LDAP_SEARCH_BASE=<OU=Usuarios,DC=…,DC=…>
LDAP_SEARCH_FILTER="(sAMAccountName=%s)"
LDAP_ATTR_NAME=<givenName-or-displayName>
LDAP_ATTR_SN=<sn>
LDAP_ATTR_USER=<sAMAccountName-or-userPrincipalName>
LDAP_ATTR_MEMBEROF=<memberOf>
LDAP_ATTR_EMAIL=<mail>
LDAP_GROUP_ADMINS_DN=<CN=GRP_grafana_admins,…>
LDAP_GROUP_EDITORS_DN=<CN=GRP_grafana_editors,…>
LDAP_GROUP_VIEWER_DN=<CN=GRP_grafana_viewers,…>
```

## Subindo o serviço

Para iniciar o Grafana em segundo plano:

```bash
docker compose up -d
```

Verifique o container:

```bash
docker ps | grep ${SRV_GRAFANA_HOST}
```

Acompanhe logs:

```bash
docker logs -f ${SRV_GRAFANA_HOST}
```

## Parando e removendo containers

Para parar e limpar tudo:

```bash
docker compose down
```

## Explicação do `docker-compose.yml`

```yaml
services:
  srv-grafana:
    image: grafana/grafana:${RELEASE}
    env_file:
      - .env
    container_name: ${SRV_GRAFANA_HOST}
    environment:
      DB_SERVER_HOST: ${DB_SERVER_HOST}
      GF_SECURITY_ADMIN_PASSWORD: ${ADMIN_PASSWD}
      GF_DATABASE_TYPE: ${DATABASE_TYPE}
      GF_DATABASE_HOST: ${DATABASE_HOST}
      GF_DATABASE_NAME: ${DATABASE_NAME}
      GF_DATABASE_USER: ${DATABASE_USER}
      GF_DATABASE_PASSWORD: ${DATABASE_PASSWORD}
    ports:
      - ${PORTS}
    networks:
      network-share:
        ipv4_address: ${IPV4_ADDRESS}
    volumes:
      - ${VOL_GRAFANA_PATH}
      - ${VOL_CONFIG_PATH}
      - ${VOL_LOCALTIME}
      - ${VOL_TZ}
    restart: unless-stopped

networks:
  network-share:
    external: true
    ipam:
      config:
        - subnet: ${SUBNET}
```

## Observações

- Certifique-se de que o banco MySQL exista e as credenciais no `.env` estejam corretas.  
- Os diretórios mapeados em `VOL_GRAFANA_PATH` e `VOL_CONFIG_PATH` devem existir no host.  
- `restart: unless-stopped` garante que o Grafana seja reiniciado automaticamente.  

---  

Pronto! Basta ajustar seus valores no `.env` e levantar o stack. Qualquer dúvida, me avise!  
