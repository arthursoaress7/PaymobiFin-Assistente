# Módulo: Consulta CAC

**Linhas:** 4984 – 5329  
**Funções principais:** `cacConsultar()`, `cacAbrir()`, `cacMostrar()`, `cacReq()`; lista PEFIN: `pfAdicionar()`, `pfMontar()`, `pfCadastro()`, `pfCac()`, `pfDivida()`, `pfValor()`, `pfTexto()`

---

## Responsabilidade

Realiza consultas à **Central de Atendimento ao Cliente (CAC)** do sistema ER. É um módulo de **somente leitura** — não faz nenhuma alteração nos dados do sistema.

A consulta retorna:
- Dados contratuais do cliente
- Parcelas (vencimentos, valores, situações)
- Posição Geral (protocolo registrado automaticamente pelo sistema)

---

## Fluxo de consulta

1. Usuário busca um cliente e clica "Usar em: CAC" (ou acessa a aba CAC diretamente)
2. O módulo verifica se a conta atual tem acesso à CAC
3. Se não tiver (`cacSemAcesso`):
   - Se `cacTrocaAuto` está ativo → troca automaticamente para a conta master
   - Senão → exibe aviso
4. Executa a consulta via `cacConsultar(termo, extra)`
5. Exibe o resultado em `cacMostrar(res)`
6. Se `cacVoltarConta` está ativo → volta para a conta anterior após a consulta

---

## Funções

| Função | Descrição |
|---|---|
| `cacReq(caminho, params, metodo)` | Requisição autenticada à API do ER |
| `cacParcelas(doc)` | Extrai as parcelas do documento retornado |
| `cacCalcular(row, posicao, det, parcelas)` | Calcula valores e situações das parcelas: parcela atual (1ª em aberto), **última parcela vencida** (`ultimaVencida`, maior número com dias de atraso > 0 e saldo em aberto; vai para "A partir da parcela" no Atendimento), **período dos vencimentos** (`periodoVenc`, "1ª a última", vai para "Vencimentos do contrato" da Renegociação), parcelas e dias em atraso, vencimento mais antigo, FPD |
| `cacConsultar(termo, extra)` | Executa a consulta pelo CPF ou número de proposta |
| `cacAbrir(row, termo, extra, dentro)` | Abre os detalhes de um contrato específico |
| `cacFim(texto, extra)` | Finaliza a consulta e exibe o resultado |
| `cacVoltarConta(extra)` | Volta para a conta anterior após a consulta |
| `cacSemAcesso(termo, motivo, extra)` | Trata o caso de conta sem acesso à CAC |
| `cacRetomar()` | Retoma a consulta após troca de conta |
| `cacMostrar(res)` | Renderiza o resultado na interface |

---

## Integração com a Busca de cliente

A Busca de cliente (topo do painel) alimenta a CAC:
- "Usar em: CAC" preenche o CPF e inicia a consulta
- Se o resultado já foi obtido antes (mesma sessão), exibe sem consultar novamente

---

## Troca automática de conta

Quando `cacTrocaAuto = true`:
1. O módulo detecta que a conta atual não tem acesso
2. Troca para a conta master silenciosamente (sem confirmação)
3. Executa a consulta
4. Se `cacVoltarConta = true`, retorna para a conta original

A troca usa a mesma infraestrutura do módulo de Contas (seção CCBs/Contas).

**Respostas do CAC sem a tabela** (`CentralPesquisaCliente`), conferidas no sistema real com a conta da Mesa:

| Resposta do sistema | Significado | O que o painel faz |
|---|---|---|
| "Nenhum registro selecionado" | o CPF não tem cadastro | "CAC: nenhum cliente encontrado"; **não** troca de conta |
| "Informe corretamente o CPF / CNPJ ou as iniciais do nome..." | termo inválido | mostra o aviso do sistema; não troca de conta |
| "Usuário sem direito de pesquisa por nome de cliente" | a conta só pesquisa pelo CPF | troca para a master (quando configurada), que pesquisa pelo nome |
| outra coisa / erro HTTP / tela de login / aviso "Sua sessão de trabalho expirou" | sem acesso ou sessão expirada | como antes (troca, ou avisa) |

