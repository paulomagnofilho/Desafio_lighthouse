# Desafio Lighthouse | Manutenção preditiva

Projeto de ciência de dados desenvolvido para o processo seletivo do programa Lighthouse, da Indicium. A tarefa era usar dados de sensores do processo de manufatura para identificar quais registros indicam uma falha e prever o tipo de falha. A entrega reúne análise exploratória, dois classificadores e um CSV com uma previsão por linha da base de teste.

## Base de dados

O desafio forneceu uma base de treino com **6.667 registros e 9 colunas**, incluindo a variável-alvo `failure_type`, e uma base de teste com **3.333 registros e 8 colunas**, sem o alvo. As entradas são os identificadores `udi` e `product_id`, a categoria de qualidade `type` (L, M ou H), as temperaturas do ar e do processo (K), a rotação (rpm), o torque (N·m) e o desgaste da ferramenta (min).

O alvo tem seis classes. Na base de treino, há 6.435 registros sem falha (`No Failure`) e 232 com falha (**3,48%**), distribuídos entre dissipação de calor (75), potência (63), sobrecarga (52), desgaste da ferramenta (30) e falhas aleatórias (12).

## Metodologia

1. **Análise exploratória.** O notebook verifica valores ausentes, compara distribuições e taxas de falha por categoria de qualidade, aplica um teste de qui-quadrado e explora relações entre variáveis com histogramas, boxplots, dispersões e uma matriz de correlação.
2. **Preparação.** Os identificadores são excluídos dos preditores. A qualidade é codificada em duas colunas binárias com one-hot encoding. O alvo original é codificado numericamente e também convertido em um alvo binário (0 para sem falha e 1 para qualquer falha).
3. **Detecção de falhas.** A etapa binária compara Random Forest e LightGBM com busca de hiperparâmetros e prioriza o **recall**, a proporção de falhas reais identificadas. O notebook aplica SMOTE para aumentar a representação da classe de falha.
4. **Classificação do tipo de falha.** A segunda etapa é treinada apenas com registros rotulados como falha e compara Random Forest, LightGBM e SVM. O critério é o **F1 macro**, a média simples do F1 de cada classe, que dá o mesmo peso aos tipos de falha independentemente de sua frequência.
5. **Previsão.** O fluxo final usa LightGBM nas duas etapas. A primeira atribui `No Failure` aos casos negativos. A segunda classifica apenas os casos sinalizados como falha. O resultado é exportado com as colunas `rowNumber` (índice da linha, começando em zero) e `predictedValues` (classe prevista).

O notebook usa Python e Jupyter, pandas e NumPy para manipulação de dados, Matplotlib, Seaborn e mlxtend para visualização, SciPy para análise estatística, scikit-learn, imbalanced-learn e LightGBM para modelagem e Pickle para serialização.

## Resultados e interpretação

- Na base de treino, a taxa de falha foi de **4,00%** na categoria L, **2,72%** na M e **2,58%** na H. Rotação e torque foram os preditores mais importantes no Random Forest da etapa binária. Nos gráficos exploratórios, os registros de falha de potência aparecem nos extremos da relação entre essas duas variáveis.
- Na validação da **etapa binária**, o LightGBM registrou **98,17% de recall** e **98,02% de precisão**. O SMOTE foi aplicado antes da divisão entre treino e validação, o que pode inflar essas métricas.
- Na validação da **classificação do tipo**, restrita a registros com falha real, o LightGBM obteve **89,46% de F1 macro**, ante **79,80%** do SVM e **67,08%** do Random Forest.
- Ao aplicar o **fluxo completo à base de treino original**, o notebook registrou **85,85% de F1 macro**, com precisão macro de **84%** e recall macro de **89%** no relatório arredondado. Essas médias incluem a classe `No Failure`. O recall para falhas aleatórias foi de **58%**, com apenas 12 ocorrências dessa classe na base. Como o conjunto de teste foi fornecido sem rótulos, o repositório apresenta apenas as previsões para essa base.

Em um cenário de manutenção, o fluxo pode apoiar a triagem de registros para inspeção ao sinalizar possíveis falhas e indicar o tipo provável. As métricas do desafio descrevem os experimentos do notebook e a avaliação em dados independentes seria necessária para estimar o desempenho em operação.

## Arquivos e visualização

| Arquivo | Conteúdo |
| --- | --- |
| [Enunciado do desafio](%5BLighthouse%5D%20Desafio%20Cientista%20de%20Dados%20-%20Manuten%C3%A7%C3%A3o%20preventiva.docx) | Contexto, descrição dos dados e formato de entrega |
| [Analise_Modelagem.ipynb](Analise_Modelagem.ipynb) | Análise exploratória, gráficos, matrizes de confusão, treinamento e exportação das previsões |
| [Base de treino](desafio_manutencao_preditiva_treino.csv) e [base de teste](desafio_manutencao_preditiva_teste.csv) | Dados fornecidos no desafio |
| [predicted.csv](predicted.csv) | Previsões para os 3.333 registros da base de teste |
| `detect_failure.pkl` e `detect_type_failure.pkl` | Classificadores de detecção e de tipo de falha |
| `label_failure.pkl`, `function_transform_type.pkl` e `function_failure_prediction.pkl` | Codificador de classes, função de transformação da qualidade e função que encadeia as previsões |
| [requirements.txt](requirements.txt) | Registro das dependências do ambiente original |

Os gráficos e as saídas salvas podem ser consultados diretamente no notebook pelo GitHub. Para executar a análise, abra o notebook no Jupyter a partir da raiz do repositório e execute as células em ordem em um ambiente compatível com Python 3.9.12, versão registrada no notebook. O `requirements.txt` contém referências a caminhos locais do ambiente original e serve como referência para configurar as dependências.
