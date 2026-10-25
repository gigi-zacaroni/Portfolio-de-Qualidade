# BUG-CHK-002 — Check-in é criado já aprovado, sem passar pelo status PENDENTE

| Campo | Informação |
|:--|:--|
| **ID** | BUG-CHK-002 |
| **Área Core** | 2 — Check-in por Foto (com impacto na Área 3 — Sistema de Pontos) |
| **Requisito** | RF-05 (Fluxo de status) |
| **Caso de teste relacionado** | CT-CHK-003 (Funcional – UI) |
| **Severidade** | Maior (erro de regra de negócio, sem travar o sistema) |
| **Prioridade** | Alta |

## Descrição

Ao realizar um check-in como usuário comum, o registro é criado diretamente com status **APROVADO**. O check-in nunca aparece como **PENDENTE**, então a etapa de moderação pelo ADMIN não tem efeito no fluxo de aprovação.

## Pré-condições

- Aplicação rodando via Docker, banco migrado e seed executado (temporada ativa existente).
- Upload de imagem funcionando (credenciais do Cloudinary configuradas).
- Usuário comum e usuário ADMIN disponíveis (ex.: `lucas@example.com` e `admin@nasalinha.com`).

## Passos para reproduzir

1. Fazer login como usuário comum.
2. Realizar um check-in com uma imagem válida.
3. Verificar o status exibido para o check-in recém-criado (tela de atividades ou perfil).
4. Fazer login como ADMIN e procurar o check-in na área de moderação.
5. Observar o status antes de qualquer ação do ADMIN.

## Resultado esperado

- O check-in recém-criado aparece com status **PENDENTE**.
- Somente após a ação do ADMIN (aprovar ou rejeitar) o status muda para **APROVADO** ou **REJEITADO** na interface.

## Resultado obtido

O check-in aparece como **APROVADO** logo após o envio, sem nunca ter sido PENDENTE e sem nenhuma ação do ADMIN.

## Análise técnica (código-fonte)

Inspeção do backend, a confirmar com a execução:

- `src/controllers/checkin.controller.js` (`create`): o check-in é gravado com `status: 'APPROVED'`, com o comentário "Auto-aprovado", em vez de usar o padrão `PENDING` definido no schema.
- `prisma/schema.prisma`: o campo `status` do check-in tem padrão `PENDING`, ou seja, o modelo prevê o fluxo PENDENTE → APROVADO/REJEITADO, mas o controller o ignora.
- No mesmo `create`, os pontos da temporada são somados na hora, antes de qualquer aprovação.

## Impacto

- Não existe moderação efetiva: toda foto enviada é aceita automaticamente, inclusive imagens inválidas ou fraudulentas.
- A pontuação é concedida sem validação do ADMIN, o que afeta o Sistema de Pontos (RF-07) e o ranking.
- As ações de aprovar e rejeitar perdem o sentido, e fica inviável cumprir o CT-CHK-003 e o CT-PTS-001 (soma de pontos após aprovação).

## Evidências

- [Check-in](Docs\evidencias\Tela_inicial.png) Print do check-in recém-criado já com status APROVADO
