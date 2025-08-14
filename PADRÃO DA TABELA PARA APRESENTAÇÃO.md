# Modelo de Tabela — Cálculo de Wr/Wm e Índices Normalizados

Esta tabela consolida **variáveis primárias**, **variáveis secundárias** e **saídas do modelo** para cada combinação `timestamp × zona × componente`.  
O objetivo é registrar os valores absolutos de desgaste (**Wr**, **Wm**) e seus índices relativos (**Wr_idx**, **Wm_idx**) normalizados pelo **baseline** (`Wr_ref`, `Wm_ref`).

---

| timestamp | zona | componente | flw_total_c_t_h | total_air_flow_knm3_h | total_paf_air_flow_knm3_h | te_of_hot_pri_air_in_aph_outl_adegc | delta_proxy | tau_densa | tau_diluida | tau_backpass | o2_excess_pct | Wr | Wm | Wr_ref | Wm_ref | Wr_idx | Wm_idx |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Data/hora da medição | Zona térmica/escoamento | Material/peça associada | Vazão carvão – grupo C (t/h) | Vazão total de ar (kNm³/h) | Vazão ar primário (kNm³/h) | Temp. ar primário (°C) | Proxy ΔP cama (adim.) | Temp. zona densa (°C) | Temp. zona diluída (°C) | Temp. backpass (°C) | Oxigênio em excesso (%) | Desgaste refratário (abs.) | Desgaste metálico (abs.) | Referência Wr (baseline) | Referência Wm (baseline) | Wr/Wr_ref (adim.) | Wm/Wm_ref (adim.) |

---

## Definições

### Variáveis primárias (operação direta)
- **`flw_total_c_t_h`** — Vazão total de carvão – grupo C (t/h)  
- **`total_air_flow_knm3_h`** — Vazão total de ar (kNm³/h)  
- **`total_paf_air_flow_knm3_h`** — Vazão de ar primário (kNm³/h)  
- **`te_of_hot_pri_air_in_aph_outl_adegc`** — Temperatura do ar primário quente na saída do APH (°C)

### Variáveis secundárias (derivadas/proxies do processo)
- **`delta_proxy`** — Proxy de ΔP representativo da cama (adimensional)  
- **`tau_densa`** — Temperatura média da zona densa (°C)  
- **`tau_diluida`** — Temperatura média da zona diluída (°C)  
- **`tau_backpass`** — Temperatura média do backpass (°C)  
- **`o2_excess_pct`** — Oxigênio em excesso (%)

### Saídas do modelo e referências
- **`Wr`**, **`Wm`** — Desgastes absolutos calculados (conforme equacionamento da Pedra de Roseta)  
- **`Wr_ref`**, **`Wm_ref`** — Referências calculadas **dentro do baseline** para cada `zona`/`componente`  
- **`Wr_idx`**, **`Wm_idx`** — Índices normalizados:  
  - `Wr_idx = Wr / Wr_ref`  
  - `Wm_idx = Wm / Wm_ref`  

---

## Observações
1. Todos os registros do **período baseline** alimentam o cálculo de `Wr_ref` e `Wm_ref`.  
2. Os índices (`Wr_idx`, `Wm_idx`) representam a **variação temporal relativa** de cada componente em relação ao seu valor de referência.  
3. O modelo permite identificar **desvios operacionais significativos** e antecipar intervenções de manutenção.
![[a1_physics_based_development.ipynb]]