# Qualidade do ar em Pequim: estimativa de PM2.5 e perfis de poluição

[![Abrir no Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/douglasarend/sistematizacao-ciencia-dados-ii/blob/main/Sistematizacao_CD2.ipynb)

Projeto acadêmico da disciplina **Ciência de Dados II** que aplica o processo
de KDD a dados horários de qualidade do ar. O Apache Spark foi utilizado na
ingestão, preparação, análise exploratória, modelagem supervisionada e
clusterização.

## Identificação

| Campo | Informação |
|---|---|
| Autor | Douglas Arend Leão |
| Registro acadêmico | 72500862 |
| Instituição | CEUB |
| Curso | Banco de Dados |
| Disciplina | Ciência de Dados II |
| Turma | B - 0726 |
| Professor | Romes Heriberto Pires de Araujo |

## Objetivos

O projeto busca:

- investigar padrões temporais e espaciais nas concentrações de PM2.5;
- comparar modelos para estimar PM2.5 a partir de outras medições disponíveis
  no mesmo horário;
- identificar perfis recorrentes de poluição e condições meteorológicas com
  K-Means.

A modelagem supervisionada realiza uma **estimativa contemporânea**, e não uma
previsão para horas ou dias futuros.

## Dataset

Foi utilizado o conjunto
[Beijing Multi-Site Air Quality](https://archive.ics.uci.edu/dataset/501/beijing+multi+site+air+quality+data),
disponibilizado pelo UCI Machine Learning Repository.

- Período: 1º de março de 2013 a 28 de fevereiro de 2017
- Registros horários: 420.768
- Estações de monitoramento: 12
- Colunas originais: 18
- Variável-alvo da regressão: PM2.5
- Registros com PM2.5 observado: 412.029

O notebook baixa automaticamente os arquivos da fonte oficial. O arquivo ZIP
utilizado na execução registrada apresentou o SHA-256
`b04da438b2f331ac0ffd45aebdfec0d20d2367feb5f6948c4b1f7ce1191e33c4`.

Referência:

> Chen, S. (2017). *Beijing Multi-Site Air Quality* [Dataset].  
> UCI Machine Learning Repository. https://doi.org/10.24432/C5RK5G

## Tecnologias

- Python
- Apache Spark e PySpark 4.0.1
- Spark SQL
- Spark MLlib
- pandas
- Matplotlib
- Google Colab
- OpenJDK

A execução registrada utilizou Python 3.13.15, OpenJDK 21 e Spark 4.0.1.

## Metodologia

O trabalho foi organizado segundo etapas do processo de KDD:

1. **Seleção e integração:** download e ingestão dos 12 arquivos das estações.
2. **Pré-processamento:** verificação de duplicatas, datas, chaves, valores
   inválidos e ausentes; padronização da direção do vento; criação de atributos
   temporais e cíclicos.
3. **Análise exploratória:** seis consultas Spark SQL sobre estações, meses,
   horários, velocidade do vento, anos completos e disponibilidade dos dados.
4. **Modelagem supervisionada:** comparação entre referência pela média,
   Regressão Linear regularizada e Random Forest, com divisão temporal entre
   treinamento, validação e teste.
5. **Modelagem descritiva:** K-Means com imputação pela mediana, padronização e
   seleção de K pelo coeficiente de silhueta.
6. **Interpretação:** síntese dos resultados, recomendações e limitações.

Na regressão, os períodos foram separados da seguinte forma:

| Conjunto | Período | Registros |
|---|---|---:|
| Treinamento | mar/2013 a fev/2015 | 205.989 |
| Validação | mar/2015 a fev/2016 | 103.287 |
| Teste | mar/2016 a fev/2017 | 102.753 |

A imputação e o restante do pré-processamento supervisionado foram ajustados
somente no treinamento. A semente aleatória utilizada foi 42.

## Principais resultados

### Análise exploratória

- Dongsi apresentou a maior média de PM2.5: **86,19 µg/m³**.
- Dingling apresentou a menor média entre as estações: **65,99 µg/m³**.
- Dezembro teve a maior média mensal histórica: **104,58 µg/m³**.
- Agosto teve a menor média mensal histórica: **53,47 µg/m³**.
- A maior média horária ocorreu às 22h: **88,89 µg/m³**.
- A menor média horária ocorreu às 7h: **73,27 µg/m³**.
- A média foi de **102,30 µg/m³** com vento abaixo de 1 m/s e de
  **36,16 µg/m³** com vento igual ou superior a 3 m/s.
- A média anual passou de **85,58 µg/m³ em 2014** para
  **71,93 µg/m³ em 2016**.

Essas comparações são descritivas e não demonstram causalidade.

### Modelagem supervisionada

| Modelo e conjunto | RMSE | MAE | R² | Previsões negativas |
|---|---:|---:|---:|---:|
| Referência — validação | 82,6044 | 61,0146 | -0,0177 | 0,00% |
| Regressão Linear — validação | 30,6096 | 19,7523 | 0,8603 | 9,44% |
| Random Forest — validação | **29,3306** | **16,3009** | **0,8717** | **0,00%** |
| Random Forest — teste | **26,6613** | **16,2067** | **0,8962** | **0,00%** |

A Random Forest foi escolhida exclusivamente pelo desempenho na validação. No
teste temporal reservado, seu RMSE foi 9,10% inferior ao da validação.

### Clusterização com K-Means

A seleção de K avaliou valores entre 2 e 6 em uma amostra reprodutível de
105.282 registros. O melhor resultado foi obtido com **K = 2**:

- silhueta na amostra: **0,4641**;
- silhueta na base completa: **0,3683**.

| Perfil | Registros | Participação | Características principais |
|---|---:|---:|---|
| Grupo 1 | 200.631 | 47,68% | PM2.5, PM10, SO₂, NO₂ e CO mais altos; menor temperatura e O₃ |
| Grupo 2 | 220.137 | 52,32% | Menores níveis dos poluentes anteriores; maior temperatura e O₃ |

O PM2.5 médio foi de **100,80 µg/m³** no Grupo 1 e **60,43 µg/m³**
no Grupo 2. Todas as estações apresentaram registros nos dois grupos,
indicando que os clusters não representam apenas uma divisão geográfica.

## Arquivos do projeto

```text
.
├── README.md
├── Sistematizacao_CD2.ipynb
└── SISTEMATIZACAO_CD2_Douglas_Arend_Leao.pdf
```

O dataset original não é versionado no repositório, pois o notebook o obtém
diretamente da UCI.

## Como executar

A forma recomendada é pelo Google Colab:

1. Abra o notebook pelo botão no início deste README.
2. Mantenha o acesso à internet habilitado para instalar o PySpark e baixar o
   dataset.
3. Execute as células em ordem, usando **Ambiente de execução → Executar tudo**.
4. Aguarde o término das etapas de treinamento antes de prosseguir.
5. Consulte os gráficos, tabelas e interpretações no próprio notebook.

Os arquivos gerados durante a execução ficam em `/content/resultados_cd2`.
Como esse armazenamento é temporário, os resultados desejados devem ser
baixados antes do encerramento da sessão do Colab.

O notebook foi estruturado para os caminhos do Colab. Uma execução local pode
exigir a adaptação dos caminhos e a instalação prévia de Python, Java, Jupyter,
PySpark 4.0.1, pandas e Matplotlib.

## Limitações

- Os dados são históricos e pertencem a uma única região.
- As associações exploratórias não comprovam relações de causa e efeito.
- O modelo estima PM2.5 com medições auxiliares do mesmo horário; não realiza
  previsão futura.
- O desempenho não está validado para outras cidades ou períodos.
- A análise exploratória examinou toda a base antes da avaliação preditiva; o
  teste temporal não equivale a uma validação externa inédita.
- K = 2 foi o melhor resultado apenas entre os valores de K avaliados.
- A silhueta final indica sobreposição entre os grupos.
- Os clusters não são categorias oficiais de qualidade do ar ou de risco à
  saúde.

## Links

- [Notebook principal](./Sistematizacao_CD2.ipynb)
- [Relatório final](./SISTEMATIZACAO_CD2_Douglas_Arend_Leao.pdf)
- [Dataset no UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/501/beijing+multi+site+air+quality+data)
- [DOI do dataset](https://doi.org/10.24432/C5RK5G)
- [Repositório do projeto](https://github.com/douglasarend/sistematizacao-ciencia-dados-ii)
- [Perfil do autor](https://github.com/douglasarend)
