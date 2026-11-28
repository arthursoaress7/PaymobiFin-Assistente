# instructions.md — Manual operacional do PaymobiFin

> **Fonte única de verdade para qualquer IA ou desenvolvedor que trabalhe neste projeto.**
> Leia este arquivo completamente antes de iniciar qualquer tarefa.

---

## 1. Identidade e contexto do projeto

O **PaymobiFin - Assistente** é um userscript Tampermonkey que roda em dois sites:

| Site | Função |
|---|---|
| `paymobi-app.ercard.com.br` (antigo: `paymobi.ercard.com.br`) | Painel principal com todas as funcionalidades |
| `enviamotor.mobfacil.com.br` | Motor de coleta de dados de crédito (iframe) |

**Arquivo de código:** `paymobifin-assistente.user.js` — único arquivo JavaScript, ~10.400 linhas, sem transpilador ou bundler.

**Nome antigo:** até a v13.27.1 o script se chamava `automacaomobfacil.user.js`. Esse arquivo ficou como ponte (congelado na 13.27.2) para migrar as instalações antigas e foi removido em 19/11/2026 (P4.8). Instalações que ainda apontem para o nome antigo não recebem mais atualizações: reinstalar pelo endereço de `paymobifin-assistente.user.js`.

**Versão atual:** 13.50.0 (fonte de verdade: campo `@version` no cabeçalho do script — ver item 12)

**Repositório:** `git@github.com:arthursoaress7/PaymobiFin-Assistente.git`  
**Proprietário do repositório:** arthursoaress7 (pode ter permissões administrativas que o usuário atual não tem)

---

## 2. Prioridades atuais

1. **Manter o sistema funcionando** — preservar toda funcionalidade existente
2. **Otimizar as funções existentes** — melhorar o que já existe antes de criar algo novo (P0 do `PRIORIDADES.md`, decisão de 23/11/2026)
3. **Documentação sempre atualizada** — a documentação acompanha o estado atual do código, com atualização automática/obrigatória a cada mudança (regras no item 23)
4. **Logs temporários de comportamento** — o script registra o que aconteceu em cada ação, para revisão e otimização (regras no item 23)
5. **Não refatorar** — a divisão do monólito não está no escopo atual

**Fora do escopo (decisão de 23/11/2026):** o plano de API/camada de acesso ao ER (antiga P0, branch `feature/api-er`), a API oficial do fornecedor e o site próprio (antiga P4.11). Não propor nem retomar sem nova decisão do responsável.

Qualquer solicitação que contrarie estas prioridades requer autorização explícita.

---

## 3. Regras de preservação do monólito

Até autorização formal para refatoração:

- **Não dividir `paymobifin-assistente.user.js` em múltiplos arquivos**
- **Não reorganizar diretórios do projeto**
- **Não alterar a arquitetura ou a ordem de execução**
- **Não remover funções** sem investigar todos os pontos de uso
- **Não alterar variáveis globais** por conveniência
- **Não substituir implementações existentes** por alternativas sem justificativa técnica aprovada
- **Não fazer melhorias** fora do escopo explicitamente solicitado

---

## 4. Hierarquia das instruções

Em caso de conflito entre documentos, esta é a ordem de precedência:

1. Instruções diretas do responsável (chat/sessão atual)
2. Este arquivo (`instructions.md`)
3. `README.md`
4. `docs/script/<módulo>.md`
5. Comentários no código-fonte

O código-fonte é sempre a fonte de verdade sobre **como o sistema funciona hoje**.

---

## 5. Política de autonomia das IAs

### Categoria A — Alterações rotineiras (autonomia total)

A IA pode executar o ciclo completo sem aguardar aprovação:

- Criar branch → implementar → commitar → push → abrir Pull Request → merge automático

**Exemplos:**
- Atualização de documentação
- Correções pontuais de bug
- Novas funcionalidades isoladas (sem impacto em módulos compartilhados)
- Ajustes visuais no painel
- Testes e manutenção de baixo risco

**Limite:** a autonomia não autoriza ultrapassar o escopo solicitado.

### Categoria B — Alterações críticas (exigem autorização explícita)

A IA pode fazer diagnóstico e preparar proposta, mas **deve aguardar aprovação antes de executar**:

