# 📋 Casos de Teste – Semana 3

## Desafio QA 2026 – Projeto NaSalinha


## 📊 Resumo Geral

| Total de Casos de Teste | 16 |
|--------------------------|----|
| Áreas Cobertas           | Autenticação JWT, Check-in por Foto, Sistema de Pontos |
| Tipos de Teste           | Funcional, API, Regressão |
| Cenários                 | Positivos e Negativos |


## 🗂️ Estrutura do Board no Trello

| Lista | Descrição |
|-------|-----------|
| 📋 Ready for Test | Casos de teste planejados, prontos para executar |
| 🔍 In Test | Casos de teste em execução |
| ✅ Test Done | Casos de teste concluídos |
| 🐛 Bugs Encontrados | Bugs identificados durante a execução |

### Etiquetas (Labels)

| Cor | Nome |
|-----|------|
| 🟢 | Positivo |
| 🔴 | Negativo |
| 🟡 | Funcional |
| 🔵 | API |
| 🟣 | Regressão |


# 🔐 Área 1: Autenticação JWT


## CT-01 – Cadastro com dados válidos

| Campo | Valor |
|-------|-------|
| Área | Autenticação JWT |
| Tipo | Funcional |
| Cenário | Positivo |
| Requisito | RF-01 |
| Etiquetas | 🟢 Positivo / 🟡 Funcional |

### Pré-condições
- Ambiente rodando
- Acesso à página de registro

### Passos
1. Acessar a página de cadastro
2. Preencher Nome, E-mail e Senha válidos
3. Clicar em "Cadastrar"

### Resultado Esperado
- Usuário criado com sucesso
- Status inicial da conta: PENDENTE
- E-mail de verificação enviado com código único
- Mensagem de confirmação exibida na tela

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-02 – Cadastro com e-mail já existente

| Campo | Valor |
|-------|-------|
| Área | Autenticação JWT |
| Tipo | Funcional |
| Cenário | Negativo |
| Requisito | RF-01 |
| Etiquetas | 🔴 Negativo / 🟡 Funcional |

### Pré-condições
- Usuário já cadastrado no sistema

### Passos
1. Acessar a página de cadastro
2. Preencher Nome, E-mail (já existente) e Senha
3. Clicar em "Cadastrar"

### Resultado Esperado
- Sistema deve rejeitar o cadastro
- Mensagem de erro: "E-mail já cadastrado"

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-03 – Login sem verificar e-mail

| Campo | Valor |
|-------|-------|
| Área | Autenticação JWT |
| Tipo | Funcional |
| Cenário | Negativo |
| Requisito | RF-02 |
| Etiquetas | 🔴 Negativo / 🟡 Funcional |

### Pré-condições
- Usuário cadastrado com status PENDENTE (não verificou e-mail)

### Passos
1. Acessar a página de login
2. Inserir e-mail e senha do usuário pendente
3. Clicar em "Entrar"

### Resultado Esperado
- Sistema deve bloquear o acesso
- Mensagem: "Por favor, verifique sua conta para continuar"
- Usuário NÃO deve ser redirecionado para área logada

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-04 – Registro via API sem senha

| Campo | Valor |
|-------|-------|
| Área | Autenticação JWT |
| Tipo | API |
| Cenário | Negativo |
| Requisito | RF-01 |
| Etiquetas | 🔴 Negativo / 🔵 API |

### Pré-condições
- Insomnia/Postman configurado
- Endpoint: POST /api/auth/register

### Passos
1. Enviar requisição POST para /api/auth/register
2. Body JSON:
   {
     "name": "Teste",
     "email": "teste@email.com"
   }
3. Omitir campo "password"

### Resultado Esperado
- Status Code: 400 (Bad Request)
- Corpo da resposta com mensagem de validação indicando campo obrigatório

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-05 – Login com credenciais válidas

