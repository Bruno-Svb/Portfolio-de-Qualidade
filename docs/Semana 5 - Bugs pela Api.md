# 🐛 Bug Reports – Semana 5 (API)

**Desafio QA 2026 – Projeto NaSalinha**


## 📊 Resumo dos Bugs Encontrados

| ID | Título | Área | CT | Severidade | Status |
|----|--------|------|-----|------------|--------|
| BUG-04 | Check-in permite envio de foto duplicada via API | Check-in | CT-09 | 🟠 Alta | Aberto |
| BUG-05 | Remoção de check-in não subtrai pontos do usuário | Pontos | CT-14 | 🟠 Alta | Aberto |
| BUG-06 | Exclusão de check-in não atualiza tabela de pontos | Pontos | CT-15 | 🟠 Alta | Aberto |
| BUG-07 | Membro não consegue visualizar ranking | Pontos | CT-17 | 🟠 Alta | Aberto |


# BUG-04 – Check-in permite envio de foto duplicada via API

| Campo | Detalhe |
|-------|---------|
| Título | API aceita a mesma foto em múltiplos check-ins do mesmo usuário |
| Localização do Erro | Backend (Service de Check-in) |
| Severidade | 🟠 Alta |
| CT Relacionado | CT-09 |


## Passos para Reproduzir

1. Fazer login com um usuário e obter token JWT
2. Enviar requisição POST para /api/checkins com uma imagem
3. Confirmar que o primeiro check-in foi criado (Status 201)
4. Enviar novamente POST para /api/checkins com a MESMA imagem
5. Verificar a resposta do servidor

## Configuração no Insomnia

Headers:
  Authorization: Bearer {{token_trainee}}

Body → Multipart Form:
  image: [mesma foto do primeiro check-in]


## Resultado Esperado

- Status Code: 400 (Bad Request)
- Mensagem: "Esta foto já foi utilizada em outro check-in"
- Sistema deve detectar foto duplicada e rejeitar o envio
- Apenas um check-in por foto deve ser permitido (conforme RF-04)


## Resultado Atual

- Status Code: 201 (Created)
- Segundo check-in é criado com sucesso
- Nenhuma validação de duplicidade de foto é aplicada
- O mesmo usuário pode enviar a mesma foto quantas vezes quiser


## Análise de Causa Raiz (RCA)

O endpoint POST /api/checkins não está verificando se a imagem já foi utilizada anteriormente pelo mesmo usuário. A validação de duplicidade de foto está ausente no service de criação de check-in.

Trecho provável do problema (backend):

// Service de check-in - CÓDIGO ATUAL (com falha)
async createCheckin(userId, imageFile) {
    // FALTA AQUI: Verificação de foto duplicada
    // const existingCheckin = await prisma.checkin.findFirst({
    //     where: {
    //         userId,
    //         imageUrl: uploadedUrl
    //     }
    // });
    // if (existingCheckin) {
    //     throw new Error('Esta foto já foi utilizada em outro check-in');
    // }
    
    const imageUrl = await this.uploadImage(imageFile);
    
    return await prisma.checkin.create({
        data: {
            userId,
            imageUrl,
            status: 'APROVADO'
        }
    });
}


## Impacto

- 📸 Usuários podem farmar pontos reenviando a mesma foto várias vezes
- 🏆 Ranking manipulado artificialmente
- 🎮 Gamificação sem integridade


# BUG-05 – Remoção de check-in não subtrai pontos do usuário

| Campo | Detalhe |
|-------|---------|
| Título | Admin remove check-in mas pontuação do usuário permanece inalterada |
| Localização do Erro | Backend (Service de Pontuação / Controller de Check-in) |
| Severidade | 🟠 Alta |
| CT Relacionado | CT-14 |


## Passos para Reproduzir

1. Verificar saldo atual do usuário no ranking
2. Fazer login como Admin e obter token JWT
3. Enviar requisição DELETE para /api/checkins/{id} com token Admin
4. Verificar se o check-in foi removido
5. Consultar novamente o ranking do usuário
6. Comparar saldo antes e depois da remoção

