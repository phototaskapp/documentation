# **2.1. Banco de Dados Atual - Supabase**

## **2.1.1. Tabelas Existentes**

| **Tabela** | **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **admin_logs** | `id` | uuid | Identificador único do log administrativo | Auditoria no Módulo Administrador | PK | Via `gen_random_uuid()` | `idx_admin_logs_admin_id`, `idx_admin_logs_data_acao` | Realizada |
|  | `admin_id` | uuid | ID do administrador que executou a ação | Vinculação ao admin | FK `users.id` | Obrigatório | `idx_admin_logs_admin_id` (otimiza consultas por admin) | Realizada |
|  | `acao` | text | Descrição da ação realizada (ex.: "visualizar_logs") | Registro de atividades | Nenhuma | Valores predefinidos | - | Realizada |
|  | `detalhes` | jsonb | Dados detalhados da ação (ex.: {"resultado": "sucesso"}) | Detalhamento da ação | Nenhuma | Estrutura JSON, flexível | - | Realizada |
|  | `data_acao` | timestamp | Data e hora da ação administrativa | Histórico de atividades | Nenhuma | Default NOW() | `idx_admin_logs_data_acao` (otimiza ordenação por data) | Realizada |
|  | `nivel_severidade` | text | Nível de severidade (ex.: "info", "warning", "error") | Classificação da ação | Nenhuma | CHECK ("info", "warning", "error") | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio relacionado à ação, se aplicável | Vinculação ao negócio | FK `negocios.id` | Nullable | - | Realizada |
| **assinaturas** | `id` | uuid | Identificador único da assinatura | Gerenciamento de assinaturas | PK, ref. por `users.assinatura_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário vinculado à assinatura | Vinculação a usuários | FK `users.id` | Obrigatório | - | Realizada |
|  | `plano` | text | Nome do plano (ex.: "Grátis", "Premium", "Pro") | Definição de acesso | Nenhuma | Valores predefinidos | - | Realizada |
|  | `data_inicio` | timestamp | Data e hora de início da assinatura | Controle de vigência | Nenhuma | Obrigatório | - | Realizada |
|  | `data_fim` | timestamp | Data e hora de término da assinatura | Controle de vigência | Nenhuma | Nullable | - | Realizada |
|  | `trial_ativo` | boolean | Indica se o período de teste está ativo | Período de teste | Nenhuma | Default `false` | - | Realizada |
|  | `cancelado` | boolean | Indica se a assinatura foi cancelada | Gestão de status | Nenhuma | Default `false` | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio associado à assinatura | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `stripe_customer_id` | text | ID do cliente no Stripe | Integração com Stripe | Nenhuma | Nullable | - | Realizada |
|  | `stripe_subscription_id` | text | ID da assinatura no Stripe | Integração com Stripe | Nenhuma | Nullable | - | Realizada |
|  | `ultimo_pagamento` | timestamp | Data e hora do último pagamento | Histórico de pagamentos | Nenhuma | Nullable | - | Realizada |
|  | `status_pagamento` | text | Estado do pagamento (ex.: "pago", "pendente") | Monitoramento de pagamentos | Nenhuma | Valores predefinidos | - | Realizada |
|  | `data_atualizacao` | timestamp | Data da ultima atualizacao em assinaturas | Facilita sincronizacao e controle | Nenhuma |  |  | Realizada |
| **campanhas_agendamento** | `id` | uuid | Identificador único da campanha | Gestão de campanhas | PK (id, negocio_id) | Via `gen_random_uuid()` | `campanhas_agendamento_pkey` | Realizada |
|  | `nome` | text | Nome da campanha (ex.: "Ensaio de Natal") | Identificação na criação | Nenhuma | Obrigatório | - | Realizada |
|  | `descricao` | text | Descrição da campanha | Detalhes na landing page | Nenhuma | Nullable | - | Realizada |
|  | `valor_sessao` | numeric | Valor por sessão em reais (ex.: 200.00) | Previsão financeira | Nenhuma | Obrigatório, em BRL | - | Realizada |
|  | `valor_reserva` | numeric | Valor da reserva em reais | Gestão financeira | Nenhuma | Nullable | - | Realizada |
|  | `pagamento_completo` | boolean | Indica se o pagamento é completo | Gestão financeira | Nenhuma | Nullable | - | Realizada |
|  | `duracao_sessao` | integer | Duração da sessão em minutos (ex.: 60) | Geração de vagas | Nenhuma | Obrigatório, mínimo 30 | - | Realizada |
|  | `intervalo_sessao` | integer | Intervalo entre sessões em minutos | Geração de vagas | Nenhuma | Nullable, default 0 | - | Realizada |
|  | `pausa_almoco` | text | Intervalo de pausa (ex.: "12:00-13:00") | Geração de vagas | Nenhuma | Nullable, texto livre | - | Realizada |
|  | `data_inicio` | timestamp | Data e hora de início da campanha | Definição de período | Nenhuma | Obrigatório | `idx_campanhas_agendamento_data_inicio` (otimiza filtros por data) | Realizada |
|  | `data_fim` | timestamp | Data e hora de término da campanha | Definição de período | Nenhuma | Obrigatório | `idx_campanhas_agendamento_data_fim` (otimiza filtros por data) | Realizada |
|  | `fotos_inclusas` | integer | Quantidade de fotos inclusas por sessão | Detalhes na landing page | Nenhuma | Obrigatório, mínimo 0 | - | Realizada |
|  | `valor_foto_extra` | numeric | Valor de cada foto extra em reais | Pagamento pós-sessão | Nenhuma | Nullable, default 0 | - | Realizada |
|  | `extras` | text | Observações adicionais (ex.: "Impressão 20x30cm") | Detalhes na landing page | Nenhuma | Nullable | - | Realizada |
|  | `imagens` | ARRAY | URLs de imagens para a landing page (máximo 4) | Visualização na landing page | Nenhuma | Obrigatório, tipo ARRAY | - | Realizada |
|  | `link_acesso` | text | Link único para a landing page pública | Acesso público | Nenhuma | UNIQUE com `negocio_id` | `idx_campanhas_agendamento_link_acesso` (otimiza busca por link) | Realizada |
|  | `status` | text | Estado da campanha (ex.: "ativa", "encerrada") | Controle de campanhas | Nenhuma | Nullable | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio associado à campanha | Vinculação ao negócio | PK (id, negocio_id), FK `negocios.id` | Obrigatório, particionado | `idx_campanhas_agendamento_negocio_id` (otimiza filtros por negócio) | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação da campanha | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **colaboradores** | `id` | uuid | Identificador único do colaborador | Gestão de equipe | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome completo ou apelido do colaborador | Identificação na equipe | Nenhuma | Obrigatório | - | Realizada |
|  | `equipamentos` | text | Lista de equipamentos usados (ex.: "Câmera") | Detalhes da equipe | Nenhuma | Nullable | - | Realizada |
|  | `valor_cache` | numeric | Valor do cachê em reais | Gestão financeira | Nenhuma | Nullable | - | Realizada |
|  | `avaliacao` | text | Comentário sobre desempenho (ex.: "Excelente") | Feedback da equipe | Nenhuma | Nullable | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio ao qual o colaborador está vinculado | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do colaborador | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **colaboradores_funcoes** | `id` | uuid | Identificador único da associação colaborador-função | Gestão de papéis | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `colaborador_id` | uuid | ID do colaborador associado à função | Vinculação ao colaborador | FK `colaboradores.id` | Obrigatório | - | Realizada |
|  | `funcao_id` | uuid | ID da função atribuída (ex.: "Fotógrafo") | Definição de papéis | FK `funcoes.id` | Obrigatório | - | Realizada |
|  | `data_associacao` | timestamp | Data e hora da associação | Histórico de vínculos | Nenhuma | Default NOW() | - | Realizada |
| **colaboradores_padrao** | `id` | uuid | Identificador único do colaborador padrão (template) | Templates de colaboradores | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do colaborador padrão | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `equipamentos` | text | Equipamentos padrão (ex.: "Câmera") | Detalhes do template | Nenhuma | Nullable | - | Realizada |
|  | `valor_cache` | numeric | Valor padrão do cachê em reais | Previsão financeira | Nenhuma | Nullable | - | Realizada |
|  | `avaliacao` | text | Avaliação padrão (ex.: "Bom") | Feedback inicial | Nenhuma | Nullable | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| **colaboradores_projetos** | `id` | uuid | Identificador único da associação colaborador-projeto | Vinculação de equipe a projetos | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `colaborador_id` | uuid | ID do colaborador associado ao projeto | Gestão de equipe | FK `colaboradores.id` | Obrigatório | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto ao qual o colaborador está vinculado | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `data_associacao` | timestamp | Data e hora da associação | Histórico de vínculos | Nenhuma | Default NOW() | - | Realizada |
|  | `confirmado` | boolean | Indica se o colaborador confirmou participação | Status da equipe | Nenhuma | Default `false` | - | Realizada |
|  | `nome` | text | Nome específico do colaborador no projeto | Identificação no projeto | Nenhuma | Pode diferir de `colaboradores.nome` | - | Realizada |
|  | `equipamentos` | text | Equipamentos usados no projeto | Detalhes do projeto | Nenhuma | Nullable | - | Realizada |
|  | `valor_cache` | numeric | Valor do cachê pago neste projeto | Gestão financeira | Nenhuma | Nullable | - | Realizada |
| **compromissos** | `id` | uuid | Identificador único do compromisso | Gestão de compromissos | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto vinculado ao compromisso | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `tarefa_id` | uuid | ID da tarefa associada ao compromisso | Vinculação à tarefa | FK `tarefas.id` | Obrigatório | - | Realizada |
|  | `data_vencimento` | timestamp | Data e hora de vencimento do compromisso | Controle de prazos | Nenhuma | Obrigatório | - | Realizada |
|  | `nome` | text | Nome do compromisso (ex.: "Reunião com Cliente") | Identificação do compromisso | Nenhuma | Obrigatório | - | Realizada |
|  | `concluido` | boolean | Indica se o compromisso foi concluído | Progresso do compromisso | Nenhuma | Default `false` | - | Realizada |
|  | `data_conclusao` | timestamp | Data e hora de conclusão do compromisso | Histórico do compromisso | Nenhuma | Nullable | - | Realizada |
|  | `responsavel_id` | uuid | ID do usuário responsável pelo compromisso | Gestão de responsáveis | FK `users.id` | Nullable | - | Realizada |
|  | `data_criacao` | timestamp | **Atualização em 17/03/2025**:
- Adicionado 'data_criacao'  à tabela: para controle da criacao dos compromissos |  |  |  |  | Realizada |
| **configuracoes_sistema** | `id` | uuid | Identificador único da configuração | Configurações globais | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `chave` | text | Chave da configuração (ex.: "zapSign_free_markup") | Identificação da configuração | Nenhuma | Obrigatório | - | Realizada |
|  | `valor` | numeric | Valor numérico da configuração (ex.: 3.99) | Valor da configuração | Nenhuma | Obrigatório | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `descricao` | text | Descrição da configuração | Detalhamento da configuração | Nenhuma | Nullable | - | Realizada |
| **configuracoes_usuario** | `id` | uuid | Identificador único das configurações do usuário | Personalização do usuário | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário vinculado às configurações | Vinculação ao usuário | FK `users.id` | Obrigatório | - | Realizada |
|  | `notificacoes_push` | boolean | Indica se o usuário recebe notificações push | Configuração de alertas | Nenhuma | Default `true` | - | Realizada |
|  | `alarme_ativado` | boolean | Indica se alarmes sonoros estão ativos | Configuração de alertas | Nenhuma | Default `false` | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado às configurações | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `idioma` | text | Idioma preferido (ex.: "pt-BR") | Localização da interface | Nenhuma | Default "pt-BR" | - | Realizada |
|  | `tema` | text | Tema visual (ex.: "light", "dark") | Personalização visual | Nenhuma | Default "light" | - | Realizada |
|  | `etapa_padrao_id` | uuid | ID da etapa padrão para novos projetos | Configuração inicial | FK `etapas.id` | Nullable | - | Realizada |
|  | `criado_em` | timestamp | Data e hora de criação das configurações | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `atualizado_em` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `primeiro_acesso` | boolean | Indica se é o primeiro acesso do usuário | Controle de onboarding | Nenhuma | Default `true` | - | Realizada |
|  | `aceitou_termos` | boolean | Indica se o usuário aceitou os termos de uso | Conformidade legal | Nenhuma | Default `false` | - | Realizada |
|  | `aceitou_lgpd` | boolean | Indica se o usuário aceitou a LGPD | Conformidade legal | Nenhuma | Default `false` | - | Realizada |
|  | `aviso_cookies` | boolean | Indica se o aviso de cookies foi aceito | Consentimento | Nenhuma | Default `true` | - | Realizada |
|  | `fuso_horario` | text | Fuso horário preferido (ex.: "UTC-3") | Suporte multi-países | Nenhuma | Nullable | - | Realizada |
|  | `regiao` | text | Região geográfica (ex.: "BR") | Suporte multi-países | Nenhuma | Nullable | - | Realizada |
|  | `moeda` | text | Moeda preferida (ex.: "BRL") | Suporte a pagamentos | Nenhuma | Default "BRL" | - | Realizada |
| **contratos** | `id` | uuid | Identificador único do contrato | Gestão de contratos | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao contrato | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário associado ao contrato | Vinculação ao usuário | FK `users.id` | Obrigatório | - | Realizada |
|  | `external_id` | text | ID externo do contrato (ex.: ZapSign) | Integração externa | Nenhuma | Nullable | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do contrato | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `status` | text | Estado do contrato (ex.: "rascunho", "assinado") | Progresso do contrato | Nenhuma | Valores predefinidos | - | Realizada |
|  | `valor` | numeric | Valor do contrato em reais | Gestão financeira | Nenhuma | Obrigatório | - | Realizada |
|  | `descricao` | text | Descrição do contrato | Detalhamento do contrato | Nenhuma | Nullable | - | Realizada |
| **custos_sistema** | `id` | uuid | Identificador único do custo | Gestão financeira | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `descricao` | text | Descrição do custo (ex.: "Hospedagem Supabase") | Identificação do custo | Nenhuma | Obrigatório | - | Realizada |
|  | `valor` | numeric | Valor do custo em reais | Controle financeiro | Nenhuma | Obrigatório, em BRL | - | Realizada |
|  | `data_registro` | timestamp | Data e hora de registro do custo | Histórico | Nenhuma | Default NOW() | - | Realizada |
|  | `recorrente` | boolean | Indica se o custo é recorrente | Gestão de recorrência | Nenhuma | Default `false` | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio relacionado, se aplicável | Vinculação ao negócio | FK `negocios.id` | Nullable | - | Realizada |
| **debug_logs** | `id` | integer | Identificador único do log de depuração | Monitoramento de erros | PK | Sequencial, gerado pelo Supabase | - | Realizada |
|  | `created_at` | timestamp | Data e hora de criação do log | Auditoria de erros | Nenhuma | Default NOW() | - | Realizada |
|  | `message` | text | Mensagem detalhada do erro | Detalhamento de erros | Nenhuma | Campo livre, inclui stack trace | - | Realizada |
| **etapas** | `id` | uuid | Identificador único da etapa | Gestão de etapas | PK, ref. por `projetos.etapa_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome da etapa (ex.: "Evento") | Identificação de etapas | Nenhuma | Obrigatório | - | Realizada |
|  | `cor` | text | Código de cor (ex.: "#FF5733") | Exibição visual | Nenhuma | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à etapa | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `ordem` | smallint | Ordem de exibição da etapa | Sequência de etapas | Nenhuma | Obrigatório | - | Realizada |
|  | `bloqueia_agenda` | boolean | Indica se a etapa bloqueia a agenda | Controle de conflitos | Nenhuma | Default `false` | - | Realizada |
|  | `prazo_padrao` | integer | Prazo padrão em dias | Gestão de prazos | Nenhuma | Nullable | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação da etapa | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `fixa` | boolean | Indica se a etapa é fixa e não editável | Configuração de etapas | Nenhuma | Default `false` | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **etapas_padrao** | `id` | uuid | Identificador único da etapa padrão | Templates de etapas | PK, ref. por `configuracoes_usuario.etapa_padrao_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome da etapa padrão (ex.: "Planejamento") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `cor` | text | Código de cor padrão (ex.: "#00FF00") | Exibição visual | Nenhuma | Obrigatório | - | Realizada |
|  | `ordem` | smallint | Ordem de exibição no template | Sequência no template | Nenhuma | Obrigatório | - | Realizada |
|  | `fixa` | boolean | Indica se a etapa padrão é fixa | Configuração de templates | Nenhuma | Default `false` | - | Realizada |
|  | `bloqueia_agenda` | boolean | Indica se bloqueia a agenda | Controle de conflitos | Nenhuma | Default `false` | - | Realizada |
|  | `prazo_padrao` | integer | Prazo padrão em dias | Gestão de prazos | Nenhuma | Nullable | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **financeiro_projetos** | `id` | uuid | Identificador único da transação financeira | Gestão financeira | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto vinculado à transação | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário responsável pela transação | Vinculação ao usuário | FK `users.id` | Obrigatório | - | Realizada |
|  | `tipo` | text | Tipo da transação (ex.: "entrada", "saída") | Categorização financeira | Nenhuma | Obrigatório | - | Realizada |
|  | `valor` | numeric | Valor total da transação em reais | Gestão financeira | Nenhuma | Obrigatório, em BRL | - | Realizada |
|  | `data` | timestamp | Data e hora da transação | Histórico financeiro | Nenhuma | Obrigatório | - | Realizada |
|  | `status` | text | Estado da transação (ex.: "pendente", "pago") | Monitoramento financeiro | Nenhuma | Valores predefinidos | - | Realizada |
|  | `descricao` | text | Descrição da transação (ex.: "Pagamento fotógrafo") | Detalhamento financeiro | Nenhuma | Nullable | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à transação | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `forma_pagamento` | text | Método de pagamento (ex.: "pix", "boleto") | Gestão de pagamentos | Nenhuma | Valores predefinidos | - | Realizada |
|  | `quantidade_parcelas` | integer | Número de parcelas, se aplicável | Parcelamento | Nenhuma | Nullable, mínimo 1 se parcelado | - | Realizada |
|  | `valor_parcela` | numeric | Valor de cada parcela em reais | Parcelamento | Nenhuma | Nullable, calculado se parcelado | - | Realizada |
|  | `datas_vencimento` | ARRAY | Lista de datas de vencimento das parcelas | Controle de vencimentos | Nenhuma | Nullable, tipo ARRAY | - | Realizada |
| **formularios_clientes** | `id` | uuid | Identificador único do formulário | Gestão de dados de clientes | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `contrato_id` | uuid | ID do contrato vinculado ao formulário | Vinculação ao contrato | FK `contratos.id` | Nullable | - | Realizada |
|  | `nome_completo` | text | Nome completo do cliente | Dados do cliente | Nenhuma | Obrigatório | - | Realizada |
|  | `tipo_identificacao` | text | Tipo de identificação (ex.: "CPF") | Registro legal | Nenhuma | Obrigatório | - | Realizada |
|  | `identificacao` | text | Número de identificação (ex.: "123.456.789-00") | Registro legal | Nenhuma | Obrigatório | - | Realizada |
|  | `endereco` | text | Endereço completo do cliente | Dados de contato | Nenhuma | Obrigatório | - | Realizada |
|  | `email` | text | Email do cliente | Comunicação com cliente | Nenhuma | Obrigatório | - | Realizada |
|  | `telefone` | text | Telefone do cliente (ex.: "+5511999999999") | Comunicação com cliente | Nenhuma | Obrigatório | - | Realizada |
|  | `nacionalidade` | text | Nacionalidade do cliente (ex.: "Brasileiro") | Dados legais | Nenhuma | Obrigatório | - | Realizada |
|  | `data_nascimento` | timestamp | Data de nascimento do cliente | Dados legais | Nenhuma | Obrigatório | - | Realizada |
|  | `estado_civil` | text | Estado civil do cliente (ex.: "Solteiro") | Dados legais | Nenhuma | Obrigatório | - | Realizada |
|  | `outro` | text | Informações adicionais do cliente | Dados extras | Nenhuma | Nullable | - | Realizada |
|  | `link_acesso` | text | URL única para o cliente preencher o formulário | Acesso público | Nenhuma | UNIQUE | - | Realizada |
| **fotos_extras** | `id` | uuid | Identificador único do pedido de fotos extras | Vendas Automáticas (Pro) | PK (id, negocio_id) | Via `gen_random_uuid()` | `fotos_extras_pkey` | Realizada |
|  | `vaga_id` | uuid | ID da vaga associada ao pedido | Vinculação à vaga | FK `vagas_campanha(id, negocio_id)` | Obrigatório | `idx_fotos_extras_vaga_id` (otimiza busca por vaga) | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao pedido | Vinculação ao negócio | PK (id, negocio_id), FK `negocios.id` | Obrigatório, particionado | - | Realizada |
|  | `quantidade` | integer | Quantidade de fotos extras | Detalhamento do pedido | Nenhuma | Obrigatório, mínimo 1 | - | Realizada |
|  | `valor_total` | numeric | Valor total das fotos extras em reais | Gestão financeira | Nenhuma | Obrigatório, em BRL | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do pedido | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `pagamento_id` | uuid | ID do pagamento associado | Vinculação ao pagamento | FK `pagamentos(id, negocio_id)` | Nullable | `idx_fotos_extras_pagamento_id` (otimiza busca por pagamento) | Realizada |
|  | `pagamento_negocio_id` | uuid | ID do negócio do pagamento (parte da FK composta) | Vinculação ao pagamento | FK `pagamentos(id, negocio_id)` | Nullable, usado com `pagamento_id` | - | Realizada |
| **funcoes** | `id` | uuid | Identificador único da função | Gestão de funções | PK, ref. por `colaboradores_funcoes.funcao_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome da função (ex.: "Fotógrafo") | Identificação de papéis | Nenhuma | Obrigatório | - | Realizada |
|  | `icone` | text | Código ou nome do ícone (ex.: "camera") | Exibição visual | Nenhuma | Nullable | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à função | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação da função | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **funcoes_padrao** | `id` | uuid | Identificador único da função padrão | Templates de funções | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome da função padrão (ex.: "Assistente") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `icone` | text | Código ou nome do ícone padrão (ex.: "person") | Exibição visual | Nenhuma | Nullable | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **funcoes_projetos** | `id` | uuid | Identificador único da associação função-projeto | Vinculação de funções a projetos | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `colaborador_projeto_id` | uuid | ID da associação colaborador-projeto | Gestão de papéis | FK `colaboradores_projetos.id` | Obrigatório | - | Realizada |
|  | `nome` | text | Nome da função no projeto (ex.: "Fotógrafo Principal") | Identificação no projeto | Nenhuma | Pode diferir de `funcoes.nome` | - | Realizada |
|  | `icone` | text | Código ou nome do ícone no projeto | Exibição visual | Nenhuma | Nullable | - | Realizada |
|  | `data_associacao` | timestamp | Data e hora da associação | Histórico de vínculos | Nenhuma | Default NOW() | - | Realizada |
| **importacoes_agenda** | `id` | uuid | Identificador único da importação | Gestão de importações | PK (id, negocio_id) | Via `gen_random_uuid()` | `importacoes_agenda_pkey` | Realizada |
|  | `usuario_id` | uuid | ID do usuário que realizou a importação | Vinculação ao usuário | FK `users.id` | Obrigatório | - | Realizada |
|  | `export_id` | uuid | ID da exportação associada | Vinculação à exportação | Nenhuma | Obrigatório | - | Realizada |
|  | `data_importacao` | timestamp | Data e hora da importação | Histórico | Nenhuma | Default NOW() | - | Realizada |
|  | `origem_usuario_id` | uuid | ID do usuário de origem da importação | Auditoria | FK `users.id` | Obrigatório | - | Realizada |
|  | `suspeita_abuso` | boolean | Indica suspeita de abuso na importação | Segurança | Nenhuma | Default `false` | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à importação | Vinculação ao negócio | PK (id, negocio_id), FK `negocios.id` | Obrigatório, particionado | - | Realizada |
| **itens_pacote** | `id` | uuid | Identificador único do item do pacote | Gestão de itens de pacotes | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `pacote_id` | uuid | ID do pacote vinculado ao item | Vinculação ao pacote | FK `pacotes.id` | Obrigatório | - | Realizada |
|  | `nome` | text | Nome do item (ex.: "Sessão de Fotos") | Identificação do item | Nenhuma | Obrigatório | - | Realizada |
|  | `ordem` | integer | Ordem de exibição do item no pacote | Sequência no pacote | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do item | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **itens_pacote_padrao** | `id` | uuid | Identificador único do item padrão | Templates de itens | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `pacote_padrao_id` | uuid | ID do pacote padrão vinculado | Vinculação ao pacote padrão | FK `pacotes_padrao.id` | Obrigatório | - | Realizada |
|  | `nome` | text | Nome do item padrão (ex.: "Álbum Digital") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `ordem` | integer | Ordem de exibição no template | Sequência no template | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do item | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **itens_pacote_template** | `id` | uuid | Identificador único do item no template | Templates de itens | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do item (ex.: "Vídeo de 5min") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `pacote_template_id` | uuid | ID do template de pacote vinculado | Vinculação ao template | FK `pacotes_template.id` | Obrigatório | - | Realizada |
|  | `ordem` | integer | Ordem de exibição no template | Sequência no template | Nenhuma | Obrigatório | - | Realizada |
|  | `ativo` | boolean | Indica se o item está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do item | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `custo` | numeric | Valor estimado do item em reais | Previsão financeira | Nenhuma | Nullable | - | Realizada |
| **itens_pacote_template_padrao** | `id` | uuid | Identificador único do item padrão global | Templates padrão | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do item (ex.: "Ensaio Básico") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `pacote_template_id` | uuid | ID do template padrão global vinculado | Vinculação ao template | FK `pacotes_template_padrao.id` | Obrigatório | - | Realizada |
|  | `ordem` | integer | Ordem de exibição no template | Sequência no template | Nenhuma | Obrigatório | - | Realizada |
|  | `ativo` | boolean | Indica se o item está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do item | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `custo` | numeric | Valor estimado do item em reais | Previsão financeira | Nenhuma | Nullable | - | Realizada |
| **metricas_sistema** | `id` | uuid | Identificador único do registro de métricas | Monitoramento do sistema | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `data_registro` | timestamp | Data e hora do registro das métricas | Histórico | Nenhuma | Default NOW() | - | Realizada |
|  | `usuarios_ativos` | integer | Número de usuários ativos no período | Análise operacional | Nenhuma | Calculado periodicamente | - | Realizada |
|  | `projetos_criados` | integer | Número de projetos criados no período | Análise de uso | Nenhuma | Calculado periodicamente | - | Realizada |
|  | `transacoes_totais` | integer | Número de transações no período | Análise financeira | Nenhuma | Calculado periodicamente | - | Realizada |
|  | `erros_registrados` | integer | Número de erros registrados no período | Monitoramento de erros | Nenhuma | Calculado via `debug_logs` | - | Realizada |
|  | `latencia_media` | numeric | Latência média em milissegundos | Desempenho | Nenhuma | Calculado via Supabase Monitoring | - | Realizada |
|  | `usuarios_inativos_7dias` | integer | Número de usuários inativos por mais de 7 dias | Recuperação comercial | Nenhuma | Calculado via `users.ultimo_acesso` | - | Realizada |
| **negocio_detalhes** | `id` | uuid | Identificador único dos detalhes do negócio | Gestão de informações | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado aos detalhes | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `unique_negocio_identificacao` (UNIQUE com `identificacao`) | Realizada |
|  | `tipo_identificacao` | text | Tipo de identificação (ex.: "PF", "PJ") | Registro legal | Nenhuma | CHECK ("PF", "PJ") | - | Realizada |
|  | `identificacao` | text | Número de identificação (ex.: "123.456.789-00") | Registro legal | Nenhuma | Obrigatório | `unique_negocio_identificacao` (UNIQUE com `negocio_id`) | Realizada |
|  | `endereco` | text | Endereço completo do negócio | Informações de contato | Nenhuma | Obrigatório | - | Realizada |
|  | `telefone` | text | Telefone do negócio (ex.: "+5511999999999") | Contato | Nenhuma | Obrigatório | - | Realizada |
|  | `email` | text | Email adicional do negócio | Contato | Nenhuma | Nullable | - | Realizada |
| **negocios** | `id` | uuid | Identificador único do negócio | Gestão de negócios | PK, ref. por várias tabelas | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do negócio (ex.: "Studio FotoX") | Identificação do negócio | Nenhuma | Obrigatório | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário proprietário ou criador | Vinculação ao proprietário | FK `users.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do negócio | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `designar_dono` | boolean | Indica se outro dono pode ser designado | Gestão de propriedade | Nenhuma | Default `false` | - | Realizada |
|  | `data_backup` | timestamp | Data e hora do último backup | Backup | Nenhuma | Nullable | - | Realizada |
|  | `acesso_modulos` | text[] | Permissões por módulo (ex.: "total") | Controle de acesso | Nenhuma | Default "total" | - | Realizada. Ajustada em 19/03/2025 para suportar multi-negócios com acesso_modulos como TEXT[]. Suporta 'Expert Pro' sem alterações adicionais no schema |
|  | `regiao` | text | Região geográfica (ex.: "BR") | Suporte multi-países | Nenhuma | Nullable | - | Realizada |
| **notificacoes** | `id` | uuid | Identificador único da notificação | Sistema de notificações | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário que recebe a notificação | Vinculação ao destinatário | FK `users.id` | Obrigatório | - | Realizada |
|  | `mensagem` | text | Texto da notificação (ex.: "Projeto X vence amanhã") | Comunicação com o usuário | Nenhuma | Obrigatório | - | Realizada |
|  | `tipo` | text | Tipo da notificação (ex.: "alerta") | Classificação | Nenhuma | Valores predefinidos | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação da notificação | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `lida` | boolean | Indica se a notificação foi lida | Controle de leitura | Nenhuma | Default `false` | - | Realizada |
|  | `push_id` | text | ID da notificação push enviada | Integração com push | Nenhuma | Nullable | - | Realizada |
| **pacotes** | `id` | uuid | Identificador único do pacote | Gestão de pacotes | PK, ref. por `itens_pacote.pacote_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do pacote (ex.: "Pacote Básico") | Identificação do pacote | Nenhuma | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao pacote | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `tipo_projeto_id` | uuid | ID do tipo de projeto associado | Categorização | FK `tipos_projeto.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do pacote | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `tipo_evento` | text | Tipo de evento (ex.: "Casamento") | Categorização | Nenhuma | Obrigatório | - | Realizada |
|  | `valor_pacote` | numeric | Valor estimado do pacote em reais | Previsão financeira | Nenhuma | Nullable | - | Realizada |
| **pacotes_padrao** | `id` | uuid | Identificador único do pacote padrão | Templates de pacotes | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do pacote padrão (ex.: "Pacote Simples") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do pacote | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| **pacotes_template** | `id` | uuid | Identificador único do template de pacote | Templates de pacotes | PK, ref. por `itens_pacote_template.pacote_template_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do template (ex.: "Pacote Ouro") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `tipo_projeto_template_id` | uuid | ID do tipo de projeto template associado | Categorização | FK `tipos_projeto_template.id` | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao template | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `tipo_evento` | text | Tipo de evento (ex.: "Aniversário") | Categorização | Nenhuma | Obrigatório | - | Realizada |
|  | `ativo` | boolean | Indica se o template está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **pacotes_template_padrao** | `id` | uuid | Identificador único do template padrão global | Templates padrão | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do template (ex.: "Pacote Inicial") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `tipo_projeto_template_id` | uuid | ID do tipo de projeto template padrão | Categorização | FK `tipos_projeto_template_padrao.id` | Obrigatório | - | Realizada |
|  | `ativo` | boolean | Indica se o template está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `tipo_evento` | text | Tipo de evento (ex.: "Evento Corporativo") | Categorização | Nenhuma | Obrigatório | - | Realizada |
| **pagamentos** | `id` | uuid | Identificador único da transação de pagamento | Gestão de pagamentos | PK (id, negocio_id) | Via `gen_random_uuid()` | `pagamentos_pkey` | Realizada |
|  | `usuario_id` | uuid | ID do usuário vinculado ao pagamento | Vinculação ao usuário | FK `users.id` | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao pagamento | Vinculação ao negócio | PK (id, negocio_id), FK `negocios.id` | Obrigatório, particionado | - | Realizada |
|  | `valor` | numeric | Valor total do pagamento em reais | Gestão financeira | Nenhuma | Obrigatório, em BRL | - | Realizada |
|  | `forma_pagamento` | text | Método de pagamento (ex.: "pix") | Gestão de pagamentos | Nenhuma | Obrigatório | - | Realizada |
|  | `status` | text | Estado do pagamento (ex.: "pago") | Progresso do pagamento | Nenhuma | Valores predefinidos | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do pagamento | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_vencimento` | timestamp | Data e hora de vencimento do pagamento | Controle de prazos | Nenhuma | Obrigatório | - | Realizada |
|  | `quantidade_parcelas` | integer | Número de parcelas, se aplicável | Parcelamento | Nenhuma | Nullable, mínimo 1 se parcelado | - | Realizada |
|  | `valor_parcela` | numeric | Valor de cada parcela em reais | Parcelamento | Nenhuma | Nullable, calculado se parcelado | - | Realizada |
|  | `link_pagamento` | text | Link para pagamento | Integração com gateway | Nenhuma | Nullable | - | Realizada |
|  | `qr_code` | text | Código QR para pagamento | Integração com gateway | Nenhuma | Nullable | - | Realizada |
|  | `boleto_codigo` | text | Código do boleto | Integração com gateway | Nenhuma | Nullable | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **postagens_marketing** | `id` | uuid | Identificador único da postagem | Gestão de postagens | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `projeto_marketing_id` | uuid | ID do projeto de marketing vinculado | Vinculação à campanha | FK `projetos_marketing.id` | Obrigatório | - | Realizada |
|  | `data_horario` | timestamp | Data e hora programada da postagem | Agendamento | Nenhuma | Obrigatório | - | Realizada |
|  | `status` | text | Estado da postagem (ex.: "pendente") | Progresso | Nenhuma | CHECK ("pendente", "realizada", "programada") | - | Realizada |
| **projeto_alteracoes** | `id` | uuid | Identificador único da alteração | Auditoria de mudanças | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto alterado | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário que realizou a alteração | Auditoria | FK `users.id` | Obrigatório | - | Realizada |
|  | `acao` | text | Tipo de ação (ex.: "editar") | Registro de mudanças | Nenhuma | Valores predefinidos | - | Realizada |
|  | `detalhes` | jsonb | Dados detalhados da alteração | Detalhamento | Nenhuma | Estrutura JSON | - | Realizada |
|  | `data_alteracao` | timestamp | Data e hora da alteração | Histórico | Nenhuma | Default NOW() | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à alteração | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `versao` | integer | Versão da alteração | Controle de versões | Nenhuma | Incrementado por alteração | - | Realizada |
| **projetos** | `id` | uuid | Identificador único do projeto | Gestão de projetos | PK, ref. por várias tabelas | Via `gen_random_uuid()` | `idx_projetos_negocio_id_data_evento_horario` | Realizada |
|  | `nome` | text | Nome do projeto (ex.: "Casamento João") | Identificação do projeto | Nenhuma | Obrigatório | - | Realizada |
|  | `descricao` | text | Descrição detalhada do projeto | Detalhamento | Nenhuma | Nullable | - | Realizada |
|  | `tipo_evento` | text | Tipo de evento (ex.: "Casamento") | Categorização | Nenhuma | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao projeto | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `idx_projetos_negocio_id_data_evento_horario` (otimiza filtros por negócio e data) | Realizada |
|  | `etapa_id` | uuid | ID da etapa atual do projeto | Progresso | FK `etapas.id` | Obrigatório | - | Realizada |
|  | `data_evento` | timestamp | Data e hora do evento | Agendamento | Nenhuma | Obrigatório | `idx_projetos_negocio_id_data_evento_horario` (otimiza filtros por data) | Realizada |
|  | `horario_evento` | time | Horário específico do evento | Agendamento | Nenhuma | Nullable | `idx_projetos_negocio_id_data_evento_horario` (otimiza filtros por horário) | Realizada |
|  | `status_equipe` | text | Status da equipe (ex.: "pendente") | Gestão de equipe | Nenhuma | Valores predefinidos | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do projeto | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `equipe_ok` | boolean | Indica se a equipe está confirmada | Gestão de equipe | Nenhuma | Default `false` | - | Realizada |
|  | `local_evento` | text | Local do evento (ex.: "Salão X") | Detalhamento | Nenhuma | Nullable | - | Realizada |
|  | `status` | text | Estado do projeto (ex.: "em andamento") | Progresso | Nenhuma | Valores predefinidos | - | Realizada |
|  | `pacote_escolhido` | text | Nome do pacote escolhido | Associação a pacotes | Nenhuma | Nullable | - | Realizada |
|  | `favorito` | boolean | Indica se o projeto é favorito | Destaque | Nenhuma | Default `false` | - | Realizada |
|  | `bloqueia_agenda` | boolean | Indica se o projeto bloqueia a agenda | Controle de conflitos | Nenhuma | Default `false` | - | Realizada |
|  | `tipo_projeto_id` | uuid | ID do tipo de projeto | Categorização | FK `tipos_projeto.id` | Obrigatório | - | Realizada |
|  | `pacote_id` | uuid | ID do pacote associado | Associação a pacotes | FK `pacotes.id` | Nullable | - | Realizada |
|  | `config_recorrencia` | jsonb | Configuração de recorrência (ex.: {"intervalo": "mensal"}) | Projetos recorrentes | Nenhuma | Nullable, estrutura JSON | - | Realizada |
|  | `notificacao_antes` | integer | Dias antes do evento para notificação | Alertas | Nenhuma | Nullable | - | Realizada |
|  | `valor_total` | numeric | Valor total estimado do projeto | Gestão financeira | Nenhuma | Nullable | - | Realizada |
|  | `data_backup` | timestamp | Data e hora do último backup | Backup | Nenhuma | Nullable | - | Realizada |
|  | `importado_em` | timestamp | Data e hora da importação | Importação | Nenhuma | Nullable | - | Realizada |
|  | `valorpacote` | numeric | Valor estimado do pacote (camelCase no BD) | Previsão financeira | Nenhuma | Nullable, renomeado de `valorPacote` | - | Realizada |
|  | `postar` | boolean | Indica se o projeto será enviado ao marketing | Integração com marketing | Nenhuma | Default `false` | - | Realizada |
|  | usuarioId | string | pra identificar o criador do projeto |  |  |  |  | index feito  |
| **projetos_marketing** | `id` | uuid | Identificador único do projeto de marketing | Gestão de marketing | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto principal associado | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `data_horario` | timestamp | Data e hora programada da campanha | Agendamento | Nenhuma | Obrigatório | - | Realizada |
|  | `status` | text | Estado da campanha (ex.: "pendente") | Progresso | Nenhuma | Valores predefinidos | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **reservas_agendamento** | `id` | uuid | Identificador único da reserva | Gestão de reservas | PK (id, negocio_id) | Via `gen_random_uuid()` | `reservas_agendamento_pkey` | Realizada |
|  | `campanha_id` | uuid | ID da campanha associada à reserva | Vinculação à campanha | FK `campanhas_agendamento(id, negocio_id)` | Obrigatório | `idx_reservas_agendamento_campanha_id` (otimiza busca por campanha) | Realizada |

| **Tabela** | **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **reservas_agendamento(continuação)** | `data_horario` | timestamp | Data e hora reservada pelo cliente | Agendamento na landing page | Nenhuma | Obrigatório | `idx_reservas_agendamento_data_horario` (otimiza filtros por data) | Realizada |
|  | `cliente_nome` | text | Nome do cliente que fez a reserva | Identificação na reserva | Nenhuma | Obrigatório | - | Realizada |
|  | `cliente_email` | text | Email do cliente para comunicação | Comunicação com cliente | Nenhuma | Obrigatório | - | Realizada |
|  | `cliente_telefone` | text | Telefone do cliente (ex.: "+5511999999999") | Comunicação com cliente | Nenhuma | Nullable | - | Realizada |
|  | `valor_pago` | numeric | Valor pago na reserva em reais | Gestão financeira | Nenhuma | Obrigatório, em BRL | - | Realizada |
|  | `status` | text | Estado da reserva (ex.: "pendente", "pago") | Controle de reservas | Nenhuma | CHECK ("pendente", "pago", "concluido") | `idx_reservas_agendamento_status` (otimiza filtros por status) | Realizada |
|  | `pagamento_id` | uuid | ID do pagamento associado | Vinculação ao pagamento | FK `pagamentos(id, negocio_id)` | Nullable | - | Realizada |
|  | `data_reserva` | timestamp | Data e hora da criação da reserva | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `fotos_extras` | integer | Quantidade de fotos extras solicitadas | Pagamento pós-sessão | Nenhuma | Default 0 | - | Realizada |
|  | `valor_extras` | numeric | Valor total das fotos extras em reais | Pagamento pós-sessão | Nenhuma | Default 0 | - | Realizada |
|  | `cancelado` | boolean | Indica se a reserva foi cancelada | Controle de reservas | Nenhuma | Default `false` | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à reserva | Vinculação ao negócio | PK (id, negocio_id), FK `negocios.id` | Obrigatório, particionado | `idx_reservas_agendamento_negocio_id` (otimiza filtros por negócio) | Realizada |
| **seguranca_eventos** | `id` | uuid | Identificador único do evento de segurança | Monitoramento de segurança | PK (id, negocio_id) | Via `gen_random_uuid()` | `seguranca_eventos_pkey` | Realizada |
|  | `tipo` | text | Tipo de evento (ex.: "login_falhou") | Classificação | Nenhuma | Obrigatório | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário relacionado ao evento | Vinculação ao usuário | FK `users.id` | Nullable | - | Realizada |
|  | `ip_origem` | text | Endereço IP de origem (ex.: "192.168.1.1") | Auditoria de segurança | Nenhuma | Obrigatório | - | Realizada |
|  | `data_evento` | timestamp | Data e hora do evento | Histórico | Nenhuma | Default NOW() | - | Realizada |
|  | `detalhes` | jsonb | Dados detalhados do evento (ex.: {"motivo": "senha errada"}) | Detalhamento | Nenhuma | Estrutura JSON | - | Realizada |
|  | `nivel_risco` | text | Nível de risco (ex.: "baixo", "alto") | Avaliação | Nenhuma | CHECK ("baixo", "médio", "alto") | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio relacionado ao evento | Vinculação ao negócio | PK (id, negocio_id), FK `negocios.id` | Obrigatório, particionado | - | Realizada |
| **sincronizacoes** | `id` | uuid | Identificador único da sincronização | Sincronização de dados | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `tabela` | text | Nome da tabela sincronizada (ex.: "projetos") | Identificação da origem | Nenhuma | Obrigatório | - | Realizada |
|  | `ultima_modificacao` | timestamp | Data e hora da última modificação | Histórico | Nenhuma | Default NOW() | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário que realizou a sincronização | Auditoria | FK `users.id` | Obrigatório | - | Realizada |
|  | `tipo_alteracao` | text | Tipo de alteração (ex.: "inserção") | Detalhamento | Nenhuma | Valores predefinidos | - | Realizada |
|  | `dados_anteriores` | jsonb | Dados antes da alteração | Histórico | Nenhuma | Nullable, estrutura JSON | - | Realizada |
|  | `dados_novos` | jsonb | Dados após a alteração | Histórico | Nenhuma | Nullable, estrutura JSON | - | Realizada |
| **sprints** | `id` | uuid | Identificador único do sprint | Gestão de sprints | PK, ref. por `tarefas.sprint_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do sprint (ex.: "Sprint 1") | Identificação | Nenhuma | Obrigatório | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto vinculado ao sprint | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do sprint | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `ordem` | smallint | Ordem de exibição do sprint | Sequência | Nenhuma | Obrigatório | - | Realizada |
| **sprints_padrao** | `id` | uuid | Identificador único do sprint padrão | Templates de sprints | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do sprint padrão (ex.: "Sprint Básico") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **sprints_template** | `id` | uuid | Identificador único do template de sprint | Templates de sprints | PK, ref. por `tarefas_template.sprint_template_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do template (ex.: "Sprint Padrão") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao template | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `responsavel_id` | uuid | ID do usuário responsável pelo template | Gestão de responsáveis | FK `users.id` | Nullable | - | Realizada |
| **sprints_template_padrao** | `id` | uuid | Identificador único do template padrão global | Templates padrão | PK, ref. por `tarefas_template_padrao.sprint_template_padrao_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do template (ex.: "Sprint Básico") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **tags** | `id` | uuid | Identificador único da tag | Gestão de tags | PK, ref. por `tags_projetos.tag_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome da tag (ex.: "Urgente") | Identificação | Nenhuma | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à tag | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação da tag | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `cor` | text | Código de cor da tag (ex.: "#FF0000") | Exibição visual | Nenhuma | Obrigatório | - | Realizada |
| **tags_projetos** | `id` | uuid | Identificador único da associação tag-projeto | Vinculação de tags a projetos | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto associado à tag | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `tag_id` | uuid | ID da tag associada ao projeto | Vinculação à tag | FK `tags.id` | Obrigatório | - | Realizada |
| **tarefas** | `id` | uuid | Identificador único da tarefa | Gestão de tarefas | PK, ref. por `compromissos.tarefa_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome da tarefa (ex.: "Editar Fotos") | Identificação | Nenhuma | Obrigatório | - | Realizada |
|  | `sprint_id` | uuid | ID do sprint vinculado à tarefa | Vinculação ao sprint | FK `sprints.id` | Obrigatório | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto vinculado à tarefa | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `subtarefa_de` | uuid | ID da tarefa pai, se subtarefa | Hierarquia | FK `tarefas.id` | Nullable | - | Realizada |
|  | `concluida` | boolean | Indica se a tarefa foi concluída | Progresso | Nenhuma | Default `false` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação da tarefa | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `ordem` | smallint | Ordem de exibição da tarefa | Sequência | Nenhuma | Obrigatório | - | Realizada |
|  | `data_conclusao` | timestamp with time zone | Data e hora de conclusão da tarefa | Histórico | Nenhuma | Nullable | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `data_vencimento` | timestamp | Data e hora de vencimento da tarefa | Controle de prazos | Nenhuma | Nullable | - | Realizada |
| **tarefas_padrao** | `id` | uuid | Identificador único da tarefa padrão | Templates de tarefas | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `sprint_padrao_id` | uuid | ID do sprint padrão vinculado | Vinculação ao sprint | FK `sprints_padrao.id` | Obrigatório | - | Realizada |
|  | `nome` | text | Nome da tarefa padrão (ex.: "Planejar Sessão") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `ordem` | smallint | Ordem de exibição no template | Sequência | Nenhuma | Obrigatório | - | Realizada |
|  | `concluida` | boolean | Indica se a tarefa padrão está concluída | Progresso | Nenhuma | Default `false` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **tarefas_template** | `id` | uuid | Identificador único da tarefa no template | Templates de tarefas | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome da tarefa (ex.: "Revisar Edição") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `sprint_template_id` | uuid | ID do template de sprint vinculado | Vinculação ao template | FK `sprints_template.id` | Obrigatório | - | Realizada |
|  | `subtarefa_de` | uuid | ID da tarefa pai, se subtarefa | Hierarquia | FK `tarefas_template.id` | Nullable | - | Realizada |
|  | `concluida` | boolean | Indica se a tarefa está concluída | Progresso | Nenhuma | Default `false` | - | Realizada |
|  | `ordem` | smallint | Ordem de exibição no template | Sequência | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `responsavel_id` | uuid | ID do usuário responsável | Gestão de responsáveis | FK `users.id` | Nullable | - | Realizada |
| **tarefas_template_padrao** | `id` | uuid | Identificador único da tarefa padrão global | Templates padrão | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `sprint_template_padrao_id` | uuid | ID do template padrão global vinculado | Vinculação ao template | FK `sprints_template_padrao.id` | Obrigatório | - | Realizada |
|  | `nome` | text | Nome da tarefa (ex.: "Planejar Sessão") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `subtarefa_de` | uuid | ID da tarefa pai, se subtarefa | Hierarquia | FK `tarefas_template_padrao.id` | Nullable | - | Realizada |
|  | `concluida` | boolean | Indica se a tarefa está concluída | Progresso | Nenhuma | Default `false` | - | Realizada |
|  | `ordem` | smallint | Ordem de exibição no template | Sequência | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `responsavel_id` | uuid | ID do usuário responsável | Gestão de responsáveis | FK `users.id` | Nullable | - | Realizada |
| **tipos_projeto** | `id` | uuid | Identificador único do tipo de projeto | Gestão de tipos | PK, ref. por `pacotes.tipo_projeto_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do tipo (ex.: "Casamento") | Identificação | Nenhuma | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao tipo | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do tipo | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **tipos_projeto_padrao** | `id` | uuid | Identificador único do tipo padrão | Templates de tipos | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do tipo padrão (ex.: "Evento Corporativo") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| **tipos_projeto_template** | `id` | uuid | Identificador único do template de tipo | Templates de tipos | PK, ref. por `pacotes_template.tipo_projeto_template_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do template (ex.: "Casamento Luxo") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao template | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `ativo` | boolean | Indica se o template está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **tipos_projeto_template_padrao** | `id` | uuid | Identificador único do template padrão global | Templates padrão | PK, ref. por `pacotes_template_padrao.tipo_projeto_template_id` | Via `gen_random_uuid()` | - | Realizada |
|  | `nome` | text | Nome do template (ex.: "Ensaio Simples") | Identificação no template | Nenhuma | Obrigatório | - | Realizada |
|  | `ativo` | boolean | Indica se o template está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |
|  | `data_criacao` | timestamp | Data e hora de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
| **uploads** | `id` | uuid | Identificador único do arquivo | Gestão de arquivos | PK | Via `gen_random_uuid()` | - | Realizada |
|  | `usuario_id` | uuid | ID do usuário que fez o upload | Vinculação ao autor | FK `users.id` | Obrigatório | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado ao arquivo | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `projeto_id` | uuid | ID do projeto associado ao arquivo | Vinculação ao projeto | FK `projetos.id` | Obrigatório | - | Realizada |
|  | `nome_arquivo` | text | Nome do arquivo (ex.: "foto1.jpg") | Identificação | Nenhuma | Obrigatório | - | Realizada |
|  | `tipo_arquivo` | text | Tipo ou extensão (ex.: "jpg") | Classificação | Nenhuma | Obrigatório | - | Realizada |
|  | `caminho` | text | Caminho ou URL do arquivo (ex.: "s3://bucket/foto1.jpg") | Acesso ao arquivo | Nenhuma | Obrigatório | - | Realizada |
|  | `tamanho_bytes` | integer | Tamanho do arquivo em bytes | Gestão de armazenamento | Nenhuma | Obrigatório | - | Realizada |
|  | `data_upload` | timestamp | Data e hora do upload | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `ativo` | boolean | Indica se o arquivo está ativo | Controle de visibilidade | Nenhuma | Default `true` | - | Realizada |
| **users** | `id` | uuid | Identificador único do usuário | Autenticação e gestão | PK, ref. por várias tabelas | Via Supabase Auth | - | Realizada |
|  | `nome_artistico` | text | Nome artístico ou profissional (ex.: "João Fotógrafo") | Identificação no perfil | Nenhuma | Obrigatório | - | Realizada |
|  | `email` | text | Email do usuário para login | Autenticação | Nenhuma | UNIQUE | - | Realizada |
|  | `foto_perfil` | text | URL da foto de perfil | Personalização | Nenhuma | Nullable | - | Realizada |
|  | `titulo_funcao` | text | Título ou função (ex.: "Fotógrafo Principal") | Identificação | Nenhuma | Nullable | - | Realizada |
|  | `negocio_id` | uuid | ID do negócio principal do usuário | Vinculação ao negócio | FK `negocios.id` | Obrigatório | - | Realizada |
|  | `criado_em` | timestamp | Data e hora de criação do usuário | Auditoria | Nenhuma | Default NOW() | - | Realizada |
|  | `atualizado_em` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `role` | text | Nível de acesso (ex.: "admin") | Controle de permissões | Nenhuma | Valores predefinidos | - | Realizada |
|  | `codigo_convite` | text | Código para convidar o usuário | Convites | Nenhuma | Nullable | - | Realizada |
|  | `assinatura_id` | uuid | ID da assinatura associada | Gestão de assinaturas | FK `assinaturas.id` | Nullable | - | Realizada |
|  | `acesso_modulos` | jsonb | Permissões de módulos (ex.: {"sprints": "total"}) | Controle de permissões | Nenhuma | Estrutura JSON | - | Realizada |
|  | `proprietario` | boolean | Indica se o usuário é proprietário | Gestão de negócios | Nenhuma | Default `false` | - | Realizada |
|  | `assinatura_responsavel` | uuid | ID do usuário responsável pela assinatura | Gestão de assinaturas | FK `users.id` | Nullable | - | Realizada |
|  | `ultimo_acesso` | timestamp | Data e hora do último acesso | Monitoramento | Nenhuma | Nullable | - | Realizada |
| **vagas_campanha** | `id` | uuid | Identificador único da vaga | Gestão de vagas | PK (id, negocio_id) | Via `gen_random_uuid()` | `vagas_campanha_pkey` | Realizada |
|  | `campanha_id` | uuid | ID da campanha associada à vaga | Vinculação à campanha | FK `campanhas_agendamento(id, negocio_id)` | Obrigatório | `idx_vagas_campanha_campanha_id` (otimiza busca por campanha) | Realizada |
|  | `negocio_id` | uuid | ID do negócio vinculado à vaga | Vinculação ao negócio | PK (id, negocio_id), FK `negocios.id` | Obrigatório, particionado | `idx_vagas_campanha_negocio_id` (otimiza filtros por negócio) | Realizada |
|  | `data_horario` | timestamp | Data e hora da vaga | Agendamento | Nenhuma | Obrigatório | `idx_vagas_campanha_data_horario` (otimiza filtros por data) | Realizada |
|  | `status` | text | Estado da vaga (ex.: "disponivel") | Controle de vagas | Nenhuma | Valores predefinidos | `idx_vagas_campanha_status` (otimiza filtros por status) | Realizada |
|  | `usuario_id` | uuid | ID do usuário que reservou a vaga | Vinculação ao usuário | FK `users.id` | Nullable | - | Realizada |
|  | `data_atualizacao` | timestamp | Data e hora da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza ao editar | - | Realizada |
|  | `pagamento_id` | uuid | ID do pagamento associado | Vinculação ao pagamento | FK `pagamentos(id, negocio_id)` | Nullable | - | Realizada |
|  | `pagamento_negocio_id` | uuid | ID do negócio do pagamento (parte da FK composta) | Vinculação ao pagamento | FK `pagamentos(id, negocio_id)` | Nullable, usado com `pagamento_id` | - | Realizada |
|  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |

### **`projeto_alteracoes`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único da alteração | Auditoria de projetos | PK | Via `gen_random_uuid()` | - | Realizada |
| `projeto_id` | uuid | ID do projeto alterado | Vinculação ao projeto | FK `projetos.id` | Obrigatório | `idx_projeto_alteracoes_projeto_id` | Realizada |
| `usuario_id` | uuid | ID do usuário que fez a alteração | Auditoria | FK `users.id` | Obrigatório | `idx_projeto_alteracoes_usuario_id` | Realizada |
| `acao` | text | Tipo de ação (ex.: "editar", "excluir") | Detalhamento da ação | Nenhuma | Obrigatório | - | Realizada |
| `detalhes` | jsonb | Dados antes/depois da alteração | Registro detalhado | Nenhuma | Estrutura JSON, flexível | - | Realizada |
| `data_alteracao` | timestamp | Data e hora da alteração | Histórico | Nenhuma | Default NOW() | `idx_projeto_alteracoes_data_alteracao` | Realizada |
| `negocio_id` | uuid | ID do negócio relacionado | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `idx_projeto_alteracoes_negocio_id` | Realizada |
| `versao` | integer | Versão da alteração | Controle de versão | Nenhuma | Obrigatório, incremental | - | Realizada |

### **`sincronizacoes`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único da sincronização | Gestão de sincronizações | PK | Via `gen_random_uuid()` | - | Realizada |
| `tabela` | text | Nome da tabela sincronizada | Identificação | Nenhuma | Obrigatório (ex.: "projetos") | - | Realizada |
| `ultima_modificacao` | timestamp | Data da última modificação | Controle de sincronização | Nenhuma | Default NOW() | `idx_sincronizacoes_ultima_modificacao` | Realizada |
| `usuario_id` | uuid | ID do usuário que sincronizou | Vinculação ao usuário | FK `users.id` | Obrigatório | `idx_sincronizacoes_usuario_id` | Realizada |
| `tipo_alteracao` | text | Tipo de alteração (ex.: "insert") | Detalhamento | Nenhuma | Obrigatório | - | Realizada |
| `dados_anteriores` | jsonb | Dados antes da alteração | Registro detalhado | Nenhuma | Nullable, JSON | - | Realizada |
| `dados_novos` | jsonb | Dados após a alteração | Registro detalhado | Nenhuma | Nullable, JSON | - | Realizada |