- Mudança de arquitetura ou divisão do monólito
- Refatoração estrutural
- Alteração da estrutura de diretórios
- Exclusão de funcionalidades existentes
- Alterações em mecanismos de autenticação
- Leitura do motor MobFacil por outra via (JSON de `/api/visualizador` em vez da árvore)
- Qualquer requisição ao ER que altere dados
- Alterações em `@connect`, `@grant` ou `@match` no cabeçalho do script
- Alterações na estrutura de armazenamento (`GM_setValue/getValue`)
- Modificações em mecanismos de segurança (criptografia, Pusher)
- Alterações no `@downloadURL` ou `@updateURL` (impacta todos os usuários)
- Mudanças em permissões ou configurações administrativas do GitHub
- Alterações em arquivos de workflows (`.github/workflows/`)
- Qualquer alteração que afete todos os usuários do script imediatamente

**A criação de uma tarefa por uma IA não constitui autorização para execução de alterações críticas.**

---

## 6. Regras de Git

### Proibido em qualquer circunstância

- Commitar diretamente na `main`
- Fazer push direto na `main`
- Executar `force push` sem autorização explícita
- Sobrescrever alterações de outras pessoas ou IAs
- Apagar branches ou histórico sem autorização
- Executar `git reset --hard` sem autorização
- Modificar permissões do repositório
- Criar tags de release manualmente (o workflow `release.yml` faz isso)
- Ignorar conflitos sem investigar a origem

### Obrigatório sempre

- Verificar o estado do Git antes de iniciar (`git status`, `git log --oneline -5`)
- Verificar os arquivos alterados antes do commit (`git diff --stat`)
- Trabalhar em branches temporárias
- Resolver conflitos compreendendo a causa antes de aceitar qualquer lado
- Criar branch a partir da `development` atualizada: `git checkout development && git pull`

### Sobre autenticação

As IAs usam a autenticação pessoal do responsável pelo desenvolvimento. O GitHub não distingue automaticamente ações de IA de ações humanas. Toda operação Git feita por uma IA é registrada como ação do usuário autenticado.

---

## 7. Padrão de branches

O projeto adota um fluxo com **duas branches permanentes** e branches temporárias por tarefa.

```
main         ←  produção — código estável, distribuído aos usuários via @updateURL
  ↑ só pelo PR de release (release/proxima), integrado pelo responsável
development  ←  ambiente de teste — todo trabalho passa por aqui antes de ir à main
  ↑ PR de branch temporária
  └── tipo/descricao-curta  (temporária, uma por tarefa)
```

### Branches permanentes

| Branch | Papel |
|---|---|
| `main` | Produção. Recebe **somente** o PR de release (`release/proxima`) e, em emergência, PR de `hotfix/*`; os dois com merge manual do responsável. Nenhum outro merge na `main`: nem de branch de tarefa, nem PR `development` → `main` aberto à mão. |
| `development` | **Ambiente de teste.** Toda implementação é mergeada aqui primeiro. O script desta branch tem `@downloadURL`/`@updateURL` apontando para a própria `development`, para quem testa não ser levado de volta à versão da `main`. |

### URLs de atualização por ambiente

| Branch | `@downloadURL` / `@updateURL` | Quem instala |
|---|---|---|
| `main` | `.../raw/refs/heads/main/paymobifin-assistente.user.js` | Toda a equipe (produção) |
| `development` (e branches de tarefa) | `.../raw/refs/heads/development/paymobifin-assistente.user.js` | Quem testa |

- As URLs **não são editadas à mão**: o PR de release troca para a `main` (`preparar_release.py`) e a sincronização depois de hotfix devolve para a `development` (`release.yml`).
- A validação reprova PR para a `main` com URLs da `development` e PR para a `development`/`feature/**` com URLs da `main`.
- A `development` **não recebe** o `@version` nem o `CHANGELOG.md` de cada release (ficam só na `main`). Por isso o `@version` da `development` pode estar atrás do da `main`, e quem testa não recebe atualização automática: para pegar as mudanças, reinstale pela URL da `development`.

### A `main` só volta para a `development` depois de hotfix

- O PR de release parte da `main` (`release/proxima` = `main` + merge da `development`), então a `main` sempre contém a `development` e não há nada para trazer de volta depois de uma release.
- Depois de um PR `hotfix/*` → `main`, o `release.yml` abre a sincronização `chore/sync-main-vX.Y.Z` → `development` (merge commit, URLs de teste mantidas), para a correção não se perder no próximo release.
- Se o "Preparar release" falhar com conflito, é sinal de hotfix na `main` que ainda não voltou: integre a sincronização pendente.

