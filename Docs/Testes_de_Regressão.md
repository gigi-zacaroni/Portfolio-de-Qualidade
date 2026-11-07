# Testes de Regressão — NaSalinha

**Projeto:** NaSalinha — Auditoria de Qualidade (Desafio QA 2026.2)
**Documento:** Planejamento dos testes de regressão das 3 áreas Core
**Casos cobertos:** CT-JWT-REG-001, CT-CHK-REG-001, CT-PTS-REG-001

---

## 1. Área Core 1 — Autenticação JWT

### CT-JWT-REG-001 — Regressão de Autenticação e Acesso

**Fluxo principal:** cadastro → verificação por e-mail → login → acesso autenticado a uma funcionalidade protegida.

**Bugs existentes na área:**

| Bug | Requisito | Situação nesta rodada |
|:--|:--|:--|
| BUG-JWT-001 — cadastro não envia código de verificação | RF-01 | Corrigido junto com o JWT-003 (simulação) |
| BUG-JWT-003 — login permitido para conta não verificada | RF-02 | **Corrigido (simulação)** |
| BUG-ADH-001 — TRAINEE acessa `GET /api/rankings` | RF-03 | Aberto |

**Correção simulada (BUG-JWT-003, que depende do BUG-JWT-001):**

Hoje o `User` não tem estado de verificação, então não existe conta "pendente". A correção adiciona esse estado e bloqueia o login enquanto ele não for confirmado.

- `prisma/schema.prisma`: novos campos `isVerified Boolean @default(false)`, `verificationCode String?` e `verificationExpiry DateTime?`.
- `auth.controller.js` (`register`): gera um código de 6 dígitos, grava com validade, envia por e-mail e responde `201` **sem** `accessToken`/`refreshToken`.
- Nova rota `POST /api/auth/verify` (`{ email, code }`): confere o código e a validade e marca `isVerified = true`.
- `auth.controller.js` (`login`), depois de validar a senha:

```js
if (!user.isVerified) {
  return res.status(403).json({
    success: false,
    message: 'Por favor, verifique sua conta para continuar',
  });
}
```

**Efeitos colaterais prováveis (o que pode quebrar):**

- Contas do **seed** e usuários já existentes passam a nascer com `isVerified = false` e ficam sem conseguir logar, se o seed e a migration não forem ajustados.
- O frontend (`Register.js`) esperava receber o token no cadastro; sem ele, o fluxo pode ficar travado.
- O token de refresh (`/auth/refresh`) pode continuar emitindo tokens para conta não verificada.
- Todo teste das outras áreas depende de login, então um erro aqui bloqueia a suíte inteira.

**Testes a repetir:**

| # | Teste (caso de origem) | O que garante | Resultado esperado |
|:--:|:--|:--|:--|
| 1 | Cadastro válido via UI com e-mail novo (CT-JWT-001) | A correção do RF-01 funciona e o e-mail chega | `201`, **sem** token na resposta; e-mail com código no Mailtrap; conta `isVerified = false` |
| 2 | Cadastro com e-mail duplicado via API (CT-JWT-002) | A alteração no `register` não afetou a unicidade | `409` "Este registro já existe"; nenhum usuário novo |
| 3 | Login com conta não verificada (CT-JWT-003) | Alvo da correção | `403` "Por favor, verifique sua conta para continuar"; sem token |
| 4 | Verificar com código inválido e com código expirado | A rota nova recusa código errado | Erro `400`; conta continua não verificada |
| 5 | Verificar com o código correto e fazer login (CT-JWT-004) | O ciclo completo funciona e o JWT é emitido | `200` com `accessToken` e `refreshToken` válidos |
| 6 | Login com as 4 contas do seed | O seed e a migration não bloquearam contas legítimas | `200` para ADMIN, MEMBER e TRAINEE |
| 7 | Hashing de senha no banco e omissão no payload (CT-JWT-007) | O novo `select` não expôs `password` nem o código de verificação | Senha em hash no banco; resposta sem `password` e sem `verificationCode` |
| 8 | RBAC: usuário comum x ADMIN em Temporadas e Usuários (CT-JWT-005 e CT-JWT-006) | A mudança no login não afetou as permissões | Comum recebe `403`; sem token recebe `401`; ADMIN acessa |
| 9 | `GET /api/rankings` com token TRAINEE (BUG-ADH-001) | Mesma camada de autorização | **Bug conhecido aberto:** continua `200` |
| 10 | Usar o token obtido para realizar um check-in | A sessão continua válida no fluxo dependente | `201` com check-in criado |

**Critério de aprovação:** os passos 1 a 8 e 10 passam e o passo 9 reproduz o bug conhecido. Qualquer falha em 6 ou 10 é bloqueante, porque indica que o login quebrou para todas as áreas.

---

