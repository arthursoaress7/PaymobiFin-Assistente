# PRIORIDADES.md — Prioridades funcionais do PayMobi

> **Referência central de prioridades.** Consulte este documento antes de solicitar qualquer desenvolvimento.
> Para regras de desenvolvimento, Git e governança, leia [`instructions.md`](./instructions.md).

---

## Como usar este documento com o Claude

1. Consultar `PRIORIDADES.md` (este arquivo).
2. Identificar a tarefa pendente de maior prioridade.
3. Solicitar ao Claude que **analise a implementação atual antes de propor alterações**.
4. Pedir um **plano de implementação** com: arquivos envolvidos, riscos e critérios de aceite.
5. Implementar **somente o escopo da tarefa solicitada**.
6. Revisar as alterações e testar.
7. **Atualizar o status** neste `PRIORIDADES.md` após a conclusão.

O Claude deve respeitar integralmente o `instructions.md` e as regras de governança do repositório. Mudanças críticas seguem o processo de autorização definido na governança. **Não ampliar o escopo por iniciativa própria.**

---

## Regras

- Este documento é a referência central de prioridades funcionais.
- Não propor refatoração estrutural como prioridade sem necessidade comprovada.
- Preservar funcionalidades existentes.
- Concluir e validar uma tarefa antes de avançar para a próxima.
- Registrar novas demandas aqui antes de iniciar seu desenvolvimento.

---

## Mudança de plano (23/11/2026)

**Decisão do responsável:** as prioridades atuais passam a ser:

1. **Otimização das funções existentes.**
2. **Documentação com fluxo automático de atualização:** a documentação sempre acompanha o estado atual do projeto.
3. **Logs temporários:** o script registra como se comportou e o que aconteceu em cada ação, para revisar quando quiser otimizar algo.

**Fora do escopo:** o plano de implementar uma API, que era a P0 de 11/11/2026 (camada única de acesso ao ER, branch `feature/api-er`), e a antiga P4.11 (API oficial do fornecedor e site próprio). Nada disso chegou ao script em produção (`erBuscar` nunca foi implementado). `docs/PLANO-API-ER.md` e `docs/PROPOSTA-CAMADA-ER.md` foram removidos (histórico no Git); o mapeamento das rotas que o script usa hoje continua em [`docs/script/13-rotas-er.md`](./docs/script/13-rotas-er.md). Não retomar sem nova decisão do responsável.

---

## P0 — Prioridades atuais

**Status geral:** Pendente  
**Objetivo:** melhorar o que já existe, com documentação sempre fiel ao código e registro do comportamento do script para orientar as otimizações.

### Tarefas

- [ ] **P0.1 — Otimização das funções existentes**  
  **Status:** Pendente  
  **Objetivo:** tornar mais rápidas, confiáveis e simples as funções já em uso, sem criar funcionalidades novas sem necessidade confirmada.  
  **Como:** as frentes P1 (lembretes), P2 (mensagens), P3 (acordos) e P4 (geral) abaixo são o backlog desta tarefa. Os logs da P0.3 indicam onde atacar primeiro (ações lentas, com erro ou repetidas).  
  **Critérios de aceite:** cada otimização com antes/depois descrito no PR e status atualizado neste arquivo.

- [ ] **P0.2 — Documentação com atualização automática**  
  **Status:** Pendente  
  **Objetivo:** a documentação nunca fica atrás do código.  
  **Como (em etapas):**
  1. Regra (Categoria A, aplicada em 23/11/2026): todo PR que muda comportamento atualiza no mesmo PR os documentos do item 18 do `instructions.md`; PR que altera o script sem tocar em documento explica o porquê (item 23 do `instructions.md`).
  2. Verificação automática no `validate-pr.yml`: avisar ou bloquear PR que altera `paymobifin-assistente.user.js` sem alterar nenhum arquivo em `docs/`, `instructions.md` ou `README.md` (Categoria B: workflow, exige autorização).
  3. Inclui o item já registrado em P4.5: conferir se a versão da documentação bate com o `@version` (a validação já confere `instructions.md`, `CLAUDE.md`, `01-visao-geral.md` e `03-painel-interface.md`).  
  **Critérios de aceite:** etapa 2 aprovada e ativa; nenhum PR integrado com documentação desatualizada.  
  **Decisão pendente:** a verificação da etapa 2 só avisa ou bloqueia o merge?

- [ ] **P0.3 — Logs temporários de comportamento** (Categoria B: nova chave de armazenamento)  
  **Status:** Pendente — aguardando desenho  
  **Objetivo:** registrar, em cada ação do script, o que aconteceu e como ele se comportou, para revisão quando for preciso otimizar.  
  **Comportamento esperado:** cada ação (coleta, CAC, relatórios, mensagens, acordos, lembretes, troca de conta etc.) grava data/hora, módulo, ação, resultado (ok/erro/aviso), duração e motivo do erro. Os registros são temporários (limite de entradas e de dias, descarte automático) e podem ser vistos, copiados/exportados e limpos no painel.  
  **Regras:** sem dados pessoais de clientes, senhas, segredos da equipe, cookies, tokens ou respostas do ER (item 23 do `instructions.md`).  
  **Critérios de aceite:** desenho aprovado pelo responsável (chave, limites, onde visualizar); logs gravados nas ações dos módulos; nenhum dado sensível nos registros; script sem lentidão perceptível.  
  **Decisões pendentes:** por quantos dias guardar e quantas entradas no máximo; se o número da proposta pode aparecer no log; onde ver os logs (aba do painel ou Configurações).