### Branches temporárias

Criadas a partir de `development`, mergeadas de volta em `development` via PR.

**Prefixos obrigatórios:**

| Prefixo | Uso |
|---|---|
| `feature/` | Nova funcionalidade |
| `fix/` | Correção de erro |
| `docs/` | Documentação |
| `refactor/` | Refatoração autorizada |
| `chore/` | Manutenção e configurações |
| `hotfix/` | Correção urgente em produção: parte da `main`, PR → `main` com o `@version` aumentado (PATCH) no próprio PR e merge manual; depois volta sozinho para a `development` |

### Regras

- **Criar sempre a partir de `development` atualizada:** `git checkout development && git pull`
- Uma branch por tarefa — não acumular múltiplas tarefas na mesma branch
- **Nunca commitar diretamente em `main` nem em `development`**
- Branches temporárias são removidas após o merge
- `hotfix/` é a única exceção: parte da `main`, entra na `main` por PR (merge manual) e volta para a `development` pela sincronização automática

---

## 8. Conventional Commits

Formato obrigatório:

```
tipo: descrição objetiva em minúsculas
```

**Tipos permitidos:**

| Tipo | Impacto SemVer | Quando usar |
|---|---|---|
| `feat` | MINOR | Nova funcionalidade |
| `fix` | PATCH | Correção de bug |
| `perf` | PATCH | Melhoria de desempenho |
| `docs` | — | Apenas documentação |
| `refactor` | — (ou MINOR/PATCH se funcional) | Refatoração |
| `chore` | — | Versão, configs, manutenção |
| `test` | — | Testes |
| `build` | — | Dependências, build |
| `ci` | — | GitHub Actions, automação |

**Mudança incompatível:** usar `BREAKING CHANGE:` no rodapé ou `!` após o tipo:
```
feat!: remover suporte ao formato antigo de exportação
```

**O título do Pull Request deve seguir o mesmo formato.**

---

## 9. Pull Requests e integração

### Alterações rotineiras (Categoria A)

```
1. Criar branch a partir de development atualizada
   git checkout development && git pull
   git checkout -b tipo/descricao-curta

2. Implementar dentro do escopo solicitado

3. Revisar o diff antes de commitar
   git diff --stat

4. Commit no formato Conventional Commits

5. Push e abrir PR → development (não main)

6. validate-pr.yml valida o PR e, se for Categoria A, faz o merge automático (Squash)

7. A cada merge na development, o workflow "Preparar release" abre ou atualiza
   o PR de release (release/proxima → main) com @version, documentação e CHANGELOG

8. Quando development estiver testada e estável:
   o responsável integra o PR de release (um clique). Não abrir PR development → main à mão
```

### Alterações críticas (Categoria B)

```
1. Preparar diagnóstico e proposta detalhada
2. Apresentar ao responsável e aguardar autorização explícita
3. Executar somente após aprovação — branch a partir de development
4. Abrir PR → development (a validação classifica como Categoria B quando altera
   workflows, .github/scripts, package.json ou o arquivo-ponte: sem merge automático)
5. Aguardar aprovação manual do responsável para o merge em development
6. Só vai para main após nova confirmação (pelo PR de release)
```

**Squash Merge é o padrão** — um commit por PR.

---

## 10. Verificações obrigatórias (checklist pré-commit)

- [ ] O diff contém apenas alterações do escopo solicitado?
- [ ] Nenhuma funcionalidade existente foi removida sem autorização?
- [ ] Nenhuma variável global (`CONFIG`, `PADRAO`, `PAGINA`) foi alterada sem necessidade?
- [ ] Novas chaves de armazenamento foram adicionadas à tabela do item 15?
- [ ] O HTML do painel usa `$('#id')` (não `document.querySelector`)?
- [ ] Novos timeouts usam `CONFIG` (não valores hard-coded)?
- [ ] Novas funções de módulo usam o prefixo correto?
- [ ] O arquivo `docs/script/` correspondente foi atualizado se necessário?
- [ ] `@downloadURL` e `@updateURL` não foram alterados?
- [ ] `'use strict'` não foi removido?
- [ ] Se o `@version` mudou, a versão atual foi atualizada nos locais listados no item 12?
- [ ] Arquivos de workflows (`.github/workflows/`) não foram alterados? (Categoria B)

