# Handoff — Financeiro v2 (Caixa + Movimentações)

Repo: `gsteixeiradossantos-alt/cobrasq` · branch `main` · arquivo alvo: **`index.html`** (SPA vanilla, seção `#page-fin`).
Rota: `#/fin`. Toda a mudança é no front; o backend já existe (`api/_repassar.js`, `finFluxoProjetar`, `_finFluxoPonte`).

## Sobre os arquivos deste pacote
`Financeiro v2.dc.html` é um **protótipo em HTML/React** (Design Component) que mostra aparência e comportamento finais. **Não copiar o código** — traduzir para o vanilla HTML/CSS/JS do `index.html`, reutilizando as classes e tokens que já existem lá (`.kcard`, `.panel`, `.tbl`, `.fin-*`, `.fincx-nav`, `.finmv-fbtn`, `.fx-sec`). `Financeiro Atual.dc.html` é a cópia da tela atual (referência do "antes"). `Financeiro Remodelado.dc.html` são as explorações — ignorar, exceto as ids 2a/2b citadas abaixo.

Fidelidade: **alta**. Hex, tamanhos e paddings abaixo são finais.

---

## 0. Regras de negócio fechadas com o gestor (fonte da verdade)

| Métrica | Fórmula | Período? |
|---|---|---|
| **Caixa livre** | soma do saldo de TODAS as `fin_conta` − soma das saídas categoria **"Aquisição de dívidas de terceiro"** com `status=0` (em aberto) | não — é "hoje" |
| **Em custódia** | soma das saídas "Aquisição de dívidas de terceiro" em aberto (mesmo número subtraído acima) | não |
| **Lucro líquido** | entradas do período (pagas **e** a receber) − saídas "Aquisição de dívidas de terceiro" do período (pagas **e** a pagar) | sim |
| **Meta de lucro** | valor mensal editável pelo gestor (`DB.config.metaLucro`, default 40.000). Em Dia/Semana/Personalizado usar proporcional: dia = meta/22, semana = meta/4,33, personalizado = meta × meses arredondados (≥28 dias) ou dias/30,4 | sim |
| **Falta fechar** | meta do período − lucro líquido do período. Se lucro < 0 mostrar "lucro negativo no período" e falta = meta inteira | sim |
| **Acordos realizados** | acordos com `assinado_em` dentro do período; valor acordado e ticket médio (valor/quantidade) | sim |
| **Acordos descumpridos** | acordos distintos com ≥1 parcela (entrada) vencida e não paga dentro do período; valor = soma dessas parcelas; % sobre acordos ativos | sim |
| **Contas a pagar vencidas** | saídas **não-repasse** com `status=0` e `data_vencimento < hoje` (independe do período) | não |
| **Contas a receber vencidas** | entradas com `status=0` e `data_vencimento < hoje` (independe do período) | não |
| **Aviso "vence hoje"** | só quando não há NENHUMA vencida: lista o que vence hoje (a pagar e a receber) numa faixa âmbar no rodapé. Se há vencidas, os dois cards aparecem e o aviso não. | — |
| **Ponte (sobra)** | usa `_finFluxoPonte`, mas começa em **Lucro líquido com realização** (pagas 100% + previstas × realização − repasses devidos − repasses contingentes × realização) e tem só 4 linhas: Vencidos anteriores · Parcela do backlog de repasses · Recorrências não lançadas · DAS. Linhas "Já liquidado" e "Realização" SAEM; realização e DAS viram chips de premissa no cabeçalho + botão "Ajustar premissas". | sim |

Classificação de "repasse": `credor_id` definido **ou** categoria "Aquisição de dívidas de terceiro(s)"; tarifa nunca é repasse (regra já em `_finFluxoClassificaSaida`).

---

## 1. Abas