| Campo | Valor |
|-------|-------|
| Área | Autenticação JWT |
| Tipo | API |
| Cenário | Positivo |
| Requisito | RF-01 |
| Etiquetas | 🟢 Positivo / 🔵 API |

### Pré-condições
- Usuário cadastrado e verificado no banco
- Endpoint: POST /api/auth/login

### Passos
1. Enviar requisição POST para /api/auth/login
2. Body JSON:
   {
     "email": "user@teste.com",
     "password": "senha123"
   }

### Resultado Esperado
- Status Code: 200 (OK)
- Corpo da resposta contendo token JWT e refresh token
- Token deve ser válido e decodificável

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-06 – Regressão: Corrigir bug de login com conta pendente

| Campo | Valor |
|-------|-------|
| Área | Autenticação JWT |
| Tipo | Regressão |
| Cenário | Simulação de correção |
| Requisito | RF-02 |
| Etiquetas | 🟣 Regressão |

### Contexto
Foi reportado um bug onde usuários com status PENDENTE conseguiam fazer login. A correção foi implementada. Testes de regressão necessários.

### Pré-condições
- Bug corrigido no código

### Testes a Repetir
1. CT-03 – Login sem verificar e-mail (deve continuar bloqueando)
2. CT-05 – Login com credenciais válidas (deve continuar funcionando)
3. Testar fluxo completo: Cadastro, Verificar e-mail, Login (deve funcionar)
4. Testar endpoint /api/auth/login com token inválido (deve rejeitar)
5. Verificar se endpoints protegidos continuam exigindo token válido

### Resultado Esperado
- Nenhum efeito colateral nas funcionalidades existentes
- Todos os testes acima passam

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


# 📸 Área 2: Check-in por Foto


## CT-07 – Check-in com foto válida

| Campo | Valor |
|-------|-------|
| Área | Check-in por Foto |
| Tipo | Funcional |
| Cenário | Positivo |
| Requisito | RF-04 |
| Etiquetas | 🟢 Positivo / 🟡 Funcional |

### Pré-condições
- Usuário logado (Membro ou Trainee)
- Temporada ativa configurada
- Nenhum check-in realizado no dia

### Passos
1. Acessar a página de check-in
2. Selecionar uma imagem válida (JPG/PNG)
3. Clicar em "Enviar Check-in"

### Resultado Esperado
- Check-in registrado com sucesso
- Status inicial: PENDENTE
- Upload enviado para Cloudinary
- Mensagem de confirmação exibida

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-08 – Dois check-ins no mesmo dia

| Campo | Valor |
|-------|-------|
| Área | Check-in por Foto |
| Tipo | Funcional |
| Cenário | Negativo |
| Requisito | RF-04 / RF-09 |
| Etiquetas | 🔴 Negativo / 🟡 Funcional |

### Pré-condições
- Usuário já realizou um check-in no dia atual

### Passos
1. Acessar a página de check-in
2. Selecionar uma imagem válida
3. Clicar em "Enviar Check-in"

### Resultado Esperado
- Sistema deve bloquear o check-in
- Mensagem: "Você já realizou um check-in hoje"
- Intervalo mínimo de 24h não atingido

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-09 – Check-in com mesma foto

| Campo | Valor |
|-------|-------|
| Área | Check-in por Foto |
| Tipo | API |
| Cenário | Negativo |
| Requisito | RF-04 |
| Etiquetas | 🔴 Negativo / 🔵 API |

### Pré-condições
- Usuário autenticado com token JWT
- Check-in já realizado com uma foto específica
- Endpoint: POST /api/checkins

### Passos
1. Enviar requisição POST para /api/checkins
2. Enviar a mesma imagem já utilizada em check-in anterior

### Resultado Esperado
- Status Code: 400 (Bad Request)
- Sistema deve detectar foto duplicada
- Mensagem indicando que a foto já foi enviada

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-10 – Tentar editar check-in de outro usuário

