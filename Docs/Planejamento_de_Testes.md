# Planejamento de Testes

### Área Core 1: Autenticação JWT

#### Requisito RF-01: Cadastro e verificação[cite: 1]

* **CT-JWT-001 (Funcional - Positivo):** Cadastro de novo usuário válido via UI[cite: 1, 3]
  * **Objetivo:** Garantir que o cadastro com dados válidos envia o código de verificação para o e-mail via Mailtrap e cria a conta com status PENDENTE[cite: 1].
  * **Pré-condições:** Aplicação rodando e e-mail não cadastrado no sistema[cite: 1].
  * **Passos:**
    1. Acessar a tela de cadastro na interface web.
    2. Preencher nome, e-mail corporativo válido e senha válida.
    3. Clicar no botão "Cadastrar".
    4. Acessar a caixa de entrada do Mailtrap[cite: 1].
  * **Resultado Esperado:** Mensagem de sucesso na UI e envio do código de verificação de 6 dígitos via Mailtrap[cite: 1]. A conta é criada no banco de dados com status `PENDENTE`[cite: 1].

* **CT-JWT-002 (API - Negativo):** Tentativa de cadastro com e-mail duplicado[cite: 1, 3]
  * **Objetivo:** Validar que a API rejeita requisição de cadastro utilizando e-mail já existente[cite: 1].
  * **Endpoint / Método:** `POST /api/auth/register`
  * **Payload:**
    ```json
    {
      "name": "Usuário Teste",
      "email": "existente@nasalinha.com",
      "password": "Senha@123"
    }
    ```
  * **Resultado Esperado:** Status Code `400 Bad Request` ou `409 Conflict`[cite: 7] com mensagem de erro informando que o e-mail já está cadastrado[cite: 1].

#### Requisito RF-02: Restrição de login[cite: 1]

* **CT-JWT-003 (Funcional - Negativo):** Tentativa de login com conta não verificada via UI[cite: 1, 3]
  * **Objetivo:** Verificar se o sistema impede o acesso de contas com status `PENDENTE`[cite: 1].
  * **Pré-condições:** Conta cadastrada no RF-01 sem realizar a verificação do código[cite: 1].
  * **Passos:**
    1. Acessar a página de login.
    2. Informar as credenciais do usuário não verificado.
    3. Clicar em "Entrar".
  * **Resultado Esperado:** O login é bloqueado exibindo exatamente a mensagem: *"Por favor, verifique sua conta para continuar"*[cite: 1].

* **CT-JWT-004 (API - Positivo):** Autenticação e geração de Token JWT após verificação de código[cite: 1, 3]
  * **Objetivo:** Validar autenticação via API de uma conta devidamente verificada[cite: 1].
  * **Endpoint / Método:** `POST /api/auth/login`
  * **Payload:**
    ```json
    {
      "email": "verificado@nasalinha.com",
      "password": "Senha@123"
    }
    ```
  * **Resultado Esperado:** Status Code `200 OK`[cite: 7] retornando um Token JWT válido no corpo da resposta ou headers[cite: 1].

#### Requisito RF-03: Controle de níveis (RBAC)[cite: 1]

* **CT-JWT-005 (Funcional - Negativo):** Acesso de usuário comum a rotas administrativas via UI[cite: 1, 3]
  * **Objetivo:** Garantir que usuário comum não visualize/acesse abas de "Temporadas" e "Moderação"[cite: 1].
  * **Pré-condições:** Usuário com perfil `USER` autenticado.
  * **Passos:**
    1. Fazer login como usuário comum.
    2. Verificar o menu de navegação.
    3. Tentar acessar a URL `/admin/moderacao` diretamente pelo navegador.
  * **Resultado Esperado:** O menu não exibe os botões de moderação/temporadas e o acesso direto pela URL redireciona para a home ou tela de erro 403 (Acesso Negado)[cite: 1].

* **CT-JWT-006 (API - Positivo):** Acesso do Administrador a endpoint restrito[cite: 1, 3]
  * **Objetivo:** Validar que o perfil ADMINISTRADOR consegue acessar endpoints de moderação informando o Token JWT[cite: 1].
  * **Endpoint / Método:** `GET /api/admin/seasons`
  * **Header:** `Authorization: Bearer <TOKEN_JWT_ADMIN>`
  * **Resultado Esperado:** Status Code `200 OK`[cite: 7] com a lista das temporadas ativas.