### **`sprints`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único do sprint | Gestão de sprints | PK | Via `gen_random_uuid()` | - | Realizada |
| `nome` | text | Nome do sprint | Identificação | Nenhuma | Obrigatório | - | Realizada |
| `projeto_id` | uuid | ID do projeto vinculado | Vinculação ao projeto | FK `projetos.id` | Obrigatório | `idx_sprints_projeto_id` | Realizada |
| `data_criacao` | timestamp | Data de criação do sprint | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| `data_atualizacao` | timestamp | Data da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza | `idx_sprints_data_atualizacao` | Realizada |
| `ordem` | smallint | Ordem de exibição | Sequência | Nenhuma | Obrigatório | - | Realizada |

**`tags`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único da tag | Gestão de tags | PK | Via `gen_random_uuid()` | - | Realizada |
| `nome` | text | Nome da tag | Identificação | Nenhuma | Obrigatório | - | Realizada |
| `negocio_id` | uuid | ID do negócio vinculado | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `idx_tags_negocio_id` | Realizada |
| `data_criacao` | timestamp | Data de criação da tag | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| `data_atualizacao` | timestamp | Data da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza | `idx_tags_data_atualizacao` | Realizada |
| `cor` | text | Código de cor (ex.: "#FF5733") | Exibição visual | Nenhuma | Obrigatório | - | Realizada |

