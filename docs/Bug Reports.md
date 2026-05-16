# Relatório de Bug Reports - Projeto NaSalinha
**Desafio QA 2026**

**Autor:** Bruno Santos Vilas Boas

**Escopo da Auditoria:** Autenticação JWT, Check-in por Foto e Sistema de Pontos

---

### Bug Report 1

**Título:** Permissão de Login para Contas com E-mail Não Verificado (Status Pendente)

**Localização do Erro:** Backend (API / Service de Autenticação) e Frontend (Tela de Login)

**Passos para reproduzir:** 
> 1. Realizar o cadastro de um novo usuário informando Nome, E-mail e Senha válidos.
> 2. Sem realizar a verificação do e-mail (mantendo o status da conta como PENDENTE), acessar a tela de login ou disparar um POST para `/login`.
> 3. Fornecer as credenciais recém-criadas e clicar em "Entrar" ou enviar a requisição via Insomnia.

**Resultado esperado:** O sistema deveria bloquear o acesso, exibir a mensagem "Por favor, verifique sua conta para continuar" e não gerar nenhum token JWT (Status Code 403 Forbidden).

**Resultado atual:** O usuário consegue logar com sucesso na plataforma e a API retorna um token JWT válido mesmo com a conta pendente de verificação.

**Análise de Causa Raiz (RCA):** Falta de uma cláusula condicional no método de login do `AuthController` ou `AuthService` que verifique se o status do usuário é igual a `VERIFICADO` antes de assinar e emitir o token JWT.

**Severidade:** Alta

---

### Bug Report 2

**Título:** Atribuição Automática e Imediata de Pontos sem Estado Pendente no Check-in

**Localização do Erro:** Backend (Regra de Negócio / Check-in Service)

**Passos para reproduzir:** 
> 1. Autenticar-se no sistema com uma conta de usuário comum.
> 2. Acessar a área de check-in e fazer o upload de uma imagem válida.
> 3. Enviar a requisição e consultar imediatamente o saldo de pontos ou o status do check-in criado.

**Resultado esperado:** O check-in deveria ser criado com o status inicial explicitamente definido como PENDENTE para auditoria do administrador, sem computar pontos prematuramente.

**Resultado atual:** O check-in pula o estágio de moderação, sendo salvo diretamente com status ativo/aprovado, computando os pontos instantaneamente.

**Análise de Causa Raiz (RCA):** O valor padrão (*default*) para o status de novos registros na tabela de check-ins está configurado incorretamente no modelo ou no banco de dados como aprovado, ou o método de criação injeta o status final sem passar pela fila de moderação.

**Severidade:** Média

---

### Bug Report 3

**Título:** Permissão de Múltiplos Check-ins Consecutivos no Mesmo Dia

**Localização do Erro:** Backend (API / Validação de Check-in)

**Passos para reproduzir:** 
> 1. Efetuar o login com uma conta de usuário ativa.
> 2. Realizar um check-in enviando uma imagem válida.
> 3. Permanecer na mesma página ou reenviar imediatamente uma nova requisição de check-in (POST para `/checkin`) segundos após a primeira.

**Resultado esperado:** O sistema deveria bloquear o envio subsequente, exibindo o erro "Apenas um check-in permitido por dia" (Status Code 400 Bad Request).

**Resultado atual:** O sistema aceita múltiplos envios em sequência, gerando vários registros de check-in para a mesma data.

**Análise de Causa Raiz (RCA):** Ausência de uma consulta de verificação (*guard clause*) no Controller ou Service de check-in que filtre se o usuário logado já possui algum registro inserido no dia atual antes de prosseguir com a persistência.

**Severidade:** Alta

---

### Bug Report 4

**Título:** Ausência de Validação de Duplicidade para o Upload do Mesmo Arquivo de Foto

**Localização do Erro:** Backend (API / Validação de Mídia)

**Passos para reproduzir:** 
> 1. Realizar o upload de uma foto para validar um check-in.
> 2. Tentar realizar um novo check-in fazendo o upload do exato mesmo arquivo de imagem (mesmo nome, tamanho ou hash).

**Resultado esperado:** A API deveria rejeitar o arquivo duplicado ou apontar que a mídia já foi utilizada como prova em um check-in.

**Resultado atual:** O sistema aceita o arquivo repetido perfeitamente e cria novos registros idênticos sem nenhuma restrição.

**Análise de Causa Raiz (RCA):** Falta de uma validação de integridade ou verificação de duplicidade de arquivo (como validação por MD5 checksum ou hash binário da imagem) no serviço de upload de mídia.

**Severidade:** Média

---

### Bug Report 5

**Título:** Quebra do Fluxo de Aprovação Obrigatória por Usuários Administradores

**Localização do Erro:** Backend (Controle de Acesso / RBAC)

**Passos para reproduzir:** 
> 1. Acessar a plataforma como usuário comum e realizar um check-in.
> 2. Verificar o saldo de pontuação no ranking global sem qualquer intervenção de uma conta moderadora.

