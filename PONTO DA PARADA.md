
perfeito — **MODO ULTRA-HF-000 ativo**. Vou propor o plano de trabalho **physics‑based** para previsão de desgaste (Wr/Wm) usando o baseline que você já consolidou. Também vou listar, com precisão, **quais insumos ainda são necessários** para garantir rastreabilidade e evitar qualquer suposição.

# PLANO DE VERIFICAÇÃO

1. Confirmar os arquivos‑fonte e máscaras oficiais (inputs e período baseline).
    
2. Fixar definições operacionais de **Wr_ref** e **Wm_ref** (derivadas do baseline) e dos **índices adimensionais** Wr/Wr_ref, Wm/Wm_ref.
    
3. Auditar a disponibilidade de variáveis necessárias para proxies físicos (fluxo de carvão _no forno_, ar primário/ secundário, ΔP cama, τ‑zonas, O₂, e—se possível—velocidades/áreas).
    
4. Validar inventário de colunas com **significado físico** (documento “NOME DAS VARIÁVEIS E SEUS SIGNIFICADOS.md”) → checagem 1‑a‑1 no CSV.
    
5. Verificar existência de dados **externos** para calibração/validação (manutenções, inspeções, trocas de refratário/placas, falhas, intervenções).
    
6. Somente após (1–5) concluído: iniciar modelagem de **taxas relativas** (MVP) e avaliação.
    

---

# PLANO DE TRABALHO (Fases e Entregáveis)

## Fase 0 — Congelamento e bases (pronto/rodando)

- **Entrada**:  
    `data/curated/a1_physics_informed_proxies.csv` (com `coal_flow_furnace_t_h`, `air_total_nm3_h`, partições, τ’s, O₂).  
    `outputs/baseline_janelas_intersection.csv` + `outputs/baseline_mask.csv`.
    
- **Ação**: baseline já consolidado (Intersection_Longest) → máscara aplicada.
    
- **Saídas**:  
    `outputs/baseline_datasets/physics_baseline_proxies.csv` (in‑baseline)  
    `outputs/baseline_datasets/physics_offbaseline_proxies.csv` (off)  
    `outputs/CHECKPOINT_BASELINE_FISICO.md` (feito).
    

## Fase 1 — Definições físicas e índices adimensionais (MVP)

- **Objetivo**: Operar com **taxas relativas** (não mm/kg) enquanto não há medição direta.
    
- **Definições**:
    
    - **Wr_ref, Wm_ref**: vetores/escalares de referência **por zona** (densa/diluída/backpass e materiais correlatos) extraídos do baseline (média/mediana das proxies físicas).
        
    - **Wr_idx = Wr / Wr_ref** e **Wm_idx = Wm / Wm_ref** (≥0).
        
- **Proxies iniciais (ajustáveis)**:
    
    - **Carga erosiva (particulado/gás)** ~ f(Flow C, ar total, partição prim/2ª, ΔP cama, τ_diluida)
        
    - **Severidade térmica (refratário)** ~ f(τ_densa, τ_backpass, gradientes com O₂, eventuais transientes)
        
    - **Química (metálico)** ~ f(O₂, SO₂ pós/ pré‑FGD como _flags_)
        
- **Saídas**:  
    `outputs/indices/indices_wr_wm_baseline.csv` (Wr_ref/Wm_ref + fórmulas adotadas)  
    `outputs/indices/indices_wr_wm_timeseries.csv` (Wr_idx, Wm_idx por timestamp/zonas)
    

## Fase 2 — Mapeamento espacial e agregação por componentes

- **Objetivo**: ligar **zonas τ** a **componentes** (refratário vs metálico).
    
- **Ação**:
    
    - Tabela **zona → componente/material** (ex.: densa→refratário de leito; diluída→revest/placas; backpass→superfícies convectivas metálicas/ciclones).
        
    - Se existirem medições duplicadas por lado (A/B), manter índices por lado e **agregar** por componente lógico.
        
- **Saídas**:  
    `outputs/mapeamentos/zona_para_componente.csv`  
    `outputs/indices/indices_wr_wm_por_componente.csv`
    

## Fase 3 — Calibração física rápida (sem geometria → com geometria se houver)

- **Objetivo**: calibrar pesos das proxies usando o baseline e períodos “bons” vs. “ruins” (off‑baseline).
    
