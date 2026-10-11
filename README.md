# Auditoria de Qualidade: NaSalinha


Portfólio de QA do desafio **Quality Assurance 2026.2 (Comp Júnior)**

Este repositório documenta a auditoria de qualidade do **NaSalinha**, um sistema preparado especificamente para o treinamento de novos Analistas de Quality Assurance
---

## Escopo da auditoria

A auditoria cobre três funcionalidades Core:

1. **Autenticação JWT:** autenticação e gestão de acesso.
2. **Check-in por Foto:** upload de provas e validação de mídia.
3. **Sistema de Pontos:** cálculo, persistência no banco e reflexo no ranking.

---

## Tecnologias e ferramentas

**Sistema auditado**

| Camada | Tecnologias |
|---|---|
| Backend | Node.js, Express, Prisma ORM |
| Frontend | React 18, TailwindCSS |
| Banco de dados | PostgreSQL (via Docker) |
| Serviços externos | Cloudinary (imagens) e Mailtrap (e-mails) |

**Ferramentas de QA**

| Finalidade | Ferramenta |
|---|---|
| Testes de API | Postman |
| Testes de interface | Navegador (Chrome) |
| Ambiente local | Docker e Docker Compose |
| Documentação | Markdowns, planilhas |


---
## Estratégia de testes

### Objetivo
Auditar a qualidade das três funcionalidades core do NaSalinha, registrando os defeitos encontrados de forma clara, reproduzível e rastreável aos requisitos.

### Escopo
**Dentro:** os requisitos RF-01 a RF-09, RNF-01 e RNF-02 do documento de requisitos oficial.
**Fora:** performance, segurança avançada (ex.: pentest), navegadores além do Chrome e testes unitários do código.

### Técnicas
Partição de equivalência, análise de valor limite e cenários positivos e negativos.

### Casos de teste
Serão escritos **2 casos por requisito**: 1 funcional (UI) e 1 de API, cada um positivo ou negativo. Nos requisitos sem interface (RNF-01 e RNF-02), o caso funcional é substituído por uma verificação no banco ou no servidor. Cada área terá ainda **1 caso de regressão**.

### Cobertura dos requisitos

| Área | Requisito | Cenários principais |
|---|---|---|
| JWT | RF-01 Cadastro e verificação | Cadastro válido gera código por e-mail (Mailtrap) e conta com status PENDENTE; e-mail duplicado ou campos vazios são rejeitados |
| JWT | RF-02 Restrição de login | Login de conta não verificada é bloqueado com a mensagem "Por favor, verifique sua conta para continuar"; login após verificar o código funciona; código inválido é recusado |
| JWT | RF-03 Controle de níveis (RBAC) | Usuário comum e requisição sem token não acessam Temporadas e Moderação; ADMINISTRADOR acessa |
| JWT | RNF-01 Segurança de dados | Senha gravada como hash no banco; a API nunca devolve a senha |
| Check-in | RF-04 Registro de atividade | Check-in com imagem válida é associado à temporada ativa; segundo check-in no mesmo dia é bloqueado; foto repetida é bloqueada; envio sem imagem ou com arquivo que não é imagem é rejeitado |
| Check-in | RF-05 Fluxo de status | Novo check-in nasce PENDENTE; só o administrador aprova ou rejeita |
| Check-in | RF-06 Propriedade | Usuário não vê, edita nem exclui check-in de outro usuário |
| Check-in | RNF-02 Integridade de arquivos | Arquivo temporário é removido do servidor após o envio ao provedor de nuvem, com sucesso e com falha |
| Pontos | RF-07 Atribuição de pontos | Pontos somam ao saldo e o ranking atualiza automaticamente |
| Pontos | RF-08 Consistência de saldo | Ao remover um check-in, os pontos correspondentes são subtraídos do saldo e do ranking |
| Pontos | RF-09 Regras de ranking | Ranking em ordem decrescente de pontuação; novo check-in em menos de 24 horas é bloqueado |

