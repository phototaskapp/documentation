**Documento 0:**

**Passos Detalhados de Multi-Negócio e Estado Atual do Desenvolvimento: (Atualizado até 06/04/2025, 09:18h)**

**Ajustar Modelo de Dados**

- **Já Realizado:**
    - Tabela `negocios` criada com colunas: id (uuid, PK), nome (text, NOT NULL), usuario_id (uuid, FK users.id), data_criacao (timestamp), designar_dono (boolean), data_backup (timestamp), acesso_modulos (TEXT[], ajustado de text em Etapa 2), regiao (text) (Etapa 1, 16/03/2025; ajustado 19/03/2025, 09:00h).
    - Tabela `assinaturas` criada com colunas: id (uuid, PK), usuario_id (uuid, FK users.id), plano (text), data_inicio (timestamp), data_fim (timestamp), trial_ativo (boolean), cancelado (boolean), negocio_id (uuid, FK negocios.id), stripe_customer_id (text), stripe_subscription_id (text), ultimo_pagamento (timestamp), status_pagamento (text), data_atualizacao (timestamp) (Etapa 1, 16/03/2025).
    - Tabela `users` ajustada com `negocio_id` (uuid, FK negocios.id) para o negócio inicial criado no signup (Etapa 1, 16/03/2025).
    - Tabela `negocios_membros` criada com colunas: id (uuid, PK, gen_random_uuid()), negocio_id (uuid, FK negocios.id, obrigatório), usuario_id (uuid, FK users.id, opcional), is_proprietario (boolean, default false), acesso_modulos (TEXT[], default ARRAY['total']), data_associacao (timestamp, default NOW()), plano_atual (text, calculado via herança do proprietário), nome (text), status (text), invite_code (text, UNIQUE) (Etapa 2, 19/03/2025, 09:00h; campos adicionais em 20/03/2025).
    - Índice `idx_negocios_membros_negocio_id_usuario_id` (UNIQUE) criado para otimizar consultas por negocio_id e usuario_id (Etapa 2, 19/03/2025, 09:00h).
    - Tabela `postagens_marketing` ajustada com adição de `negocio_id` (uuid, FK negocios.id) para suportar multi-negócios diretamente, preenchido via `projetos_marketing` e mantido por trigger (Etapa 2, 19/03/2025, 09:00h).
    - Removida a tabela `convites_pendentes`, substituída por lógica direta em `negocios_membros` para gerenciar convites pendentes (20/03/2025).
    - Removida a restrição de NOT NULL na coluna `usuario_id` da tabela `negocios_membros` para permitir convites pendentes com usuario_id=null (21/03/2025).
    - Adicionada a constraint `check_usuario_id_for_associado` na tabela `negocios_membros` para garantir que `usuario_id` seja preenchido para registros com status 'associado' (21/03/2025).
    - Ativado RLS para a tabela `negocios_membros` com as políticas:
        - `allow_negocio_owners_to_insert_negocios_membros` (papel authenticated).
        - `allow_negocio_owners_to_select_negocios_membros` (papel authenticated) - *Ajustada em 27/03/2025:*
            
            **SQL**
            
            `DROP POLICY IF EXISTS "allow_authenticated_select_negocios_membros" ON public.negocios_membros;
            CREATE POLICY "allow_authenticated_select_negocios_membros" ON public.negocios_membros
            FOR SELECT
            TO authenticated
            USING (
              negocio_id IN (
                SELECT negocio_id FROM negocios_membros WHERE usuario_id = auth.uid()
              ) OR
              usuario_id = auth.uid()
            );`
            
        - `allow_negocio_owners_to_update_negocios_membros` (papel authenticated) (20/03/2025).
        - `Allow negocio owners to delete negocios_membros` (papel authenticated) (31/03/2025):
            
            **SQL**
            
            `CREATE POLICY "Allow negocio owners to delete negocios_membros" ON public.negocios_membros
            FOR DELETE
            TO authenticated
            USING (
              negocio_id IN (
                SELECT negocio_id FROM negocios_membros WHERE usuario_id = auth.uid() AND is_proprietario = true
              )
            );`
            
    - Ativado RLS para a tabela `negocios` com as políticas: `insert_negocios` (papel public), `update_negocios` (papel public), `allow_auth_admin_insert_negocios` (papel supabase_auth_admin), `allow_authenticated_select_negocios` (papel authenticated), `allow_authenticated_update_negocios` (papel authenticated) (20/03/2025).
    - Ativado RLS para a tabela `users` com a política `allow_authenticated_select_users` (papel authenticated) (20/03/2025):
        
        **SQL**
        
        `CREATE POLICY "allow_authenticated_select_users" ON public.users
        FOR SELECT
        TO authenticated
        USING (
          id = auth.uid() OR
          id IN (
            SELECT usuario_id FROM negocios_membros
            WHERE negocio_id IN (
              SELECT negocio_id FROM negocios_membros WHERE usuario_id = auth.uid()
            )
          )
        );`
        
    - Ativado RLS para a tabela `tags` com a política `allow_authenticated_select_tags` (papel authenticated) (28/03/2025):
        
        **SQL**
        
        `DROP POLICY IF EXISTS "allow_authenticated_select_tags" ON public.tags;
        CREATE POLICY "allow_authenticated_select_tags" ON public.tags
        FOR SELECT
        TO authenticated
        USING (
          negocio_id IN (
            SELECT negocio_id FROM public.negocios_membros WHERE usuario_id = auth.uid()
          )
        );`
        
    - Ativado RLS para a tabela `tags_projetos` com a política `allow_authenticated_select_tags_projetos` (papel authenticated) (28/03/2025):
        
        **SQL**
        
        `DROP POLICY IF EXISTS "allow_authenticated_select_tags_projetos" ON public.tags_projetos;
        CREATE POLICY "allow_authenticated_select_tags_projetos" ON public.tags_projetos
        FOR SELECT
        TO authenticated
        USING (
          projeto_id IN (
            SELECT id FROM public.projetos
            WHERE negocio_id IN (
              SELECT negocio_id FROM public.negocios_membros WHERE usuario_id = auth.uid()
            )
          )
        );`
        
    - Ativado RLS para a tabela `projetos` com políticas ajustadas (31/03/2025):
        - `allow_authenticated_insert_projetos` (papel authenticated):
            
            **SQL**
            
             `DROP POLICY IF EXISTS "allow_authenticated_insert_projetos" ON public.projetos;
             CREATE POLICY "allow_authenticated_insert_projetos" ON public.projetos
             FOR INSERT
             TO authenticated
             WITH CHECK (
               negocio_id IN (
                 SELECT negocio_id FROM public.negocios_membros WHERE usuario_id = auth.uid()
               )
             );`
            
        - `allow_authenticated_update_projetos` (papel authenticated):
            
            **SQL**
            
            `DROP POLICY IF EXISTS "allow_authenticated_update_projetos" ON public.projetos;
            CREATE POLICY "allow_authenticated_update_projetos" ON public.projetos
            FOR UPDATE
            TO authenticated
            USING (
              negocio_id IN (
                SELECT negocio_id FROM public.negocios_membros WHERE usuario_id = auth.uid()
              )
            );`
            
        - `allow_authenticated_delete_projetos` (papel authenticated):
            
            **SQL**
            
             `DROP POLICY IF EXISTS "allow_authenticated_delete_projetos" ON public.projetos;
             CREATE POLICY "allow_authenticated_delete_projetos" ON public.projetos
             FOR DELETE
             TO authenticated
             USING (
               negocio_id IN (
                 SELECT negocio_id FROM public.negocios_membros WHERE usuario_id = auth.uid()
               )
             );`
            
    - Índice `idx_negocios_membros_invite_code` criado na coluna `invite_code` da tabela `negocios_membros` para otimizar buscas de convites pendentes (21/03/2025).
    - Trigger `copy_auth_user_to_users` ajustado para criar um registro inicial em `negocio_detalhes`, ao criar um novo negócio no signup (27/03/2025).
    - Trigger `copy_auth_user_to_users` ajustado para adicionar `usuario_id` (com `NEW.id`) na criação dos projetos de exemplo ("Casamento Maria & João" e "Jantar com amigos") (31/03/2025).
    - Tabela `negocio_detalhes` criada com colunas: id (uuid, PK), negocio_id (uuid, FK negocios.id, NOT NULL, UNIQUE), tipo_identificacao (text, com constraint CHECK (tipo_identificacao IN ('CPF', 'CNPJ'))), identificacao (text), endereco (text), telefone (text), email (text) (27/03/2025).
    - Constraint CHECK ajustada na coluna `tipo_identificacao` para aceitar apenas "CPF" e "CNPJ" (em letras maiúsculas) (27/03/2025).
    - Removidas as constraints NOT NULL das colunas `tipo_identificacao`, `identificacao`, `endereco`, `telefone`, e `email` na tabela `negocio_detalhes`, tornando esses campos opcionais para suportar vendas online via Impay (27/03/2025, 18:00h).
    - Adicionada constraint CHECK na coluna `status` da tabela `negocios_membros` para aceitar os valores 'associado', 'convite_enviado', e 'sem_negocio' (30/03/2025):
        
        **SQL**
        
        `ALTER TABLE negocios_membros
        DROP CONSTRAINT IF EXISTS negocios_membros_status_check;
        ALTER TABLE negocios_membros
        ADD CONSTRAINT negocios_membros_status_check
        CHECK (status IN ('associado', 'convite_enviado', 'sem_negocio'));`
        
