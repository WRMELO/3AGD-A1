# Plano de ataque (A1): predição de desgaste em CFB e orientação operacional

Você tem: (i) uma base teórica sólida com equações de desgaste para refratários e metais em CFB, com variáveis-chave como velocidade do escoamento gás‑sólido (ν), diâmetro médio de partículas (δ) e temperatura (τ), além de termos adimensionais que resumem propriedades de materiais e gás; (ii) ~150 sinais de sensores por ~2 anos (~24k amostras). Isso é perfeito para testar abordagens complementares de modelagem e controle. As sugestões abaixo usam a teoria como “fio condutor” e depois escalam de modelos explicáveis a modelos de maior capacidade.

## Premissas e alvos

- **Alvo 1:** estimar taxa de desgaste por região (fase densa, diluída, backpass) e/ou **RUL** (Remaining Useful Life) de componentes.
    
- **Alvo 2:** **orientar a operação** (setpoints) para reduzir desgaste sem violar metas de carga, eficiência e emissões.
    

A teoria fornece forma funcional e variáveis físicas: equação geral de desgaste W=k⋅νn⋅δm⋅e−τ/Tcrit⋅f(⋅)W = k \cdot \nu^n \cdot \delta^m \cdot e^{-\tau/T_{crit}} \cdot f(\cdot)W=k⋅νn⋅δm⋅e−τ/Tcrit​⋅f(⋅) e especializações para refratário WrW_rWr​ e metálico WmW_mWm​ com funções adimensionais g(σmat,ρfuel)g(\sigma_{mat},\rho_{fuel})g(σmat​,ρfuel​) e h(σsteel,μgas)h(\sigma_{steel},\mu_{gas})h(σsteel​,μgas​). 0_teoria_combustao_e_de… 0_teoria_combustao_e_de… 0_teoria_combustao_e_de…

---

## Caminho A — Modelo “physics‑informed” supervisionado (explicável)

**Ideia:** começar com um **modelo paramétrico** alinhado à teoria, estimando k,n,mk, n, mk,n,m e eventualmente partes de g(⋅),h(⋅)g(\cdot), h(\cdot)g(⋅),h(⋅) a partir dos dados operacionais. Isso dá interpretabilidade e verificação física.

1. **Engenharia de variáveis guiada pela teoria**
    
    - Reconstruir séries de ννν (relacionada a velocidades/pressões diferenciais de ar/partículas), δδδ (tamanho médio das partículas, proxies de peneiramento/recirculação), τττ (temperaturas de cama/backpass), O2O_2O2​, excesso de ar, taxas de recirculação interna/externa (proxies via aeração do loop seal, ΔP ciclone↔fornalha). A literatura interna descreve o papel da recirculação e do loop seal e como ajustes de ar afetam a combustão, úteis como features operacionais. 0_teoria_combustao_e_de… 0_teoria_combustao_e_de… 0_teoria_combustao_e_de…
        
    - Construir termos **log‑lineares**: log⁡W↔log⁡k+nlog⁡ν+mlog⁡δ−τ/Tcrit+log⁡f(⋅)\log W \leftrightarrow \log k + n\log\nu + m\log\delta - \tau/T_{crit} + \log f(\cdot)logW↔logk+nlogν+mlogδ−τ/Tcrit​+logf(⋅). 0_teoria_combustao_e_de… 0_teoria_combustao_e_de…
        
2. **Modelos**
    
    - Regressão **linear/elastic net** sobre a forma logarítmica; depois **GAM** para permitir suavidades em g,hg,hg,h.
        
    - Variante **bayesiana hierárquica** com pooling por zona (densa/diluída/backpass), que ajuda a estabilizar estimativas por região e dá intervalos de credibilidade.
        
3. **Rótulos (ground truth)**
    
    - Taxas de perda de espessura/masa (WR/WM) por inspeções, UT/boroscopia, trocas de refratário, falhas. Quando medido de forma esparsa, usar **nowcasting** de desgaste entre inspeções com estado latente suavizado.
        
4. **Saída para operação**
    
    - Regras explicáveis: sensibilidade de WWW a ν,τ,δν, τ, δν,τ,δ. Ex.: reduzir ννν efetivo em pontos de erosão do backpass ou ajustar aeração do loop seal para redistribuir carga de partículas quando WWW projetado exceder limite. 0_teoria_combustao_e_de… 0_teoria_combustao_e_de…
        

**Prós:** alta interpretabilidade; rápida validação contra a teoria; bom baseline.  
**Contras:** pode subajustar não linearidades fortes e interações complexas.

---

## Caminho B — Série temporal multivariada de alta capacidade para RUL e hotspot

**Ideia:** usar DL/ML de séries temporais para aprender padrões operacionais complexos e prever **RUL** e/ou **risco de desgaste** por componente.

1. **Modelos candidatos**
    
    - **Gradient Boosting/Random Forest** em janelas temporais agregadas (lags, estatísticas).
        
    - **Temporal Convolutional Network (TCN)** ou **Transformers temporais** (p.ex., TFT) com atenção por variáveis.
        
    - **Sobrevivência/risco**: modelos de hazard (DeepSurv/GBM de sobrevivência) para RUL entre manutenções.
        