## Configuração no Insomnia

Headers:
  Authorization: Bearer {{token_admin}}

DELETE {{base_url}}/api/checkins/{{id_checkin_aprovado}}


## Resultado Esperado

- Status Code do DELETE: 200 (OK)
- Check-in removido do banco de dados
- Pontos do check-in subtraídos do saldo do usuário
- Saldo final = Saldo inicial - pontos do check-in removido
- Ranking atualizado automaticamente (conforme RF-08)


## Resultado Atual

- Status Code do DELETE: 200 (OK)
- Check-in é removido do banco de dados
- Pontuação do usuário NÃO é alterada
- Saldo permanece o mesmo de antes da remoção
- Ranking não reflete a exclusão


## Análise de Causa Raiz (RCA)

O service de remoção de check-in está deletando o registro da tabela CheckIn, mas não está chamando a função para subtrair os pontos correspondentes na tabela Points. A lógica de subtração de pontos (RF-08) não foi implementada ou não está sendo invocada no fluxo de exclusão.

Trecho provável do problema (backend):

// Service de check-in - CÓDIGO ATUAL (com falha)
async deleteCheckin(checkinId) {
    const checkin = await prisma.checkin.findUnique({
        where: { id: checkinId }
    });
    
    // Remove o check-in
    await prisma.checkin.delete({
        where: { id: checkinId }
    });
    
    // FALTA AQUI: Subtrair os pontos do usuário
    // await this.subtractPoints(checkin.userId, POINTS_PER_CHECKIN);
    // await this.updateRanking();
    
    return { message: 'Check-in removido' };
}


## Impacto

- 🚨 Integridade dos dados comprometida: check-in some mas pontos ficam
- 👻 Pontos fantasmas: usuários mantêm pontos de check-ins que não existem mais
- 🔓 RF-08 completamente violado: sistema deveria subtrair pontos ao remover check-in
- 📊 Ranking inconsistente com a realidade


# BUG-06 – Exclusão de check-in não atualiza tabela de pontos

| Campo | Detalhe |
|-------|---------|
| Título | Registro de pontos permanece na tabela Points após exclusão do check-in |
| Localização do Erro | Backend (Service de Pontuação) / Banco de Dados |
| Severidade | 🟠 Alta |
| CT Relacionado | CT-15 |


## Passos para Reproduzir

1. Identificar um check-in aprovado com pontos já contabilizados
2. Consultar a tabela Points no banco de dados antes da exclusão
3. Remover o check-in via API (DELETE /api/checkins/{id})
4. Consultar novamente a tabela Points no banco de dados
5. Verificar se o registro de pontos ainda existe


## Resultado Esperado

- Registro de pontos correspondente ao check-in removido deve ser DELETADO
- Tabela Points deve refletir apenas check-ins existentes
- Dados persistentes e consistentes entre as tabelas CheckIn e Points
- Saldo do usuário no ranking deve diminuir (conforme RF-08)


## Resultado Atual

- Check-in é removido da tabela CheckIn
- Registro de pontos correspondente PERMANECE na tabela Points
- Inconsistência entre as tabelas: check-in não existe mais, mas pontos sim
- Ranking continua somando pontos de check-in já excluído


## Análise de Causa Raiz (RCA)

Mesmo problema do BUG-05, visto pela perspectiva do banco de dados. A função de exclusão remove apenas o registro da tabela CheckIn, mas não remove o registro correspondente na tabela Points. O cascade delete pode não estar configurado no schema do Prisma, ou a lógica de remoção em cascata não foi implementada manualmente.

Verificação no schema do Prisma:

model CheckIn {
  id        String   @id @default(uuid())
  userId    String
  imageUrl  String
  status    String   @default("APROVADO")
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // FALTA: Relacionamento com Points para cascade delete
  // points    Points[]  @onDelete(Cascade)
}

model Points {
  id        String   @id @default(uuid())
  userId    String
  seasonId  String
  points    Int
  checkinId String   // FALTA: Referência ao check-in
  createdAt DateTime @default(now())
}


## Impacto