| Campo | Valor |
|-------|-------|
| Área | Check-in por Foto |
| Tipo | API |
| Cenário | Negativo |
| Requisito | RF-06 |
| Etiquetas | 🔴 Negativo / 🔵 API |

### Pré-condições
- Dois usuários cadastrados (User A e User B)
- Check-in do User A registrado
- User B autenticado com token JWT
- Endpoint: PUT /api/checkins/:id

### Passos
1. User B tenta editar o check-in do User A
2. Enviar requisição PUT para /api/checkins/[id-do-checkin-do-user-A]

### Resultado Esperado
- Status Code: 403 (Forbidden) ou 401 (Unauthorized)
- Sistema deve impedir a manipulação
- Mensagem de erro: "Acesso negado"

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-11 – Regressão: Corrigir bug de arquivo temporário não removido

| Campo | Valor |
|-------|-------|
| Área | Check-in por Foto |
| Tipo | Regressão |
| Cenário | Simulação de correção |
| Requisito | RNF-02 |
| Etiquetas | 🟣 Regressão |

### Contexto
Foi reportado um bug onde arquivos temporários de upload não estavam sendo removidos do servidor após envio para Cloudinary. A correção foi aplicada.

### Pré-condições
- Bug corrigido no código

### Testes a Repetir
1. CT-07 – Check-in com foto válida (deve continuar funcionando)
2. Verificar diretório /tmp ou pasta de uploads após check-in (deve estar vazio)
3. Simular falha no upload para Cloudinary (arquivo temporário deve ser limpo)
4. Verificar se check-ins antigos continuam acessíveis (fotos no Cloudinary)

### Resultado Esperado
- Arquivos temporários removidos após cada upload (sucesso ou falha)
- Nenhum efeito colateral nos check-ins existentes

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


# ⭐ Área 3: Sistema de Pontos e Ranking


## CT-12 – Check-in aprovado soma pontos

| Campo | Valor |
|-------|-------|
| Área | Sistema de Pontos |
| Tipo | Funcional |
| Cenário | Positivo |
| Requisito | RF-07 |
| Etiquetas | 🟢 Positivo / 🟡 Funcional |

### Pré-condições
- Usuário com check-in PENDENTE
- Admin logado para aprovar

### Passos
1. Admin acessa painel de moderação
2. Aprova o check-in pendente
3. Verifica ranking e saldo do usuário

### Resultado Esperado
- Pontos somados ao saldo do usuário
- Ranking atualizado automaticamente
- Usuário aparece na posição correta do ranking

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-13 – Ranking ordenado corretamente

| Campo | Valor |
|-------|-------|
| Área | Sistema de Pontos |
| Tipo | Funcional |
| Cenário | Positivo |
| Requisito | RF-09 |
| Etiquetas | 🟢 Positivo / 🟡 Funcional |

### Pré-condições
- Múltiplos usuários com pontuações diferentes

### Passos
1. Acessar a página de ranking
2. Verificar a ordenação dos usuários

### Resultado Esperado
- Usuários listados em ordem decrescente de pontuação
- Usuário com maior pontuação aparece no topo
- Todos os usuários com pontuação aparecem na lista

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-14 – Remoção de check-in subtrai pontos

| Campo | Valor |
|-------|-------|
| Área | Sistema de Pontos |
| Tipo | API |
| Cenário | Positivo |
| Requisito | RF-08 |
| Etiquetas | 🟢 Positivo / 🔵 API |

### Pré-condições
- Check-in aprovado e pontos já somados
- Admin autenticado
- Endpoint: DELETE /api/checkins/:id

### Passos
1. Registrar saldo atual do usuário (antes)
2. Admin remove o check-in via API
3. Verificar novo saldo do usuário (depois)

### Resultado Esperado
- Status Code: 200 (OK)
- Saldo do usuário reduzido proporcionalmente
- Ranking atualizado automaticamente
- Saldo final = Saldo inicial - pontos do check-in removido

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-15 – Verificar persistência dos pontos no banco

