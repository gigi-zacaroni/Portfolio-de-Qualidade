# BUG-CHK-001 — Sistema permite mais de um check-in no mesmo dia

| Campo | Informação |
|:--|:--|
| **ID** | BUG-CHK-001 |
| **Área Core** | 2 — Check-in por Foto (com impacto na Área 3 — Sistema de Pontos) |
| **Requisito** | RF-04 (Registro de atividade: check-in diário) e RF-09 (intervalo mínimo de 24h) |
| **Caso de teste relacionado** | CT-CHK-001 (Funcional – UI); relacionado a CT-PTS-006 (API) |
| **Severidade** | Maior (erro de regra de negócio, sem travar o sistema) |
| **Prioridade** | Alta |

## Descrição

O usuário consegue registrar vários check-ins no mesmo dia. O sistema aceita cada envio e grava um novo registro, sem nenhuma trava de limite diário nem de intervalo mínimo entre check-ins.

## Pré-condições

- Aplicação rodando via Docker, banco migrado e seed executado (temporada ativa existente).
- Upload de imagem funcionando (credenciais do Cloudinary configuradas).
- Usuário autenticado e sem check-in registrado no dia.

## Passos para reproduzir

1. Fazer login com um usuário comum (ex.: `lucas@example.com`).
2. Acessar a tela de Check-in, selecionar uma imagem `.png` ou `.jpg` e clicar em "Enviar Check-in".
3. Confirmar que o primeiro check-in foi registrado.
4. Sem esperar, repetir o passo 2 com a mesma imagem ou com outra.
5. Repetir uma terceira vez (opcional).
6. Observar a lista de atividades e o total de pontos no dashboard.

## Resultado esperado

A partir do segundo envio, o sistema bloqueia o check-in e exibe uma mensagem informando que já houve check-in no dia (ou que o intervalo de 24h ainda não foi atingido). Apenas **um** check-in por dia é registrado e pontuado.

## Resultado obtido

Todos os envios são aceitos, com mensagem de sucesso. Cada um cria um novo check-in e soma novamente os pontos da temporada, e o saldo e a posição no ranking aumentam a cada envio.

## Análise técnica (código-fonte)

Inspeção do backend :

- `src/controllers/checkin.controller.js` (`create`): o fluxo é validar a presença do arquivo, enviar para o Cloudinary, buscar a temporada ativa, criar o check-in e atualizar os pontos. **Não há consulta** a check-ins anteriores do mesmo usuário (por dia ou nas últimas 24h) antes da criação.
- O check-in é criado direto com status `APPROVED`, e os pontos (`pointsPerCheckIn` da temporada) são somados na mesma requisição, então cada envio extra gera pontuação imediata.
- Como a regra não está no backend, a falha também é reproduzível por API (`POST /api/checkins`), o que se liga ao CT-PTS-006.

## Impacto

- Inflação de pontos: um usuário pode acumular pontuação ilimitada enviando check-ins repetidos.
- O ranking perde credibilidade, porque quem repete envios passa quem cumpre a regra.
- Prejudica a integridade dos dados do Sistema de Pontos (Área 3).

## Evidências

- [Check-in](../../evidencias/Tela_inicial.png)  Print da lista de atividades com vários check-ins no mesmo dia
- [Pontuação](../../evidencias/pontuacao.png) Print do ranking refletindo a pontuação extra

