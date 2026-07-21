Aqui está o texto do **`README.md`** completo e formatado. Basta copiar todo o conteúdo do bloco de código abaixo, colar em um editor de texto (como o Bloco de Notas ou VS Code) e salvar o arquivo com o nome **`README.md`**:

```markdown
# 📊 Dashboard de Análise de Margem de Contribuição & DRE Executiva

> **Status do Projeto:** Concluído 🚀  
> **Ferramentas Utilizadas:** Power BI Desktop, DAX, Modelagem Relacional (Star Schema), Power Query, GitHub / Notion.  
> **Autor:** Pedro G. de Marchi  

---

## 📌 1. Visão Geral do Projeto

Este projeto consiste na construção de um **Dashboard Financeiro Executivo no Power BI**, focado na análise granular de vendas, custos diretos de produto (COGS), carga tributária incidente e a **Margem de Contribuição (R$ e %)** por linha de produto e período temporal.

O objetivo principal é transformar dados brutos de transações comerciais em inteligência de negócios, permitindo que gestores identifiquem rapidamente:
* Quais categorias e produtos possuem a maior rentabilidade relativa.
* A evolução mensal da Receita Bruta *versus* a Margem de Contribuição líquida.
* A representatividade dos impostos sobre o faturamento bruto.

---

## 🏗️ 2. Arquitetura do Modelo de Dados (Star Schema)

A modelagem de dados foi estruturada no conceito **Star Schema (Esquema Estrela)** para garantir melhor performance de processamento no motor VertiPaq do Power BI e facilitar a legibilidade dos cálculos em DAX.

### 📐 Diagrama do Modelo Relacional


```

```
 ┌──────────────────┐               ┌──────────────────┐
 │   dCalendario    │               │    dProdutos     │
 ├──────────────────┤               ├──────────────────┤
 │ PK: Data         │               │ PK: ID_Produto   │
 │     Ano          │               │     Nome_Produto │
 │     Mes          │               │     Categoria    │
 │     Nome_Mes     │               │     Custo_Direto │
 └────────┬─────────┘               └────────┬─────────┘
          │ 1                                │ 1
          │                                  │
          │ *                                │ *
 ┌────────┴──────────────────────────────────┴─────────┐
 │                      fVendas                        │
 ├─────────────────────────────────────────────────────┤
 │ FK: Data_Venda ──> dCalendario[Data]                │
 │ FK: ID_Produto ──> dProdutos[ID_Produto]            │
 │     ID_Cliente                                      │
 │     Qtd_Vendida                                     │
 │     Preco_Praticado_Unit                            │
 │     Impostos_Pct                                    │
 │     Comissao_Pct                                    │
 └─────────────────────────────────────────────────────┘

```

```

#### Relacionamentos Configurados:
* **`dCalendario[Data]` $\rightarrow$ `fVendas[Data_Venda]`**: Relacionamento **1 para N (1:*)**, filtro fluindo da dimensão calendário para a fato vendas.
* **`dProdutos[ID_Produto]` $\rightarrow$ `fVendas[ID_Produto]`**: Relacionamento **1 para N (1:*)**, filtro fluindo da dimensão produtos para a fato vendas.

---

## 🧮 3. Regras de Negócio e Medidas DAX

Todas as métricas do relatório foram desenvolvidas utilizando **Medidas Explícitas em DAX** (Data Analysis Expressions), garantindo modularidade, correto contexto de filtro e performance otimizada.

### Fórmulas DAX do Projeto:

#### 1. Receita Bruta Total
Calcula o faturamento bruto multiplicando a quantidade vendida pelo preço praticado em cada transação da tabela fato.
```dax
Receita Bruta = 
SUMX(
    fVendas, 
    fVendas[Qtd_Vendida] * fVendas[Preco_Praticado_Unit]
)

```

#### 2. COGS Total (Custo das Mercadorias Vendidas)

Busca o custo unitário direto do produto na tabela de dimensão `dProdutos` e multiplica pela quantidade comercializada na `fVendas`.

```dax
COGS Total = 
SUMX(
    fVendas, 
    fVendas[Qtd_Vendida] * RELATED(dProdutos[Custo_Direto_Unit])
)

```

#### 3. Impostos Totais

Calcula o valor monetário total dos impostos incidentes sobre a venda de cada item (alíquota ajustada sobre o faturamento).

```dax
Impostos Totais = 
SUMX(
    fVendas, 
    (fVendas[Qtd_Vendida] * fVendas[Preco_Praticado_Unit]) * (fVendas[Impostos_Pct] / 100)
)

```

#### 4. Margem de Contribuição (R$)

Representa o valor absoluto resultante da subtração dos Custos Variáveis (COGS) e Impostos em relação à Receita Bruta.

```dax
Margem de Contribuição R$ = 
[Receita Bruta] - [COGS Total] - [Impostos Totais]

```

#### 5. Margem de Contribuição (%)

Métrica percentual que mede a eficiência de conversão da receita bruta em margem. Utiliza a função segura `DIVIDE` para prevenir erros de divisão por zero.

```dax
Margem de Contribuição % = 
DIVIDE(
    [Margem de Contribuição R$], 
    [Receita Bruta], 
    0
)

```

---

## 📈 4. Estrutura do Dashboard Executivo

O relatório foi organizado de forma limpa e intuitiva seguindo padrões de UX para gestão financeira:

1. **Bloco Superior (Cartões KPIs):**
* **Receita Bruta Total:** `R$ 53.900,00`
* **COGS Total:** `R$ 34.150,00`
* **Impostos Totais (18%):** `R$ 9.702,00`
* **Margem de Contribuição R$:** `R$ 10.048,00`
* **Margem de Contribuição %:** `18,64%`


2. **Visão Temporal (Gráfico de Linhas):**
* Acompanhamento mensal da evolução da Receita Bruta e da Margem de Contribuição R$.


3. **Visão de Performance de Produtos (Matriz Hierárquica):**
* Agrupamento por `dProdutos[Categoria]` $\rightarrow$ `dProdutos[Nome_Produto]`.
* Detalhamento de Receita, COGS, Impostos, Margem R$ e Margem %.
* Formatação condicional com barra de dados na coluna de **Margem de Contribuição %**.


4. **Filtros Dinâmicos (Segmentadores de Dados):**
* Seleção temporal por Mês/Ano (`dCalendario`).
* Seleção por Categoria de Produto (`dProdutos`).



---

## 💡 5. Principais Insights de Negócio

* **Margem Consolidada:** O negócio apresenta uma Margem de Contribuição saudável de **18,64%**, gerando **R$ 10.048,00** líquidos para cobertura de despesas fixas e lucro operacional.
* **Impacto Tributário:** A carga tributária consome **R$ 9.702,00** (18% exatos da Receita Bruta), sendo um fator determinante no planejamento de precificação dos produtos.
* **Análise por Categoria:** A matriz interativa permite identificar itens que possuem alto volume de vendas, mas que sofrem compressão de margem por conta de custos diretos elevados.

---

## ⚙️ 6. Como Replicar este Projeto

1. Baixe os arquivos do repositório (`.pbix` e bases `.csv`).
2. Abra o arquivo no **Power BI Desktop**.
3. Verifique os relacionamentos na **Exibição de Modelo**.
4. Confira as medidas na tabela dedicada de `_Medidas`.

```

```