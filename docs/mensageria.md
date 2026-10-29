# Q2 — Comunicação e Mensageria do BookHub

## 1. Objetivo

A proposta desta etapa é acrescentar uma camada de comunicação orientada a eventos ao BookHub, permitindo que módulos internos ou sistemas externos possam trocar informações de forma desacoplada.

A solução proposta utiliza um serviço de mensageria como intermediário entre o módulo responsável pela publicação dos eventos e os módulos consumidores.

## 2. Cenário de negócio

Um exemplo de evento do domínio é a publicação de um novo livro.

Quando um usuário publica um livro, o sistema pode gerar o evento `BookCreated`. Esse evento pode ser recebido por outros módulos interessados, como um módulo de notificações ou um módulo de estatísticas.

Dessa forma, o módulo responsável pela criação do livro não precisa conhecer diretamente todos os consumidores do evento.

Fluxo conceitual:

```text
Usuário
   |
   v
BookHub API
   |
   | cria livro
   v
BookCreated
   |
   v
Message Broker
   |
   +-------------------+--------------------+
   |                   |                    |
   v                   v                    v
Notificações       Estatísticas       Outros módulos
```

## 3. Escolha técnica

Como proposta de serviço de mensageria, foi escolhido o RabbitMQ.

O RabbitMQ atua como intermediário para recebimento e distribuição das mensagens. O produtor publica um evento e os consumidores podem processá-lo de forma independente.

A escolha está relacionada principalmente ao objetivo de reduzir o acoplamento entre módulos e permitir que novos consumidores sejam adicionados sem alterar diretamente o módulo que produz o evento.

> Observação: nesta etapa, o RabbitMQ é apresentado como escolha arquitetural da proposta. Não é necessário executar um broker para representar a arquitetura no trabalho.

## 4. Evento proposto

### BookCreated

Evento gerado quando um livro é publicado.

Exemplo conceitual de mensagem:

```json
{
  "event": "BookCreated",
  "bookId": 1,
  "userId": 10,
  "title": "Exemplo de livro"
}
```

Os dados acima representam apenas a estrutura conceitual do evento.

## 5. Componentes da comunicação

### Event Publisher

Responsável por publicar eventos no serviço de mensageria.

### Message Broker

Responsável por receber e distribuir as mensagens.

### Event Consumer

Responsável por receber e processar eventos de interesse.

### Módulos consumidores

Podem representar funcionalidades como notificações, estatísticas ou outros módulos que precisem reagir à publicação de um livro.

## 6. Representação C4 — Container

```text
+-------------------+
|      Usuário      |
+---------+---------+
          |
          | HTTP
          v
+-------------------+
|    BookHub API    |
|                   |
| Usuários          |
| Livros            |
| Comentários       |
| Curtidas          |
+---------+---------+
          |
          | Publica evento
          v
+-------------------+
|     RabbitMQ      |
|  Message Broker   |
+----+----------+---+
     |          |
     |          |
     v          v
+---------+  +--------------+
| Módulo  |  |    Módulo    |
| de      |  | estatísticas |
| notific.|  |              |
+---------+  +--------------+
```

## 7. Justificativa de negócio

A comunicação por eventos permite que novas funcionalidades reajam a acontecimentos importantes do BookHub sem aumentar o acoplamento entre os módulos.

Por exemplo, depois da publicação de um livro, uma funcionalidade de notificação pode informar outros usuários ou uma funcionalidade de estatísticas pode registrar a publicação. O módulo responsável pelo cadastro do livro não precisa executar diretamente essas funcionalidades.

Essa abordagem favorece a evolução do sistema, pois novos consumidores podem ser acrescentados conforme novas necessidades do negócio surgirem.

## 8. Justificativa técnica

A camada de mensageria separa o produtor dos consumidores dos eventos.

Com essa separação, o módulo de publicação pode continuar responsável pela criação do livro, enquanto o processamento das consequências desse evento fica a cargo dos consumidores.

A utilização de um message broker também cria uma fronteira clara de comunicação entre os componentes, permitindo organizar a troca de mensagens de maneira padronizada.

## 9. Estrutura proposta no projeto

```text
src/
└── infrastructure/
    └── messaging/
        ├── RabbitMQConnection.js
        ├── EventPublisher.js
        └── EventConsumer.js
```

Esses arquivos representam a separação arquitetural da responsabilidade de comunicação. A implementação completa da integração com o broker não é necessária para a representação desta etapa.

## 10. Relação com a arquitetura DDD

A mensageria fica na infraestrutura porque o serviço de comunicação é um detalhe técnico utilizado para transportar eventos.

Os casos de uso da aplicação podem solicitar a publicação de eventos, enquanto a infraestrutura fornece a implementação concreta do mecanismo de mensageria.

Assim, a proposta mantém a separação entre:

- regras e conceitos do domínio;
- casos de uso da aplicação;
- comunicação técnica e infraestrutura;
- interfaces HTTP.
