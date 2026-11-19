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
| **Acordo** | Resultado\* (fechou / solicitou / não concluída) + Modalidade\* + campos da modalidade |
| **Textos** | Pedido do cliente, Acordo Realizado, A partir de qual parcela |
| **Acompanhamento** | 12 colunas opcionais (botão "Trazer do acordo" preenche data e quantidade) |

### Modelo 2 — Renegociação

Aproveita o bloco Cliente do Modelo 1 e acrescenta:

| Bloco | Campos |
|---|---|
| **Acordo** | Modalidade\* (resultado fixo = fechou) |
| **Textos** | Acordo Solicitado, Acordo Realizado |
| **Renegociação** | Responsável\*, Formalizou (SIM), Data renegociação, Contrato novo, Observação, Régua, Recebeu boletos, Concluída |
| **Plano de parcelas** | Tabela editável gerada automaticamente a partir do acordo |

> **Número de contato:** é o número usado pelo cliente no RD Station. O telefone do cadastro da proposta nunca preenche esse campo automaticamente.

---

## Modalidades de acordo

Definidas em `NG_MODALIDADES`. Cada modalidade tem campos próprios, obrigatórios, texto de acordo e frases.

| Chave | Modalidade |
|---|---|
| `mensal` | Novo valor mensal |
| `parcelamento` | Parcelamento |
| `entrada` | Entrada + parcelas |
| `parcial` | Pagamento parcial + restante |
| `folego` | Campanha do Fôlego |
| `pix` | PIX semanal / Automático |
| `vencimento` | Alteração da data de vencimento |
| `antecipacao` | Antecipação / quitação |
| `regularizacao` | Regularização de parcela(s) em atraso |
| `semacordo` | Sem acordo / pagamento normal |

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
| `solicitou` | "Cliente solicitou…" (simulação) |
| `não concluída` | Apenas o pedido, sem frase de acordo |

A frase "Acordo Realizado" é idêntica à gerada na v11.38 (regressão validada com 14 casos).

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
| Entrada + parcelas | Entrada (data da entrada) + N parcelas mensais |
| Novo valor mensal / Parcelamento | N parcelas mensais a partir do 1º vencimento |
| PIX semanal / Automático | N pagamentos semanais (+ entrada se informada) |
| Pagamento parcial / Fôlego | 2 linhas: valor agora + saldo restante |
| Antecipação / quitação | 1 linha com o valor |
| Regularização | N × valor ou 1 linha |
| Alteração de vencimento | N parcelas afetadas × novo valor |

> **Dia 31 em mês curto:** cai no último dia do mês (ex.: 28/02).

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

## Aplicar cliente atual

`atAplicarCliente()` — preenche a ficha com os dados da Busca de cliente ativa: Proposta, Nome, CPF, IMEI, Data da proposta, Filial, Atendente, Valor da parcela, FPD, Parcelas em atraso, Dias de atraso, Vencimento mais antigo e Parcela atual (e "A partir da parcela", se vazio). Pergunta antes de substituir o que já foi digitado. O campo Número de contato **nunca** é preenchido automaticamente.

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
- **Grupos:** cada grupo da aba (`at-cliente`, `at-cobranca`, `at-acordo`, `at-reneg`, `at-textos`, `at-acomp`, `ng-dados`, `ng-saida`) guarda o próprio aberto/fechado.
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
- Cada modelo tem os próprios campos (`campos`, `obrig`); trocar o modelo redesenha o formulário. **Retorno ao cliente — Campanha do Fôlego** (`retornoFolego`): nome, CPF, entrada, valor das parcelas, vencimento e validade do acordo, todos obrigatórios; chave PIX obrigatória e conta para TED opcional (`incluir_ted`), ambas vindas de Configurações > Gerador de negociações. Nome e CPF são preenchidos por "Aplicar cliente atual".

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
| `ngValor(k)` | Lê o valor atual de um campo do formulário |
| `ngMontar(modelo)` | Monta a interface do modelo selecionado |
| `ngDesenharCampos()` | Renderiza os campos da modalidade ativa |
| `ngCampoEl(c, rot, obrig)` | Cria o elemento HTML de um campo |
| `ngBlocosNova(grade, m)` | Organiza campos em blocos na Nova UI |
| `ngMostrarDependentes()` | Mostra/oculta campos dependentes da modalidade |
| `ngSaldoHook()` | Atualiza o saldo (total − pago) no pagamento parcial (`valor_total_devido`) e na Campanha do Fôlego (`valor_total_juros`) |
| `ngFaltando()` | Lista campos obrigatórios não preenchidos |
