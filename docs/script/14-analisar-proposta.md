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
2. **Itens da análise** (`/FilaPropostas/DetalhePropostaAnalise`, `itensAnalise`): biometria, idade, MobFacil, registros internos etc. APROVADO = ✅, REPROVADO = ❌, outro = ⚠️.
3. **MobFacil** (`anLerMotor` no painel + `anMotorResumo`/`anResponder` no iframe do motor): o painel abre a página do motor da proposta num iframe escondido e grava `anPedido` `{ id, ts, nonce }`. No iframe, o script lê o resultado completo da análise **na memória do React** (o JSON que a página buscou em `/api/visualizador`: `Resumo proposta` e `Exibir fluxos`), sem clicar na árvore, e devolve em `anMotor` um resumo: status, mensagens, as regras de cada fluxo e os dados enviados, **sem** fotos/documentos e sem os dados dos birôs. Espera até `CONFIG.AN_ESPERA` s. Os dois valores são apagados no fim. Não usa nem muda a fila dos relatórios (`rodando`, `atualId`, `dados`).
4. **Impeditivo da BMP** (`anKyc`): a lista guardada do relatório Impeditivos BMP (`ibImpeditivos`) e o ícone 🚫 na Fila de Propostas do dia da proposta (`ibLista`).

## Conferências

| Conferência | Fonte (MobFacil) | ❌ / ⚠️ quando |
|---|---|---|
| Política de faixa (score x restrições) | P1 `classificacaoRiscoDeCredito.valorScore`, `calculaRisco` | ❌ valor ou quantidade acima do limite da faixa (`CONFIG.FAIXAS`), ou faixa F |
| Nome x Receita Federal | `validacaoCadastral.ValidacaoNome` | ❌ status diferente de "A" (mostra a similaridade e os dois nomes) |
| Data de nascimento x Receita Federal | `validacaoCadastral.ValidacaoDataDeNascimento` | ❌ status diferente de "A" |
| CPF regular na Receita Federal | P1 `requisitosBiroValores.SituacaoCpf` | ❌ |
| Sem marcação de óbito | P1 `requisitosBiroValores.MarcacaoObito` | ❌ |
| Idade (18 a 75 anos) | P1 `requisitosBasicosValores` | ❌ |
| Renda mínima | P1 `requisitosBasicosValores.RendaMinima` | ❌ |
| Renda validada no birô | P1 `validaRendavalores.RendaValidada` | ⚠️ não validada |
| Aparelho disponível para crédito | `validaAparelho` | ❌ |
| Tipos de documento | P2 `validaTipoDoc` | ❌ ex.: "Documentação Divergente - Documentos Iguais" |
| Documento x proposta (CPF e nome no documento) | P2 `validaDocumentos.ValidaDoc` | ❌ CPF ou nome não encontrado no documento |
| Foto x documento (facematch) | P2 `validaFacematch` | ❌ |
| Biometria | P2 `validaBiometria` | ❌ |
| Telefone 1 / E-mail / Endereço nos birôs | P2 `validaTelefones`, `validaEmails`, `validaEnderecos` | ⚠️ (o MobFacil só alerta) |
| Resultado do MobFacil | status final e mensagens (reprovar, derivar, alertas como parcela acima do máximo) | ❌ reprovado; ⚠️ pendente ou com mensagens |
| Análise ER: (cada item) | itens da análise da proposta | ❌ reprovado; ⚠️ pendente |
| Impeditivo da BMP (KYC) | lista guardada + Fila de Propostas | ❌ com o ícone 🚫 |

**Fluxo P2** (documentos e contato) só existe depois do cadastro completo. Sem ele, essas conferências saem ❔.

**Impeditivo da BMP:** sem o ícone na Fila só vale como ✅ quando a proposta está efetivada ou cancelada e tem até `CONFIG.AN_KYC_DIAS` dias, porque o KYC da BMP roda na efetivação e o ícone some da Fila com o tempo. Fora disso, ❔.

## Saída

Cabeçalho (proposta, cliente, CPF, data, situação no ER, digitador, valor da compra), o veredito e a lista ordenada por gravidade (❌, ⚠️, ❔, ✅). **Copiar resumo** copia o mesmo conteúdo em texto.

## Armazenamento

| Chave | Conteúdo |
|---|---|
| `anPedido` | `{ id, ts, nonce }`: pedido do painel ao iframe do motor (apagado no fim) |
| `anMotor` | resumo do resultado do motor devolvido pelo iframe (apagado assim que o painel lê) |
