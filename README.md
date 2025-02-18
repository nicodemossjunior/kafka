# Projeto KAFKA

Este projeto é composto por quatro módulos implementados com Spring Boot: `payment-service` (produtor), `json-consumer` (consumidor), `str-producer` (produtor) e `str-consumer` (consumidor). O objetivo é demonstrar a comunicação entre serviços utilizando o Apache Kafka.

## Table of Contents

- [Estrutura do Projeto](#estrutura-do-projeto)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Uso](#uso)
- [Endpoints](#endpoints)
- [Deploy](#deploy)
- [Contribuição](#contribuição)

## Estrutura do Projeto

- `payment-service`: Módulo produtor que envia mensagens para o Kafka.
- `json-consumer`: Módulo consumidor que recebe e processa mensagens do Kafka.
- `str-producer`: Módulo produtor que envia mensagens de texto para o Kafka.
- `str-consumer`: Módulo consumidor que recebe e processa mensagens de texto do Kafka.

## Pré-requisitos

- Docker
- Docker Compose
- JDK 17
- Maven

## Instalação

Clone o repositório e navegue até o diretório do projeto:

```sh
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_DIRETORIO>
```

## Configuração

### Docker Compose

O arquivo [docker-compose.yml](docker-compose.yml) configura os serviços necessários para o Kafka:

```yaml
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    networks:
      - broker-kafka
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    networks:
      - broker-kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  kafdrop:
    image: obsidiandynamics/kafdrop:latest
    networks:
      - broker-kafka
    depends_on:
      - kafka
    ports:
      - "19000:9000"
    environment:
      KAFKA_BROKERCONNECT: kafka:29092
  
  payment:
    image: nicodemossjunior/payment-service:1.0.1
    networks:
      - broker-kafka
    depends_on:
      - kafka
    ports:
      - "8000:8000"
    environment:
      KAFKA_HOST: kafka:29092
  
  consumer:
    image: nicodemossjunior/json-consumer:1.0.1
    networks:
      - broker-kafka
    depends_on:
      - kafka
      - payment
    environment:
      KAFKA_HOST: kafka:29092

networks:
  broker-kafka:
    driver: bridge
```

### Configuração do Kafka

Os módulos utilizam o Kafka para comunicação. As configurações do Kafka estão definidas nos arquivos application.yml de cada módulo.


## Uso

Passo 1: Iniciar os Serviços com Docker Compose

```sh
docker-compose up -d
```

Passo 2: Construir e Executar os Módulos

[payment-service](/payment-service/)
```sh
cd payment-service
./mvnw clean install
./mvnw spring-boot:run
```

[json-consumer](/json-consumer/)
```sh
cd json-consumer
./mvnw clean install
./mvnw spring-boot:run
```

[str-producer](/str-producer/)
```sh
cd str-producer
./mvnw clean install
./mvnw spring-boot:run
```

[str-consumer](/str-consumer/)
```sh
cd str-consumer
./mvnw clean install
./mvnw spring-boot:run
```

## Endpoints

[payment-service](/payment-service/)

- **POST /payments**: Envia um pagamento para o Kafka.

[json-consumer](/json-consumer/)

- **Kafka Listener**: Escuta mensagens do tópico payment-topic e processa os pagamentos.

[str-producer](/str-producer/)

- **POST /producer**: Envia uma mensagem de texto para o Kafka.

[str-consumer](/str-consumer/)

- **Kafka Listener**: Escuta mensagens do tópico str-topic e processa as mensagens de texto.

## Deploy

Para fazer o deploy dos serviços, você pode utilizar o Docker Compose conforme descrito na seção de configuração.

# Contribuição

Sinta-se à vontade para contribuir com este projeto. Faça um fork do repositório, crie um branch para suas alterações e envie um pull request.