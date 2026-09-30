# CP2 SERS - Energias renováveis e aprendizado de máquina

Entrega do checkpoint com duas tarefas independentes:

- classificação da fonte renovável a partir de dados da ANEEL/SIGA;
- regressão da radiação solar a partir da API histórica Open-Meteo.

## Dados

### Classificação

Arquivo: `aneel_classificacao_orange.csv`

Origem: SIGA/ANEEL. Cada linha representa um empreendimento de geração no Brasil. As entradas usadas foram `potencia_kw`, `latitude` e `longitude`; o alvo foi `fonte`
- Classes: Hidráulica=1476, Solar=1200, Eólica=1200

### Regressão

Arquivo: `meteo_regressao_orange.csv`

Origem: Open-Meteo histórico para Petrolina (PE), de 01/04/2025 a 30/06/2025, entre 7h e 17h. As entradas usadas foram `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh` e `hora`; o alvo foi `radiacao_w_m2`.

- Linhas: 1001
- Ausentes: 0
- Divisão: primeiras 80% das horas para treino e últimas 20% para teste

## Como executar

```bash
pip install -r requirements.txt
jupyter notebook checkpoint_energia_ml.ipynb
```

Execute o notebook de cima para baixo. Os CSVs já estão no repositório.

## Modelos e resultados

### Classificação

Foram treinados três classificadores. A divisão treino/teste foi estratificada, com 80% para treino, 20% para teste e semente 42.

As métricas multiclasses usam média `weighted`, adequada ao desbalanceamento entre as classes.

| modelo | accuracy | precision_weighted | recall_weighted | f1_weighted |
| --- | --- | --- | --- | --- |
| Random Forest | 0.9678 | 0.9684 | 0.9678 | 0.9677 |
| KNN | 0.9601 | 0.9606 | 0.9601 | 0.9599 |
| Regressão Logística | 0.8247 | 0.8313 | 0.8247 | 0.8233 |

Melhor resultado: **Random Forest**, com F1 weighted de **0.9677**.

### Regressão

Foram treinados três regressores. A avaliação preserva a ordem temporal.

| modelo | MAE | MSE | R2 |
| --- | --- | --- | --- |
| Random Forest | 67.2010 | 7478.8690 | 0.8410 |
| KNN Regressor | 73.9180 | 8344.3270 | 0.8220 |
| Regressão Linear | 145.2050 | 30034.2010 | 0.3600 |

Melhor resultado: **Random Forest**, com MAE de **67.20 W/m²**.

## Orange

O arquivo `Fluxos_para_Classificacao_e_Regressao.ows` contém os fluxos para Orange Data Mining. Também foram adicionadas imagens legíveis dos fluxos:

- `orange_classificacao.png`
- `orange_regressao.png`

## Conclusões

Na classificação, localização e potência ajudam a separar parte dos empreendimentos, especialmente porque as fontes tendem a se concentrar em regiões diferentes. A limitação principal é que esses três atributos não descrevem relevo, disponibilidade do recurso natural, tecnologia, fase do projeto ou detalhes regulatórios.

Na regressão, a hora do dia foi decisiva, pois a radiação segue o ciclo solar. Variáveis meteorológicas como nuvens, temperatura e umidade ajustam esse comportamento. Mesmo assim, radiação solar não é geração elétrica: a geração depende de painel, área instalada, inclinação, eficiência, perdas, sombreamento e operação do sistema.
