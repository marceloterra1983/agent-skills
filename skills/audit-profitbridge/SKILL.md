---
name: audit-profitbridge
description: Auditoria periódica completa e somente-leitura do repositório ProfitBridge, em ondas, com agentes paralelos e revisão cruzada. Cobre o portão único (nenhuma ordem real), segurança, arquitetura, contratos, testes, Portal (UI/UX/acessibilidade), documentação e operação. Entrega um Artifact com uma tabela mestra de todas as correções sugeridas, resumo por linha e detalhe expansível. Use quando o usuário pedir "auditoria", "revisar o projeto inteiro", "audit", "/audit-profitbridge", ou uma nova rodada da auditoria periódica.
---

# AUDITORIA PERIÓDICA DO PROFITBRIDGE — mutirão enxuto em ondas (somente leitura)

## OBJETIVO

Auditar todo o repositório ProfitBridge em ondas, com agentes paralelos e revisão
cruzada, entregando um artefato único: uma tabela com todas as correções sugeridas,
cada uma comprovada por `arquivo:linha`. Não implemente correções. Este prompt é
reexecutado periodicamente: compare com a auditoria anterior.

## CONTEXTO OBRIGATÓRIO (leia antes de agir)

- `AGENTS.md` (política de workflow, o portão único) e `CLAUDE.md` (contexto técnico).
- `active-projects.json` — allowlists windows/linux, `vendorProjectExclusions`,
  `defaultClassification: retired-historical`.
- `.planning/SYSTEM-SPEC.md` (invariantes AS-IS) e `.planning/STATE.md`.
- `docs/README.md` para a divisão AS-IS / TARGET / HISTORICAL.
- `.planning/codebase/host-exe/README.md` se algum achado tocar o Host.

## O QUE É ESCOPO E O QUE NÃO É

**Em escopo:** `src/` ativo (Contracts, Ipc, RemoteGateway, DataHub,
Authority.Health, Data.Migration, Portal + ClientApp), `tests/`, `scripts/ci/`,
`.github/workflows/`, `deploy/`, `config/`, `database/`, `docs/` e `.planning/`
como documentação a confrontar com o código.

**Fora de escopo — não reporte como defeito:**
- Tudo `retired-historical` (clientes WPF removidos, `archive/`, `SDK/`,
  `knowledge/nelogica`, `knowledge/v4.0.0.37`) — referência de fornecedor/histórico.
- `openspec/`, `.flow/`, `.braingrid/`, `_bmad-output/`, `specs/*/authorizations/` —
  material histórico não normativo. Ausência de "autorização" nunca é achado.
- O `Host.exe` e o binding do ProfitDLL: moram no repositório **profitbridge-host**.
  Aqui só se audita a *fronteira* (contratos Protocol/Ipc, versões de pacote).
- Boa prática genérica sem caminho de falha concreto. Dependência antiga sem
  exploração concreta. Abstração "para o futuro".

## REGRAS DE SEGURANÇA

- Somente leitura: não criar, editar, apagar, formatar nem commitar arquivo do repo.
  Sem branch, commit, PR, tag, worktree novo ou push. (O HTML do artefato vai para o
  scratchpad, fora do repositório.)
- **Não derrube nem reinicie o Host, o Gateway, o DataHub ou o Portal.** O orçamento
  nativo da B3 é agregado e deslizante (3 conexões / 15 min); um restart custa
  produção. Proibido `pkill -f` — o padrão casa com todas as instâncias.
- Nada de `systemctl restart/stop`, deploy para WIN24X7, publicação de pacote
  (`protocol-package.yml`), migração de dados ou mutação no DataHub.
- Sem POST/PUT/PATCH/DELETE para qualquer serviço. GET/HEAD de diagnóstico local, ok.
- Não abra, imprima nem copie valores de `.env`, `certs/`, credenciais DPAPI, chaves
  age/SOPS ou tokens. Verifique **nomes e configuração**, sempre redigindo valores.
