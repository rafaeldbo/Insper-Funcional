# Plano Linear de Desenvolvimento (F#)

Objetivo: entregar uma solucao em F# para otimizacao de carteiras (Sharpe-Ratio), com pipeline funcional e simulacao paralela.

## Estrutura Macro Alvo

1. Console CLI (ponto de entrada): `Main`
2. Lib de extracao de dados externos: `PortfolioAnalysis.Data`
3. Lib de processamento e analise: `PortifolioAnalysis` (ou renomear para `PortfolioAnalysis.Core`)
4. Projeto de testes comparativos: `PortfolioAnalysis.Tests`

## Principios de Arquitetura Funcional (aplicados em todas as fases)

1. Funcoes puras no nucleo matematico e no pipeline paralelo.
2. Efeitos colaterais (HTTP, disco, console, tempo, RNG) isolados em bordas.
3. Modelagem com `record` e `discriminated union`, evitando estado mutavel compartilhado.
4. Composicao com pipeline (`|>`), `map`, `filter`, `fold`, `Result` e `Async`.
5. Determinismo em testes: semente de aleatoriedade controlada.

---

## Fase 1 - Estruturacao da Solucao e Projetos

Dependencias: nenhuma.
Inicio: ambiente dotnet vazio.
Fim: estrutura macro final conectada por referencias.

Passos:

1. Manter/ajustar estrutura para ter quatro papeis claros:
   1. `Main`: orquestracao e CLI.
   2. `PortfolioAnalysis.Data`: acesso a APIs e cache local.
   3. `PortifolioAnalysis`: dominio, metricas, simulacao.
   4. `PortfolioAnalysis.Tests`: testes unitarios e comparativos.
2. Adicionar projeto de testes caso ainda nao exista.
3. Configurar referencias:
   1. `Main` referencia `PortfolioAnalysis.Data` e `PortifolioAnalysis`.
   2. `PortfolioAnalysis.Data` referencia `PortifolioAnalysis` apenas se precisar de tipos compartilhados.
   3. `PortfolioAnalysis.Tests` referencia `PortifolioAnalysis` e, quando necessario, `PortfolioAnalysis.Data`.
4. Definir padrao de nomes de modulos por responsabilidade.

Checklist de saida:

1. Solucao compila com os projetos conectados.
2. Fronteiras entre I/O e nucleo puro estao explicitas.

---

## Fase 2 - Modelagem de Dominio e Contratos

Dependencias: Fase 1.
Inicio: criar tipos centrais no projeto de analise.
Fim: contratos estaveis para alimentar data, simulador e CLI.

Passos:

1. Criar tipos imutaveis:
   1. `AssetId`, `DailyReturn`, `PricePoint`.
   2. `AssetSeries` (ticker + serie temporal).
   3. `PortfolioWeights`.
   4. `PortfolioMetrics` (`mu`, `sigma`, `sharpe`).
2. Criar DUs para erros de dominio:
   1. `ValidationError`.
   2. `DataError`.
   3. `SimulationError`.
3. Implementar validacoes puras:
   1. soma dos pesos = 1 (com tolerancia numerica).
   2. $w_i \ge 0$.
   3. $w_i \le 0.2$.
4. Padronizar funcoes que falham com `Result<'T, 'Error>`.

Checklist de saida:

1. Modelo do dominio fechado e sem dependencia de I/O.
2. Regras do enunciado representadas em validacoes puras.

---

## Fase 3 - Nucleo Matematico Puro

Dependencias: Fase 2.
Inicio: implementar metricas financeiras no projeto de analise.
Fim: modulo puro de avaliacao de carteira validado por testes.

Bibliotecas necessarias nesta fase:

1. `MathNet.Numerics.FSharp` para operacoes numericas e algebra linear em estilo F#.

Comando de instalacao:

```bash
dotnet add PortifolioAnalysis/PortifolioAnalysis.fsproj package MathNet.Numerics.FSharp
```

Passos:

1. Implementar retorno anualizado:
   1. $\mu = mean(r_p) \times 252$.
2. Implementar volatilidade anualizada:
   1. $\sigma = \sqrt{w^T C w} \times \sqrt{252}$.
3. Implementar Sharpe anualizado:
   1. $SR = \frac{\mu - r_{free}}{\sigma}$.
4. Definir estrategia para `r_free` (constante parametrizavel por CLI).
5. Garantir ausencia de side effects nesse modulo.

Checklist de saida:

1. Funcoes matematicas puras e deterministicas.
2. Interface clara para o simulador consumir.

---

## Fase 4 - Motor de Simulacao e Paralelismo

Dependencias: Fase 3.
Inicio: implementar geracao de combinacoes e avaliacao em lote.
Fim: pipeline paralelo funcional operando em chunks, sem explodir memoria.

Passos:

1. Gerar combinacoes de 20 ativos entre 30 (com estrategia de amostragem para execucao viavel).
2. Gerar pesos aleatorios validos:
   1. soma = 1.
   2. long-only.
   3. teto de 20% por ativo.
3. Montar pipeline:
   1. `map` para avaliar carteira.
   2. `filter` para validas.
   3. `maxBy` para melhor Sharpe.
4. Paralelizar avaliacao por lote com `Array.Parallel` ou `PSeq`.
5. Adicionar chunking e agregacao incremental para controle de memoria.

Checklist de saida:

1. Simulador retorna melhor carteira e metricas.
2. Execucao paralela segura (sem estado compartilhado mutavel no nucleo).

---

