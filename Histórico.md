# CHECKPOINT DE ETAPA – PROJETO OVATION

## 📍 Identificação
- **Etapa:** 1 – Planejamento para carregamento e verificação dos arquivos
- **Data/Hora:** 08/08/2025 - 18:58
- **Responsável:** [Usuário]
- **Protocolo ativo:** PROTOCOLOS_UNIFICADOS_V2.md
- **MODO ULTRA-HF-000:** ATIVADO

---

## 📄 Contexto técnico
Definido o plano detalhado para execução da Etapa 1, que consiste no carregamento e verificação dos arquivos `ovation_dados_curado.csv` e `campos_formatados-simples.csv`.  
O objetivo é confirmar a estrutura, consistência e integridade dos dados antes de criar o mapeamento de colunas.  
Nenhuma leitura, escrita ou modificação será feita neste momento pela IA — apenas fornecimento de código para execução no ambiente local do usuário.

---

## 📂 Arquivos envolvidos
1. `ovation_dados_curado.csv` – CSV com 154 colunas (`col_001` até `col_154`), dados curados.
2. `campos_formatados-simples.csv` – CSV com descrições e unidades das variáveis, base para criar `nome_curto` e `nome_curto_com_unidade`.

---

## ✅ Validações concluídas
- [x] Escopo e estrutura dos arquivos definidos.
- [x] Plano de validação documentado.
- [x] Restrições de protocolo aplicadas.
- [ ] Estrutura real dos arquivos confirmada (pendente execução local pelo usuário).
- [ ] Número real de colunas/linhas verificado (pendente execução local pelo usuário).
- [ ] Presença das colunas essenciais (`descricao_limpa`, `unidade_padrao`) confirmada (pendente execução local pelo usuário).

---

## ⚠️ Restrições aplicáveis
- Nenhuma execução no ambiente da IA.
- Nenhuma modificação no dataset original.
- Nenhuma inferência automática.
- Somente decisões validadas com evidência objetiva.

---

## 🔜 Próxima ação planejada
Execução local do código de leitura e verificação da estrutura dos dois arquivos, conforme plano definido:
1. Carregar `ovation_dados_curado.csv` e confirmar que contém exatamente 154 colunas.
2. Carregar `campos_formatados-simples.csv` e confirmar que contém no mínimo 154 linhas.
3. Validar presença das colunas `descricao_limpa` e `unidade_padrao`.
4. Exibir amostras iniciais (5 primeiras colunas do primeiro arquivo e 5 primeiras linhas do segundo).

---

## 🗒 Observações adicionais
- Etapa 1 é exclusivamente de validação; qualquer criação de mapeamento (`df_nomes_colunas`) só será feita na Etapa 2, após aprovação.
- Este checkpoint pode ser usado como ponto oficial de retomada caso haja pausa prolongada.

# ✅ CHECKPOINT – FECHAMENTO DA ETAPA DE ENGENHARIA DE NOMES E ESTRUTURAÇÃO INICIAL

**Data de registro:** (preencher manualmente)  
**Responsável:** (preencher manualmente)  
**Pipeline:** A1 – Curadoria Técnica de Variáveis e Estruturação Semântica  
**Notebook de origem:** `a1_engenharia_dados.ipynb` (início oficial)

---

## 🔹 OBJETIVO DA ETAPA

Padronizar, corrigir e consolidar os dados técnicos da operação da caldeira CFB para uso futuro em modelagem de desgaste (`Wr`, `Wm`), análise exploratória e projetos de previsão com IA (ML/DL).

---

## 🔸 RESULTADOS CONSOLIDADOS

### 📁 Arquivos Gerados

1. **`ovation_dados_curado_v2_com_nomes.csv`**  
   - Dados totalmente corrigidos (alinhamento de colunas, correção de headers, nomes curtos).
   - Coluna `datahora` validada e padronizada.
   - Referência segura para segmentações e seleção de blocos futuros.

2. **`dados_para_cura.csv`**  
   - Subconjunto segmentado de dados com foco em curadoria e engenharia inicial.
   - Contém colunas com nomes curtos + `datahora`.

3. **`ovation_mapeado_nome_unidade_classificado_v2.csv`**  
   - Dicionário estendido com as colunas:
     - `point_name`, `descricao`, `unidade`
     - `nome_curto_com_unidade` (chave de ligação)
     - `categoria` (`operacao_direta` ou `consequencia`)
     - `wr_wm_role` (manual, ainda não validado)
     - `wr_wm_role_heuristica` (auto-sugerido)

---

## 🧠 LÓGICA ADOTADA

### 1. `tipo_variavel`  
Classificação em:
- `operacao_direta`: Variáveis manipuladas diretamente pela operação (ex.: vazões, setpoints, velocidades).
- `consequencia`: Variáveis que refletem o comportamento do sistema (ex.: temperaturas, pressões medidas).

Critérios:
- Regras objetivas com base em nomes e unidades (ex.: `t/h`, `Nm3/h`, `rpm`).

### 2. `wr_wm_heuristica`  
Classificação heurística automática com base em substrings e padrões no nome da variável:
- Ex: `bed_temp`, `coal_flow`, `recirc`, `o2`, `dp`, `steam_flow`, `emission`, `rpm`, etc.
- Categoria `unknown` atribuída quando nenhum padrão é identificado.

---

## 🧾 AÇÕES FUTURAS (PRÉ-PLANEJADAS)

- Geração da versão `dados_para_cura_header3.csv`, com 3 linhas de cabeçalho:
  1. Cabeçalho original (nomes curtos)
  2. Linha `tipo_variavel`
  3. Linha `wr_wm_heuristica`

- Criação do notebook `a1_eda.ipynb` com:
  - Estatísticas descritivas
  - Validação da coerência entre classificações
  - Seleção inicial de candidatos a variáveis explicativas