A sessão expirada é reconhecida por `erSemSessao` (também usada por `ehLogin`, pelo Relatório de Pendências e pela busca de cliente): tela de login ou a página `/AvisoSessao/AvisoSessao`, para onde o ER redireciona toda requisição depois que a sessão vence.

Pelo CPF, a conta da Mesa faz a consulta inteira: pesquisa, `BuscaCartao`, `CentralCompras` e `DetalheCompra`. Quando o script trocou de conta para a consulta, o "nenhum cliente encontrado" também volta para a conta de antes (`cacFim`). Antes, ficava na master.

---

## Posição Geral

Quando `cacPosicaoGeral = true` (padrão), o módulo lê a Posição Geral do cliente (`POST /CentralAtendimento/CentralInicio`), que traz a Filial (Origem), a data de adesão e a lista de contratos. Desligado, lê os contratos por `GET /CentralAtendimento/CentralCompras` e a Filial pode ficar N/A.

## Protocolo de atendimento

Analisado em 11 e 12/11/2026 com gravações (HAR) da tela do CAC:

- **O protocolo nasce no `BuscaCartao`**, não na Posição Geral. A resposta de `GET /CentralAtendimento/BuscaCartao` já traz `lngProtocoloAtendimento`; a tela mostra esse número no botão azul "Protocolo: N" (rodapé da janela do cliente) e o repassa ao `CentralInicio`.
- **Cada `BuscaCartao` gera um número novo**, inclusive ao abrir o mesmo cliente duas vezes seguidas. A numeração é única para todo o ER (colegas usando o CAC também consomem números).
- **A tela do ER encerra o protocolo** ao fechar a janela do cliente (`GET /CentralAtendimento/CentralFinalizaProtocolo?strProtocolo=N`).
- **O painel chama o `BuscaCartao` em toda consulta** (com a Posição Geral ligada ou desligada), com a mesma requisição da tela, e **nunca encerra o protocolo**. Conclusão (por equivalência das requisições, ainda não testada isoladamente pelo painel): cada consulta CAC pelo painel deixa um protocolo de atendimento aberto no ER.
- O botão azul "Protocolo" abre o formulário de envio do protocolo ao cliente (`CentralEnvioProtocolo`); o envio em si é `EfetivaEnvioProtocolo` (SMS ou e-mail). O painel não usa nenhum dos dois.

Tratamento: em decisão (ver `PRIORIDADES.md`). Encerrar o protocolo pelo painel altera dados no ER e exige autorização explícita.

**Teste de confirmação pelo painel:** no ER, abrir um cliente no CAC e anotar N; fechar. Em seguida fazer uma consulta CAC pelo painel. Em seguida abrir um cliente no ER: N + 2 confirma. Fazer tudo em menos de um minuto para evitar números consumidos por colegas.

---

## Negativação PEFIN (`pf`, aba CAC)

Seção **Negativação PEFIN** abaixo do resultado do CAC. Monta a lista de clientes com os campos do formulário do gerador do arquivo PEFIN (Serasa, layout de 600 posições), para copiar em vez de digitar. **Somente leitura.** A lista fica só em memória (`pfLista`): some ao recarregar a página e nada vai para o `GM_setValue`, logs, diagnósticos ou Pusher.

**Como entra um cliente:**
- **Adicionar** pelo CPF/CNPJ ou pelo nome. Pelo nome, se a Mesa devolver mais de um cliente, o painel mostra a escolha.
- **Adicionar CPFs colados** (um por linha): processa um por vez, com pausa de `CONFIG.PF_PAUSA_MS` entre eles, e tem o botão **Parar**. Linhas sem CPF/CNPJ válido são ignoradas e contadas no aviso do fim.
- O mesmo CPF adicionado de novo **atualiza** o cartão (não duplica).