---

## P1 — Otimização dos lembretes

**Status geral:** Pendente  
**Objetivo:** tornar o sistema de lembretes mais eficiente para comunicação e acompanhamento das atividades da equipe.

### Implementação atual (analisada no código)

O módulo de lembretes (`lb*`) já possui:

- **Compartilhamento manual:** `lbCompartilhar(id)` compartilha com a equipe via Pusher. É um botão que o usuário clica explicitamente. Novos lembretes podem nascer compartilhados se a checkbox estiver marcada (padrão: marcada quando a equipe está configurada).
- **Edição sem restrição de autor:** qualquer membro da equipe pode editar, pagar, concluir, cancelar ou excluir um lembrete compartilhado. Não há verificação de `criadoPor`. A resolução de conflitos é por versão (`rev`) e timestamp.
- **Avisos para a equipe:** `eqAvisarLembrete()` já envia notificações para toda a equipe (exceto quem fez a alteração) quando: novo lembrete, pagamento registrado, status mudou, lembrete atualizado ou excluído. Os avisos aparecem num card flutuante no canto superior direito da tela (`#mf-eq-avisos`, `position:fixed; top:16px`).
- **Popup de lembretes do dia:** `lbMostrarPopup()` já é um card flutuante no canto inferior direito da tela (`#mf-lb-popup`, `position:fixed; bottom:16px`). Mostra até 20 lembretes devidos, com botões "Ver lembrete", "Adiar" e "Concluir". Rodapé com "Lembrar mais tarde (1 h)". Dispara também notificação do navegador (uma vez a cada 30 min). Verificado a cada minuto.
- **Renderização principal:** `lbDesenhar()` renderiza a lista dentro da aba "Agenda" do painel — não é flutuante.

### Tarefas

---

- [x] **P1.1 — Notificações para toda a equipe**

  **Status:** Concluído (já implementado; validado no código em 09/11/2026, v13.26.4). Decisão do responsável: o pop-up local no dia do acompanhamento é suficiente, sem push pelo Pusher.  
  **Objetivo:** garantir que todos os integrantes da equipe sejam notificados sobre lembretes, não somente quem criou.

  **Comportamento esperado:** quando um lembrete está vencido ou chega no dia de acompanhamento, todos os membros da equipe conectados devem ver a notificação — não apenas quem criou o lembrete.

  **Como funciona (analisado no código):**
  - **Mudanças:** `eqAvisarLembrete()` avisa toda a equipe (menos quem alterou) sobre novo lembrete, pagamento, mudança de situação, atualização e exclusão de lembretes compartilhados, no card `#mf-eq-avisos`.
  - **Dia do acompanhamento:** um lembrete compartilhado fica no navegador de **todos** os membros. `eqAplicar()` grava o que chega pelo Pusher em `mfLembretes`, e quem estava offline recebe ao reconectar, pela troca de resumos (`eqEnviarResumo` / `eqResumoRecebido`). A cada minuto, cada navegador procura em todos os lembretes, inclusive os criados por outra pessoa, os que vencem hoje ou estão atrasados (`lbVerificarPopup` → `lbAtencao`). Mostra o pop-up `#mf-lb-popup` e, se a permissão foi dada, uma notificação do navegador (no máximo a cada 30 min). Não precisa haver nenhuma alteração no lembrete para o aviso aparecer.
  - **Correção do texto anterior:** não é verdade que os outros membros "só recebem o aviso quando algo muda". Isso vale só para o card da equipe; o pop-up do dia aparece para todos que têm o lembrete compartilhado.

  **Critérios de aceite (atendidos):**
  - Um lembrete compartilhado criado pelo membro A aparece no pop-up do dia do membro B na data de acompanhamento ou depois dela, sem nenhuma alteração no lembrete.
  - Isso vale também quando B estava offline no momento da criação: ele recebe o lembrete ao reconectar.
  - Mudanças em lembretes compartilhados (novo, pagamento, situação, atualização, exclusão) avisam os outros membros no card da equipe.

  **Dependências:** nenhuma.  
  **Limites conhecidos (por design):**
  - Lembretes **não compartilhados** (checkbox desmarcada) só avisam o próprio navegador.
  - Só é avisado quem está com o ER aberto, porque o script só roda lá. Um push pelo Pusher também não alcançaria quem está offline, por isso não foi adotado. Mexer no Pusher seria Categoria B.

  **Possível melhoria futura (não solicitada):** mostrar no pop-up quem criou cada lembrete e se ele é da equipe, e notificar uma vez por lembrete em vez de um resumo a cada 30 min. Registrar em P4.5 se a equipe quiser.

---

