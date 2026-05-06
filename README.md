# Portfólio de Qualidade – Desafio QA 2026

Bem-vindo(a) ao meu repositório de testes do sistema **NaSalinha**.  
Este projeto tem como objetivo realizar uma auditoria completa de qualidade, documentando bugs, planejando casos de teste e validando funcionalidades essenciais da aplicação.

---

## Sobre o Projeto

**NaSalinha** é um sistema de check-in gamificado desenvolvido para o ecossistema da Comp Júnior.  
O ambiente contém inconsistências propositais para testar a capacidade de auditoria, configuração de ambiente e reporte de bugs de novos Analistas de Quality Assurance.

### Objetivo do Treinamento

Atuar como a **última linha de defesa** antes do deploy em produção:

- 🐛 **Mapear Bugs:** Encontrar falhas de segurança, erros de pontuação e problemas de interface.
- ⚙️ **Configuração de Infra:** Garantir que todos os serviços externos estejam integrados corretamente.
- 📋 **Validar Regras de Negócio:** Garantir que permissões de usuário (Admin, Membro e Trainee) e cálculos de ranking estejam íntegros.

### Níveis de Usuário

| Nível   | Permissões                                    |
|---------|-----------------------------------------------|
| Admin   | Acesso total, gerenciamento de temporadas     |
| Membro  | Check-ins, visualização de ranking            |
| Trainee | Check-ins e pontuação                         |

---

## Créditos de Desenvolvimento

O projeto original foi desenvolvido por **Lucas Henrique Lopes Costa** (Ex-Trainee da Comp Júnior) como parte de sua trilha de capacitação técnica.  
A manutenção atual e a coordenação deste desafio são de responsabilidade da **Diretoria de Projetos**.

---

## Tecnologias Utilizadas

| Camada      | Tecnologia                      |
|-------------|----------------------------------|
| Backend     | Node.js, Express, Prisma ORM     |
| Frontend    | React 18, TailwindCSS            |
| Banco de Dados | PostgreSQL (via Docker)        |
| Serviços    | Cloudinary (Imagens), Nodemailer/Mailtrap (E-mails) |
| Segurança   | bcrypt, JWT (refresh tokens), Helmet, CORS, Rate Limiting, Joi |

---

## Ferramentas de Teste

| Área                | Ferramenta                       |
|---------------------|----------------------------------|
| Testes de API       | Insomnia                         |
| Testes de Interface | Navegador (Chrome)               |
| Gestão de Testes    | Trello                           |
| Documentação        | Markdowns, Planilhas             |
| Ambiente            | Docker + Docker Compose          |

---

## Como Executar o Projeto Localmente

### 1. Pré-requisitos

- [Docker](https://www.docker.com/) instalado
- [Git](https://git-scm.com/) instalado
- Contas gratuitas criadas em:
  - [Mailtrap](https://mailtrap.io/) (Sandbox para e-mails)
  - [Cloudinary](https://cloudinary.com/) (Armazenamento de fotos)

### 2. Clone o Repositório

```bash
git clone https://github.com/seu-usuario/seu-repo-qa.git
cd seu-repo-qa