Barra: `Caixa · Movimentações (badge) · Judicial · Fluxo de caixa · Outros LEGADO · Configurações`.
- **Remover** a aba "Recebíveis e repasses" (e o alias de rota). Sua função — repassar ao credor — vira o botão **Repassar** na linha de Movimentações (§3).
- Badge da aba Movimentações = nº de repasses em aberto (`fin_lancamento` saída, categoria aquisição, status 0). Vermelho `#F6E9E7/#7D3F33`. Não renderizar quando 0.
- "Outros" recebe um rótulo `LEGADO` (mono 9px, `rgba(10,21,48,.42)`) e **não muda**.
- Judicial, Fluxo de caixa, Configurações: fora deste handoff.
- Aba ativa: `border-bottom:2px solid #C9A961`, `font-weight:600`, cor `#0A1530`; inativa `rgba(10,21,48,.6)`, peso 500, 13.5px. Já existe em `.pg-tab.on`.

Cabeçalho da página (mantido): eyebrow mono 9.5px `FINANCEIRO · SÁBADO, 05 DE SETEMBRO DE 2026`; H1 Fraunces 500 28px ("Caixa" / "Movimentações · Setembro de 2026" — segue o período); subtítulo 13px `rgba(10,21,48,.6)`; botões à direita "↻ Importar boletos Asaas" (ghost) e "+ Novo lançamento" (sólido `#0A1530`/`#EFEAD9`, radius 9, 12.5px 600).

---

## 2. Aba Caixa (layout "2b · três blocos" adaptado)

Fundo da área: `#F3F4F6`. Padding `20px 28px 28px`, blocos em coluna com `gap:16px`. **Todo bloco filho precisa de `flex-shrink:0`** (a coluna rola; nada colapsa).

### 2.1 Seletor de período (centralizado)
Reaproveitar `.fincx-nav` existente (`‹ | rótulo | ›`, borda `#E6E1D2`, radius 7, fundo branco), mas **centralizado** na linha e sem a faixa de datas abaixo.
- Rótulo: "Hoje" / "Esta semana" / "Setembro de 2026" / "01/09 – 15/09" (personalizado). 14px 600, min-width 190, centralizado.
- Setas: avançam/voltam 1 dia, 1 semana, 1 mês ou o tamanho do intervalo personalizado.
- Clique no rótulo abre o painel (340px, radius 16, sombra `0 8px 32px rgba(10,21,48,.14)`, padding 12), igual ao print do gestor:
  1. Atalhos: **Hoje · Esta semana · Este mês** (15px 500, padding 12/16, radius 9; ativo fundo `#0A1530` texto `#EFEAD9`).
  2. Divisor. `IR PARA UM MÊS` (mono 10px .12em) → `<input type="month">` + botão **Ir** (sólido).
  3. Divisor. `PERÍODO PERSONALIZADO` → dois `<input type="date">` empilhados + botão **Selecionar** (sólido, largura total).
- Inputs: padding 10/12, borda `1.5px solid rgba(10,21,48,.14)`, radius 9, 14px.
- O mesmo estado de período alimenta Caixa **e** Movimentações (§3.1).

### 2.2 Contas bancárias
Título mono 9.5px `.16em` `CONTAS BANCÁRIAS · 7 ATIVAS`; à direita link "Conciliar extrato →" (12px 600).
Grade `repeat(auto-fit, minmax(150px,1fr))`, gap 8:
- 1º cartão **SALDO TOTAL**: fundo `#0A1530`, texto `#EFEAD9`, label mono 8.5px `#C9A961`, valor mono 15px 600, sub "soma das 7 contas" 11px `rgba(239,234,217,.6)`.
- Demais: fundo `#fff`, borda `1px solid rgba(10,21,48,.10)`, radius 10, padding 12/14. Linha 1: avatar 20px redondo com sigla (cores: Asaas `#1E6FE8`, BTG `#0A1530`, CENSEC `#5A6472`, Conta Inicial `#4A7A52`, Nubank `#7B2CBF`, Registro Civil `#97793A`, TJ `#3A6491`) + nome 12px 600 (ellipsis); linha 2: saldo mono 15px 600; linha 3: tipo 11px `#5A5F66`.
- Dados: `fin_conta` (nome, tipo, `balance`).

