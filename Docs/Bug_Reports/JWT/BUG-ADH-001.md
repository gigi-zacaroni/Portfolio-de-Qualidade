# BUG-ADH-001 - Usuário com perfil TRAINEE acessa endpoint de ranking sem autorização

| Campo | Informação |
|:--|:--|
| **ID** | BUG-ADH-001 |
| **Área Core** | 1 — Autenticação JWT |
| **Requisito** | RF-03 — Controle de níveis de acesso (RBAC) |
| **Caso de teste relacionado** | CT-ADH-001 (Ad-hoc – API, negativo) |
| **Localização do erro** | Backend / API |
| **Severidade** | Maior (falha de controle de acesso, sem travar o sistema) |
| **Prioridade** | Alta |

## Descrição

Um usuário com perfil TRAINEE consegue consultar o ranking diretamente pela API. A interface já bloqueia o acesso desse perfil à tela de ranking, mas o backend não aplica a mesma restrição, então a regra pode ser contornada ao chamar o endpoint sem passar pela UI.

## Pré-condições

- Aplicação rodando via Docker, com o banco de dados migrado e populado (seed).
- Conta de perfil TRAINEE disponível (seed: `pedro@example.com`, senha `senha123`).
- Existência de uma temporada ativa.
- Ferramenta de requisições HTTP (Insomnia).

## Passos para reproduzir

1. Autenticar com a conta de perfil TRAINEE via `POST /api/auth/login` e obter o token JWT.
2. Enviar `GET /api/rankings` com o header `Authorization: Bearer <token>`.
3. Observar o status e o corpo da resposta.

## Resultado esperado

- A API deve identificar que o perfil TRAINEE não tem permissão para o recurso.
- A requisição deve ser bloqueada com HTTP `403 Forbidden` e a mensagem `Sem permissão para acessar este recurso`.
- Nenhum dado do ranking deve ser retornado.

## Resultado obtido

- A API retornou HTTP `200 OK`.
- O corpo da resposta trouxe o ranking completo da temporada ativa, com nome, e-mail e perfil de cada usuário.

## Análise técnica (código-fonte)

- **`src/routes/ranking.routes.js`**: a rota usa apenas `router.use(authenticate)`. O middleware `authorize` não é aplicado, então qualquer usuário autenticado acessa `GET /api/rankings` e `GET /api/rankings/season/:seasonId`.
- **`frontend/src/components/common/RankingRoute.js`**: redireciona usuários TRAINEE para `/dashboard`. A restrição existe só no frontend.
- **`src/middlewares/auth.js`**: o middleware `authorize(...roles)` já existe e retorna 403 para perfis não permitidos, mas não é usado nas rotas de ranking.
- **`src/controllers/ranking.controller.js`**: o `select` de usuário inclui `email` e `role`, e esses dados são devolvidos a quem chamar o endpoint.

## Impacto

- A restrição de acesso do perfil TRAINEE pode ser contornada chamando a API diretamente.
- Exposição de e-mail e perfil de todos os usuários da temporada a quem não deveria ver essa informação.
- Inconsistência entre as regras de frontend e backend, que tende a se repetir em outros endpoints.

## Evidências

- ![Ranking](Docs/evidencias/ranking-trainee.png) Print da requisição `GET /api/rankings` com token TRAINEE retornando 200 OK