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
**Relacionado aos:** BUG-05 e BUG-06 (Severidade Alta)  
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
**Relacionado ao:** BUG-07 (Severidade Alta)  
**Objetivo:** Validar se as permissões de acesso (Role-Based Access Control) estão configuradas corretamente para permitir que Membros e Admins visualizem o ranking, bloqueando Trainees.

| Passo | Ação | Resultado Esperado |
|:---:|:---|:---|
| **1** | Enviar GET para `/api/rankings` com token de **Membro**. | Status **200 OK** + Lista de usuários. |
| **2** | Enviar GET para `/api/rankings` com token de **Admin**. | Status **200 OK** + Lista de usuários. |
| **3** | Enviar GET para `/api/rankings` com token de **Trainee**. | Status **403 Forbidden** ou **401 Unauthorized**. |
| **4** | Validar se o corpo da resposta para Trainee indica erro de permissão. | Mensagem clara de "Acesso negado" ou similar. |

# 🐛 Bug Reports – Semana Extra (Temporadas)


## 📊 Resumo dos Bugs Encontrados

| ID | Título | Área | CT | Severidade | Status |
|----|--------|------|-----|------------|--------|
| BUG-08 | Data inválida na criação de temporada retorna 1970 via API | Temporadas | CT-AD-01 | 🟠 Alta | Aberto |
| BUG-09 | Duas temporadas aparecem como ativas no frontend quando criadas pela API | Temporadas | CT-AD-02 | 🟡 Média | Aberto |


# BUG-08 – Data inválida na criação de temporada retorna 1970 via API

| Campo | Detalhe |
|-------|---------|
| Título | API aceita data inválida e retorna timestamp zerado (época Unix) como data da temporada |
| Localização do Erro | Backend (Service de Temporadas / Validação de Entrada) |
| Severidade | 🟠 Alta |
| CT Relacionado | CT-AD-01 |


## Passos para Reproduzir

1. Fazer login como Admin e obter token JWT
2. Enviar requisição POST para /api/seasons com uma data inválida no campo de data (ex: `"startDate": "data-invalida"` ou `"startDate": ""`)
3. Verificar a resposta do servidor
4. Consultar a temporada criada e observar o valor retornado no campo de data

## Configuração no Insomnia

Headers:
  Authorization: Bearer {{token_admin}}
  Content-Type: application/json

Body → JSON:
```json
{
  "name": "Temporada Teste",
  "startDate": "data-invalida",
  "endDate": "data-invalida"
}
```


## Resultado Esperado

- Status Code: 400 (Bad Request)
- Mensagem: "Data inválida. Utilize o formato ISO 8601 (ex: 2026-01-01)"
- Sistema deve rejeitar valores de data que não possam ser convertidos para um formato de data válido
- Nenhuma temporada deve ser criada com dados inválidos


## Resultado Atual

- Status Code: 201 (Created)
- Temporada é criada com sucesso
- O campo de data retorna `1970-01-01T00:00:00.000Z` (época Unix / timestamp zerado)
- Nenhuma mensagem de erro é exibida


## Análise de Causa Raiz (RCA)

O endpoint POST /api/seasons não está validando o valor recebido nos campos de data antes de realizar a conversão. Quando uma string inválida é passada para `new Date()` no JavaScript, o resultado é `NaN`, e ao ser salvo no banco como timestamp numérico, resulta em `0` — que corresponde exatamente à época Unix: `1970-01-01T00:00:00.000Z`. A ausência de validação de entrada permite que dados corrompidos sejam persistidos no banco.

Trecho provável do problema (backend):

```javascript
// Service de temporadas - CÓDIGO ATUAL (com falha)
async createSeason(data) {
    // FALTA AQUI: Validação do formato da data
    // const startDate = new Date(data.startDate);
    // if (isNaN(startDate.getTime())) {
    //     throw new Error('Data inválida. Utilize o formato ISO 8601');
    // }

    return await prisma.season.create({
        data: {
            name: data.name,
            startDate: new Date(data.startDate), // new Date("data-invalida") → Invalid Date → salvo como 0 (1970)
            endDate: new Date(data.endDate)
        }
    });
}
```


## Impacto