### 2.3 Quatro indicadores do período
Grade `repeat(auto-fit, minmax(240px,1fr))`, gap 16 (2×2 abaixo de ~1050px). Cards radius 12, padding 20/22, gap interno 8. Label mono 9.5px `.16em`. Valor principal mono **26px** 600 `white-space:nowrap` (nunca quebrar moeda).

1. **LUCRO LÍQUIDO · MÊS** — escuro `#0A1530`/`#EFEAD9`, label `#C9A961`. Valor (vermelho `#E2B4AA` se negativo). Duas linhas 12px `rgba(239,234,217,.7)`: "entradas (recebidas + a receber)" → verde `#BFD4B9`; "− aquisição de dívidas (pagas + a pagar)" → `#E2B4AA`.
2. **META DE LUCRO · MÊS** — branco, borda `rgba(10,21,48,.10)`. Canto direito "meta mensal" 11px. Valor = **input editável** (mono 22px 600, sem borda, `border-bottom:1.5px dashed rgba(10,21,48,.25)`), prefixo "R$" mono 16px cinza. Salva em `DB.config.metaLucro` no blur/enter (aceitar "40.000,00"). Barra 8px `#EEF0F4`, preenchimento `#C9A961` (verde `#4A7A52` quando ≥100%). Linhas: "− lucro já garantido (recebido + a receber)" (verde) e "**= falta fechar**" (vermelho `#9B4A3F`; fora do mês: "= falta fechar (meta proporcional R$ X)"; batida: "= meta batida, excedente" verde).
3. **ACORDOS REALIZADOS · MÊS** — branco. Valor = quantidade + sufixo "acordos assinados" (Inter 13px `#5A5F66`). Linhas: "valor acordado" e "ticket médio" (mono).
4. **ACORDOS DESCUMPRIDOS · MÊS** — branco; borda `#E7CFC7` e valor `#9B4A3F` quando > 0. Sufixo "acordos com parcela vencida". Linhas: "parcelas vencidas sem pagamento" (valor vermelho) e nomes dos devedores (até 3) → "N% de M ativos".

### 2.4 Vencidas (condicional)
Só quando existir ao menos uma vencida. Grade 2 colunas gap 16. Cada card: borda `#E7CFC7`, radius 12; cabeçalho fundo `#FBEAE7`, título Fraunces 16px `#7D3F33` ("Contas a pagar vencidas" / "Contas a receber vencidas"), contagem 12px, total mono 14px 600 `#9B4A3F` à direita. Linhas 13px: descrição 600 + sub `venceu 03/09 · há 2 dias · Nubank` (11.5px `#9B4A3F`), valor mono 600 vermelho, botão ghost **Pagar** / **Cobrar** (11.5px 600, radius 6, borda `rgba(10,21,48,.14)`). Card vazio (quando só um lado tem vencidas): "Nenhuma conta a pagar vencida." 12.5px.

### 2.5 Ponte "Do lucro líquido à sobra de set/26"
Reaproveitar `section.fx-sec` (borda `#E6E1D2`, radius 9, overflow hidden, 14px). **Adicionar `flex-shrink:0`.**
- Cabeçalho: H2 Fraunces 500 17px "Do lucro líquido à sobra de {mês}"; à direita chips `Realização 85%` e `DAS 9,0%` (pill radius 20, borda `rgba(10,21,48,.14)`, 11.5px, valor mono 600) + botão sólido "Ajustar premissas" (abre o painel de premissas já existente em Fluxo de caixa). Nota 12px `#6B7683` abaixo.
- Linha 1 (700): "Lucro líquido de set/26 com realização de 85%" · valor.
- 4 linhas (padding 11/18, divisor `#F2EFE6`): Vencidos de meses anteriores puxados para este mês · Parcela do mês dos repasses vencidos com devedor já pago · Recorrências que ainda não viraram lançamento · DAS de 9,0% sobre a receita própria. Negativos em `#7D3F33`.
- Rodapé (700, fundo `#FBFAF6`): "Sobra antes da retirada" · valor.
- Mês do título segue o período (semana → "31/08 — 06/09").

