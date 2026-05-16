# 🔄 Testes de Regressão – Semana 6

**Projeto NaSalinha – Garantia de Qualidade e Integridade de Dados**

---

## TR-01: Validação de Duplicidade de Mídia no Check-in
**Relacionado ao:** BUG-04 (Severidade Alta)  
**Objetivo:** Garantir que o backend impeça o reuso da mesma imagem para múltiplos check-ins do mesmo usuário, protegendo a integridade da gamificação.

| Passo | Ação | Resultado Esperado |
|:---:|:---|:---|
| **1** | Realizar autenticação com conta de Trainee e armazenar o token. | Token JWT retornado com sucesso. |
| **2** | Enviar POST para `/api/checkins` com uma imagem `foto_teste.png` no corpo (multipart). | Status **201 Created**. Check-in registrado. |
| **3** | Enviar novo POST para `/api/checkins` com a **mesma** imagem `foto_teste.png`. | Status **400 Bad Request**. |
| **4** | Validar o corpo da resposta do erro. | Mensagem: "Esta foto já foi utilizada em outro check-in". |

---

## TR-02: Consistência de Saldo e Exclusão em Cascata (Admin)
**Relacionado aos:** BUG-08 (Severidade Alta)  
**Objetivo:** Verificar se a exclusão de um check-in pelo Administrador remove corretamente os dados órfãos na tabela de pontos e atualiza o ranking.

| Passo | Ação | Resultado Esperado |
|:---:|:---|:---|
| **1** | Consultar o ranking atual de um usuário específico (Ex: 500 pts). | Saldo inicial registrado. |
| **2** | Identificar o `id` de um check-in aprovado deste usuário. | ID capturado para deleção. |
| **3** | Enviar DELETE para `/api/checkins/{id}` utilizando token de **Admin**. | Status **200 OK** ou **204 No Content**. |
| **4** | Consultar o ranking do usuário novamente. | Saldo atualizado (Ex: 500 - X pts). |
| **5** | (DB Check) Consultar a tabela `Points` filtrando pelo `checkinId` excluído. | Nenhum registro retornado (Exclusão em cascata confirmada). |

---

## TR-03: Autorização de Acesso ao Ranking (RBAC)
**Relacionado ao:** BUG-11 (Severidade Alta)  
**Objetivo:** Validar se as permissões de acesso (Role-Based Access Control) estão configuradas corretamente para permitir que Membros e Admins visualizem o ranking, bloqueando Trainees.

| Passo | Ação | Resultado Esperado |
|:---:|:---|:---|
| **1** | Enviar GET para `/api/rankings` com token de **Membro**. | Status **200 OK** + Lista de usuários. |
| **2** | Enviar GET para `/api/rankings` com token de **Admin**. | Status **200 OK** + Lista de usuários. |
| **3** | Enviar GET para `/api/rankings` com token de **Trainee**. | Status **403 Forbidden** ou **401 Unauthorized**. |
| **4** | Validar se o corpo da resposta para Trainee indica erro de permissão. | Mensagem clara de "Acesso negado" ou similar. |