- **Não Realizado:**
    - Nenhuma pendência imediata no modelo de dados para o escopo atual.

**Custom Actions**

- **Já Realizado:**
    - `povoarAppStates` (5.1): Original populava FFAppState com um único `negocioAtual` e `assinaturaUsuario` para novos usuários (Etapa 1, 16/03/2025). Atualizado para mapear `negociosUsuario` (via NegociosStruct) e `negociosMembros` (via NegociosMembrosStruct), definir `negocioAtual` como o primeiro negócio no signup, armazenar `planoAtual` em `FFAppState.planoAtual`, e salvar em SharedPreferences (versão 2.3.2) apenas para dados básicos (ex.: negocioIdAtual, planoAtual) (Etapa 2, 19/03/2025, 09:00h; ajustado para SharedPreferences em 20/03/2025). Ajustado para preservar `negocioIdAtual` e `planoAtual` durante a atualização do AppState, evitando sobrescrita (26/03/2025). Adicionado suporte para `membrosNegocioAtual` (via NegociosMembrosStruct) a partir de `apiResponse['membros_negocio_atual']` (26/03/2025). Adicionado suporte para `negocioAtual` (via NegociosStruct) a partir de `apiResponse['negocio_atual']`, preenchendo os dados do negócio atual com base no `negocioIdAtual` (27/03/2025). Ajustado para incluir os dados de `negocio_detalhes` como um objeto `detalhes` (via NegocioDetalhesStruct) em `negociosUsuario` e `negocioAtual` (27/03/2025). Mantida a limpeza inicial de `projetosUsuario` para garantir que apenas os projetos do negócio atual sejam povoados, conforme o comportamento padrão de alternância (28/03/2025). Testado com signup, confirmado população inicial (19/03/2025, 11:32h; retestado com ajustes em 26/03/2025, 27/03/2025, 28/03/2025, e 30/03/2025).
    - `criarNovoNegocio` (5.3): Criado para adicionar um segundo negócio próprio em `negocios` e `negocios_membros` com `is_proprietario = true`, validando limite de 2 via mock de `verificarAcessoModulo`, atualizando FFAppState e cache (Etapa 2, 19/03/2025, 09:00h).
    - `convidarMembroNegocio` (5.4): Criado para convidar membros, inserindo em `convites_pendentes`, validando limites (0 Free, 5 Premium, 20 Pro) via mock, chamando a Edge Function `send_invite` para enviar o email de convite via Resend, e atualizando FFAppState e cache (Etapa 2, 19/03/2025, 09:00h; ajustado para chamar Edge Function em 20/03/2025). Ajustado para inserir diretamente em `negocios_membros` e incluir o campo `invite_code` (20/03/2025). Ajustado para padronizar `acesso_modulos` com `['total']` quando acesso total for selecionado (30/03/2025). Adicionada lógica de exibição de mensagem de erro visual para o limite de convites, usando `FFAppState.exibirMsgBox` e `FFAppState.msgBox` (31/03/2025). Ajustado para chamar a Edge Function `send_invite_email` **antes** de inserir o registro no banco de dados, eliminando a necessidade de rollback em caso de falha no envio do email (31/03/2025). Testado com usuários novos, confirmado envio de email (20/03/2025, 12:34h). Testado o limite de convites para o plano trial (5, obedecendo ao Premium), confirmado funcionamento (30/03/2025). Testado o fluxo de falha na Edge Function, confirmado que o registro não é inserido no banco de dados (31/03/2025).
    - `carregarDadosUsuario` (5.5): Criado para carregar dados completos de usuários existentes, mapeando `financeiroNegocio`, `campanhasNegocio`, `projetosMarketing`, e `postagensMarketing`, com suporte a multi-negócios (Etapa 2, 19/03/2025, 09:00h). Testado com login, confirmado carregamento correto (19/03/2025, 11:32h). *(Observação: Funcionalidade integrada/substituída por `povoarAppStates`)*.
    - `alternarNegocio`: Criado para alternar entre negócios, atualizando `negocioIdAtual`, recarregando dados via `listar_dados_iniciais`, e preservando `negocioIdAtual` e `planoAtual` no AppState (Etapa 2, 19/03/2025, 09:00h; implementado e ajustado em 26/03/2025). Adicionado controle de `isLoading` no FFAppState (tipo bool, valor padrão false) para gerenciar a visibilidade de widgets durante a alternância, definido como `true` no início e `false` no final (ou em caso de erro) (30/03/2025). Adicionados logs para verificar o estado de `isLoading` durante o processo (30/03/2025). Adicionada chamada à Custom Action `atualizarAcessosModulos` após `povoarAppStates`, garantindo que os acessos a módulos sejam atualizados ao alternar negócios (31/03/2025). Testado com alternância, confirmado funcionamento correto (26/03/2025, 27/03/2025, 28/03/2025, e 30/03/2025).
    - `removerAcessoModulo`: Adicionada para remover permissões de módulos de um membro em `negocios_membros` (20/03/2025). Ajustada para suportar convites pendentes (usuario_id = null) usando `listIndex` ou `registroId` para identificar o registro (30/03/2025). Ajustada para implementar a lógica de remoção: se `acesso_modulos` contém `"total"`, define como `["sem acesso"]`; se não, remove o módulo específico clicado na ListView (30/03/2025). Ajustada para definir `status = "sem_negocio"` e remover o membro das listas do FFAppState se `acesso_modulos` já for `["sem acesso"]` (30/03/2025). **Ajuste (31/03/2025):** Corrigida a lógica para remover `"total"` e garantir que `"Sem acesso à módulos"` seja incluído quando aplicável, especialmente quando já existe `"Sem acesso à Projetos"`, resultando em `["Sem acesso à Projetos", "Sem acesso à módulos"]`. Testado com remoção de acesso total e módulos específicos, confirmado funcionamento (31/03/2025).
    - `adicionarAcessoModulo`: Adicionada para adicionar permissões de módulos a um membro em `negocios_membros` (20/03/2025). *Observação: Substituída por `atualizarAcessoModulos` para gerenciar a lista completa de `acesso_modulos` (31/03/2025).*
    - `atualizarAcessoModulos`: Criada para atualizar a lista completa de `acesso_modulos` de um membro, suportando adição, remoção, e definição de acesso total (31/03/2025). Ajustada para suportar convites pendentes (usuario_id=null) usando `listIndex` OU `registroId` (31/03/2025). Ajustada para remover `"sem acesso"` da lista antes de adicionar novos módulos, garantindo que o estado seja consistente (31/03/2025). **Ajuste (31/03/2025):** Corrigida a lógica para remover `"Sem acesso à módulos"` quando novos módulos granulares são adicionados, e `"Sem acesso à Projetos"` quando novos módulos de projetos são adicionados. Ajustada para evitar múltiplos níveis de acesso conflitantes (ex.: `["Projetos", "Projetos Próprios"]` ou `["total", "Etapas", "Sprints"]`), garantindo que apenas um nível de acesso a projetos e um nível de acesso a módulos sejam permitidos. Ajustada para incluir `"Sem acesso à módulos"` quando `modulosSelecionados` está vazia e o nível de acesso a módulos não é `"total"`. Testado com diferentes cenários: atualização com `"Sem acesso à módulos"` existente, mudança de níveis de acesso a projetos, e criação com `modulosSelecionados` vazia (31/03/2025).
    - `atualizarAcessosModulos` (Função da CA): Ajustada para preencher `FFAppState().acessosModulosLista` (uma lista de `AcessoModuloStruct`) com os dados retornados pela RPC `obter_acessos_modulos`, eliminando os App States `acessosModulos` e `motivosBloqueio` (31/03/2025). Testado com adição de módulos, acesso total, e transição de `"sem acesso"`, confirmado funcionamento (31/03/2025).
    - `associarConviteAposCadastro`: Adicionada para associar um convite a um usuário após o cadastro, atualizando `negocios_membros` com o `usuario_id` do novo usuário (20/03/2025). Definido o momento de chamada: após o registro e autenticação do usuário na tela `RegisterPage`, antes de redirecionar para `HomeAgenda` (21/03/2025). Testado no fluxo de cadastro, confirmado associação do convite (21/03/2025).
    - `inicializarRealtimeMembros`: Criado para inicializar o Realtime do Supabase para a tabela `negocios_membros`, atualizando `FFAppState.negociosMembros` em tempo real (20/03/2025). Ajustado para atualizar também `FFAppState.membrosNegocioAtual` com base no `negocioIdAtual`, incluindo join com `users` para buscar `nome_artistico` (26/03/2025). *Observação: Substituído por `atualizarMembrosNegocioAtual` para realizar uma consulta única ao banco de dados, recarregando `membrosNegocioAtual` ao entrar na tela de membros, garantindo dados atualizados sem depender de cache (30/03/2025).* Testado com alterações em `negocios_membros`, confirmado funcionamento do Realtime (26/03/2025).
    - `atualizarMembrosNegocioAtual`: Criada para realizar uma consulta única ao banco de dados, atualizando `FFAppState.membrosNegocioAtual` com os membros do negócio atual (`negocioIdAtual`), chamada no evento On Page Load da tela de membros para garantir dados atualizados (30/03/2025). Ajustada para excluir membros com `status = "sem_negocio"` da consulta, garantindo que não sejam exibidos na UI (31/03/2025). Testado com reabertura da tela, confirmado que membros com `status="sem_negocio"` não são exibidos (31/03/2025).
    - `atualizarNegocio`: Criado para atualizar os detalhes do negócio atual na tela Meu Negócio, atualizando `negocios` (nome) e `negocio_detalhes` (tipo_identificacao, identificacao, endereco, telefone, email) (27/03/2025). Ajustado para usar o NegocioDetalhesStruct ao atualizar FFAppState (27/03/2025). Ajustado para lidar com valores `null` no evento `onChange` de cada TextField, mantendo os valores atuais do FFAppState para campos não alterados (27/03/2025, 18:00h). Removidas as validações obrigatórias para `tipo_identificacao` e `identificacao`, já que esses campos agora são opcionais no banco de dados (27/03/2025, 18:00h). Mantida a normalização de `tipo_identificacao` para letras maiúsculas ("CPF" ou "CNPJ") (27/03/2025, 18:00h). Testado na tela Meu Negócio, confirmado funcionamento correto com atualizações parciais (27/03/2025, 18:00h).
    - `removerAcessoProprio`: Criada para permitir que um membro saia de um negócio ao qual pertence, atualizando o registro em `negocios_membros` para `status = "sem_negocio"` e removendo o membro das listas do FFAppState (31/03/2025). Testado com um membro saindo de um negócio, confirmado funcionamento (31/03/2025).
    - `criarProjeto`: Adicionado o campo `usuario_id` na inserção no BD (31/03/2025). Corrigida verificação de permissões (`Projetos`, `Projetos Próprios` ou `is_proprietario`) (31/03/2025). Adicionada verificação do limite de 100 projetos no plano "free" (aplicável a todos) (31/03/2025). Corrigida contagem de projetos para o limite (usando `.length`) (31/03/2025). Adicionado armazenamento de mensagens de erro em `FFAppState().msgBox` (31/03/2025). Testado com diferentes cenários (permissão, limite, mensagens) (31/03/2025).
    - `podeExibirIconeVisualizarProjetos` (Custom Function): Criada e corrigida a lógica de visibilidade do ícone de olho na `homeAgenda`, retornando `true` para `"Projetos"` ou `"Projetos Espectador"` e `false` para `"Projetos Próprios"` ou `"Sem acesso à Projetos"`. Ajustada para incluir proprietários (`isProprietario = true`) (31/03/2025).
    - *Observação Geral:* Após a implementação de `removerAcessoProprio` (ou possivelmente `associarConviteAposCadastro`), foi identificado um erro de compilação nas Custom Actions ("Erro no compiling das CA"). O erro pode estar relacionado a dependências não atualizadas no FlutterFlow ou a um problema nas CAs mencionadas. No entanto, o app está rodando normalmente sem erros em tempo de execução, então o problema será monitorado para verificar se será resolvido em futuras atualizações do FlutterFlow. Por enquanto, o erro não está impactando o desenvolvimento e será ignorado (31/03/2025).
