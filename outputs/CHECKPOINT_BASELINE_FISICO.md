# CHECKPOINT – Baseline Físico Oficial (Intersection_Longest)

**Data/Hora de geração:** 2025-08-11 17:21:33

## Decisão Congelada
Adota-se como **Baseline Físico Oficial** a janela **Intersection_Longest**, definida pela interseção de:
- τ estável (densa, diluída, backpass, global)
- Proxy de carvão no forno: **Flow C** (`coal_flow_furnace_t_h = flw_total_c_t_h`)
- Ar total medido (`air_total_nm3_h`) e partições primário/ secundário
- Gate de O₂: **ATIVADO** (conforme rodada que validou a janela)

**Período oficial:** 2024-03-04 20:00:00 → 2024-03-12 15:00:00  
**Tamanho da janela:** 188 amostras

## Arquivos Gerados
- **Máscara:** `C:\Users\wilso\MBA_EMPREENDEDORISMO\3AGD\A1_LOCAL\outputs\baseline_mask.csv` (Timestamp, in_baseline)
- **Dataset baseline:** `C:\Users\wilso\MBA_EMPREENDEDORISMO\3AGD\A1_LOCAL\outputs\baseline_datasets\physics_baseline_proxies.csv`
- **Dataset off-baseline:** `C:\Users\wilso\MBA_EMPREENDEDORISMO\3AGD\A1_LOCAL\outputs\baseline_datasets\physics_offbaseline_proxies.csv`
- **Estatísticas baseline:** `C:\Users\wilso\MBA_EMPREENDEDORISMO\3AGD\A1_LOCAL\outputs\baseline_datasets\stats_baseline.csv`
- **Estatísticas off-baseline:** `C:\Users\wilso\MBA_EMPREENDEDORISMO\3AGD\A1_LOCAL\outputs\baseline_datasets\stats_offbaseline.csv`
- **Comparativo (médias):** `C:\Users\wilso\MBA_EMPREENDEDORISMO\3AGD\A1_LOCAL\outputs\baseline_datasets\compare_baseline_vs_off.csv`

## Observações Operacionais
- `Intersection_Longest` será usado como **referência de patamar** para controle e para cálculo dos índices adimensionais **Wr/Wr_ref** e **Wm/Wm_ref**.
- Qualquer reprocessamento futuro deve **reutilizar esta máscara** para garantir reprodutibilidade.
- Proxies de ar e carvão são **medidos/derivados sem somas encadeadas** (Flow C é o único proxy válido para carvão no forno).

## Próximos Passos
1. Reprocessar “indicadores de desvio” (z‑scores/índices adimensionais) tomando o baseline como referência.
2. Instrumentar alarmes de desvio persistente vs. baseline (governança).
3. Opcional: treinar modelos para prever **Δ variável** em relação ao baseline (correção/otimização).
