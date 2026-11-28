# 14 — Analisar proposta (`an`)

Aba **Analisar proposta** (menu **Mais ▾**). Digita-se o número da proposta e o painel confere, de uma vez, o que costuma divergir ou ficar fora da política. **Somente leitura:** nada é alterado no ER, no MobFacil ou na BMP.

## Regra principal

Cada conferência sai com um de quatro resultados, sempre com o motivo:

| Ícone | Significado |
|---|---|
| ✅ | ok: o dado foi lido e está dentro da regra |
| ❌ | problema: fora da política ou com divergência |
| ⚠️ | alerta: o MobFacil só alerta (contato, renda não validada, mensagens de derivação) |
| ❔ | não confirmado: o dado não foi lido (MobFacil fora, fluxo P2 ainda não rodou, ícone da BMP que pode ter sumido) |

O veredito só é **"Tudo certo"** quando **todas** as conferências são ✅. O que não deu para confirmar **nunca vira ok**: aparece como ❔ e o veredito pede para conferir à mão.

## Fontes

1. **Página da proposta** (`/MesaCredito/MesaAnaliseProposta`, `anLerPagina`): situação, data da proposta, digitador e valor da compra (`acharRotulo`). Digitador vazio (proposta da WEBAPI) sai N/A.
2. **Aba Cliente** (`/MesaCredito/MesaAnaliseCliente`, `dadosCliente`): nome, CPF e data de nascimento como estão agora no ER.
3. **Itens da análise** (`/FilaPropostas/DetalhePropostaAnalise`, `itensAnalise`): biometria, idade, MobFacil, registros internos etc. APROVADO = ✅, REPROVADO = ❌, outro = ⚠️.
4. **MobFacil** (`anLerMotor` no painel + `anMotorResumo`/`anResponder` no iframe do motor): o painel abre a página do motor da proposta num iframe escondido e grava `anPedido` `{ id, ts, nonce }`. No iframe, o script lê o resultado completo da análise **na memória do React** (o JSON que a página buscou em `/api/visualizador`: `Resumo proposta` e `Exibir fluxos`), sem clicar na árvore, e devolve em `anMotor` um resumo: status, mensagens, as regras de cada fluxo e os dados enviados, **sem** fotos/documentos e sem os dados dos birôs. Espera até `CONFIG.AN_ESPERA` s. Os dois valores são apagados no fim. Não usa nem muda a fila dos relatórios (`rodando`, `atualId`, `dados`).
5. **Impeditivo da BMP** (`anKyc`): a lista guardada do relatório Impeditivos BMP (`ibImpeditivos`) e o ícone 🚫 na Fila de Propostas do dia da proposta (`ibLista`).

## Conferências

| Conferência | Fonte (MobFacil) | ❌ / ⚠️ quando |
|---|---|---|
| Política de faixa (score x restrições) | P1 `classificacaoRiscoDeCredito.valorScore`, `calculaRisco` | ❌ valor ou quantidade acima do limite da faixa (`CONFIG.FAIXAS`), ou faixa F |
| **Decisão do MobFacil (crédito) x política da equipe** (`anDecisaoMotor`) | P1 `calculaRisco.Status` (restrições) e `classificacaoRiscoDeCredito.Status` (ponto de corte) | ❌ o MobFacil **reprovou** pelo crédito com os dados **dentro** da política, ou **aprovou** com eles **fora**; ⚠️ mandou para a mesa com os dados dentro. Mostra o limite que o MobFacil usou (da mensagem "faixa X acima do limite (Restritivo=R$…, Qtde=…)") |
| Nome x Receita Federal | `validacaoCadastral.ValidacaoNome` (Receita = `DadoBiro`) x **nome atual do ER** (aba Cliente) | ❌ nome diferente; ⚠️ nome cortado no fim (o campo tem limite de caracteres). Igual sem acento/espaço repetido = ✅ |
| Data de nascimento x Receita Federal | `validacaoCadastral.ValidacaoDataDeNascimento` (`DadoBiro`) x **nascimento atual do ER** | ❌ data diferente |
| CPF analisado pelo MobFacil | `PROP_CPF` enviado ao motor x CPF da aba Cliente | ❌ o MobFacil analisou outro CPF |
| CPF regular na Receita Federal | P1 `requisitosBiroValores.SituacaoCpf` | ❌ |
| Sem marcação de óbito | P1 `requisitosBiroValores.MarcacaoObito` | ❌ |
| Idade (18 a 75 anos) | P1 `requisitosBasicosValores` | ❌ |
| Renda mínima | P1 `requisitosBasicosValores.RendaMinima` | ❌ |
| Renda validada no birô | P1 `validaRendavalores.RendaValidada` | ⚠️ não validada |
| Parcela x máximo mensal | P1 `limiteCreditoValores` (`parceladoMensalCalculado` x `parceladoMensal`) | ⚠️ parcela acima do máximo. O MobFacil só manda a parcela calculada quando ela passa do máximo: sem ela e sem alerta de parcela = ✅ |
| Aparelho disponível para crédito | `validaAparelho` | ❌ reprovado; ⚠️ pendente mesa (ex.: IMEI/modelo não cadastrado) |
| Tipos de documento | P2 `validaTipoDoc` | ❌ ex.: "Documentação Divergente - Documentos Iguais"; ⚠️ "para análise da mesa" (ex.: segundo documento NA) |
| Documento x proposta (CPF e nome no documento) | P2 `validaDocumentos.ValidaDoc` | ❌ CPF ou nome não encontrado em **nenhum** documento, ou documento reprovado. Frente e verso dividem os dados (RG/CNH frente com o nome, verso com o CPF): vale o conjunto |
| Foto x documento (facematch) | P2 `validaFacematch` | ❌ |
| Biometria | P2 `validaBiometria` | ❌ |
| Telefone 1 / E-mail / Endereço nos birôs | P2 `validaTelefones`, `validaEmails`, `validaEnderecos` | ⚠️ (o MobFacil só alerta) |
| Resultado do MobFacil | status final e mensagens | ❌ reprovado; ⚠️ pendente (mesa/loja). Aprovado = ✅ (os alertas já estão nas conferências acima) |
| Análise ER: (cada item) | itens da análise da proposta | ❌ reprovado; ⚠️ pendente |
| Impeditivo da BMP (KYC) | lista guardada + Fila de Propostas | ❌ com o ícone 🚫 |

