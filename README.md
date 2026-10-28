<div align="center">

# ERP Industrial

**ERP industrial feito sob medida — do orçamento ao caminhão, do chão de fábrica à DRE.**

Sistema web interno que roda a operação inteira de uma indústria brasileira de equipamentos
para estética e saúde: vendas no varejo, atacado, feiras e marketplaces; produção por fluxos
configuráveis; compras; estoque endereçado com coletor; faturamento de NF-e; logística;
rastreabilidade ANVISA e controladoria.

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-3.1-000000?logo=flask&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1)
![Celery](https://img.shields.io/badge/Celery-5.5-37814A?logo=celery&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker_Swarm-2496ED?logo=docker&logoColor=white)
![Traefik](https://img.shields.io/badge/Traefik-v3-24A1C1?logo=traefikproxy&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white)

<img src="docs/prints/10-dashboard.png" alt="Dashboard de vendas e faturamento" width="100%">

</div>

> **Sobre este repositório.** É uma vitrine: o código-fonte é proprietário e não está aqui.
> Os prints são do sistema em produção com **dados mascarados**: pessoas viram *Cliente Exemplo NN*
> ou *Colaborador NN*, nomes de empresas viram *Empresa A, B, C…*, a marca e os logos foram
> ocultados, documentos e telefones aparecem zerados e todos os valores monetários foram
> multiplicados por um fator fixo. Nenhum dado aqui é real.

---

## A história

> Tudo começou com uma planilha.

Em 2023, os orçamentos eram feitos pelos vendedores em Excel. Cada um tinha o seu arquivo e nada
ficava salvo num lugar só: para saber o que tinha sido orçado, para quem e por quanto, era preciso
perguntar — ou torcer para a planilha ainda existir. Aquilo me incomodava.

Em julho de 2023 comecei um sistema pequeno, com um objetivo só: **o vendedor monta o orçamento
numa tela e ele fica salvo**. O repositório nem tinha esse nome — chamava-se *Seller*. Em agosto
já existia o primeiro "ponto de restauração: orçamento completo".

Só que cada problema resolvido deixava o próximo à vista. O orçamento salvo precisava virar pedido
no ERP fiscal sem ninguém redigitar. O pedido precisava ser cobrado — vieram o PIX e o boleto.
O cliente precisava aprovar — veio o WhatsApp. A fábrica precisava saber o que produzir e em que
ordem — veio o motor de fases. Depois o estoque endereçado, a separação, a nota fiscal, a
logística, as compras, o financeiro… Sempre a mesma pergunta guiando o próximo passo:
**o que ainda está dando trabalho para alguém aqui dentro?** Fui integrando uma coisa de cada vez
para facilitar a vida de quem usa — e o formulário de orçamento virou o ERP que roda a empresa
inteira, do vendedor ao caminhão, do chão de fábrica à DRE.

**Liderei o projeto do começo ao fim, e o sistema foi desenvolvido inteiramente por duas pessoas.**

| Quando | O que entrou |
|---|---|
| **jul–ago/2023** | Orçamento substitui o Excel, carteira de orçamentos, análise financeira do pedido |
| **2024** | Rastreabilidade ANVISA, cobrança PIX e boleto, filas assíncronas (Celery), dashboard de vendas, expedição |
| **2025** | Separação, logística, painel de operações com motor de fases, pronta entrega, DRE |
| **2026** | Gestão de produção e compras, migração para Docker Swarm, ETL do ERP fiscal, controle de acesso (RBAC), estoque endereçado com coletor, faturamento de NF-e em lote, rateio por canal, app de feira offline, DANFE em ZPL |

Commits por ano: **2023** 508 · **2024** 1.167 · **2025** 2.036 · **2026** 1.376 (até outubro).

---

## Em números

| | |
|---|---|
| **2** pessoas no desenvolvimento | **3** anos de evolução contínua |
| **910** endpoints HTTP | **235** telas/templates Jinja |
| **163** modelos de dados | **85** módulos de serviço |
| **~148 mil** linhas de Python | **~243 mil** linhas de HTML/JS/CSS |
| **5.000+** commits desde jul/2023 | **20+** integrações externas |
| **6** filas Celery priorizadas | **~30** jobs agendados |

## Sumário

- [A história](#a-história)
- [Arquitetura](#arquitetura)
- [Comercial](#comercial) · [Marketplaces](#e-commerce-e-marketplaces)
- [Operações e motor de fases](#operações-e-motor-de-fases)
- [Gestão de Produção](#gestão-de-produção) · [Compras](#compras) · [Etiquetas](#etiquetas)
- [Separação, faturamento e expedição](#separação-faturamento-e-expedição) · [Logística](#logística)
- [Estoque e WMS](#estoque-e-wms) · [Coletor (PWA)](#coletor-mstock-pwa-offline) · [Rastreabilidade](#rastreabilidade)
- [Indicadores de produção](#indicadores-de-produção)
- [Financeiro e controladoria](#financeiro-e-controladoria)
- [Cadastros](#cadastros) · [Administração e segurança](#administração-e-segurança)
- [Integrações](#integrações) · [Jobs agendados](#jobs-agendados) · [Destaques de engenharia](#destaques-de-engenharia)

---

## Arquitetura

```mermaid
flowchart LR
    subgraph CLI["Clientes"]
        B["Navegador"]
        P["PWA coletor<br/>celular do galpão"]
        F["App da feira<br/>tablet offline"]
    end
    subgraph SW["Docker Swarm"]
        T["Traefik v3<br/>TLS Let's Encrypt"]
        A["Flask + Gunicorn<br/>3 workers x 16 threads"]
        R[("Redis 7")]
        W["Workers Celery<br/>6 filas priorizadas"]
        BT["Celery beat"]
        M[("MySQL 8 / Percona")]
        PR["Prometheus + Grafana"]
    end
    EXT["APIs externas<br/>Tiny, Itaú, ML, Shopee<br/>Jamef, Melhor Envio, WhatsApp"]

    B -->|HTTPS| T
    P -->|HTTPS| T
    F -->|HTTPS| T
    T --> A
    A <-->|SQL| M
    A -->|enfileira| R
    BT -->|agenda| R
    R --> W
    W <--> EXT
    W --> M
    EXT -->|webhooks| T
    A -. "SSE kanban" .- R
    PR -. "exporters" .- M
    PR -. "exporters" .- R
```

- **Aplicação:** Python 3.12, Flask 3.1 (blueprints por módulo), SQLAlchemy 2.0, Jinja, Gunicorn.
  Atualização em tempo real do kanban de produção por **SSE + Redis pub/sub**.
- **Assíncrono:** Celery 5.5 com Redis como broker, backend, lock distribuído e cache, em seis filas
  separadas por tipo de trabalho e com prioridade.
- **Webhooks** (Tiny, Itaú PIX, Mercado Livre, Mercado Pago, PayPal, WhatsApp, Twilio, Melhor Envio)
  respondem `200` na hora e enfileiram o processamento por prioridade. Uma **reconciliação periódica**
  cobre os eventos que se perdem no caminho.
- **Deploy:** esteira com GitHub Actions publicando imagens Docker e atualizando o Swarm;
  HTTPS com certificado renovado automaticamente. Monitoramento com Prometheus e Grafana.

---

## Comercial

### Orçamento e pedido de venda

<img src="docs/prints/11-orcamento-novo.png" alt="Criação de orçamento" width="100%">

- Montagem do pedido item a item com **personalização por produto** (cor, carenagem, plastificação,
  logo/bordado), acessórios vinculados à poltrona, brindes e descontos por linha ou no total.
- **Cotação de frete na própria tela** (Jamef e Melhor Envio) e busca do cliente no Tiny pelo CPF/CNPJ.
- Venda, **comodato** (empréstimo com devolução em até 60 dias) ou **reserva de pronta entrega**
  com o saldo de cada SKU visível linha a linha.
- Regras de negócio no servidor: celular do cliente precisa receber WhatsApp e não pode ser de
  funcionário; combos de curso travam frete e itens; pedido já em produção não é editável.

### Carteira de orçamentos e aprovação pelo cliente

<img src="docs/prints/12-orcamentos.png" alt="Lista de orçamentos" width="100%">

- Lista com ~20 filtros (cliente, status, empresa, SKU, transportadora, feira, erro de integração,
  pronta entrega vencendo…). Cada vendedor enxerga só a própria carteira e a dos seus representantes.
- **Aprovação pelo cliente:** o sistema envia um link por **WhatsApp (Twilio)** e e-mail; o cliente
  aprova numa página pública marcando as confirmações, que ficam registradas.
- Depois da análise financeira, o pedido nasce no **Tiny ERP** e os itens entram no motor de fases.
- **Cobrança integrada:** PIX Itaú com página pública de pagamento, **Bolecode** (boleto + PIX) com PDF
  próprio, Mercado Pago e PayPal. Confirmação de pagamento por webhook.

<details>
<summary><b>Mais telas do comercial</b> — combos, feiras, listas de preço, pós-venda</summary>

#### Combos e promoções
Kits com validade; uma só cor vale para todos os itens variáveis, e o sistema oferece só as cores
comuns a todos. "Iniciar orçamento" já cria o pedido no servidor.

<img src="docs/prints/13-combos.png" alt="Combos" width="100%">

#### Feiras
Cadastro de feiras com sigla, local e datas, com total vendido por feira.
A sigla trava após o primeiro pedido porque está impressa no protocolo `SIGLA-XXXXXX`.

<img src="docs/prints/14-feiras.png" alt="Feiras" width="100%">

#### Listas de preço
Preço, IPI, ICMS por região, distribuidor, NCM, GTIN e peso, com paginação no banco, exportação
Excel do recorte e sincronização com as listas de preço do Tiny V3.

<img src="docs/prints/15-listas-precos.png" alt="Listas de preço" width="100%">

#### Pós-venda: análise de pedidos e cotação de frete
Pedidos ativos classificados por prazo (atrasado / alerta / ok) a partir da data de despacho
calculada. Fila de cotação com alertas de 24 h e 48 h, cotação no Melhor Envio pelas medidas das
caixas e bloqueio da Jamef para UFs onde ela só cota aéreo (com liberação auditada).

<img src="docs/prints/17-analise-pedidos.png" alt="Análise de pedidos" width="100%">

</details>

### App da feira (offline)

<img src="docs/prints/02-feira-tablet.png" alt="App de feira no tablet" width="100%">

- App web instalável que **funciona sem internet** no tablet do estande: a vendedora monta o pedido,
  o cliente digita os próprios dados.
- Fila local que sobe em lote quando a rede volta; um identificador gerado no tablet impede pedido
  duplicado quando o sinal cai, e **o lote nunca é recusado inteiro** — problemas viram avisos no orçamento.
- Cada tablet é cadastrado uma vez; depois disso o app abre e trabalha sem depender da rede.

---

## E-commerce e marketplaces

<img src="docs/prints/52-separacoes-tiny.png" alt="Separações de marketplace por canal" width="100%">

- **Mercado Livre + Shopee no mesmo painel:** faturamento, comissão, líquido, gasto com Mercado Ads,
  ROAS e lucro, vendas por dia, top anúncios e últimas vendas.
- OAuth2 do Mercado Livre com renovação automática; Shopee Open Platform v2 com assinatura **HMAC-SHA256**.
- Importação em segundo plano com trava no Redis contra execução dupla.
- **Estoque Full:** quanto está à venda no armazém do ML e por que o restante está parado.
- **DRE por canal** com as taxas reais de cada marketplace (comissão, envio, parcelamento).
- **Separações por canal** (Mercado Livre, Shopee, Site, TikTok Shop, Amazon, venda direta) lidas de um
  espelho local do Tiny atualizado a cada 10 minutos — o mesmo dado que o coletor usa no galpão —, com
  prazo de despacho e número de série gravado no pedido ao concluir.

---

## Operações e motor de fases

<img src="docs/prints/30-operacoes.png" alt="Painel de operações" width="100%">

O coração do sistema: cada **item** de cada pedido percorre fases (triagem, logo, impressão, corte e
costura, estofamento, programação, inspeção, expedição…) e o painel mostra a fila por setor.

- **Motor de transições** cadastrado em tabela (tela *Status Transições*) com **bifurcações por regra
  de negócio**: item com logo vai para a fila de logo; pendência financeira desvia para o financeiro;
  a linha de produção é decidida pelo SKU; comodato termina em "Aguardando Retorno"; ao resolver uma
  pendência o item volta à fase anterior, lida do histórico.
- **Controle de concorrência:** trava por pedido + `SELECT … FOR UPDATE`, e a tela envia a fase que
  *mostrou* — se outra pessoa já moveu o item, o avanço é recusado em vez de pular uma etapa.
- **Três relógios por item** (na fase, no setor, desde a aprovação) com selo colorido de urgência.
- Algumas fases só se deixam **bipando o QR** da peça; "Bordar Teste" só sai com a foto do bordado.
- **Histórico auditável** com vocabulário fixo de eventos e **fila ao vivo** que avisa o que mudou.
- **WhatsApp automático ao cliente:** previsão de entrega em dias úteis, pendência financeira e pedido
  concluído com o link da NF — sem nunca repetir a mesma mensagem.

---

## Gestão de Produção

<img src="docs/prints/32-gestao-producao.png" alt="Kanban de produção" width="100%">

- **Fluxos de produção configuráveis**, com etapas em sequência e **fork em ramos paralelos** (com
  dependência entre ramos); a ordem pai espera e avança sozinha quando todos terminam.
- **Versionamento de fluxo:** editar cria uma versão nova; a antiga fica "encerrando" até a última
  ordem dela sair — nada em andamento é afetado.
- Ordens nascem de forma manual, por gatilho ou pela **verificação de estoque** (ponto de reposição +
  lote econômico), que roda sozinha no Celery e tem **modo simulação** que faz tudo e desfaz no fim.
- **Moldes de co-produção:** uma corrida produz N itens (ex.: a chapa do corte a laser), com cesta
  de itens oportunistas e rateio do custo da corrida.
- **Chão de fábrica:** cronômetro por pessoa, "modo posto" (um computador por setor, cada operador
  com seu código), proteção contra toque duplo e **kanban em tempo real** (SSE + Redis pub/sub).
- Ao concluir, **baixa os insumos da estrutura e credita o estoque** com o custo daquele momento.
- Relatórios de tempo por funcionário × fase, **padrão do item** por mediana ponderada (descartando
  cronômetros esquecidos) e custo de produção. PDFs de OP e etiqueta regulatória.

<details>
<summary><b>Mais telas da produção</b> — fluxos, moldes, locais, embalagem, estrutura de produtos, controle, apontamento</summary>

#### Fluxos de produção
Cada item tem seu fluxo: etapas em sequência agrupadas por local, com blocos de **ramos paralelos**
quando o produto se divide em subconjuntos que andam ao mesmo tempo.

<img src="docs/prints/32b-fluxos.png" alt="Fluxos de produção" width="100%">

#### Moldes de co-produção e locais
Um molde rende uma cesta fixa de itens por ciclo; quando qualquer item da cesta atinge o ponto de
reposição, nasce um único kanban para a corrida inteira. Locais e fases definem onde cada etapa
acontece e como os volumes são conferidos.

<img src="docs/prints/32c-moldes.png" alt="Moldes" width="100%">
<img src="docs/prints/32d-locais.png" alt="Locais e fases" width="100%">

#### Embalagem
Fila do posto de embalagem, etiquetas de volume com QR `EMB|lote|produto|volume` e conferência volume
a volume por bipe e/ou **peso com tolerância de 10%**.

<img src="docs/prints/33-embalagem.png" alt="Embalagem" width="100%">

#### Movimentações de estoque da produção
Cada consumo e entrada com o **custo congelado na hora do movimento**, totais e alerta de movimento sem custo.

#### Estrutura de produtos (BOM)
Estrutura multinível com bloqueio de dependência circular, vigência, "onde é usado" e clonagem em lote.
**Famílias de variação em 3 camadas** (pai → valor → SKU) para famílias de ~1.500 SKUs, sempre com
prévia antes de gravar.

<img src="docs/prints/35-estrutura-produtos.png" alt="Estrutura de produtos" width="100%">

#### Controle de produção e apontamento
Registro de peças por número de série e operadores; programação de produção para estoque que entra
no mesmo motor de fases das vendas.

<img src="docs/prints/36-controle-producao.png" alt="Controle de produção" width="100%">
<img src="docs/prints/31-apontamento.png" alt="Apontamento" width="100%">

</details>

---

## Compras

<img src="docs/prints/37-compras.png" alt="Compras por fases" width="100%">

- **Fluxo por fases configurável na própria tela:** canais (reposição ou manual), checklist de
  aprovações condicionado a canal e valor, campos e anexos obrigatórios por fase, fases que só
  existem conforme o momento do pagamento.
- Recebimento parcial por item com conversão de unidade; o saldo vira um pedido novo. Devolução total
  ou parcial na conferência. "Juntar" pedidos do mesmo fornecedor.
- **XML da NF-e de entrada** é validado, cadastra o fornecedor (sem duplicar CNPJ) e vira **nota de
  entrada no Tiny V3** com de-para de produto e CFOP.
- Ordem de compra numerada com PDF e envio por e-mail.

### Planejamento de reposição

<img src="docs/prints/39-reposicao.png" alt="Planejamento de reposição" width="100%">

Substituiu uma planilha. A demanda de cada componente é **explodida pela estrutura de produtos em
todos os níveis** a partir das vendas, e o cálculo usa o lead time medido nas últimas compras:

- Estoque de segurança `SS = z·√(L·σ² + d²·σL²)`
- Ponto de reposição `ROP = d·L + SS`
- Lote econômico `LEC = √(2DS / ic)`, ajustado ao lote mínimo e ao múltiplo do fornecedor
- A decisão olha a **posição** (estoque + pedidos + solicitações abertas) e pode gerar as solicitações sozinha.

<details>
<summary><b>Ordem de compra e relatórios</b></summary>

<img src="docs/prints/38-ordem-compra.png" alt="Ordem de compra" width="100%">
<img src="docs/prints/40-compras-relatorios.png" alt="Relatórios de compras" width="100%">

Variação de preço (pelas notas de entrada, excluindo transferências internas), gastos, prazos de
entrega e tempo de cotação por responsável.

</details>

---

## Etiquetas

<img src="docs/prints/42-etiquetas-pcp.png" alt="Etiquetas PCP" width="100%">

- Etiquetas de produto com sequência de **número de série + QR**, por pedido ou de embalagem com
  **EAN-13**, símbolos de manuseio e regulatórios (Classe II, Tipo B, WEEE) e **fonte calculada
  medindo o texto** para caber na área.
- Etiquetas de PCP em PDF (WeasyPrint) para matéria-prima, semiacabado e acabado, com QR do lote.
- Geração de **ZPL** para impressoras térmicas Zebra.

---

## Separação, faturamento e expedição

<img src="docs/prints/54-faturamento-lote.png" alt="Faturamento em lote" width="100%">

- **Fila de separação** calculada numa única query (antes eram ~500), lista de separação em PDF com
  QR por pedido e status calculado no SQL (aguardando → separando → separado → embalando → embalado).
- **Faturamento em lote pela API V3 do Tiny:** gera a NF-e, emite na SEFAZ e busca a etiqueta do
  marketplace, em blocos.
- **Sem nota duplicada:** o id da nota é gravado *antes* da emissão, há lock no Redis por separação,
  e grava-se a situação real da SEFAZ (HTTP 200 não é nota autorizada) com reconsulta posterior.
- Recusa faturar se o peso ainda não chegou ao Tiny (a nota congela o pedido).
- **DANFE simplificada desenhada em ZPL 4"×6"** a partir do XML autorizado — o Tiny não oferece esse formato.
- Romaneio em PDF, etiquetas de envio, anexos fiscais (DIFAL, carta de correção) sinalizados por pedido.

<details>
<summary><b>Mais telas</b> — listas de separação e expedição</summary>

<img src="docs/prints/51-listas-separacao.png" alt="Listas de separação" width="100%">
<img src="docs/prints/53-expedicao.png" alt="Lista de expedição" width="100%">

</details>

---

## Logística

<img src="docs/prints/55-logistica.png" alt="Logística" width="100%">

- Grade dos últimos 90 dias (faturados / a faturar / todos) com filtros em chips combináveis.
- **Roteirização:** geocodifica os endereços (Nominatim), ordena pela distância da origem e gera o PDF da rota.
- Entre plantas: fila de transporte com conferência de carga por **bipe e/ou pesagem**, cargas em
  trânsito, **recebimento volume a volume** no destino e retiradas em fornecedor.

<details>
<summary><b>Fila de transporte, cargas em trânsito e recebimento</b></summary>

<img src="docs/prints/56-fila-transporte.png" alt="Fila de transporte" width="100%">
<img src="docs/prints/56b-transito.png" alt="Cargas em trânsito" width="100%">
<img src="docs/prints/56c-recebimento.png" alt="Recebimento" width="100%">

</details>

---

## Estoque e WMS

<img src="docs/prints/57-estoque.png" alt="Estoque endereçado" width="100%">

- **Estoque por endereço** (~1.300 endereços de picking, buffer, expedição e quarentena).
- **Um único ponto de escrita** (`mover()`): toda movimentação é uma transferência origem → destino,
  atômica, com locks em ordem fixa e uuid que impede gravar o mesmo evento duas vezes.
- **Lote e FIFO:** recusa retirar lote novo quando há um mais antigo e diz onde ele está.
- **Reabastecimento do picking** pelo buffer, com mínimo/máximo por SKU e endereço — a tarefa nasce
  na mesma transação em que o saldo cruza o mínimo.
- **Inventário por contagem cega** com ajuste automático e relatório de acuracidade.
- **Sinais de saúde:** saldo negativo, endereço parado e conferência de que a soma dos endereços bate
  com o estoque global.

<details>
<summary><b>Sinais, reabastecimento, movimentos, itens com foto, pronta entrega e comodato</b></summary>

#### Sinais, reabastecimento e movimentos
Painel de saúde do estoque (saldo negativo, endereço parado, soma dos endereços × estoque global),
tarefas de reabastecimento do picking e o registro de toda movimentação origem → destino.

<img src="docs/prints/57b-estoque-sinais.png" alt="Sinais do estoque" width="100%">
<img src="docs/prints/57c-estoque-reabastecimento.png" alt="Reabastecimento" width="100%">
<img src="docs/prints/57d-estoque-movimentos.png" alt="Movimentos de estoque" width="100%">

#### Catálogo de itens com foto
~31 mil SKUs agrupados em ~5 mil produtos, placar de cobertura de fotos e upload pelo celular com
rotação EXIF corrigida; a foto do pai vale para as variações.

<img src="docs/prints/58-itens-fotos.png" alt="Itens com foto" width="100%">

#### Pronta entrega
Saldo, limites e produção em andamento por SKU; reservas com prazo, encomendas e liberação automática
quando há saldo real.

<img src="docs/prints/59-pronta-entrega.png" alt="Pronta entrega" width="100%">

#### Comodato
Peças disponíveis, a separar, emprestadas (com quem e até quando) e devolvidas, com destino da peça
decidido na conferência (comodato, estoque normal ou quarentena).

<img src="docs/prints/60-comodato.png" alt="Comodato" width="100%">

</details>

### Coletor MStock (PWA offline)

<p align="center"><img src="docs/prints/01-coletor-mobile.png" alt="Coletor no celular" width="320"></p>

- App instalável no celular que **funciona sem rede no galpão**: cada bipe vira um evento com uuid
  numa fila local (IndexedDB) e o servidor responde item a item — *aplicado*, *duplicado* ou *rejeitado*.
- Entrada, saída, transferência, separação de pedido, guarda da produção, reabastecimento com
  contagem cega, expedição volume a volume, envio ao Mercado Livre Full com número de série.
- Cada aparelho é cadastrado uma vez e cada bipe fica registrado com o operador que o fez.
  O catálogo de produtos é baixado por diferença.

---

## Rastreabilidade

<img src="docs/prints/61-rastreabilidade.png" alt="Rastreabilidade" width="100%">

Equipamentos regulados pela ANVISA: número de série × NF × pedido × cliente × empresa, preenchido
automaticamente no faturamento e no envio ao ML Full, com exportação no formato do procedimento da
qualidade (PSGQ).

---

## Indicadores de produção

<img src="docs/prints/45-fluxo-setor.png" alt="Fluxo por setor" width="100%">

- **Fluxo por setor:** entradas, saídas e fila de cada setor a partir do histórico de fases, com
  comparação anual e **projeção** que distribui o ritmo de entrada pela proporção histórica de passagem
  entre setores, com fator sazonal e marcação de gargalo.
- **"Dia útil" deduzido da atividade real** da fábrica, não de um calendário fixo.
- **Atrasos por setor** em dias úteis fracionados (feriados móveis calculados: Páscoa, Carnaval, Corpus Christi).
- **Produtividade:** produzido, % no prazo, lead time, análise de safra (coorte), mix e filtros pelos
  segmentos do SKU.

<details>
<summary><b>Atrasos, produtividade, pedidos expirados, tendência e linhas de produção</b></summary>

<img src="docs/prints/43-atrasos-producao.png" alt="Atrasos de produção" width="100%">
<img src="docs/prints/44-produtividade.png" alt="Produtividade" width="100%">

**Pedidos expirados e tendência de atrasos:** datas de reacordo vencidas, % de atraso sobre a data
prevista e a evolução semanal em janelas móveis de 30 dias, filtrável por categoria e linha.

<img src="docs/prints/46-pedidos-expirados.png" alt="Pedidos expirados" width="100%">
<img src="docs/prints/47-tendencia-atrasos.png" alt="Tendência de atrasos" width="100%">

**Linhas de produção:** o cadastro que agrupa os produtos para os indicadores e projeções por setor.

<img src="docs/prints/48-linhas-producao.png" alt="Linhas de produção" width="100%">

</details>

---

## Financeiro e controladoria

<img src="docs/prints/73-metricas-faturamento.png" alt="Métricas de faturamento" width="100%">

- **DRE consolidada hierárquica** (grupo → subcategoria → categoria → lançamento) com layout editável:
  totalizadores, linhas personalizadas e mescladas, e a regra de origem de cada linha visível.
- **DRC (fluxo de caixa)** pela data real de liquidação, classificado em atividades operacionais,
  de investimento e de financiamento.
- **Mapa de rateio:** cada categoria de receita e despesa dividida entre canais (varejo, atacado,
  marketplaces, site…) somando 100%, com **ajuste manual que sobrevive à recarga** dos dados e
  processamento pesado em fila dedicada.
- **Curva ABC** por canal, vendedor e UF; **métricas de faturamento** com os mesmos números do e-mail
  semanal automático e filtros em 11 dimensões.
- **Contas em aberto:** encontra CP/CR que a cópia local mostra abertas e já mudaram no Tiny, e
  reatualiza em lote respeitando o limite da API.
- **Fretes:** importa CT-e (XML/ZIP), compara cobrado × calculado e lança as contas a pagar.
- **Estoque valorizado** com o custo visível só para quem tem permissão.

<details>
<summary><b>Mais telas financeiras</b></summary>

<img src="docs/prints/72-curva-abc.png" alt="Curva ABC" width="100%">
<img src="docs/prints/74-estoque-valorizado.png" alt="Estoque valorizado" width="100%">
<img src="docs/prints/75-fretes.png" alt="Fretes" width="100%">

</details>

---

## Cadastros

<img src="docs/prints/80-produtos.png" alt="Produtos" width="100%">

- **Produtos** espelhados do Tiny com medidas, caixa de embalagem e limites de estoque.
- **Cadastro de família de variações** (cor × carenagem × plastificação × logo) com SKU sequencial,
  enviada ao Tiny em lotes de 20 num worker dedicado que trava a família durante o processo.
- **EAN-13** na faixa GS1 da empresa, com dígito verificador e checagem de duplicidade.
- **Manuais com QR permanente:** a etiqueta física aponta para um link curto cujo destino é resolvido
  a cada leitura — troca-se o PDF ou vídeo sem reimprimir.
- **Famílias de etiqueta ANVISA** (registro, responsável técnico, AFE, advertências) servindo ~70 etiquetas de caixa.
- Acessórios compatíveis, caixas de embalagem, transportadoras e usuários.

<details>
<summary><b>Cadastrar produto, manuais e famílias de etiqueta</b></summary>

<img src="docs/prints/81-cadastrar-produto.png" alt="Cadastrar produto" width="100%">
<img src="docs/prints/82-manuais.png" alt="Manuais" width="100%">
<img src="docs/prints/83-familias-etiqueta.png" alt="Famílias de etiqueta" width="100%">

</details>

---

## Administração e segurança

<img src="docs/prints/84-acessos-rbac.png" alt="Gestão de acessos RBAC" width="100%">

- **RBAC no banco:** catálogo de permissões por tela, ação, API, download e botão, concedidas por
  setor ou por usuário com *permitir* e *negar*. Permissão efetiva = setor + permitir − negar.
  O menu inteiro é montado a partir delas.
- **Máquina de estados** dos pedidos editável por tela (*Status Transições*).
- **Sessão única por usuário**, horário de acesso por perfil e trilha de auditoria das ações.
- **Telemetria de uso** por tela e setor (contador em memória gravado em lote), incluindo telas que
  ninguém abre; auditoria de login e eventos.
- **Carga do Tiny sob demanda ou agendada** por empresa e período, com modo simular e histórico.
- **Assistente interno com LLM local** (Ollama) e chatbot de WhatsApp com Gemini.

<img src="docs/prints/00-login.png" alt="Tela de login" width="100%">

<details>
<summary><b>Status Transições, carga do Tiny e gerenciador</b></summary>

<img src="docs/prints/85-status-transicoes.png" alt="Status transições" width="100%">
<img src="docs/prints/86-tiny-pesquisa.png" alt="Carga do Tiny" width="100%">
<img src="docs/prints/87-gerenciador.png" alt="Gerenciador" width="100%">

</details>

---

## Integrações

| Integração | Uso |
|---|---|
| **Tiny ERP** (API V2 e V3 OAuth) | Pedidos, contatos, NF-e (emissão e consulta), separação, expedição, peso, etiquetas, produtos, listas de preço, contas a pagar/receber, notas de entrada. Webhook + reconciliação + respeito ao `Retry-After`. |
| **Itaú** | PIX com checkout próprio e Bolecode (boleto + PIX) com webhook de pagamento. |
| **Mercado Livre** | OAuth2, vendas, Mercado Ads, estoque Full, webhooks. |
| **Shopee** Open Platform v2 | Vendas e financeiro com assinatura HMAC-SHA256. |
| **Mercado Pago · PayPal** | Links de pagamento e webhooks. |
| **WhatsApp Cloud API · Twilio** | Aprovação de pedido, avisos de produção e entrega, chatbot. |
| **Jamef · Melhor Envio** | Cotação de frete; Melhor Envio também compra o frete e gera a etiqueta. |
| **Google Drive · Sheets** | Anexos dos pedidos e logos; custos e faixa GS1. |
| **Gemini · Ollama** | Chatbot de WhatsApp; assistente interno com modelo local. |
| **Cloudinary · SMTP · FTP/SFTP** | Artes de logo, e-mails transacionais, manuais. |
| **Receita (cnpj.ws) · Assertiva · Nominatim** | Consulta de CNPJ/IE, análise de crédito, geocodificação de rotas. |

## Jobs agendados

Celery beat no fuso de São Paulo, com minutos escalonados para os jobs não colidirem.

| Frequência | Job |
|---|---|
| 5 min | Verificação de estoque da produção (gera ordens), agenda de cargas do Tiny |
| 10 min | Cópia das separações do Tiny, movimentos do ML Full, liberação de encomendas e reposição de pronta entrega |
| 15 min | Planejamento de compras |
| 30 min – 1 h | Reconciliação de pedidos e notas dos últimos 3 dias do Tiny, por empresa |
| 1 h – 12 h | Renovação automática das autorizações das integrações |
| 2× por hora | Notas paradas na SEFAZ e etiquetas pendentes |
| 1 h | Reenvio de peso pendente, expiração e fechamento de reservas, conferência de notas de compra |
| Diário | Baixa de itens não localizados, vencimento de comodato, relatório financeiro da Shopee |
| Semanal | Relatório de faturamento por e-mail |

---

## Destaques de engenharia

1. **Faturamento de NF-e idempotente** — id gravado antes da emissão, lock distribuído e situação real da SEFAZ.
2. **DANFE simplificada em ZPL** desenhada a partir do XML autorizado, com código de barras nativo da Zebra.
3. **Dois apps offline-first** (coletor e feira) com fila local idempotente.
4. **Motor de fases com bifurcação por regra** e controle de concorrência pela "fase vista" na tela.
5. **Fluxos de produção com fork, versionamento e co-produção**, kanban em tempo real por SSE.
6. **Estoque endereçado com um único ponto de escrita**, FIFO, reabastecimento transacional e inventário cego.
7. **Planejamento de reposição** com SS/ROP/LEC sobre a estrutura explodida e lead time medido.
8. **Webhooks com rede de segurança:** resposta imediata, fila priorizada e reconciliação periódica.
9. **RBAC granular no banco** e telemetria de uso por tela.

---

<div align="center">

<img src="docs/prints/99-menu-aberto.png" alt="Menu completo do sistema" width="100%">

Projeto liderado por [@AmyNunes](https://github.com/AmyNunes) · desenvolvido por uma equipe de duas pessoas · código proprietário · prints com dados mascarados

</div>
