# 📊 Dashboard de Análise Comercial – E-commerce Olist

Dashboard interativo desenvolvido no **Power BI** para analisar o desempenho comercial de uma operação de e-commerce brasileiro, com base no dataset público da **Olist** (100k+ pedidos, 2016–2018).

![Dashboard Comercial Olist](img/dashboard-preview.png)

---

## 🎯 Visão Geral do Projeto

Este projeto simula uma demanda real de negócio: transformar dados brutos de transações de vendas em um painel interativo capaz de responder às principais perguntas estratégicas da diretoria de uma empresa de e-commerce:

- Qual foi o faturamento total, número de pedidos e a evolução mensal das vendas?
- Quais categorias de produtos mais geram receita?
- Como o faturamento se distribui geograficamente pelos estados brasileiros?
- Qual a participação de cada método de pagamento no total de vendas?

---

## 💡 Insights Chave Encontrados

- **Faturamento Total:** R$ 16,01 milhões no período analisado.
- **Faturamento do Último Mês:** R$ 996,90 mil, com uma retração de **-4,1%** em relação ao mês anterior (indicando um ponto de atenção para a diretoria).
- **Total de Pedidos Únicos:** 99 mil pedidos processados no período.
- **Top Categoria:** *beleza_saude* lidera o ranking de faturamento, seguida por *relogios_presentes*, *cama_mesa_banho*, *esporte_lazer* e *informatica_acessorios*.
- **Concentração Geográfica:** São Paulo (SP) é disparadamente o estado com maior volume de faturamento (R$6M+), seguido por Rio de Janeiro (RJ) e Minas Gerais (MG); a partir do 4º colocado (RS) o faturamento cai fortemente, reforçando a concentração de vendas no eixo Sudeste.
- **Meio de Pagamento:** Cartão de Crédito representa **78,34%** de todas as transações (R$12,54M), seguido por Boleto (17,92% / R$2,87M) e Voucher (2,37% / R$0,38M), com Cartão de Débito e pedidos "Não Definido" tendo participação marginal (juntos, <1,4%).

---

## 🛠️ Tecnologias e Etapas Utilizadas

### 1. Excel
Utilizado na etapa inicial para validação rápida e inspeção exploratória dos arquivos CSV brutos (`olist_orders_dataset.csv`, `olist_order_payments_dataset.csv`, `olist_products_dataset.csv`) antes da carga no Power BI.

### 2. Power Query (ETL)
Toda a limpeza e transformação dos dados foi feita no Power Query Editor do Power BI:
- **Ajuste de tipos de dados:** colunas de valor formatadas como Número Decimal Fixo (Moeda) e colunas de data como Data/Hora.
- **Tratamento de nulos:** pedidos com status "Cancelado" ou "Indisponível" foram filtrados para considerar apenas vendas efetivadas; categorias nulas foram substituídas por "Não Especificado".
- **Colunas calculadas:** criação de coluna de faturamento total (`Quantidade x Preço Unitário + Frete`) quando não disponível diretamente na base.
- **Mesclagem de tabelas (joins):** junção da tabela de pedidos com a tabela de produtos/categorias via `product_id`, trazendo nomes legíveis para as categorias.

### 3. Modelagem de Dados
- Relacionamento entre as tabelas originais do Olist (`olist_orders_dataset`, `olist_order_items_dataset`, `olist_order_payments_dataset`, `olist_products_dataset`), relacionadas diretamente via `order_id` e `product_id`, sem consolidação em uma tabela fato única.
- Criação de uma tabela calendário dedicada (`d_Calendario`), gerada por intervalo fixo de datas e enriquecida com colunas auxiliares para análises temporais:
```dax
d_Calendario = 
ADDCOLUMNS(
    CALENDAR(
        DATE(2016,1,1),
        DATE(2018,8,31)
    ),
    "Ano", YEAR([Date]),
    "NumeroMes", MONTH([Date]),
    "MesNome", FORMAT([Date], "MMMM"),
    "MesAbrev", FORMAT([Date], "MMM"),
    "AnoMes", FORMAT([Date], "YYYY/MM"),
    "Trimestre", "T" & FORMAT([Date], "Q"),
    "DiaSemana", FORMAT([Date], "dddd"),
    "FimDeSemana", IF(WEEKDAY([Date],2) > 5, "Fim de Semana", "Dia Útil")
)
```

### 4. DAX (Data Analysis Expressions)
Medidas centralizadas em uma tabela `_Medidas`:

```dax
Total Faturamento = 
CALCULATE(
    SUM ( olist_order_payments_dataset[payment_value] ),
    CROSSFILTER ( olist_order_items_dataset[order_id], olist_orders_dataset[order_id], BOTH )
)

Total Pedidos Únicos = 
DISTINCTCOUNT ( olist_order_payments_dataset[order_id] )

Ticket Médio = 
DIVIDE (
    [Total Faturamento],
    [Total Pedidos Únicos],
    0
)

Faturamento Último Mês = 
VAR UltimaData = MAX ( d_Calendario[Date] )
VAR AnoMesAtual = FORMAT( UltimaData, "YYYY/MM" )
RETURN
    CALCULATE (
        [Total Faturamento],
        d_Calendario[AnoMes] = AnoMesAtual
    )

Faturamento Penúltimo Mês = 
VAR UltimaData = MAX ( d_Calendario[Date] )
VAR MesAnterior = EDATE ( UltimaData, -1 )
VAR AnoMesAnterior = FORMAT( MesAnterior, "YYYY/MM" )
RETURN
    CALCULATE (
        [Total Faturamento],
        d_Calendario[AnoMes] = AnoMesAnterior
    )

Crescimento Último Mês % = 
DIVIDE (
    [Faturamento Último Mês] - [Faturamento Penúltimo Mês],
    [Faturamento Penúltimo Mês],
    BLANK ()
)

Top 5 Categorias por Faturamento = 
CALCULATE (
    [Total Faturamento],
    TOPN (
        5,
        ALL ( olist_products_dataset[product_category_name] ),
        CALCULATE ( [Total Faturamento] ),
        DESC
    )
)
```

### 5. Design e Visualização
- Layout de página única seguindo hierarquia visual: cartões de KPI no topo, gráfico de evolução temporal no centro, e gráficos de categorias/estado/pagamento na parte inferior.
- Paleta de cores sóbria (azul-marinho como cor dominante, cinza claro para fundo), sem poluição visual desnecessária.

---

## 📂 Estrutura do Repositório

```
├── /data          → Amostra da base de dados utilizada (Olist)
├── /dashboard      → Arquivo .pbix do Power BI
├── /img            → Capturas de tela e GIF demonstrativo do relatório
└── README.md
```

---

## 🚀 Como Visualizar o Dashboard

O dashboard completo e interativo está disponível para download nesse repositório:

1. Baixe o arquivo `.pbix` na pasta [`/dashboard`](./dashboard).
2. Abra no **[Power BI Desktop](https://www.microsoft.com/pt-br/download/details.aspx?id=58494)** (gratuito).
3. Explore os filtros, segmentações e interações entre os visuais.

Não é necessário nenhuma licença paga — apenas o Power BI Desktop instalado.

---

## 📌 Fonte dos Dados

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — Kaggle.

---

## 👤 Autor

Desenvolvido como projeto de portfólio em Análise de Dados / Business Intelligence.

`#AnaliseDeDados #PowerBI #BusinessIntelligence #DataAnalytics #Portfolio`