- 🗄️ Inconsistência no banco de dados: dados órfãos na tabela Points
- 📊 Relatórios de pontuação incorretos
- 🔓 RF-08 violado: sistema não mantém consistência de saldo
- 🏆 Ranking permanentemente poluído com pontos fantasmas


# BUG-07 – Membro não consegue visualizar ranking

| Campo | Detalhe |
|-------|---------|
| Título | Usuário com role MEMBRO não tem acesso ao endpoint de ranking |
| Localização do Erro | Backend (Middleware de autorização / RBAC) |
| Severidade | 🟠 Alta |
| CT Relacionado | CT-17 |


## Passos para Reproduzir

1. Fazer login com um usuário Membro e obter token JWT
2. Enviar requisição GET para /api/rankings com token do Membro
3. Verificar a resposta do servidor
4. Repetir o teste com um usuário Trainee
5. Repetir o teste com um usuário Admin

## Configuração no Insomnia/Postman

Headers:
  Authorization: Bearer {{token_membro}}

GET {{base_url}}/api/rankings


## Resultado Esperado

- Membro: Status Code 200 (OK) com lista do ranking
- Admin: Status Code 200 (OK) com lista do ranking
- Trainee: Status Code 401 (Unauthorized) - não pode ver ranking
- Apenas Admins e Membros devem ter acesso ao ranking


## Resultado Atual

- Membro: Status Code 401 (Unauthorized)
- Admin: Status Code 200 (OK)
- Trainee: Status Code 401 (Unauthorized)
- Apenas Admin consegue ver o ranking
- Membros estão sendo tratados como Trainees na verificação de permissão


## Análise de Causa Raiz (RCA)

O middleware de autorização (RBAC) está verificando as roles de forma incorreta. Provavelmente está permitindo apenas `role = 'ADMIN'` quando deveria permitir `role IN ('ADMIN', 'MEMBRO')`. Ou a role MEMBRO não está sendo incluída na lista de permissões do endpoint de ranking.

Trecho provável do problema (backend):

// Middleware de autorização - CÓDIGO ATUAL (com falha)
function authorize(...allowedRoles) {
    return (req, res, next) => {
        const userRole = req.user.role;
        
        // ERRO: Permitindo apenas ADMIN, esquecendo MEMBRO
        if (!allowedRoles.includes(userRole)) {
            return res.status(403).json({ error: 'Acesso negado' });
        }
        
        next();
    };
}

// Na rota de ranking - CÓDIGO ATUAL (com falha)
router.get('/rankings', authorize('ADMIN'), rankingController.getAll);
//                                           ❌ Deveria ser: 'ADMIN', 'MEMBRO'

// CÓDIGO CORRETO esperado:
router.get('/rankings', authorize('ADMIN', 'MEMBRO'), rankingController.getAll);
//                                           ✅ Correto


## Impacto

- 👥 Membros não conseguem ver o ranking, apenas Admin
- 🏆 Funcionalidade principal de gamificação quebrada para o nível Membro
- 🔓 RF-03 e RF-09 parcialmente violados: controle de acesso incorreto
- 😕 Experiência do usuário Membro prejudicada


## 📎 Distribuição de Severidade - Semana 5

| Severidade | Quantidade | Bugs |
|------------|------------|------|
| 🔴 Crítica | 0 | - |
| 🟠 Alta | 4 | BUG-05, BUG-06, BUG-04, BUG-07 |
| 🟡 Média | 0 | - |
| 🟢 Baixa | 0 | - |


## 📊 Comparativo Semana 4 vs Semana 5

| Semana | Total de Bugs | Críticos | Altos |
|--------|---------------|----------|-------|
| Semana 4 (Frontend) | 3 | 0 | 3 |
| Semana 5 (API) | 4 | 0 | 4 |
| **Total Acumulado** | **7** | **0** | **7** |


> 📅 Semana 5 – Testes de API e Integração
> 
> Bugs encontrados durante validação de endpoints e regras de negócio no backend.
> 
> Total de testes executados: 6
> 
> Pass: 2 | Fail: 4