#### Requisito RNF-01: Segurança de dados[cite: 1]

* **CT-JWT-007 (Banco de Dados - Negativo/Segurança):** Verificação de Hashing de Senha e não exposição em API[cite: 1, 3]
  * **Objetivo:** Garantir que as senhas não fiquem em texto claro no banco PostgreSQL e não sejam retornadas no endpoint de perfil do usuário[cite: 1].
  * **Passos:**
    1. Realizar uma consulta direta ao banco PostgreSQL: `SELECT password FROM "User" WHERE email = 'test@nasalinha.com';`.
    2. Realizar a requisição `GET /api/users/me` com Token JWT.
  * **Resultado Esperado:** No banco, a senha deve estar gravada como hash seguro (ex: Bcrypt)[cite: 1]. Na resposta da API, o atributo `password` **nunca** deve estar presente[cite: 1].

#### Cenário de Regressão (JWT)[cite: 1, 3]

* **CT-JWT-REG-001 (Regressão de Autenticação e Acesso):**[cite: 1, 3]
  * **Simulação de Bug:** Suponha que o erro no bloqueio de login de contas não verificadas (RF-02) foi corrigido[cite: 1, 3].
  * **Fluxo de Testes de Regressão a Repetir:**
    1. Repetir o cadastro de novo usuário (CT-JWT-001)[cite: 1].
    2. Repetir a validação do bloqueio de login sem código (CT-JWT-003)[cite: 1].
    3. Confirmar o código no Mailtrap e efetuar login com sucesso (CT-JWT-004)[cite: 1].
    4. Validar as permissões de acesso RBAC de usuário comum vs ADMIN (CT-JWT-005 e CT-JWT-006)[cite: 1].
    5. Testar a realização de um Check-in (para garantir que a sessão/Token mantêm a autenticidade no fluxo dependente)[cite: 1].

---

### Área Core 2: Check-in por Foto

#### Requisito RF-04: Registro de atividade[cite: 1]

* **CT-CHK-001 (Funcional - Positivo):** Upload de foto e realização de check-in diário[cite: 1, 3]
  * **Objetivo:** Garantir a criação de check-in com envio de imagem válida associada à temporada ativa[cite: 1].
  * **Pré-condições:** Usuário logado e nenhuma atividade registrada no dia[cite: 1].
  * **Passos:**
    1. Acessar a tela de Check-in.
    2. Selecionar uma imagem em formato `.png` ou `.jpg`.
    3. Clicar no botão "Enviar Check-in".
  * **Resultado Esperado:** Envio da imagem para o Cloudinary, mensagem de confirmação na UI e registro gravado[cite: 1].

* **CT-CHK-002 (API - Negativo):** Bloqueio de envio de check-in sem arquivo de imagem[cite: 1, 3]
  * **Objetivo:** Garantir validação de erro ao tentar realizar o check-in enviando um arquivo inválido (ex: `.pdf`) ou body sem imagem[cite: 1].
  * **Endpoint / Método:** `POST /api/checkins`
  * **Body (multipart/form-data):** Arquivo do tipo `documento.txt` ou campo `photo` em branco.
  * **Resultado Esperado:** Status Code `400 Bad Request`[cite: 7] com a mensagem indicando a obrigatoriedade de envio de uma imagem válida[cite: 1].

#### Requisito RF-05: Fluxo de status[cite: 1]

* **CT-CHK-003 (Funcional - Positivo):** Ciclo de vida do status do Check-in[cite: 1, 3]
  * **Objetivo:** Verificar se o check-in nasce como PENDENTE e só altera após ação do ADMIN[cite: 1].
  * **Passos:**
    1. Realizar um check-in como usuário comum e verificar seu status inicial na tela.
    2. Logar como ADMINISTRADOR na área de Moderação.
    3. Aprovar ou Rejeitar o check-in cadastrado.
    4. Recarregar a tela no perfil do usuário comum.
  * **Resultado Esperado:** Check-in recém-criado exibe status `PENDENTE`[cite: 1]. Após ação no painel ADMIN, o status é atualizado na UI para `APROVADO` ou `REJEITADO`[cite: 1].

