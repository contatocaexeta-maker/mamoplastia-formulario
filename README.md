# Funil Mamoplastia — cirurgião plástico

Formulário de captação e qualificação de leads da GreenHub para a campanha de
**tráfego pago para cirurgião plástico** (mamoplastia de aumento).

Gerado a partir do `funil-shark-v2` (biomédica injetora). **Só o copy da capa
mudou** — toda a automação (GTM, CRM, WhatsApp, Telegram, Calendly, anti-fake)
é a mesma, com os mesmos nomes de evento e os mesmos campos de CRM.

Página única, sem build: `index.html` é o arquivo que vai pro ar.

**No ar:** <https://contatocaexeta-maker.github.io/mamoplastia-formulario/>
(repo público `contatocaexeta-maker/mamoplastia-formulario`, GitHub Pages na
branch `main`, raiz — publicado em 14/08/2026).

## Fluxo

```
capa → nome → @ do Instagram → WhatsApp → faturamento → investimento → agendamento
```

Todo mundo responde as duas perguntas. **Quem decide o desfecho é a resposta do
investimento**; o faturamento só separa A de B.

| Nível | Regra | Para onde vai |
|---|---|---|
| **A** | toparia investir **e** fatura acima de 30k | tela de agendamento (Calendly) |
| **B** | toparia investir, mas fatura até 30k | tela de agendamento (Calendly) |
| **C** | "não é o meu momento" (qualquer faturamento) | tela de obrigado |

A regra vive na função `nivelDoLead()` e na constante `FATURAMENTO_NIVEL_B` no
topo do `<script>`.

## Arquivos

| Arquivo | Pra que serve |
|---|---|
| `index.html` | **produção** — CRM + GTM/pixel + notificação de WhatsApp, tudo ligado |
| `gtm-mamoplastia-import.json` | tags/acionadores/variáveis pra importar no container do GTM (ver "Montar o container") |
| `teste-debug.html` | painel no canto mostrando cada evento do dataLayer e cada chamada do pixel ao vivo (sem CRM, sem WhatsApp) |
| `teste-gtm.html` | com GTM, pra usar o Preview do Tag Assistant (sem CRM, sem WhatsApp) |
| `teste.html` | só o formulário, sem rastreio nenhum |

Os três arquivos de teste são **gerados a partir do `index.html`** — ao mexer no
formulário, mexa só no `index.html` e regere os outros.

## Rodar local

```bash
python3 -m http.server 8127 --directory /Users/caexeta/funil-mamoplastia
```

Depois `http://localhost:8127/teste-debug.html` (ou `index.html` pra testar com
CRM e WhatsApp de verdade — cuidado, cria lead).

## O que mudou em relação ao funil-shark-v2

Só a capa e o `<title>`:

| Item | Shark (biomédica) | Aqui (cirurgião plástico) |
|---|---|---|
| Headline | 💉 Biomédica injetora tenha uma **agenda cheia** de **pacientes premium** | 🩺 Cirurgião plástico lote sua **agenda** de **mamoplastia** de aumento |
| Bullet 1 | Pacientes premium de toxina | Pacientes prontas pra operar |
| Bullet 4 | Especialistas em injetáveis | Especialistas em cirurgia plástica |
| Rodapé | registro ativo no CRBM | RQE e título de especialista |
| CTA | QUERO PACIENTES PREMIUM → | QUERO MAIS MAMOPLASTIAS → |

Bullets 2 e 3 (região, script de WhatsApp), as perguntas, as faixas de
faturamento, a recomendação de R$ 35/dia e a paleta taupe continuam iguais.

**Por que as perguntas não mudaram:** manter faturamento e investimento
idênticos é o que permite reaproveitar o container do GTM e o pipeline do CRM
sem reconfigurar nada. Em especial, o campo `investiria_1000_reais_em_anuncio`
só faz sentido enquanto a recomendação for de ~R$ 1.000/mês — se subir o valor
recomendado (o ticket de mamoplastia é bem maior que o de toxina), **é preciso
criar um campo novo no CRM**, senão o nome do campo passa a mentir.

## Rastreio

**GTM:** `GTM-K5K5SFP4` (conta GREEN SHARK) · **Pixel Meta:** `1001727392691930`

⚠️ **Compartilhados com o funil Shark**, por decisão do usuário em 17/08/2026 —
o container próprio (`GTM-THCNMX4W` + pixel `914720614489747`) chegou a ser
montado, mas o formulário foi revertido pro rastreio do Shark. Vantagem: esse
container já está publicado e funcionando. Custo: os leads dos dois funis caem
no mesmo pixel e no mesmo relatório, sem como separar no Meta.

O pixel **não** fica no HTML: ele já é a tag base do `GTM-K5K5SFP4`. Colar o
pixel também no HTML duplicaria PageView e eventos.

O pixel **não** está no HTML: ele entra como tag base no GTM (All Pages), e cada
evento do dataLayer abaixo vira uma tag `trackCustom`.

### Container próprio, se um dia voltar atrás