## 2. Área Core 2 — Check-in por Foto

### CT-CHK-REG-001 — Regressão do Fluxo de Mídia e Duplicidade

**Fluxo principal:** autenticar → enviar foto → check-in criado na temporada ativa → pontos e ranking atualizados.

**Bugs existentes na área:**

| Bug | Requisito | Situação nesta rodada |
|:--|:--|:--|
| BUG-CHK-001 — mais de um check-in no mesmo dia | RF-04 / RF-09 | **Corrigido (simulação)** |
| BUG-CHK-003 — check-in nasce `APPROVED` | RF-05 | Aberto |
| BUG-CHK-005 — usuário comum vê check-ins de outros | RF-06 | Aberto |

O BUG-CHK-001 tem a mesma causa raiz do BUG-PTS-006. A correção resolve os dois.

**Correção simulada (BUG-CHK-001):**

Em `checkin.controller.js` (`create`), depois de validar a foto e **antes** do upload ao Cloudinary, o método consulta se o usuário já fez check-in nas últimas 24 horas:

```js
const since = new Date(Date.now() - 24 * 60 * 60 * 1000);
const recent = await prisma.checkIn.findFirst({
  where: { userId, createdAt: { gte: since } },
});

if (recent) {
  await fs.unlink(req.file.path).catch(console.error); // o multer já gravou o arquivo
  return res.status(400).json({
    success: false,
    message: 'Você já realizou um check-in nas últimas 24 horas',
  });
}
```

**Efeitos colaterais prováveis:**

- O multer grava o arquivo em `/app/uploads` **antes** do controller rodar. O novo `return` cria um caminho de saída em que o arquivo temporário pode ficar no servidor (risco ao RNF-02 / CT-CHK-007).
- Se o filtro esquecer o `userId`, **um** check-in bloqueia **todos** os usuários.
- A janela de 24 h móvel difere de "uma vez por dia" (RF-04). A borda das 24 h precisa ser testada.
- Pontos e contador não devem ser incrementados no caminho bloqueado.

**Testes a repetir:**

| # | Teste (caso de origem) | O que garante | Resultado esperado |
|:--:|:--|:--|:--|
| 1 | Check-in válido com foto (CT-CHK-001) | O fluxo principal continua funcionando | `201`; check-in associado à temporada ativa; pontos +10 |
| 2 | Segundo check-in do mesmo usuário em menos de 24 h (CT-CHK-001 / CT-PTS-006) | Alvo da correção | `400`; nenhum check-in novo; saldo e contador inalterados |
| 3 | Check-in de **outro** usuário no mesmo dia | O filtro é por usuário | `201` para o segundo usuário |
| 4 | Borda das 24 h: recuar o `createdAt` do último check-in para 25 h atrás e enviar de novo | A janela libera depois do prazo | `201`. Para recuar: `UPDATE checkins SET "createdAt" = now() - interval '25 hours' WHERE id = '<id>';` |
| 5 | `ls /app/uploads` no container após o `400` do passo 2 (CT-CHK-007) | O novo caminho de saída não deixou lixo | Pasta igual à anterior ao envio |
| 6 | Envio sem foto e com arquivo que não é imagem (CT-CHK-002) | A ordem das validações continua correta | `400` "Foto é obrigatória" / erro de tipo de arquivo |
| 7 | Envio da mesma foto duas vezes (RF-04 do README) | Bloqueio de foto repetida | **Possível bug novo:** o código não tem verificação de foto repetida. Se aceitar, abrir bug (não é regressão) |
| 8 | Alteração de status por ADMIN e por usuário comum (CT-CHK-003 e CT-CHK-004) | A rota de moderação não foi afetada | ADMIN: `200`. Comum: `403`. O check-in continua nascendo `APPROVED` (**BUG-CHK-003 aberto**) |
| 9 | Usuário comum lista check-ins (CT-CHK-005) e tenta excluir check-in alheio (CT-CHK-006) | O isolamento não mudou | Listagem ainda expõe check-ins de outros (**BUG-CHK-005 aberto**); exclusão retorna `403` |
| 10 | Conferir pontuação e ranking após o check-in do passo 1 (CT-PTS-001) | O check-in bloqueado não vazou pontos | Saldo +10 uma única vez; ranking com a posição correta |

**Critério de aprovação:** os passos 1 a 6 e 10 passam, o passo 7 é avaliado à parte e os passos 8 e 9 reproduzem apenas os bugs já conhecidos. Falha no passo 3 ou 5 é regressão causada pela correção.

---

## 3. Área Core 3 — Sistema de Pontos

### CT-PTS-REG-001 — Regressão de Cálculo de Saldo e Exibição de Ranking

**Fluxo principal:** check-in aprovado soma pontos → exclusão do check-in subtrai pontos → ranking reordenado.

**Bugs existentes na área:**