### 2.6 Aviso "vence hoje" (condicional)
Faixa `#FBF3E1`, borda `#DFC992`, radius 10, padding 12/18, 13px `#7A6428`, dot 8px `#C9A961`: "Nada vencido. 1 conta(s) a pagar vence(m) hoje · R$ 151,00 · …". Só quando **não** há vencidas.

**Removido da Caixa atual:** os 2 alertas do topo, card "Meta de setembro" (4 números), os 5 KPIs, "Resultado por categoria", "Inadimplência", lista "Contas" antiga, "Decidir agora". Tudo que sobrou está nos §2.2–2.6.

---

## 3. Aba Movimentações (layout "2a" sem cabeçalho)

Começa direto nos filtros (o gestor pediu para tirar caixa livre/contas/comissão daqui).

### 3.1 Linha de filtros (`flex-wrap:wrap; row-gap:10px; flex-shrink:0`, padding 16/28/12)
- `TIPO` (mono 9.5px) + chips **Tudo · Entradas · Saídas · Repasses (n)**; divisor; `SITUAÇÃO` + chips **Todos · Pendentes · Em atraso · Conciliados**. Chip: radius 20, padding 6/12, 12.5px 500, borda `rgba(10,21,48,.14)`; ativo fundo `#0A1530` texto `#EFEAD9`. Trocar filtro **limpa a seleção**.
- À direita (`margin-left:auto`): `.fincx-nav` existente (`‹ 01/09/2026 — 30/09/2026 ›`, altura 28, rótulo mono 12.5px) abrindo o **mesmo painel** de §2.1 (alinhado à direita), e o botão `.finmv-fbtn` **Filtrar** existente (ícone funil 13px, borda `#E6E1D2`, radius 7, padding 8/12).
- Abaixo: "12 de 12 lançamentos" 12.5px; com filtro Repasses, acrescentar "· repasses ao credor saem daqui pelo botão **Repassar** — não existe mais aba separada" em `#97793A`.

### 3.2 Tabela (o painel inteiro rola; tabela cresce com o conteúdo; `overflow:hidden` só para o radius)
Wrapper: margem 0/28, borda `rgba(10,21,48,.10)`, radius 12.
Grid: `30px 26px minmax(170px,1.6fr) minmax(96px,1fr) minmax(84px,.7fr) 112px 100px minmax(128px,auto)`, gap 10, padding 10/14 (mínimo ≈ 816px — cabe a 1180).
- Cabeçalho `#F4F5F7`, mono 9.5px `.14em` `rgba(10,21,48,.5)`: ☐ · · DESCRIÇÃO · CATEGORIA · CONTA · VALOR (dir.) · SITUAÇÃO · AÇÃO (dir.). Checkbox "selecionar todos" respeita o filtro.
- Grupo por dia (`.fin-daygrp` existente): fundo `#FBF8EF`, mono 10.5px `#97793A` "HOJE · SÁB 05/09" + total do dia à direita.
- Linha 13px, divisor `rgba(10,21,48,.08)`, hover `rgba(10,21,48,.04)`, selecionada `#F6F3EA`. **Linha de repasse em aberto: fundo `#FBFAF6`.**
  - checkbox 15px radius 4; ícone `.fin-ti` 26px (↓ verde `#E9EDE2/#4A7A52`, ↑ bordô `#F3E7E2/#A65A4A`);
  - descrição 500 + sub 11.5px `rgba(10,21,48,.5)`; categoria em `.fin-catpill`; conta 12.5px; valor mono 600 (`+`/`−`, verde `#4A7A52` / vermelho `#9B4A3F`);
  - situação pill mono 9.5px `.08em` radius 4: PAGO `#EEF0F4/#5A6472` · CONCILIADO `#E9F0E9/#4A7A52` · ATRASADO `#F6E9E7/#9B4A3F` · A RECEBER `#E8EEF5/#3A6491` · A PAGAR / PENDENTE `#FBF3E1/#7A6428` · PIX ENVIADO `#E8EEF5/#3A6491` · REPASSADO `#E9F0E9/#4A7A52`.
  - AÇÃO (à direita, gap 6): **Repassar** (só saída categoria aquisição, status 0, sem transfer) — sólido `#0A1530`/`#EFEAD9`, 11.5px 600, radius 7, ícone setas 11px, hover `#C9A961`/`#2A1A05`; **Confirmar** (ghost) para A PAGAR / A RECEBER não-repasse; "aguardando Asaas ◔" (`#97793A`) quando `repasse_status=preparado`; "Comprovante ↗" (verde) quando efetuado; sempre "Abrir".
