# Auditoria técnica — Órbita Controle de Entregas v5.23

## Resultado executivo
A aplicação ativa é uma PWA offline-first construída em JavaScript modular, IndexedDB e CSS sem framework. A revisão encontrou e corrigiu riscos de cache, cálculo de métricas, agrupamento de datas, exportação com período invertido, transição duplicada de retirada e, principalmente, uma normalização destrutiva que podia apagar vínculos de retornos e reentregas na abertura.

O ponto mais importante é agora seguro: `normalizeReturnQueueStatus()` não altera dados. A arquitetura ainda é local; portanto perfis, ambiente, auditoria e permissões servem para fluxo de interface, mas não constituem segurança contra alguém com acesso ao navegador/IndexedDB.

## Mapa da execução

### `index.html`
É o shell da aplicação. Define o cabeçalho, sidebar, navegação, área `#view`, modal global, toasts, boot screen, manifest, logos, botões de menu e carregamento do módulo `app.js`. A versão de cache, o `aria-label` de fechamento e os estados acessíveis do menu foram atualizados.

### `app.js`
`startApp()` inicializa o shell, ambiente, navegação, ações globais, backup contínuo, relógio, tooltips e ticker. `wireNav()` troca as telas e fecha a sidebar no mobile. `wireGlobalActions()` trata menu, exportação/importação, ambiente, backups e ações rápidas. `openModal()` e `closeModal()` controlam o modal global. `getEnv()`, `getOperatorName()`, `getOperatorRole()` e `canPerform()` leem o contexto local. `refreshApp()` rerenderiza a tela atual.

### `views.js`
`renderCentral()` é a sala operacional: entregas em fluxo, ciclos abertos, KM, alertas, capacidade e fechamento do dia. `renderDashboard()` calcula período, KPIs, tendências, SLA, previsão, ranking, qualidade, frota, financeiro e gráficos. As funções `dashboardRange()`, `dashboardInRange()`, `previousDashboardRange()`, `averageMinutes()`, `percentile()`, `changePercent()` e `dashboardTrend()` sustentam as métricas.

`openDeliveryModal()` e `saveDeliveryForm()` criam/editam entregas. `deliveryStatusActionsHtml()` e `wireDeliveryStatusActions()` exibem transições. `openDeliveryCompletionFlow()` registra chegada/finalização. `finishRetirada()` agora usa uma única transição `changeStatus()`, sem evento duplicado. `openReturnResolutionFlow()` trata retorno, motivo, mercadoria e reentrega.

`renderCycles()` e `openStartCycleModal()` formam ciclos; `openEditCycleDeliveriesModal()` mantém a composição; `openCloseCycleModal()` e `finalizeCycleAt()` fecham o ciclo com validação temporal. `renderKm()` cobre início, fim, edição, reabertura e lixeira de KM. `renderCosts()` e `computeFinancialSummary()` cobrem custos e saldo.

`renderReports()` e `buildExport()` produzem CSV por entidade e chamam o Excel completo. `renderAudit()` exibe a trilha; `renderTrash()` restaura ou esvazia itens; `renderRegistry()` administra veículos, entregadores, colaboradores, bairros, categorias e motivos; `renderSettings()` administra preferências e backups.

### `db.js`
`openDB()` abre o IndexedDB na versão 4. `genericStore()` fornece CRUD e soft delete para stores. `Deliveries` adiciona histórico de status, consultas por ambiente e transições. `OdometerLogs` aplica regras próprias de data e auditoria. `AuditLog` lista a trilha. `exportAll()` gera snapshot completo; `importAll()` agora valida versão e presença de todas as coleções antes de substituir dados. Backups automáticos são criados por `saveAutoBackup()`, listados por `listAutoBackups()` e restaurados por `restoreAutoBackup()`.

### `helpers.js`
Centraliza seleção DOM (`$`, `$$`), dinheiro, datas, horas, escape HTML, toasts, badges de status, guardas de clique, tooltips, máscara de telefone, animações de estatísticas, frases, SVG de barras/linha, termómetro, CSV e download JSON.

### `excel-report.js`
Calcula metas, datas locais, duração, médias, moda, agrupamentos, situação, receita líquida, SLA, qualidade e métricas diárias. Gera XML compatível com Excel, com nomes de folhas seguros e múltiplas abas analíticas. Não é um serviço remoto: o ficheiro é gerado no navegador.

### `styles.css`
Contém tokens Nilo, base da aplicação, sidebar, topbar, cards, tabelas, modais, dashboard, relatórios, ciclos, auditoria, lixeira, registos e modo Consulta. A revisão final acrescenta foco visível, tabela com rolagem horizontal, alvos de toque, modal confortável em celular e breakpoints para filtros e cabeçalho.

## Correções aplicadas nesta rodada

| Área | Problema | Correção |
|---|---|---|
| Dados | Retornos/reentregas eram convertidos para `na_loja` e tinham ciclo, veículo, entregador e horários zerados na abertura | Normalização destrutiva desativada; migração futura deve ser versionada e respaldada |
| Dados | Retirada gravava `update()` e depois `changeStatus()` | Um único `changeStatus()` com o patch operacional completo |
| Dashboard | Tempo médio no cliente permanecia sempre vazio | Calculado entre `clientArrivalAt` e `deliveredAt` |
| Dashboard | Dia da semana usava `toISOString()` e podia cair no dia anterior | Chave local `dashboardDayKey()` |
| PWA | HTML, módulos, README e Service Worker estavam em versões diferentes | Versão alinhada em 5.23 e cache renomeado |
| Backup | Snapshot dizia versão 3 embora o banco estivesse na versão 4 | Exportação declara versão 4 e schema; importação valida estrutura |
| Relatórios | CSV aceitava início posterior ao fim | Validação aplicada antes de todos os CSV |
| Acessibilidade | Menu e fechar modal dependiam de símbolo visual | `aria-label`, `aria-controls` e `aria-expanded` adicionados |
| Mobile | Topbar, filtros e tabelas podiam estourar largura | Quebra controlada, rolagem de tabelas e layout por breakpoint |

## Limitações que permanecem intencionais

A aplicação não possui autenticação real, autorização confiável, RLS, backend, sincronização multi-dispositivo, GPS ao vivo nem garantia de auditoria imutável. `localStorage` e IndexedDB podem ser alterados localmente. O relatório filtra cada entidade pela sua data de referência: entrada de entrega, início de ciclo, turno de KM ou data de custo; isto deve ficar explícito para o utilizador antes de interpretar períodos atravessados.

O histórico antigo dentro de `_arquivo_historico` não participa da execução da raiz. Foi preservado para consulta e não deve ser misturado ao release ativo.

## Validação executada

Foram executados `node --check` nos módulos JavaScript, busca de referências antigas de cache, conferência de referências locais, leitura do manifesto e inspeção do Service Worker. O projeto final deve ser servido por HTTP, não aberto via `file://`, porque ES modules, IndexedDB e Service Worker têm comportamentos diferentes fora de um contexto HTTP.