### Regressão
Para cada área será descrito um cenário simulado: caso um bug reportado por mim seja corrigido, quais testes seriam repetidos para garantir que a correção não causou efeitos colaterais. Os exemplos abaixo serão trocados pelos bugs reais encontrados.

- **JWT:** caso o erro de verificação de conta seja corrigido, repetir o cadastro (RF-01), o bloqueio de login com a mensagem (RF-02), o login após verificar, o acesso de ADMIN e de usuário comum (RF-03) e o check-in (que depende do login).
- **Check-in:** caso o erro de check-in duplicado seja corrigido, repetir o check-in válido, a foto repetida, a mudança de status pelo administrador (RF-05) e a conferência de pontos e ranking.
- **Pontos:** caso o erro de cálculo de saldo seja corrigido, repetir a soma após check-in (RF-07), a subtração ao remover (RF-08), a ordenação do ranking e o bloqueio de check-in em menos de 24 horas (RF-09).

### Gestão de bugs
Meta mínima: **2 bug reports por área** (6 no total). Cada bug terá ID, título, ambiente, caso de teste e requisito relacionados, passos para reproduzir, resultado esperado, resultado obtido, severidade, evidência (print, vídeo ou log) e status.

| Severidade | Critério |
|---|---|
| Crítico | Trava o sistema ou impede o uso da funcionalidade |
| Maior | Causa erro de lógica ou de regra de negócio |
| Menor | Erro visual ou de texto, sem impacto na lógica |

**Status:** Novo → Corrigido → Re-testado → Fechado. O status fica como um campo no topo de cada arquivo em `docs/bug-reports/`.

### Documentação
A documentação segue a **Opção B (documental)**: arquivos em Markdown dentro da pasta 

### Critérios de entrada e saída
- **Entrada:** ambiente rodando no Docker e casos de teste escritos antes da execução.
- **Saída:** casos planejados executados, ao menos 2 bugs documentados por área, casos ad-hoc registrados, cenários de regressão descritos e documentação revisada.

### Relatório de testes final
Ao fim da execução, será entregue um relatório com casos planejados x executados, casos aprovados (Pass) x reprovados (Fail) e distribuição dos bugs por severidade (Crítico, Maior e Menor).

## Como rodar o projeto

# 1. Pré-requisitos

Docker (com Docker Compose) e Git instalados.
Conta gratuita no Mailtrap (Sandbox), para os e-mails.
Conta gratuita no Cloudinary, para as fotos.

Confirme que o Docker está funcionando:
```
bash
docker --version
docker-compose --version
```

## 2. Clonar o repositório

```
bash
git clone LINK_DO_REPOSITORIO_NASALINHA
cd NOME_DA_PASTA
```

## Cronograma e Entregas por Semana

| Semana | Foco Principal | Entregas e Ações Previstas |
| :--- | :--- | :--- |
| **Semanas 1-2** | Setup, Exploração e Documentação | Subir ambiente em Docker; criar repositório no GitHub; explorar fluxos do utilizador; redigir o `README.md` inicial.|
| **Semana 3** | Planeamento e Design de Testes | Definir cenários para as 3 áreas Core; escrever Casos de Teste (positivos/negativos); configurar a ferramenta de gestão |
| **Semana 4** | Execução de Testes Manuais (UI) | Executar os casos de teste de interface planeados; registar bugs |
| **Semana 5** | Testes de API e Integração | Validar endpoints e status codes no Insomnia/Postman; verificar regras de negócio no backend; registar bugs de API |
| **Semana 6** | Testes de Regressão e Re-teste | Re-testar bugs reportados; descrever a simulação do teste de regressão; refinar relatórios de bugs |
| **Semana 7** | Finalização, Revisão e Entrega | Revisão geral da documentação; gravação do vídeo de demonstração (máx. 10 min); disponibilizar repositório final. |

---
