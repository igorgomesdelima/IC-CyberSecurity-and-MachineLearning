🚀 Relatório Técnico: Pipeline de Pré-processamento e Engenharia de Atributos para IDS (Módulo 2)
Autor: Igor Gomes de Lima

Contexto: Trabalho de Conclusão de Curso (TCC) - Aplicação de Foundation Models em Sistemas de Detecção de Intrusão (IDS).

📌 1. Introdução e Objetivo
Este módulo tem como objetivo construir um pipeline de engenharia de dados de ponta a ponta a partir de tráfego de rede bruto (.pcap), realizando a conversão para fluxos estatísticos, análise comparativa com bases consagradas da literatura (CICIDS2017/UNB) e normalização matemática voltada para o consumo por modelos de inteligência artificial.

🛠️ 2. Ingestão e Limpeza da Base de Referência (CICFlowMeter)
Nesta etapa, realizamos a importação e o tratamento dos arquivos oficiais disponibilizados pela Universidade de Brasília (UNB) gerados pelo CICFlowMeter, servindo como nossa baseline de comparação.

Ações realizadas:

Leitura e consolidação dos arquivos CSV de sexta-feira (PortScan e DDOS).

Limpeza de nomes de colunas e remoção de valores corrompidos (inf e NaN).

Resultado: Base consolidada contendo 511.807 fluxos limpos e 79 colunas estruturais.

🔄 3. Extração Local de Atributos via NFStream (.pcap Bruto)
Diferente de apenas consumir planilhas prontas, realizamos o processamento local do arquivo de tráfego bruto de rede (Friday-WorkingHours.pcap) utilizando a ferramenta NFStream.

Ações realizadas:

Leitura direta do arquivo .pcap local de mais de 8 GB.

Configuração do motor de análise estatística profunda (statistical_analysis=True).

Extração e exportação automatizada do resultado para o arquivo nfstream_output.csv.

Resultado: Extração bem-sucedida de 571.835 fluxos e 86 colunas estatísticas geradas de forma autônoma.

📊 4. Análise Comparativa de Pipelines (CICFlowMeter vs. NFStream)
Para fins científicos do TCC, realizamos um cruzamento analítico para avaliar o comportamento de diferentes ferramentas de parsing de rede sobre o mesmo tráfego:

Dimensionalidade: O CICFlowMeter gerou 79 colunas, enquanto o NFStream gerou 86 colunas.

Volumetria: Pequenas variações de contagem devido aos critérios de timeout de inatividade de cada motor.

Métricas Estatísticas: Avaliação de métricas temporais (como a duração média dos fluxos).

📈 5. Visualização Gráfica Acadêmica
Geramos visualizações comparativas utilizando matplotlib e seaborn para documentar a distribuição do tráfego e a volumetria das extrações, garantindo clareza para o relatório final do TCC.

🧮 6. Normalização e Preparação para Foundation Models
Para garantir que o modelo de Deep Learning aprenda de forma justa sem ser dominado por atributos de escalas numéricas discrepantes:

Ações realizadas:

Filtragem estrita de colunas numéricas (total de 73 features).

Aplicação do StandardScaler (centralização com média 0 e desvio padrão unitário) em todos os registros do dataset.

Resultado: Matriz de dados limpa, normalizada e pronta para a etapa de treinamento de modelos de aprendizado profundo.
