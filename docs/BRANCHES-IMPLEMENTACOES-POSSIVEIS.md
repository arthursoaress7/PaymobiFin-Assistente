# Branches de possíveis implementações

> Branches com trabalho que **não está** na `development` nem na `main` e que fica guardado como referência para uma possível implementação futura.
> Nenhuma delas está no escopo atual (ver `PRIORIDADES.md`). **Não integrar, não retomar e não apagar** sem decisão do responsável.
> Registrado em 23/11/2026.

---

## Plano de API / camada de acesso ao ER (fora do escopo desde 23/11/2026)

Era a P0 de 11/11/2026: uma camada única (`erBuscar`) que funcionaria como uma API interna do script para ler os dados do ER, e a coleta do motor MobFacil pelo JSON de `/api/visualizador` em vez de cliques na árvore. Saiu do escopo por decisão do responsável. O mapeamento das rotas que o script usa hoje continua em [`docs/script/13-rotas-er.md`](./script/13-rotas-er.md).

**Estado:** implementado em parte, com testes locais (Node + jsdom, não versionados) usando respostas reais dos HARs de 11–12/11/2026. **Nunca testado no ER real.** Tudo desligado por padrão (`erModulos`).

| Branch | Conteúdo | PR | Base |
|---|---|---|---|
| `feature/api-er` | Branch principal do plano. Reúne a camada base, o piloto do Relatório de Pendências, a coleta do motor pelo JSON e a correção do endereço novo do ER. Diferença para a `development` (17/11/2026): `paymobifin-assistente.user.js` +397/−15 e documentos do plano | #44 (só o plano) → `main` | parte da `development` de 12/11/2026 |
| `feature/er-camada-base` | P0.3: seção "ACESSO AO ER" com `erBuscar(nome, params, opcoes)` → `{ ok, dados }` ou `{ ok: false, motivo }`; rotas `mesasUsuario`, `pendencias`, `pendenciaDetalhe`; lista de rotas bloqueadas (encerrar protocolo, SMS, login...); `erConta()`; chaves novas `erModulos` e `erFalhas`; teste manual por evento `mf-er-teste` | #52 → `feature/api-er` | — |
| `feature/er-motor` | P0.4 + P0.6: Relatório de Pendências pela camada (comparado com `pdLista`) e coleta do motor pelo JSON (`motorDadosDeJson`, com modo "comparar com o método atual" que não grava). Move o motivo de reprovação para `reprovDeTexto()` e passa as saídas do `iniciarMotor` por `concluir()` | #53, #54 → `feature/api-er` | — |
| `fix/er-endereco-novo` | Camada reconhece o endereço novo do ER (`paymobi-app.ercard.com.br`) | #69 → `feature/api-er` | — |
| `chore/sync-development-api-er` | Sincronização da `development` na `feature/api-er` (17/11/2026) | #65 → `feature/api-er` | — |
| `docs/plano-api-er` | Documentos do plano: `docs/PLANO-API-ER.md` (fases P0.1–P0.9, inventário, riscos) e `docs/PROPOSTA-CAMADA-ER.md` (desenho aprovado em 12/11/2026, com as 8 decisões) | #41 → `feature/api-er` | — |

**Se um dia for retomado:**

1. Nova decisão do responsável registrada em `PRIORIDADES.md`.
2. Partir de uma branch nova a partir da `development` atual e trazer o código da `feature/api-er` (ela está mais de 80 commits atrás da `development`; esperar conflitos no script).
3. Categoria B: novas chaves de armazenamento (`erModulos`, `erFalhas`) e mudança na coleta do motor.
4. Testar no ER real antes de qualquer integração (os roteiros estão nos PRs #52 e #54).
5. O desenho e o plano completos podem ser lidos na branch `docs/plano-api-er` ou no histórico do Git (removidos da `development` no PR #135).

---

## Outras branches não integradas

| Branch | O que falta integrar | Observação |
|---|---|---|
| `feature/auto-merge-workflow` | 2 commits de documentação depois do PR #14 (ex.: "docs: fluxo com branch development documentado", 09/11/2026) | Provavelmente superado pelo `instructions.md` atual. Conferir antes de apagar |
| `feature/desbloqueios-v2` | 1 commit de documentação depois do PR #35 ("docs: unificar CLAUDE.md em instructions.md", 11/11/2026) | Provavelmente superado. Conferir antes de apagar |

---

## Manutenção deste documento

- Ao guardar uma branch como possível implementação, registrar aqui: nome, conteúdo, PRs, estado dos testes e o que precisa para retomar.
- Ao retomar, integrar ou apagar uma dessas branches, atualizar este documento no mesmo PR.