- **Não Realizado:**
    - Criar Custom Function `verificarAcessoModulo` para centralizar a lógica de controle de acesso (pendente 31/03/2025).
    - Implementar fallback para assinaturas expiradas (CA `verificarAssinaturaExpirada`) (pendente 31/03/2025).
    - Revisar/Ajustar a Custom Action `criarNovoNegocio` para garantir que atenda aos requisitos atuais de criação de negócios (pendente 06/04/2025).

**Edge Functions**

- **Já Realizado:**
    - `send_invite_email`: Criada para verificar a existência do usuário em `auth.users` usando `listUsers` com filtragem manual, inserir em `negocios_membros` (se o usuário existe) ou manter o convite em `negocios_membros` (se o usuário não existe), e enviar o email de convite via Resend. Suporta CORS com tratamento para requisições OPTIONS. Configurada a `RESEND_API_KEY` no Edge Function Secrets Management para envio de emails via Resend, com email padrão `noreply@resend.dev` e envio restrito ao email `phototaskapp@gmail.com` até a configuração do domínio `phototask.com` (20/03/2025). Alterado o domínio no link do email para `https://phototask-xozde6.flutterflow.app/` até a configuração do domínio oficial `phototask.com` em produção (21/03/2025).
- **Não Realizado:**
    - Configurar o domínio `phototask.com` no Resend para permitir o envio de emails para outros destinatários, ajustando o endereço `from` para um email do domínio verificado (ex.: `noreply@phototask.com`) (pendente 31/03/2025).

