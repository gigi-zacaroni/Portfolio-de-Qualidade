# BUG-PTS-006 - Bloqueio de segundo check-in em intervalo < 24h

| Campo | Informação |
|:--|:--|
| **ID** | BUG-PTS-006 |
| **Área Core** | 3 — Pontuação |
| **Requisito** | Regra de negócio — Limite de um check-in a cada 24 horas |
| **Caso de teste relacionado** | CT-PTS-006 (Funcional – API) |
| **Severidade** | Maior (erro de regra de negócio que permite acúmulo indevido de pontos) |
| **Prioridade** | Alta |

## Descrição

O sistema permite que um mesmo usuário realize mais de um check-in em um intervalo inferior a 24 horas. Ao enviar uma nova requisição para o endpoint de criação de check-ins, mesmo já existindo um registro realizado no mesmo dia, a API permite a operação, aprova automaticamente o novo check-in e concede os pontos correspondentes.

## Pré-condições

- Aplicação rodando via Docker, com o banco de dados migrado.
- Usuário cadastrado e autenticado com um token JWT válido.
- Existência de uma temporada ativa.
- Usuário já possui pelo menos um check-in realizado nas últimas 24 horas.
- Imagem válida disponível para envio na requisição.

## Passos para reproduzir

1. Autenticar um usuário e obter seu token JWT.
2. Consultar os check-ins do usuário pelo endpoint `GET /api/checkins/my-checkins`.
3. Confirmar a existência de um check-in realizado nas últimas 24 horas.
4. Enviar uma nova requisição `POST /api/checkins` utilizando o mesmo usuário.
5. Na aba **Body**, selecionar `form-data` e adicionar o campo `photo` com uma imagem válida.
6. Enviar a requisição e observar a resposta da API.



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

- ![Histórico de check-ins](https://github.com/gigi-zacaroni/Portfolio-de-Qualidade/blob/main/../../evidencias/CT-PTS-006.png) Print da consulta `GET /api/checkins/my-checkins`, demonstrando a existência de dois check-ins feitos no mesmo dia