- [ ] **P1.2 — Edição compartilhada**

  **Status:** Concluído (já implementado)  
  **Objetivo:** qualquer usuário da equipe deve conseguir editar os lembretes, independentemente de quem os criou.

  **Comportamento atual:** não há restrição de autor. O botão "Editar" aparece para qualquer lembrete pendente, compartilhado ou não. `lbSalvar()` envia via Pusher as alterações em lembretes compartilhados. A resolução de conflitos usa `rev` (versão) + timestamp, e pagamentos/histórico se somam.

  **Critérios de aceite:** já atendidos — qualquer membro pode editar, pagar, concluir, cancelar ou excluir.  
  **Dependências:** nenhuma.  
  **Observações:** confirmar com a equipe se o comportamento atual é suficiente ou se precisam de controle mais granular (ex.: impedir exclusão por quem não criou). Hoje exclusão de lembrete compartilhado exclui para todos, com confirmação.

---

- [ ] **P1.3 — Card flutuante**

  **Status:** Parcialmente implementado  
  **Objetivo:** os lembretes devem ser exibidos em um card móvel pela tela, permitindo acompanhar sem depender de uma aba específica.

  **O que já existe:**
  - `#mf-lb-popup` (canto inferior direito): card flutuante que mostra lembretes devidos hoje. Aparece automaticamente, pode ser adiado ("Lembrar mais tarde 1h"). Não é arrastável — fica fixo no canto.
  - `#mf-eq-avisos` (canto superior direito): card flutuante com notificações da equipe. Tampouco é arrastável.

  **O que pode faltar:**
  - O popup de lembretes do dia some quando não há devidos. Um card flutuante permanente (minimizável, arrastável) mostrando os lembretes ativos seria diferente do comportamento atual?
  - O card deve mostrar **todos** os lembretes ativos, ou só os devidos hoje?
  - O card deve ser arrastável pela tela, ou fixo em um canto (como hoje)?
  - O card deve funcionar fora do ER (ex.: em outras abas do navegador) ou só dentro do painel?

  **Critérios de aceite:** A definir  
  **Dependências:** A definir (depende das decisões acima)  
  **Observações:** o painel principal já é arrastável e redimensionável. Um card de lembretes flutuante adicional seria um novo elemento no `document.body`, similar ao popup atual mas permanente.

---

## P2 — Otimização das mensagens

**Status geral:** Pendente  
**Objetivo:** melhorar as funcionalidades existentes de criação, preenchimento e envio de mensagens.

### Implementação atual (analisada no código)

O módulo de mensagens (`ng*` e `at*`) já possui:

- **Mensagens para o grupo** (Negociação/Renegociação, botão "Mensagem p/ grupo"): uma por modalidade (`NG_MODALIDADES`).
- **3 modelos de mensagem para o cliente** (aba Mensagens, `NG_MODELOS`), um por modalidade + Antecipação: Retorno ao cliente — Renegociação, Retorno ao cliente — Campanha do Fôlego, Antecipação de parcela. A aba abre no retorno da modalidade feita na Negociação, já com os valores. Falta o retorno do PIX Automático (aguardando o texto da equipe). "Retorno do setor" genérico removido em 28/11/2026. Pedido de negociação, Acordo realizado, Campanha do Fôlego e Solicitação de negociação saíram da aba em 17/11/2026 (eram cópias do que o Atendimento gera para o grupo).
- **3 modalidades de acordo** (`NG_MODALIDADES`): Renegociação (entrada + parcelas; padrão; chave `entrada`), Campanha do Fôlego, PIX semanal/Automático. "Condições gerais" removida em 28/11/2026. Renegociação e Fôlego usam o mesmo padrão de título: `*RENEGOCIAÇÃO — <OPERAÇÃO>*` / `*CAMPANHA DO FÔLEGO — <OPERAÇÃO>*`. (Removidas em 19/11/2026: Novo valor mensal, Parcelamento, Alteração de vencimento, Antecipação/quitação, Regularização, Sem acordo e Pagamento parcial + restante — este porque a Campanha do Fôlego faz o mesmo.)
- **Preenchimento automático existente:**
  - `data_contato` = hoje (padrão)
  - `responsavel` = nome da equipe ou apelido da conta ativa
  - `abrangencia` = calculada a partir de `abrangencia_tipo` + `parcela_inicial`/`parcelas_especificas`
  - `antecipacao_descr` = calculada a partir de `tipo_antecipacao` + `qtd_antecipadas`
  - `valor_restante` = `valor_total_juros` − `valor_pago_parcial` (Campanha do Fôlego) e `valor_total_juros` − `valor_entrada` (retorno do Fôlego, aba Mensagens): auto-calculado, bloqueia pagar acima do total
  - `parcela_negociada` (Campanha do Fôlego): o número digitado sai por extenso ("3" → "3° parcela")
  - Dados de pagamento (`chave_pix`, `banco`, `agência`, `conta`) vindos de Configurações
  - `atSugerir()`: copia data 1° pagamento, quantidade de parcelas e "Todo Contrato" dos campos do acordo (só se estiverem vazios)
- **Preenchimento manual:** todo o resto — nome, proposta, CPF, valores das parcelas, entrada, datas, quantidades, observações, vencimentos.
- **Modelos editáveis:** o usuário pode personalizar o texto de qualquer modelo em Configurações > Gerador de negociações. Os modelos personalizados ficam em `negModelos`.

### Tarefas

---

