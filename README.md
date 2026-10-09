# API de Pedidos

API REST para cadastrar, consultar e acompanhar pedidos, desenvolvida na disciplina **Desenvolvimento de Sistemas Distribuídos**. O projeto aplica organização em camadas, validação de dados e persistência em PostgreSQL, com execução por Docker Compose.

## Funcionalidades

- Criar pedidos com cliente, produto, quantidade e valor unitário.
- Calcular automaticamente o valor total com precisão decimal.
- Listar pedidos e consultar um pedido pelo identificador.
- Alterar o status para `CRIADO`, `CONFIRMADO` ou `CANCELADO`.
- Validar campos obrigatórios, textos vazios e valores maiores que zero.
- Manter os pedidos em um volume persistente do PostgreSQL.
- Disponibilizar documentação interativa da API pelo Swagger.

## Tecnologias

| Tecnologia | Uso |
| --- | --- |
| Python 3.12 | Linguagem da aplicação |
| FastAPI | Rotas HTTP e documentação OpenAPI |
| Pydantic | Validação dos dados de entrada e saída |
| SQLAlchemy e Psycopg | Acesso ao banco de dados |
| PostgreSQL 16 | Persistência dos pedidos |
| Docker e Docker Compose | Execução da aplicação e do banco |

As versões das dependências Python estão definidas em [requirements.txt](requirements.txt).

## Arquitetura

A aplicação separa as responsabilidades de atendimento HTTP, regras de negócio e acesso aos dados:

```text
Cliente HTTP/JSON
        |
        v
Container pedidos
FastAPI: API -> Service -> Repository
        |
        v
Container postgres
PostgreSQL + volume persistente
```

As camadas da aplicação executam no mesmo processo. O banco de dados executa em um container separado.

## Como executar

**Pré-requisitos:** Git e Docker com Docker Compose disponíveis.

```bash
git clone https://github.com/ArthurR06/APIPedidos---Sistemas-Distribuidos.git
cd APIPedidos---Sistemas-Distribuidos
docker compose up -d --build
```

O Compose aguarda o PostgreSQL ficar disponível e a aplicação cria a tabela na inicialização.

Depois de iniciar:

- API: [http://localhost:8000](http://localhost:8000)
- Swagger: [http://localhost:8000/docs](http://localhost:8000/docs)
- Especificação OpenAPI: [http://localhost:8000/openapi.json](http://localhost:8000/openapi.json)

Para acompanhar a inicialização:

```bash
docker compose logs -f pedidos
```

Para encerrar os containers preservando os dados:

```bash
docker compose down
```

Os pedidos ficam no volume `pedidos_postgres_data` declarado no Compose. O comando `docker compose down -v` também remove esse volume e seus dados.

### Configuração

A execução local funciona com os valores de demonstração do [docker-compose.yml](docker-compose.yml). Para personalizá-los, copie [.env.example](.env.example) para `.env` e ajuste `POSTGRES_DB`, `POSTGRES_USER` e `POSTGRES_PASSWORD`.

O Compose fornece a conexão à aplicação pela variável `DATABASE_URL`. O arquivo `.env` está incluído no `.gitignore`.

## Usando a API

Abra o Swagger, selecione um endpoint e utilize **Try it out**. Para criar um pedido em `POST /pedidos`, envie:

```json
{
  "cliente": "Cliente de exemplo",
  "produto": "Teclado",
  "quantidade": 2,
  "valor_unitario": 150.00
}
```

A resposta retorna **201 Created**, valor total de `300.00`, status inicial `CRIADO`, identificador e data de criação. Os campos `valor_total`, `status` e `data_criacao` são definidos pela aplicação.

| Método | Rota | Finalidade |
| --- | --- | --- |
| GET | `/health` | Verificar se a aplicação responde |
| POST | `/pedidos` | Criar um pedido |
| GET | `/pedidos` | Listar pedidos |
| GET | `/pedidos/{pedido_id}` | Consultar um pedido |
| PATCH | `/pedidos/{pedido_id}/status` | Alterar o status |

Exemplo do corpo para alterar o status:

```json
{
  "status": "CONFIRMADO"
}
```

Um identificador inexistente na consulta ou alteração retorna **404 Not Found**. Dados que não passam na validação retornam **422 Unprocessable Entity**.

## Verificação manual

O repositório não inclui uma suíte de testes automatizados. Pelo Swagger, é possível verificar:

1. Criar um pedido e conferir o total calculado.
2. Listar e consultar o pedido usando o identificador retornado.
3. Alterar o status e consultar novamente.
4. Enviar quantidade zero e verificar a resposta de validação.
5. Consultar um identificador inexistente e verificar o retorno 404.
6. Reiniciar apenas a aplicação com `docker compose restart pedidos` e consultar o pedido para conferir a persistência.

## Estrutura

```text
app/
├── main.py                     # Inicialização e rotas gerais
├── database.py                 # Conexão e criação das tabelas
├── api/pedidos.py              # Endpoints HTTP
├── services/pedido_service.py  # Regras de negócio
├── repositories/pedido_repository.py
├── models/pedido.py            # Modelo persistido
└── schemas/pedido.py           # Contratos e validação
Dockerfile
docker-compose.yml
requirements.txt
.env.example
```

## Autoria

Projeto acadêmico desenvolvido por:

- **Arthur Gomes Rodrigues de Lima** — [GitHub](https://github.com/ArthurR06) · [LinkedIn](https://www.linkedin.com/in/arthur-g-r-lima-635155225/)
- **Joice Oliveira Jardim** — [GitHub](https://github.com/Joice-O)

Repositório da integrante: [Joice-O/APIPedidos---Sistemas-Distribuidos](https://github.com/Joice-O/APIPedidos---Sistemas-Distribuidos).

## Versão acadêmica entregue

A versão final do Trabalho 1 está preservada na tag [`APIPedidos-1-final`](https://github.com/ArthurR06/APIPedidos---Sistemas-Distribuidos/tree/APIPedidos-1-final). Para consultar essa versão depois de clonar, use `git checkout APIPedidos-1-final`.