### **`tags_projetos`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único da associação | Vinculação tag-projeto | PK | Via `gen_random_uuid()` | - | Realizada |
| `projeto_id` | uuid | ID do projeto associado | Vinculação ao projeto | FK `projetos.id` | Obrigatório | `idx_tags_projetos_projeto_id` | Realizada |
| `tag_id` | uuid | ID da tag associada | Vinculação à tag | FK `tags.id` | Obrigatório | `idx_tags_projetos_tag_id` | Realizada |

### **`tarefas`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único da tarefa | Gestão de tarefas | PK | Via `gen_random_uuid()` | - | Realizada |
| `nome` | text | Nome da tarefa | Identificação | Nenhuma | Obrigatório | - | Realizada |
| `sprint_id` | uuid | ID do sprint vinculado | Vinculação ao sprint | FK `sprints.id` | Obrigatório | `idx_tarefas_sprint_id` | Realizada |
| `projeto_id` | uuid | ID do projeto vinculado | Vinculação ao projeto | FK `projetos.id` | Obrigatório | `idx_tarefas_projeto_id` | Realizada |
| `subtarefa_de` | uuid | ID da tarefa pai (se subtarefa) | Hierarquia | FK `tarefas.id` | Nullable | `idx_tarefas_subtarefa_de` | Realizada |
| `concluida` | boolean | Indica se a tarefa foi concluída | Status | Nenhuma | Default `false` | - | Realizada |
| `data_criacao` | timestamp | Data de criação da tarefa | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| `ordem` | smallint | Ordem de exibição | Sequência | Nenhuma | Obrigatório | - | Realizada |
| `data_conclusao` | timestamp | Data de conclusão | Histórico | Nenhuma | Nullable | `idx_tarefas_data_conclusao` | Realizada |
| `data_atualizacao` | timestamp | Data da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza | `idx_tarefas_data_atualizacao` | Realizada |
| `data_vencimento` | timestamp | Data de vencimento | Controle de prazos | Nenhuma | Nullable | `idx_tarefas_data_vencimento` | Realizada |

