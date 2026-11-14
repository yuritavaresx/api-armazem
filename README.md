# 🛒 API Armazém

API REST para gerenciar **produtos**, **clientes** e **pedidos** de um pequeno armazém. O projeto foi construído em **TypeScript + Node.js + Express + SQLite**, seguindo uma **arquitetura em camadas**, e desenvolvido passo a passo em 10 etapas com **Git e GitHub**.

> Projeto da disciplina de **Linguagem de Programação** (Análise e Desenvolvimento de Sistemas, FATEC).

---

## 📌 Status do projeto

✅ **A API está completa em relação ao escopo descrito em `Analise.md`.**

O que ela faz hoje:

- Cadastra, consulta, atualiza e remove **produtos** e **clientes**
- Registra **pedidos** com vários itens, calculando o total automaticamente
- Guarda o **preço do produto no momento da venda**, então mudar o preço depois não altera pedidos antigos
- Valida todos os dados de entrada e devolve **erros claros** com o código HTTP correto
- Já vem com **dados de exemplo (seed)** e um arquivo `requests.http` para testar tudo com um clique

---

## 🧰 Tecnologias

| Tecnologia | Para que serve |
|---|---|
| **Node.js** (v20+) | Ambiente de execução do JavaScript no servidor |
| **TypeScript** | JavaScript com tipos, que evita erros antes de rodar |
| **Express** | Framework web: rotas, requisições e respostas HTTP |
| **SQLite** (`sqlite3`) | Banco de dados em arquivo, sem precisar instalar servidor |
| **Zod** | Validação dos dados que chegam na API |
| **tsx** | Executa TypeScript direto, com recarga automática no desenvolvimento |
| **Git + GitHub** | Controle de versão e colaboração (branches, commits e Pull Requests) |

---

## 🏗️ Arquitetura em camadas

Cada camada tem **uma única responsabilidade**. Isso deixa o código organizado, fácil de entender e de testar.

```
Requisição HTTP
      │
      ▼
┌─────────────┐   Associa cada URL e método HTTP a um controller
│   Routes    │
└──────┬──────┘
       ▼
┌─────────────┐   Recebe a requisição, valida o corpo com Zod
│ Controllers │   e devolve a resposta HTTP
└──────┬──────┘
       ▼
┌─────────────┐   Regras de negócio (ex.: "e-mail é único",
│  Services   │   "não remover produto que já foi vendido")
└──────┬──────┘
       ▼
┌─────────────┐   Único lugar que escreve SQL
│Repositories │
└──────┬──────┘
       ▼
   🗄️ SQLite

   Qualquer erro, em qualquer camada, vai para o
   middleware errorHandler, que monta a resposta HTTP.
```

---

## 📁 Estrutura de pastas

```
api-armazem/
├── src/
│   ├── config/
│   │   └── database.ts          # Conexão compartilhada com o SQLite
│   ├── controllers/             # Camada HTTP (product, customer, order)
│   ├── database/
│   │   ├── schema.sql           # Criação das 4 tabelas
│   │   ├── init.ts              # Cria o banco a partir do schema
│   │   └── seed.ts              # Popula o banco com dados de exemplo
│   ├── errors/
│   │   └── app-error.ts         # Erro com mensagem + status HTTP
│   ├── middlewares/
│   │   └── error-handler.ts     # Tratamento centralizado de erros
│   ├── models/                  # Interfaces TypeScript (formato dos dados)
│   ├── repositories/            # Acesso ao banco (SQL)
│   ├── routes/                  # Rotas de cada entidade + agregador (index.ts)
│   ├── schemas/                 # Validações com Zod
│   ├── services/                # Regras de negócio
│   ├── app.ts                   # Configura o Express
│   └── server.ts                # Sobe o servidor na porta 3000
├── requests.http                # Requisições prontas para testar a API
├── package.json
├── tsconfig.json
└── README.md
```

---

## 🚀 Como rodar o projeto

