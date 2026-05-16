# Plano de Testes - Projeto NaSalinha

## Módulo 1: Autenticação e Acesso (JWT)

### RF-01: Cadastro e Verificação

#### Caso 1 – Funcional (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Cadastro de usuário |
| Cenário | Registrar novo usuário com dados válidos e receber código de verificação |
| Tipo | Positivo |
| Passos | > 1. Acessar tela de cadastro<br>2. Preencher Nome, E-mail e Senha válidos<br>3. Clicar em "Cadastrar" |
| Resultado esperado | > Sistema exibe mensagem de sucesso. Código de verificação enviado ao e-mail. Status da conta: PENDENTE. |

#### Caso 2 – API (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Cadastro de usuário via API |
| Cenário | Tentar cadastrar com e-mail já existente |
| Tipo | Negativo |
| Passos | > 1. Enviar POST `/register` com e-mail já cadastrado<br>2. Body: `{"nome":"Teste","email":"existente@email.com","senha":"123456"}` |
| Resultado esperado | > API retorna status 400 com mensagem: "E-mail já registrado". |

---

### RF-02: Restrição de Login

#### Caso 1 – Funcional (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Login de usuário |
| Cenário | Tentar fazer login com conta não verificada (status PENDENTE) |
| Tipo | Negativo |
| Passos | > 1. Cadastrar usuário, mas não inserir código de verificação<br>2. Acessar tela de login<br>3. Inserir e-mail e senha corretos e clicar em "Entrar" |
| Resultado esperado | > Sistema exibe mensagem: "Por favor, verifique sua conta para continuar". Login não é realizado. |

#### Caso 2 – API (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Login via API |
| Cenário | Enviar credenciais de conta não verificada |
| Tipo | Negativo |
| Passos | > 1. Criar usuário via API sem verificar código<br>2. Enviar POST `/login` com e-mail e senha corretos |
| Resultado esperado | > API retorna status 403 com mensagem: "Por favor, verifique sua conta para continuar". Nenhum token JWT é gerado. |

---

### RF-03: Controle de Níveis (RBAC)

#### Caso 1 – Funcional (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Acesso a área administrativa |
| Cenário | Usuário comum tenta acessar página de gestão de temporadas |
| Tipo | Negativo |
| Passos | > 1. Fazer login como usuário comum<br>2. Tentar acessar URL `/admin/temporadas` diretamente<br>3. Tentar clicar em botão de moderação (se visível) |
| Resultado esperado | > Sistema bloqueia acesso e redireciona para página inicial ou exibe "Acesso negado". |

#### Caso 2 – API (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Endpoint administrativo via API |
| Cenário | Administrador acessa endpoint de criação de temporada |
| Tipo | Positivo |
| Passos | > 1. Autenticar como ADMINISTRADOR, obter token JWT<br>2. Enviar POST `/admin/temporadas` com dados válidos<br>3. Header: `Authorization: Bearer <token>` |
| Resultado esperado | > API retorna status 201 (Created). Temporada criada com sucesso. |

---

## Módulo 2: Check-in (Mídia)

### RF-04: Registro de Atividade

#### Caso 1 – Funcional (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Realizar check-in |
| Cenário | Usuário autenticado envia imagem válida, sem check-in no dia |
| Tipo | Positivo |
| Passos | > 1. Fazer login<br>2. Acessar área de check-in<br>3. Selecionar imagem e clicar em "Enviar" |
| Resultado esperado | > Check-in registrado com status PENDENTE. Mensagem: "Check-in realizado com sucesso". |

#### Caso 2 – API (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Envio de check-in via API |
| Cenário | Tentar enviar segunda imagem no mesmo dia |
| Tipo | Negativo |
| Passos | > 1. Autenticar e enviar POST `/checkin` com imagem válida<br>2. Enviar novamente POST `/checkin` com a mesma imagem |
| Resultado esperado | > API retorna status 400 com mensagem: "Apenas um check-in permitido por dia". |

---

### RF-05: Fluxo de Status do Check-in

#### Caso 1 – Funcional (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Alteração de status por Administrador |
| Cenário | Admin aprova check-in pendente via interface |
| Tipo | Positivo |
| Passos | > 1. Admin faz login<br>2. Acessa lista de check-ins pendentes<br>3. Seleciona um check-in e clica em "Aprovar" |
| Resultado esperado | > Status do check-in muda para APROVADO. Sistema confirma ação. |