**Resultado esperado:** Os pontos só deveriam aparecer no saldo do usuário e refletir no ranking após a validação e aprovação manual feita por um perfil com nível de acesso ADMINISTRADOR.

**Resultado atual:** O sistema ignora a necessidade de aprovação pelo administrador, aplicando as regras de ganho de forma irrestrita.

**Análise de Causa Raiz (RCA):** Falha de lógica na arquitetura de negócios do sistema, onde o gatilho de somar pontos está atrelado diretamente ao evento de criação do registro de check-in, em vez de estar vinculado estritamente ao evento de alteração de status efetuado pelo Admin.

**Severidade:** Alta

---

### Bug Report 6

**Título:** Interface Permite Reenvio de Novo Check-in Imediatamente Após a Confirmação

**Localização do Erro:** Frontend (Comportamento dos Componentes da Tela)

**Passos para reproduzir:** > 1. Acessar a tela de check-in pelo navegador.
> 2. Selecionar uma foto válida e clicar no botão "Enviar".
> 3. Aguardar a mensagem de confirmação de sucesso na tela.
> 4. Tentar selecionar uma nova foto e clicar em "Enviar" imediatamente na mesma sessão.

**Resultado esperado:** Após a exibição da mensagem de sucesso, a interface deveria bloquear, ocultar ou desabilitar o formulário de envio de mídia para evitar que o usuário realize múltiplos check-ins consecutivos.

**Resultado atual:** O sistema exibe a confirmação de sucesso, mas mantém o formulário e o botão de envio totalmente ativos e liberados, permitindo que o usuário envie um novo check-in logo em seguida sem qualquer restrição visual.

**Análise de Causa Raiz (RCA):** O componente do Frontend não altera seu estado (*state*) para "desabilitado" ou "enviado" após o retorno positivo da API, falhando em congelar o formulário ou redirecionar o usuário para a tela de ranking/perfil.

**Severidade:** Alta

---

### Bug Report 7

**Título:** Contabilização Prematura de Pontos no Ranking sem Validação de Moderação

**Localização do Erro:** Backend (Sistema de Pontos e Ranking)

**Passos para reproduzir:** 
> 1. Criar um check-in de testes pela API ou interface.
> 2. Consultar o endpoint de pontuação (`GET /user/points`) ou a página de classificação geral.

**Resultado esperado:** O saldo de pontos do usuário e sua classificação deveriam permanecer inalterados até que o check-in correspondente saísse do estado pendente.

**Resultado atual:** Os pontos entram na contagem geral imediatamente após o envio, deturpando a confiabilidade do ranking de usuários.

**Análise de Causa Raiz (RCA):** A query de agregação ou a procedure que calcula o ranking soma todos os check-ins vinculados ao usuário sem filtrar pela cláusula condicional `WHERE status = 'APROVADO'`.

**Severidade:** Alta

---

### Bug Report 8

**Título:** Inconsistência de Saldo – Pontuação Não É Subtraída após a Exclusão de um Check-in

**Localização do Erro:** Backend (Service de Pontuação / Banco de Dados)

**Passos para reproduzir:** 
> 1. Autenticar-se como Administrador e efetuar a exclusão de um check-in que já havia computado pontos ao usuário (DELETE para `/checkin/{id}`).
> 2. Autenticar-se com o usuário afetado e consultar a pontuação atualizada (`GET /user/points`) ou o ranking.

**Resultado esperado:** A pontuação associada àquele check-in excluído deveria ser decrementada do saldo total do usuário de forma consistente.

**Resultado atual:** O check-in é deletado do banco de dados, mas o saldo de pontos do usuário não diminui, gerando pontos fantasmas acumulados.

**Análise de Causa Raiz (RCA):** O método de deleção remove o registro da tabela de check-ins, mas não dispara o recálculo do saldo do usuário, mantendo os pontos de forma estática no perfil do usuário sem integridade referencial vinculada às transações existentes.

**Severidade:** Crítica

---

### Bug Report 9

**Título:** Burlar Restrição de Tempo – Envio de Check-in Permitido antes do Intervalo de 24 Horas

**Localização do Erro:** Backend (Regra de Negócio / API)

**Passos para reproduzir:** 
> 1. Disparar um POST para `/checkin` contendo uma foto válida e registrar o horário do envio.
> 2. Aguardar algumas poucas horas (menos do que o ciclo completo de 24 horas requerido).
> 3. Tentar submeter um novo check-in.

**Resultado esperado:** A API deve rejeitar o envio exibindo Status Code 400 (Too Many Requests) ou 400, bloqueando a criação até o término do intervalo regulamentar.

**Resultado atual:** O sistema ignora a janela temporal exigida por completo e processa o novo check-in com sucesso.

**Análise de Causa Raiz (RCA):** A lógica de controle de data analisa apenas se o dia civil mudou ou carece totalmente de uma comparação matemática detalhada de timestamps (subtração entre o horário atual e o horário do último check-in para verificar se o resultado é menor que 86400 segundos).

**Severidade:** Alta