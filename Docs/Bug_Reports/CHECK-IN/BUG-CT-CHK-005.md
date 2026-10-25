# BUG-CHK-005 — Usuário comum consegue visualizar check-ins de outros usuários

| Campo | Informação |
|:--|:--|
| **ID** | BUG-CHK-005 |
| **Área Core** | 2 — Check-in por Foto |
| **Requisito** | RF-06 (Propriedade: isolamento dos check-ins) |
| **Caso de teste relacionado** | CT-CHK-005 (Funcional – UI) |
| **Severidade** | Maior (falha de controle de acesso e privacidade, sem travar o sistema) |
| **Prioridade** | Alta |

## Descrição

Estando logado como usuário comum (Usuário A), é possível visualizar os check-ins e as fotos de outro usuário (Usuário B). O sistema não isola os check-ins por dono.

## Pré-condições

- Aplicação rodando via Docker, banco migrado e seed executado.
- Pelo menos um check-in registrado pelo Usuário B (ex.: `maria@example.com`).
- Usuário A autenticado como usuário comum (ex.: `lucas@example.com`).

## Passos para reproduzir

1. Fazer login com o Usuário A.
2. Acessar o feed/histórico de check-ins.
3. Localizar um check-in do Usuário B e observar as informações e a foto exibidas.

## Resultado esperado

O Usuário A não vê edição, exclusão nem visualização dos check-ins e mídias do Usuário B. Cada usuário enxerga apenas os próprios check-ins.

## Resultado obtido

 O Usuário A consegue visualizar o check-in do Usuário B.

## Análise técnica (código-fonte)

Inspeção do backend, a confirmar com a execução:

- `src/routes/checkin.routes.js`: `GET /api/checkins` usa apenas autenticação, sem restrição de perfil.
- `src/controllers/checkin.controller.js` (`getAll`): retorna **todos** os check-ins do sistema, incluindo `name` e `email` do dono de cada um.
- `getById` (`GET /api/checkins/:id`): busca o check-in só pelo `id`, sem comparar `userId` com o usuário do token. Qualquer usuário autenticado acessa o check-in de qualquer pessoa, bastando conhecer o `id`.
- O backend já tem a rota `GET /api/checkins/my-checkins`, que serve ao caso de listar só os próprios check-ins.

## Impacto

- Quebra de privacidade: fotos e dados dos check-ins ficam expostos a qualquer usuário logado.
- Exposição de dados pessoais: o nome e o e-mail de outros usuários são retornados pela API.
- Descumpre o RF-06, e o mesmo problema vale para a API, independentemente do que a interface mostre.

## Evidências

- [check-in_historico](Docs\evidencias\check-in_historico.png) Print do histórico do Usuário A exibindo o check-in do Usuário B

