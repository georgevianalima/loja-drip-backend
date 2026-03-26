# Loja Drip - Backend API

API RESTful para a **Loja Drip**, um e-commerce de moda desenvolvido com Node.js, Express e PostgreSQL (Supabase). A API oferece gerenciamento completo de usuarios, categorias e produtos, com autenticacao JWT, busca avancada com filtros, relacionamentos N:N e operacoes transacionais.

---

## Indice

- [Tecnologias](#tecnologias)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Configuracao e Instalacao](#configuracao-e-instalacao)
- [Scripts Disponiveis](#scripts-disponiveis)
- [Endpoints da API](#endpoints-da-api)
- [Banco de Dados](#banco-de-dados)
- [Testes](#testes)
- [Autor](#autor)

---

## Tecnologias

| Tecnologia | Versao | Finalidade |
|---|---|---|
| Node.js | 18+ | Runtime JavaScript |
| Express | 5.x | Framework web |
| Sequelize | 6.x | ORM para PostgreSQL |
| PostgreSQL | - | Banco de dados (hospedado no Supabase) |
| JWT | - | Autenticacao via token |
| bcryptjs | 3.x | Hash de senhas |
| dotenv | 17.x | Variaveis de ambiente |
| Jest | 30.x | Framework de testes |
| Supertest | 7.x | Testes de requisicoes HTTP |
| nodemon | 3.x | Hot-reload em desenvolvimento |
| CORS | - | Controle de acesso cross-origin |

---

## Estrutura do Projeto

```
project-root/
├── package.json
├── .env                        # Variaveis de ambiente (nao versionado)
├── src/
│   ├── app.js                  # Configuracao do Express, middlewares e rotas
│   ├── server.js               # Ponto de entrada - inicia servidor e sincroniza DB
│   ├── config/
│   │   └── database.js         # Configuracao do Sequelize
│   ├── controllers/
│   │   ├── AuthController.js   # Geracao de token JWT (login)
│   │   ├── CategoryController.js # CRUD de categorias
│   │   ├── ProductController.js  # CRUD de produtos (delega ao Service)
│   │   └── UserController.js    # CRUD de usuarios
│   ├── database/
│   │   └── index.js            # Inicializacao do Sequelize e carregamento dos models
│   ├── middleware/
│   │   └── auth.js             # Middleware de autenticacao JWT (Bearer Token)
│   ├── models/
│   │   ├── Category.js         # Model de Categoria (tabela: categorias)
│   │   ├── Product.js          # Model de Produto (tabela: produtos)
│   │   ├── ProductImage.js     # Model de Imagem do Produto (tabela: imagens_produtos)
│   │   ├── ProductOption.js    # Model de Opcao do Produto (tabela: opcoes_produtos)
│   │   └── User.js             # Model de Usuario (tabela: usuarios)
│   ├── routes/
│   │   ├── categoryRoutes.js   # Rotas /v1/categoria
│   │   ├── productRoutes.js    # Rotas /v1/produto
│   │   └── userRoutes.js       # Rotas /v1/usuario e autenticacao
│   └── services/
│       └── ProductService.js   # Logica de negocio de produtos (busca, CRUD transacional)
└── tests/
    ├── category.test.js        # Testes da API de categorias
    ├── product.test.js         # Testes da API de produtos
    └── user.test.js            # Testes da API de usuarios
```

---

## Configuracao e Instalacao

### 1. Clonar o repositorio

```bash
git clone https://github.com/georgevianalima/loja-drip-backend.git
cd loja-drip-backend
```

### 2. Instalar dependencias

```bash
npm install
```

### 3. Configurar variaveis de ambiente

Crie um arquivo `.env` na raiz do projeto com as seguintes variaveis:

```env
PORT=3001

JWT_SECRET=sua-chave-secreta
JWT_EXPIRES_IN=1d

DB_HOST=db.xxxxx.supabase.co
DB_USER=postgres
DB_PASSWORD=sua-senha
DB_NAME=postgres
DB_PORT=5432
DB_DIALECT=postgres
```

### 4. Executar

```bash
# Desenvolvimento (com hot-reload)
npm run dev

# Producao
npm start
```

O servidor sincroniza automaticamente as tabelas do banco de dados ao iniciar usando `sync({ alter: true })`.

---

## Scripts Disponiveis

| Comando | Descricao |
|---|---|
| `npm start` | Inicia o servidor em modo producao |
| `npm run dev` | Inicia com nodemon (hot-reload) |
| `npm test` | Executa todos os testes com Jest |

---

## Endpoints da API

**Base URL:** `http://localhost:3001`

### Autenticacao

| Metodo | Rota | Auth | Descricao |
|---|---|---|---|
| POST | `/v1/usuario/token` | Nao | Gera token JWT |

**Request body:**
```json
{
  "email": "usuario@email.com",
  "password": "123456"
}
```

**Response (200):**
```json
{
  "token": "eyJhbG..."
}
```

Para acessar rotas protegidas, envie o header:
```
Authorization: Bearer <token>
```

---

### Usuarios - `/v1/usuario`

| Metodo | Rota | Auth | Descricao |
|---|---|---|---|
| POST | `/v1/usuario` | Nao | Criar usuario |
| GET | `/v1/usuario/:id` | Nao | Buscar usuario por ID |
| PUT | `/v1/usuario/:id` | Sim | Atualizar usuario |
| DELETE | `/v1/usuario/:id` | Sim | Deletar usuario |

**POST - Criar usuario:**
```json
{
  "firstname": "George",
  "surname": "Lima",
  "email": "george@email.com",
  "password": "123456",
  "confirmPassword": "123456"
}
```

**GET - Resposta (200):**
```json
{
  "id": 1,
  "firstname": "George",
  "surname": "Lima",
  "email": "george@email.com"
}
```

- A senha e automaticamente hasheada com bcrypt antes de ser salva.
- O campo `confirmPassword` e validado no controller (deve ser igual a `password`).
- Emails duplicados retornam status 400.

---

### Categorias - `/v1/categoria`

| Metodo | Rota | Auth | Descricao |
|---|---|---|---|
| GET | `/v1/categoria/pesquisa` | Nao | Buscar categorias com filtros |
| GET | `/v1/categoria/:id` | Nao | Buscar categoria por ID |
| POST | `/v1/categoria` | Sim | Criar categoria |
| PUT | `/v1/categoria/:id` | Sim | Atualizar categoria |
| DELETE | `/v1/categoria/:id` | Sim | Deletar categoria |

**POST - Criar categoria:**
```json
{
  "nome": "Eletronicos",
  "slug": "eletronicos",
  "use_in_menu": true
}
```

**GET /pesquisa - Query params:**

| Param | Tipo | Descricao |
|---|---|---|
| `limit` | number | Itens por pagina (padrao: 12, use -1 para todos) |
| `page` | number | Pagina (padrao: 1) |
| `fields` | string | Campos retornados, separados por virgula |
| `use_in_menu` | boolean | Filtrar categorias visiveis no menu |

**Resposta da busca (200):**
```json
{
  "data": [...],
  "total": 10,
  "limit": 12,
  "page": 1
}
```

---

### Produtos - `/v1/produto`

| Metodo | Rota | Auth | Descricao |
|---|---|---|---|
| GET | `/v1/produto/pesquisa` | Nao | Buscar produtos com filtros avancados |
| GET | `/v1/produto/:id` | Nao | Buscar produto por ID (com imagens, opcoes e categorias) |
| POST | `/v1/produto` | Sim | Criar produto com associacoes |
| PUT | `/v1/produto/:id` | Sim | Atualizar produto e associacoes |
| DELETE | `/v1/produto/:id` | Sim | Deletar produto |

**POST - Criar produto:**
```json
{
  "nome": "Camiseta Drip",
  "slug": "camiseta-drip",
  "enabled": true,
  "stock": 50,
  "description": "Camiseta estilosa da Loja Drip",
  "preco": 89.90,
  "price_with_discount": 69.90,
  "category_ids": [1],
  "images": [
    { "type": "image/jpeg", "content": "base64..." }
  ],
  "options": [
    {
      "title": "Tamanho",
      "shape": "quadrado",
      "radius": "0",
      "type": "texto",
      "values": ["P", "M", "G", "GG"]
    }
  ]
}
```

**PUT - Atualizar produto (parcial):**
```json
{
  "preco": 79.90,
  "options": [
    { "id": 1, "deleted": true },
    { "title": "Material", "type": "texto", "values": ["Algodao"] }
  ]
}
```

Na atualizacao, opcoes e imagens podem ser adicionadas, atualizadas ou removidas enviando `"deleted": true` com o `id`.

**GET /pesquisa - Query params:**

| Param | Tipo | Descricao |
|---|---|---|
| `limit` | number | Itens por pagina (padrao: 12, use -1 para todos) |
| `page` | number | Pagina (padrao: 1) |
| `fields` | string | Campos retornados, separados por virgula |
| `match` | string | Busca por nome ou descricao |
| `category_ids` | string | Filtrar por IDs de categorias, separados por virgula |
| `price-range` | string | Faixa de preco (ex: `50-200`) |
| `option[id]` | string | Filtrar por valores de uma opcao especifica |

A criacao e atualizacao de produtos utiliza **transacoes** do Sequelize para garantir a integridade dos dados. Caso qualquer etapa falhe, todas as alteracoes sao revertidas automaticamente.

---

## Banco de Dados

### User (tabela: `usuarios`)

| Campo | Tipo | Observacao |
|---|---|---|
| id | INTEGER | PK, auto-incremento |
| nome | STRING | obrigatorio |
| sobrenome | STRING | obrigatorio |
| email | STRING | obrigatorio, unico |
| senha | STRING | hash bcrypt automatico (hook beforeSave) |

### Category (tabela: `categorias`)

| Campo | Tipo | Observacao |
|---|---|---|
| id | INTEGER | PK, auto-incremento |
| nome | STRING | obrigatorio |
| slug | STRING | obrigatorio, unico |
| use_in_menu | BOOLEAN | padrao: false |

### Product (tabela: `produtos`)

| Campo | Tipo | Observacao |
|---|---|---|
| id | INTEGER | PK, auto-incremento |
| enabled | BOOLEAN | padrao: false |
| nome | STRING | obrigatorio |
| slug | STRING | obrigatorio, unico |
| stock | INTEGER | padrao: 0 |
| description | TEXT | opcional |
| preco | FLOAT | obrigatorio |
| price_with_discount | FLOAT | obrigatorio |

### ProductImage (tabela: `imagens_produtos`)

| Campo | Tipo | Observacao |
|---|---|---|
| id | INTEGER | PK, auto-incremento |
| enabled | BOOLEAN | padrao: true |
| path | STRING | obrigatorio |
| product_id | FK | referencia ao produto |

### ProductOption (tabela: `opcoes_produtos`)

| Campo | Tipo | Observacao |
|---|---|---|
| id | INTEGER | PK, auto-incremento |
| titulo | STRING | obrigatorio |
| shape | ENUM | `quadrado` ou `circulo` |
| radius | STRING | padrao: "0" |
| type | ENUM | `texto` ou `cor` |
| valores_do_produto | JSON | obrigatorio |
| product_id | FK | referencia ao produto |

### Relacionamentos

```
Product N:N Category      (tabela pivo: produtos_categorias)
Product 1:N ProductImage
Product 1:N ProductOption
```

Todas as tabelas possuem campos `created_at` e `updated_at` gerados automaticamente pelo Sequelize.

---

## Testes

O projeto possui testes automatizados usando **Jest** e **Supertest** que cobrem todos os endpoints da API.

### Executar os testes

```bash
npm test
```

### Cobertura dos testes

| Arquivo | O que testa |
|---|---|
| `tests/user.test.js` | Criacao, login (JWT), busca por ID, atualizacao, delecao de usuarios e validacao de email duplicado |
| `tests/category.test.js` | Criacao, listagem com filtros, busca por ID, atualizacao, delecao de categorias e validacao de autenticacao |
| `tests/product.test.js` | Criacao com categorias/opcoes, busca por ID com associacoes, atualizacao parcial com adicao/remocao de opcoes |

Os testes rodam com `--runInBand` para execucao sequencial, evitando conflitos no banco de dados. Cada suite recria as tabelas com `sync({ force: true })` para garantir isolamento.

**Total: 17 testes em 3 suites.**

---

## Autor

**George Viana Lima**

Desenvolvido durante o curso **Geracao Tech 3.0 - Full Stack**