| Bug | Requisito | Situação nesta rodada |
|:--|:--|:--|
| BUG-PTS-003 — pontos não são subtraídos ao excluir check-in | RF-08 | **Corrigido (simulação)** |
| BUG-PTS-006 — segundo check-in em menos de 24 h | RF-09 | Aberto (mesma causa do BUG-CHK-001) |

**Correção simulada (BUG-PTS-003):**

Em `checkin.controller.js` (`delete`), a exclusão e o ajuste do saldo passam a ocorrer na mesma transação, e só para check-in aprovado:

```js
const operations = [prisma.checkIn.delete({ where: { id } })];

if (checkIn.status === 'APPROVED') {
  operations.push(
    prisma.point.update({
      where: { userId_seasonId: { userId: checkIn.userId, seasonId: checkIn.seasonId } },
      data: {
        totalPoints: { decrement: checkIn.points },
        checkInsCount: { decrement: 1 },
      },
    })
  );
}

await prisma.$transaction(operations);
```

**Efeitos colaterais prováveis:**

- Segunda exclusão do mesmo check-in debitar de novo (idempotência, CT-PTS-004).
- Excluir check-in `PENDING` ou `REJECTED` debitar pontos que nunca foram creditados.
- Saldo ficar **negativo** ou divergente se o ADMIN alterou o campo `points` do check-in via `PUT` antes da exclusão (o `PUT` não ajusta o saldo).
- Registro `Point` inexistente (dado legado): o `update` falha com `P2025`, o `errorHandler` responde `404` e, por causa da transação, o check-in também não é excluído.
- Ranking continuar mostrando posição antiga após a exclusão.

**Testes a repetir:**

| # | Teste (caso de origem) | O que garante | Resultado esperado |
|:--:|:--|:--|:--|
| 1 | Check-in aprovado soma pontos e atualiza o ranking (CT-PTS-001 e CT-PTS-002) | A criação continua somando corretamente | Saldo +10 e `checkInsCount` +1; ranking atualizado |
| 2 | Excluir o check-in aprovado e consultar saldo (CT-PTS-003) | Alvo da correção | Saldo -10 e `checkInsCount` -1; volta ao valor anterior ao check-in |
| 3 | Repetir o `DELETE` do mesmo check-in (CT-PTS-004) | Idempotência | Segunda chamada: `404` "Check-in não encontrado"; saldo **não** muda de novo |
| 4 | Excluir check-in com status diferente de `APPROVED` (mudar para `REJECTED` via ADMIN e excluir) | Não debita pontos que não foram creditados | Saldo não sofre débito indevido |
| 5 | Alterar `points` do check-in via `PUT` e depois excluir | A exclusão debita o valor coerente com o creditado | Saldo não fica negativo. **Teste exploratório:** divergência indica bug novo |
| 6 | Excluir check-in de usuário sem registro de pontos | A transação não deixa a exclusão pela metade | Comportamento consistente: ou exclui tudo ou nada; resposta sem erro `500` |
| 7 | Consultar o ranking depois das exclusões (CT-PTS-005) | A ordem decrescente reflete o saldo corrigido | Ordem por `totalPoints` decrescente; usuário que perdeu pontos mudou de posição |
| 8 | Novo check-in em menos de 24 h (CT-PTS-006) | A trava temporal não foi afetada | Com a correção do BUG-CHK-001 na build: `400`. Sem ela: **bug conhecido aberto** (`201`) |
| 9 | Perfil/saldo do usuário no endpoint de pontos (CT-PTS-002) | O saldo retornado bate com o do banco | Mesmo valor que `Point.totalPoints` |

**Critério de aprovação:** os passos 1 a 4, 7 e 9 passam. O passo 8 segue a situação da build, e os passos 5 e 6 são exploratórios. Saldo negativo ou débito duplicado (passos 3 e 4) é regressão com severidade Maior.

---

## Ordem de execução recomendada

1. **Autenticação (CT-JWT-REG-001) primeiro.** As outras duas suítes dependem de login e token.
2. **Check-in (CT-CHK-REG-001) em seguida.** Ele gera os dados (check-ins e pontos) que a suíte de Pontos consome.
3. **Pontos (CT-PTS-REG-001) por último.**

## Resumo

| Suíte | Bug corrigido (simulação) | Passos | Bugs conhecidos que devem continuar abertos |
|:--|:--|:--:|:--|
| CT-JWT-REG-001 | BUG-JWT-003 (e JWT-001) | 10 | BUG-ADH-001 |
| CT-CHK-REG-001 | BUG-CHK-001 | 10 | BUG-CHK-003, BUG-CHK-005 |
| CT-PTS-REG-001 | BUG-PTS-003 | 9 | BUG-PTS-006 (se a correção do CHK-001 não estiver na build) |