**Frontend**

- **Já Realizado:**
    - `HomeAgenda`: Exibe projetos do negócio inicial criado no signup, usando `FFAppState.projetosUsuario` (Etapa 1, 16/03/2025). Ajustado para exibir projetos do `negocioIdAtual` após alternância de negócios (26/03/2025). Adicionada condicional de visibilidade `FFAppState().isLoading` a todos os ListView e dados do FFAppState (ex.: membrosNegocioAtual, negociosUsuario, projetosUsuario) para evitar renderizações prematuras durante a alternância (30/03/2025). Adicionado um CircularProgressIndicator com condicional de visibilidade `FFAppState().isLoading` e animação de rotação (rotate) para exibir um loading animado durante a alternância, preenchendo o intervalo de transição (30/03/2025). Corrigido o erro "Unexpected null value" ao adicionar condicional `FFAppState().isLoading` aos componentes que exibem nome do negócio e proprietário, evitando acesso a valores nulos durante a alternância (30/03/2025).
    - `MembrosNegocioPage`: Adicionada para listar membros associados e convites pendentes, usando `FFAppState.membrosNegocioAtual` (20/03/2025; ajustado para `membrosNegocioAtual` em 26/03/2025). Ajustada para chamar a Custom Action `atualizarMembrosNegocioAtual` no evento On Page Load, garantindo dados atualizados diretamente do banco de dados, sem depender de cache (30/03/2025). Ajustada para exibir apenas "Acesso total (Administrador)" se `acesso_modulos` contiver `"total"`, economizando espaço e melhorando a estética (30/03/2025). Ajustada para excluir membros com `status = "sem_negocio"` da exibição, garantindo que não sejam mostrados na UI (31/03/2025). Adicionada funcionalidade para remover e atualizar acessos de módulos via `removerAcessoModulo` e `atualizarAcessoModulo` (31/03/2025). Adicionada opção para o membro sair do negócio, chamando a Custom Action `removerAcessoProprio` (31/03/2025). Confirmado funcionamento com Realtime (26/03/2025).
    - `EnviarConvitePage`: Adicionada para enviar convites e adicionar acessos a membros existentes (20/03/2025). Ajustada para exibir mensagem de erro visual quando o limite de convites é atingido, usando `FFAppState.exibirMsgBox` e `FFAppState.msgBox` (31/03/2025).
    - `RegisterPage`: Adicionado parâmetro `inviteCode` e chamada à Custom Action `associarConviteAposCadastro` após o registro, antes de redirecionar para `HomeAgenda` (21/03/2025).
    - `EditarAcessosPage`: Adicionada para gerenciar os módulos de acesso de um membro, permitindo adicionar, remover, ou definir acesso total (31/03/2025). Configurada para inicializar `selectedModules` com os módulos atuais do membro, excluindo `"sem acesso"`, e chamar `atualizarAcessoModulo` ao salvar (31/03/2025).
    - `MsgBox`: Criado como um componente reutilizável para exibir mensagens ao usuário, usando `FFAppState.msgBox` para armazenar a mensagem (31/03/2025). Configurado para exibir mensagens de erro como "Limite de convites atingido para seu plano atual" (31/03/2025). Ajustado para ser usado como Custom Dialog, eliminando a necessidade de variáveis de exibição e de embutir o componente diretamente na página (31/03/2025).
    - Adicionado um componente de UI (botão de ícone) para alternar entre negócios, chamando a Custom Action `alternarNegocio` e atualizando a exibição dos dados (ex.: projetos, membros) com base no `negocioIdAtual`. Inserimos `listar_dados_iniciais` e `povoarAppStates` para realizar o povoamento quando o usuário alterna entre negócios (27/03/2025).
    - `MeuNegocio`: Adicionada para permitir que o proprietário edite os detalhes do negócio atual (nome de `negocios`; tipo_identificacao, identificacao, endereco, telefone, email de `negocio_detalhes` via `NegocioDetalhesStruct`) e acesse a tela `MembrosNegocioPage`. Inclui verificação de visibilidade para exibir apenas para proprietários (27/03/2025). Ajustada para chamar a Custom Action `atualizarNegocio` no evento `onChange` de cada TextField (e RadioButton/Dropdown para `tipoIdentificacao`), com valores iniciais definidos com base no FFAppState (27/03/2025, 18:00h). Campo `tipoIdentificacao` armazenado em um Page State para evitar problemas de sincronização (27/03/2025, 18:00h).
    - `Menu Principal`: Adicionado link para a tela `MeuNegocio`, visível apenas para proprietários (27/03/2025).
    - `BloqueioAcesso`: Criado como um componente global para exibir dialogs personalizados quando o acesso a um módulo é negado, com condicionais de visibilidade para ícones, textos e botões de ação (31/03/2025). Configurado para usar `FFAppState().acessosModulosLista` (Data Type `AcessoModuloStruct`) com a operação Find Element in List, permitindo comparações diretas com o parâmetro `modulo` (31/03/2025). Ajustado para exibir:
        - Ícone de estrela e botão "Quero conhecer os planos disponíveis" para motivos `plano_insuficiente_dono_premium`, `plano_insuficiente_dono_pro`, `plano_insuficiente_dono_expertpro`.
        - Ícone de cadeado e sem botão para motivos `membro_sem_acesso`, `plano_insuficiente_membro`, e `assinatura_inativa`.
        - Ícone de cadeado e botões "Continuar" e "Resolver pendência" para o motivo `assinatura_vencida_dono`.
        - Ícone genérico para outros motivos (ex.: `modulo_invalido`, `cache_nao_preenchido`).
