Com base na **Pedra de Roseta** e na base já filtrada `a1_physics_informed.csv` (`status_operacao == 1`), os próximos passos do desenvolvimento da linha _Physics-Based_ podem seguir um fluxo lógico que mantenha **alinhamento conceitual** e **rastreabilidade** para quando formos integrar com o _ML/DL_.

---

## **1. Diagnóstico inicial da base**

**Objetivo:** confirmar se todos os insumos para cálculo dos proxies e índices Wr/Wm normalizados estão presentes.

**Ações:**

- Verificar presença e integridade das variáveis primárias:  
    `flw_total_a_t_h`, `flw_total_b_t_h`, `flw_total_c_t_h`  
    (necessárias para `v_proxy`)
    
- Verificar presença de ΔP’s regionais (para `delta_proxy`).
    
- Identificar variáveis térmicas regionais (`*_adegc`) para `tau_densa`, `tau_diluida`, `tau_backpass`.
    
- Checar existência (ou ausência) de `Wr`, `Wr_ref`, `Wm`, `Wm_ref`.
    

**Saída:**  
Relatório de disponibilidade e ranges das variáveis.

---

## **2. Construção dos Proxies Físicos**

**Objetivo:** transformar as medições brutas em proxies normalizados.

**Proxies mínimos:**

1. **`v_proxy`** → média z-score de fluxos primários (A/B/C).
    
2. **`delta_proxy`** → média z-score dos ΔP representativos da cama.
    
3. **`tau_*`** → médias z-score das temperaturas por região.
    
    - Se não for possível separar por região, criar `tau_global`.
        

**Saída:**  
Nova versão do `a1_physics_informed.csv` contendo colunas `v_proxy`, `delta_proxy` e `tau_*`.

---

## **3. Cálculo dos índices Wr/Wm**

**Objetivo:** gerar `Wr_idx` e `Wm_idx` conforme Pedra de Roseta.

**Ações:**

- Se `Wr` e `Wr_ref` existirem → `Wr_idx = Wr / Wr_ref`
    
- Se `Wm` e `Wm_ref` existirem → `Wm_idx = Wm / Wm_ref`
    
- Caso as referências estejam ausentes → criar campos `Wr_idx` e `Wm_idx` preenchidos com `NaN` até integração futura.
    

**Saída:**  
Dataset enriquecido com índices normalizados (`Wr_idx`, `Wm_idx`).

---

## **4. Análise de correlação e envelopes operacionais**

**Objetivo:** entender como os proxies se relacionam entre si e com Wr/Wm.

**Ações:**

- Calcular correlação de Pearson/Spearman entre proxies e índices.
    
- Construir envelopes operacionais (média ± 2σ ou quantis 5%–95%).
    
- Identificar períodos fora do envelope.
    

**Saída:**  
Tabelas de correlação + gráficos de envelopes.

---

## **5. Estabelecimento do baseline físico**

**Objetivo:** criar referência operacional que sirva de comparação no futuro.

**Ações:**

- Selecionar períodos de operação estável.
    
- Calcular valores médios dos proxies e índices.
    
- Salvar baseline como tabela separada (`physics_baseline.csv`).
    

**Saída:**  
Baseline físico documentado.

---

## **6. Artefatos finais e integração**

**Objetivo:** garantir reuso e integração com a linha ML/DL.

**Ações:**

- Salvar dataset final enriquecido (`a1_physics_informed_enriched.csv`).
    
- Salvar baseline físico.
    
- Documentar processo e variáveis derivadas.