* **CT-CHK-004 (API - Negativo):** Tentativa de alteração de status por usuário comum[cite: 1, 3]
  * **Objetivo:** Garantir que apenas administradores alterem o status do check-in[cite: 1].
  * **Endpoint / Método:** `PATCH /api/checkins/:id/status`
  * **Header:** `Authorization: Bearer <TOKEN_USUARIO_COMUM>`
  * **Payload:** `{"status": "APROVADO"}`
  * **Resultado Esperado:** Status Code `403 Forbidden`[cite: 7].

#### Requisito RF-06: Propriedade[cite: 1]

* **CT-CHK-005 (Funcional - Negativo):** Tentativa de modificação de check-in de outro usuário via UI[cite: 1, 3]
  * **Objetivo:** Garantir o isolamento e privacidade dos check-ins de outros usuários[cite: 1].
  * **Passos:**
    1. Logar com o Usuário A.
    2. Acessar o feed/histórico de check-ins do Usuário B.
  * **Resultado Esperado:** Não são exibidas opções de edição, exclusão ou visualização restrita referentes às mídias do Usuário B[cite: 1].

* **CT-CHK-006 (API - Negativo):** Tentativa de exclusão de check-in alheio via API[cite: 1, 3]
  * **Objetivo:** Impedir exclusão de check-in que não pertença ao token autenticado[cite: 1].
  * **Endpoint / Método:** `DELETE /api/checkins/:id_checkin_do_usuario_B`
  * **Header:** `Authorization: Bearer <TOKEN_USUARIO_A>`
  * **Resultado Esperado:** Status Code `403 Forbidden` ou `404 Not Found`[cite: 7].

#### Requisito RNF-02: Integridade de arquivos[cite: 1]

* **CT-CHK-007 (Infraestrutura/Servidor - Positivo):** Remoção de arquivos temporários do servidor[cite: 1, 3]
  * **Objetivo:** Garantir que os arquivos temporários criados no upload local sejam excluídos do servidor Docker após o envio ao Cloudinary[cite: 1].
  * **Passos:**
    1. Enviar um check-in com foto via API/UI.
    2. Verificar a pasta temporária do container Node.js (`/tmp` ou `/uploads`) via comando `docker exec`.
    3. Forçar um upload com falha no Cloudinary e verificar novamente a pasta temporária.
  * **Resultado Esperado:** Tanto no fluxo de sucesso quanto no fluxo de falha, o arquivo temporário é sumariamente removido do servidor local[cite: 1].

#### Cenário de Regressão (Check-in)[cite: 1, 3]

* **CT-CHK-REG-001 (Regressão do Fluxo de Mídia e Duplicidade):**[cite: 1, 3]
  * **Simulação de Bug:** Suponha que o erro que permitia enviar um segundo check-in no mesmo dia foi corrigido[cite: 1, 3].
  * **Fluxo de Testes de Regressão a Repetir:**
    1. Executar um check-in válido no dia (CT-CHK-001)[cite: 1].
    2. Tentar realizar um segundo check-in na mesma data para validar o bloqueio[cite: 1].
    3. Tentar fazer o upload com uma foto repetida[cite: 1].
    4. Logar como ADMIN para alterar o status do check-in realizado (CT-CHK-003)[cite: 1].
    5. Conferir a pontuação refletida e a atualização do ranking[cite: 1].

---

### Área Core 3: Sistema de Pontos

#### Requisito RF-07: Atribuição de pontos[cite: 1]

* **CT-PTS-001 (Funcional - Positivo):** Atualização do saldo de pontos e ranking após aprovação[cite: 1, 3]
  * **Objetivo:** Verificar se a aprovação de um check-in adiciona os pontos corretamente à conta e atualiza o ranking[cite: 1].
  * **Passos:**
    1. Verificar o saldo inicial e a posição no Ranking do Usuário.
    2. Realizar um check-in e aprovar através do painel ADMIN.
    3. Consultar novamente a tela do Ranking e Perfil do usuário.
  * **Resultado Esperado:** O saldo de pontos é incrementado conforme a regra do negócio e a posição no ranking é recalculada instantaneamente[cite: 1].

