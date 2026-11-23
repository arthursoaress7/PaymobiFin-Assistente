# Módulo: Atendimento (Negociação e Renegociação)

**Linhas:** 5537 – 5900  
**Gerador de textos:** 4092 – 4590 (modelos, campos, `ngValor`, `ngMontar`); `ngFrases` logo antes do Atendimento  
**Funções principais:** `atDesenhar()`, `atLinhas()`, `atMontarPlano()`, `ngGerarTextos()`, `ngFrases()`

---

## Responsabilidade

O módulo de Atendimento centraliza o registro de negociações e renegociações. Ele mantém uma **ficha única em memória** que serve a Negociação, a Renegociação e a aba Mensagens — o que foi digitado em uma aparece nas outras. Nada é salvo até clicar em "Registrar no histórico".

O Atendimento tem 4 telas (`.mf-at-seg`): **Negociação**, **Renegociação**, **Desbloqueios** (`12-desbloqueios.md`) e **Mensagens** (retorno ao cliente — ver "Mensagens: grupo × cliente").

---

## Modelos

### Modelo 1 — Negociação

Campos organizados em blocos:

| Bloco | Campos |
|---|---|
| **Cliente** | Proposta\*, Nome\*, CPF, IMEI, Data da proposta, Filial, Atendente, Número de contato (RD Station)\*, Quem negociou, Data do contato (padrão: hoje) |
| **Cobrança** | Valor parcela, FPD, Parcelas em atraso, Dias atraso, Vencimento mais antigo, Parcela atual |
| **Débito atualizado (calculadora)** | Valor da parcela, Parcelas em atraso, Vencimento mais antigo, Data do pagamento → tabela por parcela com encargos, multa e total (ver "Débito atualizado") |
| **Acordo** | Resultado\* (fechou / pagou tudo / solicitou / não concluída) + Modalidade\* + campos da modalidade |
| **Textos** | Pedido do cliente, Acordo Realizado, A partir de qual parcela |
| **Acompanhamento** | 12 colunas opcionais (botão "Trazer do acordo" preenche data e quantidade) |

### Modelo 2 — Renegociação

Aproveita o bloco Cliente do Modelo 1 e acrescenta:

| Bloco | Campos |
|---|---|
| **Acordo** | Resultado\* (as mesmas opções da Negociação) + Modalidade\* |
| **Textos** | Acordo Solicitado, Acordo Realizado |
| **Renegociação** | Responsável\*, Formalizou (SIM), Data renegociação, Contrato novo, Observação, Régua, Recebeu boletos, Concluída |
| **Plano de parcelas** | Tabela editável gerada automaticamente a partir do acordo |
| **Débito atualizado (calculadora)** | O mesmo grupo da Negociação (fica visível nas duas telas) |

> **Número de contato:** é o número usado pelo cliente no RD Station. O telefone do cadastro da proposta nunca preenche esse campo automaticamente.

---

## Modalidades de acordo

Definidas em `NG_MODALIDADES`. Cada modalidade tem campos próprios, obrigatórios, texto de acordo e frases.

| Chave | Modalidade |
|---|---|
| `entrada` | Renegociação (entrada + parcelas) — padrão |
| `folego` | Campanha do Fôlego |
| `pix` | PIX semanal / Automático |
| `geral` | Condições gerais (modelo anterior) |

**Removidas em 19/11/2026** (mensagens excessivas): `mensal` (Novo valor mensal), `parcelamento`, `vencimento` (Alteração da data de vencimento), `antecipacao` (Antecipação / quitação), `regularizacao`, `semacordo` (Sem acordo) e `parcial` (Pagamento parcial + restante: a Campanha do Fôlego faz o mesmo). Modalidade padrão: `entrada` (Renegociação). Registros antigos do histórico com essas modalidades continuam mostrando o nome (`NG_MOD_REMOVIDAS` / `ngModNome`); "Abrir no Atendimento" num deles abre em Renegociação, com os valores do registro. Textos personalizados delas salvos em `negModelos` ficam guardados, sem uso.

