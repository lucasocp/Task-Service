# Task Service
 
API REST simples para criação e listagem de tarefas, escrita em Go. Usa SQLite como armazenamento e processa cada tarefa de forma assíncrona em uma goroutine, através de um channel.
 
## Como funciona
 
- `POST /tasks` grava a tarefa no banco com status `pending` e a envia para um channel (`TaskChannel`).
- Uma goroutine (`ProcessTasks`) consome esse channel, simula um processamento (`time.Sleep`) e atualiza o status para `completed` no banco.
- `GET /tasks` lista todas as tarefas e seus status atuais.
## Requisitos
 
- Go 1.22 ou superior (usa o novo roteamento de métodos do `net/http`, ex: `"POST /tasks"`)
- CGO habilitado (o driver `mattn/go-sqlite3` depende disso)
## Instalação
 
```bash
git clone <url-do-repositorio>
cd task-service
go mod tidy
```
 
## Executando
 
```bash
go run main.go
```
 
O servidor sobe na porta `8081` e cria automaticamente o arquivo `tasks.db` e a tabela `tasks`, caso não existam.
 
## Endpoints
 
### Criar tarefa
 
```
POST /tasks
Content-Type: application/json
```
 
Body:
```json
{
  "title": "Minha tarefa",
  "description": "Descrição da tarefa"
}
```
 
Resposta (`201 Created`):
```json
{
  "id": 1,
  "title": "Minha tarefa",
  "description": "Descrição da tarefa",
  "status": "pending",
  "created_at": "2026-08-27T10:00:00Z"
}
```
 
O status inicial é sempre `pending`. Alguns segundos depois (processamento assíncrono), ele muda para `completed`.
 
### Listar tarefas
 
```
GET /tasks
```
 
Resposta (`200 OK`):
```json
[
  {
    "id": 1,
    "title": "Minha tarefa",
    "description": "Descrição da tarefa",
    "status": "completed",
    "created_at": "2026-08-27T10:00:00Z"
  }
]
```
 
## Estrutura do banco
 
```sql
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    status TEXT NOT NULL,
    created_at DATETIME NOT NULL
);
```
 
## Limitações conhecidas / próximos passos
 
- Não há endpoint para buscar uma tarefa específica por ID.
- Não há validação de campos obrigatórios (`title` vazio, por exemplo, é aceito).
- O processamento é sequencial dentro da goroutine (uma tarefa por vez); para paralelismo real seria necessário um pool de workers.
- Sem testes automatizados ainda.
 