- Nunca rode nada a partir de `~/profitbridge` (checkout compartilhado). Use o worktree.
- Não rode o audit SDD nem suítes longas de governança (13 min, frágeis, colidem com
  outros agentes).
- O código real prevalece sobre README, `specs/`, `.planning/` e comentários.
- Não pergunte ao usuário fato que dá para descobrir no código.
- Atualize o usuário a cada ~60s no formato FEITO / FAZENDO / FALTA.

### Comandos permitidos

```bash
pwsh ./scripts/ci/Test-ActiveSurface.ps1
pwsh ./scripts/ci/Invoke-ActiveProjects.ps1 -Surface linux -Action Build
pwsh ./scripts/ci/Test-PowerShellQuality.ps1
python3 scripts/ci/test-ai-sdd-adapters.py --self-test
npm ci --prefix src/Web/ProfitBridge.Portal/ClientApp     # só se node_modules faltar
npx --prefix src/Web/ProfitBridge.Portal/ClientApp tsc --noEmit
npm test --prefix src/Web/ProfitBridge.Portal/ClientApp -- --run
npm audit --omit=dev --prefix src/Web/ProfitBridge.Portal/ClientApp
dotnet list package --vulnerable --include-transitive
git status / git log / gh pr list / gh run list           # leitura
```

`-Action Test` usa `--no-build`: rodar sem o Build antes reprova a maioria dos projetos
com erro de VSTest que **parece** defeito do repo. Se rodar, rode Build antes e diga
isso. Build/teste da superfície **windows** não roda aqui — declare como limitação de
ambiente, nunca como falha. Não afirme validação nativa do ProfitDLL a partir do Linux.

## SKILLS

Leia o `SKILL.md` inteiro antes de usar, e use tudo em modo auditoria (sem escrita):

1. `survey-context` — mapa inicial
2. `dispatch-agents` — paralelismo das ondas
3. `security-review` — Agente A
4. `deepen-architecture` + `ponytail:ponytail-audit` — Agente B
5. `impeccable` — Agente C (UI/UX/acessibilidade do Portal)
6. `doc-rot` — coordenador
7. `enforce-first` + `test-quality-analyzer` — coordenador
8. `validate-contracts` — coordenador
9. `deps-audit` e `type-safety-audit` — se houver sinal que justifique
10. `artifact-design` — obrigatória antes de escrever o HTML da entrega

`grill-with-docs` **só depois** do artefato publicado, e só para decisões que dependam
de tecnologia externa (.NET/ASP.NET Core, DuckDB, MessagePack, SignalR, Named
Pipes/ACL Windows, GitHub Actions self-hosted, systemd). Cite links oficiais.

## ONDA 0 — BASELINE (coordenador)

1. `git status`, `git log -5`, branch, worktree; `git worktree list` para saber quem
   mais está em voo.
2. Ler os documentos de contexto acima; conferir `active-projects.json` contra a árvore
   real (projeto ativo que sumiu, projeto novo não listado).
3. Mapear: entradas (Portal/BFF, gRPC do Gateway, Named Pipe, SignalR), fluxo de dados
   (Host → Ipc → Gateway → DataHub → Portal), persistência (DuckDB, parquet,
   migrações), autenticação/mTLS, evidências e observabilidade.
4. Rodar os checks permitidos e registrar a saída literal.
5. PRs abertos e últimos runs de CI (`CI Gate`, `CI Change Plan`, `Security Scan`).
6. Localizar a auditoria anterior (`Artifact` com `action: "list"`) e, se houver,
   `git diff --stat <commit-anterior>..HEAD` para focar o delta — sem deixar de
   reverificar os P0/P1 antigos.

## ONDA 1 — TRÊS AUDITORES PARALELOS

Cada brief contém `goal`, `in_scope`, `out_of_bounds`, `verify`, `prior_decisions`.
Todos herdam as regras de segurança e a lista de fora-de-escopo.