- Vazio: "Nenhum lançamento com esses filtros. **Limpar filtros**" centralizado, padding 38/16.
- Rodapé sem seleção: "Totais do filtro · entradas +X · saídas −Y · líquido Z" (12.5px, mono nos valores) — **do filtro**, nunca fixo.
- Barra de lote (com seleção): `position:sticky; bottom:12px`, fundo `#0A1530`, radius 12, sombra `0 8px 24px rgba(10,21,48,.28)`: "N selecionados / soma", **Repassar N ao credor** (verde `#4A7A52`, só se há repasses selecionados), Confirmar pagamento · Alterar categoria · Reagendar (ghost claro), "Limpar seleção ✕".

### 3.3 Modal "Repasse ao credor · PIX via Asaas"
Reaproveitar `.overlay/.modal` (max-width 560, `max-height:90vh`, corpo com `overflow-y:auto`). Chama **`POST /api/repassar` com `{ lancamento_id, credor_id?, pix_key? }`** (já suportado).
- Cabeçalho: eyebrow mono 9.5px `#97793A` "REPASSE AO CREDOR · PIX VIA ASAAS"; título Fraunces 20px = descrição do lançamento; ×.
- Grid 2 col: **VALOR DO PIX** (mono 24px 600) + "parcela 3 de 6 · devedor X"; **DESCRIÇÃO NO EXTRATO DO CREDOR** = `descricaoPix(ref)` ("3/6 - Bidão Auto Mecânica") em caixa `#F4F5F7` mono 13px.
- **CREDOR**: se `credor_id` → cartão avatar dourado + nome + CNPJ. Se não → aviso `#F6E9E7/#E7CFC7` "Lançamento sem credor vinculado — escolha abaixo. A escolha vale para as outras parcelas em aberto deste devedor." + lista de rádios (clientes). Envia `credor_id`.
- **CHAVE PIX**: `clientes.chave_pix` (fonte em verde "chave PIX do cadastro do cliente"); sem credor, "será a chave do credor escolhido" em `#97793A`. Editável → envia `pix_key`.
- Caixa `#FBFAF6` borda `#E6E1D2`: Capital do caso · Já repassado · **Resta após este PIX** (via `saldoDeCapital`); barra verde (já) + dourada (este). Se estourar: texto vermelho "passa do teto em R$ X" e **Confirmar desabilitado** (espelha o 409 `teto_capital`).
- Nota 12px: "Ao confirmar: PIX sai pelo Asaas, esta saída é baixada como paga, o comprovante do banco vai ao credor por WhatsApp e fica anexado na ficha do caso."
- Rodapé: Cancelar (ghost) · **Confirmar PIX de R$ 2.400,00** (verde `#4A7A52`; cinza `#EEF0F4` quando inválido).
- Resposta: `repasse_status=efetuado` → linha vira REPASSADO + "Comprovante ↗", toast "PIX de R$ X enviado · comprovante indo ao credor por WhatsApp"; `preparado` → PIX ENVIADO + "aguardando Asaas ◔", toast "…Asaas ainda processando"; `skipped` → toast informativo, sem duplicar. Toast escuro, radius 10, 6s, com **Desfazer** apenas visual (não há undo de PIX — o botão só fecha o toast; remover se preferir).
- Lote "Repassar N ao credor": abre o modal para o primeiro; ao confirmar, segue para o próximo. Idempotência já garantida pelo backend (claim pendente→preparado).

