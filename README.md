# BookHub — Arquitetura Orientada a DDD

## 1. Domínio da aplicação

O BookHub é uma plataforma Web para publicação e interação com livros digitais. O sistema permite que usuários criem suas contas, realizem autenticação, publiquem livros, consultem livros disponíveis na plataforma e interajam com as publicações.

Entre as principais funcionalidades estão o gerenciamento de usuários e livros, publicação de arquivos e capas, consulta e busca de livros, comentários e curtidas. Dessa forma, o domínio central da aplicação está relacionado ao gerenciamento e compartilhamento de livros digitais e às interações realizadas pelos usuários dentro da plataforma.

## 2. Entidades principais

- **User:** representa o usuário da plataforma.
- **Book:** representa um livro publicado na plataforma.
- **Comment:** representa um comentário realizado sobre um livro.
- **Like:** representa uma interação de curtida realizada sobre um livro.

Relacionamentos principais:

- User publica Book.
- User realiza Comment.
- User realiza Like.
- Book recebe Comment.
- Book recebe Like.

## 3. Organização DDD

A proposta separa o sistema em quatro áreas principais:

### Domain

Concentra os conceitos e regras centrais do negócio. Essa camada não deve depender diretamente de Express, Sequelize ou do banco de dados.

### Application

Organiza os casos de uso da aplicação e coordena as operações necessárias para executar cada funcionalidade.

### Infrastructure

Concentra detalhes técnicos, como persistência no banco de dados, Sequelize, repositórios concretos e armazenamento de arquivos.

### Interfaces

Responsável pela comunicação HTTP com o exterior da aplicação, incluindo rotas, controllers e middlewares.

## 4. Estrutura proposta

```text
BookHub-DDD/
├── README.md
├── src/
│   ├── domain/
│   │   ├── entities/
│   │   ├── repositories/
│   │   └── services/
│   ├── application/
│   │   ├── use-cases/
│   │   └── dtos/
│   ├── infrastructure/
│   │   ├── database/
│   │   ├── repositories/
│   │   └── storage/
│   └── interfaces/
│       └── http/
│           ├── controllers/
│           ├── routes/
│           └── middlewares/
└── docs/
    └── arquitetura-ddd.md
```

## 5. Casos de uso considerados

- RegisterUser
- LoginUser
- CreateBook
- UpdateBook
- DeleteBook
- AddComment
- ToggleLike

A organização proposta permite separar as regras do domínio dos detalhes técnicos da aplicação, facilitando manutenção e evolução futuras.

## 6. Relação com o projeto original

Esta estrutura representa uma proposta de transformação arquitetural do projeto BookHub para uma organização orientada a DDD. O projeto Web original permanece como referência da aplicação e deve estar disponível em seu respectivo repositório GitHub.