**Agente A — Portão único, segurança e fronteira nativa**
- Primeiro: prove que o portão continua fechado — `CommandLifecycle.cs`,
  `ForbiddenSurfaceTests.cs` (inclusive o controle negativo) e
  `tests/architecture/NoTradingCapabilityBeforeP11Tests.cs`. Qualquer caminho novo que
  chegue perto de enviar ordem é P0 automático.
- Depois: autenticação/autorização do Gateway e do Portal, mTLS e cadeia de CAs, ACLs
  de Named Pipe, segredos em código/log/appsettings, path traversal, injeção em SQL
  DuckDB, desserialização MessagePack de origem não confiável, SSRF nos jobs de ingest,
  XSS no Portal, superfície `/api/*` e endpoints de saúde expostos.
- Confiança mínima 8/10. Cada achado com `arquivo:linha`, CWE, exploração concreta,
  impacto e correção mínima. Liste também o que examinou **sem** achado.

**Agente B — Arquitetura, simplificação e Ponytail**
- Módulos, seams, adapters, caches, ownership de lifetime (invariante 10), duplicação
  entre DataHub/Gateway/Portal, contratos vazando entre camadas.
- Deletion test em cada oportunidade; Module Depth 1–5.
- Código morto, projeto órfão, script de CI sem chamador, workflow não referenciado,
  dependência dispensável, pacote sem uso em `Directory.Packages.props`, analisadores e
  ratchets que custam mais do que entregam.
- Estime linhas e dependências removíveis sem inventar precisão.

**Agente C — Portal: UI, UX, acessibilidade e desempenho**
- `src/Web/ProfitBridge.Portal/ClientApp/src/modules/{batch,datahub,health,history,host,manual,realtime,scrapper}` e o shell.
- Acessibilidade, semântica, teclado, foco, formulários, estados vazio/erro/loading,
  responsividade, alvos de toque, `prefers-reduced-motion`, tema, CLS.
- Específico daqui: `lightweight-charts` renderiza em `<table>` — CSS global de tabela
  desenha grade falsa sobre o canvas e escala os candles; verifique. Polling vs
  SignalR: reconexão, backoff, vazamento de listener, estado obsoleto em aba de fundo.
- Superfície compartilhada (`PortalComponents.tsx`, `usePolling.ts`, `lib/http.ts`,
  `styles/tokens.css`, `PortalContracts.cs`) merece atenção extra: mudança ali atinge
  todos os módulos.

**Em paralelo, o coordenador executa:**
- `doc-rot`: README, `docs/`, `.planning/codebase/*`, `CLAUDE.md`, comentários,
  comandos, portas e ADRs divergentes do código real.
- `enforce-first` + `test-quality-analyzer`: teste comportamental vs teste que só
  procura texto/arquivo; asserção tautológica; isolamento de DuckDB por teste
  (invariante 8); `.Result`/`.Wait()` em caminho async (invariante 4); flakiness.
- `validate-contracts`: Protocol ↔ Ipc ↔ Gateway ↔ DataHub ↔ `PortalContracts.cs` ↔
  tipos do ClientApp; janela de compatibilidade N/N-1; versões de pacote publicadas vs
  consumidas; formatos de evidência e de métricas.
- Operação: health checks, logging, rollback, deploy dos três serviços, systemd,
  runners de CI, e riscos operacionais conhecidos (recovery do Host, janela B3,
  worktree que sustenta o serviço do Portal).

## ONDA 2 — REVISÃO CRUZADA (agentes novos, contexto limpo)

Encerre os agentes da Onda 1. Suba três críticos que não viram o trabalho original:

1. **Crítico de segurança** — tente refutar cada vulnerabilidade; elimine falso
   positivo; exija caminho de exploração reproduzível.
2. **Crítico de arquitetura** — rejeite refatoração especulativa; exija benefício
   concreto e mensurável em cada simplificação.
3. **Crítico de qualidade** — valide achados de teste, contrato, documentação e UI.

O coordenador então reconfere cada `arquivo:linha` pessoalmente, remove duplicata,
arbitra divergência entre auditor e crítico, e registra o que foi rejeitado e por quê.
Conclusão sem evidência reproduzível não entra no artefato.

