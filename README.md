# Auditoria de Qualidade: NaSalinha

Portfólio de QA do desafio **Quality Assurance 2026.2 (Comp Júnior)**.

Este repositório documenta a auditoria de qualidade do **NaSalinha**, um sistema de check-in gamificado em que usuários registram presença com foto, acumulam pontos e competem em um ranking por temporada.

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

**Tipos de teste:**

- **Funcional:** valida o comportamento na tela.
- **API:** valida endpoints, status codes e regras de negócio via Postman.
- **Regressão:** garante que correções não quebraram o que já funcionava.

**Planejamento:** para cada área Core, um caso funcional, um caso de API e um caso de regressão, com 2 casos de teste por requisito.

**Bugs:** mínimo de 2 por área, classificados por severidade (Crítico, Maior ou Menor) e registrados com passos para reproduzir, resultado esperado, resultado obtido e evidência.

---

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