| Campo | Valor |
|-------|-------|
| Área | Sistema de Pontos |
| Tipo | API |
| Cenário | Positivo |
| Requisito | RF-07 |
| Etiquetas | 🟢 Positivo / 🔵 API |

### Pré-condições
- Check-in aprovado
- Endpoint: GET /api/rankings ou consulta direta ao banco

### Passos
1. Aprovar um check-in
2. Consultar tabela de Points no banco de dados
3. Verificar se o registro foi criado

### Resultado Esperado
- Registro criado na tabela Points
- user_id, season_id e pontuação corretos
- Dados persistentes após refresh da página

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## CT-16 – Regressão: Corrigir bug de pontos duplicados

| Campo | Valor |
|-------|-------|
| Área | Sistema de Pontos |
| Tipo | Regressão |
| Cenário | Simulação de correção |
| Requisito | RF-07 / RF-09 |
| Etiquetas | 🟣 Regressão |

### Contexto
Foi reportado um bug onde aprovar o mesmo check-in duas vezes resultava em pontos duplicados. A correção foi implementada.

### Pré-condições
- Bug corrigido no código

### Testes a Repetir
1. CT-12 – Check-in aprovado soma pontos (deve continuar somando corretamente)
2. CT-14 – Remoção de check-in subtrai pontos (deve subtrair apenas uma vez)
3. Tentar aprovar o mesmo check-in duas vezes (deve bloquear segunda aprovação)
4. Verificar ranking após aprovação única (soma deve aparecer apenas uma vez)
5. Verificar se a correção não afetou a aprovação de check-ins diferentes

### Resultado Esperado
- Pontos somados apenas uma vez por check-in
- Nenhum efeito colateral no fluxo normal de aprovação/rejeição

### Resultado Obtido
(Preencher durante a execução)

### Status
⬜ Ready for Test / 🔍 In Test / ✅ Pass / ❌ Fail


## 📎 Resumo para o Trello

| ID    | Nome                                             | Área     | Tipo       | Etiqueta  |
|-------|--------------------------------------------------|----------|------------|-----------|
| CT-01 | Cadastro com dados válidos                       | JWT      | Funcional  | 🟢🟡      |
| CT-02 | Cadastro com e-mail já existente                 | JWT      | Funcional  | 🔴🟡      |
| CT-03 | Login sem verificar e-mail                       | JWT      | Funcional  | 🔴🟡      |
| CT-04 | Registro via API sem senha                       | JWT      | API        | 🔴🔵      |
| CT-05 | Login com credenciais válidas                    | JWT      | API        | 🟢🔵      |
| CT-06 | Regressão: Bug login conta pendente              | JWT      | Regressão  | 🟣        |
| CT-07 | Check-in com foto válida                         | Check-in | Funcional  | 🟢🟡      |
| CT-08 | Dois check-ins no mesmo dia                      | Check-in | Funcional  | 🔴🟡      |
| CT-09 | Check-in com mesma foto                          | Check-in | API        | 🔴🔵      |
| CT-10 | Editar check-in de outro usuário                 | Check-in | API        | 🔴🔵      |
| CT-11 | Regressão: Arquivo temporário não removido       | Check-in | Regressão  | 🟣        |
| CT-12 | Check-in aprovado soma pontos                    | Pontos   | Funcional  | 🟢🟡      |
| CT-13 | Ranking ordenado corretamente                    | Pontos   | Funcional  | 🟢🟡      |
| CT-14 | Remoção de check-in subtrai pontos               | Pontos   | API        | 🟢🔵      |
| CT-15 | Persistência dos pontos no banco                 | Pontos   | API        | 🟢🔵      |
| CT-16 | Regressão: Bug de pontos duplicados              | Pontos   | Regressão  | 🟣        |