---

## 11. Tratamento de falhas

**Política: bloqueio por padrão.**

Quando uma verificação ou teste falhar:

1. Identificar qual verificação falhou
2. Investigar a causa
3. Determinar se é relacionada à alteração realizada
4. **Se relacionada:** corrigir antes de prosseguir
5. **Se não relacionada:** documentar a evidência de que não é relacionada
6. Avaliar o risco — falhas desconhecidas ou inconclusivas não podem ser ignoradas
7. Permitir exceção somente com justificativa técnica defensável

**Nunca:**
- Ignorar falhas para facilitar o merge
- Desabilitar verificações sem autorização
- Assumir que uma falha pré-existente justifica deixar outra passar

---

## 12. Versionamento SemVer

O campo `@version` no cabeçalho de `paymobifin-assistente.user.js` é a fonte de verdade da versão.

**Versão na documentação:**

- A documentação só pode exibir a versão atual nestes locais:

  | Arquivo | Local |
  |---|---|
  | `README.md` | Linha `**Versão atual:**` no topo |
  | `instructions.md` | Linha `**Versão atual:**` do item 1 |
  | `CLAUDE.md` | Linha `Versão atual` da tabela "Contexto rápido" |
  | `docs/script/01-visao-geral.md` | Linha `**Versão atual:**` no cabeçalho |
  | `docs/script/03-painel-interface.md` | Barra de título no desenho "Estrutura do painel" (`vX.Y.Z`) |

- Sempre que o `@version` mudar, a IA ou o desenvolvedor atualiza todos esses locais no mesmo commit/PR.
- A versão da documentação nunca pode divergir do `@version`.
- Não acrescentar a versão atual em outros lugares; se for preciso, incluir o novo local nesta tabela no mesmo PR.
- Referências históricas (ex.: "Concluído (v13.27.0)", "congelado na v13.27.2"), os exemplos de incremento abaixo e o `CHANGELOG.md` não são "versão atual" e não seguem esta regra.

**Regras de incremento (aplicadas automaticamente pelo `release.yml`):**

| Tipo de commit | Incremento |
|---|---|
| `feat` | MINOR (ex.: 13.22 → 13.23.0) |
| `fix`, `perf` | PATCH (ex.: 13.22 → 13.22.1) |
| `BREAKING CHANGE` | MAJOR (ex.: 13.22 → 14.0.0) |
| `docs`, `chore`, `test`, `ci`, `refactor` sem impacto funcional | nenhum |

O incremento é calculado no **PR de release** (workflow "Preparar release", script `.github/scripts/preparar_release.py`), que atualiza o `@version`, os locais da tabela acima e o `CHANGELOG.md` antes do merge na `main`. **Não** fazer mais commits manuais `chore: versão X.Y.Z`: as branches de tarefa não mudam o `@version`.

---

## 13. GitHub Actions e fluxo automatizado

### Workflows disponíveis

| Arquivo | Trigger | Responsabilidade |
|---|---|---|
| `.github/workflows/validate-pr.yml` | PR para `main`, `development` ou `feature/**` | Valida título, origem (PR para a `main` só de `release/proxima` ou `hotfix/*`; hotfix precisa aumentar o `@version`), conflitos, integridade e sintaxe do script (`node --check`), URLs de atualização do ambiente (`main` → produção; `development`/`feature/**` → teste) e versão da documentação. Classifica Categoria A/B. Job **Merge automático**: Categoria A para `development`/`feature/**` é integrada sozinha (Squash; sincronização `chore/sync-main-*` → `development` por merge commit). Depois do merge na `development`, dispara o "Preparar release" (`workflow_dispatch`), porque o merge feito pelo `GITHUB_TOKEN` não dispara o `push`. |
| `.github/workflows/release-pr.yml` ("Preparar release") | `push` → `development`; `workflow_dispatch` (disparado pelo merge automático ou à mão em Actions) | Monta a `release/proxima` a partir da `main` com merge da `development` e abre ou atualiza o PR de release → `main` com a próxima versão, documentação, `CHANGELOG.md` e as URLs de atualização trocadas para a `main` (`.github/scripts/preparar_release.py`). Fecha o PR se a `main` já contiver a `development`. O PR de release é integrado com **merge commit** (não Squash). |
| `.github/workflows/release.yml` | `push` → `main` | Cria a tag `vX.Y.Z` (do `@version`) e a GitHub Release com as notas do `CHANGELOG.md`. **Só depois de merge de `hotfix/*`** (ou rodado à mão), cria a branch `chore/sync-main-vX.Y.Z` (`development` + merge da `main`, com as URLs devolvidas para a `development`) e abre o PR "chore: sincroniza main na development". Não faz commits na `main`. |