**Dados atuais do ER:** a loja pode corrigir o nome ou o nascimento depois da análise, sem o MobFacil rodar de novo (ex.: 5585, GIOVANE corrigido para GIOVANI; 5591, 28/05 corrigido para 28/06). Por isso o lado "proposta" do nome e do nascimento é o que está **agora** na aba Cliente do ER (`dadosCliente`, que passou a ler também a "Data de Nascimento"); a Receita continua sendo o `DadoBiro` do MobFacil. Quando o valor atual difere do que o MobFacil recebeu, o detalhe diz "corrigido no ER depois da análise do MobFacil (o MobFacil recebeu …)". Sem a aba Cliente, vale o que o MobFacil recebeu. O cabeçalho também mostra o nome e o CPF atuais do ER.

**Fluxo P2** (documentos e contato) só existe depois do cadastro completo. Sem ele, sai uma linha só, ❔ "Documentos e contato (fluxo P2)", dizendo se a proposta foi reprovada antes ou ainda não chegou nessa etapa.

**Reanálise:** quando a proposta foi reanalisada, o motor só tem `fluxos_reanalise`, uma lista de rodadas em que cada uma traz todas as regras (crédito, documentos e contato). Vale a rodada mais recente (`Timestamp`), no lugar do P1 e do P2. Ex.: 5555. Nos fluxos normais também vale a rodada mais recente.

**Impeditivo da BMP:** sem o ícone na Fila só vale como ✅ quando a proposta está efetivada ou cancelada e tem até `CONFIG.AN_KYC_DIAS` dias, porque o KYC da BMP roda na efetivação e o ícone some da Fila com o tempo. Fora disso, ❔.

## Saída

Cabeçalho (proposta, cliente, CPF, data, situação no ER, digitador, valor da compra), o veredito e a lista ordenada por gravidade (❌, ⚠️, ❔, ✅). **Copiar resumo** copia o mesmo conteúdo em texto.

## Análise em massa

Seção **Analisar várias propostas**, na mesma aba. Os números podem ser digitados (um por linha, ou separados por vírgula/espaço) ou buscados com **Buscar na Fila** (período De/Até pela data de inclusão, e o tipo: reprovadas, efetivadas, aprovadas não efetivadas, ou as três). Com mais de 500 propostas no período, a Fila é lida dia a dia. **Analisar todas** roda a mesma análise de uma proposta (`anAnalisarUm`), uma por vez; **Parar** para depois da atual. A Fila de cada dia (KYC) é lida uma vez só na massa. A sessão do ER expirada para a massa e diz quantas faltaram.

**Conferência de cada proposta** (`anConferencia`), pelo tipo da situação (`anTipo`):

| Tipo | Resultado |
|---|---|
| Reprovada | ✅ "Reprovação confere" quando há problema de política ou divergência (fora o próprio "reprovado" do MobFacil). ❌ "o MobFacil reprovou errado" quando a reprovação foi pelo crédito e os dados estão dentro da política da equipe (ex.: 5545, 5557, 5569 e 5580, reprovadas com o limite antigo do MobFacil). ⚠️ quando nenhum motivo foi encontrado |
| Efetivada / aprovada | ❌ com qualquer problema (fora da política, divergência, ou o MobFacil aprovou fora da política); ⚠️ só alertas; ❔ conferências não confirmadas; ✅ tudo certo |

A lista fica ordenada por gravidade, e cada proposta abre os detalhes ao clicar. Cópias:
- **Copiar resumo:** texto com a conferência de cada proposta e os ❌/⚠️.
- **Copiar planilha:** colunas (Proposta, Cliente, CPF, Data, Situação, Score, Faixa, Valor restrições, Qtd restrições, Conferência, Problemas, Alertas, Não confirmados), para colar na célula A1.
- **Copiar conferência (formato da equipe):** `5545 - Score 602, Faixa C, R$2238,64 (Intervalo: 552 a 674)`, com o título "CONFERÊNCIA DE REPROVAÇÕES:" quando todas são reprovadas.

