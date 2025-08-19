perfeito — vou direto ao ponto, usando o que já está no seu ambiente: notebook `a1_physics_based_development.ipynb`, o **PROTOCOLOS v2** (HF-000 ativo), o **HISTÓRICO** e o **padrão da tabela**. Abaixo está o plano para “fechar” a linha **physics-based** com critério técnico, mínima variação e particionamento temporal para calibração/validação/teste.

# 1) Onde estamos (estado consolidado)

- **Dados e máscara de baseline:** já existem janelas de baseline por interseção e a máscara (`baseline_mask.csv`).
    
- **Bases canônicas:** `a1_physics_informed*.csv` (curated), além de `*_proxies.csv` e artefatos em `outputs/baseline_datasets`.
    
- **Dicionário único de nomes (`nome_pt`)** aplicado e auditado; inventário aponta poucos **não mapeados** residuais.
    
- **Padrão de apresentação** definido (tabela com Wr, Wm, referências e índices normalizados).
    

# 2) Próximos passos — versão final do physics-based

## 2.1 Fechar o dicionário e o padrão de colunas

- **Garantir presença** no dicionário das colunas usadas no padrão de apresentação (τ_densa, τ_diluida, τ_backpass, ΔP proxy, O₂ excesso, vazões, etc.).
    
- **Zerar os “não mapeados”** nas principais fontes (`data/curated/*` e `outputs/baseline_datasets/*`), reexecutando a normalização de cabeçalhos com backup `_old.csv`.  
    **Saídas**
    
- `outputs/auditoria_normalizacao_features.csv` (verde para as principais fontes).
    

## 2.2 Delta-proxy (ΔP representativo da cama)

- **Triagem objetiva** das colunas de pressão/diferencial (regex auditáveis) no _curated_ para formar um conjunto candidato.
    
- **Normalização intra-coluna** (z-score robusto por janela deslizante ou por baseline).
    
- **Agregação robusta**: mediana das **Top-K** colunas mais consistentes (K pequeno: 3–5) → `delta_proxy`.
    
- **Referência de baseline**: `delta_proxy_ref` calculado **na mesma janela** do baseline (por zona/componente, quando aplicável).  
    **Saídas**
    
- `outputs/pedra/DELTA_PROXY_DIAGNOSTICS.md` (quais colunas entraram, K, correlações, estabilidade).
    
- `data/curated/a1_physics_informed_proxies.csv` **atualizado** com `delta_proxy` (+ `_ref`).
    

## 2.3 Cálculo de Wr e Wm (com referências)

> Princípio: índices **adimensionais** por normalização ao baseline; pesos monotônicos alinhados à física (semestatística oportunista).

- **Drivers normalizados** (adimensionais):
    
    - Temperaturas: `I_tau_* = τ_* / τ_*_ref` (densa, diluída, backpass).
        
    - Oxigênio: `I_o2 = O2_excesso / O2_ref`.
        
    - Vazões: `I_flow = flow_C / flow_ref` (ou total/primário conforme o proxy físico de interesse).
        
    - Cama: `I_dp = delta_proxy / delta_proxy_ref`.
        
- **Estrutura paramétrica e transparente** (multiplicativa e monotônica):
    
    - `Wr_idx = Π I_k ^ w_k` (mais sensível a **temperaturas** e **O₂**).
        
    - `Wm_idx = Π J_k ^ v_k` (mais sensível a **ΔP (carga/cama)** e **vazões** → erosão/impacto).
        
- **Referências absolutas**: `Wr_ref` e `Wm_ref` = estatística robusta (mediana) de `Wr` e `Wm` **dentro do baseline** por `zona × componente`.
    
- **Validação dimensional**: todos os termos adimensionais, índices > 0, e coerência física (ex.: queda de O₂ não pode “reduzir” desgaste refratário em regime de sobreaquecimento, etc.).  
    **Saídas**
    
- `outputs/pedra/PEDRA_ROSETA_WR_WM.md` (formulação, pesos, justificativas, checks de coerência).
    
- Colunas finais no dataset canônico: `Wr`, `Wm`, `Wr_ref`, `Wm_ref`, `Wr_idx`, `Wm_idx`.
    