**Campanha do Fôlego — formato da mensagem (aprovado pela gestora, 17/11/2026):** título `*CAMPANHA DO FÔLEGO*` (negrito no WhatsApp); Cliente, CPF, Parcela(s) negociada(s), Valor Original da Parcela, Vencimento; "Cliente pagou parcialmente R$ X (data)."; "Valor total com juros"; dia e valor do restante. Campos da modalidade: `parcela_negociada` (digita-se o número: "3" sai "3° parcela", "8, 9 e 10" sai "8°, 9° e 10° parcelas"), `vencimento` ("Vencimento da parcela negociada"), valor/data do pagamento parcial e valor/dia do restante; Cliente, CPF e Valor original vêm dos blocos fixos (na Renegociação, que esconde o bloco Cobrança, o "Valor original da parcela" aparece na própria modalidade). Campo `valor_total_juros` ("Valor total com juros", obrigatório): o **valor do restante é calculado** = total com juros − valor pago parcialmente (`saldo: 'valor_total_juros'` em `ngSaldoHook`; fica só leitura enquanto os dois estão preenchidos; pagamento maior que o total dá erro). (Atualizado em 19/11/2026.)

**Quebra de acordo (opcional):** o checkbox **"Houve quebra de acordo"** (`quebra_acordo`, depois de "Dia do pagamento do restante", desmarcado por padrão) libera os campos do pagamento anterior (`valor_pago_anterior`, `data_pago_anterior`) e acrescenta à mensagem ", após quebra de acordo" e a linha "Anteriormente o cliente pagou R$ X (data) sobre o valor da parcela." Desmarcado, os campos ficam ocultos e não entram no texto mesmo que tenham sido preenchidos.

**Trecho opcional nos modelos:** `{{?variavel}}` some quando vazio sem tirar a linha (ex.: `{{?quebra_acordo_txt}}`). `{{variavel}}` sem `?` continua tirando a linha quando vazio.

---

## Geração de textos

`ngGerarTextos(silencioso)` — monta os textos "Pedido do cliente" e "Acordo Realizado" com base na modalidade e nos campos preenchidos.

`ngFrases(tipo)` — retorna a frase do acordo para a modalidade atual, variando conforme o resultado:

| Resultado | Frase gerada |
|---|---|
| `fechou` | "Cliente fechou acordo de…" |
| `finalizou` | Mesma frase de `fechou` (o cliente pagou tudo) |
| `solicitou` | "Cliente solicitou…" (simulação) |
| `não concluída` | Apenas o pedido, sem frase de acordo |

A frase "Acordo Realizado" é idêntica à gerada na v11.38 (regressão validada com 14 casos).

**Operação na mensagem do grupo (19/11/2026):** o título da mensagem diz a operação escolhida em Resultado, nas duas telas (Negociação e Renegociação). Variáveis calculadas `{{operacao}}` ("Acordo solicitado" / "Acordo realizado" / "Acordo finalizado" / "Negociação não concluída") e `{{operacao_maiusc}}` (o mesmo em maiúsculas). Títulos: Renegociação `*RENEGOCIAÇÃO — {{operacao_maiusc}}*` e Campanha do Fôlego `*CAMPANHA DO FÔLEGO — {{operacao_maiusc}}*` (mesmo padrão); PIX semanal usa `NG_CAB` = `{{operacao}} — <forma>`; acordo geral: `{{operacao}}`. Na Campanha do Fôlego, `{{acao_parcial}}` troca "Cliente pagou parcialmente" por "Cliente solicita pagar parcialmente" quando o Resultado é solicitou ou não concluída. Fora do Atendimento (aba Mensagens) as variáveis ficam vazias. Textos personalizados salvos em `negModelos` antes dessa mudança não têm a variável: restaurar o padrão em Configurações. Status no histórico: fechou → `realizado`, pagou tudo → `finalizado`, solicitou → `pendente`, não concluída → `nao_concluido`.

---

## Plano de parcelas (Renegociação)

`atSugestao()` — sugere as linhas do plano com base na modalidade. Todas são editáveis antes de copiar.

`atMontarPlano()` — monta a lista de parcelas com campos: Ação, Valor, Vencimento, Situação, Valor Pago.