- **Não Realizado:**
    - Adicionar controle de acesso ao módulo Projetos usando `FFAppState().acessosModulosLista` na `HomeAgenda`, com condicional de visibilidade para exibir o conteúdo apenas se permitido (PENDENTE).
    - Implementar a exibição visual de notificações (ex.: ícone com contador de notificações pendentes) (pendente 31/03/2025).
    - Garantir que, ao alternar de negócio, a etapa padrão (`etapaPadraoId` do `configuracoesUsuario`) seja automaticamente selecionada na UI (pendente 31/03/2025).
    - Ajustar o contador de projetos na `ListView` horizontal de etapas na `homeAgenda` para contar apenas os projetos visíveis na `ListView` vertical de projetos ativa abaixo (ex.: se exibindo apenas projetos próprios, contar apenas projetos próprios, em vez de todos os projetos do negócio) (pendente 06/04/2025).

**Testes**

- **Já Realizado:**
    - Testes manuais de `povoarAppStates` com signup, confirmando `negociosUsuario`, `negociosMembros`, `membrosNegocioAtual`, `negocioAtual`, e `projetosUsuario` (Etapa 2, 19/03/2025, 09:00h; validado com widget de texto em 19/03/2025, 11:32h; retestado com ajustes em 26/03/2025, 27/03/2025, 28/03/2025, e 30/03/2025).
    - Testes do trigger `copy_auth_user_to_users` corrigido, confirmando criação do negócio inicial, assinatura, projetos, etapas, e dependências (19/03/2025, 10:00h; validado via API em 19/03/2025, 11:32h).
    - Testes de `carregarDadosUsuario` com login, confirmado carregamento correto de todos os dados no FFAppState (19/03/2025, 11:32h).
    - Testes manuais de `convidarMembroNegocio` no FlutterFlow, confirmando a integração com a Edge Function `send_invite_email` e o envio de emails via Resend para usuários novos (20/03/2025, 12:34h). Confirmado o teste manual para usuários novos, verificando a inserção em `negocios_membros` e o envio de emails via Resend (21/03/2025). Testado o limite de convites para o plano trial (5, obedecendo ao Premium), confirmado funcionamento (30/03/2025). Testado o fluxo de falha na Edge Function `send_invite_email` (ex.: email inválido `sad@as.com`), confirmado que o registro não é inserido no banco de dados e que a mensagem de erro é exibida via MsgBox (31/03/2025).
    - Testado o fluxo de convite para um usuário não registrado (`phototaskapp@gmail.com`), confirmando o envio do email com o link de registro contendo o `invite_code` (`https://phototask-xozde6.flutterflow.app/register?invite_code=...`) e a exibição correta dos metadados na URL ao abrir o link (21/03/2025).
    - Testado o fluxo de cadastro com convite, confirmando a associação do `invite_code` e a atualização de `negocios_membros` (21/03/2025).
    - Testado o funcionamento de `alternarNegocio`, confirmando que `negocioIdAtual`, `membrosNegocioAtual`, e `negocioAtual` são atualizados corretamente (26/03/2025, 27/03/2025, 28/03/2025, e 30/03/2025).
    - Testado o Realtime com `inicializarRealtimeMembros`, confirmando que `negociosMembros` e `membrosNegocioAtual` são atualizados em tempo real e refletidos na UI (26/03/2025).
    - Testada a UI de alternância de negócios no frontend, confirmando a exibição dos dados do novo negócio (27/03/2025, 28/03/2025, e 30/03/2025).
    - Testada a Custom Action `atualizarNegocio` na tela Meu Negócio, confirmando que atualizações parciais (ex.: alterar apenas `identificacao`) não sobrescrevem outros campos com `null`, e que os valores são preservados no banco de dados e no FFAppState (27/03/2025, 18:00h).
    - Testado o login e signup com a API `listar_dados_iniciais` ajustada para aceitar `p_negocio_id` como opcional, confirmando que os dados iniciais são retornados corretamente (28/03/2025).
    - Testada a alternância de negócios com a política de RLS ajustada para `negocios_membros`, confirmando que todos os membros do negócio atual são retornados (27/03/2025).
    - Testada a alternância de negócios com as políticas de RLS ajustadas para `tags` e `tags_projetos`, confirmando que as tags dos projetos são retornadas corretamente (28/03/2025).
    - Testada a Custom Action `atualizarMembrosNegocioAtual`, confirmando que membros com `status = "sem_negocio"` não são exibidos na UI ao reabrir a tela (31/03/2025).
    - Testada a Custom Action `removerAcessoModulo`, Confirmando que:
        - Se `acesso_modulos` contém `"total"`, define como `["sem acesso"]` e mantém o membro no negócio.
        - Se `acesso_modulos` não contém `"total"`, remove apenas o módulo específico clicado na ListView.
        - Se `acesso_modulos` já é `["sem acesso"]`, define `status="sem_negocio"` e remove o membro das listas do FFAppState.
        - A lógica de adição de `"Sem acesso à módulos"` funciona corretamente com `"Sem acesso à Projetos"` (31/03/2025).
    - Testada a Custom Action `atualizarAcessoModulo`, confirmando que:
        - Adiciona módulos específicos corretamente.
        - Define `acesso_modulos` como `['total']` quando "Acesso total (Administrador)" é selecionado.
        - Remove `"sem acesso"` da lista antes de adicionar novos módulos, garantindo consistência.
        - A lógica de remoção de `"Sem acesso à módulos"` e `"Sem acesso à Projetos"` funciona.
        - Evita níveis de acesso conflitantes.
        - Inclui `"Sem acesso à módulos"` quando aplicável (31/03/2025).
    - Testada a Custom Action `removerAcessoProprio`, confirmando que um membro pode sair de um negócio, definindo `status = "sem_negocio"` e removendo o membro das listas do FFAppState (31/03/2025).
    - Testada a exibição de mensagens de erro via o componente MsgBox, confirmando que a mensagem "Limite de convites atingido para seu plano atual" é exibida corretamente quando o limite de convites é atingido (31/03/2025).
    - Testado o controle de acesso com a RPC `obter_acessos_modulos` ajustada:
        - Membro com plano Free e acesso liberado apenas a Etapas → Apenas Etapas é permitido (31/03/2025).
        - Membro com plano Expert Pro e acesso liberado apenas a Etapas → Apenas Etapas é permitido, mesmo com plano superior (31/03/2025).
        - Dono com assinatura vencida → Bloqueio com motivo `assinatura_vencida_dono` (até 10 dias) (31/03/2025).
        - Membro com assinatura vencida → Sem bloqueio (até 10 dias), acesso permitido se o plano e as permissões permitirem (31/03/2025).
    - Testado o componente `BloqueioAcesso` com diferentes cenários:
        - Membro com plano Free e `acesso_modulos = ['Etapas']` → Exibe "Você não tem acesso a este recurso" para Painel de Marketing (motivo: `plano_insuficiente_membro`), sem botão de ação (31/03/2025).
        - Dono com plano Free → Exibe "Faça o upgrade para o plano Premium para acessar este recurso" para Painel de Marketing (motivo: `plano_insuficiente_dono_premium`), com botão "Quero conhecer os planos disponíveis" (31/03/2025).
        - Dono com assinatura vencida → Exibe "Sua assinatura está vencida. Regularize o pagamento para continuar acessando este recurso" (motivo: `assinatura_vencida_dono`), com botões "Continuar" e "Resolver pendência" (31/03/2025).
        - Membro com assinatura vencida → Sem bloqueio, acesso permitido se o plano e as permissões permitirem (até 10 dias) (31/03/2025).
    - Testada a CA `criarProjeto` com validação de permissão e limite do plano "free" (31/03/2025).
