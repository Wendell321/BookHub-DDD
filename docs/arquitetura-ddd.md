# Arquitetura DDD — BookHub

## Domínio

O domínio do BookHub está relacionado ao gerenciamento e compartilhamento de livros digitais e às interações realizadas pelos usuários dentro da plataforma.

## Camadas

### 1. Domain

Responsável pelos conceitos centrais do negócio:

- User
- Book
- Comment
- Like
- regras e contratos do domínio

### 2. Application

Responsável pela organização dos casos de uso:

- cadastro de usuário;
- autenticação;
- criação de livro;
- atualização de livro;
- exclusão de livro;
- inclusão de comentário;
- alteração de curtida.

### 3. Infrastructure

Responsável pelas implementações técnicas:

- banco de dados;
- Sequelize;
- repositórios concretos;
- armazenamento de capas e arquivos dos livros.

### 4. Interfaces

Responsável pela entrada e saída HTTP:

- routes;
- controllers;
- middlewares.

## Diagrama conceitual

```text
                 ┌─────────────────────┐
                 │      Interfaces     │
                 │ Routes / Controllers│
                 │     Middlewares     │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │     Application     │
                 │      Use Cases      │
                 │ Register / Login    │
                 │ Book / Comment/Like │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │       Domain        │
                 │ User / Book         │
                 │ Comment / Like      │
                 │ Regras de negócio   │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │   Infrastructure    │
                 │ Database / Sequelize│
                 │ Repositories/Storage│
                 └─────────────────────┘
```

A proposta busca separar as regras de negócio dos detalhes relacionados à tecnologia utilizada para executar o sistema. O domínio concentra os conceitos principais da aplicação, enquanto a camada de aplicação organiza os casos de uso. A infraestrutura fica responsável pelas implementações técnicas, como persistência de dados e armazenamento de arquivos, e a camada de interfaces realiza a comunicação com os usuários e sistemas externos.

Essa separação facilita a manutenção e evolução do sistema, pois alterações em tecnologias de infraestrutura não precisam modificar diretamente as regras centrais do domínio.