### **`tipos_projeto`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único do tipo | Gestão de tipos | PK | Via `gen_random_uuid()` | - | Realizada |
| `nome` | text | Nome do tipo de projeto | Identificação | Nenhuma | Obrigatório | - | Realizada |
| `negocio_id` | uuid | ID do negócio vinculado | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `idx_tipos_projeto_negocio_id` | Realizada |
| `data_criacao` | timestamp | Data de criação do tipo | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| `data_atualizacao` | timestamp | Data da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza | `idx_tipos_projeto_data_atualizacao` | Realizada |

### **`tipos_projeto_template`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único do template | Templates de tipos | PK | Via `gen_random_uuid()` | - | Realizada |
| `nome` | text | Nome do template | Identificação | Nenhuma | Obrigatório | - | Realizada |
| `negocio_id` | uuid | ID do negócio vinculado | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `idx_tipos_projeto_template_negocio_id` | Realizada |
| `ativo` | boolean | Indica se o template está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |
| `data_criacao` | timestamp | Data de criação do template | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| `data_atualizacao` | timestamp | Data da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza | `idx_tipos_projeto_template_data_atualizacao` | Realizada |

### **`uploads`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único do upload | Gestão de uploads | PK | Via `gen_random_uuid()` | - | Realizada |
| `usuario_id` | uuid | ID do usuário que fez o upload | Vinculação ao usuário | FK `users.id` | Obrigatório | `idx_uploads_usuario_id` | Realizada |
| `negocio_id` | uuid | ID do negócio vinculado | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `idx_uploads_negocio_id` | Realizada |
| `projeto_id` | uuid | ID do projeto vinculado | Vinculação ao projeto | FK `projetos.id` | Nullable | `idx_uploads_projeto_id` | Realizada |
| `nome_arquivo` | text | Nome do arquivo | Identificação | Nenhuma | Obrigatório | - | Realizada |
| `tipo_arquivo` | text | Tipo do arquivo (ex.: "image/jpeg") | Detalhamento | Nenhuma | Obrigatório | - | Realizada |
| `caminho` | text | URL ou caminho do arquivo | Localização | Nenhuma | Obrigatório | - | Realizada |
| `tamanho_bytes` | integer | Tamanho do arquivo em bytes | Detalhamento | Nenhuma | Obrigatório | - | Realizada |
| `data_upload` | timestamp | Data do upload | Auditoria | Nenhuma | Default NOW() | `idx_uploads_data_upload` | Realizada |
| `ativo` | boolean | Indica se o upload está ativo | Controle de disponibilidade | Nenhuma | Default `true` | - | Realizada |

