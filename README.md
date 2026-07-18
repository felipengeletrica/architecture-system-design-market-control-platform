# Architecture System Design — Market Infrastructure

Infraestrutura Docker da plataforma orientada a eventos para coleta, processamento, análise e controle preditivo de dados do mercado financeiro.

Nesta primeira etapa, o repositório contém somente a infraestrutura básica:

- RabbitMQ
- ClickHouse
- Grafana
- Rede Docker compartilhada
- Volumes persistentes

## Arquitetura inicial

```text
┌──────────────────┐
│ RabbitMQ         │
│ Filas e eventos  │
└────────┬─────────┘
         │
         │ market-network
         │
┌────────▼─────────┐
│ ClickHouse       │
│ Dados analíticos │
└────────┬─────────┘
         │
         │ consultas
         │
┌────────▼─────────┐
│ Grafana          │
│ Dashboards       │
└──────────────────┘
```

Nesta etapa ainda não são executados:

- coletor de dados;
- worker de ingestão;
- avaliação de modelos;
- GPC/MPC;
- paper trading;
- coleta de fundamentos.

Esses componentes serão adicionados gradualmente como serviços desacoplados.

## Estrutura do repositório

```text
architecture-system-design-market-infrastructure/
├── .env.example
├── .gitignore
├── compose.yaml
└── README.md
```

## Requisitos

Verifique se Docker e Docker Compose estão instalados:

```bash
docker --version
docker compose version
```

## Configuração

Crie o arquivo de ambiente local:

```bash
cp .env.example .env
```

O arquivo `.env` contém credenciais e configurações locais e não deve ser enviado ao GitHub.

Exemplo:

```env
COMPOSE_PROJECT_NAME=architecture-market

RABBITMQ_USER=market
RABBITMQ_PASSWORD=market
RABBITMQ_AMQP_PORT=5672
RABBITMQ_MANAGEMENT_PORT=15672

CLICKHOUSE_DATABASE=market
CLICKHOUSE_USER=market
CLICKHOUSE_PASSWORD=market
CLICKHOUSE_HTTP_PORT=8123
CLICKHOUSE_NATIVE_PORT=9000

GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=admin
GRAFANA_PORT=3000
```

Para uso em servidor ou ambiente público, altere todas as senhas padrão.

## Validar o Docker Compose

Antes de iniciar os containers:

```bash
docker compose config
```

Listar os serviços reconhecidos:

```bash
docker compose config --services
```

Saída esperada:

```text
rabbitmq
clickhouse
grafana
```

## Baixar as imagens

```bash
docker compose pull
```

## Iniciar a infraestrutura

```bash
docker compose up -d
```

A opção `-d` executa os containers em segundo plano.

## Verificar os containers

```bash
docker compose ps
```

Após alguns segundos, RabbitMQ e ClickHouse devem aparecer como saudáveis:

```bash
sleep 15
docker compose ps
```

## Acompanhar os logs

Todos os serviços:

```bash
docker compose logs -f
```

Somente RabbitMQ:

```bash
docker compose logs -f rabbitmq
```

Somente ClickHouse:

```bash
docker compose logs -f clickhouse
```

Somente Grafana:

```bash
docker compose logs -f grafana
```

Exibir somente as últimas 100 linhas:

```bash
docker compose logs --tail=100
```

## RabbitMQ

Interface de administração:

```text
http://localhost:15672
```

Credenciais padrão de desenvolvimento:

```text
Usuário: market
Senha: market
```

Porta AMQP:

```text
localhost:5672
```

Teste pelo terminal:

```bash
curl -u market:market   http://localhost:15672/api/overview
```

## ClickHouse

Interface HTTP:

```text
http://localhost:8123
```

Porta nativa:

```text
localhost:9000
```

Teste simples:

```bash
curl   -u market:market   "http://localhost:8123/?query=SELECT%201"
```

Saída esperada:

```text
1
```

Listar bancos:

```bash
curl   -u market:market   "http://localhost:8123/?query=SHOW%20DATABASES"
```

O banco `market` deve aparecer na resposta.

Também é possível executar o cliente dentro do container:

```bash
docker compose exec clickhouse   clickhouse-client   --user market   --password market
```

Dentro do cliente:

```sql
SHOW DATABASES;
SELECT version();
```

Para sair:

```text
exit
```

## Grafana

Interface:

```text
http://localhost:3000
```

