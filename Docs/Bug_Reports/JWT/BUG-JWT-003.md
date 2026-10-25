# BUG-JWT-002 — Login é permitido para conta que não verificou o e-mail

| Campo | Informação |
|:--|:--|
| **ID** | BUG-JWT-002 |
| **Área Core** | 1 — Autenticação JWT |
| **Requisito** | RF-02 (Login e bloqueio de conta pendente) |
| **Caso de teste relacionado** | CT-JWT-003 (Funcional – UI, negativo) |
| **Severidade** | Maior (erro de regra de negócio, sem travar o sistema) |
| **Prioridade** | Alta |


Um usuário que acabou de se cadastrar e não realizou nenhuma verificação de e-mail consegue fazer login normalmente. O sistema não bloqueia o acesso nem exibe a mensagem exigida para contas não verificadas.

## Pré-condições

- Aplicação rodando via Docker, com o banco migrado.
- Conta criada pelo fluxo de cadastro (RF-01), sem nenhuma etapa de verificação realizada.

## Passos para reproduzir

1. Acessar a página de login.
2. Informar o e-mail e a senha da conta recém-cadastrada e não verificada.
3. Clicar em "Entrar".

## Resultado esperado

O login é bloqueado, exibindo exatamente a mensagem: **"Por favor, verifique sua conta para continuar"**.

## Resultado obtido

O login é concluído com sucesso e o usuário acessa o sistema, sem nenhuma mensagem de verificação pendente.

## Análise técnica (código-fonte)

Inspeção do backend e do schema Prisma

- `src/controllers/auth.controller.js` (`login`): depois de validar e-mail e senha, a única checagem de estado da conta é `isActive`, que responde `401` com "Usuário inativo". Não existe checagem de verificação.
- `prisma/schema.prisma`: o modelo `User` não tem status de verificação (como PENDENTE) nem campo de código; `isActive` começa como `true` no cadastro.
- Como consequência, não há conta "pendente" no sistema, então a regra de bloqueio do RF-02 não é aplicável.

## Impacto

- A regra de verificação de conta definida no RF-02 não é cumprida.
- Em conjunto com o BUG-JWT-001, qualquer pessoa cria conta com e-mail de terceiros e já acessa o sistema.
- Os casos CT-JWT-004 (login após verificação) e o fluxo da suíte de regressão CT-JWT-REG-001 ficam sem base para validar a verificação.

## Evidências

- [login](../../evidencias/login.png) Print da tela de login com as credenciais da conta não verificada

