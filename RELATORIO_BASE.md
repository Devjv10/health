# Relatório Técnico — HealthSearch

**Instituição:** UNIPÊ — Centro Universitário de João Pessoa  
**Disciplina:** Tendências em Ciência da Computação  
**Professor:** Me. Ricardo Roberto de Lima  
**Integrantes:** [PREENCHER]

## 1. Objetivo e arquitetura

O HealthSearch é um protótipo de motor de busca híbrido aplicado a um corpus fixo de seis diretrizes médicas. A solução executa dois mecanismos em paralelo: busca léxica com Okapi BM25 e busca semântica por embeddings. Em seguida, o Reciprocal Rank Fusion (RRF) combina as posições dos dois rankings. Como desafio bônus, um Cross-Encoder reavalia somente os três primeiros candidatos do RRF.

Fluxo da solução:

`Consulta → BM25 + Embeddings/Cosseno → RRF → Cross-Encoder Top-3 (opcional)`

## 2. Pré-processamento e mecanismos de busca

Antes da indexação léxica, os documentos e a consulta são convertidos para minúsculas, têm caracteres especiais removidos, são tokenizados e passam pela remoção de stopwords em português. Hífens e caracteres Unicode são preservados para não destruir códigos médicos. Assim, `CÓD-ECG-12D` permanece como o token único `cód-ecg-12d`.

O BM25 foi implementado com `BM25Okapi`. O parâmetro `k1`, ajustável entre 0,0 e 3,0, controla a saturação da frequência dos termos. O parâmetro `b`, ajustável entre 0,0 e 1,0, controla a normalização pelo comprimento dos documentos. O índice é recalculado quando esses valores são alterados.

Na busca semântica, o modelo multilíngue `intfloat/multilingual-e5-small` gera embeddings densos da consulta e dos documentos. A relevância é calculada pela similaridade de cosseno, permitindo recuperar conceitos relacionados mesmo quando não possuem as mesmas palavras.

Os rankings são combinados pela fórmula definida no enunciado, com ranks iniciados em 1 e constante `k_RRF = 60`:

`Score_RRF(D) = α/(60 + Rank_BM25) + (1 − α)/(60 + Rank_Semântico)`

O parâmetro `α` varia entre 0 e 1. Valores próximos de 1 favorecem o BM25, valores próximos de 0 favorecem a busca semântica e `α = 0,5` equilibra os dois métodos.

## 3. Interface e desafio bônus

A interface Streamlit contém controles para `k1`, `b`, `α` e ativação do Cross-Encoder. Os resultados são organizados nas abas Busca Léxica, Busca Semântica, Híbrido RRF, Matriz Comparativa e Bônus. Cada tabela apresenta documento, título, posição e score correspondente.

No bônus, o modelo `cross-encoder/ms-marco-MiniLM-L-6-v2` recebe somente os pares formados pela consulta e pelos Top-3 documentos do RRF. A interface compara a posição original, a nova posição e os scores RRF e Cross-Encoder.

## 4. Resultados dos testes

Na consulta exata `CÓD-ECG-12D`, o BM25 colocou os dois documentos que contêm o código nas primeiras posições. A matriz obtida com `k1 = 1,2`, `b = 0,75` e `α = 0,5` foi:

| Documento | Rank BM25 | Rank Semântico | Rank RRF |
|---|---:|---:|---:|
| Doc 1 — Protocolo Emergência ECG | 2 | 1 | 1 |
| Doc 6 — Procedimentos de UTI Geral | 1 | 2 | 2 |
| Doc 2 — Guia de Farmacologia Cardíaca | 3 | 5 | 3 |
| Doc 5 — Protocolo de Reanimação RCR | 6 | 3 | 4 |
| Doc 4 — Manual de AVC Isquêmico | 5 | 4 | 5 |
| Doc 3 — Diretriz de Hipertensão Arterial | 4 | 6 | 6 |

O Cross-Encoder alterou o Top-3 da consulta: Doc 6 passou da 2ª para a 1ª posição, Doc 1 passou da 1ª para a 2ª e Doc 2 permaneceu em 3º. Nas consultas conceituais, `ataque cardíaco` recuperou o Doc 2 em primeiro no ranking semântico, enquanto `pressão alta` recuperou o Doc 3 em primeiro.

Também foram validados: preservação do código médico, remoção de stopwords, funcionamento dos sliders, cálculo de cosseno, fórmula RRF, ranks iniciados em 1, re-ranking limitado a três candidatos e inicialização do servidor Streamlit, cujo endpoint de saúde respondeu `ok`.

## 5. Conclusão

Os testes evidenciam a complementaridade dos métodos. O BM25 oferece precisão para códigos e termos exatos; os embeddings ampliam a recuperação por contexto e sinônimos; e o RRF combina os rankings sem comparar diretamente scores de escalas diferentes. O Cross-Encoder acrescenta uma avaliação mais detalhada somente aos candidatos mais promissores, atendendo ao desafio bônus com baixo custo adicional.

## 6. Divisão de tarefas

**[PREENCHER COM OS NOMES DOS INTEGRANTES E AS RESPECTIVAS ATIVIDADES]**
