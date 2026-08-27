# 🚀 Módulo 2: Pipeline de Pré-processamento e Engenharia de Atributos para IDS

> **Autor:** Igor Gomes de Lima  
> **Contexto:** Trabalho de Conclusão de Curso (TCC) - Aplicação de *Foundation Models* em Sistemas de Detecção de Intrusão (IDS).

---

## 📌 1. Introdução e Objetivo
Este módulo descreve a construção de um pipeline de engenharia de dados de ponta a ponta a partir de tráfego de rede bruto (`.pcap`), realizando a conversão para fluxos estatísticos, análise comparativa com bases consagradas da literatura (CICIDS2017/UNB) e normalização matemática voltada para o consumo por modelos de inteligência artificial.

---

## 🛠️ 2. Ingestão e Limpeza da Base de Referência (CICFlowMeter)
Nesta etapa, realizamos a importação e o tratamento dos arquivos oficiais disponibilizados pela Universidade de Brasília (UNB) gerados pelo *CICFlowMeter*, servindo como nossa *baseline* de comparação.

* **Ações realizadas:**
  * Leitura e consolidação dos arquivos CSV de sexta-feira (`PortScan` e `DDOS`).
  * Limpeza de nomes de colunas e remoção de valores corrompidos (`inf` e `NaN`).
* **Resultado:** Base consolidada contendo **511.807 fluxos limpos** e **79 colunas** estruturais.