---

## ⚙️ STATUS DE ETAPA

✅ Etapa encerrada e validada. Pronto para seguir para **EDA + definição de proxy de desgaste (Wr/Wm)**.  
Nenhum erro estrutural ou semântico pendente nos dados atuais.

---
Segue o **checkpoint** formatado no padrão de registro para o seu Obsidian, documentando a decisão de modelo de salvamento adotado.

---

## **CHECKPOINT – MODELO DE SALVAMENTO DE DADOS DE OPERAÇÃO**

**Data:** 2025-08-09  
**Projeto:** A1 – Operação e Paradas  
**Responsável:** Wilson Melo  
**Status:** Aprovado e implementado

---

### **1. Contexto**

Durante o tratamento e segmentação dos dados de operação, definimos um critério único para classificar o status de cada ponto temporal:

- **Em operação normal (1):** `flw_total_c_t_h > 40`
    
- **Fora de operação normal (0):** `flw_total_c_t_h <= 40` ou valores `NaN`
    

Esse critério substituiu os anteriores e passou a ser **regra única** para qualquer derivado.

---

### **2. Modelo de Salvamento**

Foi adotado um modelo **híbrido** de persistência:

1. **Arquivo consolidado**:
    
    - Nome: `dados_operacao_consolidado.csv`
        
    - Contém **todas as observações** (operação normal e fora de operação normal), com coluna `status_operacao` (`1` ou `0`) para filtragem futura.
        
    - Serve como **única fonte de verdade** para derivações.
        
2. **Arquivos derivados** (gerados automaticamente a partir do consolidado):
    
    - `dados_em_operacao_normal.csv` → apenas registros com `status_operacao == 1`.
        
    - `dados_fora_operacao_normal.csv` → apenas registros com `status_operacao == 0`.
        
    - Facilitam análises e gráficos sem necessidade de filtro manual.
        
3. **Arquivo de auditoria**:
    
    - `resumo_split_operacao.csv` → resumo de contagem e proporção de cada classe, validando se os derivados batem com os valores do consolidado.
        

---

### **3. Estrutura de Pastas**

```
A1_LOCAL/
  data/
    staged/
      dados_operacao_consolidado.csv
      dados_em_operacao_normal.csv
      dados_fora_operacao_normal.csv
      resumo_split_operacao.csv
      antigos/  ← backups de versões anteriores
```

---

### **4. Regras de Operação**

- O consolidado **nunca é sobrescrito manualmente** — apenas via pipeline definido.
    
- Arquivos anteriores ao consolidado são movidos para `staged/antigos/`.
    
- Qualquer nova derivação parte exclusivamente do consolidado.
    
- O `status_operacao` é calculado sempre via **limiar fixo de 40 t/h** em `flw_total_c_t_h`.
    

---

### **5. Próximos Passos**

- Utilizar `dados_em_operacao_normal.csv` para análises de performance e variação interna.
    
- Utilizar `dados_fora_operacao_normal.csv` para análises de paradas e eventos críticos.
    
- Manter registro de cada atualização do consolidado neste mesmo formato no Obsidian.
    

---


---

## **Checkpoint – Estado Atual do Projeto A1 (Predição de Desgaste CFB)**

### **Base e Estrutura**

- Arquivo atual de trabalho:  
    `dados_operacao_consolidado_dimcorrigida.csv`  
    (local: `data/staged/`)
    
- Colunas **protegidas**:
    
    - Timestamp (coluna original `Timestamp`, preservando maiúscula inicial)
        
    - `status_operacao`
        
    - `flw_total_a_t_h`
        
    - `flw_total_b_t_h`
        
    - `flw_total_c_t_h`
        
- Dimensão física extraída a partir do sufixo do nome curto (ex.: `kpa`, `adegc`, `nm3_h`, `%`).
    
- Indicadores “sem dimensão” tratados como `%` quando aplicável.
    
- Prefixos de **parciais** conhecidos (`flw_output_of_belt_coal_fdr_`) são removidos antes do agrupamento.
    

### **Objetivo Atual**

Aplicar **redução de cardinalidade com consistência dimensional**, usando:

- Correlação de Pearson absoluta por dimensão.
    
- Limite: **|r| ≥ 0,90**.
    
- Para cada grupo correlacionado:
    
    - Criar variáveis **média** (`mean_<dim>_Gxxx`) e **desvio padrão** (`std_<dim>_Gxxx`).
        
    - Remover as colunas originais do grupo, exceto as protegidas.
        
- Garantir que totais A/B/C **nunca sejam removidos**.
    
- Preservar ordem final: Timestamp no início, `status_operacao` no fim.
    

### **Saídas Definidas**

Todos os arquivos vão para `output/eda/`:

- `correlacoes_<dim>.csv` → Matriz de correlação absoluta por dimensão.
    
- `mapa_grupos_<dim>.csv` → Mapeamento de grupos correlacionados.
    
- `curado_dimensional.csv` → Dataset final único reduzido (base para curated).
    
- `resumo_correlacoes_por_dimensao.csv` → Estatísticas de redução por dimensão.
    

### **Status da Execução**

- Código revisado e ajustado para:
    
    - Remover parciais antes do cálculo.
        
    - Proteger variáveis essenciais.
        
    - Evitar erros por colunas duplicadas ou já removidas.
        
- Última ação: Código final montado para execução completa da redução com consistência dimensional.
    
- Próxima etapa: Rodar o script para gerar as saídas finais.
    

---

Se usarmos este checkpoint, o próximo chat começa **diretamente na execução da redução** e geração dos arquivos de saída, sem necessidade de relembrar decisões anteriores.

Quer que eu já salve esse checkpoint também em formato **Markdown para download**?