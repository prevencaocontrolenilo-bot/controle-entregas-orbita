# Plano de revisão — Órbita Controle de Entregas

## Objetivo
Saneiar a versão ativa na raiz do projeto, mantendo o histórico intacto, corrigindo inconsistências de funcionamento e tornando Central Operacional, Central de Inteligência e Relatórios mais claros em desktop e mobile.

## Escopo ativo
- `index.html`: shell, navegação, cabeçalho, modais e referências de cache.
- `app.js`: inicialização, navegação, perfis locais, backups automáticos e ações globais.
- `views.js`: telas operacionais, dashboard, relatórios, cadastros, auditoria, lixeira e configurações.
- `db.js`: IndexedDB, auditoria, lixeira, contadores e backups.
- `helpers.js`: formatação, segurança de HTML, tooltips, gráficos e downloads.
- `excel-report.js`: relatório Excel XML com abas analíticas.
- `styles.css`: tokens, componentes, responsividade e hierarquia visual.
- `sw.js`/`manifest.webmanifest`: cache e instalação PWA.

## Decisões de design
**Estilos considerados:** painel logístico editorial (0,06), cockpit escuro de frota (0,04), dashboard SaaS genérico (0,03). Escolhido: painel logístico editorial.

- **Movimento:** sala de controle operacional com hierarquia de informação, não um painel decorativo.
- **Princípios:** prioridade visível; números com contexto; ações seguras; leitura confortável em celular.
- **Cores:** azul Nilo para confiança e navegação, amarelo para ação/prioridade, verde para conclusão e vermelho reservado a exceções.
- **Layout:** sidebar de navegação + cabeçalho contextual + blocos de decisão; o dashboard organiza leitura por fluxo, SLA, frota, financeiro e qualidade.
- **Assinaturas:** chips de estado; faixa executiva de saúde; cards com barra inferior semântica.
- **Interação:** ações destrutivas pedem motivo; filtros preservam contexto; tooltips explicam métricas.
- **Animação:** curta e funcional; respeita `prefers-reduced-motion`.
- **Tipografia:** sistema nativo, números tabulares e títulos compactos para alta densidade.
- **Essência:** operação de entregas offline-first para equipas que precisam decidir rápido e deixar rastreabilidade.

## Implementação
1. Alinhar versão/cache PWA para evitar mistura de assets antigos.
2. Corrigir métricas e agrupamentos de data que podem produzir números incorretos.
3. Melhorar estados vazios, foco, toque e tabelas em telas estreitas.
4. Atualizar documentação para declarar claramente o que está implementado e o que depende de backend.
5. Validar sintaxe, referências, build estático e integridade do ZIP final.

## Fora do escopo desta rodada
Login real, RLS, sincronização multi-dispositivo, GPS ao vivo, viagens individuais de Entrega Grande e backend Supabase. Estas funcionalidades não existem na arquitetura offline atual e não serão simuladas como prontas.