2. **Incorporação fraca da teoria**
    
    - Injetar features “físicas” ν,δ,τ,e−τ/T\nu, \delta, \tau, e^{-\tau/T}ν,δ,τ,e−τ/T e indicadores de recirculação/excesso de ar; usar **penalização** ou **features auxiliares** para guiar o modelo onde dados são escassos. 0_teoria_combustao_e_de…
        
3. **Saída para operação**
    
    - Heatmaps de **hotspots de desgaste** previstos por zona; **alertas de RUL**; envelope operacional recomendável para carga, O2, ar primário/ secundário, setpoints do loop seal.
        

**Prós:** captura interações não lineares e atrasos; bom para previsão de eventos.  
**Contras:** menos explicável; requer mais cuidado com validação e drift.

---

## Caminho C — Gêmeo cinza (gray‑box): modelo híbrido + otimização de setpoints

**Ideia:** tratar as equações Wr,WmW_r, W_mWr​,Wm​ como **núcleo** e aprender **correções** Δ(⋅)\Delta(\cdot)Δ(⋅) dependentes de sinais operacionais (ex.: fouling, composição de combustível) que a teoria não cobre. Em seguida, usar o modelo para **otimizar setpoints**.

1. **Forma híbrida**
    
    - W^=Wteoˊrico(ν,δ,τ,...)⋅exp⁡(Δθ(X))\hat W = W_{\text{teórico}}(\nu,\delta,\tau,...) \cdot \exp(\Delta_\theta(X))W^=Wteoˊrico​(ν,δ,τ,...)⋅exp(Δθ​(X)), onde Δθ\Delta_\thetaΔθ​ é um modelo leve (GAM/MLP pequeno). As formas Wr,WmW_r, W_mWr​,Wm​ estão no documento com definições de g,hg,hg,h. 0_teoria_combustao_e_de… 0_teoria_combustao_e_de…
        
2. **Otimização operacional (multiobjetivo)**
    
    - Problema: minimizar desgaste previsto sujeito a restrições de carga, eficiência, O2, NOx/SOx. Saída: **recomendações de setpoints** (ar primário/ secundário, aeração do loop seal, distribuição de combustível) por horizonte curto.
        

**Prós:** respeita a física e melhora aderência aos dados; gera recomendações operáveis.  
**Contras:** exige calibração cuidadosa por zona e bom mapeamento de atuadores→variáveis.

---

## Protocolo de comparação (A/B/C)

1. **Auditoria e alinhamento dos dados (S0)**
    
    - Mapa de sensores→zona (densa/diluída/backpass); frequências, latências, perdas; reconciliação de tags de operação/ manutenção.
        
    - Construção de proxies de ν,δν, δν,δ, recirculação interna/externa, excesso de ar, ΔP em ciclone↔fornalha, conforme a lógica operacional descrita. 0_teoria_combustao_e_de… 0_teoria_combustao_e_de…
        
2. **Definição de rótulos (S1)**
    
    - Taxa de desgaste por inspeção/UT; quando ausente, eventos de manutenção como marcos e interpolação entre inspeções.
        
3. **Baselines e variantes (S2)**
    
    - A: regressão log‑linear + GAM.
        
    - B: GBM/TCN/TFT para risco/RUL.
        
    - C: híbrido multiplicativo com Δ(⋅)\Delta(\cdot)Δ(⋅).
        
4. **Métricas**
    
    - MAE/RMSE para taxa de desgaste; **concordância de ranking** de hotspots; **C‑index**/Brier para RUL; impacto operacional simulado: redução % de WWW vs. variação em eficiência/O2/NOx.
        
5. **Validação temporal e por zona**
    
    - Split por tempo; validação cruzada por “campanhas” de operação; análise separada para fase densa, diluída e backpass. Tabelas do documento ajudam a estratificar parâmetros por região. 0_teoria_combustao_e_de…
        
6. **Teste de orientação operacional**
    
    - “What‑if” com o modelo: variar ar primário/ secundário, aeração do loop seal, recirculação, e medir ΔW^\Delta \hat WΔW^ e efeitos em O2/combustão. 0_teoria_combustao_e_de…
        

---

## Dados que vamos precisar consolidar antes do S2

- **Matriz de features** por zona com ν,δ,τ,O2ν, δ, τ, O_2ν,δ,τ,O2​, proxies de recirculação interna/externa e excesso de ar; séries de ΔP, vazões, setpoints e composição do combustível. 0_teoria_combustao_e_de…
    
- **Rótulos**: espessura/taxas de desgaste ou RUL por componente e data; eventos de troca/reparo.
    
- **Máscara de qualidade**: lacunas, sensores em falha, recalibrações.
    
- **Mapa de atuadores**: quais alavancas de operação afetam ννν, recirculação e τττ.
    

---

## Próximo passo sugerido

Se concordar com os três caminhos e com o protocolo de comparação, começamos pelo **S0/S1**: inventário dos sensores, mapeamento por zona e definição dos rótulos disponíveis. Em seguida, levantamos rapidamente um **baseline A** (log‑linear + GAM) para checar aderência da teoria aos seus dados e estabelecer o piso de desempenho; depois seguimos para B e C. Quer que eu detalhe o checklist de S0/S1 com os campos exatamente esperados a partir dos seus 150 sinais?