Armadilhas que já produziram falso positivo aqui, cheque antes de reportar: ler campo
isolado sem ler o produtor; confundir mock/fixture com produção; métrica que satura
tomada como saúde; concluir "é vendor" pelo nome da pasta.

## PADRÃO DO ACHADO

`ID | prioridade P0–P3 | categoria | arquivo:linha | evidência | impacto real |
cenário de falha ou exploração | correção mínima | esforço | confiança 1–10 |
agente autor | resultado da revisão cruzada`

P0 = toca o portão único, perde dado, expõe segredo ou derruba produção.

## ENTREGA — ARTEFATO ÚNICO EM TABELA

O relatório é um **Artifact publicado**, não texto na conversa. Na conversa deixe só:
veredito em 3 linhas, contagem por prioridade e o link do artefato.

### Como construir

1. Carregue a skill `artifact-design` antes de escrever qualquer linha de HTML.
2. Escreva o HTML no scratchpad como `audit-profitbridge-YYYY-MM-DD.html` — data da
   execução, um artefato novo por rodada, para comparação histórica.
3. Publique com a ferramenta `Artifact`:
   - `title`: `Auditoria ProfitBridge` + data curta
   - `favicon`: `🔎` — **sempre o mesmo**, em toda rodada
   - `description`: uma frase com o veredito e a contagem P0/P1
4. Antes de publicar, use `Artifact` com `action: "list"` para achar a auditoria
   anterior e linkar o delta.

### Estrutura do artefato

**Topo — cartão de estado, sem prosa:** contagem P0 / P1 / P2 / P3 / rejeitados;
branch, commit curto, data; resultado de cada check executado; estado do portão único
(FECHADO/ABERTO) em destaque; e o que não pôde ser verificado neste ambiente.

**Corpo — uma tabela mestra com TODAS as correções sugeridas.** Uma linha por achado,
ordenada por prioridade:

| ID | Prio | Categoria | Local | Problema | Correção mínima | Esforço | Conf | Revisão |
|---|---|---|---|---|---|---|---|---|

- `Local` = `arquivo:linha`, link clicável relativo.
- `Problema` e `Correção mínima` ≤ 90 caracteres cada. Frase, não parágrafo.
- `Esforço` = estimativa concreta (`15min`, `2h`, `1 dia`), nunca "baixo/médio/alto".
- `Conf` = 1–10. `Revisão` = confirmado / reduzido / rejeitado.
- Prioridade com cor **e** rótulo textual, nunca só cor.

**Detalhe embutido:** cada linha expande (`<details>` ou linha-irmã acionada por botão)
para evidência literal do código, impacto real, cenário de falha ou exploração,
correção detalhada passo a passo, arquivos afetados, agente autor e — se rejeitada — o
motivo da rejeição.

**Filtros:** botões que filtram por prioridade e por categoria, mais busca por texto.
JavaScript inline, sem dependência externa. A tabela rola dentro do próprio contêiner
(`overflow-x: auto`); a página nunca rola na horizontal.

**Depois da tabela, seções curtas** — só o que não cabe em linha de tabela:

1. Mapa da arquitetura e do fluxo de dados (Mermaid em `<pre class="mermaid">`)
2. Pontos fortes
3. Lista Ponytail: o que apagar, com linhas e dependências estimadas
4. Achados rejeitados na revisão cruzada, com o motivo
5. Delta desde a auditoria anterior: corrigido / regrediu / persiste, com link
6. Plano mínimo: 24h / 7 dias / depois — cada item referenciando IDs da tabela
7. Decisões que exigem julgamento do dono
8. Skills e agentes usados, com o status de cada onda

Todo achado tem que estar na tabela, com ID; nada existe só na prosa. Nenhuma linha da
tabela sem `arquivo:linha` verificado pelo coordenador.

Ao terminar, não implemente nada. Se houver decisão que dependa de tecnologia externa,
proponha iniciar `grill-with-docs` com **uma única** primeira pergunta e sua recomendação.
