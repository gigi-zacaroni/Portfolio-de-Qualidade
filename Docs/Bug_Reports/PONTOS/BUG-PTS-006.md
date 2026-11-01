## Resultado esperado

- A API deve identificar que o usuário já possui um check-in realizado nas últimas 24 horas.
- A nova requisição deve ser bloqueada, retornando um erro HTTP `400 Bad Request`.
- Nenhum novo registro de check-in deve ser criado.
- A pontuação total e a quantidade de check-ins do usuário não devem ser incrementadas.

## Resultado obtido

- A API permitiu a realização de um segundo check-in no mesmo dia.
- A requisição retornou sucesso, com status HTTP `201 Created`.
- Um novo registro foi criado com status `APPROVED`.
- O usuário recebeu mais 10 pontos pelo novo check-in.
- A quantidade de check-ins e a pontuação puderam ser incrementadas indevidamente.

**Evidência da resposta da API:**

- ID do novo check-in: `b63a07b3-f28f-48f1-b7f6-dbc28fe9d858`
- Pontos concedidos: `10`
- Status: `APPROVED`
- Data de criação: `2026-11-01T16:44:25.489Z`

## Análise técnica (código-fonte)

Inspeção de `src/controllers/checkin.controller.js`:

- **Método `create`**: após verificar a existência da foto, realizar o upload e localizar a temporada ativa, o método executa diretamente `prisma.checkIn.create()`.
- Não existe consulta aos check-ins anteriores do usuário para verificar se algum foi realizado nas últimas 24 horas.
- Após criar o registro, o método executa `prisma.point.upsert()`, incrementando `totalPoints` e `checkInsCount`.
- Não foi identificada nenhuma validação temporal que impeça a criação de um segundo check-in dentro do intervalo estabelecido.

## Impacto

- Usuários podem realizar check-ins repetidos em um intervalo inferior a 24 horas.
- O sistema permite o acúmulo indevido de pontos.
- A quantidade de check-ins registrada pode não representar a utilização correta da funcionalidade.
- O ranking pode apresentar pontuações inconsistentes, comprometendo a competitividade e a confiabilidade do sistema.

## Evidências

- [Histórico de check-ins](Portfolio-de-Qualidade/Docs/evidencias/CT-PTS-006.png) Print da consulta `GET /api/checkins/my-checkins`, demonstrando a existência de dois check-ins feitos no mesmo dia