## Fase 5 - CLI (Ponto de Entrada)

Dependencias: Fase 1, Fase 3 e Fase 4.
Inicio: conectar bibliotecas ao app console.
Fim: comando unico executa pipeline completo e imprime resultado final.

Bibliotecas necessarias nesta fase:

1. `Argu` para parsing de argumentos de linha de comando no app console.

Comando de instalacao:

```bash
dotnet add Main/Main.fsproj package Argu
```

Passos:

1. Definir argumentos de linha de comando:
   1. periodo de dados.
   2. taxa livre de risco.
   3. numero de simulacoes por lote.
   4. modo serial/paralelo.
2. Implementar orquestrador no `Program.fs`:
   1. carregar dataset.
   2. rodar simulacao.
   3. exibir carteira vencedora.
3. Exibir saida final com:
   1. ativos selecionados.
   2. pesos.
   3. $\mu$, $\sigma$, Sharpe.
   4. duracao total de execucao.

Checklist de saida:

1. CLI funcional de ponta a ponta.
2. Mensagens de erro claras para dados invalidos/falhas externas.

---

## Fase 6 - Testes Unitarios (Obrigatorios)

Dependencias: Fase 3, Fase 4 e Fase 5.
Inicio: criar base de testes automatizados.
Fim: cobertura minima das regras de dominio e pipeline principal.

Bibliotecas necessarias nesta fase:

1. `FsUnit.Xunit` para assercoes mais idiomaticas em F# sobre xUnit.

Comando de instalacao:

```bash
dotnet add PortfolioAnalysis.Tests/PortfolioAnalysis.Tests.fsproj package FsUnit.Xunit
```

Passos:

1. Testes unitarios de dominio:
   1. validacao de pesos.
   2. formulas de retorno/volatilidade/sharpe.
2. Testes de integracao leve:
   1. dataset fixture -> simulador -> carteira resultado.

Checklist de saida:

1. Testes passando em `dotnet test`.
2. Regressao basica coberta para evoluir com seguranca.

---

## Fase 7 - Documentacao e Entrega (Nucleo Obrigatorio)

Dependencias: Fases 0 a 6.
Inicio: consolidar uso e resultados.
Fim: repositorio pronto para submissao.

Passos:

1. Atualizar README com:
   1. visao geral da arquitetura.
   2. instrucoes de setup.
   3. comandos de build/test/run.
   4. explicacao do pipeline funcional.
2. Incluir secoes de resultados:
   1. melhor carteira encontrada.
   2. limitacoes e proximos passos.
3. Revisar criterios do enunciado e marcar conformidade item a item.

Checklist de saida:

1. Projeto executavel por terceiros.
2. Entrega alinhada com rubrica e requisitos obrigatorios.

---

## Fase 8 - Obtencao de Dados via API (Bonus)

Dependencias: Fase 2 (pode iniciar antes), recomendada apos Fase 7.
Inicio: implementar cliente de dados externo em biblioteca dedicada.
Fim: dataset real de 30 ativos no periodo 01/07/2025 a 31/12/2025 disponivel em formato padrao.

Fonte obrigatoria de dados nesta fase:

1. API Yahoo Finance via endpoint base `https://query1.finance.yahoo.com/v8/finance/`.

Bibliotecas necessarias nesta fase:

1. `FSharp.Data` para consumo de dados externos e manipulacao de CSV/JSON.

Comando de instalacao:

```bash
dotnet add PortfolioAnalysis.Data/PortfolioAnalysis.Data.fsproj package FSharp.Data
```

Passos:

1. Criar modulo de API com funcoes impuras encapsuladas (`Async<Result<...>>`).
2. Usar obrigatoriamente a Yahoo Finance com endpoint base `https://query1.finance.yahoo.com/v8/finance/`.
3. Implementar fluxo:
   1. baixar historico de preco ajustado para 30 ativos do Dow Jones.
   2. normalizar datas.
   3. tratar faltas/inconsistencias.
4. Converter preco em retorno diario (simples ou log, documentando escolha).
5. Adicionar cache local (CSV/JSON) para reproducibilidade e menor custo de API.

Checklist de saida:

1. Processo repetivel para gerar dataset real.
2. Erros de API convertidos para `DataError`.

---

## Fase 9 - Testes Comparativos Serial vs Paralelo (Bonus)

Dependencias: Fase 4, Fase 5 e Fase 8 (se usar dados reais).
Inicio: medir desempenho do motor em cenario controlado.
Fim: relatorio comparativo com no minimo 5 execucoes por modo.

Passos:

1. Definir carga de entrada fixa para comparacao justa.
2. Rodar versao serial 5 vezes.
3. Rodar versao paralela 5 vezes.
4. Consolidar tempo medio, desvio e speedup.
5. Registrar resultados no README.

Checklist de saida:

1. Evidencia objetiva de ganho (ou nao) com paralelismo.
2. Metodologia de medicao documentada.

---

## Dependencias entre Fases (resumo)

1. Fase 0 -> Fase 1 -> Fase 2.
2. Fase 3 depende da Fase 2.
3. Fase 4 depende da Fase 3.
4. Fase 5 depende de Fase 1 + Fase 3 + Fase 4.
5. Fase 6 depende de Fase 3 + Fase 4 + Fase 5.
6. Fase 7 depende de Fases 0 a 6.
7. Fase 8 (bonus) depende de Fase 2, recomendada apos Fase 7.
8. Fase 9 (bonus) depende de Fase 4 + Fase 5 e opcionalmente Fase 8.