### Fluxo completo

```
branch de tarefa → PR → development
  ↓ validate-pr.yml (Validar alterações)
  ↓ Categoria A → Merge automático (Squash)          Categoria B → merge manual
development
  ↓ release-pr.yml (Preparar release)
PR release/proxima → main   (versão, documentação e CHANGELOG já calculados)
  ↑ release/proxima = main + merge da development (sem conflito)
  ↓ merge manual pelo responsável (Create a merge commit)
main → Tampermonkey de todos os usuários
  ↓ release.yml: tag vX.Y.Z + GitHub Release (a development não recebe nada de volta)

hotfix/* (parte da main, @version +PATCH) → PR → main   merge manual
  ↓ release.yml: tag + branch chore/sync-main-vX.Y.Z (URLs de teste mantidas)
  ↓ PR "chore: sincroniza main na development" → merge automático (merge commit)
```

- **A `main` nunca tem merge automático e só recebe o PR de release ou hotfix.** Cada merge nela chega na hora a toda a equipe.
- **A `main` só volta para a `development` depois de hotfix**, e a sincronização mantém as URLs de teste.
- **Rótulo `sem-auto-merge`:** impede o merge automático de um PR específico.
- **Incremento:** `feat` → MINOR · `fix`/`perf` → PATCH · `!`/`BREAKING CHANGE` → MAJOR · demais → nenhum (o PR de release se chama "chore: integrar development na main").

### Token `RELEASE_TOKEN`

PRs e merges feitos com o `GITHUB_TOKEN` padrão não disparam outros workflows. Por isso o merge automático, o PR de release e o PR de sincronização usam o secret `RELEASE_TOKEN` (token pessoal do proprietário). Sem ele, os workflows funcionam com o `GITHUB_TOKEN`, mas: o merge na `development` não abre o PR de release, e os PRs de release e de sincronização ficam sem a verificação "Validar alterações".

### Configurações necessárias pelo proprietário

Ver `.github/SETUP.md`.

---

## 14. Segurança e proteção de dados

- **`senhasContas`** — armazena senhas no Tampermonkey, que não é criptografado. Nunca expor em logs.
- **`equipeSecret` e `equipeChave`** — idem. Nunca exibir valores.
- **`@updateURL` aponta para `main`** — qualquer merge na `main` distribui para todos os usuários.
- **`PAGINA.eval()`** — usado no módulo CCBs. Frágil se o site adicionar CSP restritivo.
- **Diagnósticos (`debug`)** — nunca incluem valores pessoais.
- **Acesso ao ER** — a autenticação é o cookie de sessão do ER. Nenhum cookie, cabeçalho de sessão ou resposta do ER é gravado em `GM_setValue`, exibido no painel, enviado pelo Pusher ou registrado em log/diagnóstico. Ao mapear rotas (DevTools, HAR), remover `Cookie` e dados pessoais de clientes antes de registrar ou compartilhar.

---

## 15. Arquitetura interna do script

### Ponto de entrada

```javascript
if (location.hostname.endsWith('mobfacil.com.br')) iniciarMotor();
else if (window.top === window) iniciarPainel();
```

### Mapa de seções internas