### **`users`**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único do usuário | Gestão de usuários | PK | Via `gen_random_uuid()` | - | Realizada |
| `nome_artistico` | text | Nome artístico do usuário | Identificação | Nenhuma | Nullable | - | Realizada |
| `email` | text | Email do usuário | Autenticação | Nenhuma | Obrigatório, UNIQUE | `idx_users_email` | Realizada |
| `foto_perfil` | text | URL da foto de perfil | Personalização | Nenhuma | Nullable | - | Realizada |
| `titulo_funcao` | text | Título ou função (ex.: "Fotógrafo") | Identificação | Nenhuma | Nullable | - | Realizada |
| `negocio_id` | uuid | ID do negócio principal | Vinculação ao negócio | FK `negocios.id` | Obrigatório | `idx_users_negocio_id` | Realizada. NegocioId (UUID) - Vincula ao negócio inicial criado no signup, ajustado em 19/03/2025. |
| `criado_em` | timestamp | Data de criação do usuário | Auditoria | Nenhuma | Default NOW() | - | Realizada |
| `atualizado_em` | timestamp | Data da última atualização | Auditoria | Nenhuma | Default NOW(), atualiza | `idx_users_atualizado_em` | Realizada |
| `role` | text | Papel do usuário (ex.: "admin") | Controle de acesso | Nenhuma | Default "user" | - | Realizada |
| `codigo_convite` | text | Código de convite para membros | Gestão de convites | Nenhuma | Nullable | - | Realizada |
| `assinatura_id` | uuid | ID da assinatura do usuário | Vinculação à assinatura | FK `assinaturas.id` | Nullable | `idx_users_assinatura_id` | Realizada |
| `acesso_modulos` | jsonb | Permissões por módulo | Controle de acesso | Nenhuma | Estrutura JSON, flexível | - | Realizada |
| `proprietario` | boolean | Indica se é proprietário do negócio | Gestão de propriedade | Nenhuma | Default `false` | - | Realizada |
| `assinatura_responsavel` | uuid | ID da assinatura responsável | Vinculação à assinatura | FK `assinaturas.id` | Nullable | - | Realizada |
| `ultimo_acesso` | timestamp | Data do último acesso | Monitoramento | Nenhuma | Nullable | `idx_users_ultimo_acesso` | Realizada |
|  |  |  |  |  |  |  |  |

