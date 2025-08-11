A ideia do **baseline físico** é fixar um “retrato de referência” do regime térmico do CFB em **operação estável**, contra o qual compararemos variações futuras.  
Usaremos os proxies:

- `tau_densa`, `tau_diluida`, `tau_backpass` (e `tau_global`) — regime térmico por zonas.
    
- (quando existirem) `v_proxy` — intensidade/carga global.
    
- (quando existirem) `delta_proxy` — regime fluidodinâmico (ΔP).
    

Critério de estabilidade (auditatório e ajustável):

1. filtrar outliers grossos por **quantis** (ex.: 1%–99%) dos `tau_*`;
    
2. exigir **baixa variabilidade local** (desvio-padrão rolante pequeno) nas séries de `tau_*`;
    
3. exigir **persistência** mínima do estado estável (ex.: janelas consecutivas de N amostras).
    

Com isso, geramos:

- `physics_baseline.csv` → médias e dispersões dos proxies no estado estável;
    
- `baseline_janelas.csv` → janelas/intervalos temporais que compuseram o baseline (auditoria).