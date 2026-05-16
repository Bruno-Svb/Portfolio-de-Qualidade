## 1. Casos Teste relacionados à Temporadas

#### Caso 1 – API (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Criar corretamente nova temporada |
| Tipo | Positivo |
| Passos | > 1. Autenticar no sistema com uma conta autorizada<br>2. Enviar requisição POST para o endpoint de criação de temporada com dados válidos e formatação de data correta |
| Resultado esperado | > API retorna status 201 (Created), o registro é salvo no banco de dados e os dados da nova temporada são retornados no body. |

#### Caso 2 – API (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Criar nova temporada com formatação incorreta de data |
| Tipo | Negativo |
| Passos | > 1. Enviar requisição POST para o endpoint de criação de temporada<br>2. Incluir no body uma data em formato inválido (ex: DD/MM/AAAA em vez de AAAA-MM-DD) |
| Resultado esperado | > API retorna status 400 (Bad Request) com uma mensagem de erro validando a formatação da data. |

#### Caso 3 – API / Permissão (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Deletar temporada sendo Administrador (Admin) |
| Tipo | Positivo |
| Passos | > 1. Autenticar no sistema com uma conta de perfil "Admin"<br>2. Enviar requisição DELETE informando o ID da temporada no endpoint correspondente |
| Resultado esperado | > API retorna status 200 (OK) ou 204 (No Content) e a temporada é removida/inativada com sucesso. |

#### Caso 4 – API / Permissão (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Tentar deletar temporada sendo Membro |
| Tipo | Negativo |
| Passos | > 1. Autenticar no sistema com uma conta de perfil "Membro"<br>2. Enviar requisição DELETE informando o ID da temporada |
| Resultado esperado | > API retorna status 403 (Forbidden) com a mensagem de que o usuário não tem permissão para esta ação. |

#### Caso 5 – API / Permissão (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Alterar nome da temporada sendo Administrador (Admin) |
| Tipo | Positivo |
| Passos | > 1. Autenticar no sistema com uma conta de perfil "Admin"<br>2. Enviar requisição PUT para o endpoint da temporada com o novo nome no body |
| Resultado esperado | > API retorna status 200 (OK) e o nome da temporada é atualizado com sucesso no sistema. |

#### Caso 6 – API / Permissão (Negativo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Tentar alterar nome da temporada sendo Membro |
| Tipo | Negativo |
| Passos | > 1. Autenticar no sistema com uma conta de perfil "Membro"<br>2. Enviar requisição PUT para o endpoint da temporada tentando alterar o nome |
| Resultado esperado | > API retorna status 403 (Forbidden), bloqueando a alteração por falta de privilégios. |

#### Caso 7 – API (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Editar dados da temporada (PATCH) |
| Tipo | Positivo |
| Passos | > 1. Enviar requisição PATCH para o endpoint da temporada<br>2. Enviar no body apenas os campos específicos que necessitam de atualização parcial |
| Resultado esperado | > API retorna status 200 (OK) modificando apenas os campos enviados, mantendo o restante dos dados intactos. |

#### Caso 8 – API (Positivo)

| Campo | Descrição |
|-------|------------|
| Funcionalidade | Gerenciamento de Temporadas |
| Cenário | Consultar dados de uma temporada (GET) |
| Tipo | Positivo |
| Passos | > 1. Enviar requisição GET informando o ID da temporada ou os parâmetros de busca no endpoint correspondente |
| Resultado esperado | > API retorna status 200 (OK) contendo as informações detalhadas da temporada solicitada no body da resposta. |
# Relatório de Bug Reports - Módulo Extra: Temporadas
**Desafio QA 2026** **Escopo da Auditoria:** Módulo Diferencial (CRUD Completo de Temporadas)

Este documento centraliza de forma isolada o plano de testes executado, o detalhamento das falhas encontradas e o fechamento métrico para a funcionalidade opcional de gerenciamento de Temporadas.

---

## 1. Resumo Executivo (Métricas Gerais - Temporadas)

A tabela abaixo consolida os cenários de testes executados especificamente no fluxo de gerenciamento de temporadas, avaliando o comportamento das requisições via API e a integridade da interface.

| Módulo Ad-hoc / Funcionalidade | Testes Planejados | Testes Executados | Testes Passados (Pass) | Testes Falhados (Fail) | Total de Bugs Encontrados |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Módulo Extra: Temporadas** | 8 | 8 | 6 | 2 | **2** |

