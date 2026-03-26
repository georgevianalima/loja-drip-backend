# Loja Drip - Backend API

API RESTful para a **Loja Drip**, um e-commerce desenvolvido com Node.js, Express e PostgreSQL (Supabase).

## Tecnologias

| Tecnologia | Versão | Descrição |
|---|---|---|
| Node.js | — | Runtime JavaScript |
| Express | 5.x | Framework web |
| Sequelize | 6.x | ORM para PostgreSQL |
| PostgreSQL | — | Banco de dados (Supabase) |
| JWT | — | Autenticação via token |
| bcryptjs | — | Hash de senhas |
| dotenv | 17.x | Variáveis de ambiente |
| nodemon | 3.x | Hot-reload em dev |
| CORS | — | Controle de acesso cross-origin |

## Estrutura do Projeto

```
src/
├── app.js                  # Configuração do Express (middlewares e rotas)
├── server.js               # Ponto de entrada - inicia servidor e sincroniza DB
├── config/
│   └── database.js         # Configuração do Sequelize (host, porta, SSL, etc.)
├── controllers/
│   ├── AuthController.js   # Geração de token JWT
│   ├── CategoryController.js # CRUD de categorias
│   ├── ProductController.js  # CRUD de produtos
│   └── UserController.js    # CRUD de usuários
├── database/
│   └── index.js            # Inicialização do Sequelize e carregamento dos models
├── middleware/
│   └── auth.js             # Middleware de autenticação JWT (Bearer Token)
├── models/
│   ├── Category.js         # Model de Categoria (tabela: categorias)
│   ├── Product.js          # Model de Produto (tabela: produtos)
│   ├── ProductImage.js     # Model de Imagem do Produto (tabela: imagens_produtos)
│   ├── ProductOption.js    # Model de Opção do Produto (tabela: opcoes_produtos)
│   └── User.js             # Model de Usuário (tabela: usuarios)
├── routes/
│   ├── categoryRoutes.js   # Rotas de categorias
│   ├── productRoutes.js    # Rotas de produtos
│   └── userRoutes.js       # Rotas de usuários e autenticação
└── services/
    └── ProductService.js   # Lógica de negócio de produtos (busca, CRUD transacional)
```

## Configuracao

### 1. Clonar o repositório

```bash
git clone https://github.com/georgevianalima/loja-drip-backend.git
cd loja-drip-backend
```

### 2. Instalar dependências

```bash
npm install
```

### 3. Configurar variáveis de ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
# Server
PORT=3001

# JWT
JWT_SECRET=sua-chave-secreta
JWT_EXPIRES_IN=1d

# PostgreSQL (Supabase)
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

# Produção
npm start
```

O servidor sincroniza automaticamente o banco de dados ao iniciar (`sync({ alter: true })`).

## Endpoints da API

**Base URL:** `http://localhost:3001`

### Autenticação

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| POST | `/v1/usuario/token` | ❌ | Gera token JWT |

**Body:**
```json
{
  "email": "usuario@email.com",
  "password": "123456"
}
```

**Resposta:** `{ "token": "eyJhbG..." }`

> Para rotas protegidas, envie o header: `Authorization: Bearer <token>`

---

### Usuários (`/v1/usuario`)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| POST | `/v1/usuario` | ❌ | Criar usuário |
| GET | `/v1/usuario/:id` | ❌ | Buscar por ID |
| PUT | `/v1/usuario/:id` | ✅ | Atualizar usuário |
| DELETE | `/v1/usuario/:id` | ✅ | Deletar usuário |

**POST - Criar usuário:**
```json
{
  "firstname": "George",
  "surname": "Lima",
  "email": "george@email.com",
  "password": "123456",
  "confirmPassword": "123456"
}
```

---

### Categorias (`/v1/categoria`)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/v1/categoria/pesquisa` | ❌ | Buscar categorias (com filtros) |
| GET | `/v1/categoria/:id` | ❌ | Buscar por ID |
| POST | `/v1/categoria` | ✅ | Criar categoria |
| PUT | `/v1/categoria/:id` | ✅ | Atualizar categoria |
| DELETE | `/v1/categoria/:id` | ✅ | Deletar categoria |

