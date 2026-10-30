# Contratos da API REST — EcoByteMetrics

Especificação das rotas registradas em `backend/src/app.ts` e nos roteadores dos módulos, revisada em 30/10/2026 sobre as branches #33–#36. Os exemplos refletem os nomes usados pelo backend; não use o prefixo `/api` nas rotas de saúde e monitoramento.

## Base e formatos

- Backend local: `http://localhost:3000` (porta definida por `PORT`).
- Corpo e respostas JSON: `Content-Type: application/json`.
- As rotas CRUD listadas em “Módulos” usam o prefixo `/api`.
- A autenticação é registrada em `/auth`, sem prefixo `/api`.

## Saúde e monitoramento

### `GET /health`

Verifica se o processo Express responde.

Resposta `200`:

```json
{ "status": "ok" }
```

### `GET /services`

Retorna a lista descoberta pelo backend e o horário de sucesso da última consulta.

```json
{
  "services": [
    {
      "id": "service-id",
      "name": "Serviço",
      "location": {
        "country": "Brasil",
        "region": "São Paulo",
        "city": "São Paulo",
        "latitude": -23.55,
        "longitude": -46.63,
        "region_code": "BR-SP"
      },
      "metrics_path": "/metrics/service-id"
    }
  ],
  "lastUpdatedAt": "2026-10-30T12:00:00.000Z"
}
```

O valor de `lastUpdatedAt` é `null` antes da primeira descoberta bem-sucedida. Se uma nova consulta falhar, os dados da última descoberta bem-sucedida permanecem disponíveis.

### `GET /metrics`

Retorna as amostras mantidas em memória:

```json
{
  "metrics": [
    {
      "serviceId": "service-id",
      "metrics": {
        "collection_interval_seconds": 30,
        "metrics": {
          "cpu_percent": 45.2,
          "memory_gb": 0.5,
          "disk_gb": 12.4,
          "network_gb": 1.2
        }
      },
      "updatedAt": "2026-10-30T12:00:00.000Z"
    }
  ]
}
```

### `GET /metrics/:serviceId`

Retorna a amostra mais recente associada ao identificador de serviço. `serviceId` é um identificador textual e deve ser codificado como segmento de URL.

- `200`: objeto com `serviceId`, `metrics` e `updatedAt`.
- `400`: identificador inválido.
- `404`: `metrics_missing` quando o serviço respondeu sem uma amostra válida; para identificador sem amostra conhecida, retorna mensagem de métricas não encontradas.
- `503`: `service_unavailable` quando a consulta ao serviço falha; o monitoramento dos demais continua.

## Autenticação

### `POST /auth/login`

Recebe as credenciais no corpo JSON e autentica o usuário.

```json
{ "email": "admin@example.com", "password": "senha" }
```

O controller retorna o token JWT e os dados públicos do usuário em caso de sucesso. Credenciais inválidas retornam `401`; campos inválidos retornam `400`. Não envie a senha em respostas ou logs.

## Módulos REST sob `/api`

| Caminho | Métodos | Parâmetros de caminho | Autenticação |
|---|---|---|---|
| `/api/users` | `POST` | — | — |
| `/api/users/:id_user` | `GET`, `PUT`, `DELETE` | `id_user` | — |
| `/api/metrics` | `POST` | — | — |
| `/api/metrics/:id_metric` | `GET` | `id_metric` | — |
| `/api/metrics/list/:id_service` | `GET` | `id_service` | — |
| `/api/services/all` | `GET` | — | — |
| `/api/services/:id` | `GET` | `id` | — |
| `/api/services/loc/:id` | `GET` | `id` | — |
| `/api/services` | `POST` | — | Bearer JWT |
| `/api/locations/all` | `GET` | — | — |
| `/api/locations/:id_location` | `GET` | `id_location` | — |
| `/api/locations` | `POST` | — | — |
| `/api/carbons/all` | `GET` | — | — |
| `/api/carbons/:id_carbon` | `GET` | `id_carbon` | — |
| `/api/carbons/loc/:id_location` | `GET` | `id_location` | — |
| `/api/carbons` | `POST` | — | — |
| `/api/statistics/summary` | `GET` | — | — |
| `/api/statistics/ranking` | `GET` | — | — |

Os corpos de criação e atualização seguem os tipos dos módulos e colunas do banco. Exemplos das entidades: usuário (`name_user`, `email_user`, `password_user`), serviço (`id_location`, `name_service`, `path_metric`, `active_service`), métrica (`id_service`, `cpu_metric`, `ram_metric`, `hd_metric`, `network_metric`, `date_houry`, `colletion_interval_metric`), localização e carbono conforme `database/DDL.sql`.

### Filtros da rota de ranking

`GET /api/statistics/ranking` aceita `periodo` e `ordenarPor` como parâmetros de query. O roteador documenta exemplos `?periodo=24h&ordenarPor=kwh` e `?periodo=7d&ordenarPor=co2e`.

## Erros e métodos

- As rotas de identificador podem retornar `400` para parâmetros inválidos e `404` quando o registro não existe.
- Métodos não permitidos nos roteadores de módulos retornam `405` com `{ "error": "Método não permitido" }`.
- Caminhos não registrados retornam `404` em JSON.
- Erros inesperados do Express retornam `500` com `error: internal_error` e mensagem segura ao cliente.
- A forma detalhada do erro e os dados aceitos por cada controller devem ser mantidos alinhados ao código do respectivo módulo.

## Referências

- [Arquitetura](./arquitetura.md)
- [Plano de entregas](./plano-de-entregas.md)
- [DDL PostgreSQL](../database/DDL.sql)
- [Código das rotas](../backend/src/app.ts)
