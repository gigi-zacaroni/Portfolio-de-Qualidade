# Backlog e Matriz de Execução de Testes

**Projeto:** NaSalinha — Auditoria de Qualidade (Desafio QA 2026.2)  
**Documento:** Planejamento e Acompanhamento de Execução dos Casos de Teste  

---

## Legenda de Status e Severidade

| Campo | Opções / Definições |
| :--- | :--- |
| **Status** | `Não Iniciado` | `Em Execução` | `Pass` (Aprovado) | `Fail` (Reprovado/Bug) | `Bloqueado` |
| **Tipo** | `Funcional (UI)` | `API` | `Banco de Dados / Servidor` | `Regressão` | `Ad-hoc (Extra)` |
| **Severidade (Bugs)** | **Crítico:** Trava o sistema ou impede o uso da funcionalidade principais.<br>**Maior:** Erro de regra de negócio ou lógica sem travar tudo.<br>**Menor:** Erro visual, texto ou ajuste de layout. |

---

## 1. Área Core 1: Autenticação JWT

| ID | Requisito | Descrição Resumida | Tipo | Executado Por | Data Execução | Status | Bug Report Relacionado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-JWT-001** | RF-01 | Cadastro válido via UI e envio de e-mail Mailtrap | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-JWT-002** | RF-01 | Cadastro com e-mail duplicado via API | API | | | `Não Iniciado` | N/A |
| **CT-JWT-003** | RF-02 | Bloqueio de login com conta pendente via UI | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-JWT-004** | RF-02 | Login e geração de JWT após verificação | API | | | `Não Iniciado` | N/A |
| **CT-JWT-005** | RF-03 | Restrição de menu/rotas ADMIN para Usuário Comum | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-JWT-006** | RF-03 | Acesso a endpoint de moderação com perfil ADMIN | API | | | `Não Iniciado` | N/A |
| **CT-JWT-007** | RNF-01 | Hashing de senha no banco e omissão no payload | Banco / API | | | `Não Iniciado` | N/A |
| **CT-JWT-REG-001**| Regressão | Suíte de regressão para fluxo de Autenticação | Regressão | | | `Não Iniciado` | N/A |

---

## 2. Área Core 2: Check-in por Foto

| ID | Requisito | Descrição Resumida | Tipo | Executado Por | Data Execução | Status | Bug Report Relacionado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-CHK-001** | RF-04 | Check-in com imagem válida via UI | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-CHK-002** | RF-04 | Envio de check-in sem arquivo de imagem válido | API | | | `Não Iniciado` | N/A |
| **CT-CHK-003** | RF-05 | Transição de status PENDENTE -> APROVADO pelo ADMIN | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-CHK-004** | RF-05 | Tentativa de alteração de status por Usuário Comum | API | | | `Não Iniciado` | N/A |
| **CT-CHK-005** | RF-06 | Garantia de não edição/visualização de check-in alheio | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-CHK-006** | RF-06 | Tentativa de deleção de check-in de outro usuário | API | | | `Não Iniciado` | N/A |
| **CT-CHK-007** | RNF-02 | Remoção de arquivo temporário local do Docker | Servidor | | | `Não Iniciado` | N/A |
| **CT-CHK-REG-001**| Regressão | Suíte de regressão para fluxo de Check-in e Mídia | Regressão | | | `Não Iniciado` | N/A |

---

## 3. Área Core 3: Sistema de Pontos

| ID | Requisito | Descrição Resumida | Tipo | Executado Por | Data Execução | Status | Bug Report Relacionado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-PTS-001** | RF-07 | Soma de pontos e atualização de ranking após aprovação | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-PTS-002** | RF-07 | Validação do saldo retornado no endpoint de perfil | API | | | `Não Iniciado` | N/A |
| **CT-PTS-003** | RF-08 | Subtração de pontos ao excluir check-in aprovado | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-PTS-004** | RF-08 | Idempotência em requisições seguidas de exclusão | API | | | `Não Iniciado` | N/A |
| **CT-PTS-005** | RF-09 | Validação da ordem decrescente na tabela de Ranking | Funcional (UI) | | | `Não Iniciado` | N/A |
| **CT-PTS-006** | RF-09 | Bloqueio de segundo check-in em intervalo < 24h | API | | | `Não Iniciado` | N/A |
| **CT-PTS-REG-001**| Regressão | Suíte de regressão para Cálculo de Pontos e Ranking | Regressão | | | `Não Iniciado` | N/A |

---

## 4. Testes Ad-hoc / Extras (Bugs Não Planejados)

> *Utilize esta seção para registrar falhas encontradas durante a exploração livre do sistema que não estavam cobertas no planejamento inicial.*

| ID | Área / Módulo | Descrição do Teste Ad-hoc | Resultado Obter | Status | Bug Report Gerado |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-ADH-001** | | | | `Não Iniciado` | |
| **CT-ADH-002** | | | | `Não Iniciado` | |

---

## 5. Resumo da Execução (Métricas)

- **Total Planejado:** 24 Casos
- **Executados:** 0
- **Aprovados (Pass):** 0
- **Reprovados (Fail):** 0
- **Bloqueados:** 0
- **Taxa de Sucesso:** 0%