**POST - Criar categoria:**
```json
{
  "nome": "Eletrônicos",
  "slug": "eletronicos",
  "use_in_menu": true
}
```

**GET - Query params de busca:**
| Param | Tipo | Descrição |
|---|---|---|
| `limit` | number | Itens por página (padrão: 12, use -1 para todos) |
| `page` | number | Página (padrão: 1) |
| `fields` | string | Campos separados por vírgula |
| `use_in_menu` | boolean | Filtrar por uso no menu |

---

### Produtos (`/v1/produto`)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/v1/produto/pesquisa` | ❌ | Buscar produtos (com filtros avançados) |
| GET | `/v1/produto/:id` | ❌ | Buscar por ID (com associações) |
| POST | `/v1/produto` | ✅ | Criar produto |
| PUT | `/v1/produto/:id` | ✅ | Atualizar produto |
| DELETE | `/v1/produto/:id` | ✅ | Deletar produto |

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

**GET - Query params de busca:**
| Param | Tipo | Descrição |
|---|---|---|
| `limit` | number | Itens por página (padrão: 12) |
| `page` | number | Página |
| `fields` | string | Campos separados por vírgula |
| `match` | string | Busca por nome ou descrição |
| `category_ids` | string | IDs separados por vírgula |
| `price-range` | string | Faixa de preço (ex: `50-200`) |
| `option[id]` | string | Filtro por opção |

## Models / Banco de Dados

### User (tabela: `usuarios`)
| Campo | Tipo | Observação |
|---|---|---|
| nome | STRING | obrigatório |
| sobrenome | STRING | obrigatório |
| email | STRING | obrigatório, único |
| senha | STRING | hash bcrypt automático |

### Category (tabela: `categorias`)
| Campo | Tipo | Observação |
|---|---|---|
| nome | STRING | obrigatório |
| slug | STRING | obrigatório, único |
| use_in_menu | BOOLEAN | padrão: false |

### Product (tabela: `produtos`)
| Campo | Tipo | Observação |
|---|---|---|
| enabled | BOOLEAN | padrão: false |
| nome | STRING | obrigatório |
| slug | STRING | obrigatório, único |
| stock | INTEGER | padrão: 0 |
| description | TEXT | — |
| preco | FLOAT | obrigatório |
| price_with_discount | FLOAT | obrigatório |

### ProductImage (tabela: `imagens_produtos`)
| Campo | Tipo | Observação |
|---|---|---|
| enabled | BOOLEAN | padrão: true |
| path | STRING | obrigatório |
| product_id | FK | referência ao produto |

### ProductOption (tabela: `opcoes_produtos`)
| Campo | Tipo | Observação |
|---|---|---|
| titulo | STRING | obrigatório |
| shape | ENUM | `quadrado` ou `circulo` |
| radius | STRING | padrão: "0" |
| type | ENUM | `texto` ou `cor` |
| valores_do_produto | JSON | obrigatório |
| product_id | FK | referência ao produto |

### Relacionamentos
- **Product** ↔ **Category**: N:N (tabela pivô `produtos_categorias`)
- **Product** → **ProductImage**: 1:N
- **Product** → **ProductOption**: 1:N

## Autenticacao

A API usa **JWT (JSON Web Token)** com middleware Bearer Token:

1. Faça login em `POST /v1/usuario/token` para obter o token
2. Envie o token no header das rotas protegidas:
   ```
   Authorization: Bearer eyJhbG...
   ```
3. Token expira conforme configurado em `JWT_EXPIRES_IN` (padrão: 1 dia)

## Testes

O projeto utiliza **Jest** e **Supertest** para testes automatizados da API.

### Executar os testes

```bash
npm test
```

### Arquivos de teste

| Arquivo | Descricao |
|---|---|
| `tests/user.test.js` | Testes do CRUD de usuarios e autenticacao |
| `tests/category.test.js` | Testes do CRUD de categorias |
| `tests/product.test.js` | Testes do CRUD de produtos |

Os testes rodam com `--runInBand` para execucao sequencial, evitando conflitos no banco de dados.

## Autor

**George Viana Lima**

---

Desenvolvido durante o curso **Geracao Tech 3.0 - Full Stack**
