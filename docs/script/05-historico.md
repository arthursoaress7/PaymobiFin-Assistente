# Módulo: Histórico de Acordos

**Linhas:** 6822 – 7050  
**Chave de armazenamento:** `mfAcordos` (versão 1)  
**Retenção:** 30 dias após a última alteração

---

## Responsabilidade

Armazena e exibe os acordos registrados pela equipe. Funciona como um log local (por navegador) que pode ser sincronizado via Pusher quando o compartilhamento de equipe está ativo.

---

## Estrutura de um registro

| Campo | Armazenado | Notas |
|---|---|---|
| Proposta | ✓ | |
| Nome | ✓ | |
| Número de contato | ✓ | Formato `(11) 912345678` |
| Modalidade | ✓ | Chave da modalidade (ex.: `entrada`) |
| Situação | ✓ | `realizado`, `finalizado`, `pendente`, `nao_concluido`, `anulado` |
| Textos | ✓ | Pedido do cliente, Acordo Realizado |
| Campos digitados | ✓ | Todos os campos preenchidos no Atendimento |
| Parcelas | ✓ | Plano de parcelas completo |
| CPF | ✗ | Nunca é guardado |
| Quem registrou | ✓ | Nome do funcionário |
| Quando | ✓ | Data/hora do registro |
| Versão (`rev`) | ✓ | Número de revisões |

---

## Operações

### Registrar

Acionado pelo botão "Registrar no histórico" no Atendimento.

- Se o registro é idêntico ao existente → não cria duplicata
- Se já existe registro da mesma proposta → pergunta: atualizar (vira revisão) ou criar novo

### Campanha do Fôlego: acordos em aberto e "Cliente cumpriu" (26/11/2026)

Fluxo: o cliente solicita (Negociação) → fecha o acordo (Renegociação) → o acordo fica **em aberto** no Histórico → quando o cliente cumpre, conclui-se aqui.

- **Entrada automática:** na tela **Renegociação**, com modalidade **Campanha do Fôlego** e Resultado **"fechou"**, copiar a mensagem do grupo ("Copiar mensagem") ou a linha da planilha registra o acordo no histórico sozinho (`atFolegoAuto` → `atRegistrar(true)`): não pede as parcelas e não troca de aba. Mesmo acordo já registrado: não duplica. Mesma proposta com dados diferentes: pergunta, como no botão "Registrar no histórico". Faltando campo obrigatório: copia, mas avisa que não registrou.
- **Acordos em aberto** (`#mf-hs-folego`, acima da lista, `hsFolegoDesenhar`): registros da modalidade `folego` com situação `realizado` (dentro dos 30 dias), do mais antigo ao mais novo, com data do acordo, quem registrou e o restante (total com juros − pago parcialmente) e o dia. Some quando não há nenhum.
- **"Cliente cumpriu"** (`hsFolegoCumprir`; também no detalhe do registro): pede confirmação, muda a situação para `finalizado` com revisão (motivo "Cliente cumpriu o acordo (Campanha do Fôlego)", vai para a equipe), conclui o **lembrete pendente da mesma proposta**, se houver (com registro no acompanhamento do lembrete), e gera a mensagem.
- **Mensagem para o grupo** (`hsFolegoTexto`), copiada na hora e editável numa caixa (só em memória):

```
*CAMPANHA DO FÔLEGO — ACORDO CUMPRIDO*

Cliente cumpriu o acordo realizado no dia 21/11
Nome: LEONARDO VIEIRA NUNES
CPF: 473.374.938-43
```

  A data é a do registro do acordo no histórico (`criadoEm`). O **CPF continua fora do histórico**: é buscado no ER pela proposta (`dadosCliente`, aba Cliente da Mesa) só para a mensagem; sem resposta (outro site, sessão expirada), é pedido na hora e pode ficar em branco. Registros do Fôlego já finalizados têm o botão "Mensagem: acordo cumprido" para gerar de novo.
- Nenhuma chave de armazenamento nova: usa a situação `finalizado` que já existia.

### Buscar

Pesquisa por texto livre em: nome, proposta, telefone ou responsável.

### Filtros

- Situação
- Modalidade
- Funcionário
- Período (intervalo de datas)

### Detalhe de um registro

Ao clicar num item do histórico, expande e permite:
- Copiar pedido do cliente
- Copiar acordo realizado
- Copiar telefone
- Copiar resumo
- Abrir no Atendimento (preenche o formulário com os dados do registro)

### Corrigir

Permite editar: situação, número de contato, textos e observações.

- **Motivo obrigatório** para qualquer correção
- A revisão guarda: valor anterior → novo valor, autor, data

### Anular

Muda a situação para `Anulado`, com motivo obrigatório. O registro **nunca é apagado**.

---

## Retenção

Registros ficam por **30 dias** após a última alteração. Depois disso são removidos da cópia local do navegador.

---

## Integração com o módulo de Equipe

Quando o compartilhamento está ativo:
- Novos registros e correções são enviados aos colegas em tempo real
- Conflitos de versão: vence a versão maior; em empate, a alteração mais recente
- Pagamentos e revisões se **somam** (dois funcionários simultâneos não perdem dados)
- O contador de novos acordos aparece como badge na aba Histórico

---

## Exibição na interface

Cada item usa classes CSS para indicar a situação visualmente:

| Situação | Cor da borda lateral |
|---|---|
| `realizado` | Verde |
| `finalizado` | Azul (cliente pagou tudo) |
| `pendente` | Laranja |
| `nao_concluido` | Cinza |
| `anulado` | Vermelho (com opacidade reduzida) |
