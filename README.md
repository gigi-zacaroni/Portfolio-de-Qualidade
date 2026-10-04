# Auditoria de Qualidade: NaSalinha

Portfólio de QA do desafio **Quality Assurance 2026.2 (Comp Júnior)**.

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