| Bloco | Responsabilidade |
|---|---|
| Cabeçalho (linhas 1–23) | Metadados Tampermonkey, permissões, dependências |
| Constantes globais | `VERSAO`, `CONFIG`, `PAGINA`, `PADRAO`, `cfg`, `sleep` |
| Leitura da árvore | Funções de coleta React/DOM do motor MobFacil |
| `iniciarMotor()` | Navegação automática na árvore do iframe |
| `iniciarPainel()` | Painel completo — contém todos os módulos abaixo |
| → CCBs | Download de CCBs em lote |
| → Contas | Troca rápida de conta, `contaAtiva()` |
| → Gerador de Negociações (`ng`) | Formulário + geração de textos |
| → Lembretes (`lb`) | Pagamentos parciais v2 |
| → Consulta CAC (`cac`) | Leitura da Central de Atendimento |
| → Atendimento (`at`) | Negociação e Renegociação |
| → Timers (`tm`) | Agendamentos de expediente |
| → Navegação/Aparência | Nova UI / UI Antiga |
| → Equipe (`eq`) | Pusher, criptografia AES-GCM |
| → Histórico (`hs`) | Acordos registrados; acordos em aberto da Campanha do Fôlego e "Cliente cumpriu" (`hsFolego*`) |
| → Busca de cliente (`bs`) | Central de busca (somente leitura) |
| → Navegação automática | Troca de tela automática no ER |
| → Relatório de Pendências (`pd`) | Fila de Pendências da conta master (somente leitura) |
| → Negativação PEFIN (`pf`) | Lista de clientes para o gerador do arquivo PEFIN (Serasa): cadastro da Mesa de Crédito + dívida do CAC, cópia em texto/CSV (somente leitura; lista só em memória; ver docs/script/07-cac.md) |
| → Analisar proposta (`an`) | Conferência de uma proposta: política de faixa, Receita, documentos, contato, MobFacil, itens da análise e KYC da BMP (somente leitura; ver docs/script/14) |
| → Impeditivos BMP (`ib`) | Fila de Propostas (ícone 🚫 do KYC da BMP) + Histórico da Mesa (somente leitura) |
| → RD Atendimento (`rd`) | Análise com Gemini e encerramento do dia |
| Boleto+ (`bp`, `iniciarBoletoPlus`, em `paymobi.com.br`) | Responde ao ER o primeiro acesso no app do Boleto+ por CPF (Acesso ao APP das Efetivadas; somente leitura) |

### Variáveis globais compartilhadas

| Nome | Tipo | Descrição |
|---|---|---|
| `VERSAO` | string | Versão lida via `GM_info` |
| `CONFIG` | objeto | Todos os timeouts e limites |
| `PAGINA` | objeto | `unsafeWindow` — acesso ao JS do site |
| `PADRAO` | objeto | Valores padrão das preferências |
| `cfg(k)` | função | Lê preferência com fallback para `PADRAO` |
| `sleep(ms)` | função | Aguarda N ms via Web Worker |

### Prefixos de funções por módulo

| Prefixo | Módulo |
|---|---|
| `ng` | Gerador de Negociações |
| `at` | Atendimento |
| `lb` | Lembretes |
| `tm` | Timers |
| `cac` | Consulta CAC |
| `ccb` | CCBs |
| `eq` | Equipe (Pusher) |
| `hs` | Histórico |
| `bs` | Busca de cliente |
| `pd` | Relatório de Pendências |
| `ib` | Relatório de Impeditivos BMP |
| `bp` | Boleto+ (Acesso ao APP das Efetivadas) |
| `an` | Analisar proposta |
| `pf` | Negativação PEFIN |
| `rd` | RD Atendimento |
| `er` | Acesso ao ER (sessão, base, requisições) |

### Seletor interno do painel

```javascript
const $ = s => painel.querySelector(s);  // dentro de iniciarPainel()
```

**Nunca usar `document.querySelector` dentro do painel.**

### Chaves de armazenamento

Documentar novas chaves aqui antes de usar. Renomear sem migração quebra dados de usuários reais.