`atPlanoDesenhar()` — renderiza a tabela editável do plano.

### Regras das situações

- 1ª linha sempre = Entrada: situação `Efetivado (Entrada)`
- Na quitação: `Efetivado (Quitação)`
- Demais linhas: `Em Aberto`, Valor Pago = `N/A`

### Sugestão por modalidade

| Modalidade | Linhas sugeridas |
|---|---|
| Renegociação (`entrada`) | Mensagem do grupo no formato da equipe (19/11/2026): título no padrão do Fôlego, `*RENEGOCIAÇÃO — {{operacao_maiusc}}*` (ex.: `*RENEGOCIAÇÃO — ACORDO SOLICITADO*`), cliente/CPF, parcela atual e abrangência, o pedido do cliente ("Cliente solicitou pagamento todo dia X e parcelas de R$ Y" — campos `dia_solicitado` e `valor_solicitado`), a entrada (opcional: some se vazia; data entre parênteses via `data_entrada_txt`), "Foram simuladas N parcelas de R$ Z." e "Vencimento: todo dia D". Obrigatórios: nome, quantidade e valor da parcela simulada |
| PIX semanal / Automático | N pagamentos semanais (+ entrada se informada) |
| Campanha do Fôlego | 2 linhas: valor agora + saldo restante |

> **Dia 31 em mês curto:** cai no último dia do mês (ex.: 28/02).

### Datas automáticas do plano

A **Data renegociação** (data do contrato novo, a mesma do comprovante da entrada) é a base das datas quando o acordo não traz outra:

- **Data da entrada:** a do acordo; sem ela, a Data renegociação (só quando há valor de entrada).
- **1º vencimento das parcelas:** com "Dia de vencimento" (ou "Novo dia de vencimento") no acordo, o próximo dia D depois da entrada (ou da renegociação); sem dia D, **1 mês depois** (Mensal) ou **7 dias depois** (Semanal).
- A periodicidade escolhida à mão vale para o cálculo; senão, a da modalidade.
- Como as outras sugestões (`atPlanoAuto`), só preenchem campo vazio ou que ainda tem a sugestão anterior: data digitada à mão não é trocada. Mudar a Data renegociação ou a Periodicidade atualiza as sugestões na hora.

---

## Cópia para planilha

`atLinhas()` — monta as linhas prontas para colar na planilha.

### Negociação: 1 linha, 30 colunas

```
Proposta · Nome · CPF · IMEI · Data da Proposta · Filial · Atendente ·
Número Contato · Quem negociou · Data do Contato · Valor parcela · FPD ·
Parcelas Em Atraso · Dias Atraso · Vencimento Mais Antigo · A partir de qual
parcela pediu acordo · Pedido do cliente · Acordo Realizado · Data do envio da
proposta · Data 1° Pagamento · Vencimento(s) · Parcela(s) negociadas ·
Quantidade de parcelas · Pagamento da entrada · Disponibilizado boleto para o
cliente · Número Contrato Novo · Data Renegociação · Procedimento feito na
régua · Recebeu Boletos? · Negociação Concluída?
```

### Renegociação: 1 linha por parcela, 27 colunas

```
Proposta · Nome · Responsável pela Negociação · Formalizou o acordo? · CPF ·
IMEI · Data da Proposta (Contrato Original) · Número da Parcela Renegociada ·
Data Renegociação · Filial ·
Atendente · Acordo Solicitado · Acordo Realizado · Valor parcela · Valor Pago ·
Número Contato · Data do Contato · Ação · Situação · Baixa Parcela ER ·
Quebra de Acordo · Vencimento · Observação · Número Contrato Novo ·
Procedimento feito na régua · Recebeu Boletos? · Negociação Concluída?
```

"Número da Parcela Renegociada" (campo `parcela_renegociada`, bloco Renegociação) vai igual em todas as linhas do acordo. Digita-se só o número (`3`, `3 4 5`, `8, 9 e 10`) e sai no mesmo formato do texto do acordo (`3° parcela`, `3°, 4° e 5° parcelas`); texto livre fica como digitado. Se ficar vazio, usa a "Parcela(s) negociada(s)" do acordo (`parcela_negociada`).