---

## 4. Estado
`finPeriodo = { modo: 'hoje'|'semana'|'mes'|'custom', ref: 'YYYY-MM-DD', de, ate }` (compartilhado pelas duas abas, persistir em `localStorage`). `movFiltro = { tipo, situacao }`, `movSel = []`, `metaLucro` em `DB.config`. Recalcular tudo no `change` do período.

## 5. Tokens (já existem em `:root` do index.html)
`#0A1530` ink · `#EFEAD9` side-fg · `#C9A961` accent · `#97793A` accent-2 · `#4A7A52` verde · `#9B4A3F` bordô · `#F3F4F6` bg · `#FBFAF6` creme · `#E6E1D2` borda creme · `rgba(10,21,48,.10)` borda · `rgba(10,21,48,.6)` texto-2 · `rgba(10,21,48,.42)` texto-3. Fontes: Fraunces (títulos), Inter Tight (UI), JetBrains Mono (valores, labels uppercase). Radius: cards 12, pills 20, botões 7–9.

## 6. Armadilhas (custaram bug no protótipo)
1. Card com `overflow:hidden` dentro de coluna flex que rola → **`flex-shrink:0`** obrigatório, senão colapsa para 0.
2. Valores em moeda com `white-space:nowrap`; nunca `overflow-wrap:anywhere` em número.
3. Contadores e totais sempre derivados da lista filtrada.
4. "Comissão a entrar"/lucro previsto **não** subtrai os repasses já em custódia (senão fica negativo); custódia só entra no Caixa livre.
5. Layout íntegro a 1180 e 1440px.

## Prompt sugerido para o Claude Code
> Leia `design_handoff_financeiro_v2/README.md` e abra `Financeiro v2.dc.html` no navegador (referência visual; não copiar o código) e os PNGs em `screenshots/`. Aplique no `index.html` (`#page-fin`), um diff por etapa: (1) barra de abas sem "Recebíveis e repasses", badge de repasses em Movimentações, rótulo LEGADO em Outros; (2) estado `finPeriodo` compartilhado + painel do seletor (Hoje/Esta semana/Este mês/Ir para mês/Personalizado) reaproveitando `.fincx-nav`; (3) aba Caixa conforme §2 com as fórmulas do §0 — meta editável em `DB.config.metaLucro`; (4) aba Movimentações conforme §3 com botão Repassar por linha, modal e chamada `POST /api/repassar { lancamento_id, credor_id?, pix_key? }`, refletindo `preparado/efetuado/skipped`; (5) ponte da sobra com 4 linhas partindo do lucro líquido. Use só tokens/classes existentes no `index.html`. Não tocar em `api/*`, migrações nem na aba Outros. Confirmar layout a 1180 e 1440px e rodar `node test/f29_fluxo_caixa_projecao.test.js` e `f07` ao final.

## 7. Aceite
- [ ] Aba Recebíveis removida; badge de Movimentações = repasses em aberto.
- [ ] Caixa: seletor centralizado com painel (Hoje/Esta semana/Este mês/Ir para mês/Personalizado); contas + total; 4 indicadores com as fórmulas do §0; meta editável persistida; cards de vencidas só quando há vencidas; aviso "vence hoje" só quando não há.
- [ ] Ponte começa no lucro líquido, 4 linhas, chips de premissa.
- [ ] Movimentações: filtros + `.fincx-nav` + Filtrar; tabela por dia; Repassar só em saídas de aquisição em aberto; modal chama `/api/repassar` com `lancamento_id`; estados preparado/efetuado refletidos na linha; lote.
- [ ] Nenhum texto cortado a 1180px; nada colapsa ao rolar.