- 🗄️ Dados corrompidos persistidos no banco: temporadas com datas em 1970
- 📅 Lógica de ativação/expiração de temporadas comprometida
- 🔓 Ausência de validação de entrada viola boas práticas de segurança e integridade de dados
- 📊 Relatórios e rankings vinculados a temporadas podem exibir informações incorretas


# BUG-09 – Duas temporadas aparecem como ativas no frontend quando criadas pela API

| Campo | Detalhe |
|-------|---------|
| Título | Interface exibe duas temporadas simultaneamente como ativas quando uma segunda temporada é criada via API |
| Localização do Erro | Frontend (Componente de exibição de temporadas) |
| Severidade | 🟡 Média |
| CT Relacionado | CT-AD-02 |


## Passos para Reproduzir

1. Verificar que existe uma temporada ativa exibida no frontend
2. Fazer login como Admin e obter token JWT
3. Enviar requisição POST para /api/seasons via API (Insomnia/Postman) criando uma nova temporada com status ativo
4. Sem recarregar a página, ou após recarregar, acessar a tela de temporadas no frontend
5. Observar quantas temporadas são exibidas como ativas


## Configuração no Insomnia

Headers:
  Authorization: Bearer {{token_admin}}
  Content-Type: application/json

Body → JSON:
```json
{
  "name": "Nova Temporada Ativa",
  "startDate": "2026-06-01",
  "endDate": "2026-12-31",
  "isActive": true
}
```


## Resultado Esperado

- Apenas uma temporada deve estar ativa por vez
- Ao criar uma nova temporada ativa via API, a temporada anterior deve ser automaticamente desativada
- Frontend deve exibir somente a temporada ativa atual
- Comportamento deve ser idêntico independentemente do canal de criação (UI ou API)


## Resultado Atual

- Duas temporadas aparecem como ativas simultaneamente no frontend
- O problema ocorre exclusivamente quando a segunda temporada é criada via API (não pela interface)
- O banco de dados pode estar correto, indicando que o problema é de exibição/renderização no frontend
- A interface não reflete corretamente o estado real do sistema quando a criação é feita fora dela


## Análise de Causa Raiz (RCA)

O bug é visual e está isolado no frontend. A causa provável é que o componente de listagem de temporadas não está filtrando ou priorizando corretamente o campo `isActive` ao renderizar os cards, exibindo todas as temporadas cujo período de datas ainda está vigente, independentemente do status real. Como a criação pela UI pode disparar uma atualização de estado local (store/context) que a criação via API não dispara, a inconsistência aparece apenas neste fluxo.

Trecho provável do problema (frontend):

```javascript
// Componente de temporadas - CÓDIGO ATUAL (com falha)
const activeSeasons = seasons.filter(
    // ERRO: Filtra por período de datas, não pelo campo isActive
    s => new Date(s.startDate) <= new Date() && new Date(s.endDate) >= new Date()
);

// CÓDIGO CORRETO esperado:
const activeSeasons = seasons.filter(s => s.isActive === true);
```


## Impacto

- 😕 Experiência do usuário prejudicada: informação ambígua sobre qual temporada está em vigor
- 🏆 Usuários podem ficar confusos sobre qual temporada está acumulando seus pontos
- 🔍 Inconsistência entre o canal de criação (UI vs API) indica falta de cobertura de testes de integração
- 🟡 Severidade Média: dados no backend permanecem íntegros; o impacto é restrito à camada visual


## 📎 Distribuição de Severidade – Semana Extra (Temporadas)

| Severidade | Quantidade | Bugs |
|------------|------------|------|
| 🔴 Crítica | 0 | - |
| 🟠 Alta | 1 | BUG-08 |
| 🟡 Média | 1 | BUG-09 |
| 🟢 Baixa | 0 | - |


## 📊 Comparativo Acumulado

| Semana | Total de Bugs | Críticos | Altos | Médios |
|--------|---------------|----------|-------|--------|
| Semana 4 (Frontend) | 3 | 0 | 3 | 0 |
| Semana 5 (API) | 4 | 0 | 4 | 0 |
| Semana Extra (Temporadas) | 2 | 0 | 1 | 1 |
| **Total Acumulado** | **9** | **0** | **8** | **1** |


> 📅 Semana Extra – Auditoria Completa (Temporadas)
>
> Bugs encontrados durante validação do CRUD de temporadas via API e interface.
>
> Total de testes executados: 8
>
> Pass: 6 | Fail: 2