Formatos: valores `R$ 0,00`, datas `dd/mm/aaaa`, telefone `(11) 912345678`.

---

## Débito atualizado (calculadora de encargos)

Grupo **"Débito atualizado (calculadora)"** (`#mf-at-g-debito`, `data-grupo="at-debito"`), logo abaixo de Cobrança, nas telas Negociação e Renegociação. Calcula quanto o cliente deve hoje (ou numa data escolhida) pelas parcelas em atraso, com a **mesma regra do "Registro de Pagamento de Fatura" do ER**. Só em memória: nada é salvo, nenhuma chave nova de armazenamento, nenhuma requisição ao ER.

**Entradas:** Valor da parcela, Parcelas em atraso, Vencimento mais antigo e Data do pagamento (padrão: hoje). "Trazer da Cobrança" copia os três primeiros da ficha (`valor_parcela`, `parcelas_atraso`, `vencimento_antigo`); ao abrir o grupo e ao calcular, os campos vazios também são preenchidos pela ficha. "Aplicar cliente atual" e "Novo atendimento" limpam o cálculo anterior.

**Regra (por parcela, valores em centavos, `AT_ENCARGOS`):**

| Parte | Fórmula |
|---|---|
| Dias | data do pagamento − vencimento (dias corridos) |
| Encargos de mora | parcela × [(1 + 13,842539%)^(dias/30) − 1] (juros do contrato, compostos pro rata dia) + parcela × 0,99984% ÷ 30 × dias (mora, simples); arredondado ao centavo |
| Multa | parcela × 2% (fixa, fora da base dos juros) |
| Total | parcela + encargos + multa |

- As parcelas são **mensais no mesmo dia** do vencimento mais antigo (dia 31 em mês curto: último dia do mês, `atSomarMeses`). Parcela que vence depois da data do pagamento fica de fora (aviso na mensagem).
- **Taxas calibradas em 23/11/2026** com 3 telas do ER (contrato de R$ 274,92, pagamento em 23/11/2026): 263 dias → encargos R$ 605,84; 141 dias → R$ 243,63; 18 dias → R$ 23,89. As três batem ao centavo. Com a mora em 1% redondo, a de 141 dias sai R$ 0,01 acima.
- **Limites conhecidos:** taxa conferida em um só contrato (outros produtos podem ter outra taxa do contrato); parcela com pagamento parcial (saldo menor que a parcela) não é tratada; vencimentos fora do padrão mensal precisam ser calculados à parte. Por isso o aviso "Valor estimado: confira no ER antes de fechar".

**Saída:** tabela (Vencimento, Dias, Parcela, Encargos, Multa, Total) com a linha de total e o débito em destaque. Botões:

- **Copiar resumo** — texto com uma linha por parcela e os totais (`GM_setClipboard`).
- **Usar como "Valor total com juros"** — grava o total em `valor_total_juros` (ficha em memória). Se o campo está na tela (modalidade Campanha do Fôlego ou mensagem Retorno ao cliente — Campanha do Fôlego), atualiza a caixa e o saldo restante (`ngSaldoHook`).

---

## Aplicar cliente atual

`atAplicarCliente()` — preenche a ficha com os dados da Busca de cliente ativa: Proposta, Nome, CPF, IMEI, Data da proposta, Filial, Atendente, Valor da parcela, FPD, Parcelas em atraso, Dias de atraso, Vencimento mais antigo e Parcela atual (e "A partir da parcela", se vazio). **"A partir da parcela" (`parcela_inicial`) = última parcela vencida do contrato** (`ultimaVencida`: a de maior número com dias de atraso > 0 e saldo em aberto, calculada em `cacCalcular`); sem parcela vencida, a 1ª em aberto (`parcelaAtual`). Corrigido em 20/11/2026 (antes era sempre a 1ª em aberto). Esse valor alimenta a Abrangência da **Mensagem p/ grupo** ("a partir da parcela N") e o campo "A partir de qual parcela" dos Textos da planilha. Pergunta antes de substituir o que já foi digitado. O campo Número de contato **nunca** é preenchido automaticamente.