## 2.4 Ajuste para **mínima variação** (sem perder resposta física)

> Objetivo: suavizar ruído e oscilação espúria, preservando _degraus reais_ do processo.

- **Filtragem**: teste entre **mediana móvel** (janelas curtas), **EWMA** (λ escolhido por critério objetivo) e **Hampel** para outliers.
    
- **Função-objetivo (sem rótulos):**
    
    - Minimizar **coeficiente de variação (CoV)** de `Wr_idx` e `Wm_idx` em períodos de operação **quase estacionária** (tag via `status_operacao==1` + estabilidade de carga).
        
    - Penalizar **energia de alta frequência** (soma dos |Δ|) e preservar **degraus** acima de limiar (detectados em `I_dp`, `I_flow` ou `I_tau`).
        
- **Seleção de λ/janela** por busca em grade pequena (determinística) com relatório de trade-offs (variância vs. resposta a degraus).  
    **Saídas**
    
- `outputs/pedra/SMOOTHING_REPORT.md` (janela/λ escolhidos, métricas antes/depois, exemplos).
    

## 2.5 Particionamento de dados (calibração, validação, teste)

> Physics-based não “aprende” parâmetros livres como um ML, mas **calibra pesos** e **suavização**. Para isso:

- **Baseline**: _não entra_ em calibração; serve **exclusivamente** para referências.
    
- **Split temporal, com embargo** (sem fuga de informação):
    
    - **Treino/Calibração**: primeiros **60%** do período **pós-baseline**.
        
    - **Validação**: próximos **20%** (verificação de mínima variação e coerência).
        
    - **Teste (hold-out)**: últimos **20%** (avaliação final).
        
    - **Embargo temporal** entre blocos (ex.: 60 min) para evitar bleed-over de estados do processo.
        
- **Estratificação operacional**: garantir que cada bloco cubra **faixas de carga** (quantis de `flw_total_c_t_h`) e condições de ar (ex.: `total_air_flow_knm3_h`).
    
- **Eventos exógenos**: se houver manutenções/perturbações marcadas, **respeitar fronteiras** (não dividir no meio de uma intervenção).  
    **Saídas**
    
- `outputs/pedra/SPLIT_REPORT.md` (datas absolutas de cada bloco, cobertura de cargas, embargo aplicado).
    

## 2.6 Métricas de aceitação (sem rótulos de falha)

- **Estabilidade**: CoV de `Wr_idx`/`Wm_idx` em regime estacionário (alvo: mínimo).
    
- **Sensibilidade**: amplitude do índice durante **degraus operacionais** típicos (deve aumentar em coerência com a física).
    
- **Coerência cruzada**: correlação saudável com `I_tau`, `I_dp`, `I_flow` sob mudanças controladas.
    
- **Limites**: ausência de explosões/zeros; percentis 5/95 dentro de faixas plausíveis definidas na revisão final.
    

## 2.7 Tabela final de apresentação (padrão)

Gerar a tabela exatamente no seu formato (uma linha por `timestamp × zona × componente`), com:

- **Primárias**: `flw_total_c_t_h`, `total_air_flow_knm3_h`, `total_paf_air_flow_knm3_h`, `te_of_hot_pri_air_in_aph_outl_adegc` (ou seus `nome_pt` equivalentes).
    
- **Secundárias**: `delta_proxy`, `tau_densa`, `tau_diluida`, `tau_backpass`, `o2_excess_pct`.
    
- **Saídas e refs**: `Wr`, `Wm`, `Wr_ref`, `Wm_ref`, `Wr_idx`, `Wm_idx`.  
    **Saídas**
    
- `outputs/physics_model_outputs.csv` (completo).
    
- `outputs/physics_model_outputs_sample.md` (5–10 linhas ilustrativas com explicações de cada coluna).
    

## 2.8 Governança e versionamento

- **Hash** do dicionário após qualquer adição aprovada (registrar no HISTÓRICO).
    
- **Checklist** de entrega com caminhos absolutos e _timestamps_ dos artefatos acima.
    
- **Checkpoint** no `HISTÓRICO.md` ao final desta etapa.
    

# 3) Critérios claros para as escolhas

- **Modelo multiplicativo adimensional**: mantém monotonicidade, interpretabilidade e combinações físicas (produtos de razões).
    
