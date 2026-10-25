## BUG-JWT-001 — Cadastro não envia código de verificação e leva direto ao login

| Campo | Informação |
|:--|:--|
| **ID** | BUG-JWT-001 |
| **Área Core** | 1 — Autenticação JWT |
| **Requisito** | RF-01 (Cadastro e verificação) |
| **Caso de teste relacionado** | CT-JWT-001 (Funcional – UI) |
| **Severidade** | Maior (erro de regra de negócio, sem travar o sistema) |
| **Prioridade** | Alta |

## Descrição

Ao realizar um cadastro válido pela interface, o sistema não envia o e-mail com o código de verificação esperado. Ao concluir o formulário, a aplicação segue direto para o fluxo de login/acesso, sem nenhuma etapa de confirmação de e-mail.

## Pré-condições

- Aplicação rodando via Docker, com o banco migrado.
- Variáveis `EMAIL_*` do `.env` configuradas com as credenciais SMTP da inbox do Mailtrap.
- E-mail a ser cadastrado ainda inexistente no sistema.

## Passos para reproduzir

1. Acessar a tela de cadastro na interface web.
2. Preencher nome, e-mail válido e senha válida.
3. Clicar em "Cadastrar".
4. Observar a tela exibida após o envio do formulário.
5. Abrir a caixa de entrada do Mailtrap e procurar o e-mail de verificação.

## Resultado esperado

- Mensagem de sucesso na UI indicando que a verificação foi enviada.
- Envio, via Mailtrap, de um e-mail com **código de verificação de 6 dígitos**.
- Conta criada no banco com status **PENDENTE**, sem acesso liberado até a verificação.

## Resultado obtido

- Nenhum e-mail com código de verificação é enviado; não existe etapa de confirmação.
- Após o cadastro, o usuário é levado diretamente da tela de cadastro para a tela de login, sem passar por verificação.
- A conta não fica pendente: ela é criada já ativa.

## Análise técnica (código-fonte)

Inspeção do backend e do schema Prisma

- `src/controllers/auth.controller.js` (`register`): cria o usuário e chama apenas `sendWelcomeEmail`, um e-mail de **boas-vindas**, não de verificação. A chamada não é aguardada; falhas de envio só vão para o log e o cadastro continua com sucesso.
- O `register` responde `201` já com `accessToken` e `refreshToken`, o que dispensa qualquer confirmação.
- `prisma/schema.prisma`: o modelo `User` só tem `isActive` (padrão `true`). Não há status PENDENTE, nem campo de código de verificação ou expiração.
- Não existe endpoint de verificação de e-mail nas rotas de autenticação.

## Impacto

- Qualquer pessoa pode criar uma conta com e-mail que não controla, sem comprovar a posse.
- Contas falsas ou com e-mail de terceiros entram no sistema e no ranking.
- Os requisitos RF-02 (bloqueio de login com conta pendente) e CT-JWT-003/004 ficam sem como serem exercitados.

## Evidências

- [login](../../evidencias/login.png) Print da tela de cadastro e do redirecionamento para o login
- [Caixa-de-entrada](../../evidencias/caixa_de_entrada.png) Print da inbox do Mailtrap sem e-mail de verificação 