---

---

### Atualização do Documento 2 (Arquitetura de Dados)

### **Tabela `negocios_membros` (Atualizada)**

| **Coluna** | **Tipo de Dado** | **Descrição** | **Contexto** | **Relações** | **Notas** | **Índices** | **Implementação** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | uuid | Identificador único da associação usuário-negócio | Gestão de membros em negócios | PK | Gerado via `gen_random_uuid()` | - | Realizada |
| `negocio_id` | uuid | ID do negócio associado | Vinculação ao negócio | FK `negocios.id`, obrigatório | Identifica o negócio ao qual o usuário está associado | `idx_negocios_membros_negocio_id_usuario_id` (UNIQUE) | Realizada |
| `usuario_id` | uuid | ID do usuário associado | Vinculação ao usuário | FK `users.id` | Identifica o usuário membro do negócio; pode ser `null` para convites pendentes | `idx_negocios_membros_negocio_id_usuario_id` (UNIQUE) | Realizada; ajustado para permitir `null` (21/03/2025) |
| `is_proprietario` | boolean | Indica se o usuário é proprietário do negócio | Controle de propriedade | Nenhuma | Default `false`, `true` para o criador do negócio | - | Realizada |
| `acesso_modulos` | text[] | Lista de permissões por módulo (ex.: "total", "view-only") | Controle de acesso | Nenhuma | Default `ARRAY['total']::TEXT[]`, suporta multi-negócios e herança | - | Realizada |
| `data_associacao` | timestamp | Data e hora da associação ao negócio | Auditoria | Nenhuma | Default `NOW()`, registra quando o usuário foi associado | - | Realizada |
| `plano_atual` | text | Plano herdado do proprietário (ex.: "expert_pro") | Herança de plano | Nenhuma | Calculado via herança do plano do proprietário, suporta "Expert Pro" | - | Realizada |
| `nome` | text | Nome temporário do convidado (email) até o registro | Gestão de convites | Nenhuma | Usado para convites pendentes, preenchido com o email do convidado | - | Adicionado (20/03/2025) |
| `status` | text | Estado do convite (ex.: "convite_enviado", "associado") | Gestão de convites | Nenhuma | Indica o estado do convite ou associação | - | Adicionado (20/03/2025) |
| `invite_code` | text | Código único do convite | Gestão de convites | Nenhuma | UNIQUE, usado para associar o convite ao usuário após o cadastro | `idx_negocios_membros_invite_code` | Adicionado (20/03/2025); índice adicionado (21/03/2025) |
| - | - | - | - | - | Constraint `check_usuario_id_for_associado`: Garante que `usuario_id` seja preenchido para registros com `status = 'associado'` | - | Adicionada (21/03/2025) |
|  |  |  |  |  |  |  |  |

