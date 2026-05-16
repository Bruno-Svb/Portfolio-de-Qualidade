# Resumo de Métricas e Resultados do Projeto NaSalinha
**Desafio QA 2026** **Escopo:** Funcionalidades Core (Autenticação JWT, Check-in por Foto e Sistema de Pontos)

Este documento apresenta o consolidado dos testes planejados versus executados, bem como o mapeamento e a distribuição por severidade de todos os bugs críticos e lógicos identificados na auditoria do sistema.

---

## 1. Resumo Executivo (Métricas Gerais)

A tabela abaixo detalha o balanço de testes para cada uma das funcionalidades fundamentais do projeto. Ela quantifica o planejamento inicial frente à execução real e separa os resultados obtidos em cenários de sucesso (*Pass*) e falhas operacionais (*Fail*).

| Módulo Core / Funcionalidade | Testes Planejados | Testes Executados | Testes Passados (Pass) | Testes Falhados (Fail) | Total de Bugs Encontrados |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Módulo 1: Autenticação e Acesso (JWT)** | 6 | 6 | 5 | 1 | **1** |
| **Módulo 2: Check-in (Mídia/Foto)** | 6 | 6 | 3 | 3 | **3** |
| **Módulo 3: Pontuação e Ranking** | 7 | 7 | 1 | 6 | **6** |
| **TOTAL** | **19** | **19** | **9** | **10** | **10** |

* **Descrição e Análise:** O plano de testes cobriu um total de 18 cenários estratégicos divididos igualmente entre os três blocos centrais. A execução resultou em uma taxa de falha de aproximadamente **52%**, indicando uma fragilidade significativa nas regras de validação aplicadas no backend e na comunicação de estados na interface visual do usuário.

---

## 2. Distribuição e Mapeamento de Bugs por Severidade

A tabela de rastreabilidade a seguir vincula cada falha documentada no relatório ao seu respectivo componente técnico e nível de impacto, servindo de guia para que a equipe de desenvolvimento priorize as correções mais urgentes.

| ID do Bug | Título Resumido do Erro | Módulo Vinculado | Localização | Severidade |
| :---: | :--- | :--- | :--- | :---: |
| **BR-01** | Login permitido com e-mail não verificado | Autenticação JWT | Backend / API | **Alta** |
| **BR-02** | Atribuição imediata de pontos sem status pendente | Check-in (Mídia) | Backend | **Média** |
| **BR-03** | Múltiplos check-ins permitidos no mesmo dia | Check-in (Mídia) | Backend / API | **Alta** |
| **BR-04** | Permite upload da mesma foto repetidas vezes | Check-in (Mídia) | Backend / API | **Média** |
| **BR-05** | Quebra do fluxo de aprovação obrigatória do Admin | Pontos e Ranking | Backend | **Alta** |
| **BR-06** | Interface ativa permite reenvio após confirmação | Check-in (Mídia) | Frontend | **Alta** |
| **BR-07** | Contabilização prematura de pontos no ranking | Pontos e Ranking | Backend | **Alta** |
| **BR-08** | Pontuação não diminui após exclusão do check-in | Pontos e Ranking | Backend / BD | **Crítica** |
| **BR-09** | Burlar restrição temporal de 24 horas | Pontos e Ranking | Backend / API | **Alta** |
| **BR-10** | Acesso ao ranking bloqueado para Membros | Controle de Acesso (RBAC) | Backend / API | **Média** |

* **Descrição e Análise:** É possível observar que a maior densidade de problemas está concentrada nos módulos de *Check-in* e *Pontuação*. O backend falha sistematicamente em atuar como uma barreira rígida de validação de dados de entrada, confiando de forma insegura nas ações executadas pelo cliente.

---

## 3. Consolidado por Nível de Severidade (Impacto Técnico)

Esta visão agrupa as ocorrências pelo seu potencial de dano à experiência do usuário, integridade do banco de dados e regras críticas de negócio estipuladas no edital.

| Severidade | Quantidade | Impacto Prático no Sistema NaSalinha |
| :---: | :---: | :--- |
| ⚠️ **Crítica** | 1 | **Bloqueante / Corrupção de Dados:** Causa inconsistência matemática direta no saldo de pontuações e gera pontos falsos que nunca expiram ou diminuem após exclusões (BR-08). |
| 🛑 **Alta** | 6 | **Quebra de Segurança e Regras de Negócio:** Ignora janelas temporárias vitais, fura barreiras de autenticação JWT e permite reenvios imediatos e contínuos de formulários. |
| 🟡 **Média** | 3 | **Falhas de Validação e Moderação:** Permite inserção de dados redundantes (mesma foto) e burla a fila obrigatória de análise preliminar que deveria ficar restrita ao perfil Admin. |
| 🟢 **Baixa** | 0 | **Cosméticos/Estéticos:** Não foram registrados bugs puramente visuais nesta rodada, uma vez que as falhas de tela encontradas impactavam o fluxo de envio ativo. |

* **Descrição e Análise:** O alto número de bugs classificados como **Alta** e **Crítica** (7 das 9 ocorrências) evidencia a necessidade imediata de refatoração nos *Controllers* e *Services* do ecossistema. Correções focadas em travas de tempo e integridade referencial devem ser priorizadas imediatamente antes de qualquer liberação em ambiente de produção.