* **CT-PTS-002 (API - Positivo):** Consulta do saldo atualizado via endpoint de Pontuação[cite: 1, 3]
  * **Objetivo:** Validar se o endpoint de perfil/pontos reflete o valor exato persistido no banco de dados[cite: 1].
  * **Endpoint / Método:** `GET /api/users/ranking` ou `GET /api/users/points`
  * **Resultado Esperado:** Status Code `200 OK`[cite: 7] e JSON contendo a propriedade `points` calculada de forma exata sem divergências.

#### Requisito RF-08: Consistência de saldo[cite: 1]

* **CT-PTS-003 (Funcional - Positivo):** Subtração de pontos ao remover/cancelar check-in[cite: 1, 3]
  * **Objetivo:** Garantir a remoção proporcional de pontos ao deletar ou revogar a aprovação de um check-in[cite: 1].
  * **Passos:**
    1. Anotar a pontuação atual do usuário.
    2. Excluir/Revogar um check-in anteriormente aprovado.
    3. Atualizar a página de Ranking/Perfil.
  * **Resultado Esperado:** Os pontos do check-in removido são deduzidos do saldo total do usuário e o ranking é ajustado automaticamente[cite: 1].

* **CT-PTS-004 (API - Negativo):** Inconsistência por exclusão repetida de check-in[cite: 1, 3]
  * **Objetivo:** Impedir que a chamada duplicada de deleção subtraia pontos múltiplas vezes (Garantia de Idempotência)[cite: 1].
  * **Endpoint / Método:** `DELETE /api/checkins/:id` (executado duas vezes seguidas).
  * **Resultado Esperado:** A primeira chamada retorna `200 OK` (subtrai os pontos de forma correta)[cite: 7]. A segunda chamada retorna `404 Not Found`[cite: 7], sem debitar novamente o saldo do usuário.

#### Requisito RF-09: Regras de ranking[cite: 1]

* **CT-PTS-005 (Funcional - Positivo):** Ordenação decrescente do Ranking[cite: 1, 3]
  * **Objetivo:** Validar se a lista de classificação exibe os usuários da maior pontuação para a menor[cite: 1].
  * **Passos:**
    1. Acessar a página de Ranking da aplicação.
    2. Analisar a lista exibida.
  * **Resultado Esperado:** Os usuários são listados rigorosamente em ordem decrescente por pontuação acumulada[cite: 1].

* **CT-PTS-006 (API - Negativo):** Bloqueio de check-in efetuado em intervalo inferior a 24 horas[cite: 1, 3]
  * **Objetivo:** Validar a regra de tempo limite entre pontuações via API[cite: 1].
  * **Endpoint / Método:** `POST /api/checkins` (realizado 2 horas após um check-in com sucesso).
  * **Resultado Esperado:** Status Code `400 Bad Request` ou `422 Unprocessable Entity`[cite: 7] informando o bloqueio por intervalo de 24 horas não atingido[cite: 1].

#### Cenário de Regressão (Pontos)[cite: 1, 3]

* **CT-PTS-REG-001 (Regressão de Cálculo de Saldo e Exibição de Ranking):**[cite: 1, 3]
  * **Simulação de Bug:** Suponha que o erro de inconsistência do cálculo do saldo de pontos após a remoção de um check-in foi corrigido[cite: 1, 3].
  * **Fluxo de Testes de Regressão a Repetir:**
    1. Realizar um check-in e aprovar para somar pontos (CT-PTS-001)[cite: 1].
    2. Excluir o check-in e verificar se a pontuação foi devidamente deduzida do saldo (CT-PTS-003)[cite: 1].
    3. Consultar a página do ranking para verificar se a reordenação em ordem decrescente está correta para todos os participantes (CT-PTS-005)[cite: 1].
    4. Tentar efetuar um novo check-in em menos de 24 horas para retestar a trava temporal (CT-PTS-006)[cite: 1].