- **Mínima variação**: sintonizada por função-objetivo explícita (estabilidade vs. degraus), não por tentativa-erro.
    
- **Split temporal com embargo**: evita vazamento; garante avaliação honesta da calibragem.
    
- **Estratificação por carga**: evita “vencer” por operar só em uma faixa.
    

# 4) Riscos & mitigação

- **Não mapeadas residuais**: reexecutar a auditoria após a atualização do dicionário; bloquear (fail-fast) em `data/curated/*`.
    
- **Colunas removidas por engano**: `*_old.csv` permite restauração imediata.
    
- **Ausência de marcadores operacionais** (manutenção/eventos): usar _heurísticas de degrau_ em `I_dp`/`I_flow` para avaliar sensibilidade.
    

# 5) Ações imediatas (execução direta, sem perguntas)

1. Atualizar o dicionário (garantir τ/ΔP/O₂/vazões presentes) e **reprocessar cabeçalhos** nas fontes principais.
    
2. Implementar a **triagem ΔP** e gravar `delta_proxy` + `delta_proxy_ref`.
    
3. Gerar `Wr`, `Wm` e índices (`Wr_idx`, `Wm_idx`) com a formulação transparente.
    
4. Rodar a **sintonização de mínima variação** e fixar a janela/λ escolhidos.
    
5. Aplicar o **split temporal com embargo** e emitir o `SPLIT_REPORT.md`.
    
6. Exportar a **tabela final de apresentação** e registrar tudo em checkpoint no `HISTÓRICO.md`.
    

se quiser, eu já escrevo o **checkpoint-modelo** para colar no histórico assim que finalizarmos as etapas 2.2 → 2.7.


# Regra de ouro

**Tudo que vira “referência” é calculado exclusivamente no baseline e congelado.**  
Depois, **todas** as janelas fora do baseline usam essas referências congeladas para os cálculos.

# O que sai do baseline (e fica congelado)

- **Referências de processo** (estatística robusta, tipicamente mediana):
    
    - `tau_densa_ref`, `tau_diluida_ref`, `tau_backpass_ref`
        
    - `o2_excesso_ref`
        
    - `flow_ref` (ex.: `flow_C_ref` ou o proxy definido)
        
    - `delta_proxy_ref`
        
- **Índices de referência dos compostos**:
    
    - `Wr_ref`, `Wm_ref` (avaliados dentro do baseline para conferência; servem de âncora)
        
- **Parâmetros de suavização** (mínima variação):
    
    - janela/λ escolhidos por **minimizar o CoV** em trechos **quase estacionários** **dentro do baseline** (ex.: `status_operacao==1` + estabilidade de carga).
        
    - Esses parâmetros são **fixados** e reaplicados fora do baseline.
        
- **Limites operacionais/thresholds** que dependam do nível do processo (se existirem).
    

# Como aplicar fora do baseline

- Índices adimensionais sempre como **razão ao baseline**:
    
    - `I_tau_* = tau_*/tau_*_ref`, `I_o2 = o2_excesso/o2_excesso_ref`, `I_flow = flow/flow_ref`, `I_dp = delta_proxy/delta_proxy_ref`
        
- Com esses `I_*`, calcular:
    
    - `Wr_idx = ∏ I_k ^ w_k`
        
    - `Wm_idx = ∏ J_k ^ v_k`
        
- A **mesma suavização** (λ/janela) escolhida no baseline é aplicada aos sinais/índices fora do baseline.
    
- **Baseline não entra** em treino/validação/teste; serve só para **referência**. Os splits temporais (60/20/20 com embargo) são **pós-baseline**.
    

# Entrega técnica (objetiva)

1. Materializar um artefato `baseline_refs` (ex.: JSON/CSV) com todas as `*_ref` + parâmetros de suavização.
    
2. Em qualquer cálculo fora do baseline, ler `baseline_refs` e aplicar as fórmulas acima.
    
3. Se o baseline oficial mudar, **recalcular** `baseline_refs`, gerar novo hash e registrar no HISTÓRICO.
    

Se quiser, eu já listo exatamente quais colunas do seu curated entram em cada referência e a ordem das transformações para virar o `baseline_refs`.