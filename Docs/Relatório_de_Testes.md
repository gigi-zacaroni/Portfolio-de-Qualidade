# Resumo dos Casos de Teste — NaSalinha

**Projeto:** NaSalinha — Auditoria de Qualidade (Desafio QA 2026.2)

## Resultado por caso de teste

| ID | Tipo | Descrição | Resultado | Severidade |
|:--|:--|:--|:--:|:--:|
| CT-JWT-001 | Positivo | Cadastro válido via UI e envio do código por e-mail | Fail | Maior (BUG-JWT-001) |
| CT-JWT-002 | Negativo | Cadastro com e-mail duplicado via API | Pass | — |
| CT-JWT-003 | Negativo | Bloqueio de login de conta não verificada via UI | Fail | Maior (BUG-JWT-003) |
| CT-JWT-004 | Positivo | Login e geração de JWT após verificação | Pass | — |
| CT-JWT-005 | Negativo | Restrição de menu e rotas ADMIN para usuário comum | Pass | — |
| CT-JWT-006 | Positivo | Acesso a endpoint restrito com perfil ADMIN | Pass | — |
| CT-JWT-007 | Negativo | Hash de senha no banco e omissão no payload | Pass | — |
| CT-JWT-REG-001 | Regressão | Suíte de regressão de autenticação | Executado (simulação) | — |
| CT-CHK-001 | Positivo | Check-in com foto válida (limite de um por dia) | Fail | Maior (BUG-CHK-001) |
| CT-CHK-002 | Negativo | Envio sem arquivo de imagem válido | Pass | — |
| CT-CHK-003 | Positivo | Status PENDENTE → APROVADO pelo ADMIN | Fail | Maior (BUG-CHK-003) |
| CT-CHK-004 | Negativo | Alteração de status por usuário comum | Pass | — |
| CT-CHK-005 | Negativo | Visualização ou edição de check-in alheio via UI | Fail | Maior (BUG-CHK-005) |
| CT-CHK-006 | Negativo | Exclusão de check-in de outro usuário via API | Pass | — |
| CT-CHK-007 | Positivo | Remoção do arquivo temporário do servidor (sucesso e falha) | Pass | — |
| CT-CHK-REG-001 | Regressão | Suíte de regressão de check-in e mídia | Executado (simulação) | — |
| CT-PTS-001 | Positivo | Soma de pontos e atualização do ranking | Pass | — |
| CT-PTS-002 | Positivo | Saldo retornado no endpoint de perfil | Pass | — |
| CT-PTS-003 | Positivo | Subtração de pontos ao excluir check-in aprovado | Fail | Maior (BUG-PTS-003) |
| CT-PTS-004 | Negativo | Idempotência em exclusões seguidas | Pass | — |
| CT-PTS-005 | Positivo | Ranking em ordem decrescente | Pass | — |
| CT-PTS-006 | Negativo | Bloqueio de segundo check-in em menos de 24h | Fail | Maior (BUG-PTS-006) |
| CT-PTS-REG-001 | Regressão | Suíte de regressão de pontos e ranking | Executado (simulação) | — |
| CT-ADH-001 | Negativo (ad-hoc) | TRAINEE acessa `GET /api/rankings` via API | Fail | Maior (BUG-ADH-001) |

## Totais

| Indicador | Quantidade |
|:--|:--:|
| Casos planejados | 23 (20 funcionais + 3 regressão) |
| Casos executados | 24 (23 planejados + 1 ad-hoc) |
| **Passaram (Pass)** | **13** |
| **Falharam (Fail)** | **8** (7 planejados + 1 ad-hoc) |
| Regressão executada (sem Pass/Fail) | 3 |

Os 8 bugs são todos de severidade **Maior**. Não há bugs Críticos nem Menores.

As três suítes de regressão ficaram fora da contagem de Pass/Fail porque rodam sobre uma correção simulada e não têm resultado registrado. Por isso 13 + 8 dá 21, e os outros 3 casos executados são as regressões.