| Chave | Conteúdo | Módulo |
|---|---|---|
| `dados` | `{ [id]: { qtd, val, score, consultas, cpf, reprov } }` | Motor/Coleta |
| `debug` | Snapshots de diagnóstico | Motor |
| `fila` / `feitas` / `ignoradas` / `verificadas` | Controle da fila | Coleta |
| `rodando` / `batida` / `atualId` / `fim` | Estado da coleta | Coleta |
| `modo` | `'coleta' \| 'aprovadas' \| 'reprovadas'` | Coleta |
| `mfLembretes` | `{ versao: 2, itens: [...] }` | Lembretes |
| `mfLembretesAviso` | Estado do pop-up | Lembretes |
| `mfAcordos` | `{ versao: 1, itens: [...] }` | Histórico |
| `equipeNome/Secret/Chave/Ativa/Id` | Config da equipe | Equipe |
| `equipeOnline/Versao/Avisos/AvisosVistos` | Estado da equipe | Equipe |
| `senhasContas` | Senhas das contas | Contas |
| `negModelos/negModalidade/negModeloAtual/negPagamento` | Negociação | Negociação |
| `atModelo` | Modelo ativo | Atendimento |
| `abaPainel/posPainel/tamPainel/tamAmpliado/min` | Interface | Interface |
| `navAuto` | Navegação automática | Navegação |
| `tmExpediente/tmVolume` | Config de timers | Timers |
| `geminiChave/geminiModelo` | Chave e modelo Gemini | RD |
| `rdAnalises/rdLinhas/erBase` | Dados do RD | RD |
| `gruposCfg/buscaAberta/buscaComCac` | Config e busca | Config/Busca |
| `pdRecolhido` | Seção Pendências recolhida (padrão: sim) | Relatório de Pendências |
| `ibImpeditivos` | Impeditivos BMP já encontrados: `{ versao: 1, itens: { proposta: { nome, cpf, data, ts, motivo, vistoEm } } }` (90 dias; "Limpar lista" apaga) | Impeditivos BMP |
| `ibRecolhido` | Seção Impeditivos BMP recolhida (padrão: sim) | Impeditivos BMP |
| `bpPedido` / `bpResposta` | Consulta do ER ao site da Paymobi: `{ id, ts, cpfs }` e `{ id, ts, itens: { cpf: { acesso, em } \| { ausente } \| { erro } }, erro }` | Boleto+ |
| `anPedido` / `anMotor` | Analisar proposta: pedido do painel ao iframe do motor `{ id, ts, nonce }` e o resumo do resultado do motor (os dois apagados no fim da análise) | Analisar proposta |
| `bpVivo` | Hora da última "batida" do site paymobi.com.br aberto (o ER só espera resposta com o site aberto) | Boleto+ |
| `extra[id].cpfCliente / acessoApp / acessoAppEm` | CPF da aba Cliente e o Acesso ao APP do Boleto+ ("Sim"/"Não"/N/A, data do primeiro acesso) | Efetivadas |

---

## 16. Padrões de código obrigatórios

- IIFE única: `(function () { 'use strict'; ... })();`
- Indentação: 2 espaços
- Seções: `/* ================= NOME ================= */`
- Logs: prefixo `[PaymobiFin]` em todos
- Erros esperados: `catch (_) {}` (silencioso)
- Erros críticos: `catch (e) { console.warn('[PaymobiFin] módulo', e && e.message); }`
- Esperas em loops: `sleep(ms)` (nunca `setTimeout` direto)
- Timeouts: sempre em `CONFIG`
- Acessos ao site: sempre em `try/catch`
- Seletor do painel: `$()` (não `document.querySelector`)

---

## 17. Comportamento diante de conflitos de merge

1. Nunca aceitar um lado automaticamente sem entender o contexto
2. Identificar a origem de cada conflito (qual branch, qual commit)
3. Resolver mantendo a funcionalidade mais recente e completa
4. Conflitos envolvendo armazenamento ou criptografia: **interromper e consultar** o responsável

---

## 18. Documentação — quando atualizar

| Arquivo | Atualizar quando |
|---|---|
| `instructions.md` | Novo módulo, novo prefixo, nova chave, nova regra |
| `README.md` | Mudança visível ao usuário, nova dependência |
| `docs/script/<módulo>.md` | Mudança de comportamento, nova função |
| `docs/script/13-rotas-er.md` | Rota do ER ou do motor passa a ser usada, muda ou deixa de ser usada |
| `PRIORIDADES.md` | Tarefa iniciada, concluída ou com decisão nova |
| `CHANGELOG.md` | Gerado automaticamente no PR de release (`release-pr.yml`) |
| `.github/SETUP.md` | Mudança nos workflows ou requisitos de configuração |
| Locais de versão listados no item 12 | Sempre que o `@version` mudar (mesmo commit/PR) |

---

## 19. Comportamento diante de falha no merge automático

Se o merge automático não acontecer (validação falhou, Categoria B, rótulo `sem-auto-merge`, configuração pendente):

1. **Não forçar o merge manualmente** — investigar a causa no log do workflow (GitHub Actions)
2. Validação falhou por causa do código ou do título: corrigir na branch e fazer novo push (ou editar o título)
3. Categoria B: é intencional — aguardar o responsável
4. Falha de configuração (pendência do proprietário): documentar e aguardar
5. O merge só acontece quando **todas** as verificações passam

---

## 20. Checklist obrigatório antes de finalizar uma tarefa

