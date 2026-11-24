To-Do List TuttiPet

1. Configuração Base e Infraestrutura (Setup Inicial)

[ ] Repositório & Estrutura de Pastas:

[ ] Criar estrutura base do projeto (/backend em Python/Flask ou Node.js + /frontend em React + TailwindCSS).

[ ] Configurar ficheiro .env (nunca fazer commit de credenciais) com as chaves do SAP, Moloni, Supabase e Stripe.

[ ] Configurar CORS no backend para permitir pedidos do frontend (http://localhost:3000).

[ ] Base de Dados Interna (Supabase / PostgreSQL):

[ ] Criar tabelas mestras e operacionais: Cliente, Fornecedor, Funcionario, Categoria, Produto, Servico, Armazem, Stock_Armazem, Marcacao, Fatura, Item_Encomenda e Compra_Fornecedor.

[ ] Garantir colunas de mapeamento de IDs externos (sap_bp_id, moloni_customer_id, sap_supplier_id, moloni_invoice_id).

[ ] Popular dados iniciais de teste (categorias, funcionários com especialidades, serviços de Pet Sitting e armazém principal).

[ ] Postman (Fecho de Validação de Endpoints):

[x] S01 / S03: Obter CSRF Token e Criar Cliente (Draft) no SAP.

[x] M01 / M02 / M03: Autenticação OAuth2, Listar e Criar Clientes no Moloni.

[ ] M04 / M05: Listar e Criar Artigos/Serviços no Moloni.

[ ] S02: Listar Business Partners no SAP com filtro por NIF (TaxNumber1).

[ ] SAP Produtos: Testar GET em API_PRODUCT_SRV para extração de catálogo.

[ ] Moloni Faturação: Testar POST /invoiceReceipts/insert para emissão de Fatura-Recibo.

2. Backend & Middleware (API + Orquestração n8n)

Módulo 1: Clientes e Autenticação (UC01)

[ ] POST /api/auth/signup (UC1.1):

[ ] Validar campos obrigatórios e aplicar algoritmo de verificação de 9 dígitos ao NIF.

[ ] Verificar duplicação de NIF no Supabase, SAP e Moloni.

[ ] Orquestrar criação no SAP (MD_BUSINESSPARTNER_SRV com gestão automática de X-CSRF-Token) e replicação no Moloni (/customers/insert).

[ ] Encriptar password (bcrypt) e guardar registo completo no Supabase.

[ ] POST /api/auth/signin:

[ ] Autenticar cliente/funcionário/gestor/admin e devolver token de sessão e perfil (role).

Módulo 2: Catálogo, Marcações e Checkout (UC01 & UC02)

[ ] GET /api/catalog (UC1.2):

[ ] Listar produtos e serviços ativos, cruzando dados do Supabase com o stock atualizado do Moloni.

[ ] Suportar filtros por categoria, pesquisa textual e ordenação por preço.

[ ] POST /api/bookings (UC2.1):

[ ] Verificar conflitos de horário e disponibilidade de funcionários na tabela Marcacao para a data/hora escolhida.

[ ] Alocar funcionário disponível (cod_func), criar marcação com estado "Pendente de Pagamento" e implementar libertação automática após 15 min caso não seja paga.

[ ] POST /api/checkout (UC2.2):

[ ] Criar intenção de pagamento via Stripe API e validar confirmação (stripe_payment_id).

[ ] Disparar emissão de Fatura-Recibo no Moloni (/invoiceReceipts/insert) com o stripe_payment_id nas observações.

[ ] Dar baixa de stock, guardar Fatura + Item_Encomenda no Supabase e alterar estado das marcações associadas para "Confirmado".

[ ] Configurar fila de reprocessamento (Error Trigger no n8n) caso o Moloni falhe após pagamento confirmado.

[ ] GET /api/client/history & GET /api/invoices/:id/pdf (UC2.3):

[ ] Devolver histórico de marcações e faturas do cliente.

[ ] Permitir cancelamento de marcações futuras (até 24h antes) e download do PDF da fatura do Moloni.

Módulo 3 & 4: Operações de Equipa e Gestão (UC03 & UC04)

[ ] GET /api/staff/schedule & PATCH /api/bookings/:id/status (UC3.1, UC3.2):

[ ] Devolver agenda personalizada por cod_func e permitir transição de estado (Agendado → Concluído).

[ ] PATCH /api/manager/bookings/:id/reassign (UC4.1):

[ ] Validar ausência de conflitos de horário e permitir ao gestor reatribuir funcionários.

[ ] POST /api/manager/purchases (UC4.2):

[ ] Detetar alertas de stock_minimo, criar pedido Compra_Fornecedor (associado a fornecedor sincronizado SAP BBP000 ↔ Moloni) e atualizar stock na receção.

[ ] GET /api/manager/dashboard (UC4.3):

[ ] Agregar métricas de vendas, serviços mais requisitados, taxa de ocupação e estado das faturas.

Módulo 5: Administração e Sincronização ERP (UC05)

[ ] POST /api/admin/sync-catalog (UC5.1):

[ ] Extrair materiais do SAP (API_PRODUCT_SRV), transformar referências/categorias/preços e sincronizar com Moloni (/products/insert) e Supabase.

[ ] POST /api/admin/services (UC5.2):

[ ] Criar/editar serviços próprios e sincronizar como artigo faturável no Moloni (bloqueando eliminação se houver marcações ativas).

[ ] GET /api/admin/logs & POST /api/admin/retry/:log_id (UC5.3):

[ ] Expor logs de execução do middleware/n8n e permitir reprocessamento manual em caso de falha ou expiração de token.

Módulo 6: Chatbot Inteligente (UC06)

[ ] POST /api/chatbot/webhook (UC6.1 a UC6.4):

[ ] Configurar agente AI no n8n ligado às ferramentas do backend para:

[ ] Verificar vagas e pré-agendar serviços em linguagem natural (UC6.1).

[ ] Pesquisar produtos e validar stock em tempo real (UC6.2).

[ ] Consultar estado de encomendas/marcações e enviar link de PDF da fatura (UC6.3).

[ ] Responder a alertas de stock mínimo no backoffice (UC6.4).

3. Frontend (React + TailwindCSS)

Ecrãs Públicos e do Cliente (UC01, UC02, UC06)

[ ] Layout Base & Navegação: Header responsivo (Loja, Serviços, Carrinho com contador, Perfil/Login) + Widget flutuante do Chatbot no canto inferior direito.

[ ] Autenticação (/login & /register — UC1.1): Formulário de registo com validação visual de NIF e feedback de sincronização.

[ ] Catálogo & Detalhes (/shop & /product/:id — UC1.2): Grelha de produtos/serviços, barra de pesquisa, filtros por categoria e badge "Esgotado" quando stock = 0.

[ ] Agendamento de Serviço (/booking/:serviceId — UC2.1): Seletor de data/hora, morada e sugestão automática de horários alternativos em caso de indisponibilidade.

[ ] Carrinho & Checkout (/cart & /checkout — UC2.2): Resumo misto (produtos + marcações), cálculo de IVA, integração com Stripe Elements e ecrã de confirmação com download imediato da Fatura-Recibo.

[ ] Área Pessoal (/account — UC2.3): Separadores para "Minhas Marcações" (com botão de cancelar) e "Minhas Faturas" (com botão de descarregar PDF).

Backoffice Operacional & Admin (UC03, UC04, UC05)

[ ] Painel do Funcionário (/staff — UC03): Vista de calendário/lista diária das marcações atribuídas com botão rápido para marcar como "Concluído".

[ ] Painel do Gestor (/manager — UC04):

[ ] Dashboard com gráficos de faturação e serviços mais requisitados.

[ ] Quadro de alocação de equipa para reatribuir marcações.

[ ] Gestão de compras a fornecedores com alertas visuais de stock crítico.

[ ] Painel do Administrador (/admin — UC05):

[ ] Botão de ação rápida "Sincronizar Catálogo SAP → Moloni" com barra de progresso e relatório de importação.

[ ] Gestão CRUD de serviços e preçário.

[ ] Tabela de monitorização de integrações/logs com botão "Reprocessar" para falhas.

4. Testes Finais e Entregáveis (MA2 & MAF)

[ ] Testes Ponta-a-Ponta (End-to-End): Validar fluxo completo sem erros (Registo → Sincronização SAP/Moloni → Agendamento → Compra + Pagamento Stripe → Emissão Fatura PDF → Atualização Stock).

[ ] Documentação Técnica: Atualizar diagramas UML/BPMN, anexar coleção exportada do Postman e evidências dos fluxos n8n no relatório final (máx. 30 páginas).

[ ] Vídeo Demonstrativo: Gravar demonstração executável de todos os casos de uso e publicar no YouTube.