- **Caminhos**:
    
    - **Sem geometria (MVP)**: regressão regularizada/GLM com _priors físicos_ para obter pesos de combinação nas proxies (apenas para **Wr_idx**/**Wm_idx**).
        
    - **Com geometria (se disponível)**: converter vazões em **velocidades** (A=área de duto/seção), estimar **fluxo de partículas** (carvão/cinzas), e usar formas clássicas de erosão (Finnie/Bitter) e _thermal‑shock_ para refratário → melhora do significado físico.
        
- **Saídas**:  
    `outputs/calibracao/calibracao_wr_wm_mvp.json` (pesos/hiperparâmetros)  
    `outputs/calibracao/diagnosticos.pdf` (resíduos, sensibilidade, estabilidade)
    

## Fase 4 — Validação operacional

- **Objetivo**: confrontar **Wr_idx/Wm_idx** com evidências de campo.
    
- **Fontes**: logs de inspeção, trocas/reparos, ganho de perdas de carga, tendência de ΔP, aumento de taxa de cinzas, _heat rate_ penalizado, etc.
    
- **Métricas**: AUC/precision para “eventos de manutenção”, correlação com indicadores independentes, estabilidade por campanha.
    
- **Saídas**:  
    `outputs/validacao/val_wr_wm_eventos.csv`  
    `outputs/validacao/relatorio_validacao.md`
    

## Fase 5 — Operacionalização e governança

- **Objetivo**: produzir **sinal diário** por componente, com alarmes.
    
- **Entregáveis**:
    
    - `outputs/monitor/wr_wm_daily.csv` (agregado diário; P50/P95)
        
    - Faixas de **atenção/ação** (ex.: Wr_idx>1.2 por ≥N horas)
        
    - Painel simples (notebook/CSV) + instruções de uso
        

---

## Fórmulas/heurísticas iniciais (claras e auditáveis, **ajustáveis**)

_(MVP, tudo **sem** suposições fora dos dados disponíveis; pesos calibráveis)_

- **Carga erosiva (E*)** ~ `k1 * norm(coal_flow_furnace_t_h) + k2 * norm(air_total_nm3_h) + k3 * norm(air_secondary_nm3_h)`
    
- **Severidade térmica refratário (T*_ref)** ~ `a1 * norm(tau_densa) + a2 * norm(tau_backpass)`
    
- **Severidade térmica metálico (T*_met)** ~ `b1 * norm(tau_diluida) + b2 * norm(tau_backpass)`
    
- **Oxidação/combustão incompleta (Q*)** ~ `c1 * norm(o2_excess_pct)` _(apenas como modulador/flag; sem substituir medição direta)_
    
- **Wr_idx (refratário)** ~ `wE*E\* + wT*T\*_ref` (normalizado por **Wr_ref** do baseline)
    
- **Wm_idx (metálico)** ~ `mE*E\* + mT*T\*_met + mQ*Q\*` (normalizado por **Wm_ref**)
    

> `norm(x)` = padronização por baseline (z‑score ou escala [0,1] usando P05–P95 do baseline).  
> **Wr_ref, Wm_ref**: os valores de referência (média/mediana no baseline) por **zona/componente**.

---

## O que **ainda** preciso (para eliminar qualquer suposição)

1. **Geometria/áreas** dos dutos/zonas principais (para converter vazões → **velocidade**).
    
    - Seções em backpass, entradas de ciclones, saídas de fornalha A/B, dutos de ar primário/secundário.
        
2. **Materiais** por zona/componente:
    
    - Tipo de refratário (densidade, dureza, resistência a choque térmico) e **espessuras**;
        
    - Materiais metálicos (ligas) em cada superfície exposta.
        
3. **Histórico de manutenção/inspeção** (datas, localização, descrição, severidade) para **validação factual**.
    
4. **Qualidade do combustível** por campanha (PCS, cinzas, enxofre, umidade, granulometria) para **efeito de partícula/química**.
    
5. **Mapa de sensores → zonas** (garantir que τ_densa/τ_diluida/backpass estejam corretamente ancoradas em regiões físicas).
    
6. (Opcional) **Ca/S operacional** e **injeção de calcário** (para avaliar agressividade química/abrasão adicional).
    

> Se (1)–(6) **não** estiverem disponíveis, seguimos no **MVP** com índices **adimensionais** (já válidos para decisão relativa: desvio ↑/↓ vs baseline). Com esses insumos, evoluímos para um **modelo semiadimensional/absoluto** (mais explicável e transferível).

---

## Artefatos que vou produzir (com nomes fixos)

- `outputs/indices/indices_wr_wm_baseline.csv` (Wr_ref, Wm_ref por zona/componente)
    
- `outputs/indices/indices_wr_wm_timeseries.csv` (Wr_idx, Wm_idx por timestamp)
    
- `outputs/mapeamentos/zona_para_componente.csv`
    
- `outputs/calibracao/calibracao_wr_wm_mvp.json` + `diagnosticos.pdf`
    
- `outputs/validacao/relatorio_validacao.md`
    
- `outputs/monitor/wr_wm_daily.csv`
    

---

## Critérios de aceite (MVP)

- Baseline aplicado de forma reprodutível; índices **Wr_idx/Wm_idx** calculados sem NAN sistêmico (>99% linhas válidas no baseline e >95% fora).
    
- Validação: **sinal antecipa** ou **correlaciona** com ao menos 60–70% dos eventos de manutenção relevantes (regra ajustável).
    
- Documentação: todos os passos e pesos versionados e rastreáveis.
    

---

Quer que eu já **codifique a Fase 1** (gerar `indices_wr_wm_baseline.csv` e `indices_wr_wm_timeseries.csv` a partir dos seus arquivos atuais), enquanto você separa os itens (1)–(6) que conseguir recuperar? Se preferir, eu também preparo um **template CSV** para você preencher (geometrias, materiais e inspeções) — isso reduz o atrito e mantém o ULTRA sem suposições.

Perguntar ao ChatGPT