* **Descrição e Análise:** Foram executados oito cenários de teste Ad-hoc para validar a robustez das validações de regras de negócios de tempo e concorrência. Duas das validações falharam na camada do servidor, resultando em uma taxa de erro de **25%** nos pontos auditados deste módulo específico.

---

## 2. Detalhamento dos Bug Reports

### Bug Report 10

**Título:** Data Inválida na Criação de Temporada Retorna Ano 1970 via API

**Localização do Erro:** Backend (Service de Temporadas / Validação de Entrada)
**Passos para reproduzir:** 
> 1. Autenticar-se no sistema com uma conta que possua perfil de Administrador e obter o Token JWT.
> 2. Configurar uma requisição POST para o endpoint `/api/seasons` preenchendo o corpo da mensagem com uma string inválida ou em branco no parâmetro de data (ex: `"startDate": "data-invalida"` ou `"startDate": ""`).
> 3. Disparar a requisição através do Insomnia.
> 4. Realizar uma requisição de consulta à temporada criada para checar a persistência do dado.

**Resultado esperado:** A API deveria recusar a criação do registro, retornando um Status Code 400 Bad Request com uma mensagem explicativa explicitando o formato correto da data.

**Resultado atual:** O servidor aceita a payload inválida com sucesso (Status Code 201), gerando o registro no banco de dados com a data convertida para o timestamp zero da época Unix (`01/01/1970`).

**Análise de Causa Raiz (RCA):** Ausência de um validador de formato de string de data (como regex ou parse estrito de instância de `Date`) no controller/dto. O interpretador de backend realiza o casting forçado de um valor inválido, o que resulta na data inicial padrão do sistema (Unix Epoch).

**Severidade:** Alta

---

### Bug Report 11

**Título:** Multiplicidade de Temporadas Ativas Simultaneamente no Frontend via Criação por API

**Localização do Erro:** Backend (Lógica de Filtro / Regra de Negócio) e Frontend (Exibição Visual)
**Passos para reproduzir:** 
> 1. Certificar-se de que já existe uma temporada cadastrada e ativa no sistema.
> 2. Disparar um POST via Insomnia criando uma segunda temporada cujo período de vigência (`startDate` e `endDate`) englobe o dia atual.
> 3. Efetuar o login no sistema pelo navegador (Frontend) e acessar o painel principal de visualização de temporadas.

**Resultado esperado:** O sistema deveria possuir uma trava de concorrência onde apenas uma única temporada pudesse figurar com o status ativa por vez. Caso uma nova temporada em período vigente fosse criada, a anterior deveria ser desativada automaticamente.

**Resultado atual:** A interface do usuário exibe duas temporadas concomitantemente ativas na tela, gerando ambiguidade e quebrando as diretrizes de integridade visual.

**Análise de Causa Raiz (RCA):** No componente do Frontend ou na rota do banco de dados, a checagem das temporadas ativas está sendo feita erroneamente por meio de um filtro temporal estático de período (`new Date(s.startDate) <= new Date() && new Date(s.endDate) >= new Date()`) em vez de validar a propriedade booleana real de ativação (`s.isActive === true`).

**Severidade:** Média

---

## 3. Distribuição de Bugs por Severidade (Temporadas)

| ID do Bug | Título Resumido do Erro | Localização | Severidade |
| :---: | :--- | :--- | :---: |
| **BR-10** | API aceita data inválida e salva como data época Unix (1970) | Backend / API | **Alta** |
| **BR-11** | Duas temporadas aparecem como ativas no frontend simultaneamente | Backend / Frontend | **Média** |

---

## 4. Consolidado por Nível de Severidade

| Severidade | Quantidade | Impacto Prático no Módulo de Temporadas |
| :---: | :---: | :--- |
| ⚠️ **Crítica** | 0 | Nenhum bug bloqueante ou de corrupção total de tabelas persistidas foi identificado neste lote. |
| 🛑 **Alta** | 1 | **Falha de Sanitização de Dados:** Permite a gravação de dados corrompidos temporais de forma permanente nas tabelas do sistema (BR-10). |
| 🟡 **Média** | 1 | **Falha de Regra Visual/Lógica:** Gera inconsistência na interface e ambiguidade para o usuário final sobre qual temporada está contabilizando os pontos acumulados (BR-11). |
| 🟢 **Baixa** | 0 | Sem ocorrências estéticas puras registradas. |