- [ ] **P2.1 — Revisar os modelos de mensagens existentes**

  **Status:** Pendente  
  **Objetivo:** avaliar se os modelos (3 para o cliente) e as 3 modalidades atuais atendem às necessidades da equipe.  
  **Comportamento esperado:** modelos cobrem todos os cenários do dia a dia sem precisar edição manual excessiva.  
  **Critérios de aceite:** lista de modelos que precisam ajuste, novos modelos necessários ou modelos obsoletos.  
  **Dependências:** feedback da equipe sobre o uso real.  
  **Observações:** não criar modelos novos sem necessidade confirmada.  
  **Andamento (17/11/2026):** mensagens do grupo e do cliente separadas (#58, #60); Campanha do Fôlego no formato aprovado pela gestora, com quebra de acordo opcional (#59, #61, #62). Falta revisar os textos dos 3 modelos para o cliente (ex.: "Retorno do setor" ainda tem cabeçalho de texto interno).

---

- [ ] **P2.2 — Melhorar o preenchimento automático dos campos**

  **Status:** Pendente  
  **Objetivo:** reduzir digitação manual identificando campos que poderiam ser preenchidos automaticamente a partir de dados já disponíveis.  
  **Comportamento esperado:** ao aplicar o cliente atual (`atAplicarCliente`), o máximo de campos relevantes deve vir preenchido.  
  **Critérios de aceite:** lista de campos que hoje são manuais mas poderiam ser automáticos, com origem do dado.  
  **Dependências:** `atAplicarCliente` (Negociação, Renegociação e Mensagens) já preenche: nome, proposta, CPF, IMEI, data da proposta, filial, atendente, valor da parcela, FPD, parcelas em atraso, dias de atraso, vencimento mais antigo, parcela atual. O Número de contato é sempre manual (por design).  
  **Observações:** o número de contato (RD Station) nunca deve ser preenchido com o telefone da proposta — isso é uma regra confirmada.

---

- [ ] **P2.3 — Padronizar as mensagens utilizadas pela equipe**

  **Status:** Pendente  
  **Objetivo:** garantir que todos os membros usem os mesmos modelos e formatos.  
  **Comportamento esperado:** mensagens geradas sejam consistentes independentemente de quem as gerou.  
  **Critérios de aceite:** equipe confirma que os textos gerados estão padronizados e corretos.  
  **Dependências:** feedback da equipe.  
  **Observações:** os modelos personalizados (`negModelos`) são por navegador — não são compartilhados via Pusher.

---

- [ ] **P2.4 — Identificar oportunidades de reduzir preenchimentos manuais e erros**

  **Status:** Pendente  
  **Objetivo:** mapear onde os erros mais acontecem e onde a automação pode ajudar.  
  **Comportamento esperado:** relatório de pontos de fricção no fluxo de mensagens.  
  **Critérios de aceite:** lista de melhorias identificadas com prioridade.  
  **Dependências:** observação do uso real pela equipe.  
  **Observações:** prioridade é otimizar o que já existe, não criar funcionalidades novas.

---

## P3 — Otimização da geração de acordos

**Status geral:** Pendente  
**Objetivo:** melhorar o processo de geração e formalização de acordos.

### Implementação atual (analisada no código)

O módulo de geração de acordos (`at*`) já possui:

- **Plano de parcelas sugerido:** `atSugestao()` gera o plano a partir da modalidade. `atPlanoCampos()` preenche campos vazios com a sugestão, preservando edições manuais.
- **Cálculos automáticos:**
  - Vencimentos das parcelas: `atSomarMeses` (mensal, corrige dia 31 em mês curto) e `atSomarDias` (semanal, soma 7 dias)
  - 1º vencimento: próximo "dia D" após a data de entrada (auto-calculado quando entrada e dia de vencimento estão informados)
  - Rótulos de ação: "Pagamento Parcela 01/24" (mensal) ou "Pagamentos 01/88" (PIX semanal)
  - Situação padrão: "Efetivado (Entrada)" na 1ª linha, "Em Aberto" nas demais, "Efetivado (Quitação)" na quitação
  - Valor pago: valor da 1ª linha, "N/A" nas demais
  - Saldo restante na Campanha do Fôlego: total com juros − pago (bloqueia pagar acima do total)
- **Edição manual preservada:** tudo no plano é editável. Campos que o usuário já preencheu não são sobrescritos pela sugestão.
- **Cópia para planilha:** `atLinhas()` monta as linhas tabuladas — Negociação (1 linha, 30 colunas) ou Renegociação (1 linha por parcela, 27 colunas).
- **Manual:** quantidades, valores, data da entrada, 1º vencimento (embora sugerido), observações, procedimento na régua, recebeu boletos, negociação concluída.

### Tarefas

---

- [ ] **P3.1 — Revisar o fluxo atual de geração de acordos**

  **Status:** Pendente  
  **Objetivo:** mapear o fluxo completo e identificar gargalos.  
  **Comportamento esperado:** documentação do fluxo atual com pontos de melhoria.  
  **Critérios de aceite:** fluxo documentado e gargalos identificados.  
  **Dependências:** nenhuma.  
  **Observações:** o fluxo já é bem automatizado — foco em identificar o que ainda é manual desnecessariamente.

---

- [ ] **P3.2 — Identificar oportunidades de automatizar cálculos e preenchimentos**

  **Status:** Pendente  
  **Objetivo:** reduzir etapas manuais no preenchimento do acordo e do plano de parcelas.  
  **Comportamento esperado:** lista de campos que poderiam ser calculados ou preenchidos automaticamente.  
  **Critérios de aceite:** melhorias identificadas com viabilidade técnica.  
  **Dependências:** análise do fluxo atual (P3.1).  
  **Observações:** o script já calcula vencimentos, situações, rótulos e saldo. Possíveis melhorias: cálculo de juros no restante, preenchimento automático de "procedimento na régua" e "recebeu boletos" baseado na modalidade.

  **Avanço (23/11/2026):** primeira melhoria entregue: **calculadora de débito atualizado** no Atendimento (grupo "Débito atualizado (calculadora)", `atDeb*`; detalhes em `docs/script/04-atendimento.md`). Calcula encargos de mora e multa das parcelas em atraso com a regra do "Registro de Pagamento de Fatura" do ER e leva o total para "Valor total com juros".  
  **Decisão (25/11/2026):** o responsável aprovou as taxas de `AT_ENCARGOS` (calibradas em 23/11/2026) como base padrão da calculadora para todos os contratos. Mudar só se o ER mostrar outro valor.  
  **Pendente na calculadora:** tratar parcela com pagamento parcial; avaliar ler as parcelas direto da Consulta CAC (vencimento e saldo de cada uma) em vez de supor vencimentos mensais.

---

- [ ] **P3.3 — Melhorar a consistência das informações geradas**

  **Status:** Pendente  
  **Objetivo:** garantir que os dados do acordo, do plano de parcelas e da planilha estejam sempre consistentes.  
  **Comportamento esperado:** nenhum campo da planilha com valor divergente do que está na ficha.  
  **Critérios de aceite:** auditoria de consistência entre ficha, plano e linhas copiadas.  
  **Dependências:** nenhuma.  
  **Observações:** `atValidar()` já verifica campos obrigatórios. Possível melhoria: validar consistência entre modalidade e plano (ex.: PIX semanal com período mensal).

---

- [ ] **P3.4 — Reduzir erros e etapas manuais**

  **Status:** Pendente  
  **Objetivo:** diminuir erros de digitação e etapas desnecessárias.  
  **Comportamento esperado:** menos cliques e menos digitação para concluir um registro.  
  **Critérios de aceite:** lista de etapas que podem ser eliminadas ou combinadas.  
  **Dependências:** análise do fluxo (P3.1).  
  **Observações:** `atSugerir()` já combina 3 preenchimentos. Possível: auto-gerar textos ao montar o plano, auto-copiar ao concluir.

---

- [ ] **P3.6 — Fluxo da Campanha do Fôlego até o cumprimento do acordo**

  **Status:** Implementado (26/11/2026), aguardando teste da equipe  
  **Objetivo:** acompanhar os acordos do Fôlego até o cliente cumprir e avisar o grupo.  
  **Comportamento:** Renegociação com Fôlego e Resultado "fechou" vai direto para o histórico ao copiar a mensagem ou a linha; aba Histórico lista os acordos em aberto com "Cliente cumpriu", que finaliza o registro, conclui o lembrete pendente da proposta e gera a mensagem `*CAMPANHA DO FÔLEGO — ACORDO CUMPRIDO*` (data do acordo, nome, CPF buscado no ER). Detalhes em `docs/script/05-historico.md`.  
  **A confirmar com o responsável:** título da mensagem, data usada ("realizado no dia" = data do registro) e busca do CPF pela proposta (assumidos na implementação).

---

- [ ] **P3.5 — Documentar melhorias necessárias identificadas na análise do código**

  **Status:** Pendente  
  **Objetivo:** registrar tecnicamente o que foi encontrado para referência futura.  
  **Comportamento esperado:** documento com as melhorias e suas prioridades.  
  **Critérios de aceite:** documento atualizado em `docs/script/04-atendimento.md` se necessário.  
  **Dependências:** conclusão das análises P3.1 a P3.4.  
  **Observações:** não criar documento novo — atualizar o existente.

---

## P4 — Otimização geral do sistema

**Status geral:** Pendente  
**Objetivo:** aperfeiçoar funcionalidades já existentes no PayMobi.

### Tarefas

---

- [ ] **P4.1 — Identificar funcionalidades com lentidão ou dificuldade de uso**

  **Status:** Pendente  
  **Objetivo:** mapear pontos onde a experiência é lenta ou confusa.  
  **Comportamento esperado:** lista de funcionalidades com problema, com descrição do sintoma.  
  **Critérios de aceite:** itens priorizados pela equipe.  
  **Dependências:** feedback da equipe sobre o uso diário.  
  **Observações:** a coleta de auditoria já tem timeouts configuráveis em `CONFIG`. Possíveis pontos: tempo de abertura da árvore, lentidão do painel com muitos lembretes, troca de conta.

---

- [ ] **P4.2 — Identificar tarefas repetitivas que podem ser automatizadas**

  **Status:** Pendente  
  **Objetivo:** listar operações manuais repetitivas que o script poderia fazer sozinhas.  
  **Comportamento esperado:** lista de tarefas com estimativa de viabilidade.  
  **Critérios de aceite:** tarefas priorizadas pela equipe.  
  **Dependências:** observação do uso diário.  
  **Observações:** o script já automatiza coleta, navegação, troca de conta, login automático, timers de expediente. Foco no que ainda é manual e repetitivo.

---

- [ ] **P4.3 — Melhorar a experiência de uso**

  **Status:** Pendente  
  **Objetivo:** ajustes de usabilidade no painel e nos fluxos.  
  **Comportamento esperado:** interface mais fluida e intuitiva.  
  **Critérios de aceite:** melhorias validadas pela equipe.  
  **Dependências:** itens identificados em P4.1 e P4.2.  
  **Observações:** a Nova UI já melhorou muito a experiência. Foco em detalhes que ainda incomodam.

---

- [ ] **P4.4 — Corrigir problemas identificados**

  **Status:** Pendente  
  **Objetivo:** corrigir bugs e comportamentos inadequados.  
  **Comportamento esperado:** bugs documentados e corrigidos.  
  **Critérios de aceite:** bug corrigido e validado.  
  **Dependências:** bugs reportados pela equipe.  
  **Observações:** registrar cada bug como item separado quando identificado.

---

- [ ] **P4.5 — Registrar oportunidades futuras de melhoria**

  **Status:** Pendente  
  **Objetivo:** manter um backlog de ideias que surgirem do uso.  
  **Comportamento esperado:** seção atualizada continuamente.  
  **Critérios de aceite:** ideia registrada com descrição e contexto.  
  **Dependências:** nenhuma.  
  **Observações:** esta prioridade é alimentada continuamente conforme a equipe utiliza o sistema.

  **Backlog:**
  - Fazer o validate-pr.yml verificar se a versão da documentação bate com o @version — Categoria B, exige autorização.

---

- [x] **P4.6 — Relatório de pendências (conta master)**

  **Status:** Concluído (v13.27.0, 09/11/2026)  
  **Objetivo:** gerar, sem copiar à mão da Fila de Pendências, a lista das propostas com pendência em aberto no formato usado pela equipe (Proposta, Pendência, Cliente, Vendedor, Autorização, Data, Loja).  
  **Comportamento:** botão "Gerar relatório" na seção Pendências da aba Relatórios. Lê a Fila de Pendências (filtro "TODAS PENDENTES") e o detalhe de cada autorização, só leitura. Troca para a conta master quando ela está configurada.  
  **Critérios de aceite (atendidos):** o texto gerado bate com o exemplo da equipe (propostas 5017 e 4959 conferidas no ER real); pendências antigas ainda abertas também entram; fila vazia, sessão expirada e conta sem acesso dão aviso em vez de erro.  
  **Dependências:** conta master com acesso à Fila de Pendências.  
  **Observações:** detalhes técnicos em `docs/script/03-painel-interface.md` (Relatório de Pendências).

- [x] **P4.7 — Nome do arquivo do script**

  **Status:** Concluído (v13.27.2, 09/11/2026)  
  **Objetivo:** o arquivo principal passa de `automacaomobfacil.user.js` para `paymobifin-assistente.user.js`, nome que bate com o que aparece no Tampermonkey ("PaymobiFin - Assistente").  
  **Comportamento:** `@downloadURL`/`@updateURL` apontam para o arquivo novo. O arquivo antigo ficou como ponte congelada (cópia idêntica da 13.27.2), para que as instalações antigas atualizem e passem a seguir o arquivo novo. `@name` e `@namespace` não mudaram, então o Tampermonkey trata como o mesmo script e os dados salvos continuam.  
  **Critérios de aceite:** os workflows (validação e release) leem o arquivo novo; a validação confere que os endereços de atualização apontam para o arquivo novo e bloqueia alterações no arquivo-ponte.  
  **Observações:** alteração Categoria B (endereços de atualização e workflows), autorizada pelo responsável.

- [x] **P4.8 — Remover o arquivo-ponte `automacaomobfacil.user.js`**

  **Status:** Concluído (19/11/2026) — removido antes da data prevista (09/12/2026) por decisão do responsável, que confirmou a migração da equipe  
  **Objetivo:** apagar o nome antigo depois que todas as instalações já migraram para `paymobifin-assistente.user.js`.  
  **Critérios de aceite:** confirmar com a equipe que o Tampermonkey de cada pessoa mostra a versão 13.27.2 ou mais nova e o endereço de atualização terminando em `paymobifin-assistente.user.js`.  
  **Riscos:** quem ainda estiver no endereço antigo deixa de receber atualizações sem aviso.  
  **Observações:** alteração Categoria B (afeta a distribuição), autorizada pelo responsável em 19/11/2026. A validação (`validate-pr.yml`) continua tratando esse nome como Categoria B; não foi alterada.

- [x] **P4.9 — Compatibilidade com a atualização do ER (sessão expirada)**

  **Status:** Concluído (v13.29.0, 11/11/2026)  
  **Objetivo:** recuperar o funcionamento do painel depois da atualização do ER (visual novo, CSS de 05/11/2026), sem mudar o que continua funcionando.  
  **Análise (ER real, só leitura):** Mesa de Crédito, Ler da tela, navegação automática com datas do dia, Relatório Diário (valor, vendedor, IMEI), Auditoria no motor MobFacil, CCBs (até antes do download), busca de cliente (proposta, CPF, nome), Aplicar cliente atual, CAC, Relatório de Pendências, tela de login e modo escuro sobre o ER continuam com a mesma estrutura e funcionando. O que mudou: com a sessão vencida, o ER não abre mais a janela `#dlgRelogin`; redireciona toda requisição para `/AvisoSessao/AvisoSessao` ("Sua sessão de trabalho expirou").  
  **Correções:** `erSemSessao` reconhece esse aviso (Diário, Auditoria, aba Cliente, CAC, Pendências, busca, CCBs, RD); a fila pausa com aviso em vez de gravar N/A e concluir; o login automático volta a disparar; a navegação automática diz na hora quando a sessão expirou ou quando a conta não é aprovadora da Mesa ("Aprovador CONTA não cadastrado").  
  **Critérios de aceite (atendidos no ER real):** Diário com a sessão vencida pausa sem gravar dados (a versão anterior marcava as 3 propostas como feitas e copiava tudo N/A); Pendências e CCBs avisam a sessão expirada; o login automático leva à tela de login; nenhuma página normal é confundida com sessão expirada.  
  **Limitações:** a entrada automática com senha salva não foi testada (exige digitar senha no ER real); o RD (app.tallos.com.br) não foi aberto, só os endereços do ER que ele usa.

- [x] **P4.10 — Desbloqueios no arquivo principal**

  **Status:** Concluído (v13.29.0, 11/11/2026)  
  **Objetivo:** a aba Desbloqueios do Atendimento (#35, release v13.28.0) foi feita no arquivo-ponte `automacaomobfacil.user.js`, que ninguém mais recebe, e sem mudar o `@version`: não chegou a nenhum usuário.  
  **Comportamento:** o mesmo código do #35, sem alteração, levado para `paymobifin-assistente.user.js`; o arquivo-ponte voltou a ser a cópia congelada da 13.27.2. Documentação do módulo em `docs/script/12-desbloqueios.md`.  
  **Critérios de aceite:** a aba Desbloqueios aparece no Atendimento, "Aplicar cliente atual" preenche os campos do cliente e "Copiar linha da planilha" gera a linha de 15 colunas.

- [ ] **P4.12 — Investigar protocolos de atendimento abertos pela consulta CAC**

  **Status:** Diagnóstico concluído (12/11/2026); tratamento aguardando decisão do responsável. Dica da opção e `docs/script/07-cac.md` corrigidos no PR #42.  
  **Diagnóstico (HARs de 11 e 12/11/2026):**
  - O número do protocolo nasce no `BuscaCartao` (a tela o mostra no botão azul "Protocolo: N" e o repassa ao `CentralInicio`).
  - Cada `BuscaCartao` gera um número novo, inclusive para o mesmo cliente aberto duas vezes seguidas (`…255` e `…256`). A numeração é única para todo o ER.
  - A tela do ER encerra o protocolo ao fechar a janela do cliente (`CentralFinalizaProtocolo`).
  - O painel chama `BuscaCartao` em toda consulta (`cacAbrir`), com a mesma requisição da tela, e nunca encerra o protocolo.
  - **Conclusão:** cada consulta CAC pelo painel deixa um protocolo de atendimento aberto no ER, com a opção "Abrir a Posição Geral" ligada ou desligada. Obtida por equivalência das requisições; o teste isolado pelo painel ficou inconclusivo porque um colega usou o CAC durante o teste.  
  **Teste de confirmação (opcional):** no ER, abrir um cliente no CAC e anotar N; fechar; logo em seguida, uma consulta CAC pelo painel; logo em seguida, abrir um cliente no ER. N + 2 confirma. Tudo em menos de um minuto.  
  **Decisão pendente (tratamento):**
  1. Só avisar a equipe e manter como está.
  2. O painel encerrar o protocolo ao fim da consulta, como a tela faz (Categoria B: altera dados no ER).
  3. Procurar uma rota do CAC que traga os dados sem abrir protocolo (nenhuma encontrada nos HARs).  
  **Critérios de aceite:** tratamento escolhido pelo responsável, implementado (se houver) e documentado em `docs/script/07-cac.md`.  
  **Observações:** chamar `CentralFinalizaProtocolo` altera dados no ER: só com autorização explícita.

---

- [ ] **P4.13 — Testes automatizados e cobertura mínima no CI (evolução futura)**

  **Status:** Ideia registrada (12/11/2026)  
  **Objetivo:** bloquear o merge quando algum teste falhar e, depois, exigir cobertura mínima (ex.: 80%) nas partes testáveis do script.  
  **Como (em etapas):**
  1. Estrutura de testes em Node (sem build) e exemplos de resposta do ER/motor tirados dos HARs de 11 e 12/11/2026, com dados pessoais trocados por fictícios.
  2. Testes de cálculos puros já existentes (vencimentos, conversão de valores) e das funções que leem as respostas do ER.
  3. `validate-pr.yml` rodando os testes também em PRs para `development` (Categoria B: workflow).
  4. Cobertura mínima aplicada só às funções testáveis (80% do arquivo inteiro não é realista: boa parte é interface e cliques no ER).  
  **Dependências:** funções testáveis no nível de cima do script; proteção de branch com o teste como verificação obrigatória (configuração do proprietário, ver `.github/SETUP.md`).  
  **Observações:** o script precisa de um gancho mínimo que exponha essas funções só quando roda nos testes (nunca no navegador).

---

- ~~**P4.11 — API oficial do ER e site próprio**~~ — **Fora do escopo** (decisão de 23/11/2026).

- [x] **P4.14 — Relatório de Impeditivos BMP**

  **Status:** Concluído (21/11/2026)  
  **Objetivo:** gerar, sem copiar à mão, a lista de clientes impedidos pelo KYC da BMP no formato da equipe (`*IMPEDITIVOS BMP*` + NOME, CPF, DATA e MOTIVO).  
  **Comportamento:** aba Relatórios > seção Impeditivos BMP. Lê a Fila de Propostas do período (ícone 🚫, todos os motivos) e a data/hora da efetivação no Histórico da Mesa. O que encontra fica guardado no navegador (o ícone some da Fila depois de um tempo) até "Limpar lista".  
  **Critérios de aceite (atendidos no ER real):** a lista de 13/11 a 21/11 bate com a da equipe em nome, CPF e data (5475, 5440, 5452, 5188...); gerar de novo não duplica; Copiar e Limpar lista funcionam.  
  **Limitação:** quem perdeu o ícone antes da primeira geração (ex.: 5211 e 5241) não aparece: a partir de agora, gerar com frequência guarda os casos.

---

- [ ] **P4.15 — Lista de clientes para a negativação PEFIN (Serasa)**

  **Status:** Implementado (25/11/2026), aguardando teste no ER real  
  **Objetivo:** parar de digitar à mão os dados do cliente no formulário do gerador do arquivo PEFIN (layout de 600 posições).  
  **Comportamento:** aba CAC > seção Negativação PEFIN. Adicionar por CPF/nome ou colar vários CPFs; cadastro e endereço da aba Cliente da Mesa de Crédito; vencimento mais antigo, término e valor (parcelas em aberto, vencidas + a vencer, pelo valor original, sem juros) do CAC. Botões Copiar e Copiar todos, em texto ou CSV, legível ou no formato do arquivo. Detalhes em `docs/script/07-cac.md`.  
  **Decisões do responsável (25/11/2026):** as duas formas de entrada; dívida pelo CAC; valor = parcelas em aberto (vencidas + a vencer) pelo valor original, sem juros, sem as pagas; lista só na tela (sem chave nova de armazenamento).  
  **Critérios de aceite:** em clientes reais, os 14 campos batem com o ER; o formato do arquivo é aceito pelo gerador; CSV abre certo na planilha.  
  **Pendências:** confirmar os rótulos do endereço da aba Cliente (o cartão avisa os campos não achados e os rótulos lidos) e o formato de cada campo no gerador. Cada cliente consultado abre um protocolo no ER (P4.12).

---

## Decisões do responsável (12/11/2026)

| # | Tema | Decisão |
|---|---|---|
| A | Proposta da camada `er` (antiga P0.2) | **Cancelada em 23/11/2026:** plano de API fora do escopo |
| B | Arquitetura (módulos com build, gerenciamento de estado) | **Decidir depois de mapear todo o projeto.** Até lá, tudo dentro do monólito |
| C | Tratamento dos protocolos abertos pelo CAC (P4.12) | **Em revisão:** rever as opções antes de decidir |
| E | Configurações do proprietário (`.github/SETUP.md`) | **Aprovado:** aplicar quando o proprietário estiver disponível. Até lá, o fluxo de merges funciona em modo parcial |
| F | Uso das rotas internas × contrato com o fornecedor (dúvida 9) | **Em estudo** pelo responsável |
| G | Dúvidas de P1 a P3 (tabela abaixo) | Retomadas com a P0.1 (otimização das funções existentes) |

---

## Dúvidas funcionais que precisam de decisão da equipe

| # | Dúvida | Contexto |
|---|---|---|
| 1 | ~~Notificação no vencimento deve ser push ativo para todos, ou popup local já é suficiente?~~ **Decidido (09/11/2026): o pop-up local é suficiente.** | P1.1 — o pop-up do dia já aparece para todos os membros que têm o lembrete compartilhado |
| 2 | O card flutuante deve ser permanente (mesmo sem devidos) ou só quando há lembretes do dia? | P1.3 — hoje o popup só aparece com devidos |
| 3 | O card flutuante deve ser arrastável ou fixo em um canto? | P1.3 — hoje é fixo |
| 4 | O card flutuante deve mostrar todos os lembretes ativos ou só os devidos hoje? | P1.3 — hoje só mostra devidos |
| 5 | Devemos impedir que alguém exclua um lembrete compartilhado que não criou? | P1.2 — hoje qualquer um pode excluir |
| 6 | Os modelos personalizados (`negModelos`) devem ser compartilhados entre a equipe? | P2.3 — hoje são por navegador |
| 7 | Há campos do acordo que poderiam ser preenchidos automaticamente a partir do ER? | P3.2 — hoje muitos campos são manuais |
| 8 | ~~O ER autentica por cookie ou por token?~~ **Respondido no código (11/11/2026): cookie de sessão.** | As rotas são chamadas com `credentials: 'include'` |
| 9 | O uso das rotas internas do ER é compatível com o contrato e os termos de uso? | Uso atual do script — em estudo pelo responsável (decisão F de 12/11/2026) |