Credenciais padrão de desenvolvimento:

```text
Usuário: admin
Senha: admin
```

Teste de saúde:

```bash
curl http://localhost:3000/api/health
```

O datasource ClickHouse e os dashboards serão configurados em uma etapa posterior.

## Rede Docker

Os serviços compartilham a rede:

```text
market-network
```

Dentro dessa rede, os containers são acessados pelos nomes dos serviços:

```text
rabbitmq
clickhouse
grafana
```

Exemplos futuros:

```text
amqp://market:market@rabbitmq:5672/
http://clickhouse:8123
http://grafana:3000
```

## Volumes persistentes

Os dados são preservados nos volumes:

```text
rabbitmq_data
clickhouse_data
grafana_data
```

Listar volumes:

```bash
docker volume ls
```

Exibir os volumes do projeto:

```bash
docker volume ls | grep architecture-market
```

## Parar a infraestrutura

Parar e remover os containers, mantendo os dados:

```bash
docker compose down
```

Iniciar novamente:

```bash
docker compose up -d
```

## Apagar também os dados

Atenção: este comando remove os volumes persistentes do projeto.

```bash
docker compose down -v
```

Use somente quando quiser reiniciar completamente RabbitMQ, ClickHouse e Grafana.

## Reiniciar um serviço individual

RabbitMQ:

```bash
docker compose restart rabbitmq
```

ClickHouse:

```bash
docker compose restart clickhouse
```

Grafana:

```bash
docker compose restart grafana
```

## Recriar um serviço individual

```bash
docker compose up -d --force-recreate rabbitmq
```

```bash
docker compose up -d --force-recreate clickhouse
```

```bash
docker compose up -d --force-recreate grafana
```

## Atualizar as imagens

```bash
docker compose pull
docker compose up -d
```

## Comandos de diagnóstico

Estado geral:

```bash
docker compose ps
```

Processos em execução:

```bash
docker compose top
```

Uso de recursos:

```bash
docker stats
```

Inspecionar a rede:

```bash
docker network inspect architecture-market_market-network
```

Verificar o RabbitMQ:

```bash
docker compose exec rabbitmq   rabbitmq-diagnostics -q ping
```

Verificar o ClickHouse:

```bash
docker compose exec clickhouse   clickhouse-client   --user market   --password market   --query "SELECT 1"
```

## Versionamento no Git

Adicionar os arquivos:

```bash
git add   .gitignore   .env.example   compose.yaml   README.md
```

Criar o commit:

```bash
git commit   -m "feat: add RabbitMQ ClickHouse and Grafana infrastructure"
```

Enviar ao GitHub:

```bash
git push origin main
```

## Atualizar o submódulo no repositório principal

Depois de enviar o commit da infraestrutura:

```bash
cd ../..
```

Confira:

```bash
git status
```

O submódulo deve aparecer como modificado.

Atualize a referência:

```bash
git add platform/infrastructure
```

```bash
git commit   -m "chore: update infrastructure submodule"
```

```bash
git push origin main
```

## Próximas etapas

A evolução planejada é:

```text
1. Criar tabelas e migrações do ClickHouse
2. Provisionar o datasource ClickHouse no Grafana
3. Criar contratos de eventos RabbitMQ
4. Adicionar o coletor de dados de mercado
5. Adicionar o worker de ingestão
6. Adicionar avaliação de modelos
7. Adicionar GPC/MPC e paper trading
8. Criar dashboards de compra, venda, patrimônio e drawdown
```

## Aviso

Este projeto é destinado a estudo de arquitetura, processamento de dados e controle preditivo. Nenhuma ordem real de compra ou venda é enviada a corretoras nesta etapa.
---

## Debug do coletor no PyCharm

O coletor orientado a classes está em:

```text
services/data-collector
```

A documentação completa de instalação, variáveis, breakpoints, RabbitMQ e
ClickHouse está em:

```text
services/data-collector/README.md
```

O projeto também inclui uma configuração compartilhada:

```text
.run/Collector - Debug PETR4.run.xml
```

Para o primeiro teste, mantenha a infraestrutura Docker ativa e execute o
coletor localmente no PyCharm com:

```text
RUN_ONCE=true
SYMBOL_FILTER=PETR4.SA
MAX_ASSETS=1
PUBLISH_ENABLED=true
RABBITMQ_URL=amqp://market:market@localhost:5672/%2F
```
