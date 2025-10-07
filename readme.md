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

## Permissões das Pastas

```bash
chown -R 472:472 config data maps
chmod -R 755 config data maps
```

## Configuração das variáveis de ambiente

Copie o template e preencha os valores:

```bash
cp /bskp/ctr-grafana/.env.example /bskp/ctr-grafana/.env
```

Ajuste as variáveis no arquivo `.env`.

## Criando a base de dados

Acesse o container do ctr-mysql e crie a base de dados para o Grafana:

```bash
docker exec -it ctr-mysql mysql -u root -p
```

```sql
CREATE DATABASE IF NOT EXISTS grafana CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'grafana'@'localhost' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON grafana.* TO 'grafana'@'localhost';
CREATE USER IF NOT EXISTS 'grafana'@'%' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON grafana.* TO 'grafana'@'%';
FLUSH PRIVILEGES;
EXIT;
```

## Subindo o serviço

Para iniciar todos os containers em segundo plano:

```bash
docker compose up -d
```

Verifique o status:

```bash
docker ps | grep ctr-grafana
```

Acompanhe os logs do Server:

```bash
docker logs -f ctr-grafana
```

Qualquer dúvida ou sugestão, abra uma issue ou envie uma contribuição!

## Observações

- Certifique-se de que o banco MySQL exista e as credenciais no `.env` estejam corretas.  
- Os diretórios mapeados em `VOL_GRAFANA_PATH`, `VOL_CONFIG_PATH` e `VOL_MAPS_PATH` devem existir no host.  
- Ajuste as permissões dos diretórios conforme necessário para garantir que o Grafana possa ler e escrever neles.