- **Pendência:**
    - Para ser testado quando lançarmos a URL final: se o parâmetro `inviteCode` será passado automaticamente para o `AppState.inviteCode` (atualmente estamos atualizando a variável manualmente no FlutterFlow para realizar os testes, devido ao link atual não ser o domínio final e estarmos em ambiente de desenvolvimento).
- **Não Realizado:**
    - Testes manuais de `criarNovoNegocio`.
    - Testar o fluxo de convite para usuários existentes, confirmando que recebem uma notificação visual.
    - Testar os limites de convites para os planos Free (0) e Pro (20), e exibir a mensagem de erro "Exception: Limite de convites atingido para o plano <plano>" para o usuário via o componente MsgBox (pendente 31/03/2025).

**Lacunas Identificadas**

- Escalabilidade do `listUsers`: O uso de `listUsers` na Edge Function `send_invite_email` pode se tornar ineficiente com um grande número de usuários. Uma solução futura pode ser criar uma função RPC para buscar usuários por email de forma mais eficiente.
- Seleção de Etapa Padrão na Alternância: O toggle de seleção de etapas mantém o histórico do negócio anterior ao alternar, mas na primeira alternância não seleciona a etapa padrão (`etapaPadraoId` do `configuracoesUsuario`), deixando o toggle sem seleção (31/03/2025).
- Erro de Compilação das Custom Actions: Identificado um erro de compilação nas Custom Actions após a implementação de `removerAcessoProprio` OU `associarConviteAposCadastro`. O erro pode estar relacionado a dependências não atualizadas no FlutterFlow ou a um problema nas CAs. No entanto, o app está rodando normalmente sem erros em tempo de execução, então o problema será monitorado para verificar se será resolvido em futuras atualizações do FlutterFlow. Por enquanto, o erro não está impactando o desenvolvimento e será ignorado (31/03/2025).
