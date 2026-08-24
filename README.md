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