**De onde vem cada campo (`pfMontar`):**

| Campo | Fonte |
|---|---|
| Proposta usada | Mesa de Crédito (`bsMesa` → `MesaCreditoLista`): a **efetivada** mais recente; sem efetivada, a mais recente (`pfEscolherProposta`) |
| CPF/CNPJ, nome, nascimento | Aba Cliente da proposta (`MesaAnaliseCliente`, `pfCadastro`) |
| CEP, logradouro, número, complemento, bairro, cidade, UF | Aba Cliente (`pfCadastro`). Rótulos aceitos: `CEP`; `Logradouro`/`Endereço`/`Rua`; `Número`/`Nº`; `Complemento`; `Bairro`; `Cidade`/`Município`; `UF`/`Estado`; também `Cidade/UF` num campo só ("CAMPINAS - SP") e endereço com número ("RUA X, 45 - APTO 12") |
| Natureza da dívida | Campo da seção, padrão `CB - CDC BENS` (vale para a lista inteira; não é gravado) |
| Data de vencimento | CAC: vencimento da parcela vencida **mais antiga** com saldo em aberto (dias de atraso > 0) |
| Data de término do contrato | CAC: vencimento da **última** parcela do contrato |
| Valor total da dívida | CAC: soma das parcelas vencidas e não pagas pelo **valor original da parcela**, sem juros, encargos nem multa (decisão de 25/11/2026). Parcela paga em parte entra pelo valor original |

**CAC (`pfCac`):** as mesmas requisições do `cacAbrir`, sem a Posição Geral: `CentralPesquisaCliente` pelo CPF, `BuscaCartao`, `CentralCompras` e `DetalheCompra` (até 3 produtos e 5 contratos; só contratos do mesmo CPF). Com vários contratos, usa o mais atrasado e avisa os outros. **Cada cliente abre um protocolo de atendimento no ER** (`BuscaCartao`, ver P4.12). Se o CAC pedir a confirmação de identidade, o cliente fica sem os dados da dívida.

**Avisos no cartão:** CAC sem dados; nenhuma parcela vencida em aberto ("confira antes de negativar"); outros contratos; campos que não foram achados na aba Cliente, com a lista dos **nomes** dos rótulos lidos (sem valores), para ajustar os rótulos se a tela do ER for diferente.

**Cópia:** **Copiar** (um cliente) e **Copiar todos** (os que aparecem com a busca da lista), em **Texto** (`Rótulo: valor`, um campo por linha) ou **CSV** com `;` e cabeçalho. A busca da lista filtra por CPF (dígitos) ou nome (sem acento).

| Campo | Leitura (tela e cópia padrão) | Formato do arquivo (opção marcada) |
|---|---|---|
| CPF/CNPJ | `123.456.789-09` / `12.345.678/0001-95` | 15 dígitos com zeros à esquerda |
| Datas | `dd/mm/aaaa` | `AAAAMMDD` |
| Valor | `R$ 1.234,56` | centavos, 15 dígitos com zeros à esquerda (`000000000123456`) |
| CEP | `01310-100` | 8 dígitos |
| Natureza | `CB - CDC BENS` | só o código (`CB`) |
| Textos | como no ER | maiúsculas sem acento |

**A confirmar no uso real:** os rótulos do endereço na aba Cliente (o script não lia o endereço antes desta função) e se o formato do arquivo acima é o que o gerador da equipe espera.

---

## O que foi removido na v12.0

- Preenchimento manual dos campos de proposta/IMEI/atendente/contato/quem negociou/data
- Botão "Copiar linha (Proposta → Vencimento Mais Antigo)"
  - Essa linha agora sai do módulo de Atendimento

O que permanece:
- A consulta em si
- Troca para conta master
- Posição Geral
- Resultado completo de parcelas
- Integração com a Busca de cliente