Os resultados ficam só na memória da aba (somem ao recarregar).

**Calibração (25/11/2026):** a análise foi rodada em 26 propostas reais de 24 e 25/11 (10 reprovadas, 8 efetivadas, 7 aprovadas, 1 reprovada manual). Daí saíram os ajustes de nome (espaço/acento, nome cortado), documentos frente/verso, aparelho e tipos "para a mesa" como alerta, parcela e reanálise.
## Base de aparelhos da equipe

`CONFIG.APARELHOS`: a planilha de aparelhos da equipe (28/11/2026), uma linha por TAC: [TAC = "IMEI (RAIZ)", descrição técnica, nome comercial, valor base, valor teto, valor médio, limite máx. de financiamento]. O mesmo TAC pode ter mais de um modelo (ex.: 86579607 = C75X e 15X). A conferência "IMEI na base de aparelhos da equipe" (`anBaseAparelho`) usa o IMEI enviado ao MobFacil (`PROP_IMEI`): na base = ✅ com o modelo e o limite; fora = ❌ "conferir no site da Paymobi". O limite máximo de financiamento só é mostrado (com mais de um modelo no TAC, o maior): a regra de comparação com o valor da proposta ainda não foi definida. Quando a planilha mudar, atualizar `CONFIG.APARELHOS`.

## Aprovadas manualmente: motivo da derivação e motivo da aprovação

Toda análise (uma ou em massa) lê também o **histórico** da proposta (`anHistorico`: inclusão, quem aprovou e efetivação) e o **ponto de atendimento** da formalização (`anPontoAtendimento`, no formato da Fila, sem o código). Aprovação **manual** = há "APROVADA" no histórico (o analista, ex.: "ROBERTOTAS I ROBERTO MASTER" vira "ROBERTO MASTER"); só "APROVADOR" = automática. O cabeçalho mostra Ponto de atendimento, Inclusão, Aprovação, Motivo da derivação e, nas manuais, Motivo da aprovação.

- **Motivo da derivação** (`anDerivacoes`): as mensagens "Derivar mesa/loja" e "Reprovar" do MobFacil (P1, P2 e reanálise, sem repetir), em texto de leitura: documentação ("Documentos Iguais", "para Análise Mesa de Crédito"), nome/CPF não conferidos no documento, nome ou nascimento divergente da Receita, crédito acima do limite do MobFacil, IMEI/modelo não cadastrado.
- **Motivo da aprovação** (`anMotivoAprovacao`), para cada motivo da derivação, o que a análise confirma:
  - documento: o **CPF** foi encontrado no documento e o facematch e a biometria estão ok = "Documento conferido" (o nome que o MobFacil não leu no documento não impede, ex.: 5491); sem o CPF = ⚠️;
  - nome / nascimento: o dado atual do ER bate com a Receita ("corrigido no ER"); nome cortado no fim ou **incompleto** (todas as palavras da proposta estão no nome da Receita, ex.: 5508 "LOUISIANA CASTILHO" x "... REGO") é explicado; diferente de verdade = ⚠️;
  - crédito: dentro da política da equipe (`CONFIG.FAIXAS`), mesmo acima do limite do MobFacil (ex.: 5536, que foi só por passagens); fora = ⚠️ "Crédito FORA da política";
  - IMEI/modelo não cadastrado no MobFacil: confere o TAC (8 primeiros dígitos do IMEI) na **base de aparelhos da equipe** (`CONFIG.APARELHOS`, planilha de 28/11/2026, 65 modelos): na base = "IMEI na base da equipe (modelo, limite)"; fora = ⚠️ "conferir no site da Paymobi";
  - sem derivação: "aprovada na mesa sem pendência apontada pelo motor".
- **Buscar na Fila > "Aprovadas manualmente (efetivadas e não efetivadas)"**: pega as efetivadas e aprovadas do período e confere no histórico de cada uma (6 por vez) quais foram aprovadas por um analista.
- **Copiar Excel (aprovadas manualmente)**: só as manuais, sem cabeçalho, colunas A a I: CPF (000.000.000-00), Nome Cliente, N° Proposta, Ponto Atendimento, Vendedor (digitador), DataInclusao (dd/mm/aaaa hh:mm:ss), Analista, Motivo Derivação, Motivo Aprovação.

Testado em 25-26/11/2026 com 10 aprovadas na mesa (5487, 5488, 5491, 5508, 5511, 5512, 5515, 5536, 5537, 5597).
## Armazenamento

| Chave | Conteúdo |
|---|---|
| `anPedido` | `{ id, ts, nonce }`: pedido do painel ao iframe do motor (apagado no fim) |
| `anMotor` | resumo do resultado do motor devolvido pelo iframe (apagado assim que o painel lê) |