### Pré-requisitos
- [Node.js](https://nodejs.org) versão **20 ou superior** (use a LTS)
- [Git](https://git-scm.com)

### Passo a passo

```bash
# 1. Clonar o repositório
git clone https://github.com/yuritavaresx/api-armazem.git
cd api-armazem

# 2. Instalar as dependências
npm install

# 3. Criar o banco de dados (gera o arquivo database.db)
npm run db:init

# 4. (Opcional) Popular com dados de exemplo
npm run db:seed

# 5. Subir o servidor em modo desenvolvimento
npm run dev
```

Se tudo deu certo, o terminal mostra:

```
Servidor rodando em http://localhost:3000
```

Teste no navegador abrindo **http://localhost:3000/products**.

### Scripts disponíveis

| Comando | O que faz |
|---|---|
| `npm run dev` | Sobe o servidor com recarga automática ao salvar arquivos |
| `npm run build` | Compila o TypeScript para JavaScript na pasta `dist/` |
| `npm start` | Executa a versão compilada (`dist/server.js`) |
| `npm run db:init` | Cria o banco SQLite e as tabelas |
| `npm run db:seed` | Apaga os dados atuais e insere 2 clientes e 3 produtos de exemplo |

> ⚠️ O `db:seed` **limpa as tabelas** antes de inserir. Use apenas em ambiente de desenvolvimento.

---

## 🗃️ Modelo de dados

```
customers (clientes)        products (produtos)
─────────────────────       ─────────────────────
id (PK)                     id (PK)
name                        name
email (único)               description (opcional)
phone (opcional)            price (> 0)
created_at                  created_at
     │                           │
     │ 1                         │ 1
     │                           │
     ▼ N                         ▼ N
orders (pedidos)            order_items (itens do pedido)
─────────────────────       ─────────────────────────────
id (PK)                     id (PK)
customer_id (FK) ◄──────┐   order_id (FK)  → orders
total                   └── product_id (FK) → products
created_at                  quantity (> 0)
                            unit_price (> 0)  ← preço fotografado
```

- Um **cliente** tem vários **pedidos**.
- Um **pedido** tem vários **itens**, e cada item aponta para um **produto**.
- As chaves estrangeiras são ativadas no SQLite com `PRAGMA foreign_keys = ON`.

---

## 📡 Endpoints

URL base: `http://localhost:3000`

### Produtos

| Método | Rota | Descrição | Respostas |
|---|---|---|---|
| `GET` | `/products` | Lista todos os produtos | `200` |
| `GET` | `/products/:id` | Busca um produto | `200`, `404` |
| `POST` | `/products` | Cria um produto | `201`, `400` |
| `PUT` | `/products/:id` | Atualiza (parcial) | `200`, `400`, `404` |
| `DELETE` | `/products/:id` | Remove um produto | `204`, `404`, `409` |

### Clientes

| Método | Rota | Descrição | Respostas |
|---|---|---|---|
| `GET` | `/customers` | Lista todos os clientes | `200` |
| `GET` | `/customers/:id` | Busca um cliente | `200`, `404` |
| `GET` | `/customers/:id/orders` | Lista os pedidos do cliente | `200`, `404` |
| `POST` | `/customers` | Cria um cliente | `201`, `400`, `409` |
| `PUT` | `/customers/:id` | Atualiza (parcial) | `200`, `400`, `404`, `409` |
| `DELETE` | `/customers/:id` | Remove um cliente | `204`, `404`, `409` |

### Pedidos

| Método | Rota | Descrição | Respostas |
|---|---|---|---|
| `GET` | `/orders` | Lista todos os pedidos | `200` |
| `GET` | `/orders/:id` | Busca um pedido **com seus itens** | `200`, `404` |
| `POST` | `/orders` | Cria um pedido | `201`, `400`, `404` |

> Pedidos **não têm** `PUT` nem `DELETE` de propósito: um pedido é um registro histórico de venda.

---

## 💡 Exemplos de uso

### Criar um produto

```http
POST /products
Content-Type: application/json

{
  "name": "Açúcar Refinado 1kg",
  "description": "Açúcar cristal refinado",
  "price": 5.90
}
```

Resposta `201 Created`:

```json
{
  "id": 4,
  "name": "Açúcar Refinado 1kg",
  "description": "Açúcar cristal refinado",
  "price": 5.9,
  "created_at": "2026-11-14 18:30:00"
}
```

### Criar um pedido

Você informa **apenas** o cliente e os produtos com as quantidades. O total e o preço de cada item são calculados pelo servidor.

```http
POST /orders
Content-Type: application/json

{
  "customer_id": 1,
  "items": [
    { "product_id": 1, "quantity": 2 },
    { "product_id": 2, "quantity": 3 }
  ]
}
```

Resposta `201 Created`:

```json
{
  "id": 1,
  "customer_id": 1,
  "total": 86.3,
  "created_at": "2026-11-14 18:35:00",
  "items": [
    { "id": 1, "order_id": 1, "product_id": 1, "quantity": 2, "unit_price": 28.9 },
    { "id": 2, "order_id": 1, "product_id": 2, "quantity": 3, "unit_price": 9.5 }
  ]
}
```

Conta feita pelo sistema: `2 × 28,90 + 3 × 9,50 = 86,30`.

### Exemplo de erro de validação

```http
POST /products
Content-Type: application/json

{ "price": 10 }
```

Resposta `400 Bad Request`:

```json
{
  "error": "Dados inválidos",
  "details": [ { "path": ["name"], "message": "Invalid input: expected string, received undefined" } ]
}
```

---

## 📏 Regras de negócio implementadas

| Código | Regra | Resposta quando violada |
|---|---|---|
| **RN01** | O e-mail do cliente é **único** (no cadastro e na atualização) | `409` |
| **RN05** | O preço do produto é **fotografado** no pedido (`unit_price`) | – |
| **RN06** | Cliente que **já tem pedidos** não pode ser removido | `409` |
| **RN07** | Produto que **já está em algum pedido** não pode ser removido | `409` |

Além delas:

- Todo pedido precisa ter **pelo menos 1 item**
- Cliente e produtos de um pedido precisam **existir** (senão `404`)
- Preços e quantidades precisam ser **positivos**
- O **total do pedido** é sempre calculado pelo servidor

---

## 🚦 Tratamento de erros

Todo erro, de qualquer camada, chega ao middleware `errorHandler`, que o transforma em uma resposta padronizada:

| Situação | Status | Exemplo de corpo |
|---|---|---|
| Dados inválidos (Zod) | `400` | `{ "error": "Dados inválidos", "details": [...] }` |
| Recurso não encontrado | `404` | `{ "error": "Produto não encontrado" }` |
| Conflito com regra de negócio | `409` | `{ "error": "E-mail já cadastrado" }` |
| Erro inesperado | `500` | `{ "error": "Erro interno no servidor" }` |

---

## 🧪 Como testar

### Opção 1: arquivo `requests.http` (recomendado)

1. Instale a extensão **REST Client** (Huachao Mao) no VS Code.
2. Rode `npm run dev` em um terminal.
3. Abra o `requests.http` e clique em **Send Request** acima de cada requisição.

O arquivo traz um exemplo para **todos os endpoints** e 3 casos de erro (`400`, `404` e `409`).

### Roteiro de teste sugerido

1. `GET /products` e `GET /customers` → devolvem os dados do seed
2. `POST /orders` → `201`, com `total` = **86.3**
3. `PUT /products/1` mudando o preço → `200`
4. `GET /orders/1` → o `unit_price` do item continua **28.9** (RN05 funcionando)
5. `DELETE /products/1` e `DELETE /customers/1` → `409` (RN07 e RN06)

### Opção 2: pelo terminal

```bash
curl http://localhost:3000/products
```

No PowerShell do Windows, use `curl.exe` em vez de `curl`.

---

## 🌿 Fluxo de desenvolvimento (Git e GitHub)

O projeto foi construído em **10 etapas**, cada uma em uma **branch própria**, com commits pequenos no padrão **Conventional Commits** e integrada à `main` por **Pull Request**.

| Etapa | Branch | O que foi feito |
|---|---|---|
| 1 | `chore/setup-inicial` | `.gitignore`, `package.json`, TypeScript e estrutura de pastas |
| 2 | `feat/banco-de-dados` | Schema SQL, script de inicialização e conexão com o SQLite |
| 3 | `feat/models` | Interfaces de Product, Customer e Order |
| 4 | `feat/erros-e-validacoes` | `AppError` e schemas Zod |
| 5 | `feat/product` | CRUD de produtos (repository, service, controller e routes) |
| 6 | `feat/customer` | CRUD de clientes com RN01 e RN06 |
| 7 e 8 | `feat/rotas-e-middleware` | Pedidos com RN05, agregador de rotas e middleware de erros |
| 9 | `feat/app-e-server` | `app.ts` e `server.ts` (API no ar) |
| 10 | `feat/seed-e-requests` | Seed de dados e `requests.http` |
| – | `docs/readme` | Este README |

### Padrão de commits

```
tipo(escopo): descrição curta no imperativo
```

| Tipo | Uso | Exemplo |
|---|---|---|
| `feat` | Funcionalidade nova | `feat(product): adiciona rotas de produtos` |
| `fix` | Correção de bug | `fix(customer): converte phone undefined em null ao criar cliente` |
| `chore` | Configuração e manutenção | `chore: adiciona .gitignore` |
| `docs` | Documentação | `docs: adiciona requests.http com exemplos de todos os endpoints` |

---

## ⚠️ Limitações conhecidas

- A criação de um pedido grava o pedido e depois cada item, **sem transação**. Se algo falhar no meio, pode sobrar um pedido incompleto.
- Não há autenticação nem paginação.
- A porta (`3000`) está fixa no código.

---

## 👤 Autor

**Yuri**
Estudante de Análise e Desenvolvimento de Sistemas, FATEC
GitHub: [@yuritavaresx](https://github.com/yuritavaresx)