#### Caso 2 – API (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Alteração de status via API |
| Cenário | Usuário comum tenta alterar status de seu próprio check-in |
| Tipo | Negativo |
| Passos | > 1. Autenticar como usuário comum<br>2. Enviar PUT `/checkin/{id}/status` com body `{"status":"APROVADO"}` |
| Resultado esperado | > API retorna status 403 com mensagem: "Apenas administradores podem alterar status". |

---

### RF-06: Restrição de Propriedade

#### Caso 1 – Funcional (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Visualização de check-ins |
| Cenário | Usuário tenta visualizar check-in de outro usuário pela interface |
| Tipo | Negativo |
| Passos | > 1. Usuário A faz login<br>2. Usuário A tenta acessar URL `/checkin/123` (check-in do Usuário B)<br>3. Usuário A tenta editar/deletar esse check-in |
| Resultado esperado | > Sistema exibe mensagem "Acesso negado" ou redireciona. Nenhuma informação do check-in de outro usuário é mostrada. |

#### Caso 2 – API (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Deleção de check-in via API |
| Cenário | Usuário tenta deletar check-in que não lhe pertence |
| Tipo | Negativo |
| Passos | > 1. Autenticar como Usuário A<br>2. Enviar DELETE `/checkin/456` (check-in do Usuário B) |
| Resultado esperado | > API retorna status 403 ou 404 com mensagem: "Você não tem permissão para acessar este recurso". |

---

## Módulo 3: Pontuação e Ranking

### RF-07: Atribuição de Pontos

#### Caso 1 – Funcional (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Atribuição de pontos ao aprovar check-in |
| Cenário | Admin aprova check-in e pontos são somados ao ranking |
| Tipo | Positivo |
| Passos | > 1. Usuário faz check-in (status PENDENTE)<br>2. Admin aprova o check-in<br>3. Usuário acessa página de ranking |
| Resultado esperado | > Pontuação do usuário aumenta. Ranking é atualizado automaticamente. |

#### Caso 2 – API (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Verificação de pontos via API |
| Cenário | Consultar pontuação do usuário após check-in aprovado |
| Tipo | Positivo |
| Passos | > 1. Realizar check-in como usuário<br>2. Admin aprova via API (PUT `/checkin/{id}/approve`)<br>3. Usuário envia GET `/user/points` |
| Resultado esperado | > API retorna status 200 com body contendo pontuação atualizada corretamente. |

---

### RF-08: Consistência de Saldo

#### Caso 1 – Funcional (Negativo / Validação)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Subtração de pontos ao remover check-in |
| Cenário | Admin remove check-in aprovado e pontos são subtraídos |
| Tipo | Negativo (validação de perda de pontos) |
| Passos | > 1. Usuário com check-in aprovado (+10 pontos)<br>2. Admin remove o check-in<br>3. Usuário consulta ranking novamente |
| Resultado esperado | > Pontos são subtraídos do saldo total. Ranking reflete nova pontuação. |

#### Caso 2 – API (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Consistência de saldo via API |
| Cenário | Remover check-in por API e verificar recálculo |
| Tipo | Positivo |
| Passos | > 1. Autenticar como Admin<br>2. Enviar DELETE `/checkin/789` (check-in aprovado)<br>3. Autenticar como usuário dono do check-in e enviar GET `/user/points` |
| Resultado esperado | > API retorna pontuação reduzida. Saldo consistente com os check-ins restantes. |

---

### RF-09: Regras de Ranking

#### Caso 1 – Funcional (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Ordenação do ranking |
| Cenário | Múltiplos usuários com pontuações diferentes |
| Tipo | Positivo |
| Passos | > 1. Criar Usuário A com 50 pontos<br>2. Criar Usuário B com 100 pontos<br>3. Criar Usuário C com 75 pontos<br>4. Acessar página de ranking |
| Resultado esperado | > Ranking exibido na ordem: Usuário B (100), Usuário C (75), Usuário A (50). |

#### Caso 2 – API (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Restrição de tempo entre check-ins |
| Cenário | Tentar segundo check-in antes de completar 24h |
| Tipo | Negativo |
| Passos | > 1. Enviar POST `/checkin` com imagem válida às 10:00<br>2. Enviar novo POST `/checkin` às 09:59 do dia seguinte (<24h)<br>3. Aguardar até 10:01 e tentar novamente |
| Resultado esperado | > Primeira tentativa após 24h: retorna status 429 com erro.<br>Segunda tentativa (após 24h): retorna status 201 (sucesso). |

---