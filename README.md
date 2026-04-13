# Diário de QA: Setup e Estratégia de Testes (Semana 1)
* **Projeto:** NaSalinha - Sistema de Check-in Gamificado
* **Analista:** Trainee -> Bruno Santos Vilas Boas
* **Status:** Iniciado - Semana 1

---

## Visão Geral do Desafio
Iniciei minha jornada como observador da experiência do usuário no projeto NaSalinha. Meu objetivo é realizar uma auditoria completa para identificar falhas de segurança, erros de interface e inconsistências nas regras de negócio, atuando em correções antes do deploy.

---

## Tecnologias e Infraestrutura
Para este ciclo de testes, o ambiente foi configurado com a seguinte stack:
* **Frontend:** React 18 e TailwindCSS.
* **Backend:** Node.js, Express e Prisma ORM.
* **Banco de Dados:** PostgreSQL rodando via Docker.
* **Serviços Externos:** Mailtrap para sandbox de e-mails e Cloudinary para gestão de mídias.

---

## Configuração do Ambiente (Checklist Semana 1)
Concluí as etapas de setup obrigatórias para garantir a rastreabilidade dos testes:
* [x] **Variáveis de Ambiente:** Configurei o arquivo `.env` no backend com minhas credenciais do Mailtrap e Cloudinary para validar integrações de e-mail e fotos.
* [x] **Containers:** Subi o ambiente completo via `docker-compose up --build`.
* [x] **Banco de Dados:** Executei as migrations e seeders (`npx prisma db seed`) para popular o sistema com usuários Admin, Membro e Trainee.
* [x] **Repositório:** Estruturei este repositório no GitHub para servir como meu Portfólio de Qualidade.

---

## Planejamento de Auditoria
Focarei meus esforços nas 3 funcionalidades "Core" exigidas pelo desafio:

1.  **Autenticação JWT:** Testes de login, refresh tokens e gestão de acesso.
2.  **Check-in por Foto:** Validação de upload de provas e integridade de mídia.
3.  **Sistema de Pontos:** Auditoria de cálculos, persistência no banco e atualização do ranking.

---

## Estrutura de Documentação
Adotei a **OPÇÃO B (Documental)** para organizar os artefatos de teste:
* `/docs/test-cases`: Planejamento de casos funcionais, de API e de regressão.
* `/docs/bug-reports`: Relatórios detalhados
