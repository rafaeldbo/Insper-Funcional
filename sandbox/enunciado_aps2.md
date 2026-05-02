# Projeto 2: Programação Funcional - Otimização de Carteiras

## Informações Gerais
* **Disciplina:** Programação Funcional.
* **Docentes:** Raul Ikeda e Fábio Ayres.
* **Instituição:** Insper.
* **Semestre:** 2026-1.
* **Prazo de Entrega:** 10 de maio de 2026, às 23:59 via GitHub.

---

## Contexto do Problema
Um gerente de portfólio deseja simular uma carteira de investimentos baseada em critérios específicos para descobrir a melhor alocação de ativos.

### Definição de Carteira
* Uma carteira é um vetor de percentuais ($w$) que indica a alocação em cada ativo.
* A soma de todos os pesos no vetor deve ser igual a 1.

### Critério de Qualidade: Sharpe-Ratio
Uma boa carteira busca alto retorno e baixa volatilidade. A métrica utilizada é o Sharpe-Ratio anualizado:

$SR = \frac{\mu - r_{free}}{\sigma}$

**Onde:**
* $SR$: Sharpe-ratio anualizado.
* $\mu$: Retorno anualizado da carteira.
* $r_{free}$: Taxa livre de risco anual.
* $\sigma$: Volatilidade anualizada da carteira.

---

## Cálculos Matemáticos

### Retorno da Carteira
* **Diário:** $r_p = r \cdot w$, onde $r$ é a matriz de retornos diários e $w$ é o vetor de pesos.
* **Anualizado:** Média de $r_p$ multiplicada por 252.

### Volatilidade da Carteira
* **No período:** $\sigma_p = (w^T \cdot C \cdot w)^{1/2}$, onde $C$ é a matriz de covariância.
* **Anualizada:** $\sigma_p$ multiplicado por $\sqrt{252}$.

---

## Restrições e Complexidade
O objetivo é maximizar o Sharpe-Ratio através de força bruta.

* **Seleção de Ativos:** Escolher 20 ações entre as 30 disponíveis no índice Dow Jones.
* **Alocação:** Carteira *long-only* ($w_i \ge 0$).
* **Concentração:** Máximo de 20% em um único ativo ($w_i \le 0.2$).
* **Escala do Problema:**
    * ~30 milhões de combinações de ações.
    * 1 milhão de simulações de pesos por combinação.
    * Total de aproximadamente 30 trilhões de simulações.

---

## Requisitos do Projeto
* **Trabalho individual**.
* **Dados:** Utilizar o período de 01/07/2025 a 31/12/2025.
* **Ativos:** 30 ações do Dow Jones.
* **Implementação:**
    * Simulação paralelizada.
    * Uso obrigatório de **funções puras** nas partes paralelizadas.
    * Linguagem livre (verificar rubrica para impacto na nota).
* **Documentação:** README contendo descrição, instruções de instalação e execução.

---

## Rubrica de Avaliação
* **I:** Projeto incompleto.
* **D:** Ausência de paralelismo ou elementos funcionais.
* **C+:** Linguagem multi-paradigma (ex: Python, Java).
* **B+:** Linguagem estritamente funcional (ex: Haskell, OCaml, Scala).

### Itens Opcionais (Bônus/Ajuste de conceito)
* Obtenção de dados via API.
* Backtesting dos resultados para o primeiro trimestre de 2025.
* Comparação de performance entre execução serial e paralela (mínimo de 5 execuções cada).

---

## Guia de Implementação Funcional

### Por que Programação Funcional?
* Ausência de efeitos colaterais facilita a concorrência.
* Estado não compartilhado permite paralelismo seguro.
* Abstrações como `map`, `filter` e `reduce` são ideais para processamento massivo.

### Funções Puras Necessárias
As seguintes operações devem ser puras para facilitar o pipeline paralelo:
* Cálculo de retorno, volatilidade e Sharpe Ratio.
* Geração de simulação de pesos aleatórios.

### Exemplo de Pipeline
```haskell
carteirasPossiveis
  |> parMap avaliarCarteira
  |> filter carteiraValida
  |> maximumBy compararSharpe
```


### Estratégias de Mitigação de Carga
Dado o volume de 30 trilhões de simulações, sugere-se:
* Testar com subconjuntos de combinações.
* Persistência de resultados parciais e logs intermediários.