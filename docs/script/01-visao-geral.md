# Visão Geral do Script — paymobifin-assistente.user.js

**Arquivo:** `paymobifin-assistente.user.js` (até a v13.27.1 se chamava `automacaomobfacil.user.js`; esse nome ficou como ponte para migrar as instalações antigas e foi removido em 19/11/2026)  
**Versão atual:** 13.43.0  
**Tipo:** Tampermonkey Userscript  
**Sites alvo:**
- `https://paymobi-app.ercard.com.br/*` — painel principal (ER). O endereço antigo `https://paymobi.ercard.com.br/*` continua aceito (`@include` com `paymobi(-app)?`)
- `https://enviamotor.mobfacil.com.br/*` — motor de análise de crédito
- `https://app.tallos.com.br/*` — RD Atendimento (encerramento com IA)
- `https://paymobi.com.br/*` — site da Paymobi: só responde ao ER o Acesso ao APP (Boleto+) das Efetivadas; não mostra nada no site

---

## O que o script faz

O **PaymobiFin - Assistente** é um userscript que adiciona um painel flutuante ao sistema ER da Paymobi. Ele automatiza e centraliza tarefas do dia a dia da equipe de cobrança e crédito:

- Coleta dados de análise de crédito diretamente da árvore do MobFacil (score, restritivos, consultas, CPF, motivo de reprovação)
- Gera e copia linhas formatadas para as planilhas de Negociação e Renegociação
- Registra e compartilha o histórico de acordos entre os funcionários
- Gerencia lembretes de pagamentos parciais com sincronização em equipe
- Realiza consultas CAC (Central de Atendimento ao Cliente)
- Faz download de CCBs em lote
- Permite troca rápida entre contas do sistema
- Gera mensagens de acordos para grupos de WhatsApp (Negociação/Renegociação) e mensagens de retorno ao cliente (aba Mensagens)
- Fornece timers e agendamentos de expediente

---

## Estrutura de módulos

O script é dividido em dois pontos de entrada principais:

| Função | URL | Responsabilidade |
|---|---|---|
| `iniciarMotor()` | `enviamotor.mobfacil.com.br` | Coleta dados da árvore de análise de crédito dentro do iframe do motor |
| `iniciarPainel()` | `paymobi-app.ercard.com.br` (e o antigo `paymobi.ercard.com.br`) | Renderiza o painel completo com todas as abas e funcionalidades |

### Módulos internos ao painel (`iniciarPainel`)

| Seção no código | Módulo | Linha aprox. |
|---|---|---|
| Leitura da árvore | Motor de coleta React | 153 |
| Iframe do motor | Navegação da árvore MobFacil | 452 |
| Interface base | Painel, abas, arrastar, redimensionar | 974 |
| CCBs | Download de certificados | 3624 |
| Contas | Troca rápida de conta | 3816 |
| Gerador de Negociações | Modelos (grupo e cliente), campos e montagem dos textos | 4108 |
| Lembretes | Pagamentos parciais (v2) | 4591 |
| Consulta CAC | Central de Atendimento | 4984 |
| Relatório de Pendências | Pendências do ER (somente leitura) | 5330 |
| Atendimento | Negociação, Renegociação, Desbloqueios e Mensagens | 5537 |
| Timers | Agendamentos de expediente | 5907 |
| Navegação/Aparência | Nova UI / UI Antiga | 6243 |
| Equipe | Compartilhamento via Pusher | 6361 |
| Histórico | Acordos registrados pela equipe | 6828 |
| Busca de cliente | Central de busca somente leitura | 7057 |
| Navegação automática | Troca de tela automática no ER | 7363 |
| Mesa de crédito | Datas do dia | 7371 |
| RD Atendimento | Encerramento com IA (app.tallos.com.br) | 7427 |
| Boleto+ (`iniciarBoletoPlus`) | Acesso ao APP das Efetivadas (paymobi.com.br, só responde ao ER) | fim do arquivo |

---

## Dependências e permissões

```
@grant  GM_setValue / GM_getValue / GM_deleteValue  — armazenamento persistente
@grant  GM_setClipboard                             — copiar para área de transferência
@grant  unsafeWindow                                — acesso aos objetos do site (React)
@grant  GM_xmlhttpRequest                           — requisições HTTP externas
@connect api-sa1.pusher.com                         — envio via Pusher
@require https://js.pusher.com/8.4.0/pusher.min.js  — biblioteca do Pusher
@run-at  document-idle                              — executa após o carregamento da página
```

---

## Armazenamento

Todos os dados ficam no armazenamento do Tampermonkey (por navegador). Nenhum dado é enviado para servidores externos, exceto as mensagens criptografadas do compartilhamento de equipe (Pusher).

Chaves principais:

| Chave | Conteúdo |
|---|---|
| `dados` | Resultados da coleta de crédito por proposta |
| `fila` / `feitas` / `ignoradas` | Controle da fila de auditoria |
| `mfLembretes` | Lembretes de pagamentos (v2) |
| `mfAcordos` | Histórico de acordos (30 dias) |
| `equipeNome/Secret/Chave/Ativa` | Configuração do compartilhamento |
| `negModelos` / `negModalidade` | Modelos e preferências de negociação |
| `navAuto` | Estado da navegação automática |

Documentação completa das chaves: ver [README principal](../../README.md#9-armazenamento-tampermonkey).

---

## Documentação por módulo

- [02 - Motor de Coleta (MobFacil)](./02-motor-coleta.md)
- [03 - Painel Principal e Interface](./03-painel-interface.md)
- [04 - Atendimento (Negociação e Renegociação)](./04-atendimento.md)
- [05 - Histórico de Acordos](./05-historico.md)
- [06 - Lembretes e Timers](./06-lembretes-timers.md)
- [07 - Consulta CAC](./07-cac.md)
- [08 - CCBs e Contas](./08-ccbs-contas.md)
- [09 - Equipe e Compartilhamento](./09-equipe.md)
- [10 - Navegação Automática e Busca](./10-navegacao-busca.md)
- [11 - RD Atendimento: Encerramento com IA](./11-rd-encerramento.md)
- [12 - Desbloqueios](./12-desbloqueios.md)
- [14 - Analisar proposta](./14-analisar-proposta.md)
