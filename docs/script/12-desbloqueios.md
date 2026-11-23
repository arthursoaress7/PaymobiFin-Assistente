# Módulo: Desbloqueios

**Localização:** dentro do Atendimento, ao lado de Negociação, Renegociação e Mensagens  
**Arquivo:** `paymobifin-assistente.user.js` (até a v13.28.0 o código estava, por engano, só no arquivo-ponte `automacaomobfacil.user.js`; foi levado para o arquivo principal na v13.29.0)  
**Contexto:** `ngCtx === 'desbloqueios'`

---

## Responsabilidade

Documentar desbloqueios de clientes. O usuário busca um cliente pelo CPF ou nome no campo de busca do topo, clica em "Aplicar cliente atual" e os campos automáticos são preenchidos. Os campos específicos do desbloqueio são preenchidos manualmente. Ao final, clica em "Copiar linha da planilha" e cola na aba de Desbloqueios da planilha.

---

## Como acessar

No painel, aba **Atendimento**, botão **Desbloqueios** no controle segmentado (ao lado de Negociação, Renegociação e Mensagens).

---

## Campos da planilha

1 linha, 15 colunas:

```
Proposta · Nome · CPF · Tipo Desbloqueio · IMEI · Data da proposta ·
Data do desbloqueio · Loja · Vendedor · Motivo · Adicionado à régua ·
Cumpriu Acordo? · Tipo de atendimento · Parcelas em atraso · Observação
```

### Preenchimento automático (via "Aplicar cliente atual")

| Campo | Origem |
|---|---|
| Proposta | Busca de cliente |
| Nome | Busca de cliente |
| CPF | Busca de cliente |
| IMEI | Busca de cliente |
| Data da proposta | Busca de cliente |
| Loja | Busca de cliente (campo Filial) |
| Vendedor | Busca de cliente (campo Atendente) |
| Tipo de atendimento | Padrão: RD |

### Preenchimento manual

| Campo | Tipo |
|---|---|
| Tipo de desbloqueio | Seleção: Desbloqueio por Pagamento / Desbloqueio por confiança |
| Data do desbloqueio | Data |
| Motivo | Texto livre |
| Adicionado à régua | Seleção: SIM / NÃO |
| Cumpriu acordo? | Seleção: SIM / NÃO |
| Tipo de atendimento | Seleção: RD / Paymobi / Recupere (padrão: RD) |
| Parcelas em atraso | Número |
| Observação | Seleção (preenchida pelo tipo de desbloqueio; pode ser trocada à parte) |

---

## Funções

| Função | Descrição |
|---|---|
| `atDesbBloco(campos)` | Renderiza os campos do desbloqueio no container `#mf-at-desb-campos` e liga o tipo de desbloqueio à observação (`AT_DESB_OBS`) |
| `atDesbAplicarCliente()` | Preenche os campos do cliente a partir da busca ativa (sem preencher campos de negociação) |
| `atDesenhar()` (modificada) | Quando `ngCtx === 'desbloqueios'`, esconde `#mf-at-form` e `#mf-at-mensagens`, mostra `#mf-at-desbloqueio` e renderiza os campos |
| `atValidar()` (modificada) | Para desbloqueios, valida apenas Proposta e Nome como obrigatórios |
| `atLinhas()` (modificada) | Para desbloqueios, retorna 1 linha com `AT_COL_DESBLOQUEIO` |

---

## Tipo de desbloqueio e Observação (opções fixas)

Os dois campos são seleções. Cada tipo de desbloqueio tem uma observação vinculada (constante `AT_DESB_OBS`):

| Tipo de desbloqueio | Observação preenchida |
|---|---|
| Desbloqueio por Pagamento | Cliente realizou o pagamento, compartilhou o comprovante |
| Desbloqueio por confiança | Cliente precisava do desbloqueio para acessar comprovante |

- Escolher o tipo de desbloqueio preenche a observação vinculada (`atDesbBloco` liga o evento).
- A observação pode ser trocada depois, sem alterar o tipo.
- Voltar o tipo para "(em branco)" não apaga a observação.
- O Motivo continua texto livre.
- A planilha recebe o texto da opção, como antes recebia o texto digitado.

---

## Integração com o Atendimento

A funcionalidade de Desbloqueios compartilha a mesma ficha em memória (`ngValores`) usada por Negociação e Renegociação. Isso significa que:

- Os campos de cliente (proposta, nome, cpf, imei, data_proposta, filial, atendente) preenchidos em um contexto aparecem nos outros
- Os campos específicos de desbloqueio (`tipo_desbloqueio`, `data_desbloqueio`, etc.) não interferem com os campos de negociação
- O botão "Novo desbloqueio" limpa apenas os campos do desbloqueio, preservando dados de negociação se houver

---

## Chaves de armazenamento

Nenhuma chave nova. O contexto ativo é salvo em `atModelo` (existente), que agora aceita `'desbloqueios'` como valor.

---

## Campos do script (NG_CAMPOS)

| Campo | Rótulo | Tipo |
|---|---|---|
| `tipo_desbloqueio` | Tipo de desbloqueio | select (em branco / Desbloqueio por Pagamento / Desbloqueio por confiança) |
| `data_desbloqueio` | Data do desbloqueio | data |
| `motivo_desbloqueio` | Motivo | area |
| `adicionado_regua` | Adicionado à régua | select (SIM/NÃO) |
| `cumpriu_acordo` | Cumpriu acordo? | select (SIM/NÃO) |
| `tipo_atendimento_desb` | Tipo de atendimento | select (RD/Paymobi/Recupere) |
| `desb_parcelas_atraso` | Parcelas em atraso | num |
| `obs_desbloqueio` | Observação | select (em branco / as duas observações de `AT_DESB_OBS`) |