```python
import pandas as pd
import numpy as np

# Definindo o carregamento com tipagem otimizada para economizar memória
def carregar_dados(caminho):
    df = pd.read_csv(caminho, encoding="latin1", low_memory=True)
    df.columns = df.columns.str.strip()
    return df

# Carregando os dois arquivos de sexta-feira
df_cic1 = carregar_dados(r"C:\Users\Usuario\Videos\Dados\Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv")
df_cic2 = carregar_dados(r"C:\Users\Usuario\Videos\Dados\Friday-WorkingHours-Afternoon-DDoS.pcap_ISCX.csv")

# Consolidação
df_raw = pd.concat([df_cic1, df_cic2], ignore_index=True)

# Limpeza de valores corrompidos (Essencial para Foundation Models)
df_raw.replace([np.inf, -np.inf], np.nan, inplace=True)
df_raw.dropna(inplace=True)

print(f"Base consolidada com {len(df_raw):,} fluxos limpos.")
print(f"Colunas presentes: {df_raw.shape[1]}")


🔄 3. Extração Local de Atributos via NFStream (.pcap Bruto)
Diferente de apenas consumir planilhas prontas, realizamos o processamento local do arquivo de tráfego bruto de rede (Friday-WorkingHours.pcap) utilizando a ferramenta NFStream.

Ações realizadas:

Leitura direta do arquivo .pcap local de mais de 8 GB.

Configuração do motor de análise estatística profunda (statistical_analysis=True).

Extração e exportação automatizada do resultado para o arquivo nfstream_output.csv.

Resultado: Extração bem-sucedida de 571.835 fluxos e 86 colunas estatísticas geradas de forma autônoma.

from nfstream import NFStreamer
import time

pcap_path = r"C:\Users\Usuario\Videos\Dados\Friday-WorkingHours.pcap"

print("🔄 Iniciando a extração de fluxos com NFStream a partir do PCAP bruto...")
inicio = time.time()

streamer = NFStreamer(
    source=pcap_path,
    statistical_analysis=True,
    idle_timeout=15,
    active_timeout=120
)

df_nfstream = streamer.to_pandas()

caminho_csv_nf = r"C:\Users\Usuario\Videos\Dados\nfstream_output.csv"
df_nfstream.to_csv(caminho_csv_nf, index=False)

fim = time.time()
print(f"✅ Extração via NFStream concluída em {round((fim - inicio)/60, 2)} minutos!")
print(f"• Total de fluxos gerados pelo NFStream: {len(df_nfstream):,}")
print(f"• Total de colunas geradas pelo NFStream: {df_nfstream.shape[1]}")

📊 4. Análise Comparativa de Pipelines (CICFlowMeter vs. NFStream)
Para fins científicos do TCC, realizamos um cruzamento analítico para avaliar o comportamento de diferentes ferramentas de parsing de rede sobre o mesmo tráfego:

Dimensionalidade: O CICFlowMeter gerou 79 colunas, enquanto o NFStream gerou 86 colunas.

Volumetria: Pequenas variações de contagem devido aos critérios de timeout de inatividade de cada motor.

Métricas Estatísticas: Avaliação de métricas temporais (como a duração média dos fluxos).

# Recarregando o dataframe do NFStream direto do CSV salvo em disco
caminho_csv_nf = r"C:\Users\Usuario\Videos\Dados\nfstream_output.csv"
df_nfstream = pd.read_csv(caminho_csv_nf, low_memory=False)

df_nfstream.replace([np.inf, -np.inf], np.nan, inplace=True)
colunas_numericas_nf = df_nfstream.select_dtypes(include=[np.number]).columns
df_nfstream[colunas_numericas_nf] = df_nfstream[colunas_numericas_nf].fillna(0)

print("==================================================")
print("     ANÁLISE COMPARATIVA DE PIPELINES DE REDE     ")
print("==================================================")
print(f"1. Dimensionalidade (Estrutura de Colunas):")
print(f"   • CICFlowMeter (UNB): {df_raw.shape[1]} colunas")
print(f"   • NFStream (Local):   {df_nfstream.shape[1]} colunas")

print(f"\n2. Volumetria de Fluxos (Pós-Tratamento Seguro):")
print(f"   • CICFlowMeter (UNB): {len(df_raw):,} fluxos")
print(f"   • NFStream (Local):   {len(df_nfstream):,} fluxos")

duracao_cic = df_raw['Flow Duration'].mean() if 'Flow Duration' in df_raw.columns else 0
duracao_nf = df_nfstream['bidirectional_duration_ms'].mean() if 'bidirectional_duration_ms' in df_nfstream.columns else 0

print(f"\n3. Métrica Estatística - Duração Média:")
print(f"   • CICFlowMeter (Média): {round(duracao_cic, 2)}")
print(f"   • NFStream (Média):     {round(duracao_nf, 2)} ms")
print("==================================================")

📈 5. Visualização Gráfica Acadêmica
Geramos visualizações comparativas utilizando matplotlib e seaborn para documentar a distribuição do tráfego e a volumetria das extrações.

import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="whitegrid")
fig, axes = plt.subplots(1, 2, figsize=(16, 5))

comparativo_volumetria = pd.DataFrame({
    'Ferramenta': ['CICFlowMeter (UNB)', 'NFStream (Local)'],
    'Total de Fluxos': [len(df_raw), len(df_nfstream)]
})

sns.barplot(
    data=comparativo_volumetria, 
    x='Ferramenta', 
    y='Total de Fluxos', 
    hue='Ferramenta', 
    palette='viridis', 
    legend=False, 
    ax=axes[0]
)
axes[0].set_title('Comparação de Volumetria de Fluxos')

sns.histplot(
    df_nfstream['bidirectional_duration_ms'], 
    bins=50, 
    ax=axes[1], 
    color='salmon', 
    log_scale=True
)
axes[1].set_title('Distribuição de Duração (NFStream - Escala Log)')
axes[1].set_xlabel('Duração (ms)')

plt.tight_layout()
plt.show()

🧮 6. Normalização e Preparação para Foundation Models
Para garantir que o modelo de Deep Learning aprenda de forma justa sem ser dominado por atributos de escalas numéricas discrepantes:

Ações realizadas:

Filtragem estrita de colunas numéricas (total de 73 features).

Aplicação do StandardScaler (centralização com média 0 e desvio padrão unitário) em todos os registros do dataset.

Resultado: Matriz de dados limpa, normalizada e pronta para a etapa de treinamento de modelos de aprendizado profundo.

from sklearn.preprocessing import StandardScaler

cols_to_drop = ['src_ip', 'dst_ip', 'Label_Recriado'] if 'Label_Recriado' in df_nfstream.columns else []
features_nf = df_nfstream.drop(columns=cols_to_drop, errors='ignore').select_dtypes(include=[np.number])

scaler = StandardScaler()
X_scaled = scaler.fit_transform(features_nf)

df_scaled = pd.DataFrame(X_scaled, columns=features_nf.columns)


# 🚀 TCC: Application of Foundation Models in Intrusion Detection Systems (IDS)
## Módulo 2: Engenharia de Atributos, Captura de Tráfego e Rotulagem via NFStream

> **Autor:** Igor Gomes de Lima  
> **Repositório Pessoal / Módulo 2**

---

### 📂 Estrutura dos Notebooks do Módulo 2

O Módulo 2 foi estruturado de forma modular para cobrir todo o ciclo de vida dos dados de rede, desde a captura/processamento bruto até a validação comparativa com o dataset de referência (CICIDS2017/UNB):

1. **`modulo2_captura_nfstream.ipynb`**
   * *Objetivo:* Captura de tráfego de rede ao vivo da interface local ou processamento inicial via NFStream.
   
2. **`modulo2_nfstream_pcap_cicids2017.ipynb`**
   * *Objetivo:* Aplicação do NFStream sobre arquivos `.pcap` brutos (ex: tráfego de quinta-feira) gerando os fluxos bidirecionais e estatísticos.

3. **`modulo2_rotulagem.ipynb`** (Utiliza o script `rotulagem_cicids2017.py`)
   * *Objetivo:* Reconstrução autônoma da coluna `Label` utilizando as janelas de tempo, endereços IP de atacantes/vítimas e tratamento de NAT do firewall descritos na documentação oficial da UNB.

4. **`modulo2_analise_comparativa.ipynb`**
   * *Objetivo:* Cruzamento estatístico descritivo e de frequência entre a base gerada localmente (`NFStream + Rotulagem`) e os arquivos CSV originais da CICIDS2017, incluindo testes de aderência (Qui-Quadrado) e gráficos de distribuição.

print(f"✅ Normalização concluída!")
print(f"• Forma do dataset normalizado: {df_scaled.shape}")
print(f"• Exemplo das primeiras 5 linhas normalizadas:\n", df_scaled.head())