O botão existe em Negociação, Renegociação e Mensagens (`#mf-at-aplicar` / `#mf-ng-aplicar`) e **fica na tela em que foi clicado**. Em Mensagens, campo com rótulo próprio no modelo (`rotulos`) não é preenchido pela proposta. Desbloqueios tem botão próprio (`#mf-at-desb-aplicar`).

---

## Interface (Nova UI)

Só aparência e retorno visual: os campos, ids, botões, textos gerados e linhas copiadas são os mesmos da UI Antiga.

- **Aviso sempre à vista:** o aviso dos botões fica preso no rodapé da aba (ver `03-painel-interface.md`), com × para esconder até o próximo.
- **Resultado e Modalidade:** um embaixo do outro no painel estreito, para o texto das opções não ser cortado. Lado a lado a partir de 560 px.
- **Textos da planilha:** os botões "Copiar" do Pedido e do Acordo são compactos, à direita, logo abaixo de cada texto.
- **Plano de parcelas:** cada parcela num cartão; a 1ª (entrada) com borda verde.
- **Botões que apagam o que foi digitado** ("Novo atendimento", "Limpar campos", "Novo texto"): texto vermelho. A confirmação continua a mesma.
- **Validação (`atValidar`):** ao copiar ou registrar com campo faltando, o grupo recolhido que tem o campo em vermelho é aberto. O Número de contato inválido também fica em vermelho. A lista de pendências e a regra não mudaram.
- **Grupos:** cada grupo da aba (`at-cliente`, `at-cobranca`, `at-debito`, `at-acordo`, `at-reneg`, `at-textos`, `at-acomp`, `ng-dados`, `ng-saida`) guarda o próprio aberto/fechado.
- **Grades:** `repeat(auto-fit, minmax(130px, 1fr))`. São 2 colunas no painel padrão e até 5 no ampliado, e um grupo com poucos campos ocupa a largura toda. Campo comum até 420 px. Os botões de ação ficam sempre 2 por linha.
- **Rótulo em cima, caixa embaixo** (`justify-content: space-between`): as caixas de uma linha ficam alinhadas mesmo quando um rótulo quebra.
- **Obrigatório:** o ` *` fica num `<span class="mf-obrig">`, com espaço que não quebra, em azul na Nova UI. O texto do rótulo é o mesmo.
- **Campo automático** (`input[readonly]`, ex.: Saldo restante calculado): fundo neutro e borda tracejada. Números e datas usam algarismos de largura fixa.
- **Seções de "Dados" (Mensagens, `.mf-ng-bloco`):** título pequeno em maiúsculas com divisória fina e a mesma seta dos grupos (fechado ▸, aberto ▾). Não viram cartões.
- **Negociação | Renegociação | Mensagens (`.mf-at-seg`):** opção ativa em azul (claro: fundo branco e texto azul; escuro: fundo azul suave), inativas mais apagadas e foco visível.

---

## Mensagens: grupo × cliente

O fluxo tem duas etapas e cada uma fica num lugar só:

1. **Grupo + planilha — Negociação / Renegociação.** O botão **"Mensagem p/ grupo"** gera a mensagem da modalidade atual (`NG_MODALIDADES`, chave `acordo:<modalidade>`; a modalidade "Condições gerais" usa `NG_ACORDO_GERAL`, chave `acordo`). Os mesmos dados geram a linha da planilha e os textos "Pedido do cliente" / "Acordo Realizado".
2. **Cliente — aba Mensagens** ("Mensagens para o cliente"). Só os retornos ao cliente, enviados **depois** de validar no grupo (`NG_MODELOS`): Retorno do setor, Retorno do setor — Instruções de pagamento (PIX/TED), Retorno ao cliente — Campanha do Fôlego, Antecipação de parcela.
- **Retorno do setor — Instruções de pagamento** (`retornoPix`, retorno das renegociações; formato de 20/11/2026): "Quantidade de parcelas", "Valor" e "Data do primeiro vencimento após a entrada", instruções de PIX/TED (Configurações > Gerador de negociações), comprovante, validade em dias úteis e próximo vencimento. "Próximo vencimento" é opcional: vazio, usa a data do primeiro vencimento.
- Cada modelo tem os próprios campos (`campos`, `obrig`); trocar o modelo redesenha o formulário. **Retorno ao cliente — Campanha do Fôlego** (`retornoFolego`, formato de 20/11/2026): confirmação das condições com o cliente — "Senhor Adriano, conforme sua solicitação, gostaríamos de confirmar as condições do acordo referente à 1ª parcela do seu contrato." + valor total atualizado com juros, primeiro pagamento (valor e data) e pagamento do restante (valor e data) + pedido de confirmação. Campos (todos obrigatórios): nome, tratamento (Senhor/Senhora), parcela do contrato, valor total com juros, valor e data do primeiro pagamento, valor restante e data do restante. Calculados: `primeiro_nome` (1ª palavra do nome, só a inicial maiúscula), `parcela_ref` ("à 1ª parcela" / "às 3ª, 4ª e 5ª parcelas"), e o restante = total com juros − primeiro pagamento (`saldo`/`saldoPago` em `ngSaldoHook`). Não leva mais PIX/conta nem prazo em dias úteis. "Aplicar cliente atual" preenche o nome.

- Os modelos Pedido de negociação, Acordo realizado, Campanha do Fôlego e Solicitação de negociação saíram da aba Mensagens (eram cópias do que o Atendimento já gera para o grupo). Textos personalizados deles que existam em `negModelos` ficam guardados, sem uso; nada é apagado.
- Se o modelo salvo em `negModeloAtual` não existe mais, a aba abre em "Retorno do setor".
- **"Aplicar cliente atual"** também existe na aba Mensagens (`#mf-ng-aplicar`, mesma função `atAplicarCliente` do Atendimento): preenche com o cliente da busca do topo (CAC/proposta) e **continua em Mensagens**. Campo que o modelo usa com outro sentido (rótulo próprio em `rotulos`, ex.: "Valor da parcela" do retorno PIX = parcela do acordo) não é preenchido pela proposta.
- Modelos personalizados salvos em `negModelos` (Configurações > Gerador de negociações).

---

## Funções auxiliares

| Função | Descrição |
|---|---|
| `atBloco(sel, campos)` | Renderiza um bloco de campos na interface |
| `atDesenhar()` | Monta toda a interface do Atendimento |
| `atTrocar(ctx)` | Alterna entre Negociação e Renegociação |
| `atSomarMeses(iso, n)` | Soma N meses a uma data ISO (respeitando últimos dias do mês) |
| `atPlanoCampos()` | Retorna os campos do plano de parcelas |
| `atValidar()` | Valida campos obrigatórios antes de registrar |
| `atDebParcela(valor, venc, pgto)` | Encargos, multa e total de uma parcela em atraso (centavos), regra do ER (`AT_ENCARGOS`) |
| `atDebCalcular(valor, qtd, vencIni, pgto)` | Parcelas mensais em atraso a partir do vencimento mais antigo + totais |
| `atDebTrazer(tudo)` / `atDebLimpar()` | Preenche a calculadora com a ficha (Cobrança) / limpa o cálculo |
| `atDebDesenhar(r)` / `atDebResumo(r)` | Tabela do débito / texto do "Copiar resumo" |
| `ngValor(k)` | Lê o valor atual de um campo do formulário |
| `ngMontar(modelo)` | Monta a interface do modelo selecionado |
| `ngDesenharCampos()` | Renderiza os campos da modalidade ativa |
| `ngCampoEl(c, rot, obrig)` | Cria o elemento HTML de um campo |
| `ngBlocosNova(grade, m)` | Organiza campos em blocos na Nova UI |
| `ngMostrarDependentes()` | Mostra/oculta campos dependentes da modalidade |
| `ngSaldoHook()` | Atualiza o saldo (total − pago) na Campanha do Fôlego (`valor_total_juros`) e no retorno do Fôlego (`valor_total_juros` − `valor_entrada`, via `saldoPago`) |
| `ngFaltando()` | Lista campos obrigatórios não preenchidos |
