
---

**Data:** 2025-08-10  
**Protocolo ativo:** PROTOCOLOS_UNIFICADOS_V2 — MODO ULTRA-HF-000  
**Referências:**

- Documento "Teoria da Combustão e Fenômenos de Desgaste em Caldeiras CFB"
    
- "Plano de Ataque A1"
    
- Histórico e checkpoints A1
    

---

## 1. Objetivo

Estabelecer a base técnica e o roteiro operacional para calcular e aplicar modelos de desgaste **Wr** (refratário) e **Wm** (metálico) em caldeiras CFB, visando:

1. **Estimativa** da taxa de desgaste por região (fase densa, diluída, backpass).
    
2. **Orientação operacional** para reduzir desgaste sem comprometer carga, eficiência ou emissões.
    
3. Disponibilizar **duas versões** do trabalho:
    
    - **Baseline explicável** (physics-informed).
        
    - **Modelagem ML/DL** de maior capacidade.
        

---

## 2. Fundamentos Teóricos — Wr/Wm

### 2.1 Equação geral do desgaste

W=k⋅νn⋅δm⋅exp⁡(−τTcrit)⋅f(p1,p2,…)W = k \cdot \nu^n \cdot \delta^m \cdot \exp\left(- \frac{\tau}{T_{\text{crit}}} \right) \cdot f(p_1, p_2, \ldots)

**Termos:**

- **W**: taxa de desgaste [M·L⁻²·T⁻¹]
    
- **k**: constante de proporcionalidade
    
- **ν**: velocidade do fluxo gás-sólido [L·T⁻¹]
    
- **δ**: diâmetro médio das partículas [L]
    
- **τ**: temperatura de operação [K]
    
- **Tcrit**: temperatura crítica [K]
    
- **f(·)**: função adimensional com propriedades do material e condições de operação
    
- **n, m**: expoentes que indicam a contribuição de velocidade e tamanho de partículas.
    

### 2.2 Especializações

**Refratário (Wr):**

Wr=α⋅ν2⋅δ1⋅exp⁡(−τTcrit)⋅g(σmat,ρfuel)Wr = \alpha \cdot \nu^2 \cdot \delta^1 \cdot \exp\left(- \frac{\tau}{T_{\text{crit}}} \right) \cdot g(\sigma_{\text{mat}}, \rho_{\text{fuel}})

**Metálico (Wm):**

Wm=β⋅ν3⋅δ0.5⋅exp⁡(−τTopt)⋅h(σsteel,μgas)Wm = \beta \cdot \nu^3 \cdot \delta^{0.5} \cdot \exp\left(- \frac{\tau}{T_{\text{opt}}} \right) \cdot h(\sigma_{\text{steel}}, \mu_{\text{gas}})

**g(·)** e **h(·)** são funções adimensionais:

- g=σmat/ρfuelg = \sigma_{\text{mat}} / \rho_{\text{fuel}}
    
- h=σsteel/μgash = \sigma_{\text{steel}} / \mu_{\text{gas}}
    

---

## 3. Adequações para MVP

### 3.1 Índices adimensionais

Como não temos dados absolutos de desgaste (mm ou kg) para calibrar α e β, adotaremos índices relativos:

Ir(t)=W^r(t)Wrref,Im(t)=W^m(t)WmrefI_r(t) = \frac{\widehat{W}_r(t)}{W_r^{\text{ref}}}, \quad I_m(t) = \frac{\widehat{W}_m(t)}{W_m^{\text{ref}}}

**Definição de Wref:**

- Mediana de W^\widehat{W} durante operação normal (status_operacao = 1) por zona.
    
- Alternativas: percentil 60–70 ou janela "benchmark".
    

Quando houver evento medido de desgaste, recalibraremos Wref para converter índices em valores absolutos.

### 3.2 Proxies

- **ν_proxy**: derivado de vazões de ar, ΔP em fornalha e recirculação.
    
- **δ_proxy**: de indicadores de peneiramento, loop seal, ΔP ciclone↔fornalha.
    
- **τ**: médias ponderadas por zona (densa/diluída/backpass).
    

---

## 4. Itens e Artefatos

### 4.1 Arquivos base

- **Consolidado:** `dados_operacao_consolidado.csv` (17.448 × 150)
    
- **Derivados:**
    
    - `dados_em_operacao_normal.csv` (status_operacao = 1)
        
    - `dados_fora_operacao_normal.csv` (status_operacao = 0)
        
    - `resumo_split_operacao.csv` (auditoria)
        

### 4.2 Resultados esperados

- `correlacoes_altas_*.csv` (pares com |r| ≥ 0,90)
    
- `mapa_grupos_correlacao.csv` (grupos + membros + agregados)
    
- `dados_agreg_wr.csv` (agregados mean/std por grupo)
    
- `features_fisicas.csv` (ν_proxy, δ_proxy, τ por zona)
    
- `dicionario_proxies.csv` (fórmulas e variáveis)
    
- `indices_adimensionais.csv` (Ir, Im por zona)
    

---

## 5. Caminho de Trabalho (Plano de Ataque)

### Caminho A — Physics-informed

- Regressão log-linear + GAM para estimar parâmetros.
    
- Alta interpretabilidade e baseline para comparação.
    

### Caminho B — ML/DL

- Modelos como GBM, TCN, Transformers temporais para risco e RUL.
    
- Incorporação fraca da teoria via features físicas.
    

### Caminho C — Híbrido

- Núcleo teórico multiplicado por correções aprendidas.
    
- Usado para otimização multiobjetivo de setpoints.
    

---

## 6. Próximos Passos

1. **EDA final**:
    
    - Matriz de correlação.
        
    - Formação de grupos (|r| ≥ 0,90).
        
    - Criação de agregados mean/std mantendo `flw_total_*` e descartando parciais.
        
2. **Proxies físicos**:
    
    - Calcular ν_proxy, δ_proxy, τ.
        
    - Gerar features_fisicas.csv e dicionario_proxies.csv.
        
3. **Índices adimensionais**:
    
    - Calcular Wr_hat, Wm_hat e Ir, Im.
        
    - Salvar indices_adimensionais.csv.
        

---

## 7. Disclaimer

- Índices são relativos até calibração.
    
- Proxies iniciais serão refinados com dados futuros.
    
- PCA e autoencoder só para a trilha ML/DL.
    

---

Se quiser, já posso usar esse documento como **referência fixa** para as próximas etapas do EDA e geração dos datasets curados.