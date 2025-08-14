# Conceito de Cálculo e Normalização de Wr e Wm no Projeto A1

## 1. Objetivo
Estabelecer o método para cálculo de **Wr** (desgaste do refratário) e **Wm** (desgaste do metálico) ao longo de todo o período de dados disponíveis, e normalizá-los com base em um **período de referência (baseline)** previamente definido.

---

## 2. Etapas do Processo

### 2.1 Cálculo de Wr e Wm (absolutos)
- Utilizar o equacionamento físico definido no documento **"Pedra de Roseta – Conceito Wr & Wm"**.
- Os cálculos de Wr e Wm serão feitos para **todos os instantes** da série temporal, a partir das **variáveis secundárias** (derivadas das primárias) obtidas dos sensores.

### 2.2 Determinação de Wr_ref e Wm_ref (referências)
- No **período baseline operacional** já estabelecido, extrair os valores de Wr e Wm calculados.
- Determinar **Wr_ref** e **Wm_ref** como os valores de referência desse período (média, mediana ou outro critério acordado).
- Wr_ref e Wm_ref representam a condição considerada **normal/saudável** da caldeira.

### 2.3 Normalização para índices relativos
- Para **todo o período** da série:
  - **Wr_idx = Wr / Wr_ref**
  - **Wm_idx = Wm / Wm_ref**
- Os índices Wr_idx e Wm_idx representam a **variação temporal relativa** do desgaste em relação ao baseline.

---

## 3. Estrutura da Tabela Final de Saída

Cada linha conterá:
1. **Variáveis primárias** (medidas diretas de operação)  
   Ex.: `flw_total_c_t_h`, `air_total_nm3_h`, `air_primary_nm3_h`, temperatura do ar primário.
   
2. **Variáveis secundárias** (proxies derivadas)  
   Ex.: `delta_proxy`, `tau_densa`, `tau_diluida`, `tau_backpass`, `o2_excess_pct`.
   
3. **Wr e Wm absolutos** (calculados pelo modelo físico).

4. **Wr_idx e Wm_idx** (valores relativos normalizados pelo baseline).

---

## 4. Interpretação
- **Wr_idx > 1** ou **Wm_idx > 1** → desgaste acima da condição de referência.
- **Wr_idx < 1** ou **Wm_idx < 1** → desgaste abaixo da condição de referência.
- A variação temporal desses índices será usada para análises operacionais e preditivas.

---

## 5. Aplicação
Esse método permitirá:
- Monitorar o desgaste em tempo real com relação a uma condição de referência validada.
- Comparar diferentes períodos de operação.
- Fornecer subsídios para ações preventivas e de manutenção.

