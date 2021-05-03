:spiral_calendar: Atualizado em 10 de abril de 2021 :heart:

<img align="right" alt="GIF" height="160px" src="https://github.com/rdeconti/rdeconti-resources/blob/main/Digital%20Innovation%20One%20-%20Logotipo.png" />

# Projeto Digital Innovation One Java

# Desenvolvimento de testes unitários para validar uma API REST de gerenciamento estoques de cerveja

- Este projeto foi proposto pela Digital Innovation One - Link do código original: https://github.com/wesleyfuchter/cqrs-quarkus-kafka
- Professor: Wesley Fuchter
- Aulas: https://web.digitalinnovation.one/lab/criando-um-sistema-de-orcamento-utilizando-cqrs-quarkus-kafka-e-deploy-no-eks/learning/e06e539f-50b2-4703-a55a-4870e4184e6a

# Descrição

Neste Labs vamos implantar uma aplicação escrita em Java/Kotlin no serviço Elastic Kubernetes Service da Amazon. A aplicação é um exemplo do padrão CQRS que contempla dois serviços Quarkus que se comunicam através de um barramento assíncrono usando o Kafka. Você vai aprender a criar os manifestos do Kubernetes para implantação no EKS e quais configurações são necessárias para ter o ambiente rodando em produção.

# Detalhes da aula (obtidos do link GitHub original)

## About CQRS - Command Query Responsibility Segregation

According with [Martin Folwer](https://martinfowler.com/bliki/CQRS.html)
> At its heart is the notion that you can use a different model to update information than the model you use to read information.
> For some situations, this separation can be valuable, but beware that for most systems CQRS adds risky complexity.

## The application

Simulates a bank account scenario where an end user adds a income or expense transaction, and it is processed in a ascyncronous event sourcing and CQRS architecture to recalculate the user's bank account balance. The user can also request the balance of it's account. Down here you can see the design:

![Design](/images/design.png)

## Deploying the external services

```
docker-compose up -d --build
```
It will deploy four docker containers on your environment with MongoDB, PostgreSQL, Kafka and Zookepper (required by Kafka)

After deploying Kafka, you'll need to [create the topic on the Kafka cluster](https://kafka.apache.org/quickstart). For example:

```
docker exec -it bankaccount-kafka \
  ./bin/kafka-topics.sh --create \
  --topic transactions \
  --zookeeper bankaccount-zookeeper:2181 \
  --replication-factor 1 \
  --partitions 1
```

## Testing the application

#### Running a CURL request to create a income transaction
```
curl -X POST -H "Content-Type: application/json" -d @income-transaction.json http://localhost:8080/transactions
```
#### Running a CURL request to create a expense transaction
```
curl -X POST -H "Content-Type: application/json" -d @expense-transaction.json http://localhost:8080/transactions
```
#### Running CURL request to fetch the balance
```
curl http://localhost:8081/balance\?accountId\=wesley | json_pp
```
#### Running [K6's](https://k6.io) simple performance test
````
k6 run --vus 10 --duration 60s performance-tests/income.js
k6 run --vus 10 --duration 60s performance-tests/expense.js
````