Existe um container montado e pronto pra este funil: **greenmamoplastia**
(`GTM-THCNMX4W`, conta GREEN SHARK, `accounts/6369595452/containers/261278808`),
com as 11 tags do `gtm-mamoplastia-import.json` já importadas no espaço de
trabalho — mas **nunca publicadas**, e hoje o formulário não aponta mais pra ele.

Pra voltar a usá-lo: trocar o ID nos HTMLs, clicar em **Enviar** no GTM e
conferir no Preview do Tag Assistant.

| Evento no dataLayer | Evento no Meta | Quando |
|---|---|---|
| `iniciou_formulario` | `Etapa1_IniciouForm` | clicou no CTA da capa |
| `colocou_nome` | `Etapa2_Nome` | nome validado |
| `colocou_instagram` | `Etapa3_Instagram` | @ preenchido |
| `colocou_telefone` | `Etapa4_WhatsApp` | número validado |
| `colocou_faturamento` | `Etapa5_Faturamento` | 1ª pergunta (param `faturamento`) |
| `colocou_investimento` | `Etapa6_Investimento` | 2ª pergunta (param `investimento`) |
| `lead_nivel_a/b/c` | `LeadNivelA/B/C` | classificação final (params `nivel`, `faturamento`) |
| `visualizou_calendario` | `VisualizouCalendario` | o calendário apareceu de fato na tela |
| `agendou_reuniao` | — | o Calendly avisou que a reunião foi marcada |
| `lead_classificado` | — | genérico, com `nivel` como parâmetro |

Como todo mundo responde as duas perguntas, **todas as quedas do funil são
gargalo de verdade** — não tem filtro proposital no meio. A Etapa 6 é o último
passo antes da classificação.

`agendou_reuniao` e `lead_classificado` continuam **sem tag no Meta**, igual ao
Shark — os acionadores nem existem no container. Se quiser medir agendamento como
conversão, é só criar a tag de `agendou_reuniao` depois.

Testado em 14/08/2026 no `teste-debug.html`: as 6 etapas, `lead_nivel_a` e
`lead_nivel_c` dispararam e viraram `trackCustom` no pixel (ainda no container
antigo, antes da troca). Depois da troca, confirmado que `GTM-THCNMX4W` carrega e
fica ativo na página — as tags só passam a disparar depois de importar o JSON e
publicar.

## CRM

Webhook do CRM (Green Hub), disparado em **três** momentos: depois do telefone
(cria o card), depois do faturamento (grava a faixa mesmo se largar na pergunta
seguinte) e no final (completa com o investimento). A dedupe é por telefone —
as chamadas seguintes atualizam o mesmo card, não duplicam.

Campos enviados: `nome_e_sobrenome`, `instagram`, `whatsapp` (número),
`qual_seu_faturamento_medio_mensal`, `investiria_1000_reais_em_anuncio`.

`investiria_1000_reais_em_anuncio` só fica vazio quando a pessoa responde o
faturamento e abandona antes da última pergunta.

⚠️ **`pipeline_id` herdado do Shark** (`bef88278-…`) — os leads de mamoplastia
vão cair no mesmo pipeline das biomédicas. Trocar antes de subir a campanha, se
a ideia for separar.

## Calendly

Embed inline na tela final (`CALENDLY_URL`), com `name` e `a1` (WhatsApp)
preenchidos pela query string — o objeto `prefill` do widget não chega no iframe.

Evento: **`assessoriagreenlab/especialistas-em-mamoplastia`** (próprio deste
funil desde 14/08/2026 — antes era o `30min`, herdado do Shark).

Dois detalhes que já custaram tempo, não mexa sem saber:

- o script do Calendly **não** aplica classe no container; a altura vive na nossa
  `.calendly-box`, senão o iframe colapsa pra ~150px;
- **não** dá pra pré-carregar o iframe fora da tela: o navegador não renderiza
  iframe escondido e o Calendly aparece em branco ao ser revelado.

Se o Calendly não der sinal em 12s, a caixa vira um botão "agendar em outra aba".

## Pendências

- [x] GTM e pixel próprios (`GTM-THCNMX4W` / `914720614489747`) — feito em 14/08/2026
- [x] importar o `gtm-mamoplastia-import.json` no container — feito em 14/08/2026
- [ ] **publicar o container** (botão Enviar) — as 24 alterações estão no
      workspace, ainda não no ar
- [x] evento próprio no Calendly — feito em 14/08/2026
- [ ] ainda compartilhado com o Shark, decidir: **pipeline do CRM** e os
      **destinatários do CallMeBot/Telegram**
- [ ] logo do cliente em PNG (a capa hoje não tem logo)
- [ ] conferir se a paleta taupe (`#7A5747`) casa com o criativo desta campanha —
      ela veio do criativo nude da biomédica
- [x] hospedar — GitHub Pages, 14/08/2026
- [ ] apontar a campanha pra URL
- [ ] **conferir se a 1ª pergunta do evento `especialistas-em-mamoplastia` é o
      WhatsApp** — é onde o `a1` cai; se for outra, o telefone entra no campo
      errado. Não deu pra verificar automaticamente (o Calendly não renderiza
      no navegador de teste)
- [ ] conferir o idioma do evento novo (o do Shark estava em inglês)
