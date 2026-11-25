# CLAUDE.md — Entrada para o Claude Code

> Direciona o Claude Code ao contexto correto. Diretrizes completas em `instructions.md`.

---

## Antes de qualquer trabalho, leia:

1. **`instructions.md`** — regras de desenvolvimento, autonomia, Git e arquitetura
2. **`docs/script/<módulo>.md`** — documentação do módulo que será alterado
3. **Bloco correspondente em `paymobifin-assistente.user.js`** — o código é a fonte de verdade

---

## Contexto rápido

| Item | Valor |
|---|---|
| Arquivo principal | `paymobifin-assistente.user.js` (~8.800 linhas) |
| Tipo | Tampermonkey userscript (monólito, sem build) |
| Versão atual | 13.41.0 |
| Sites alvo | `paymobi-app.ercard.com.br` (ER; o antigo `paymobi.ercard.com.br` também é aceito), `enviamotor.mobfacil.com.br`, `app.tallos.com.br` (RD) e `paymobi.com.br` (só o Acesso ao APP do Boleto+) |
| Arquitetura | **Monolítica** — não dividir em arquivos |
| Fluxo Git | branch temporária → `development` (testes) → `main` (produção) |

---

## Regras inegociáveis

- **Não divida o arquivo** — refatoração não está no escopo atual
- **Não altere `@downloadURL`/`@updateURL`** — push na `main` atualiza todos os usuários
- **Nunca commitar em `main` ou `development` diretamente** — sempre via branch + PR
- **Branch temporária parte de `development`**, não de `main`
- **PR de trabalho vai para `development`** — a `main` recebe só pelo PR de release (`release/proxima`), integrado pelo responsável
- **Não sobrescreva dados de armazenamento existentes** — siga o padrão de `salvar()`
- **Dentro do painel, use `$()`, não `document.querySelector()`**
- **Timeouts apenas em `CONFIG`** — nunca valores hard-coded fora dele
- **Use o prefixo correto** para novas funções (`ng`, `at`, `lb`, `tm`, `cac`, `eq`, `hs`, `bs`, `pd`, `ib`, `bp`, `an`, `rd`, `er`)
- **Alterações críticas (arquitetura, armazenamento, segurança, workflows) exigem autorização explícita**
- **Prioridades atuais (P0, 23/11/2026):** otimizar as funções existentes, manter a documentação sempre igual ao estado do projeto e registrar logs temporários de cada ação — ver `PRIORIDADES.md` e o item 23 do `instructions.md`
- **Fora do escopo:** API/camada de acesso ao ER, API oficial do fornecedor e site próprio. Não propor nem retomar sem nova decisão do responsável
- **Documentação acompanha o código:** todo PR que muda comportamento atualiza no mesmo PR o `docs/script/<módulo>.md` (e `instructions.md`/`PRIORIDADES.md` quando aplicável)

---

## Ciclo de trabalho (Categoria A — rotineiro)

```bash
# Sempre partir de development atualizada
git checkout development && git pull
git checkout -b tipo/descricao-curta

# implementar
git add <arquivos>
git commit -m "tipo: descrição"
git push -u origin tipo/descricao-curta

# abrir PR → development (não main): a validação faz o merge automático (Categoria A)
# a main recebe pelo PR de release (release/proxima), integrado pelo responsável
# não mudar @version nas branches de tarefa: o PR de release calcula a versão
```

---

## Como usar via chat (sem Claude Code)

Cole no início da conversa:
1. `instructions.md` completo
2. `docs/script/<módulo>.md` do módulo envolvido
3. O trecho relevante de `paymobifin-assistente.user.js`
4. Descrição do que precisa

---

## Documentação disponível

```
instructions.md                     ← LEIA PRIMEIRO
docs/script/01-visao-geral.md
docs/script/02-motor-coleta.md
docs/script/03-painel-interface.md
docs/script/04-atendimento.md
docs/script/05-historico.md
docs/script/06-lembretes-timers.md
docs/script/07-cac.md
docs/script/08-ccbs-contas.md
docs/script/09-equipe.md
docs/script/10-navegacao-busca.md
docs/script/11-rd-encerramento.md
docs/script/12-desbloqueios.md
docs/script/13-rotas-er.md         ← rotas do ER e do motor usadas pelo script
docs/script/14-analisar-proposta.md ← aba Analisar proposta (conferência de uma proposta)
PRIORIDADES.md                      ← prioridades funcionais
docs/BRANCHES-IMPLEMENTACOES-POSSIVEIS.md ← branches guardadas (plano de API etc.); não integrar nem apagar
README.md
.github/SETUP.md                    ← configurações para o proprietário
```