### **Remoção da Tabela `convites_pendentes`**

---

## **Log de Alterações Banco de Dados**

- Criadas tabelas: campanhas_agendamento, vagas_campanha, fotos_extras, configuracoes_sistema, importacoes_agenda, seguranca_eventos, pagamentos, projetos_marketing, contratos, negocio_detalhes, formularios_clientes, postagens_marketing, sprints_template, tarefas_template, sprints_template_padrao, tarefas_template_padrao, reservas_agendamento, compromissos, admin_logs, metricas_sistema, custos_sistema (16/03/2025).
- Adicionadas tabelas pré-existentes ao DEV: projeto_alteracoes, sincronizacoes, sprints, tags, tags_projetos, tarefas, tipos_projeto, tipos_projeto_template, uploads, users (17/03/2025), completando 43 tabelas operacionais.
- Adicionados campos pendentes: primeiro_acesso, aceitou_termos, aceitou_lgpd, aviso_cookies, fuso_horario, regiao, moeda em configuracoes_usuario; regiao em negocios; postar em projetos; ordem em sprints; data_vencimento em tarefas; acesso_modulos, proprietario, assinatura_responsavel, ultimo_acesso em users; valor_pacote (como valorpacote) em pacotes (16/03/2025).
- Implementado particionamento por negocio_id (p0-p3) em: campanhas_agendamento, vagas_campanha, fotos_extras, importacoes_agenda, seguranca_eventos, pagamentos, reservas_agendamento (16/03/2025).
- Ajustada PK composta (id, negocio_id) em campanhas_agendamento, pagamentos, e FKs relacionadas em vagas_campanha, fotos_extras, reservas_agendamento (16/03/2025).
- Ajustada FK composta em fotos_extras (pagamento_id, pagamento_negocio_id) para referenciar pagamentos(id, negocio_id) (17/03/2025), corrigindo erro de integridade.
- Adicionados índices: idx_campanhas_agendamento_negocio_id, idx_vagas_campanha_status, idx_projeto_alteracoes_projeto_id, idx_sincronizacoes_ultima_modificacao, idx_sprints_projeto_id, idx_tags_negocio_id, idx_tarefas_sprint_id, idx_tipos_projeto_negocio_id, idx_uploads_projeto_id, idx_users_negocio_id (16/03/2025 e 17/03/2025).
- Inseridos valores iniciais em configuracoes_sistema: zapSign_free_markup = 3.99, impay_free_markup = 3.00, impay_premium_markup = 1.49, impay_pro_markup = 0.79, zapsign_premium_markup = 1.99, zapsign_pro_markup = 1.77 (16/03/2025 e 17/03/2025).
- Corrigida referência de marketing para projetos_marketing e postagens_marketing (16/03/2025).
- Adicionado trigger log_sync_pagamentos_vagas em vagas_campanha como fallback (16/03/2025), registrando inconsistências em seguranca_eventos.
- **Criada tabela negocios_membros** com colunas: id (uuid, PK), negocio_id (uuid, FK), usuario_id (uuid, FK), is_proprietario (boolean), acesso_modulos (TEXT[]), data_associacao (timestamp), plano_atual (text) (19/03/2025). Índice idx_negocios_membros_negocio_id_usuario_id (UNIQUE) adicionado para otimizar consultas por negocio_id e usuario_id.
- **Ajustado campo acesso_modulos** em negocios de text para TEXT[] (19/03/2025), suportando multi-negócios e permissões detalhadas.
- **Adicionado campo data_atualizacao** em assinaturas (timestamp) (19/03/2025), para controle de sincronização.
- **Ajustado campo negocio_id** em users para vincular ao negócio inicial criado no signup (19/03/2025).
- **Ajustado campo negocio_id** em postagens_marketing para suportar multi-negócios, preenchido via projetos_marketing e mantido por trigger (19/03/2025).