**Escopo**
- [ ] Implementação dentro do escopo solicitado?
- [ ] Nenhuma funcionalidade fora do escopo alterada?

**Código**
- [ ] `'use strict'` presente?
- [ ] `@downloadURL` e `@updateURL` não alterados?
- [ ] Novos timeouts em `CONFIG`?
- [ ] Funções com prefixo correto?
- [ ] `$()` usado dentro do painel?
- [ ] Acessos a `PAGINA.*` em `try/catch`?
- [ ] `sleep()` em loops de espera?
- [ ] Nenhum arquivo de workflow alterado sem autorização?

**Armazenamento**
- [ ] Novas chaves documentadas na tabela do item 15?
- [ ] Nenhuma chave renomeada sem migração?
- [ ] Dados existentes não sobrescritos?

**Git**
- [ ] Em branch temporária (não na `main`)?
- [ ] Commit segue Conventional Commits?
- [ ] Diff revisado antes do commit?
- [ ] Nenhum arquivo fora do escopo incluído?

**Documentação**
- [ ] `docs/script/<módulo>.md` atualizado?
- [ ] `instructions.md` atualizado se novo prefixo/chave criado?
- [ ] `README.md` atualizado se comportamento visível mudou?
- [ ] Se o `@version` mudou, a versão atual foi atualizada nos locais listados no item 12?

**Ao finalizar, apresentar:**
1. Arquivos alterados
2. Chaves de armazenamento afetadas
3. Módulos com impacto indireto
4. Documentação atualizada
5. Pendências (bugs fora do escopo, configurações do proprietário)

---

## 21. Configurações pendentes do proprietário (arthursoaress7)

Ver `.github/SETUP.md` para o passo a passo completo.

| Item | Por que é necessário |
|---|---|
| Secret `RELEASE_TOKEN` (token pessoal) | Para os PRs e merges feitos pelos workflows dispararem a validação e o "Preparar release" |
| Habilitar "Allow auto-merge" | Merge automático espera as verificações em vez de integrar direto |
| "Read and write permissions" e "Allow GitHub Actions to create and approve pull requests" | Workflows criam PRs, tags e releases |
| Proteção da `main` e da `development` com status check `Validar alterações` | Garante que nada entra sem validação, nem por merge manual |

**Riscos sem essas configurações:**
1. Sem proteção, um merge manual na `main` com a validação reprovada ainda passa (aconteceu nos PRs #43 e #44)
2. Sem `RELEASE_TOKEN`, o PR de release fica sem a verificação "Validar alterações". A branch `release/proxima` é atualizada depois de cada merge automático (o merge dispara o "Preparar release"), mas sem "Allow GitHub Actions to create and approve pull requests" o PR de release precisa ser aberto à mão (o resumo da execução traz o link). Aconteceu em 17/11/2026: a correção do endereço do ER ficou na `development` e a `main` parou de iniciar no `paymobi-app.ercard.com.br`

---

## 23. Prioridades de 23/11/2026: documentação viva e logs temporários

Decisão do responsável em 23/11/2026. Tarefas em `PRIORIDADES.md`, P0. O plano anterior (camada/API de acesso ao ER, branch `feature/api-er`) saiu do escopo e seus documentos foram removidos; o histórico continua no Git.

**Documentação acompanha o código (P0.2):**

- Todo PR que muda comportamento, função, chave de armazenamento ou rota usada atualiza no **mesmo PR** os documentos da tabela do item 18.
- PR que altera `paymobifin-assistente.user.js` sem tocar em nenhum documento deve explicar no corpo do PR por que nenhum documento mudou.
- A verificação automática disso no `validate-pr.yml` é Categoria B (workflow): só com autorização explícita.

**Logs temporários (P0.3) — regras para o desenho, a aprovar:**

- Registrar por ação: data/hora, módulo (prefixo), ação, resultado (ok/erro/aviso), duração e motivo do erro.
- **Nunca** registrar dados pessoais de clientes (nome, CPF, telefone, valores de um cliente), senhas, `equipeSecret`/`equipeChave`, cookies, tokens ou respostas do ER/motor. Identificar proposta só se o responsável aprovar.
- Temporário: limite de entradas e de dias, com descarte automático das mais antigas; botão para copiar/exportar e limpar.
- Nova chave de armazenamento = Categoria B (item 5): o desenho é aprovado antes da implementação.
- Os tempos e limites ficam em `CONFIG`.
